---
layout: post
title: fluentdのテスト環境をvagrantでセットアップしてみるべ
date: 2014-07-31 14:17:39 +0900
comments: true
published: false
author: xengineer01
categories: 
---

こんにちは。Vagrant使ってみよ。と思った[@xengineer01](https://twitter.com/xengineer01)です。  

# 概要
---
まぁ使ってみたかっただけですわ。  
fluentdの検証するし、今後も環境構築は何回もするし、ついでだから vagrant使ってみよ、  
なノリです。

## vagrantって？
---


## インストール
----------

## セットアップ
----------
まずは、[公式サイト](http://www.vagrantup.com/)のドキュメンツを辿ってみるべし。

### Vagrantfile なる設定ファイルが肝
### Projectディレクトリと、Vagrantfile を作る！

下記コマンドを実行してね。
```
$ mkdir <PROJECT_ROOT>
$ cd <PROJECT_ROOT>
$ vagrant init
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.

```

コマンドで生成された Vagrantfileから、コメントの行を消すと、、、
```
$ cat Vagrantfile

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "base"
end
```
ちっさ！(コメントすげー多い)
書いてあることもすぐわかるレベルすな。

そして、Vagrantfile は、git,svn等の、VCSにcommitすべきものらしい。

### Box とは？

Vagrant では、box っていうのが、ひとつのベースイメージになるんだと。
仮想イメージの呼び方違い版かね。

#### Box のインストール
このコマンド実行しましょう。

```
$ vagrant box add hashicorp/precise32
```

そうすると、下記ディレクトリ配下に、VirtualBoxのイメージがダウンロードされたり、
Vagrantfileのような諸情報が格納されます。

${HOME}/.vagrant.d
${HOME}/.vagrant.d/boxes
${HOME}/.vagrant.d/data
${HOME}/.vagrant.d/gems
${HOME}/.vagrant.d/rgloader
${HOME}/.vagrant.d/tmp

たぶん大事なのは、boxex配下なのかな？きっとそうだろう。  
イメージのダウンロード元は、[ここ](https://vagrantcloud.com/)からみたい。  


- Vagrantfile で、下記を定義する
  - 起動するマシンスペック
  - インストールするアプリケーション
  - どうやってアクセスするか

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
  
<a href="http://c.af.moshimo.com/af/c/click?a_id=442315&p_id=170&pc_id=185&pl_id=4157&guid=ON" target="_blank"><img src="http://image.moshimo.com/af-img/0068/000000004157.gif" width="300" height="250" style="border:none;"></a><img src="http://i.af.moshimo.com/af/i/impression?a_id=442315&p_id=170&pc_id=185&pl_id=4157" width="1" height="1" style="border:none;">
