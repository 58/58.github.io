---
title: "Scala & Play frameworkで一行掲示板を作ってみた"
date: 2024-12-29
categories: ["programming"]
tags: ["programming", "web", "scala", "play"]
---

# 前提
## Scala
Scala入門者なので、[公式のBOOK](https://docs.scala-lang.org/scala3/book/introduction.html)のA Taste of Scalaまでを読んで勉強した。
日本語がないのが厳しいが、Scala、特に3系の情報が少ないのでこれを読むしかない。
プログラミング言語公式の入門にしては、それなりに丁寧に書かれていて分かりやすいと思う。
個人的にPythonの公式の入門ドキュメントは初心者に読ませる気がないと思っている。
合計10時間ぐらいは読んで理解したように思う。

## Play
WebフレームワークとしてPlayを選択した。
選択した理由はChatGPTに聞いたらこれを教えてくれたから。
一応Gogleでも調べてみたが、信頼できるScalaプログラマーのWindymeltさんが[以下のように書いてくれていた](https://blog.3qe.us/entry/2023/03/03/231522)ので、まぁ採用するかという気持ちになった。
> 巨大フレームワークを許容するならPlay


ただ、この[Play frameworkの公式ドキュメント](https://www.playframework.com/documentation/3.0.x/Home)がちょっと雑で、後々苦労することにはなった。
いや、自分の知識・経験不足なのかもしれないが、なんとなくWebサイトの構造が体系的でなさすぎる気がする。
Hello, Worldは簡単だが、その先の具体的なものはexampleプログラムの配布で解説がない。
甘えと言われたらそうなのだが、最近のWebフレームワークのチュートリアルは簡単なシステムを解説付きで順々に作っていくことが多い気がするが、それがない。
また、個別のページでDB接続法など見てみても、プロジェクトファイルのどこにそれを追加するのかなどが分かりづらい。
これは、自分がScalaやsbtに詳しくないのが悪いとは思う。

## その他
ChatGPTに聞きながら色々やった。Scalaをやりたくて一行掲示板を作ろうと思ったのだが、正直Play frameworkとH2データベースを学んだだけになった気がする。

Windymeltさんの[Twitter](https://x.com/windymelt)や[ブログ](https://blog.3qe.us)は適宜チェックした。信頼できるScala日本語発信者なのでとても助かった。

# 開発内容
成果物は[Github](https://github.com/58/scaline-board)に挙げた。

## 環境
[build.sbt](build.sbt)を見てもらえば分かるが、以下のような構成で行った。
- Scala 3.3.4
- Play 7.0.1
- H2 database 2.2.224
- Slick 6.1.0

Dockerfile等を用意しているが、production環境は飾りになっていて、development環境しか使っていない。

## 苦労したところとか
### Docker環境
JavaでのAndroidアプリの開発経験はあったが、それ以外にJVM系言語に触れた経験が全くなく、コンテナに何を用意すべきか全然分からなかった。
そもそもJVM系は[30億のデバイスで走る](https://dic.nicovideo.jp/a/30%E5%84%84%E3%81%AE%E3%83%87%E3%83%90%E3%82%A4%E3%82%B9%E3%81%A7%E8%B5%B0%E3%82%8Bjava)ことが売りであり、コンテナ技術に頼るべきシステムなのかが判断付かなかった。
とは言え、JVM系動くようにするためにはコンパイルが必要で、そのコンテナが必要ではありそう。

ということで、ChatGPTに相談してDockerfileを書いたが、[hseeberger/scala-sbt](https://hub.docker.com/r/hseeberger/scala-sbt/)という最終更新が三年前のimageを提案されてしまった。
本来は丁寧にDockerfileを書くべきだと思うが、早くScalaとPlayに触れたかったので、とりあえずこれを採用して開発を進めることにした。

ScalaのDockerコンテナについては、みんなそれなりに苦労してそうで、これといったベストプラクティスがなさそうな印象を受けた。
公式が用意してほしいところではあるが、そういう余裕がないのだろうか。
正攻法でやるなら、debian辺りにCoursierをインストールして、`cs setup`とか叩くのが良いのかな？
この辺は次のScala開発の課題にしたいところ。

### Scala 3
Scala自体、日本語・英語問わずインターネットに情報が少ないのだが、Scala 3となるとさらに少なくなる。情報があっても2系であったり、3年以上前の記事であったりする。
そのためか、ChatGPTのレスポンスもあまり良くない。
Playが3系になっているか、Slickが3系対応しているのか、そもそもライブラリの指定が合っているのかなど手探りで躓きながらPlayのプロジェクトファイルを書き換えていった。

具体的には、[このコミット](https://github.com/58/scaline-board/commit/adc3b788dc45a6d7406c08dfdfb00c193dbdb95e)は苦労感が強い。
3系のSlickライブラリを使う場合、`com.typesafe.play`ではなく`org.playframework`と書く必要がある。
また、DB設定は`db.default`ではなく、`slick.dbs.default.profile`を指定して`slick.dbs.default`で設定しなくてはいけない。
この辺は[公式のSlickサンプル](https://github.com/playframework/play-samples/tree/3.0.x/play-scala-slick-example)を見て、何とかエラーを解決することができた。
まぁ公式サンプルをちゃんと読み込まないぼくが悪いと言えば悪い。


# まとめ
正直、公式のSlickサンプル以下のものを作っただけで、我ながら技術的にもプロダクト的にも面白さはないと思う。
しかし、日本語のScala 3・Play・Slick利用の開発体験記はあまりインターネットに多くないと思うので、これでも十分誰かにとって有益になるんじゃないかと信じてる。

ぼくとしては、Scalaの面白さというより、フレームワーク・ライブラリでかなり消耗したので、不完全燃焼なところが大きい。
RESTful APIで表示・書込を実装とかスレッド式掲示板作成など、もっと規模を大きくした開発をやってScalaを書きたいところ。
また、今回はユニットテストには触れなかったので、その辺の体験がどうなるかも今後試していきたいなと思いました。
