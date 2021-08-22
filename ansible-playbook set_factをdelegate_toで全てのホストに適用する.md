---
title: ansible-playbook set_factをdelegate_toで全てのホストに適用する
tags: Ansible ansible-playbook
author: yumenomatayume
slide: false
---
[ansible.builtin.set_fact – Set host variable(s) and fact(s). — Ansible Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/set_fact_module.html)

ansible-playbookには`set_fact`モジュールというものがあり、新しい変数を設定する事ができます。

モジュールの概要は以下になります。

> ・このモジュールでは、新しい変数を設定できます。
・変数は、セットアップモジュールによって検出されたファクトと同じように、ホストごとに設定されます。
・これらの変数は、ansible-playbookの実行中に後続の再生で使用できます。
・ファクトキャッシュを使用して実行間で変数を保存するには、cacheableをyesに設定します。
・set_factで作成された変数は、キャッシュされているかどうかによって優先順位が異なります。
・標準のAnsible変数の優先順位ルールに従って、他の多くのタイプの変数の優先順位が高いため、この値はオーバーライドされる可能性があります。
・このモジュールは、Windowsターゲットでもサポートされています。

ansible実行中に取得した出力を、後から変数にしたい時に使う場面が多いかと思います。

概要にもある通り、これらの変数は**ホスト毎**に設定されます。

特定のホストに設定された`set_fact`を別のホストに使用したい場合は、`delegate_to`を指定して、他のホストに引き渡す必要があります。

## delegate_toとは

[Controlling where tasks run: delegation and local actions — Ansible Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_delegation.html)

delegated_toを使うと、指定したタスクを別ホストで実行できます。本来、Playbookでは`hosts`に一致するものに対して実行されますが、これを使用する事で`hosts`以外や特定のホストのみtaskが実行できます。

## Playbookの例

ubuntuグループに属した、ubuntu01とubuntu02に`set_fact`を試してみます。

- ホストがubuntu01 →  `ubuntu01: defined`のみ
- ホストがubuntu02 → `ubuntu02: defined`のみ

と設定します。

### Inventory

```yaml:inventory.yml
all:
  children:
    ubuntu:
      hosts:
        ubuntu01:
        ubuntu02:
```

### Playbook(`delegate_to`を使わない場合)

`set_fact`で定義された変数は`defined`、定義されていない変数は`undefined`となるようなPlaybookを作成します。

```yaml:set_fact.yml
---
- hosts: ubuntu
  tasks:
    - name: set_fact to ubuntu01
      set_fact:
        ubuntu01: defined
      when: inventory_hostname == "ubuntu01"
    - name: set_fact to ubuntu02
      set_fact:
        ubuntu02: defined
      when: inventory_hostname == "ubuntu02"
    - name: debug set_fact
      debug:
        msg: |
          "ubuntu01 is {{ ubuntu01 | default('undefined') }}"
          "ubuntu02 is {{ ubuntu02 | default('undefined') }}"
```

実行結果

```bash
$ ansible-playbook set_fact.yml -i inventory.yml -v

PLAY [ubuntu] *******************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************
ok: [ubuntu02]
ok: [ubuntu01]

TASK [set_fact to ubuntu01] *****************************************************************************************************************
ok: [ubuntu01] => {"ansible_facts": {"ubuntu01": "defined"}, "changed": false}
skipping: [ubuntu02] => {"changed": false, "skip_reason": "Conditional result was False"}

TASK [set_fact to ubuntu02] *****************************************************************************************************************
skipping: [ubuntu01] => {"changed": false, "skip_reason": "Conditional result was False"}
ok: [ubuntu02] => {"ansible_facts": {"ubuntu02": "defined"}, "changed": false}

TASK [debug set_fact] **************************************************************************************************************
ok: [ubuntu01] => {
    "msg": "\"ubuntu01 is defined\"\n\"ubuntu02 is undefined\"\n"
}
ok: [ubuntu02] => {
    "msg": "\"ubuntu01 is undefined\"\n\"ubuntu02 is defined\"\n"
}

PLAY RECAP **********************************************************************************************************************************
ubuntu01                   : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
ubuntu02                   : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
```

ubuntu01では以下である事がわかりました。(ubuntu02はその逆です)

- `ubuntu01: defined`
- `ubuntu02: undefined`

