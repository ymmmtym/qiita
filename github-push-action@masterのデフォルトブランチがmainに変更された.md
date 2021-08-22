---
title: github-push-action@masterのデフォルトブランチがmainに変更された
tags: Git GitHub GitHubActions yml
author: yumenomatayume
slide: false
---
## ad-m/github-push-action@masterとは

[ad-m/github-push-action: GitHub actions to push back to repository eg. updated code](https://github.com/ad-m/github-push-action)<br>

- リモートリポジトリに変更点をPushをするGithub Actionsである。
- GitHub Actionsで何かの処理をしたときリポジトリに変更が加わった場合、自動でPushさせることができる。

以前までPushするbranchのデフォルトが**master**だったが、**main**に変更されたのでymlファイルも以下のように変更した。<br>

```yaml:action.yml
on:
  push:
    branches:
      - main # 修正

#snip...

    - name: Push changes
      uses: ad-m/github-push-action@master
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        branch: main # 追記(デフォルトがmasterからmainになった)
```

- デフォルトのbranchは**main**であるが、今後変更される可能性があるので明示的に書いておく
- まだmaster branchを使用している人は`branch: master`と修正すれば良い

