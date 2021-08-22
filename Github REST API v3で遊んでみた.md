---
title: Github REST API v3で遊んでみた
tags: GitHub Bash api curl
author: yumenomatayume
slide: false
---
# GithubAPIとは

Githubでも開発者のためのAPIが利用できる。
公式サイト：https://developer.github.com/v3/

# GithubAPIの使い方

HTTPリクエストを送信すると、json形式でレスポンスが帰ってくる。
以下でuserのリポジトリ一覧を取得することができる

```
curl -u ${USER_NAME}:${PWD} -ks https://api.github.com/users/${USER_NAME}/repos
```

## GithubAPIで少し遊んでみた

以下のコードを書いて少し遊んでみました。

```git_clone_repos.sh
#! /bin/bash

json=$(curl -u ymmmtym: -ks "https://api.github.com/users/ymmmtym/repos")
count=$(($(echo $json | jq '. | length') - 1))

echo "Follow repos are found"
echo $json | jq .[].name | nl
echo -n "clone repository number: "
read i
number=$((i - 1))
git clone $(echo $json | jq -r .[$number].ssh_url)
echo "Done!"

exit 0
```
[github](https://github.com/ymmmtym/ymmmtym-ansible/blob/master/dev_utils/git_clone_repos.sh)

全てのリポジトリ名を表示し、番号を入力したリポジトリをクローンする。
※sshでクローンするため、公開鍵を設定している場合のみ使用できる。

これで、web上でcloneするディレクトリのurlを調べる手間が省けた。
他にも色々遊べそう。

