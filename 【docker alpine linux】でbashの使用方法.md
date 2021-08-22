---
title: 【docker alpine linux】でbashの使用方法
tags: Docker Bash ash alpine
author: yumenomatayume
slide: false
---
# docker alpine linuxでbashを使用するために
## 結論
以下のコマンドを叩くとbashが使用できるようになります。

```
apk add bash
```

## alpine linuxのshell
alipine linuxでは、デフォルトのshellに`ash`が使用されています。
`bash`を使う場合は、事前に`apk add bash`を叩いておく必要があります。

```
$ docker run -it alpine:latest apk add bash && bash -c "echo test"
fetch http://dl-cdn.alpinelinux.org/alpine/v3.11/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.11/community/x86_64/APKINDEX.tar.gz
(1/5) Installing ncurses-terminfo-base (6.1_p20191130-r0)
(2/5) Installing ncurses-terminfo (6.1_p20191130-r0)
(3/5) Installing ncurses-libs (6.1_p20191130-r0)
(4/5) Installing readline (8.0.1-r0)
(5/5) Installing bash (5.0.11-r1)
Executing bash-5.0.11-r1.post-install
Executing busybox-1.31.1-r9.trigger
OK: 15 MiB in 19 packages
test
```

P.S: `sh`は既にインストール済みなので`#!/bin/sh`で記載されたスクリプトなどは動かせます。

