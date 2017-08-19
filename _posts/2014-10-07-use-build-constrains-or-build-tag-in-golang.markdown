---
layout: post
title: "go build -tagsを使ってRelease/Debugを切り替える"
date: 2014-10-07 21:06:27 +0900
comments: false
tags: golang
---

Goでウェブアプリの[クライアント](https://github.com/shinpei/comstock)を書いている。
ローカルでテストする場合は、サーバもローカルにたてるので、アクセスする先は`localhost`になる。
リリースするときは正しいウェブアプリのURLを指定しなければならない。
この際、切り替えはナイーブにコメントアウトで行ってきた。
```
const (
  // APIServer = "http://www.mywebapp.com"
  APIServer = "http://localhost:8080"
  )
```
当然、デバッグ用を有効にしたままリリースしてしまったり、その逆だったりと、混乱があった。
このような、デバッグ・リリースで機能を切り替えたい場合はBuild constrainsを使えば良いらしい。

## Build Constrainsとは？

従来の機能が充実しているGoに抜かりはない。Build constrainsとは必要に応じてビルドするファイルを切り替える、[Goの機能](http://golang.org/pkg/go/build/#pkg-overview)だ。
公式ドキュメントに紹介されている例は、アーキテクチャ毎にビルドするソースを切り替える方法だ。

ビルドするファイルを切り替える方法は以下の３つ。

### 1. コメントで切り替える

```
// +build linux
```

このコメント行をファイルの先頭に書いておくと、`GOOS=linux`の場合のみビルドされる。`i386`などのGOARCHもタグとして使うことが可能だ。
ちなみに、否定は以下のように記述する。

```
// +build !linux
```

### 2. ファイル名で切り替える
ファイル名でもBuild constrainsが行える。以下のファイルは、`GOOS=windows, GOARCH=amd64`の環境でしかビルドされない。
ファイル名からビルド対象がわかるため、開発者にも優しい。ここについては、後に[補足](#a1)を入れた。

```
source_windows_amd64.go
```

### 3. `build -tags`で切り替える
これは[Cheney氏のブログ](http://dave.cheney.net/2014/09/28/using-build-to-switch-between-debug-and-release)をきっかけで知ったのだが、
`+build`の後に付加されるシンボルはタグと呼ばれ、`go build -tags tagA`のようにオプションで指定できる（らしい）。
Build constrainsがBuild tagともよばれる所以か？これを使えば以下のようにコメントされたファイルは、先のコマンドからのみビルドされることになる。

```
// +build tagA
```

## `build -tags`を使ってRelease/Debugを切り替える

さて、上に挙げた3番目の手法を使えば、debug/releaseのビルドを切り替えられる。没頭にあげたように、テスト時は`localhost`を。
リリース時は正しいアドレスを使いたい場合、以下のように２つのファイルを記述すればよい。

release.go
```
// +build !debug

package main

const APIServer = "http://localhost:8080"
```

debug.go
```
// +build debug

package main

const APIServer = "http://www.mywebapp.com"
```

**注意すべきは、1行目のコメントに続く3行目の間に空行が必要な点だ。**

そして、debug時は以下のようにビルドを行う。
```
go build -tags debug
```


## まとめ

+ Build constrainsを使えばファイル単位でビルドを制御することができる
+ `go build -tags tagName`で好きなタグを指定できる
+ タグは`// +build`で指定する。コメントの後の空行を忘れずに。

### <a name="a1">補足１</a>

ちなみに、手法1, 2ともに適用する必要はなく、どちらかで良い。どちらか選ぶ場合は、ファイル名で切り替えを選べば、grepなどで簡単に
ビルド対象のファイルを選別できるのでおすすめだそうだ。

### 補足２
デフォルトで用意されているbcは、`GOOS`や`GOARCH`以外に、`ignore`がある。
```
// +build ignore
```
と書かれたファイルはビルドされない。
個人的にもっとわかりやすくて、よく使うのは、以下のように`_`でファイル名を始めることだ。

```
_source.go
```
エラーが多いファイルなどをとりあえず無視したいときに良い。


## 参考
[go/build](http://golang.org/pkg/go/build/#pkg-overview) from Go Document

[Using //+build to swtich between debug and release builds](http://dave.cheney.net/2014/09/28/using-build-to-switch-between-debug-and-release) by Dave Cheney

[How to properly use build tags?](http://stackoverflow.com/questions/15214459/how-to-properly-use-build-tags) from Stackoverflow
