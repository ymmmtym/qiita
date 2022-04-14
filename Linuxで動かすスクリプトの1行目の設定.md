---
title: Linuxで動かすスクリプトの1行目の設定
tags: Linux ShellScript shell Python
author: yumenomatayume
slide: false
---
Linux環境である言語のスクリプトを実行する場合、
**エラーが出る可能性を低くするために**に、シバン(スクリプトの1行目)には以下のように記載した方がいいです。

```bash
#!/usr/bin/env (language)
```

具体的には、

- `#!/usr/bin/env ruby`
- `#!/usr/bin/env python`

といった風に使います。

## そもそもenvコマンドとは

envコマンドとは、環境変数の値を一時的に設定 or 削除してコマンドを実行することができるコマンドです。

例えば、dateコマンドで比較してみると以下のようになります。

```console
# 通常時(LANG=ja_JP.UTF-8)のコマンド

$ date
2020年 12月 8日 火曜日 08時54分06秒 JST


# LANG=Cに設定したコマンド

$ env LANG=C date
Tue Dec  8 08:53:42 JST 2020


# bashの場合は、頭のenvは省略できる

$ LANG=C date
Tue Dec  8 08:53:47 JST 2020


# 環境変数LANGを一時的に設定せずにdateを実行

$ env -u LANG date
Tue Dec  8 08:59:18 JST 2020


# 全ての環境変数を一時的に設定せずにdateを実行

$ env - date
Tue Dec  8 08:59:21 JST 2020
```

また、引数なしで`env`と実行すると環境変数一覧が表示されます。
これは `printenv` コマンドと同様の出力結果になります。

## スクリプトへの活用

シバンとは、Linux環境でスクリプトの1行目に記述する特殊な文字列のことであり、
そのスクリプトを実行するインタープリタを示します。

シバンを `#!/usr/bin/env (language)` としてenvコマンドを使用することで、
**PATH環境変数の通っている場所**から言語のインタープリタが検索されます。

pythonスクリプトの場合を比較してみます。

- `#!/usr/bin/python` と記載した場合
  - pythonのインストールパスが `#!/usr/local/bin/python` だった場合にエラーとなる
  - virtualenvなどの仮想環境で使用されるpythonパスではないため、仮想環境にてスクリプトを実行できない
- `#!/usr/bin/env python`と記載した場合
  - PATH環境変数にpythonコマンドが通っていれば使用できる
      - 汎用性・システム間の移植性が高い
  - virtualenvなどの仮想環境上でも使用できる(仮想環境起動時にPATHがうわがかれ)

後者の方がメリットが大きいことは明らかです。

## 最後に

シェルスクリプトを使い場合には、おまじないのように`#!/bin/bash`と記載すれば問題なさそうですが、
他のプログラミング言語を使用するときには、 `#!/usr/bin/env command` を記載した方が良さそうです。

## Reference

- [Pythonプログラムの#!/usr/bin/env pythonの意味を現役エンジニアが解説【初心者向け】 | TechAcademyマガジン](https://techacademy.jp/magazine/22254)
- [bash スクリプトの先頭によく書く記述のおさらい | Money Forward Engineers' Blog](https://moneyforward.com/engineers_blog/2015/05/21/bash-script-tips/)
- [/usr/bin/envを使って移植性の高いスクリプトを書く方法 | マイナビニュース](https://news.mynavi.jp/article/20180620-650003/)

