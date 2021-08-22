---
title: Vagrantでawait_response_stateエラーが発生し、共有フォルダの同期がとれない
tags: Vagrant Windows Ubuntu SSH
author: yumenomatayume
slide: false
---
# Vagrantでawait_response_stateエラーが発生し、共有フォルダの同期がとれない

vagrant up/reloadしたらネットワーク関係のエラーが発生し、
VMは立ち上がりますが共有フォルダが空ディレクトリになり、利用できなくなりました。

```powershell
==> default: Configuring and enabling network interfaces...
C:/HashiCorp/Vagrant/embedded/gems/2.1.2/gems/net-scp-1.2.1/lib/net/scp.rb:398:in `await_response_state': Agent pid 1839 (RuntimeError)

# snip...
```

Vagrantfile内容

```ruby:Vagrantfile
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  
  config.vm.define "ubuntu" do |node|
    node.vm.box = "bento/ubuntu-18.04"
    node.vm.hostname = "ubuntu"
    node.vm.network "public_network", ip: "192.168.0.3", netmask: "255.255.255.0", bridge: "en1: Wi-Fi (AirPort)"
    node.vm.synced_folder "../html", "/var/www/html", :mount_options => [ "dmode=777", "fmode=777" ]
    node.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
    end
  end
end
```

## 動作環境

- Windows 10 Home(Host OS)
- vagrant == 2.1.2
- Ubuntu18.04(Guest OS)

## 原因について

**VMにSSHしたときに標準出力がある場合**は`await_response_state`エラーが発生します。

```bash
Last login: Wed Aug  5 07:48:04 2020 from 192.168.1.1
Agent pid 3480
vagrant@ubuntu:~$
```

私の場合、上記の出力のように`~/.bashrc`に`ssh-agent`の設定をしていたため、
ログイン時に`Agent pid`が出力されるため、エラーとなってしまいました。

```bash:~/.bashrc
eval `ssh-agent`
```

## 対応策

`~/.bashrc`を`ssh-agent`設定の出力を捨てることで、vagrant upでエラーがでなくなりました。
(ここでは、標準出力と標準エラー出力を出さないように設定)

```bash:~/.bashrc
eval `ssh-agent` > /dev/null 2&>1
```

## Reference

[Vagrantでawait_response_stateエラー](https://blog.freedom-man.com/vagrant-await_response_state)
[vagrant up/reload で「scp.rb:nnn:in `await_response_state' ... (RuntimeError)」エラー](http://les-r-pan.hatenablog.jp/entry/2017/09/04/105455)

