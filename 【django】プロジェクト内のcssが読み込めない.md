---
title: 【django】プロジェクト内のcssが読み込めない
tags: Python Django virtualenv
author: yumenomatayume
slide: false
---
# 事象
ローカル開発環境にてDjangoの管理サイトにアクセスしましたが、
以下のように、管理サイトでcss(静的ファイル)が読み込めませんでした。

![django_admin_no_css_login.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/251749/c3a8077d-f881-2cee-bb13-19f5d271360b.png)
![django_admin_no_css.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/251749/cb740567-914f-a30f-7601-f9e55e1c773b.png)

---
## 前提条件

```
Python 3.7.4
Django 2.2.6
virtualenv 16.1.0
```

# 解析
## HTTPログ

```
$ python manage.py runserver
Performing system checks...

System check identified no issues (0 silenced).
November 02, 2019 - 16:45:33
Django version 2.2.6, using settings 'studysite.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
[02/Nov/2019 16:24:55] "GET /admin/ HTTP/1.1" 200 3080
[02/Nov/2019 16:24:55] "GET /static/admin/css/responsive.css HTTP/1.1" 404 77
[02/Nov/2019 16:24:55] "GET /static/admin/css/dashboard.css HTTP/1.1" 404 77
[02/Nov/2019 16:24:55] "GET /static/admin/css/base.css HTTP/1.1" 404 77
```

どうやらcssが読み込めてないようです。
原因を探るべく、`settings.py`を確認してみた。

```settings.py
# snip

DEBUG = False
ALLOWED_HOSTS = ['localhost','0.0.0.0']

# snip
```

上記の`DEBUG = False`が怪しかったのでTrueにすると、cssが読み込めるようになりました。
DEBUG = Falseは本番環境が想定されており、静的ファイルはnginxなどのwebサーバから読み込む仕様になっているようです。

# 対応策
cssが読み込まれるためには、以下の2つのうち**どちらか**の対応が必要です。

## 1. settings.pyでDEBUG = Trueにする
Trueにするとプロジェクト内のcssも読み込まれます。

## 2. `python manage.py runserver --insecure`コマンドを使う
--insecureオプションを付けると、プロジェクト内のcssが読み込まれます。

# おまけ

```settings.py
DEBUG = False
ALLOWED_HOSTS = ['*']
```

この設定ではcssは適用されませんでした。
ALLOWED_HOSTSを任意にしただけではcssは読み込めないようです。

