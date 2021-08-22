---
title: 【mac】diskutilコマンドでliveUSBを作成する
tags: Mac ubuntu16.04
author: yumenomatayume
slide: false
---
# はじめに
liveUSBを作成するフリーソフトは数多く存在するが、
mac環境を汚したくない人向けに下記の手順でliveUSBを作成できる

### 参考
mac環境が汚れてもいい人はこれを進める
フリーソフト「UNetbootin」
https://unetbootin.github.io/

## コマンドライン
ubuntu16.04のイメージディスクが入ったliveUSBを作成した例
 
```
# mountされているディスクの確認
diskutil list

# ディスクのフォーマット(MS-DOS:ファイルシステム,UNTITLED:ディスク名) 
diskutil eraseDisk MS-DOS UNTITLED /dev/disk3

# ディスクのアンマウント
diskutil unmountDisk /dev/disk3

# isoイメージの書き込み(if=<isoイメージ>,of=<ディスク>,bs=<ブロックサイズ>) *1
sudo dd if=~/Downloads/ubuntu-16.04.6-server-i386.iso of=/dev/disk3 bs=8056

# ディスクの取り出し
diskutil eject /dev/disk3
```

*1:書き込み終了時に下記のエラーぽい表示が出るが、きちんとliveUSBとして読み込めるので問題ない。
![write_iso_to_usb_finished.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/251749/cb0528fc-3b6c-b2bd-8e48-9ac22870c8b9.png)

