---
layout: post
title: " Vagrant内のJVMに対してJconsoleとか使う、JMX over SSH"
date: 2015-03-28 15:08:43 +0900
comments: false
categories: java
---

Javaのパフォーマンス解析には、[Jconsole](http://docs.oracle.com/javase/7/docs/technotes/guides/management/jconsole.html)とか、[VisualVM](https://visualvm.java.net/ja/gettingstarted.html) などを使う。

最初の小さなジレンマは、これらの解析ツールがGUIであることだ。私が使うテスト環境Vagrant内部にはGUIをインストールしてないため、Vagrant外部から解析したいわけだが、VMの外から中のポートはたたけない。ポートフォワーディングでいいかとおもいきや、単純にフォワードしてもダメだった。なぜかはよくわからない。

というわけで、今日は簡単な方法を発見したのでシェアだ。

## SOCKS プロトコル

SOCKSというプロトコルがあるそうだ。クソなプロトコルのことではない。Socketsからきている。SOCKSは、あるポートにバインドした通信路へのソケットを提供してくれるらしい。これがTCP/IP全般を肩代わりできるらしいので、用途は広い。この通信路でSSHを経由してJMXを通す。`ssh`にはこのSOCKSを使うことができるオプション`-D`がついている。これと、バックグラウンド実行の`-f`, リモートコマンド実行しない`-N`をつけて、

```
ssh -fND $(PORT) vagrant@myenv
```
これで`$(PORT)`番のポートにSOCKSができる。

## Jconsoleから使う

```
jconsole -J-DsocksProxyHost=localhost -J-DsocksProxyPort=$(PORT) service:jmx:rmi:///jndi/rmi://localhost:$(JMX_PORT)/jmxrmi
```

これでいけた。

## Visual VMから使う

```
jvisualvm -J-Dnetbeans.system_socks_proxy=localhost:$(JMX_PORT) -J-Djava.net.useSystemProxies=true
```

こっちはまだ試していないが、これでいけるらしい。

## まとめ
今回の話は、vagrant内のJVMに限らない。Firewallで囲まれてる環境の外からJMXツールを使う場合は汎用的に使えるはずだ。
ただし、CUIから使うことに苦がない、RMIの呼び出しなどは、[jmxterm](http://wiki.cyclopsgroup.org/jmxterm/)がおすすめである。
グラフをGUIでみるなど外部からモニタリングしたい、かつ、プロダクションみたく監視ソフトウェアが充実している環境ではない場合に、使える小技である。


##参考

* ["ssh -Dとtsock"](http://www.kmc.gr.jp/advent-calendar/ssh/2013/12/14/tsocks.html), 京大マイコンクラブ(KMC)

* ["JConsole over ssh local port forwarding"](http://stackoverflow.com/questions/15093376/jconsole-over-ssh-local-port-forwarding), stackoverflow,

* ["Tunnel VisualVM (JMX) over SSH"](https://bowerstudios.com/node/731), BowerStudios.com,
