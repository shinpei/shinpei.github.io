---
layout: post
title: "VPNとDockerを併用する"
date: 2014-11-11 18:22:09 +0900
comments: false
tags: docker
---

VPNが導入されている環境で、Dockerを使いたい、という場合。
VPNは製品によるが、Linuxであればtun/tapをつかって実現されている
ことも多いそうだ。とうわけでメモ。

## 問題
Dockerを使ってみよう。なんて素敵だ、こんなに簡単にLinuxが立ち上がるなんて。
あれ。`yum update`ができない。おかしいな、PROXYは設定してるのに。お、VPNを切ったら動くぞ。
いや、しかしVPNが使えないなら意味ないじゃないか。。。

という場合。

## 結論
私の環境では、tun/tapを利用したVPNは、クラスBのプライベートアドレスのすべてをtun0で
上書きしてた。ここはdocker0も使う。なので、docker0の設定を変えて、クラスCのアドレスを
バインドした。

## 1. Dockerの動作範囲を確認
Dockerを起動して、`netstat -r`を打ってみる。
```
Destination     Gateway         Genmask         Flags MSS Window irtt iface
192.168.0.0     *               255.255.255.0   U     0      0    0   eth0
169.0.0.0       *               255.255.0.0     U     0      0    0   eth0
172.17.0.0      *               255.255.0.0     U     0      0    0 docker0
default         192.168.0.1     0.0.0.0         UG    0      0    0   eth0
```

`docker0`に割り当てられるのはデフォルトでは、`172.17.0.0/255.255.0.0`のようだ。

## 2. tun0の動作範囲を確認

VPNに接続して`netstat -r`を打ってみる。

```
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
172.16.0.0      x.x.x.x     255.240.0.0     UG    1      0        0 tun0
```


関係がありそうなのは、`172.16.0.0/255.240.0.0`だ。

## 3. サブネットマスクから重複を確認
ぱっと見てわからない場合は、以下のようなサイトがある。

http://note.cman.jp/network/subnetmask.cgi

サブネットマスクを計算すると、被ってるかどうか確認できる。
実際、`172.16.0.0/255.240.0.0`は`172.16.0.0 ~ 172.31.255.255`を示してる。クラスB全部だ。

## 4. 被ってたら`--bip`で再設定
上では、`172.16.0.0`〜`172.16.255.255`が`tun0`に割り当てられてる。
ここは、Dockerのネットワークインタフェースである`docker0`が使う部分だ。
スーパーユーザで起動していれば起動順序によってはdockerで上書きされる
かもしれないが、dockerユーザを使ってる場合がほとんどだろう。
このため、docker0が使っていないアドレス空間をブリッジすればよい。

dockerの設定は`/etc/default/docker`にある。今回は以下のようにバインドする。

```
DOCKER_OPTIONS="--bip 192.168.51/24"
```
再起動は必要。


ちなみに、`docker1`などの別ブリッジを作ってもよい。ブリッジの作り方は、[How Docker networks a container](https://docs.docker.com/articles/networking/#how-docker-networks-a-container)
にも書いてあるため、割愛。

## 参考

[How Docker networks a container](https://docs.docker.com/articles/networking/#how-docker-networks-a-container)


[Customizing Docker0](https://docs.docker.com/articles/networking/#customizing-docker0)


[TUN/TAPがまともに使えるようになるまで](http://slashdot.jp/journal/429079/TUNTAP%E3%81%8C%E3%81%BE%E3%81%A8%E3%82%82%E3%81%AB%E4%BD%BF%E3%81%88%E3%82%8B%E3%82%88%E3%81%86%E3%81%AB%E3%81%AA%E3%82%8B%E3%81%BE%E3%81%A7)

[route ~ ルーティングテーブルの表示・設定を行う(@IT)](http://www.atmarkit.co.jp/fnetwork/netcom/route/route.html)
