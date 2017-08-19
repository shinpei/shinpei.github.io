---
layout: post
title: "Go言語でのバインディング。FFIであるCGOを使う。"
date: 2014-04-29 13:06:42 +0900
comments: true
categories: golang
---

個人的に、Go言語からのバインディングに興味をもち、ちょうど必要だった[mecab](http://mecab.googlecode.com/svn/trunk/mecab/doc/index.html)のバインドをつくっている。（[shinpei/go-mecab](http://www.github.com/shinpei/go-mecab)）Go言語には、バインディングのためのCGO(シーゴー）というツールがついている。そこでgo-mecabの経験から、Goでのバインディングメモを書いてみた。先に、まとめておくと、CGOには以下のよい点、よろしくない点があった。

1. Cの関数を直接呼べる。このためラッパー関数は（ほぼ）必要ないのは楽で**良い**。
2. CのソースをGoの中に書ける。必要なラッパーはここで追加できるのは楽で**良い**。
3. typedefされたシンボルを区別する。このため、型変換関数を定義する必要があり、**よろしくない**。
4. コンパイルに必要なLDFLAGSなどをテキストとして読む。pkg-configなどが使えず**よろしくない**。

3,4について、特に4について、workaroundをお持ちの方は、教えて頂きたい。

## Cの関数を呼ぶ

たとえば、libmに含まれる、sin関数は以下のように呼び出せる。

```go
package main

/*
#cgo LDFLAGS:-lm
#include <math.h>
*/
import "C"

func main () {
        println(C.sin(3.14));
}
```

ここでの厄介な注意点は以下だ。なぜそうなるのかはよくわかってない。質問してみようかしら。

- import "C"は、*/* * */の後に空行を置かずに書くこと **。
- 他パッケージとまとめて、`import ("C" "fmt")`のようにしてもNG

## 型変換 （Goの世界とCの世界)

上の例では、意識していないが、バインディングでは必ず互いの世界の型にあわせる必要がある。先ほどの`C.sin()`の型は、`func (_Ctype_double) _Ctype_double`となる。`_Ctype_double`というのが、C言語の型である。一方でGo型対応する型はfloat64だ。`_Ctype_double`に関しては、プリミティブを代入可能である。このため、先の例は問題なくコンパイルできた。
```
var d _Ctype_double = 3.14; // doubleはプリミティブ代入可能
```
ちなみに、`var d C.double = 3.14`と書いても良い。

では、型変換が問題になる例をあげよう。import "math"を追加して、printlnを以下のように替える。
```
    println(math.Abs(C.sin(3.14)));
```
こうすると、以下のエラーがでる。
```
➜  sin2  go build
./sin.go:14: cannot use _Cfunc_sin(3.14) (type C.double) as type float64 in function argument
```
goコンパイラはC.doubleとfloat64が交換可能であることをしらない。このため、エラーがでている。そのため、こう書き換えないといけない。
```
    println(math.Abs(float64(C.sin(3.14))));
```

この型変換について最も厄介なのが、typeofされたCの構造体だった。Cでは以下の２つは同じ型だが、Goでは２つは異なる型として扱われる。
```c
struct mecab_t;
typeof struct mecab_t mecab_t; //これ以降はmecab_tと書かれる
```

この２つの型については変換関数をCGOを用いて定義してやり、それを呼び出すことで解決した。他にいい方法があれば知りたい。

## 構造体要素へのアクセス
Go言語はポインタを持つ。このためreferenceとdefeferenceを区別する必要があるが、実際はなんとも透過的に扱える。これは、Goが参照か値かを区別してくれて、アクセス演算子の動作を変えてくれるから、とどっかで読んだ。(Cでは、値なら. (ドット)、参照なら -> (アロー）がアクセス演算子として用いられる）

シンプルな例をあげる。Goのコードでは参照か値かを区別せずに使えることがわかる。

```
package main

import "fmt";

type Hoge struct {
    Name string;
    Age int;
}

func main () {
    var valA Hoge = Hoge {"taro", 16};
    fmt.Printf("%s\n",valA.Name);
    var refA *Hoge = &valA;
    fmt.Printf("%s\n", (refA.Name));
}
```
このコンパイルは無事通り、以下が実行結果だ。
```
➜  reference  ./reference
taro
taro
```
この機能のおかげで、構造体のアクセスは、最上位の構造体の参照さえとってこれれば、アクセス可能となる。このあたりは[実コード](https://github.com/shinpei/go-mecab/blob/master/node.go)をサンプルにあげる。ptrというのが、C構造体へのポインタであり、Getterあたりを見れば、C構造体の要素にアクセスしてるのがわかる。

## 終わりに
もっと複雑なライブラリをバインドする場合は他にも必要な情報があるだろう。関数ポインタのマップとか、`__attribute__((pack))`付きの構造体とか。それらはバインドの機会があれば書きたい。

ちなみにサンプルにあげたgo-mecabはfree(defer)も未実装である。言うまでもないが、freeなしの現状では、富豪以外、使ってはいけない。


## 参考
[cgo 公式ドキュメント](http://golang.org/cmd/cgo/)

[公式レポにあるgmpのバインディング](https://code.google.com/p/go/source/browse/misc/cgo/gmp/gmp.go?r=release)

[Good or recommended way to translate C struct to Go struct](http://stackoverflow.com/questions/23004474/good-or-recommended-way-to-translate-c-struct-to-go-struct)