### Playbook(`delegate_to`を使う場合)

`delegate_to`を使用して、全てのホストに変数を受け渡してみます。

```yaml::set_fact.yml
---
- hosts: ubuntu
  tasks:
    - name: set_fact to ubuntu01
      set_fact:
        ubuntu01: defined
      delegate_to: "{{ item }}" # 変数を渡すホスト
      delegate_facts: true # fact変数の更新する場合trueにする必要がある
      with_items: "{{ groups['all'] }}" # 全ホストに受け渡す
      when: inventory_hostname == "ubuntu01"
    - name: set_fact to ubuntu02
      set_fact:
        ubuntu02: defined
      delegate_to: "{{ item }}" # 
      delegate_facts: true
      with_items: "{{ groups['all'] }}"
      when: inventory_hostname == "ubuntu02"
    - name: debug set_fact
      debug:
        msg: |
          "ubuntu01 is {{ ubuntu01 | default('undefined') }}"
          "ubuntu02 is {{ ubuntu02 | default('undefined') }}"
```

実行結果

```bash
ansible-playbook set_fact.yml -i inventory.yml -v
Using /Users/yukihisa/.ghq/github.com/ymmmtym/terraform-cloud-oci/ansible.cfg as config file

PLAY [ubuntu] *******************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************
ok: [ubuntu02]
ok: [ubuntu01]

TASK [set_fact to ubuntu01] *****************************************************************************************************************
ok: [ubuntu01 -> ubuntu01] => (item=ubuntu01) => {"ansible_facts": {"ubuntu01": "defined"}, "ansible_loop_var": "item", "changed": false, "item": "ubuntu01"}
ok: [ubuntu01 -> ubuntu02] => (item=ubuntu02) => {"ansible_facts": {"ubuntu01": "defined"}, "ansible_loop_var": "item", "changed": false, "item": "ubuntu02"}
skipping: [ubuntu02] => (item=ubuntu01)  => {"ansible_loop_var": "item", "changed": false, "item": "ubuntu01", "skip_reason": "Conditional result was False"}
skipping: [ubuntu02] => (item=ubuntu02)  => {"ansible_loop_var": "item", "changed": false, "item": "ubuntu02", "skip_reason": "Conditional result was False"}

TASK [set_fact to ubuntu02] *****************************************************************************************************************
skipping: [ubuntu01] => (item=ubuntu01)  => {"ansible_loop_var": "item", "changed": false, "item": "ubuntu01", "skip_reason": "Conditional result was False"}
skipping: [ubuntu01] => (item=ubuntu02)  => {"ansible_loop_var": "item", "changed": false, "item": "ubuntu02", "skip_reason": "Conditional result was False"}
ok: [ubuntu02 -> ubuntu01] => (item=ubuntu01) => {"ansible_facts": {"ubuntu02": "defined"}, "ansible_loop_var": "item", "changed": false, "item": "ubuntu01"}
ok: [ubuntu02 -> ubuntu02] => (item=ubuntu02) => {"ansible_facts": {"ubuntu02": "defined"}, "ansible_loop_var": "item", "changed": false, "item": "ubuntu02"}

TASK [debug set_fact] ***********************************************************************************************************************
ok: [ubuntu01] => {
    "msg": "\"ubuntu01 is defined\"\n\"ubuntu02 is defined\"\n"
}
ok: [ubuntu02] => {
    "msg": "\"ubuntu01 is defined\"\n\"ubuntu02 is defined\"\n"
}

PLAY RECAP **********************************************************************************************************************************
ubuntu01                   : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
ubuntu02                   : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
```

`set_fact`のtaskから分かる通り、ubuntu01,02とも両方の変数が`defined`になっている事が分かりました。

今回はgroup['all']で全てのホストに引き渡しましたが、単一ホストをIP(ホスト名)で指定できます。
また`set_fact`だけではなく、もちろん他のモジュールにも使用する事ができます。

## Reference

- [＜Ansible＞「delegate_to」ディレクティブの使い方 - Qiita](https://qiita.com/fumiya-konno/items/d2d6b67296e5ee94340b)
- [[Ansible] playをまたいだ変数保持をdelegate_toとfact変数で行う (Ansibleでkubeadmのworkerノード追加) - zaki work log](https://zaki-hmkc.hatenablog.com/entry/2020/04/15/083708)

