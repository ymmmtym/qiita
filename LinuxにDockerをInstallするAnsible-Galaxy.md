---
title: LinuxにDockerをInstallするAnsible-Galaxy
tags: Ansible Docker Linux Ubuntu
author: yumenomatayume
slide: false
---
<!-- # LinuxにDockerをInstallするAnsible-Galaxy -->

[Linux](https://scrapbox.io/yumenomatayume/Linux)に[Docker](https://scrapbox.io/yumenomatayume/Docker)をインストールするたびに、

「ubuntu docker install」
「centos docker install」

などをググって、Docker公式サイトより手動でインストールしていました。

毎回インストールするのは手間なので、自動化したいと思っていた矢先、
[OSS](https://scrapbox.io/yumenomatayume/OSS)で素晴らしいansible-galaxyを見つけましたので、紹介したいと思います。

## 使用するansible-galaxy

Ansible Galaxy: [https://galaxy.ansible.com/geerlingguy/docker](https://galaxy.ansible.com/geerlingguy/docker)
GitHub: [https://github.com/geerlingguy/ansible-role-docker](https://github.com/geerlingguy/ansible-role-docker)

## Ansible Playbookを書いてみる

VirtualBox上にUbuntu20.04のVMを作成します。
[Vagrant](https://scrapbox.io/yumenomatayume/Vagrant)を用いて、Provisioning Toolとして[Ansible](https://scrapbox.io/yumenomatayume/Ansible)を使用し、そのPlaybookでDockerをインストールします。

### Vagrantfileを作成する

以下の`Vagrantfile`を作成します。  
IPアドレスは`192.168.0.3/24`としてあります。

```Vagrantfile
# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.define "ansible-docker" do |node|
    node.vm.box = "bento/ubuntu-20.04"
    node.vm.hostname = "ansible-docker"
    node.vm.network "private", ip: "192.168.0.3", netmask: "255.255.255.0"
    node.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
    end
  end

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "./install_docker.yml"
    ansible.inventory_path = "./inventory.ini"
    ansible.limit = 'all'
  end
end
```

### roleをインストールする

rolesディレクトリを作成して、その中に今回使用するroleをインストールします。

```bash
mkdir roles
ansible-galaxy install geerlingguy.docker -p roles/
```

### InventoryとPlaybookを作る

`inventory.ini`

```ini
[ansible-docker]
192.168.0.3
```

`install_docker.yml`

```yaml
---
hosts: all
  roles:
    role: geerlingguy.docker
    become: yes
```

## Playbookを実行する

ディレクトリ構造は以下のようになっています。

```console
.
├── Vagrantfile
├── install_docker.yml
├── inventory.ini
└── roles
     └── geerlingguy.docker
         ├── LICENSE
         ├── README.md
         ├── defaults
         │   └── main.yml
         ├── handlers
         │   └── main.yml
         ├── meta
         │   └── main.yml
         ├── molecule
         │   └── default
         │       ├── converge.yml
         │       └── molecule.yml
         └── tasks
             ├── docker-compose.yml
             ├── docker-users.yml
             ├── main.yml
             ├── setup-Debian.yml
             └── setup-RedHat.yml
```

以下のコマンドでVM作成します。(ホストOSでAnsible実行環境が整備されている必要があります。)

```bash
vagrant up
```

`docker`,`docker-compose`がインストールされていることを確認できます。

```console
vagrant@ansible-docker:~$ sudo docker -v
Docker version 19.03.12, build 48a66213fe
vagrant@ansible-docker:~$ sudo docker-compose -v
docker-compose version 1.26.0, build d4451659
```

