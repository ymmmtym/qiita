---
title: DockerhubのイメージをGithub Container Registryに移行する
tags: Docker DockerHub GithubContainerRegistry
author: yumenomatayume
slide: false
---
DockerhubからGithub Container Registry にイメージを移行する際の自分用メモです。

### 背景
[https://github.blog/2020-09-01-introducing-github-container-registry/](https://github.blog/2020-09-01-introducing-github-container-registry/)
9/1 に公開された GitHub Container Registry(以下、ghcr) を使用してみる。
現在(2020/09/03)ではパブリックベータ版なので、変更になる可能性がある。

[Docker Hubに保存したコンテナイメージ、無料プランでは6カ月間使われないと削除へ － Publickey](https://www.publickey1.jp/blog/20/docker_hub6.html)
また、Dockerhubの無料プランでは使用されていないコンテナが削除されてしまうように仕様が変更された。

つまり、
「Docker Hub の需要が増えて制約が多くなったので、無料プランなら Github Container Registry を使っていこう。」

### 前提条件

- Githubに登録していること
    - パーソナルアクセストークンを作成していること
        - packagesにread,write,delete権限があること

ghcrには以下のコマンドでログインする

```console
dockerer login ghcr.io -u $OWNER #Githubのアカウント名
Password: <パスワードはパーソナルアクセストークンを入力> 
Login Succeeded
```

以下のコマンドで、Docker Hub から移行する

```bash
docker pull $OWNER/$IMAGE_NAME:$VERSION
docker tag $OWNER/$IMAGE_NAME:$VERSION ghcr.io/$OWNER/$IMAGE_NAME:$VERSION
docker push ghcr.io/$OWNER/IMAGE_NAME:$VERSION
```

ghcrにpushしたcontaner image をPublicに設定して`docker pull`できることを確認する

```bash
docker pull ghcr.io/$OWNER/$IMAGE_NAME:$VERSION
```

### Reference
[GitHub Container Registry 入門](https://www.kaizenprogrammer.com/entry/2020/09/03/060236)<br>
[GitHub Container Registryのパブリックベータをリリース - GitHubブログ](https://github.blog/jp/2020-09-07-introducing-github-container-registry/)

