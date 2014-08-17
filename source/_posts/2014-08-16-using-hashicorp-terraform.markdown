---
layout: post
title: Hashicorpの新プロダクト、Terraformを使ってみようかね
date: 2014-08-13 22:46:05 +0900
comments: true
published: false
author: xengineer
categories: 
---

はい、前回の宣言を無視して、Terraformエントリを書く[@xengineer01](https://twitter.com/xengineer01)です。  
(前回は、「次は Ansible」って言ってた)

## 目的
---
タイトルそのままですわ。  
まぁ、Ansibleとかそんなに目新しいものでもないし、ちょい前に出た Terraform のほうが
需要あるかなと思って。(つーか PV 取れるかな、という邪な気持ちで)  

## Terraform とは？
---
Serf/Consul/Vagrant/Packer を出してる、[Hashicorp](http://www.hashicorp.com)の新プロダクト。  
製品紹介には、、、  

Terraform is a tool for building, changing, and combining infrastructure safely and efficiently.

って書いてある。つまり・・・  

「Terraform は、安全で、効率的にインフラを構築/変更/組み合わせることができるツールですよこれ」  

と・・・。Hashicorpさん！いまいち伝わらないっす！
海外のサイトの説明ってこんな感じな気がする。他のも調べてみよう・・・ 

<!-- more -->  

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

Vagrantの特徴
-  Development environments made easy
  - 元々、開発環境、サンドボックス環境構築用のツール
- 構築したいインフラ環境を、コードとして記述できるので、ビルドアンドスクラップしまくれて、すげー便利
- Provider は、下記から一つ、選択できる
  - VirtualBox
  - VMware
  - Docker
  - KVM
  - Hyper-V
  - AWS
  - Rackspaceなどなど
- Provisioning はかなり柔軟
  - script execution
  - ansible/chef などのprovisioning toolも利用可能
- destroy が結構気軽に打てる(実行までのステップが超短い)
- 統合的な管理はできない(Load balancer/DNS/storage等の管理は現状できない)

Terraformの特徴
- Terraform is a tool for building, changing, and combining infrastructure safely and efficiently.
  - 特に開発環境用、とは銘打ってないから本番用なんだ、きっと
  - しかも、安全で効率的に、って書いてある
- 構築したいインフラ環境を、コードとして記述できるので、ビルドアンドスクラップしまくれて、すげー便利
  - でも Vagrant より少し destroy が面倒
- たぶん作業フローが
  - 設定ファイルを書く
  - 実行プランを確認する
  - 実行する
  - 実行結果を確認する
  になるので、Vagrant よりも、いちいち確認するために止まる箇所をコマンドベースで置いてるのが違う気がする
- Provider は、下記から複数選択できる
  - AWS
  - CloudFlare
  - Consul
  - DigitalOcean
  - DNSimple
  - Heroku
- Provisioning はたぶんかなり柔軟
  - Vagrant ほど、オプションやらdirective的なものが揃ってはいないけど、
    remote_execで任意コマンドキックできるので、柔軟性でいえば、同等
- destroy を間違えて実行する確率は、Vagrant に比べたら相当低い
- AWS は、ELB/RDS/EIP/EC2/VPC/Route53/S3/Security Groupなどなど、結構いじれるので、かなり統合的管理ができる

という感じで、Terraform vs Vagrant だと、
- そもそも最初の思想が違ってたり、
- 環境破壊の容易さが違ったり、
- 作業フローもたぶん違ったり、
- multi providerへの対応有無が違ったり、
- 統合管理できる/できないがあったり

ということを考えるとやっぱり、  

- 開発環境 == Vagrant  
- 本番環境 == Terraform  

になるんだろうな。もちろん、VPS 1台使ってサービス展開する分には、ぶっちゃけどっちでもええわ。
たぶんちゃんと管理しとけば(してなくても)、Vagrant で運営できる。でもたぶんそのレベルは、Vagrant すら必要ない。  

でかい環境が必要な場合とか、小さい環境が何個も必要な場合とかは、うまく使い分けるのがええだろね。
provisioning のステージングを Vagrant でやってみて、うまくいったら、それ commit して Terraform 環境で適用、みたいな。
でもterraformは、初期構築ツールで、運営開始後に全部実行することはないのかも？
その辺まだわからん。

ま、なんとなくな違いを書いてみたところで、実際に書いてみようか。

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

### terraform のルール
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

そして次はやること。

1. システム構成検討
これはそのまんま。まずはどんなシステム作りたいかを考えようよ。
2. tfファイル記述
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
  

<script type="text/javascript" language="javascript">
  num = Math.floor( Math.random() * 6 );
  document.write( aff[ num ]);
</script>
