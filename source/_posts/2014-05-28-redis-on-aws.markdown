---
layout: post
title: "ElastiCache vs Redis@EC2"
date: 2014-04-17T23:20:00-07:00
comments: true
published: false
categories:
 - misc
---

# Title
---
  
  

### Lists
----------

1. list1
   - list1
   - list2
   - list3
   - list4
1. list2
1. list3
1. list4

### Table
----------

左揃え | 中央揃え | 右揃え
:----- | :------: | -----:
1111   | 2222     | 3333 
4444   | 5555     | 6666
7777   | 8888     | 9999  


### Codes
----------
`$ command !!`

    $ ls -l
    $ ls -ltr
    $ sl -l
    $ sl -al

    param1 :
    説明  

### Links
************

[google test link](http://google.com "google")  
[google test link wo title](http://google.com)

### Images
************

DynamoDB運営の場合との比較を考えると・・・

◎メリット
・ElastiCacheで問題がある場合に、 Redis on EC2に移行しやすい
・インスタンス単位での課金なので、コストが読みやすい
・冗長構成の構築は簡単(でもDynamoDBは常に冗長構成だから優位性はない)

◎デメリット
・インスタンスを起動して運用するタイプのサービスなので、基本的に運用が発生する
・サポートされないRedisの機能、コマンドがいくつかある
　→RDBのダンプ機能は未サポート
　→rename command/saveコマンド等は未サポート
・メンテナンスウィンドウの設定が必須(指定した時間帯はAWS側メンテが入る可能性がある)
　→マルチAZ構成にして、他 AZにレプリを作成、通信断時は自動Failoverの構成が必須
・現実的に考えて、RDBダンプは必須(未サポートだけど)
　→EC2上でredisを動かして、ElastiCacheからレプリを貼る。そこでダンプする構成が必要
　　→ElastiCacheだけで完結しないので意味が・・・
oot4Logo](http://root04.github.com/images/email.png)  
source/images配下にcommitしてpushした画像は、http://branch4.github.com/images/xxx.xxx  
でみれる。


### Emphasize
----------
*EMPHASIZE*  
_EMPHASIZE_  
**EMPHASIZE**  
__EMPHASIZE__  

### Quote
************
> a said a  
>> b said ba  
>>> c said ca  



ElastiCacheについて調べた結果です。
調査方法としては、ElastiCache vs Redis on EC2での比較調査を
して、そこからDynamoDBとの比較をしています。
(DynamoDBについては前回以降それほど深い調査はしていないので、
明確な資料はない状態です)

いくつかファイルを添付しています。

elasticache.pdf：
　簡単にまとめてます。
　ここからみていただくのがよいかと思います。

redis_elasticache_config.pdf：
　redis/elasticacheに共通している設定項目で、違いがあるかの比較。
　一部redisにはない設定項目も含まれています。
redisonly_config.pdf
　redisにしか存在しない設定項目。ざっくり、下記に分かれています。
　あんまりいじられるとAWSがつらくなりそうな項目が多かったです。
　- 基本項目
　- replication関連
　- RDBダンプ関連
　- AOF関連

現状のインフラエンジニアの人数、規模感を考えると、なるべく運用
必須のシステムはないほうがよいと思います。
まだコスト比較できていないので、決定打ではないかもしれませんが、
ElastiCacheは、DynamoDB比だとまだ運用コストが高い(手がかかる)
かな、と思うので、可能であればDynamoがお勧めかと思います。

