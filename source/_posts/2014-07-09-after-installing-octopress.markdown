---
layout: post
title: "What I did after the installation of Octopress"
date: 2014-07-11 01:30:00 +0900
comments: true
published: true
author: xengineer01
categories: 
 - misc
---


こんちには、おフランス帰りの[@xengineer01](https://twitter.com/xengineer01)です。  
ほっほっほ。

![セーヌ川](http://blog.branch4.pw/images/2014/07/seine.png)

パリで開催されている、Japan Expoに、僕が所属している青空応援団が参加してたので、  
珍しく海外におりました。  
ほとんどExpo会場にいたからあんまり観光的なことはしてないけどね。  
セーヌ川と聞くと、いつも、セーム革ね、と思っているのは内緒です。  

さてさてまたしてもOctopress関連です。  

Octopressは、インストールしてすぐ公開できるけど、その状態はほんとに真っ新なわけで。  
サイドバーモジュール出したり、コメントつけられるようにしたり、と多少手をかけないと  
いけなかったりしたので、インストール後にやったことの作業メモ。  

### prerequisite
----------
Octopressはinstallして、github pagesでblogを書く直前まで行ってることが前提条件。  
[ここ](http://blog.branch4.pw/blog/2014/06/07/blog-on-github.io-with-octopress/)でいうとこの、  

```
$ git push origin source
```

までは終わってること。  
インストール後にいまのところやったのは、  

- 記事にコメントをつけられるようにした  
- google plusの+1モジュールを出した  
- サイドバーにGoogle Adsense/Affiliateを出した  
- github repositoryを出した、そして引っ込めた  
- category generator/category listモジュールを導入した  
- Google Analyticsで色々みれるようにした
こんなもんだったかな？  

<!-- more -->

今後やりたいなー、と思ってるのは、  

- 英語のブログと日本語のブログを分けて表示できるようにしたい  
- code blockいけてない表示を変更したい  
くらいかな。


### 記事にコメントをつけるには
----------
Octopress自体は、以前も紹介した通り、静的サイトGenerator Jekyllのラッパー的な  
存在なので、当然、コメントなんつー動的コンテンツ管理機能はないっす。  

そこは、[disqus](https://disqus.com/)っつー外部サービスに依存してるので、disqusアカウントを作りましょう。  
disqusアカウントの作り方は、ぐぐればいいよ。  

作ったら、下図的な感じで、"Add Disqus to Site"を選択。  
![add disqus to site](http://blog.branch4.pw/images/2014/07/add_disqus_to_site.png)

そんで、Site name入れたりする。例は適当にいれてみた。  
これでも問題ない。  
![site profile](http://blog.branch4.pw/images/2014/07/site_profile.png)

次は、この画面の、"Install"タブが選択された状態で出てくるので、  
下図みたいに、"General"を選択する。  
![configure disqus1](http://blog.branch4.pw/images/2014/07/configure_disqus1.png)

下の方にいくと、Your website's shortname is ikmenblog と出てくるので、  
これを覚えておく。  
![configure disqus2](http://blog.branch4.pw/images/2014/07/configure_disqus2.png)

そして、_config.ymlを下記のように編集してちょ。  
```
...
中略
...
disqus_short_name: 'ikemenblog'
disqus_show_comment_count: true
...
中略
...
```

short nameは、さっき覚えた名前ね。  
あとは、各postのmetadataに、下記を追記しておけばOK。
```
comments: true
```
これで投稿すれば漏れなくコメントがつけられるようになりますわ。Tre bien。  

### google plusの+1モジュールを出すには
----------
ブログのPVを増やすには内容が面白くないと何やってもだめなんだろうけどね。  
このモジュール、簡単に出せるので出してみようかな、と。  
もうちょい記事が増えて、Fanpageにも投稿できるようになったらFacebookモジュールも  
出しますかね。  

まずは、_config.ymlにこんな行があるので探しましょう。  
```
# Google +1
google_plus_one: false
google_plus_one_size: medium

# Google Plus Profile
# Hidden: No visible button, just add author information to search results
googleplus_user: 
googleplus_hidden: false
```

そんで、googleplus_userのところを埋めればOK。  
googleplus_userって何？という方は、[こちら](http://gphangouts.com/googleplusurl.html)。  
ちなみに僕もなんのことかわからなかった。  

あとは、どういう仕組みで表示されるかというと、これをみると少しわかるかね。  

```
$ ls -l source/_includes/asides
-rw-r--r--  1 user  group  406  6 15 08:32 delicious.html
-rw-r--r--  1 user  group 1005  6 15 08:32 github.html
-rw-r--r--  1 user  group  360  6 15 08:32 googleplus.html
-rw-r--r--  1 user  group  799  6 15 08:32 pinboard.html
-rw-r--r--  1 user  group  329  6 15 08:32 recent_posts.html

$ cat source/_includes/asides/googleplus.html
{% if site.googleplus_user %}
<section class="googleplus{% if site.googleplus_hidden %} googleplus-hidden{% endif %}">
  <h1>
    <a href="https://plus.google.com/{{ site.googleplus_user }}?rel=author">
      <img src="http://www.google.com/images/icons/ui/gprofile_button-32.png" width="32" height="32">
        Google+
    </a>
  </h1>
</section>
{% endif %}
```

source/\_includes/asidesにhtmlファイルがあって、それを読み込んで生成するというわけで。  
同じディレクトリに他にもファイルがあるけど、どれを表示するかは、_config.ymlに記載してありんす。  

```
default_asides: [asides/recent_posts.html, asides/delicious.html, asides/pinboard.html, asides/googleplus.html]
```
こんな感じで。  
これでgoogle plus +1モジュールは表示されるはず。

### github repositoryを出して、そして引っ込めるには
----------
これもgoogle plusモジュールと似たようなもん。  
さっきのlsの結果をみると、source/_includes/asides配下に、github.htmlがあったわけで。  

```
$ ls -l source/_includes/asides
cat source/_includes/asides/github.html
{% if site.github_user %}
<section>
  <h1>GitHub Repos</h1>
  <ul id="gh_repos">
    <li class="loading">Status updating...</li>
  </ul>
  {% if site.github_show_profile_link %}
    <a href="https://github.com/{{site.github_user}}">@{{site.github_user}}</a> on GitHub
  {% endif %}
  <script type="text/javascript">
    $(document).ready(function(){
      if (!window.jXHR){
        var jxhr = document.createElement('script');
        jxhr.type = 'text/javascript';
        jxhr.src = '{{ root_url}}/javascripts/libs/jXHR.js';
        var s = document.getElementsByTagName('script')[0];
        s.parentNode.insertBefore(jxhr, s);
      }

      github.showRepos({
        user: '{{site.github_user}}',
        count: {{site.github_repo_count}},
        skip_forks: {{site.github_skip_forks}},
        target: '#gh_repos'
      });
    });
  </script>
  <script src="{{ root_url }}/javascripts/github.js" type="text/javascript"> </script>
</section>
{% endif %}
```

で、またどれを表示するかは、_config.ymlに記載してあります。  
(github.htmlを追加した)  
```
default_asides: [asides/recent_posts.html, asides/delicious.html, asides/pinboard.html, asides/googleplus.html, asides/github.html]
```

更に、_config.yml、下記箇所をあわせて編集。  

```
# Github repositories
github_user: xxxxxxxx
github_repo_count: 0
github_show_profile_link: true
github_skip_forks: true
```

これで表示される、と。非表示にしたい場合は、default_asidesから抜いてしまえばOK。  

### サイドバーにGoogle Adsense/Affiliateを出すには
----------
せっかくブログ書くので、Adsense貼ったり、affiliate出したりして億万長者になろう！  
というわけで、こんな感じにすると貼れまっせ。  
ただ貼るだけだとaffiliateを貼ろうとすると、広告だらけの酷いサイトになっちゃうので、  
affiliate枠は一つにして、そこにランダムに複数affiliateを表示するようにしよう。  
そうすればABテストも兼ねられるし、一石二鳥。  

まずは、affiliateから。  

ざっくりやること。  
1. sidebarに読み込むhtmlの設定と設置
2. 1のhtmlから読み込んで実行するjsファイルの設置
3. jsファイルのinclude
4. sidebarのwidth変更
かな。  

[ここ](http://lblevery.com/sfn/aff/course/aff-banner-randomview/)参考にしましたね、私。  

#### sidebarに読み込むhtmlの設定と設置
_config.ymlのいつもの行探しと追記。  

```
default_asides: [asides/recent_posts.html, asides/delicious.html, asides/pinboard.html, asides/googleplus.html, asides/github.html]
default_asides: [asides/recent_posts.html, asides/delicious.html, asides/pinboard.html, asides/googleplus.html, asides/github.html, custom/asides/affiliate.html] <- 追記後
```

上記で、custom/asides配下に、affiliate.htmlを設置することにしてるので、  
source/_include/custom/asides/affiliate.htmlを下記内容で設置。  

```
$ cat source/_includes/custom/asides/affiliate.html
<script type="text/javascript" language="javascript">
  num = Math.floor( Math.random() * 6 );
  document.write( aff[ num ]);
</script>
```

Math.random() * 6 の、"6"は、何個のaffiliateバナーを回すかを書く。  

#### 1のhtmlから読み込んで実行するjsファイルの設置
次は、実行するjsファイルを設置。  
source/javascripts/affiliate.jsを、下記内容で設置。  

```
var aff = new Array();
aff[0] = 'affiliate tag1'
aff[1] = 'affiliate tag2'
aff[2] = 'affiliate tag3'
aff[3] = 'affiliate tag4'
aff[4] = 'affiliate tag5'
aff[5] = 'affiliate tag6'
```
affiliate tagXのところに、表示したいaffiliateのtagを突っ込む。  
(ダブルクオーテーションは、backslashでエスケープ)  

#### jsファイルのinclude
次は、html &lt;head&gt; - &lt;/head&gt;の中に、上記で設置したjsファイルをincludeするように、下記を追記する。  
僕は、source/_includes/head.html中に追記したです。  

```
<script type="text/javascript" src="http://xxx.yyy.zzz/javascripts/affiliate.js" charset="utf-8"></script>
```

#### sidebarのwidth変更
最後に、affiliateのサイズに合わせて、sidebar widthの調整。  
僕は、全部 300 x 250 でそろえたので、sidebar widthを、310pxにしたですよ。  

```
$ vi sass/base/_layout.scss

...
略
...
// Sidebar widths used in media queries
$sidebar-width-medium: 310px !default;
$sidebar-pad-medium: 5px !default;
$sidebar-pad-wide: 5px !default;
$sidebar-width-wide: 310px !default;
...
略
...
```

これでいける、はず。  

### category generator/category listモジュールを導入するには
----------
category generatorは、install直後から使えるpluginなので、_config.ymlを設定するだけで、  
category分けしたアーカイブページ作ってくれます。
設定は簡単に[公式サイト](http://octopress.org/docs/plugins/category-generator/)に説明あり。  

category generatorは、このブログでいうと、例えば、  
[こんな感じ](http://blog.branch4.pw/blog/categories/misc)のアーカイブページを作ってくれますよ。

category listは、[こっち](http://qiita.com/amay077/items/3296fdf1ea11c7c9ace4)を参照。  

こんな感じのリストを作れますよ。  
![category list](http://blog.branch4.pw/images/2014/07/category_list.png)

### Google Analyticsで色々みれるようにするには
----------
最後に、Google Analytics導入。  

これもあんまり書くことないんだけど、Analyticsアカウント登録して、  
tracking_id を、_config.ymlの下記箇所に書けばOK。

```
# Google Analytics
google_analytics_tracking_id: UA-XXXXXX-Y
```

Analyticsに数字が反映されるまで結構（24hr以上かかった）時間かかるけど、  
それを除けば、特に難しいことはないっす。  

と、いうわけで、いまのところ僕がoctopressインストール後にやったことをまとめてみましたです。  

色々間違ってメモってそうなので、その時はこっそり教えていただけるとうれしいです。  
Au revoir !
