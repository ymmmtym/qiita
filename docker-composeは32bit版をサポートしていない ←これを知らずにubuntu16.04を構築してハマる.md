---
title: docker-composeは32bit版をサポートしていない ←これを知らずにubuntu16.04を構築してハマる
tags: Ubuntu 32bit Docker docker-compose
author: yumenomatayume
slide: false
---
<!-- docker-composeは32bit版をサポートしていない ←これを知らずにubuntu16.04を構築してハマる -->

# はじめに
自宅に古いwindows10(32bit)があったので、自宅サーバ用途で使うためにubuntu16.04を入れました。
スペックは、下記の通り低スペックです

- Intel(R) Core(TM) i3 CPU         530  @ 2.93GHz
- メモリ 4GB

また、開発環境を整えるために、下記の最低限の要件を満たすサーバを構築していきます

1. ローカル端末(mac)からssh接続出来る
2. dockerが使用できる
3. docker-composeが使用できる

以下、参考にしたサイトをあげておきます。
https://www.server-world.info/query?os=Ubuntu_16.04&p=download

## 0.Download+とInstall
下記サイトの「Server install image」からisoイメージをDLして、適当なLIVE USBやらDVDに書き込みましょう
http://releases.ubuntu.com/16.04/

私の場合は、「32-bit PC (i386) server install image」をDLしました。
ちなみにubuntuは17.04から32bitのサポートをしていないので、32bitのPCで使用できるubuntuの最新ver.は16.04になります。
(有志で18.04の32bit版も開発されるかもですが、今回は公式なimageで構築していきます)

# 1.ローカル端末(mac)からssh接続出来る

## ネットワークの設定

```bash;ubuntu16.04
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master br0 state UP group default qlen 1000
    link/ether 40:61:86:9a:81:eb brd ff:ff:ff:ff:ff:ff
    inet6 fe80::4261:86ff:fe9a:81eb/64 scope link
       valid_lft forever preferred_lft forever
```

IPアドレスはデフォルトで上記になっていたので、/etc/network/interfacesを下記に修正


```/etc/network/interfaces
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto enp2s0

iface enp2s0 inet static
address 192.168.0.2
network 192.168.0.0
netmask 255.255.255.0
broadcast 192.168.0.255
gateway 192.168.0.1
dns-nameservers 8.8.8.8
```

最後にプロセスの再起動、もしくはubuntu自体を再起動すると、同一ネットワークに属する端末からログイン可能になります。

```ubuntu16.04
sudo service networking restart

# もしくは

reboot
```


```ローカル端末
ssh 192.168.0.2 -l <ubuntuのログインユーザ>
```
これで、要件1を満たせました。


# 2.dockerのインストール
```ubuntu16.04
$ sudo apt -y install docker.io
$ sudo docker -v
Docker version 18.09.2, build 6247962
```

無事にdockerがインストールされました。
このままでも使用できますが、sudo権限なしで実行できるように下記のコマンドを打ちます

```ubuntu16.04
sudo groupadd docker
sudo gpasswd -a ${USER} docker
sudo systemctl restart docker
```
再ログインすれば適用されます

# 3.docker-composeのインストール
## 3.1.aptでインストールする方法
aptを用いてdocker-composeをインストールします

```ubuntu16.04
$ sudo apt -y install docker-compose
$ docker-compose exec hoge bash

ERROR: Version in "./docker-compose.yml" is unsupported. You might be seeing this error because you're using the wrong Compose file version. Either specify a version of "2" (or "2.0") and place your service definitions under the `services` key, or omit the `version` key and place your service definitions at the root of the file to use version 1.
For more on the Compose file format versions, see https://docs.docker.com/compose/compose-file/
```

コマンド実行時に、version:'3'(docker-compose.ymlの記載)が使えないと怒られてしまったので、docker-composeのversionを確認する

```ubuntu16.04
$ docker-compoes -v
docker-compose version 1.8.0, build unknown
```

versionが古く、version:'3'の記述が使えないとのこと、、
version:'2'なら動作するのですが、環境によってdocker-compose.ymlの中身を変更するのも手間なので、
公式サイト(https://docs.docker.com/compose/install/)通りにgithubリポジトリにあるバイナリからインストールしてみる

## 3.2.githubのリポジトリからインストールする方法
公式サイト(https://docs.docker.com/compose/install/)通りにコマンドを打っていきます

```ubuntu16.04
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                               Dload  Upload   Total   Spent    Left  Speed
0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0Warning: Failed to create the file .: Is a directory
0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
curl: (23) Failed writing body (0 != 9)

$ uname -m
i686
```

curlでこけました。`uname -m`の中身が32bitとなっており、docker-composeでは32bit版のバイナリファイルは存在しないようです(詰んだ)

# 4.対処法について
結論から言うと、64bit版でないとdocker-composeの最新versionは使用できません。

色々と調べていくと、`/proc/cpuinfo`を見れば64bit版に対応しているか分かるらしい。
具体的には、flagsにlmがあればいいとのことなので、
下記コマンドを実行し、どこかに*lm*が表示されれば64bit版に対応していることになります。

```ubuntu16.04
$ grep flags /proc/cpuinfo
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf pni dtes64 monitor ds_cpl vmx est tm2 ssse3 cx16 xtpr pdcm sse4_1 sse4_2 popcnt lahf_lm pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid dtherm arat flush_l1d
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf pni dtes64 monitor ds_cpl vmx est tm2 ssse3 cx16 xtpr pdcm sse4_1 sse4_2 popcnt lahf_lm pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid dtherm arat flush_l1d
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf pni dtes64 monitor ds_cpl vmx est tm2 ssse3 cx16 xtpr pdcm sse4_1 sse4_2 popcnt lahf_lm pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid dtherm arat flush_l1d
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf pni dtes64 monitor ds_cpl vmx est tm2 ssse3 cx16 xtpr pdcm sse4_1 sse4_2 popcnt lahf_lm pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid dtherm arat flush_l1d
```

lm.....あった！

まさか64bitに対応しているとは思いませんでした(初めからこれを確認すればよかった、、)
と言うことで、32bitは諦めてubuntu(64bit)を入れ直して再チャレンジしてみます。

