---
title: 【備忘録】Macの起動音を消す方法
tags: Mac NVRAM
author: yumenomatayume
slide: false
---
# Macの起動音について
[公式サイト](https://support.apple.com/ja-jp/HT202768)によると、
Macの起動音には意味を表しているようです。

## 環境を汚したくない
アプリケーションをインストールして起動音を消すことができますが、
環境を汚したくないと思い、CLIから設定しました。

## Macの環境音を設定
```
# 起動音を消す                                                                                                                        
sudo nvram SystemAudioVolume=%80
<パスワードを入力>

# 起動音をつける
sudo nvram -d SystemAudioVolume
<パスワードを入力>

# 起動音の音量を確認する
nvram -p | grep SystemAudioVolume

