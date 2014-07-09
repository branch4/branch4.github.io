---
layout: post
title: "What I did after the installation of Octopress"
date: 2014-07-10 03:00:00 +0900
comments: true
published: false
author: xengineer01
categories: 
 - misc
---


こんちには、[@xengineer01](https://twitter.com/xengineer01)です。  
今日はおフランス帰りの飛行機の中で書いております。ほっほっほ。
※ 書き終わりませんでした
[セーヌ川](http://blog.branch4.pw/blog/images/2014/)

パリで開催されている、Japan Expoに、僕が所属している青空応援団が参加してたので、  
珍しく海外におりました。  
ほとんどExpo会場にいたからあんまり観光的なことはしてないけどね。  

さてさてまたしてもOctopress関連です。  

Octopressは、インストールしてすぐ公開できるけど、その状態はほんとに真っ新なわけで。  
サイドバーモジュール出したり、コメントつけられるようにしたり、と多少手をかけないと  
いけなかったりしたので作業メモ。  

### prerequisite
----------
Octopressはinstallして、github pagesでblogを書く直前まで行ってることが前提条件。  
![ここ](http://blog.branch4.pw/blog/2014/06/07/blog-on-github.io-with-octopress/)でいうとこの、  

```
$ git push origin source
```

までは終わってること。  
いまのところやったのは、  

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
disqus_short_name: 'shortname'
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

source/_includes/asidesにhtmlファイルがあって、それを読み込んで生成するというわけで。  
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

で、またどれを表示するかは、_config.ymlに記載してありんす。  
(github.htmlを追加した)  
```
default_asides: [asides/recent_posts.html, asides/delicious.html, asides/pinboard.html, asides/googleplus.html, asides/github.html]
```

更に、_config.ymlに、下記を記入。  

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

まずは、_config.ymlのいつもの行探しと追記。  

```
default_asides: [asides/recent_posts.html, asides/delicious.html, asides/pinboard.html, asides/googleplus.html, asides/github.html]
default_asides: [asides/recent_posts.html, asides/delicious.html, asides/pinboard.html, asides/googleplus.html, asides/github.html, custom/asides/affiliate.html] <- 追記後
```

### category generator/category listモジュールを導入するには
----------

### Google Analyticsで色々みれるようにするには
----------

