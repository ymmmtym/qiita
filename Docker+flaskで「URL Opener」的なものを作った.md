---
title: Docker+flaskで「URL Opener」的なものを作った
tags: Docker Python pip Flask jinja2
author: yumenomatayume
slide: false
---
# はじめに

知らない人向けに説明すると、「URL Opener」とは下記のようなサイトです

http://www.url-opener.com/

使い方は

1. textareaに開きたいURLを１行づつ入力する
2. Open allをクリックすると、入力された複数のURLが別タブで開かれる

この時、ポップアップブロックを使用していると全て開かれないことがあるので、許可してから使用してください

# Keywooとは
textareaに「URL」でなく「検索キーワード」を入力します
検索サイトはjson(後述)にあらかじめ記載するか、フォームより追加します

src: https://github.com/ymmmtym/keywoo
heroku: https://keywoo.herokuapp.com/

## Keywooの特徴

- urlでなく検索キーワードを入力するので、毎回URLを準備する必要がなくなる
- タブを開きっぱなしにする必要がなくなる
- flaskで実装したので、軽量に動作する(無料で使えるGCPでも快適に動作します)

#### flaskとは

- pythonのwebフレームワークであり、軽量であることが主な特徴
- 中小規模のサイトで利用され、Djangoに次ぐ2位の利用率

## 動作環境

```
$ docker --version
Docker version 18.09.2, build 6247962
$ docker-compose --version
docker-compose version 1.23.2, build 1110ad01

```

## ディレクトリ構成

```
.
├── Dockerfile
├── README.md
├── app
│   ├── data
│   │   └── sites.json
│   ├── run.py
│   ├── static
│   │   ├── favicon.ico
│   │   ├── layout.js
│   │   ├── main.js
│   │   └── stylesheet.css
│   └── templates
│       ├── index.html
│       ├── layout.html
│       └── result.html
├── docker-compose.yml
└── requirements.in
```

### 各ディレクトリ説明

## 各ファイル説明

flaskの動作に必要なライブラリをインストール

```requirements.in
flake8
scipy
matplotlib
scikit-learn
requests
beautifulsoup4
Flask
```

### Dockerfile

```Dockerfile
FROM python:3.6
MAINTAINER ymmmtym
USER root
WORKDIR /root
ENV HOSTNAME="keywoo-container" \
    PS1="[\u@\h \W]# "
ADD ["requirements.in", "/root/requirements.in"]
RUN apt-get -y update && \
    pip install --upgrade pip && \
    pip install --upgrade setuptools && \
    pip install pip-tools && \
    pip-compile /root/requirements.in && \
    pip-sync
ADD ["app", "/root/app"]
WORKDIR /root/app
EXPOSE 5000
CMD ["python", "/root/app/run.py"]
```

主にpipのインストールやreqiremets.inに記載されたpythonのライブラリの追加などを行なっています
(現在は、dockerhubにbuildしたimageをアップしたので、Dockerfile自体は使用しない)

### docker-compose.yml

`docker build`した後でも`docker restart`で設定が反映されるように、
appディレクトリ配下は永続的にマウントされるようvolumesに記載しています。
デフォルトでportは80を使用しています

```docker-compose.yml
version: '3'
services:
  keywoo:
    image: yumemo/keywoo
    container_name: keywoo-container
    hostname: keywoo-container
    tty: true
    volumes:
      - ./app:/root/app
    ports:
      - "5000:5000"
```

### app/run.py
```python:app/run.py
#!/usr/bin/python

from flask import Flask, render_template, request, jsonify
import json
    
app = Flask(__name__)
def get_toppage(str):
    list = str.split('/')
    return list[0] + '//' + list[2]
app.jinja_env.globals['get_toppage'] = get_toppage
app.config['JSON_AS_ASCII'] = False

with open("./data/sites.json", "r", encoding="utf-8") as sites_json:
    search_dic = json.load(sites_json)

@app.route('/', methods=["GET","POST"])
def index():
    if request.method == "POST":
        if request.form["radio"]:
            global search_dic
            if request.form["radio"] == "delete":
                del_sites = request.form.getlist("check")
                for site in del_sites:
                    del search_dic[site]
            if request.form["radio"] == "default":
                with open("./data/sites.json", "r", encoding="utf-8") as sites_json:
                    search_dic = json.load(sites_json)
            if request.form["radio"] == "reset":
                search_dic.clear()
            if request.form["radio"] == "add":
                if request.form["site_name"] and request.form["url"]:
                    search_dic.update({str(request.form["site_name"]):str(request.form["url"])})
    return render_template("index.html", search_dic = search_dic)

@app.route('/result', methods=["GET", "POST"])
def result():
    if request.form["search"]:
        search_text = str(request.form["search"])
        search_list = search_text.splitlines()
        return render_template("result.html", search_list = search_list, search_dic = search_dic)
    else:
        return render_template("index.html", search_dic = search_dic)

if __name__ == '__main__':
  app.run(host='0.0.0.0',port=5000,threaded=True)
```
flaskを起動するためのpythonファイルです

### app/data/json
data配下にはjsonファイルを格納しています。
検索サイトのデータを保持する目的で利用しています。
大量のデータを扱うわけではないので、DBではなくjsonファイルに記載することにしました

```json:sites.json
{
    "Google": "https://www.google.com/search?q=",
    "Weblio English": "https://ejje.weblio.jp/content/",
    "Amazon": "https://www.amazon.co.jp/s?k=",
    "Rakuten": "https://search.rakuten.co.jp/search/mall/",
    "Yahoo Auctions": "https://auctions.yahoo.co.jp/search/search?p=",
    "Yahoo Auctions(record)": "https://auctions.yahoo.co.jp/search/search?auccat=22260&p=",
    "Spotify": "https://open.spotify.com/search/results/",
    "Discogs": "https://www.discogs.com/ja/search/?q="
}
```

### app/static
css,jsや画像などの静的ファイルを格納しています。
(デザインを良くするために作成しました。無くても問題なく動作はします)

### app/templates
htmlを格納しています。

## 操作方法

### server側

```shell
pwd
# /root/keywoo
docker-compose up -d
```

その後、下記にアクセスすると使用できます
http://localhost:5000/

### web側
![keywoo_top.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/251749/e0eead78-a700-d08d-896f-e41e355c4a4e.png)

1. searchの下のtextareaに検索したい複数キーワードを１行ずつ入力
2. submitをクリックするとsearch sitesにあるそれぞれ検索結果へのリンクを表示するページを返す

#### search sitesの設定
対象の項目にチェックをつけてapplyをクリックすると下記のような処理が行われます

- delete selected sites
search sitesのテーブル内でチェックの付いているサイトを検索対象から削除する

- load default sites from json
jsonファイルから検索サイトを読み込む

- reset all sites
全ての検索サイトを検索対象から削除する

- add site
Name:検索サイトの名称(任意)
URL:検索サイト(クエリ文字列までを記載する)

# 今後は
今後は以下のような機能をつけてみたいと思います

- レイアウトの改修(レスポンシブデザインとか）
- jsonをweb上で編集
- login機能をつけてuser毎の検索サイトを保持(現在はpythonの変数に、検索サイトを辞書型で保存しているだけ)

以上です。

