---
layout: post
title: "ansible-for-beginners-playbook"
date: 2014-10-10 17:38:38 +0900
comments: true
published: false
description: 
keywords: internet, web, 
author: xengineer
categories: 
---

またしても Ansible エントリでございます。[@xengineer01](https://twitter.com/xengineer01)です。  

![ansible logo](http://blog.branch4.pw/images/2014/09/ANSI_logotype_web.png)  
![ansible badge](http://blog.branch4.pw/images/2014/09/ansible_badge.png)  

ansible の基本的なとことか、ansible の ad-hoc mode については、
[以前の記事](http://blog.branch4.pw/blog/2014/09/09/ansible-for-beginners/)
をみていただきまして・・・  

今日は、ansible で playbook 使ってってみようかね。  
include とか、vars とか、role とか沢山便利機能があるけど、  
今回は playbook を使う！以上のことはしませぬ。  
なぜなら僕は ansible 初級。  

## ざっくりagenda

1. システム構成
    * 今回の検証環境のシステム構成紹介
2. Ansible playbookについて
    * 簡単に紹介
3. 目標
    * 本エントリで構築するシステム
4. Playbook 作成
    * 実際作って使ってみる

こんな感じ。

<!-- more -->  

## 写真紹介

まずは、いつも通り唐突ながら、なべちゃんの写真紹介。  
いつもいい写真撮るんですよね。

![nabechan kamiiso](http://blog.branch4.pw/images/2014/09/nabechan02_kamiiso.jpg)  
※ Hiroyuki Watanabeの写真で、[http://my-eyes.net/](http://my-eyes.net/)に元があります。

では本題に・・・

## システム構成
---
今回も、vagrant 使って、下記構成を作って使ってみました。
今回も、vagrant + ansible の連携とか、vagrantの設定まで踏み込んでいくつもりはなく、
いつでも検証自体を再現できるように使ってるだけです。  

```
       +-------------------------------------------+
       |      192.168.101.102                      |
       |                                           |
       |       +---------+            +---------+  |
 port  +--+    |         |            |         |  |
       |  |    |  nginx  |            |  MySQL  |  |
       +--+    |         |            |         |  |
       |       +----+----+            +----+----+  |
       |            |                      |       |
       |            |                      |       |
       |            |                      |       |
       |       +----+----+            +----+----+  |
       |       |         |            |         |  |
       |       | php-fpm +------------+wordpress|  |
       |       |         |            |         |  |
       |       +---------+            +---------+  |
       |                                           |
       +-------------------------------------------+

```

上のシステムで、ansible01 から２台のサーバに、deploy したり、
configuration managementしたり、ってのをやってみようかな、と。

### 環境について
- OS
  - Ubuntu 12.04 LTS \n \l


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
