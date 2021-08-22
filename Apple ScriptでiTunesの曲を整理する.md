---
title: Apple ScriptでiTunesの曲を整理する
tags: Mac AppleScript iTunes
author: yumenomatayume
slide: false
---
# はじめに
*この記事はiTunesを使用している方向けの記事です

iTunesで曲を探していると、

- アルバムアーティストが書いていない
- コメントが書いてある

など、統一性のない曲がちらほら存在します。

以前はiTunes上で複数の曲を選択して編集していたのですが、
面倒になったのでスクリプトを書いてみました。

# Apple Scriptを書いてみる
### コードの目的
editというプレイリストに入ってるそれぞれの曲に対して、

- コメントを空にする
- アルバムアーティストが書いていなければ、アーティストと同一にする

### 実行方法
1.Macのアプリケション「スクリプトエディタ」から、新規書類でapple scriptを作成する
![script_editor.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/251749/e24377db-a1a5-e8f1-452a-3f6203c7f6db.png)

2.以下のコードを書いて、▶️をクリック

```artist2albumartist.scpt
tell application "iTunes"
	set tmp to AppleScript's text item delimiters
	set AppleScript's text item delimiters to {":"}
	repeat with aTrack in tracks of user playlist "edit"
		set comment of aTrack to ""
		if album artist of aTrack is "" and album of aTrack is not "" then
			set album artist of aTrack to text item -3 of (location of aTrack as string)
		end if
	end repeat
	set AppleScript's text item delimiters to tmp
end tell
```

3.出力結果が以下のようになればOK

```result
{:}
```

## Script解説
editプレイリストにある曲に対して繰り返し(repeat)実行する

```
repeat with aTrack in tracks of user playlist "edit"
  # 処理  ← #でコメントアウトできます
end repeat
```

コメントを空にして、
アルバム名が空でなければ、アーティスト名をアルバムアーティスト名に設定する

```
set comment of aTrack to ""
if album artist of aTrack is "" and album of aTrack is not "" then
    set album artist of aTrack to text item -3 of (location of aTrack as string)
end if
```
---

編集したい曲をプレイリストに入れて実行するだけなので、以前と比べて作業が楽になりました。

*注意:アルバム内で異なるアーティスト(.feat とか)がある場合でも、曲単位で設定されます


以上です。


