---
title: 【初学者向け】MacでDockerを使うための最低限の事
tags: Docker Mac
author: yumenomatayume
slide: false
---
<!-- MacでDockerを使うための最低限の事 -->
# はじめに
今更ですがDockerの魅力を知ったので、
ローカル環境(Mac)にインストールしてみました。

その際に行った初期設定などを、備忘録を含めてまとめます。

ローカル環境でMacOSを使用している前提とします。
また、dockerをインストールした後の操作などは対象外です。

## 開発環境
- Macbook Pro (Core i5)
- MacOS


# DockerをMacで使う
macにアプリケーションをインストールするだけです。
(カップラーメンを作るより早く終わります。)

## Dockerのインストール
### 1.Docker for MacのDL & Install
https://hub.docker.com/editions/community/docker-ce-desktop-mac
上記のdockerhubより、**Get Docker**をクリックし、**Docker Desktop for Mac**をDLする。
(dockerhubにログインしないとDL出来ません。登録&ログインを済ませておいてください。)
![dockerhub_docker_for_mac.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/251749/2d7b9799-0c98-5012-90ec-ad7b464d3912.png)

DLした**docker.dmg**ファイルを開き、**Docker.app**を**Applications**へ追加する。
![install_docker_for_mac.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/251749/102f26eb-88fa-7c9d-5b46-d30609cdca2c.png)

アプリケーションからDockerを開くとメニューバーにdockerアイコンが追加される。
![docker_menu_icon.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/251749/fc6d421c-a574-d629-acce-f05c50ceba14.png)

アイコンをクリックし、緑色で**Docker Desktop is running**と表示されたら正常に動いています


## 2.Dockerのバージョン確認
MacのTerminalより下記のコマンドでバージョンを確認する。

```
$ docker --version
Docker version 18.09.2, build 6247962

$ docker-compose --version
docker-compose version 1.24.0, build 0aa59064

$ docker-machine --version
docker-machine version 0.16.1, build cce350d7
```

上記のような出力がされれば、dockerが正常作動しているのでOK！

# まとめ
これだけで、macでdockerを使用出来ます。
今回は本当に**最低限の事**のみを記事にしました。
docker,docker-composeの操作などは、別記事にて更新予定です。

