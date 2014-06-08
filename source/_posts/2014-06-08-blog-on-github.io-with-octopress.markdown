---
layout: post
title: "Blog on Github Pages with Octopress"
date: 2014-06-06T23:20:00-07:00
comments: true
published: true
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

<!-- more -->

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

ざっくりなイメージを図にまとめきれていないけども・・・図にすると・・・  
![zakkuri flow](http://blog.branch4.pw/images/2014/06/jekyll_octopress_flow.png)  

こんな感じ。  
Octopressが、rakeで何から何まで出来るようにしてくれているので、  
rakeコマンド打ちまくって、
- サイト生成
- サイトプレビュー
- サイトデプロイ（公開）
する感じ。

### how to setup github.io
----------
まずは、github.ioでブログを公開するためのセットアップから。
概ね、[koko](http://octopress.org/docs/setup/)のパクリです。

1. gitをinstallする  
  これができない人は[ameba blog](http://ameba.jp)へどうぞ
2. ruby(>=1.9.3)をinstallする  
  これができない人は[ameba blog](http://ameba.jp)へどうぞ
3. Octopressをdownloadしてsetupする

        $ git clone git://github.com/imathis/octopress.git octopress  
        $ cd octopress

    octopress directory配下の、Gemfileをみると、octopressでやってくれる  
    諸々の便利機能に必要なgemが列挙されている。

        $ cat Gemfile
        source ¥"https://rubygems.org  

        group :development do  
          gem 'rake', '~> 0.9'  
          gem 'jekyll', '~> 0.12'  
          gem 'rdiscount', '~> 2.0.7'  
          gem 'pygments.rb', '~> 0.3.4'  
          gem 'RedCloth', '~> 4.2.9'  
          gem 'haml', '~> 3.1.7'  
          gem 'compass', '~> 0.12.2'  
          gem 'sass', '~> 3.2'  
          gem 'sass-globbing', '~> 1.0.0'  
          gem 'rubypants', '~> 0.2.0'  
          gem 'rb-fsevent', '~> 0.9'  
          gem 'stringex', '~> 1.4.0'  
          gem 'liquid', '~> 2.3.0'  
          gem 'directory_watcher', '1.4.1'  
        end  

        gem 'sinatra', '~> 1.4.2'  

    なにはともあれinstall。

        $ gem install bundler
        $ rbenv rehash # rbenv使ってる場合のみ
        $ bundle install
        $ rake install # octopressのデフォルトテーマがinstallされる


4. githubに下記repositoryを作成する  
  xxxx.github.io(xxxx:ユーザ名 or 組織名)

5. 下準備諸々  
  ここがコマンド１個しか打たない割に重要。というか複雑。
  打つコマンドは、

      $ rake setup_github_pages

  この時、さきほど作成した、repository情報を要求されます。(git@github.com:xxxx/xxxx.github.io.git的な)  
  以下、色々事前情報説明  
  - xxxx.github.ioで一般公開されるのは、master branch  
  - つまり、誰かが、http://xxxx.github.io にアクセスすると、master branchがみえる  
  - 公開前作業はsource branchで実施する  
  - 今作業してるdirectoryとrepository
    octopressのdirectoryで、更に、repositoryとしても、imathis/octopressなので、このまま記事書いても、どこにcommitするんだ？になる  

  上記を踏まえた作業を、rake setup_github_pagesがやってくれている、と。  

  具体的には、  

  - 実際に公開したいrepositoryを確認して、設定してくれる
  - remote(imathis/octopress)をoriginではなくoctopressに設定してくれる
  - originは、さっき確認したrepositoryに設定してくれる
  - active branchをmasterではなくsourceにしてくれる
  - さっき確認したrepositoryから、ブログのURLを設定してくれる
  - master branchを、_deploy directoryに準備してくれる  

  さて、ここまで来ると、ほとんど出来たも同然。  
  - コンテンツはない
  - Jekyllに食わせるファイルは全部そろっている
  - 本番にpushするためのdirectory(_deploy)の準備は整っている

  なので、
  - コンテンツを作って
  - Jekyllに食わせて
  - 本番にpushすればOK!
  です。

6. ブログ生成＆本番公開  
  コンテンツを作るところは別途エントリを書こうと思うので、その後から。  
  まずは、下記コマンドで、Jekyllに必要ファイルを食わせて、サイトを生成します。  

      $ rake generate

  で、生成はされてるものの、本番前に確認したいので、下記でpreviewします。  

      $ rake preview

    上記で、webrickが立ち上がるので、適当なブラウザで、  
    http://127.0.0.1:4000  
    にアクセスして確認。問題なければ、下記を実行してソースをsource branchにpushしておきます。

      git add .
      git commit -m 'your message'
      git push origin source

    そして、下記を実行して本番にpushします。

      $ rake deploy

はい、これで晴れて公開完了！  
要点は、こんなところでしょうかな。  

1. octopressを持ってくる  
2. githubに専用repository作る  
3. repositoryのsource branchでoctopress/jekyll用のファイル編集をする  
4. ファイル編集が終わったら、サイトを生成する  
5. repositoryのmaster branchに本番用ファイルをpushする  

実はまだ謎なところが多いんですが、、、次回は、コンテンツの書き方について！
