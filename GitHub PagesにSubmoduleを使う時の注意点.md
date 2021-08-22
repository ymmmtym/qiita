---
title: GitHub PagesにSubmoduleを使う時の注意点
tags: Git GitHub github-pages
author: yumenomatayume
slide: false
---
以下の2つの制約を守る必要があります。

- httpsで使用されていること
- submoduleがパブリックリポジトリであること

## GitHub Pagesのデプロイに失敗してしまった時

制約を守れなかった場合は、以下のような警告文が表示されます。(`discogs-python-api`というsubmoduleを、SSHで使用してしまいました)

![github-pages-build-error.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/251749/fe07fb8a-ec57-3f33-c6e0-34c0d9a128ba.png)

この状態になると、GitHub Pagesのデプロイに失敗しているので、サイトも404になってしまいます。

また、警告にあるリンクを辿るとSubmoduleを削除する手順が記載されています。
[GitHub Pages サイトの Jekyll ビルドエラーに関するトラブルシューティング - GitHub Docs](https://docs.github.com/ja/pages/setting-up-a-github-pages-site-with-jekyll/troubleshooting-jekyll-build-errors-for-github-pages-sites#invalid-submodule)

```bash
git submodule deinit PATH-TO-SUBMODULE
git rm PATH-TO-SUBMODULE
git commit -m "Remove submodule"
rm -rf .git/modules/PATH-TO-SUBMODULE
```

GitHub Pagesをデプロイするリポジトリにsubmoduleを使う場合、
制約を守って使うか、守れない場合はsubmoduleを使わないで管理する必要があります。

## Reference

https://docs.github.com/ja/pages/setting-up-a-github-pages-site-with-jekyll/troubleshooting-jekyll-build-errors-for-github-pages-sites

