---
layout: post
title: "Posting a Blog with Octopress"
date: 2014-06-19T00:20:00-07:00
comments: true
published: true
author: xengineer01
categories:
 - misc
---

※  内容間違ってんぞ！と、@adorechicさんに指摘いただいたので、訂正しております。

こんちには、[@xengineer01](https://twitter.com/xengineer01)です。  
前回に引き続きOctopress関連です。

今回は、実際投稿するときどうするか、という点を。  
投稿するにあたって、ちょいとだけ前回に関連する内容も書くので、少しかぶり気味。  

### prerequisite
----------
Octopressはinstallして、github pagesでblogを書く直前まで行ってることが前提条件。  
[ここ](http://blog.branch4.pw/blog/2014/06/07/blog-on-github.io-with-octopress/)でいうとこの、  

```
$ git push origin source
```

までは終わってること。  
そこまできてたらこのblogは読まなくても書け・・・ます、はい。  
強いて言えば、準備はできてるけど、なんかよくわからんうちに準備できた人用。  
それか、このブログみたいに、複数で更新してるので、セットアップした人と、  
書く人が違う場合用。  

<!-- more -->

### Cloning
----------
では、setupしたrepositoryを、  

```
https://github.com/xxxxxx/xxxxxx.github.io.git
```

とした場合。まずは、repositoryをlocalにcloneします。  

```
$ git clone https://github.com/xxxxxx/xxxxxx.github.io.git
```

~~この時点で、active branchは、source branchに設定されております。~~  
これは、うそでしたｗｗ  
clone直後は、master branchでした。  
cloneしたディレクトリに、cdして、下記を実行するとsource branchに変わるです。  

```
$ git checkout source
```
この時点で、active branchは、source branchに設定されております。  
が、正解であります！

```
$ git status
```
でみるとわかるよん。  
前回の記事でも書いたけど、記事書くときの実作業は、このままsource branchでやって、  
本番投稿は、master branchにする、てな手口。  

### deploy用設定
----------
次はdeploy用の設定しときます。  
cloneしたdirectoryにcdして、下記コマンドを実行。  

```
$ bundle exec rake setup_github_pages
```

前回の記事でも出てきたんだけど、簡単にいうと、  
_deploy directoryを作ってくれる。  
で、本番deployのときは、この _deploy directoryにあるmaster branchにpushするので、  
_deploy に、master branchをpullしとく、と。  

```
$ cd _deploy
$ git pull origin master
```
これでdeploy用の設定は完了。

### しっぴつ！
----------
ここでは２点紹介。  

- どこに書く？
- 画像はどうする？

#### どこに書くか。  
例えば、この記事のファイルは下記に格納されておる。  

```
$ ls source/_post/2014-06-08-blog-on-github.io-with-octopress.markdown
```

ポイントはファイル名。  

&lt;year&gt;-&lt;month&gt;-&lt;day&gt;-title.&lt;markdown/html&gt;

的な。これ以外にすると静的ファイルが生成されまてん。  
内容は、こんな感じ。  

```
---
layout: post
title: "Blog on Github Pages with Octopress"
date: 2014-06-06T23:20:00-07:00
comments: true
published: true
categories:
 - misc
---

ここから下に本文を書く
```

上のほうに、meta-dataを突っ込んで、その下にコンテンツを書く感じでござい。  
meta-dataの意味は・・・みればすぐわかるような内容しか設定してないので解説なしで。  
他にも色々あるので使いたい人は調べてみてちょ。  

#### 画像はどうするか  
はい、画像は、source/images配下にcommitします。  
で、commit/pushすると、  

http://github.com/xxxxxx/images/yyyyyy.png

でみれるようになります。  
markdownの場合は、以下な感じで書いとけば、deployしたときに見えるようになります。  

```
[test](http://github.com/xxxxxx/images/yyyyyy.png)
```

当然のようにpreviewのときは見えないので、previewでみたい場合は、画像だけ  
先にpushしとかないとだめかと。(面倒なので僕は一発本番チェック派)  

たぶん、この２つがわかればコンテンツ作るとこまではできるはず。  
コンテンツ書いたら、

```
$ bundle exec rake generate; bundle exec rake preview
```

を実行すると、http://127.0.0.1:4000 に、Webrickがあがるので、  
ブラウザで見た目チェックと修正を繰り返して、完成までがんばりましょう。  

### 投稿！
----------
そんな工程を経て記事が無事完成したら、本番公開と、作業ファイルのcommitをしましょう。  
Octopressは、Jekyllに作業ファイルを食わせて本番用ファイルを生成するので、作業ファイルを  
commitしとかないと、次回本番生成時に記事がなくなっちゃうので要注意。  

そんなわけで、下記で作業ファイルのcommitと本番deploy。

```
$ git add .
$ git commit -m 'nanka message'
$ git push origin source
$ bundle exec rake deploy
```

あとは無事投稿されてるかを本番で確認したら、完了！  
おつかれちゃん。  

### rake task for generatign new posts
----------
はい、もう終わりなんだけど、新規投稿するときに、いちいちファイルを0から作るのは  
めんどくさいので、専用raketaskが用意されております。一応その共有。  

```
$ rake new_post['title']
```
を実行すると、下記ファイルができあがります、と。  
source/_posts/2014-06-19-title.markdown  

正直、そんなに重要じゃなさげにみえるんだけど、例えば、  

- テンプレを用意しておきたい
- adsenseを毎回貼るのがめんどくさい
- &lt;!-- more --&gt;を忘れないようにしたい

などなどある人は、Rakefileのtaskに加筆するとスーパー便利になったり。  
そんな使い方もありますよ。

今日はこの辺で。

