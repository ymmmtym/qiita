---
title: gitpodでjupyternotebookの開発環境を作る
tags: GitHub Gitpod Python Jupyter
author: yumenomatayume
slide: false
---
# はじめに
gitpodとは、githubのリポジトリをworkspaceとして読み込むクラウドIDEです.
dockerでVScodeのようなIDEを動かしているようです.

公式ドキュメント
https://www.gitpod.io/docs/

## gitpodの特徴
- 月100時間まで**無料**
- ブラウザとGitHubアカウントがあれば利用できる
- 開くディレクトリのtreeを指定できる
- VScodeライクで使える

## 構築する開発環境
今回は以下のような機械学習環境を構築していきます.

1. **python3.7**のdocker imageを元に
2. pipで**機械学習関連のライブラリ**をインストールして
3. **jupyternotebook**で[kaggleのtitanic](https://www.kaggle.com/c/titanic)をやってみる

gitpodで使用するリポジトリ
1. https://github.com/ymmmtym/ml/tree/test
2. https://github.com/ymmmtym/ml/tree/test_docker

(2つ使う理由は後述)

# gitpodを使ってみよう
使用するgithubのリポジトリを開き、URLの頭に`gitpod.io/#`をつける　←これだけ！

ちなみに、chrome拡張機能をインストールすれば、1クリックで開けます
![gitpod_chrome_extension.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/251749/1ed52ff2-0ba5-11af-204a-5de7d57510f2.png)

## 今回使用する2つのリポジトリについての違い
結論からいうと、test treeでは環境構築に失敗したためtest_docker treeを使用しました。

### test tree
**デフォルト**で設定されるgitpodのdocker環境のリポジトリ
この環境でvirtualenvで作成したpython仮想環境を読み込もうとしたが、ダメでした.
(python仮想環境をコンテナにCOPYしても、コンテナ内では参照されないみたいです)

```
gitpod /workspace/ml $ cat /etc/os-release 
NAME="Ubuntu"
VERSION="19.04 (Disco Dingo)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 19.04"
VERSION_ID="19.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=disco
UBUNTU_CODENAME=disco
gitpod /workspace/ml $ python -V
Python 2.7.15
gitpod /workspace/ml $ source .myvenv/bin/activate
(.myvenv) gitpod /workspace/ml $ python -V
Python 2.7.15
```
3系になっておらず、pipのライブラリもインストールされていない

### test_docker tree
test treeに以下の２ファイルを追加しました.

- .gitpod.Dockerfile
- .gitpod.yml 

```..gitpod.Dockerfile
FROM python:3.7

USER root

COPY ["requirements.txt", "/requirements.txt"]
RUN pwd
RUN apt-get -y update && \
    pip install --upgrade pip && \
    pip install --upgrade setuptools && \
    pip install -r requirements.txt
```
(コンテナbuild時のデフォルトpathは`/`なので、そこに`requirements.txt`をcopyする)


```..gitpod.yml
image:
  file: .gitpod.Dockerfile

ports:
- port: 8080
  onOpen: open-preview
- port: 8888
  onOpen: open-browser
```
(jupyternotebookが使えるように、port8888を使用)

2つのファイルを追加することにより、gitpodの環境を設定できるようになります.

## test_dockerリポジトリでgitpodを使う
それでは、URLの頭に`gitpod.io/#`をつけてみてください.
初回時の注意点として、
- 認証が必要になるので許可する
- 内部的にcontainerをbuildしているので、処理が多いと時間がかかる
![gitpod_pull_image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/251749/cfde9262-519f-6af0-8a0e-a5132f667d71.png)
↓ 少し待つ
![gitpod_ide.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/251749/787ea27d-6095-c077-f86b-3161e161d936.png)


無事にVScode(みたいなやつ)が起動されれば完了です.

それでは、terminalから`jupyter notebook --ip=*`コマンドを打ってみましょう.

```
gitpod /workspace/ml $ jupyter notebook --ip=*
[W 07:34:17.085 NotebookApp] WARNING: The notebook server is listening on all IP addresses and not using encryption. This is not recommended.
[I 07:34:17.088 NotebookApp] Serving notebooks from local directory: /workspace/python_env
[I 07:34:17.088 NotebookApp] Jupyter Notebook 6.4.0 is running at:
[I 07:34:17.088 NotebookApp] https://f6dd9c78-9281-4225-bec5-a2c05a810209.ws-ap0.gitpod.io:8888/?token=e4acea7f5f530d16bf660f914e13d8308fc2cc33ac5fffc1
[I 07:34:17.088 NotebookApp]  or http://127.0.0.1:8888/?token=eb377d6f673c4f5ec0fe4ca5b994f5956be12be6dbb8c6a4
[I 07:34:17.088 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
[C 07:34:17.093 NotebookApp] 
    
    To access the notebook, open this file in a browser:
        file:///home/gitpod/.local/share/jupyter/runtime/nbserver-288-open.html
    Or copy and paste one of these URLs:
        http://f6dd9c78-9281-4225-bec5-a2c05a810209.ws-ap0.gitpod.io:8888/?token=eb377d6f673c4f5ec0fe4ca5b994f5956be12be6dbb8c6a4
     or http://127.0.0.1:8888/?token=eb377d6f673c4f5ec0fe4ca5b994f5956be12be6dbb8c6a4
```
ポップアップがブロックされていなければ、別タブでjupyter notebookが開きます.

もしくは、「Open Browser」をクリック
もしくは https://8888-f6dd9c78-9281-4225-bec5-a2c05a810209.ws-ap0.gitpod.io/login にアクセス

![gitpod_jupyter_login.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/251749/0f00a352-3060-1c06-6561-2f8209598d1e.png)

ログイン画面が表示されるので、tokenを入力します.

今回は下記のようにterminalに表示されているので、

```
http://localhost:8888/?token=eb377d6f673c4f5ec0fe4ca5b994f5956be12be6dbb8c6a4
```

こちらを入力する.
`eb377d6f673c4f5ec0fe4ca5b994f5956be12be6dbb8c6a4`

![gitpod_jupyter_top.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/251749/8031ec70-b585-578e-92dc-57ad763777b2.png)

無事にjupyter notebookが開けました！以上です.

## 追記

コメント頂きまして、以下を修正しました。

### jupyter起動コマンドを`jupyter notebook --ip=*`に変更

`--ip=*`オプションを付けない場合、jupyter notebook上でファイルを新規作成・修正ができず、以下のエラーが返ってくることがわかりました。

```
[W 07:31:25.900 NotebookApp] Blocking Cross Origin API request for /api/contents.  Origin: https://8888-amethyst-vicuna-c6nx02vq.ws-us11.gitpod.io, Host: localhost:8888
[W 07:31:25.900 NotebookApp] Not Found
[W 07:31:25.901 NotebookApp] 404 POST /api/contents (::1) 2.050000ms referer=https://8888-amethyst-vicuna-c6nx02vq.ws-us11.gitpod.io/tree?
```

デフォルトでは、アクセスできるIPアドレスはlocalhostのみで、CORS設定ではOriginが異なる場合にブロックされるようです。
今回は、アクセス元(ブラウザ)とアクセス先(workspace)のIPアドレスは同じですが、Originが異なるため発生しました。

`--ip=*`オプションをつけることで、上記を解決することが出来ました。

## Reference
jupyternotebookを簡単に開く
https://github.com/jins-tkomoda/dash-and-jupyter-notebook-with-gitpod

python仮想環境のディレクトリを移動した結果
https://teratail.com/questions/99419

