---
title: "後からgitignoreを追加する"
emoji: "😊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["git"]
published: true
---
# リポジトリにプッシュした後から.gitignoreファイルを追加する。
## 1.gitignoreファイルを作成する。
管理の対象から外したいファイル、もしくはフォルダを記入する。

## 2 削除したいフォルダ・ファイルをリポジトリから削除する
git rm --cached -r 削除したいディレクトリ
git rm --cached ファイル名

## 3 コミット
git add .
git commit -m 'commit message'

## 参考サイト
https://qiita.com/yutosa3/items/25ab031c8061e8c9a4c4