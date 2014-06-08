---
layout: post
title: "Blog on Github Pages with Octopress"
date: 2014-06-06T23:20:00-07:00
comments: true
published: false
categories:
 - misc
---

  
こんちには、[@xengineer](https://twitter.com/xengineer01)です。
twitterのアカウント出しておきながら、ほとんどつぶやきませんが。
少し前から、Github Pagesでブログをはじめました。

が、これ、いまのところ書きづらいす。
githubなだけあって、投稿自体もレポジトリ管理だから、
「簡単ぽん！」ではない、という意味で。

でもその辺がエンジニアぽくていいのかもしれない。
(仕事はエンジニアではないけど、自称エンジニアです)

### prerequisite
----------

まずは、前提知識から。
今回のエントリで使うざっくり知識を下記に。

- [Github Pages](https://pages.github.com/)  
  - 安心  
    みなさんご存知github.com提供。  
  - repositoryから直接静的ページを配信してくれる  
    つまり、配信するHTMLページの作成は全然手伝ってくれない(そこが大変)  
  - サービス/プロダクトに紐づいたサイトを配信できる  
    github.com上に作成している個人/組織レポジトリに対応したwebsiteを立ち上げて公開できます。  
  - たぶん高速- 
    Githubのレポジトリから直接配信されることのどこが嬉しいかというと、  
    恐らく、世界中どこからでも使われてるサービスなので、  
  　どこからアクセスされても、かなり高速に配信できるんじゃないか？という点。  
- [Jekyll](http://jekyllrb.com/)  
  - 静的サイト生成ツール。 [middleman](http://middlemanapp.com/)的な  
  - Github PagesでやってくれないことをやってくれるYO !  
  - 特にGithub Pages専用なわけではなく、どこでも使えるぽい  
  - ただ、素で使おうとすると、細かいところまで全部手作業で、複雑らしい  
  - Jekyllについては、[koko](http://melborne.github.io/2012/05/13/first-step-of-jekyll/)がわかりやすかったので、リンクのみで
- [Octopress](http://octopress.org/)  
  - Jekyllを使ってブログサイトを構築するためのフレームワーク  
    便利！らしい  
    他にも、[Jekyll-bootstrap](http://jekyllbootstrap.com/)なるものもあるけど、  
    違いは、Octopressのほうが、楽。でも自由度が低い。そうです。  


### how to setup github.io
----------
まずは、github.ioでブログを公開するためのセットアップから。

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



