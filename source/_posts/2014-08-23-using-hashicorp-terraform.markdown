---
layout: post
title: Hashicorpの新プロダクト、Terraformのドキュメントを読みましたよ！
date: 2014-08-23 01:00:00 +0900
comments: true
published: false
author: xengineer
categories: 
  - Infra
  - Web Development
---

はい、前回の宣言を無視して、Terraformエントリを書く[@xengineer01](https://twitter.com/xengineer01)です。  
(前回は、「次は Ansible」って言ってた)

## 目的と概要
---
タイトルそのままです。  
ほんとに読んだだけなんです！w

ちなみに、ドキュメント読みながらエントリ書いてみて、最終 terraform を実際使ってみるには至りませんでした。
結局こんなことに使うのね、というところまでは行ったものの、まだProviderサポートが弱いのと、
AWS をサポートしてるものの、EBS は作れなそうだったり、CloudFrontもこれからぽいので、
時間かけて使ってみても、僕が実用するのは先かな、ということで、  

- ドキュメント読んでみた感想

のみ！にとどまってますw

![readonly](http://blog.branch4.pw/images/2014/08/medium_2044819979.jpg)  
photo credit: <a href="https://www.flickr.com/photos/takomabibelot/2044819979/">takomabibelot</a> via <a href="http://photopin.com">photopin</a> <a href="http://creativecommons.org/licenses/by/2.0/">cc</a>

<!-- more -->  

一応ね、terraform インストールして、実行計画見たり、グラフみたりはしましたけど、
普通に公式ドキュメントに載ってるやつなので、ここでは割愛。  

全体的な感想としては、もうちょっと Providerが充実したら、使いどころによってはいいツールなんだろうなー。

## Terraform とは？
---

![terraform](http://blog.branch4.pw/images/2014/08/medium_4575417487.jpg)  
photo credit: <a href="https://www.flickr.com/photos/torley/4575417487/">▓▒░ TORLEY ░▒▓</a> via <a href="http://photopin.com">photopin</a> <a href="http://creativecommons.org/licenses/by-sa/2.0/">cc</a>

Serf/Consul/Vagrant/Packer を出してる、[Hashicorp](http://www.hashicorp.com)の新プロダクト。  
製品紹介には、、、  

Terraform is a tool for building, changing, and combining infrastructure safely and efficiently.

って書いてある。つまり・・・  

「Terraform は、安全で、効率的にインフラを構築/変更/組み合わせることができるツールですよこれ」  

と・・・。Hashicorpさん！いまいち伝わらないっす！
海外のサイトの説明ってこんな感じな気がする。他のも調べてみよう・・・ 


#### ansible
Ansible is the simplest way to automate IT.  
つまり、IT を自動化する、一番簡単な方法だよーん。と。

・・・ちょっとわかりづらい・・・かな。

#### chef
Automation for Web-Scale IT.  
お、ちょっとわかりやすい。

#### apache
The Number One HTTP Server On The Internet  
あ、すごい直球だった。

#### MySQL
The world's most popular open source database  
うむ。ちょっと有名どころすぎるとわかりやすくなりすぎる。

#### github
Build software better, together.  
コンセプトわかりやすい。

さて、茶番はこの辺にして本題へ。

結局、Terraform が何者か、よくわからないので、"learn more" ボタンをぽちっとな。  

## ざっくり概要
### INFRASTRUCTURE AS CODE
コードでインフラを表現するためのテンプレ的なのを用意してくれてる、な感じぽい。
Vagrant を自由自在に使いこなしてる場合と、どう違うのかよくわからない。(Vagrantも使いこなせてないし)

![infra](http://blog.branch4.pw/images/2014/08/medium_8106189987.jpg)  
photo credit: <a href="https://www.flickr.com/photos/andrewfhart/8106189987/">andrewfhart</a> via <a href="http://photopin.com">photopin</a> <a href="http://creativecommons.org/licenses/by-sa/2.0/">cc</a>

### COMBINE MULTIPLE PROVIDERS
ほほう。なんかここは、Vagrant と違いそうじゃないか？
そもそも Vagrant の Provider をまだ勉強してないからなんとも言えないけど。。。  

ちょっと戻ってみる限り、Vagrantは、単一 Provider にしか対応してなし。  

つまり、AWS でサーバ起動して、お名前に登録してる DNS を修正してくれる、的なこと？
(AWS と、お名前の 2 Provider)  

それか、AWS/GCP 両方で使ってるインフラ管理できる？ま、使ってみればわかるかもしれないので、次を読んでいこう。

### EVOLVE YOUR INFRASTRUCTURE
設定ファイルを、VCS に登録して、インフラをどんどん進化させていけるよ！
って・・・Vagrant でも Ansible でも、Chef でもなんでもそうだからな・・・  

![evolve](http://blog.branch4.pw/images/2014/08/medium_2922128673.jpg)  
photo credit: <a href="https://www.flickr.com/photos/spidermandragon5/2922128673/">bryanwright5@gmail.com</a> via <a href="http://photopin.com">photopin</a> <a href="http://creativecommons.org/licenses/by-nd/2.0/">cc</a>

まだ凄さがわかりまてん。。。  

## Terraform で何ができる？
いきなり飛びますが、ざーーーーーっと、ドキュメントを読んでみました。
読み進めても、使いどころが、ふわっ、としてて、ちゃんと落ちてこなかった部分もあり、
読み進めてたら大体読み終わっちゃった、的なね。

以下僕が読んだ感じだと、、、

- インフラをコード(というか設定)で表現できる
- インフラ設定を記述して、実行すると、設定通りの環境を構築できるよ！というもの
  - ここでいう「インフラ設定」は、大きく分けて下記2つかな、と思った
    - resource
      - サーバインスタンスから、IP 設定くらいまで
      - ようは、外部サービス(AWSみたいな)で、API 操作して決められるものを resource として扱ってる感じ
    - provisioning
      - 上のresourceに入らない設定
      - ansibleとかchefとか、はたまたスクリプト実行して設定するような設定
- 設定を書いて、実行する前に、実行計画が確認できる
  - つまーり、ちゃんと何が起きるかを確認してから実行できる
  - しかもグラフィカルに依存性をチェックできる
  - これは便利
- 実行後、実際の状態がどうなっているか、もコマンドで確認できる

こんなことをできますよ、と。  

これだけ見ると、この前ブログに書いてた、Vagrant と何が違うんだろうね・・・
って感じになるわけです。

## Terraform と Vagrant の違い
そのままだけど。何が違うんだろうね、と思ったので、まとめてみましたよ。
勘違いしてるところは、誰か教えて。

items| Vagrant  | Terraform 
:-----: | :------ | :-----
Catch copy | Development environments made easy | Terraform is a tool for building, changing, and combining infrastructure safely and efficiently.
Provider | VirtualBox | AWS
         | VMware     | CloudFlare
         | Docker     | Consul
         | KVM        | DigitalOcean
         | Hyper-V    | DNSimple
         | AWS        | Heroku
         | RackSpace  | 
Provisioning | File Upload      | たぶんなんでもござれ
             | script execution | 
             | Ansible          | 
             | CFengine         | 
             | Chef Solo        | 
             | Chef Client      | 
             | Docker           | 
             | Puppet Apply     | 
             | Puppet Agent     | 
             | Salt             | 
Others | Destroy が結構気軽に実行できる(実行までのステップが超短い！) | Destroy を間違えて実行する確率は、Vagrant に比べたら相当低い
       | 統合的な管理はできない(Load balancer/DNS/Storege等の管理は現状できない) | AWS は、ELB/RDS/EIP/EC2/VPC/Route53/S3/Security Groupなどなど、結構いじれるので、かなり統合的管理ができる
       | 元々の思想が、開発環境/サンドボックス環境構築用のツール | 特に開発環境用、とは銘打ってないから本番用なんだ、きっと
       | インフラ環境をコードで記述して、Build & Scrap しまくれて、すげー便利 | インフラ環境を設定で記述して、構築できるので便利(destroy はVagrantのほうが気軽)

こんな感じなのかなぁ。  
Terraform に関しては、ちょっと作業フローも付け加えておこうかな。
きっとこんな感じになるんじゃないかなぁ、ってレベルなので、実際はわからんけど。

- たぶん作業フローが
  - 設定ファイルを書く
  - 実行プランを確認する
  - 実行する
  - 実行結果を確認する

になるので、Vagrant よりも、いちいち確認するために止まる箇所をコマンドベースで置いてるのが違う気がする

という感じで、Terraform vs Vagrant だと、

- そもそも最初の思想が違ってたり、
- 環境破壊の容易さが違ったり、
- 作業フローもたぶん違ったり、
- multi providerへの対応有無が違ったり、
- 統合管理できる/できないがあったり

っていう違いがあるのかな。だとするとやっぱり、  

- 開発環境 == Vagrant  
- 本番環境 == Terraform  

になるんだろうな。もちろん、VPS 1台使ってサービス展開する分には、ぶっちゃけどっちでもええわ。
たぶんちゃんと管理しとけば(してなくても)、Vagrant で運営できる。でもたぶんそのレベルは、Vagrant すら必要ない。  

でかい環境が必要な場合とか、小さい環境が何個も必要な場合とかは、うまく使い分けるのがええだろね。
provisioning のステージングを Vagrant でやってみて、うまくいったら、それ commit して Terraform 環境で適用、みたいな。
でもterraformは、初期構築ツールで、運営開始後に全部実行することはないのかも？
その辺まだわからん。まぁとりあえず、CloudFormation の進化版になりうるやーつだ。やっと理解できたw  

ではインストール。

### terraform インストール
----------
[ここ](http://www.terraform.io/intro/getting-started/install.html)を参考にインストールしましょう。  

- ダウンロード
- 展開
- PATH 通す

これが出来ない人は、これ以上進んでも理解できない可能性が高いので、やめといたほうがいいです。  

インストール終わったら、動くかだけみてみる。
```
$ terraform version
Terraform v0.1.1
```
なるほど、僕が今回使うのは、v0.1.1 だそうです。

### terraform 使う時のざっくりフロー
----------
まずは、登場用語リストとその説明から。

- .tf ファイル
  - 設定ファイル。これにどんなインフラを構築するか設定する
  - terraformコマンド実行するディレクトリに入ってる、.tfファイルが全部読み込まれて実行される
- provider
  - インフラを構築する手段
  - aws なのか、heroku なのか、的な感じ
- resource
  - 各Provider で設定できる項目のこと
  - aws だったら、eipとか、elbが一つのresource になってる

そして次は、実際 terraform 使うかー、ってなったら踏みそうな手順。
きっと、僕はこんな感じなんだろう。

1. システム構成検討
これはそのまんま。まずはどんなシステム作りたいかを考えようよ。
2. tf ファイル記述
これもそのまんま。考えたシステム構成図に則って書くだけといえば書くだけ。
3. 実行計画確認

   下記コマンド実行。

   ```
   $ terraform plan
   ```

   で実行計画が表示されるので、OKかを確認する。
  
4. 実行

   計画が OK だったら、下記コマンドで実行。

   ```
   $ terraform apply
   ```

   実行、というより、適用のほうがしっくりくる。

5. 結果確認

   で、結果を下記コマンドで確認。

   ```
   $ terraform show hogehoge.tfstatus
   ```

ほむほむ。
なんとなくこんな感じなのね。

と、ここまで書いて、実際使おうかな、という段で、一旦 Provider の種類を
みてみよう、というわけで、公式ドキュメントに載ってるのをざっと表にしてみた。

Service name|Terraform name|Attribute
:---|:---:|:---
AWS|Basic|aws_autoscaling_group
|RDS|aws_db_instance
||aws_db_security_group
||aws_eip
||aws_elb
||aws_security_group
||aws_subnet
|EC2|aws_instance
||aws_internet_gateway
||aws_launch_configuration
|Route53|aws_route_table
||aws_route_table_association
||aws_route53_record
||aws_route53_zone
|S3|aws_s3_bucket
|VPC|aws_vpc

な、なにぃ・・・EBS ないのかな・・・CloudFront ないのかな・・・。
残念だなぁ・・・ちょっと動作みてみるのめんどくさくなってきたなぁ・・・
などなど、色々と去来する思いもあり、今回はこの辺でやめて、Ansible に戻ろうかなー、
ってなりましたw

  

<script type="text/javascript" language="javascript">
  num = Math.floor( Math.random() * 6 );
  document.write( aff[ num ]);
</script>
