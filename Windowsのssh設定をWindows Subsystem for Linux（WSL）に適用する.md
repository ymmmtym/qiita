---
title: Windowsのssh設定をWindows Subsystem for Linux（WSL）に適用する
tags: Windows WSL Ubuntu SSH rsync
author: yumenomatayume
slide: false
---
# Windowsのssh設定をWindows Subsystem for Linux（WSL）に適用する

WindowsとWSL(Ubuntu20.04)ではホームディレクトリが違うため、Windowsのssh設定をそのまま使うことは出来ません。

ホームディレクトリを変更する方法などもあるみたいですが、ssh鍵などのパーミッションなどの問題がありうまくいかないため、
Linux標準のホームディレクトリを使用した自分なりのやり方をまとめてみます。

WSLの使用するOSバージョンによって、ホームディレクトリが変わる可能性がありますので、
使用する環境に合わせるようにお願いします。

## 動作環境

- Windows 10 Home
- OpenSSH 7.7.2.1
- Ubuntu20.04 (Windows Subsystem for Linux)

## WSLの起動

WSLを起動すると、各ドライブが`/mnt`ディレクトリにマウントされた状態になっており、
`/mnt/c/Users/${USER}`(Windowsのホームディレクトリ)が起動時のカレントディレクトリになります。
(私の場合はE:ドライブまでマウントされています)

```bash
ymmmtym@windows10:/mnt/c/Users/ymmmtym$ ls -l /mnt
total 0
drwxrwxrwx 1 ymmmtym ymmmtym 4096 Aug  9 19:41 c
drwxrwxrwx 1 ymmmtym ymmmtym 4096 Aug  9 19:41 d
drwxrwxrwx 1 ymmmtym ymmmtym  512 Jan  1  1980 e
```

また、`USER`(ymmmtym)と`HOSTNAME`(windows10)は引き継がれますが、ホームディレクトリが`/home/${USER}`変更されます。

```bash
ymmmtym@windows10:/mnt/c/Users/ymmmtym$ getent passwd ${USER}
ymmmtym:x:1000:1000:,,,:/home/ymmmtym:/bin/bash
```

そのため、ssh設定をWSLのホームディレクトリで内の`/home/${USER}/.ssh`に置く必要があります。

## Windows側のssh設定をWSLに適用する

結論、rsyncでホームディレクトリに`.ssh`ディレクトリのバックアップを作成するだけで使用できます。
パーミッションはバックアップ先で`600`となるように指定します。

```bash
rsync -av --delete --chmod=600 .ssh ~
```

### ssh設定の詳細

#### `~/.ssh`ディレクトリ

バックアップした`/home/${USER}/.ssh`ディレクトリは以下の構成であるとします。

```bash
ymmmtym@windows10:/mnt/c/Users/ymmmtym$ ls -l /home/${USER}/.ssh
-rw------- 1 ymmmtym ymmmtym   770 Jul  3 15:10 authorized_keys
-rw------- 1 ymmmtym ymmmtym  6608 Aug 10 13:09 config
-rw------- 1 ymmmtym ymmmtym  3326 Jul  3 15:10 id_rsa
-rw------- 1 ymmmtym ymmmtym   743 Jul  3 15:10 id_rsa.pub
-rw------- 1 ymmmtym ymmmtym 45573 Aug  7 17:04 known_hosts
```

#### `~/.ssh/config`ファイル

`/mnt/c/Users/${USER}/.ssh/config`は以下の内容になっています。
vagrant上の2VMにログインする際のssh設定が記載してあります。

```:~/.ssh/config
Host centos
    User vagrant
    Hostname 192.168.0.3
    IdentityFile ~/.ssh/id_rsa

Host ubuntu
    User vagrant
    Hostname 192.168.0.4
    IdentityFile ~/.ssh/id_rsa
    ProxyCommand ssh.exe -W %h:%p centos
```

一つ重要点として、**ProxyCommandの記載がWindowsの記載であったとしても問題なくssh出来る**ということです。

#### ssh実行

```bash=
ymmmtym@windows10:/mnt/c/Users/ymmmtym$ ssh ubuntu -vvv
OpenSSH_8.2p1 Ubuntu-4, OpenSSL 1.1.1f  31 Mar 2020
debug1: Reading configuration data /home/ymmmtym/.ssh/config
debug1: /home/ymmmtym/.ssh/config line 248: Applying options for ubuntu
debug1: Reading configuration data /etc/ssh/ssh_config
debug1: /etc/ssh/ssh_config line 19: include /etc/ssh/ssh_config.d/*.conf matched no files
debug1: /etc/ssh/ssh_config line 21: Applying options for *
debug2: resolve_canonicalize: hostname 192.168.211.4 is address
debug1: Executing proxy command: exec ssh.exe -W 192.168.211.4:22 centos
debug1: identity file /home/ymmmtym/.ssh/id_rsa type 0
debug1: identity file /home/ymmmtym/.ssh/id_rsa-cert type -1
debug1: Local version string SSH-2.0-OpenSSH_8.2p1 Ubuntu-4
Enter passphrase for key 'C:\Users\ymmmtym/.ssh/id_rsa':
```

内部的にWindowsで`ssh.exe`が起動して、`C:\Users\ymmmtym/.ssh/id_rsa`を読み込んでくれます。
（通常のLinuxではこのような動作はしません）

`~/.ssh/id_rsa`を読み込みたい時は、`~/.ssh/config`以下のLinuxの場合に修正する必要があります。

```:~/.ssh/config
# Windowsの場合
    ProxyCommand ssh.exe -W %h:%p centos
# Linuxの場合
    ProxyCommand ssh -W %h:%p centos
```

実際に標準のLinuxで行った場合は以下のエラーが返されます。

```bash
vagrant@linux:~$ ssh ubuntu
bash: Unknown command 'ssh.exe -W 192.168.0.4 centos'
bash:
exec ssh.exe -W 192.168.0.4 centos
     ^
ssh_exchange_identification: Connection closed by remote host
vagrant@linux:~$
```

### 同期設定

ssh設定を頻繁に修正する方は、cronを使用して同期することをお勧めします。

#### Windows起動時にWSLでcronを起動する

sudo実行時にパスワードを聞かれないように設定する。

```bash
sudo usermod -G sudo ${USER}
sudo visudo

# 以下の記載になるように修正する
%sudo   ALL=(ALL:ALL) NOPASSWD: ALL

```

`C:\Users\USERNAME\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup`フォルダに、
以下の`bat`ファイルを作成する。

```bat
wsl /bin/bash -l -c "sudo service cron start"
```

#### cronの設定をする

`crontab -e`を実行し、以下の行を追加する(以下は1時間毎に実行される例)

```
* */1 * * * /usr/bin/rsync -av --delete --chmod=600 .ssh ~
```

参考)

[WSL で cron を利用する方法・Windows 起動時に自動実行する方法](https://loumo.jp/archives/24595)
[同期設定(rsync)を今一度整理してみました](https://qiita.com/mitzi2funk/items/9308db56829d7b4cb90d)

## おまけ

rsyncやcronなどの面倒なことをせず単純に`.ssh`ディレクトリのコピーだけしたい場合は、
以下のコマンドを実行してください。

```bash
cp -r /mnt/c/Users/${USER}/.ssh /home/${USER}
chmod 700 ~/.ssh
chmod 600 ~/.ssh/*
chmod 644 ~/.ssh/*.pub
if [[ -f ~/.ssh/config ]]; then
  sed -i -e 's/ProxyCommand ssh\.exe /ProxyCommand ssh /g' ~/.ssh/config
fi
```

