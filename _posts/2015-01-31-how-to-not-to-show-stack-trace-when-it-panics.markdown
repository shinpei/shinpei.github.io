---
layout: post
title: "recoverでGoのテストのスタックトレースを省略する"
date: 2015-01-31 00:50:03 +0900
comments: false
categories: golang
---

## 概要
Goでテストを書く際に`t.Errorf`などで問題を表示することは多々ある。しかし、テストがいくつかあった際にダンプをズラズラと表示されたときは、一概でどのテストが失敗したのか追うのがめんどくさいと感じることもあるだろう。そういう時は、`recover`と`defer`をうまく使えばよいらしい。
具体的にはテストに以下ようなコードを足せば、うまく省略できる。
```
defer func() {
  if r := recover(); r != nil {
    t.Error("Too lazy to show stack trace. This test has failed. Fix it.")
  }
}()
```
## 問題

たとえば、以下のようなテストを考える。例では`Stack`という型を定義したパッケージでも作ったこととしよう。
```
package stack

import (
  "testing"
  )

  func TestNewStack(t *testing.T) {
    var s Stack
    if s.Len() != 0 {
      t.Errorf("The length of stack shold be 0, not %d", s.Len())
    }
  }
```
このテストは、ヌルポでpanicを起こす(`t.Errorf`は内部で`panic`を呼ぶ)。なぜなら、この例で使ってる`Stack`は`interface`として定義しており、
実際の変数宣言は`stack`というstructで宣言しなければならないからだ。（実際のコードは[Gist](https://gist.github.com/shinpei/602faaff5af1f3f725d7)に置いた。)

```
➜  stack  go test
--- FAIL: TestNewStack (0.00s)
panic: runtime error: invalid memory address or nil pointer dereference [recovered]
panic: runtime error: invalid memory address or nil pointer dereference
[signal 0xb code=0x1 addr=0x20 pc=0x5b97f]

goroutine 5 [running]:
testing.func·006()
/usr/local/Cellar/go/1.4.1/libexec/src/testing/testing.go:441 +0x181
_/Users/xxx/D/prac/go/stack.TestNewStack(0x2082ac000)
/Users/xxx/D/prac/go/stack/main_test.go:9 +0x2f
testing.tRunner(0x2082ac000, 0x201f10)
/usr/local/Cellar/go/1.4.1/libexec/src/testing/testing.go:447 +0xbf
created by testing.RunTests
/usr/local/Cellar/go/1.4.1/libexec/src/testing/testing.go:555 +0xa8b

goroutine 1 [chan receive]:
testing.RunTests(0x1906c0, 0x201f10, 0x1, 0x1, 0xb80634fbdb741301)
/usr/local/Cellar/go/1.4.1/libexec/src/testing/testing.go:556 +0xad6
testing.(*M).Run(0x2082800a0, 0x20e380)
/usr/local/Cellar/go/1.4.1/libexec/src/testing/testing.go:485 +0x6c
main.main()
_/Users/xx/D/prac/go/stack/_test/_testmain.go:52 +0x1d5
exit status 2
FAIL	_/Users/xxx/D/prac/go/stack	0.079s
```

##　`recover`の動作と`defer`の連携

`recover`はこれまで使ったことなかったが、`panic`によるスタックのトレースを中断して、
`panic`が呼ばれたところに戻る関数らしい。
`recover()`は、事前に`panic`が呼ばれていなければ`nil`を返し、呼ばれていなければそうじゃないものを返す。

つまり、`if r:=recover(); r != nil` で`panic`判定を行い、そこでの動作を終えれば**`panic`が呼ばれた場所に戻る**。

`defer`は、その関数のスタックを抜けるときに呼び出される関数を登録できる。
これらを合わせると、`panic`によるトレースダンプを中断することができる。

## 結果

以下が実際のコードだ。
```
func TestNewStack(t *testing.T) {
  defer func() {
    if r := recover(); r != nil {
      t.Error("New allocation for stack has failed")
    }
    }()
    var s Stack
    if s.Len() != 0 {
      t.Errorf("The length of stack shold be 0, not %d", s.Len())
    }
  }
```

すると、こうなる。
```
➜  stack  go test
--- FAIL: TestNewStack (0.00s)
main_test.go:10: New allocation for stack has failed
FAIL
exit status 1
FAIL	_/Users/xxx/D/prac/go/stack	0.072s
```

## まとめ

`recover`は使い方よくわからなかったが、テストだけでなく、`panic`によるスタックトレースを止めて、戻ることができる。ってことを書いておきたかったんだなぁ、きっと。


## 参考
[Understanding defer, panic, and recover](http://www.goinggo.net/2013/06/understanding-defer-panic-and-recover.html)
