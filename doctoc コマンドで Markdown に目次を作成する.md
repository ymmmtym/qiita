---
title: doctoc コマンドで Markdown に目次を作成する
tags: Markdown Node.js
author: yumenomatayume
slide: false
---
doctocとは、markdownファイルの目次を自動的に生成するCLIです。
Node.jsで記載されています。

https://github.com/thlorenz/doctoc

## インストール

npmでインストールする場合

```bash
npm install -g doctoc
```

## 使い方

### Sample

```doctoc.md
# doctoc test

## test1

### test1-1

### test1-2

#### test1-2-1

## test2

### test2-1

```

上記の`doctoc.md`に以下のコマンドで目次を作成します。

```bash
$ doctoc doctoc.md

DocToccing single file "doctoc.md" for github.com.

==================

"doctoc.md" will be updated

Everything is OK.
```

`Everything is OK.`と出力されると、ファイルに目次が追記されています。

```markdown:doctoc.md
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [doctoc test](#doctoc-test)
  - [test1](#test1)
    - [test1-1](#test1-1)
    - [test1-2](#test1-2)
      - [test1-2-1](#test1-2-1)
  - [test2](#test2)
    - [test2-1](#test2-1)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# doctoc test

## test1

### test1-1

### test1-2

#### test1-2-1

## test2

### test2-1
```

`<!-- START doctoc generated TOC please keep comment here to allow auto update -->`から
`<!-- END doctoc generated TOC please keep comment here to allow auto update -->`までが
doctocによって自動的に追加された目次になります。

2行目に警告が記載されてますが、**doctocで追記された部分は編集しないように**してください。

markdownの内容を更新して、doctocを再実行したときにうまく作成されない恐れがあります。

### オプションについて

よく使うのは以下のオプションです。

| オプション                      | 内容                                                  |
| ---------------------------- | ----------------------------------------------------- |
| `[--notitle, --title title]` | `**Table of Contents**`の行の有無、もしくは名前の指定 |
| `[--maxlevel level]`         | 追記する目次の深さ                                    |
| `[-s, --stdout]`              | 作成時のコマンド出力に目次を表示する                          |

使用するサイトの markdown 形式に合わせたオプションを使用することができます。

```txt
--bitbucket bitbucket.org
--nodejs    nodejs.org
--github    github.com
--gitlab    gitlab.com
--ghost     ghost.org
```

### 特定ファイルをSkipしたい場合

公式サイトによると markdown ファイルの先頭に`<!-- DOCTOC SKIP -->`と記載します。

```markdown:skip.md
<!-- DOCTOC SKIP -->
# doctoc test

## test1

### test1-1

### test1-2

#### test1-2-1

## test2

### test2-1

```

次に以下のコマンドを実行します。`ack` コマンドをインストールする必要があります。

```bash
ack -L 'DOCTOC SKIP' | xargs doctoc
```

単純に、`DOCTOC SKIP` という文字列が入っていないファイルに対して目次を作成しているだけの動作になります。

