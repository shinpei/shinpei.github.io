---
layout: post
title: "Mac OSXで、IntelliJをJDK1.7で動かす"
date: 2014-11-22 17:32:34 +0900
comments: false
categories: OSX
---

Goのコードがでかくなってきたので、リファクタの手間がかかる。昔から、リファクタはIDEと決めているので、
IntelliJでやってみた。

現時点の最新版は、IntelliJ14。この前出たばっかり。

しかし、まさかのJDK1.6を要求してくる。そういう時は、`Info.plist`を書き換えましょう。

```
cd /Application/IntelliJ\ IDEA\ 14\ CE.app/
emacs Info.plist
```

で、`JVMVersion`というところをみつけて、`1.6*`を`1.7*`に書き換えたらオッケー。
```
      <key>JVMVersion</key>
      <string>1.7*</string>
```

## 参考

[How Do I Run idea IntelliJ on Mac OSX with JDK 17, stackoverflow](http://stackoverflow.com/questions/13019199/how-do-i-run-idea-intellij-on-mac-os-x-with-jdk-7)
