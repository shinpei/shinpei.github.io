---
layout: post
title: "Javaのテスト実行時間を62%削るvmvmを試してみた"
date: 2015-09-25 22:14:19 +0900
comments: true
tags: java ICSE test maven
---

## 概要
[vmvm](https://github.com/Programming-Systems-Lab/vmvm)てのを使うと、テスト実行時間が短くなることがあるらしいので、試してみた。
結果、私の試行では早くならなかったが、早くなる人もいるかと思われ、使い方をシェアしたい。

## vmvmて何？

論文漁ってたらたまたま見つけた、[ICSE '14のペーパ](http://jonbell.net/publications/vmvm)。すでに１年経過してる。タイトルは、"Unit Test Virtualization with VMVM"。Unit test virtualization??と思いつつも、読み進める。どうやらJavaのテスト実行時間を62%短くする、`vmvm`てのを作ったらしい。62%て。
なんとも驚異的な結果。さすがICSE。

## 使ってみる(maven編)

(antは[公式README](https://github.com/Programming-Systems-Lab/vmvm)を参照)

論文の中身はさておき、まずはチェックアウト。ビルドしてインストール。バイナリもあるのでそれにパスとおしても良い。
```
git clone https://github.com/Programming-Systems-Lab/vmvm.git
mvn install
```

`vmvm`は`maven`の[`surefire`](https://maven.apache.org/surefire/maven-surefire-plugin/)プラグインから使うコンポーネントとして提供されている。
`surefire`の`2.15`以上がいるので、そうでない場合はアップグレード。

dependencyまで含めた`surefire`部分のpom記述はこうなる。

```

    <dependencies>
        <dependency>
            <groupId>edu.columbia.cs.psl.vmvm</groupId>
            <artifactId>vmvm</artifactId>
            <version>1.0.0-SNAPSHOT</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>edu.columbia.cs.psl.vmvm</groupId>
            <artifactId>vmvm-ant-junit-formatter</artifactId>
            <version>1.0.0-SNAPSHOT</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.18.1</version>
                <configuration>
                    <properties>
                        <property>
                            <name>listener</name>
                            <value>edu.columbia.cs.psl.vmvm.MvnVMVMListener</value>
                        </property>
                    </properties>
                </configuration>
            </plugin>
        </plugins>
    </build>

```

最後にテストを実行する。
```
mvn clean test
```


## 結果

残念ながら、私がピックアップした小さなレポジトリではあまり変わらなかった。

## どんな場合に早くなりそうか
そもそも、`vmvm`が対象にするのは、大規模かつ、プロセスが分かれてる(isolatedな)テストセットのようだ。
プロセスが分かれてるというのは、おそらくテストクラスがわかれてるということだ。
これを同じプロセスで実行すると、リソースが再利用、節約できて、実行時間が劇的にはやくなるよ
（ざっくりすぎて怖い。詳しくはぜひ論文を）、というわけだ。
従って、同じ初期化処理を共有する、小さなテストクラスがいくつも存在する大規模ソフトウェアに対して
有用であると考えられる。私が試したプロジェクトは、テストクラスはたかだか20程度だろう。

## まとめ
`vmvm`は劇的な数字と一流学会論文成果という敷居が高そうなソフトウェアだが、気軽に使えたので記事を書いてみた。

個人的には、これまでで、慣習的に、同じ初期化処理があるものは、同じテストクラスに書いている場合もあった。
しかし、これはクラス内のコードが長くなり、かつ、読みにくく、エラーもひろいにくくなる。
vmvmがあれば、わざわざテストを同じところに書かずに、小分けにしても実行速度が担保できるのかも。


## 参考

* [Programming-Systems-Lab/vmvm](https://github.com/Programming-Systems-Lab/vmvm)

* [Jonathan Bell, "Unit Test Virtualization with VMVM", ICSE '14'](http://jonbell.net/publications/vmvm)

* [Maven Surefire Plugin](https://maven.apache.org/surefire/maven-surefire-plugin/examples/junit.html)
