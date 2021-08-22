---
title: Ansible Gathering Factsを上手く使う方法
tags: Ansible ansible-playbook
author: yumenomatayume
slide: false
---
AnsibleのGathering Factsについて、今まで熟知せずに使っていましたが、
内容を知ることで良質なplaybookが作れると感じました。

- これだけ知っておけば便利
- 結構ハマったこと

などを中心に記載します。

## 伝えたいこと

Gathering Factsとは

- 対象ホストの情報をansible_facts変数というものに格納し、tasks内の変数で使用することができる。
    - 格納された変数一覧はdebug/setupモジュールを使用することで確認できる。
- **playbook実行時**の初回に1度だけ実行される。
    - デフォルトで実行される。
        - **実行時の状態を格納する。**
    - 実行しない場合はplaybookに`gather_facts: no`と記載する。
        - playbookの実行時間が少し短くなる。
    - あるタスクで内で実行したい場合は、タスクとして記載する。
- tasks内で実行すると、実行時の環境で変数を取得する。
    - `become: yes`などを指定している場合は、実行ユーザなどの環境変数が異なる。
- 対象ホストが正常な状態でない場合にエラーになる可能性がある。

## Ansible Facts変数とは

ansible対象ホストのシステム情報などが格納された変数です。

playbookで対象ホストの情報(OS,Hostname,IP Address...)を使いたい時、この変数を用いて定義することができます。

例えば、OSがRHEL/Ubuntuによって実行したいtaskが異なる時、
Facts変数を使用することで、タスクを実行するか否かをplaybookで定義できます。

```yaml
- name: 'Install vim for CentOS 8'
  dnf:
    name: vim
    state: present
  when:
    - ansible_os_family == "Redhat"
    - ansible_distribution_major_version == "8"
```

`ansible_facts`変数一覧はdebug/setupモジュールを使用することで確認できます。

playbook内でdebugモジュールを使う

```yaml
debug: var=ansible_facts
```

ansible(CLI)でsetupモジュールを使う

```bash
ansible ${target_host} -m setup
```

## Playbook利用時のGathering Facts

Gathering Factsは、**playbook実行時**の初回に1度だけ実行されます。

デフォルトで実行されるため、実行しない場合はplaybookに`gather_facts: no`と記載します。
(playbook実行時間を少し減らすことができます。)

```yaml
- hosts: all
  gather_facts: no
  roles:
    - common
```

注意しておきたい点は、**実行時の状態を格納すること**です。
例えば、タスク内で `become: yes`で実行ユーザを切り替えた場合、格納されている変数は切り替え前のユーザのもので、切り替え後のユーザに更新されません。

task実行後にfact変数を上書きしたい場合は、以下のtaskを記載をします。

```yaml
- name: 'Gathering Facts'
  gather_facts:
```

## Gathering Factsがエラーになる場合

対象ホストからfacts変数を取得できない場合、エラーが発生します。
同様にsetupモジュールもエラーになります。
この時、対象ホストに何らかのエラーが発生しています。

試しに、対象ホストの `/etc/sudoers`にtypoを含ませてみます。

```
root		ALL = (ALL) ALL
%admin		ALL = (ALL) ALL
% typo		ALL = (ALL) ALL # わざとtypoさせた行
```

この状態でplaybookを実行するとエラーになります。

### 切り分け方法

以下のコマンドについて `gather_subset`を絞りながら実行し、
該当となるソースコードを見にいくことで、該当エラー箇所が特定できそうです。

```bash
ansible -m setup -a 'gather_subset=network,virtual,ohai,facter,hardware'
```

gather_factの処理は以下のソースコードになってます。

[ansible/lib/ansible/module_utils/facts at devel · ansible/ansible](https://github.com/ansible/ansible/tree/devel/lib/ansible/module_utils/facts)

## Reference

- [Ansible チュートリアル | Ansible Tutorial in Japanese](https://yteraoka.github.io/ansible-tutorial/)
- [変数の使用 — Ansible Documentation](https://docs.ansible.com/ansible/2.9_ja/user_guide/playbooks_variables.html)
- [gather_facts – Gathers facts about remote hosts — Ansible Documentation](https://docs.ansible.com/ansible/2.9_ja/modules/gather_facts_module.html)
- [Ansible実行が"GATHERING FACTS"で止まる - Qiita](https://qiita.com/pioho07/items/d50af79e9e7adb0dabc1)
- [OSによって処理を変えるPlaybookディレクトリの作成 - Qiita](https://qiita.com/nl0_blu/items/bb8cb9ac44fac01ea141)

