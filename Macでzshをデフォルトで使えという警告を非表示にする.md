---
title: Macでzshをデフォルトで使えという警告を非表示にする
tags: Mac Bash Zsh Linux fish
author: yumenomatayume
slide: false
---
<!-- Macでzshをデフォルトで使えという警告を非表示にする -->

# 以前まで表示されなかった警告が発生

macOS Catalinaにアップデートしてから、bashを起動すると以下のメッセージが表示されるようになった。
(デフォルトでfishを使用しています)

```
ymmmtym@localhost ~> bash

The default interactive shell is now zsh.
To update your account to use zsh, please run `chsh -s /bin/zsh`.
For more details, please visit https://support.apple.com/kb/HT208050.

ymmmtym@localhost:~$
```

## 解決方法

結論からいうと、~/.bashrcに以下を追記することで解決できた。

```shell:~/.bashrc
export BASH_SILENCE_DEPRECATION_WARNING=1
```

## 警告メッセージについて

警告メッセージで素直に表示されたサイト(<https://support.apple.com/ja-jp/HT208050>)にアクセスしてみる。

> macOS Catalina から、Mac は zsh をデフォルトのログインシェルおよびインタラクティブシェルとして使うようになります。それ以前のバージョンの macOS でも、zsh をデフォルトにすることができます。

zsh以外を使用すると、警告メッセージが表示される仕様になっているらしい。

## おまけ

Macの場合だけ、bashのバイナリファイルが少し違うみたいです。

### Ubuntu16.04の場合

```
ymmmtym@ubuntu16.04:~$ strings /bin/bash |grep BASH_SILENCE_DEPRECATION_WARNING
ymmmtym@ubuntu16.04:~$
```

### Mac(macOS Catalina)の場合

```
ymmmtym@localhost:~$ strings /bin/bash |grep BASH_SILENCE_DEPRECATION_WARNING
BASH_SILENCE_DEPRECATION_WARNING
ymmmtym@localhost:~$
```

