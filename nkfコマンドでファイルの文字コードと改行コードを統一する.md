---
title: nkfコマンドでファイルの文字コードと改行コードを統一する
tags: Linux nkf 文字コード 改行コード
author: yumenomatayume
slide: false
---
nkfは「Network Kanji Filter」の略であり、文字コードと改行コードを変換するコマンドです。
異なるOS(WindowsやLinux)でファイルを共有するときの問題を解決します。

今回はLinux上でnkfコマンドを利用して、文字コードと改行コード以下に統一します。

- 文字コード: UTF-8(BOMなし)
- 改行コード: LF(Unix系)

## nkfコマンドのインストール

以下のコマンドでnkfをインストールできます。

```bash
# CentOS7
sudo yum install -y nkf

# CentOS8
sudo dnf install -y --enablerepo=PowerTools nkf

# ubuntu
sudo apt install -y nkf
```

## ファイル変換

以下のテキストファイルを変換します。

```test.txt
test
```

あえて、文字コードはUTF-8(BOMあり)で改行コードはCRLF(Windows系)にしてあります。

`cat -e test.txt`で確認すると、BOMが入っていることと改行コードがCRLFになっていることを確認できます。

```bash
$ cat -e test.txt
M-oM-;M-?test^M$
```

`M-oM-;M-?`はBOMであり、`^M$`は改行コードのCRLFです。(`^M`がCR、`$`がLF)

### 文字コードの変換

まずは文字コードをUTF-8(BOMなし)に変換します。`-w`オプションで変換ができます。

```bash
$ nkf -w test.txt
test^M$
```

### 改行コードの変換

次に改行コードをLFに変換します。`-d`オプションで変換できます。

```bash
$ nkf -d test.txt
M-oM-;M-?test$
```

## まとめ

もちろんオプションを併用することもできるので、一括で変換する場合は以下になります。

`--overwrite`オプションを指定すると、指定ファイルが上書きされます。
オプションがない(デフォルト)だと標準出力に結果が出力されます。

```bash
$ nkf -wd --overwrite test.txt
$ cat -e test.txt
test$
```

## Reference

- [UTF-8のhttpd.confでエラー - think-t の晴耕雨読](https://think-t.hatenablog.com/entry/20100713/1279034152)
- [【 nkf 】コマンド――文字コードと改行コードを変換する：Linux基本コマンドTips（51） - ＠IT](https://www.atmarkit.co.jp/ait/articles/1609/29/news016.html)

