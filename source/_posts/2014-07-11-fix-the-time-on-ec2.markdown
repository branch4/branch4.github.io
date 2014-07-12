---
layout: post
title: "EC2の時間を合わせる UTCに戻ってしまって困ってるあなたに"
date: 2014-07-12 01:30:00 +0900
comments: true
published: true
author: risterlab
categories: 
 - misc
---

はいはい、どもども、最近また大好きな野菜の奥深さに感動してばっかりのインフラエンジニア[@risterlab](http://diary.risterlab.com)です。 
来週群馬の農家さんにお邪魔して生トウモロコシ取りにいきます！ 
晴れますように。

![時計](http://blog.branch4.pw/images/2014/07/degital_clock.jpg)

今回はちょっとした小ネタを。

AWSのEC2のAmazon Linuxを立ち上げた人はまずUTCから日本時間に合わせるでしょう。
しかし、大抵検索すると出てくる方法で合わせてしまうと、
再起動したときにたまに時間がUTCに戻っちゃって、困ってる人向けに。

### よくネットで出てくるEC2の時間の合わせ方
----------

`cp /usr/share/zoneinfo/Asia/Tokyo /etc/localtime`

だいたい、これがでてきますね。
これをやるとすぐに日本時間になりますよ。じゃんじゃん。
って。
でもこれだとたまーに時間UTCに戻っちゃうので注意です。
しかもたまーに戻るからこれがまた困るのです。
私の場合実際にリリース前は何度再起動しても戻らなかったものが
リリース後半年以上経ってから、再起動を機にしれっと戻っていました。

cronとかアプリが時間見ていろいろやるから、大変でした。

### 時間がUTCに戻ってしまう理由
----------

実は/etc/localtime はglibcパッケージが更新されると、
パッケージ更新に含まれるスクリプトによりUTCに設定されます。
Amazon Linuxの場合、cloud_initがインスタンスの起動時にセキュリティの更新を実行するので、
そのタイミングで設定が初期化されてしまいます。

なのでglibcパッケージが更新された後、インスタンスを再起動すると、
glibcパッケージが更新され、/etc/localtimeが初期化されてしまう、
というカラクリ。

### 時間が初期化されないようにするには
----------

上記の防止策は２つ。

1. /etc/localtimeではなく、/etc/sysconfig/clockを編集する

```
ZONE="Asia/Tokyo"
UTC=False
```

2.cloud_initの設定ファイル（/etc/cloud/cloud.cfg）でセキュリティの更新を無効にする

セキュリティの更新はしたいけど、
時間は勝手に戻ってしまっては困る、っていうのが大抵の場合だと思いますし、
セキュリティの更新を無効にするのはオススメできません。

### まとめ
----------

いろんなサイトに書いてる
`cp /usr/share/zoneinfo/Asia/Tokyo /etc/localtime`
は今すぐ時間を合わせたいときには有効だけど
そのインスタンスの時間の設定をFIXすることにはならないので注意。
うっかりUTCに戻ってしまわないようにするには
/etc/localtimeではなく、/etc/sysconfig/clockを編集すること。

```
ZONE="Asia/Tokyo"
UTC=False
```

