---
layout: post
title: "Effective Go+αで書かれてるネーミング規約"
date: 2014-05-03 15:44:43 +0900
comments: true
tags: golang
---
[Effective Go](http://golang.org/doc/effective_go.html#names)と[Go language community wiki](https://code.google.com/p/go-wiki/wiki/CodeReviewComments)を
必要になったので読んでみたが、いろいろ長ったらしい。なのでまとめた。

## パッケージ名
`bytes`みたく、小さくまとめる。そしてすべて小文字で。名前衝突は考えなくていい。起こることがレアだし、最悪、Goでは違う名前でのimportができる。たとえば`mp3.MP3File`というのではなく、`mp3.File`　でよい。どうせパッケージ名付きで呼び出すのだから。

## アクセサ (Getter, Setter)名
Getter / Setterをつくるのは良い。しかし、Getterの先頭にGetはいらない。
```
obj.Field() // getter
obj.SetField(); // setter
```

## インターフェース名
`Reader`, `Writer`のように、`-er`で終えろ。

## 変数名 
アンダースコアはいれない。`MixedCap`, `mixedCap`というようなスタイルで。
短い名前推奨。`lineCount`より、`c`。メソッドのレシーバーなんかは１,2文字でよい。あと、省略した場合はすべて大文字で。`URI`と略したら、`SetUri`ではなく`SetURI`と書くように。


## 参考
[Effective go (Naming)](http://golang.org/doc/effective_go.html#names)

[Go language community wiki](https://code.google.com/p/go-wiki/wiki/CodeReviewComments)