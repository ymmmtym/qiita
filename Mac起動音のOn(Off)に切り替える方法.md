---
title: Mac起動音のOn(Off)に切り替える方法
tags: Mac 初心者
author: yumenomatayume
slide: false
---
電源ONや再起動したときに鳴る「ジャーーン」というやつです。
過去にも備忘録として書きましたが、新しいコマンドが見つかったので再び備忘録として書きます。

過去の記事: [【備忘録】Macの起動音を消す方法 - Qiita](https://qiita.com/ymmmtym/items/f58ab2195f7dd67e1e48)

使用しているもの

- macOS Catalina
- 2016年より古いモデル(デフォルトで起動音がなる仕様)

# Macの起動音をOn/Offに切り替える方法

ターミナルで以下のコマンドを実行してください。

**Onにするコマンド**

```bash
sudo nvram StartupMute=%00
```

**Offにするコマンド**

```bash
sudo nvram StartupMute=%01
```

**現在のOn/Offを確認するコマンド**

```bash
nvram -p | grep StartupMute
```

Offの場合の出力例

```console
$ nvram -p | grep StartupMute
StartupMute	    %01
```

私は起動音をOffにする派ですが、
macOSをCatalinaにアップグレードしたタイミングでOnになっていたようです。

## Reference

[Bring back your Mac's startup chime with this simple terminal command - 9to5Mac](https://9to5mac.com/2020/02/21/bring-back-mac-startup-chime-command/)

