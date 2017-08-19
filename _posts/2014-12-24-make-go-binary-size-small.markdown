---
layout: post
title: "Goプログラムのサイズを小さくする"
date: 2014-12-24 22:04:24 +0900
comments: false
categories: golang
---

Goで作られるプログラムサイズはそこそこ大きい。配布する場合はやはり小さい方が良い。
そんなときは、リンカの設定を利用するといいようだ。

```
go build -ldflags '-s -w'
```

Linuxならバイナリサイズが小さくなるはずだ。拙作、[comstock](https://comstock.herokuapp.com)では8MiBから5.2MiBに縮んだ。

## 何をやっているか？
`-ldflags`は、`gcc`などを使ったことある方はよくご存知だろうが、`ld`への引数である。
`ld`とは`gcc`が使うリンカであり、Goもリンカを持っている。Goのリンカは、例えば、`go tool 6l`（番号はアーキテクチャ依存）から呼び出せる。伝統的にリンカへのフラグは
`LDFLAGS`であるので、goでは小文字で指定するようになってる。

`go build`はリンカへの引数を`-ldflags`の後で文字列として渡すことができる。

指定できるリンカフラグは様々で、[標準ドキュメント](https://golang.org/cmd/ld/)よりも`go 1.4`では数が増えているように見える。とりあえず、上で使ったのは以下の２つだ。

```
-s    disable symbol table
-w    disable DWARF generation
```

シンボルテーブルは、シンボルと、シンボルの位置（アドレス）が記述された表である。
```
➜  nm `which comstock-cli` | grep Log | grep Printf
000000000007d8c0 t log.(*Logger).Printf
```
例えば、上は`Logger.Printf()`の位置を示している。

このシンボルテーブルは外部から参照することがない限り、
不必要だ。
よって、`-s`を渡して削る。
ちなみに、ビルドしてるのがライブラリだったり、ビルド後に
デバッガを使う場合は、シンボルテーブルは必要になるので、削ってはいけない。

`DWARF`というのは、Linuxの実行可能形式である[`ELF`](http://ja.wikipedia.org/wiki/Executable_and_Linkable_Format)に付属する
デバッグ情報である。
こちらも配布用のプログラムには必要無い。よって`-w`で削る。
こちらはデバッグしない限り、ライブラリでも削って良い。

`darwin`では`ELF`ではなく[`Mach-O`](http://ja.wikipedia.org/wiki/Mach-O)という実行可能形式を用いるため、`DWARF`はそもそも生成されない。
シンボルテーブルは削れると思うが、私の環境では削れてなかった。
とりあえずLinuxで縮んだので良しとする。

## 参考
[Command ld (Go document)](https://golang.org/cmd/ld/)

[Golang application auto build versioning](http://stackoverflow.com/questions/11354518/golang-application-auto-build-versioning) ※直接関係ないが、ldflag利用の一例。
