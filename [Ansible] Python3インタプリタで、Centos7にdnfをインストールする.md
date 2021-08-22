---
title: [Ansible] Python3インタプリタで、Centos7にdnfをインストールする
tags: Ansible Python dnf Yum CentOS
author: yumenomatayume
slide: false
---
# [Ansible] Python3インタプリタで、Centos7にdnfをインストールする

## dnfとは

[fedora公式ページ](https://fedoraproject.org/wiki/DNF?rd=Dnf)

- Dandified Yum(ダンディファイド ヤム)とも言うらしい
- Fedoraのパッケージ管理システム(バージョン22からデフォルトになった)
- RHEL7,CentOS7でも使用できる(`yum install -y dnf`)
- RHEL8,CentOS8では標準でインストール済み

簡単に言うと、`dnf`は`yum`の後継種であり、CentOS8からは`yum`ではなく`dnf`がデフォルトとなる。
CentOS8でも`yum`コマンドは使えるが、`dnf`へのシンボリックリンクになっているだけである。

## なぜyumではなくdnfを使うのか

2020/1/1にPython2.Xのサポートが終了した。

- yumは、python2.Xでしか動作しません
- dnfは、python2.x or 3.Xで動作します

なのでこれからは`dnf`とが主流となっていく。

## (まずは)手動でCentOS 7にインストールしてみる

CentOS 7.XのDockerコンテナにdnfをインストールする。

```
[dnfuser@localhost ~]$ docker run -it --rm centos:7 bash

[root@3b4a088d7e6f /]# yum update -y
[root@3b4a088d7e6f /]# yum install -y epel-release
[root@3b4a088d7e6f /]# yum install -y dnf

[root@3b4a088d7e6f /]# head -n1 /usr/bin/dnf
#!/usr/bin/python2
```

どうやらCentOS7.Xの`dnf`はpython2.Xで動いている様子。
python3.Xをインストールした後にも試したが、`dnf`は2.Xになってしまう。

## Ansibleでdnfをインストールする

<https://docs.ansible.com/ansible/latest/modules/dnf_module.html>

インタプリタがPython2.Xであれば、すんなりインストールできる。

```yml:playbook.yml
- name: Install epel-release
  yum:
    name: epel-release
    state: present
  become: yes

- name: Install dnf
  yum:
    name: dnf
    state: present
  become: yes
```

1. yumモジュールでdnfをインストール
2. その後のタスクはyumモジュールをdnfモジュールに変更すれば使用できる

### (本題)Ansibleのインタプリタをpython3.Xにしている場合

```cfg:ansible.cfg
[default]
ansible_python_interpreter=/usr/bin/python3
```

上記のように、デフォルトでPython3.Xのインタプリタを使用している場合は、無理やり2.Xに戻す必要がある。

```yml:playbook.yml
- name: Install dnf for Centos7
  block:
    - name: Change interpreter
      set_fact:
        ansible_python_interpreter: "/usr/bin/python"

    - name: Install epel-release
      yum:
        name: epel-release
        state: present
      become: yes

    - name: Install dnf
      yum:
        name: dnf
        state: present
      become: yes
  when:
    - ansible_distribution_major_version | float < 8
```

その後のタスクでもインタプリタは2.Xになる。

## 最後に(Ansibleインタプリタ ベストプラクティス)

Ansible 2.12 ver.では、デフォルトのインタプリタの振る舞いが変わるらしい。
(多分Python3.Xが優先的に選ばれるのであろう)

参考）
<https://docs.ansible.com/ansible/latest/reference_appendices/interpreter_discovery.html>
<https://rheb.hatenablog.com/entry/ansible_interpreter_discovery>

現行ver.(2.9)ではデフォルトで`/usr/bin/python`が指定されるので、OSによって使用されるPythonのver.が違う場合がある。
2.12 ver.が待てなくて、`dnf`コマンドを使いたい人のベストプラクティスは以下のようになった。

### 諦めてデフォルト(Python2.X)のインタプリタを使用する

1. Ansibleに関しては、デフォルトの`/usr/bin/python`を使用して、Python2.Xを使い続ける
2. yumモジュールではなく、dnfモジュールに統一する

「別にPython2.Xでいいよ」と言う人は、デフォルトのインタプリタを使用すれば問題ない。

### どうしてもPyhton3.Xのインタプリタを使用したい場合

**CentOS7.Xのホストに対して、inventoryでPython2.Xインタプリタを指定する**

```yml:inventory.yml
all:
  children:
    centos7:
      hosts: centos7-1
      vars:
        ansible_python_interpreter: /usr/bin/python # /usr/bin/python2 でもOK
```

CentOS7だけグルーピングしておけば、簡単に実装できそう。

**Playbookで強制的にインタプリタを2.Xに変更する**

```yml:playbook.yml
- name: Change interpreter
  set_fact:
    ansible_python_interpreter: /usr/bin/python
```

ちなみに私は、Python3.Xがデフォルトになるのに備えて、こちらを採用している。

