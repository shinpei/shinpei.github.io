---
layout: post
title: "jdbからbreakpointをいれる"
date: 2014-10-07 20:28:32 +0900
comments: false
categories: java, note
---

リモートでJDB使うことは稀なのだが、たまにしかしないので忘れる為、メモ。
基本的には処理を一時的に止めて、問題を再現させたいときなどに使う。

## Remote Debugger用のポートをオプションで指定して起動する
通常のプログラムの起動に、以下のオプションを追加する。
やっていることは、デバッグオプションで起動し、コネクト用のポートを指定している。

```
-Xdebug -Xrunjdwp:transport=dt_socket,address=12347,server=y,suspend=n
```

## JDBから接続する
```
jdb -connect com.sun.jdi.SocketAttach:hostname=localhost,port=12347
```

## ブレークポイントの挿入

+ `stop in SomeClass.someMethod` : メソッドへ挿入
+ `stop in SomeClass.someMethod(int)` : オーバーロード時はシグネチャごと
+ `stop in SomeClass.<init>` :  コンストラクタへ

以上。
