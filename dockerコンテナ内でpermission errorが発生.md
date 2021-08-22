---
title: dockerコンテナ内でpermission errorが発生
tags: Docker HTTP Flask CentOS Python
author: yumenomatayume
slide: false
---
# dockerコンテナ内でpermission errorが発生

### 動作環境(VM)
- CentOS Linux release 7.6.1810 (Core)
- Docker version 1.13.1, build 7f2769b/1.13.1
- docker-compose version 1.18.0, build 8dd22a9

### コンテナ内の環境

- Python 3.6.9 (FROM python:3.6)
- Flask 1.1.1
- Werkzeug 0.15.5

## 事象
`docker-compose up`を実行したところ、/root配下のファイルを操作した中でPermission deniedとなってしまいました

```
$ docker-compose up
Creating network "default" with the default driver
Creating container ... done
Attaching to container
container | python: can't open file '/root/app/run.py': [Errno 13] Permission denied
container exited with code 2
```

## 解析
dockerの再起動と、centosの再起動をしてみるが、どちらも解決せず、、
とりあえずググってみて、関連した記事を参考に解析してみる

### 1.SELinuxの無効化
https://qiita.com/koutwiring/items/10c3a67c1b58cf7d02c7
    
下記のファイルを編集して、SELinuxをオフにする

```/usr/lib/systemd/system/docker.service
~ snip ~
# 下記の１行を追記
OPTIONS=--selinux-disabled
~ snip ~
```

その後dockerを再起動して、`docker-compose up`を実行しましたが解決せず、、
    
```再起動コマンド
$ sudo systemctl restart docker
```

上記では解決できませんでした。


### 2.DockerでTCPポート80の使用をやめる
https://qiita.com/KEINOS/items/fba92ca0fb7662da0a00

flaskでコード(app.runの部分)を下記のように編集してポート5000を使用する

```python/run.py
~ snip ~

if __name__ == '__main__':
  app.run(host='0.0.0.0',port=5000,threaded=True)
```

これも解決せず、、

### 3. rootユーザでコマンドを実行する
Dockerfileの内容に下記を追記して、rootユーザでコマンド実行するようにしました

```Dockerfile
~ snip ~
USER root ←これを追記
CMD ["python", "/root/app/run.py"]
```

これも解決せず、、

### 4. dockerの最新版をインストール

dockerのversionを最新版にしてみました。

```

$ docker-compose up
Creating network "default" with the default driver
Creating container ... done
Attaching to container
container |  * Serving Flask app "run" (lazy loading)
container |  * Environment: production
container |    WARNING: This is a development server. Do not use it in a production deployment.
container |    Use a production WSGI server instead.
container |  * Debug mode: off
container |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
```
解決しました！！

## 原因
dockerのversionが古かったのでエラーを吐いていたようです
下記にバージョンアップすることで解決しました

```
$ docker --version
Docker version 19.03.1, build 74b1e89
```

---
1. [Dockerコンテナ内でのpermission denied](https://qiita.com/koutwiring/items/10c3a67c1b58cf7d02c7)
2. [Python3 + Web 80 番ポートで PermissionError: [Errno 13] Permission denied on Docker](https://qiita.com/KEINOS/items/fba92ca0fb7662da0a00)

