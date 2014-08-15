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
VagrantとAnsibleとDocker組み合わせた感じ？？？

わからないので、"learn more" ボタンをぽちっとな。  

## ざっくり概要
### INFRASTRUCTURE AS CODE
きっと、コードでインフラを表現するためのテンプレ的なのを用意してくれてる、的な感じぽい。
まだ、Vagrant を使いこなしてる場合と、どう違うのかよくわからない。(Vagrantも使いこなせてないし)

### COMBINE MULTIPLE PROVIDERS
ほほう。なんかここは、Vagrant と違いそうじゃないか？
そもそも Vagrant の Provider をまだ勉強してないからなんとも言えないけど。。。  

つまり、AWS でサーバ起動して、お名前に登録してる DNS を修正してくれる、的なこと？
AWS/GCP 両方で使ってるインフラ管理できる？ま、使ってみればわかるか。

### EVOLVE YOUR INFRASTRUCTURE
設定ファイルを、VCS に登録して、インフラをどんどん進化させていけるよ！
って・・・Vagrant でも Ansible でも、Chef でもなんでもそうだからな・・・  

まだわかりません。  

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

<!-- more -->  

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

![Root4Logo](http://root04.github.com/images/email.png)  
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
  
<script type="text/javascript" language="javascript">
  num = Math.floor( Math.random() * 6 );
  document.write( aff[ num ]);
</script>
