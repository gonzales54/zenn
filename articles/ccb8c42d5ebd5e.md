---
title: ""
emoji: "😊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['git', 'github']
published: true
---

# gitでerror Object file .git/~ is emptyというエラーが起きた。

## 解決策
1 cp -a .git .git-old 
2 git fsck --full
3 find . -type f -empty -delete -print
4 git fsck --full
5 tail -n 2 .git/logs/refs/heads/main //二個目のハッシュ値をコピーする。
6 git show ハッシュ値 //一番上にあるcommitの行にあるハッシュ値をコピーする。
7 git update-ref HEAD ハッシュ値
8 git fsck --full
9 rm .git/index 
10 git reset
11 git fsck --full
12 git status
13 git commit -a
14 git add .
15 git commit -m '~'
16 git fetch
17 git merge
18 git pull origin main
19 git add .
20 git commit -m '~'
21 git push -u origin main