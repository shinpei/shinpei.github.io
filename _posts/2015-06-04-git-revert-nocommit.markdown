---
layout: post
title: "Gitで一個前のコミットから作業したいケースと方法"
date: 2015-06-04 22:30:48 +0900
comments: false
tags: git java
---
## 概要
`git revert --no-commit`が使えるね、というメモ。

## 動機

Javaを書いていると、以下のようなバージョニングを行う。（`pom.xml`などの記述）

```
version=1-SNAPSHOT
```

そもそも、`1.0.0-SNAPSHOT`じゃないのか、という話はここでは一旦置いておく。このバージョンをリリースしたら、こうなる。

```
version=1
```

これで`git tag version-1`などして、タグ付けしたら、バージョンをあげてコミットしたいだろう。

```
version=2-SNAPSHOT
```

この作業を手作業でやることになったが（自動リリーススクリプトがうごかないケースがあった）、記述箇所が多い為、`sed`で置換などする。
`1-SNAPSHOT`を`1`にするのは、簡単だった。しかし、`1`を`2-SNAPSHOT`にするのは至難である。
`1`はバージョン以外にも使われてる為である。
だったらコミット前の状態（`1-SNAPSHOT`から`1`にしたもの）に戻して、`1-SNAPSHOT`を`2-SNAPSHOT`に置換したい。

## git reset HEAD^ではなく？

`git reset HEAD^`を試してみる。`1-SNAPSHOT`に戻った。さて、これを`2-SNAPSHOT`にしよう。
置換できた。コミットしよう。。。
いや、だめだ。これだと分岐してしまう。
この状態をパッチにして、HEADに当てようか、、いや、それもだめだ。
HEADは`1`がbaseであるためパッチはあたらない。
commitの逆操作をcommitなしにパッチとして取り出して、それを書き換えたい。

## どうやるか

`git revert`は、あるコミットを打ち消すコミットをすると記憶していた。
具体的には
+ 逆操作を行うパッチを作成する
+ パッチをコミットする
という２段階の操作が行われる。これを1段階目だけで終えたい。そのためには、`git revert --no-commit`(もしくは`-n`)とすればいいそうだ。

```
git revert HEAD --no-commit
git reset HEAD　
（1-SNAPSHOTを2-SNAPSHOTへ置換)
git commit
```

かゆいところに手が届く。やはり良い。

##参考

* [git-revert Documentation](http://git-scm.com/docs/git-revert)

* [Git で コミットを無かったことにする方法 （git revert の使い方）](http://akiyoko.hatenablog.jp/entry/2014/08/21/220255)
