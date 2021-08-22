---
title: Vagrantのゴミを削除する
tags: Vagrant VirtualBox Mac
author: yumenomatayume
slide: false
---
## ゴミを削除する方法まとめ

1. `vagrant global-status`で、ゴミを確認する
2. `vagrant global-status —-prune`で、vagrant上のゴミを削除する
3. `VBoxManage unregistervm ${VM}`で、VMを削除する

## はじめに

Vagrantを使っていると、使わなくなったものを放置して、そのままディレクトリを削除してしまうことがよくあります。

`vagrant destroy`を実行してVMを削除しないと、VagrantやVirtualboxの中にゴミが残ってしまいます。
ディレクトリを復活させれば`vagrant destroy`コマンドが実行できるようになりますが、それが出来ない場合はゴミだけを消します。

## 実行環境

- MacOS: 10.15.7
- Vagrant: 2.2.9
- virtualbox: 6.1.8

Macにvirtualboxとvagrantを入れて使用しています。詳細環境は以下になります。

```bash
$ sw_vers
ProductName:	Mac OS X
ProductVersion:	10.15.7
BuildVersion:	19H524

$ vagrant --version
Vagrant 2.2.9

$ VBoxManage --version
6.1.8r137981
```

## Vagrantのゴミを削除

`vagrant global-status`コマンドを実行すると、Macにある全てのVagrant VMを確認することが出来ます。これにはゴミも含まれます。

```bash
$ vagrant global-status
 
id       name    provider   state    directory                                                    
--------------------------------------------------------------------------------------------------
d032dbc  web     virtualbox running  /var/tmp/web/vagrant            # ゴミ
5186e88  default virtualbox running  /var/tmp/vagrant-centos8-sample # ゴミ
33fbe44  web     virtualbox poweroff /opt/webserver                  

The above shows information about all known Vagrant environments
on this machine. This data is cached and may not be completely
up-to-date (use "vagrant global-status --prune" to prune invalid
entries). To interact with any of the machines, you can go to that
directory and run Vagrant, or you can use the ID directly with
Vagrant commands from any directory. For example:
"vagrant destroy 1a2b3c4d"
```

`vagrant global-status --prune`を実行すると、ゴミのVM情報が削除されて、vagrantで管理されているVMのみ表示することができます。

これ以降は既にゴミが削除されているので、—pruneオプションを外してもゴミは表示されません。

```bash
$ vagrant global-status --prune
 
id       name    provider   state    directory                                                    
--------------------------------------------------------------------------------------------------
33fbe44  web     virtualbox poweroff /opt/webserver                  

The above shows information about all known Vagrant environments
on this machine. This data is cached and may not be completely
up-to-date (use "vagrant global-status --prune" to prune invalid
entries). To interact with any of the machines, you can go to that
directory and run Vagrant, or you can use the ID directly with
Vagrant commands from any directory. For example:
"vagrant destroy 1a2b3c4d"
```

しかし、virtualbox上にあるゴミのVM自体は削除されません。

また、出力例にある通り`vagrant destroy ${id}`とすることで、指定のディレクトリに移動しなくても1つずつVMを削除することができます。

`vagrant global-status`には出力されなくなりますが、
そもそもディレクトリが存在しない場合は、以下の様にVM自体削除することが出来ません。

```bash
$ vagrant destroy 89363d2
The working directory for Vagrant doesn't exist! This is the
specified working directory:
```

## virtualboxのゴミを削除

普通にGUIかコマンドから削除すればOKです。

コマンドの場合、`VBoxManage list vms`コマンドでvirtualboxにあるVM一覧が表示されるので、`VboxManage unregistervm ${VM名}`コマンドを実行して削除することができます。

```bash
$ VBoxManage list vms
"webserver_web_1609474127863_21675" {b7796093-c127-4f5c-b84b-70bc2500c73f}
"minikube" {c2febc29-b21c-44c7-8cd5-4d0738b6a708}
"GNS3 VM" {59b55fc5-3af4-4010-b7a7-7339d767edb3}
"trash" {9a3abb30-524b-4929-8d06-060813521796}

$ VboxManage unregistervm trash
```

## Reference

- [Vagrantで起動しているVMを一覧する - Qiita](https://qiita.com/ringo/items/e30761b89fb6c9a1c45d)
- [使ってないVagrant Boxを削除する - Qiita](https://qiita.com/mochizukikotaro/items/52f4434c3f69c4ba1f54)
- [不要な Vagrant 仮想マシンを削除する (vagrant destroy) | まくまくVagrantノート](https://maku77.github.io/vagrant/destroy-vm.html)
- [VirtualBoxのよく使うコマンドまとめ - ほたてメモ](http://hotatekun.hatenablog.com/entry/2016/07/11/095218)

