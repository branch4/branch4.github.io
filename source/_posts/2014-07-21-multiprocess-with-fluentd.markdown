---
layout: post
title: fluentdを複数起動したい
date: 2014-07-21 01:36:47 +0900
comments: true
published: false
author: xengineer01
categories: 
---

## 概要
---
タイトル通りなんだけど、「fluentdを複数起動したいなー」  
って、質問いただいたので、簡単に手順を書いておきまっせ。  

### fluentdとは？
知らない人はいないね。はい。  

![fluentd logo](http://blog.branch4.pw/images/2014/07/fluentd-logo.png)

[tresuredata](www.treasuredata.com)で開発されたオープンソースのデータコレクタですのん。  
詳しいことは、[fluentdのwebsite](http://www.fluentd.org/)にいけば大体書いてあるね。  

### 前提
multiprocessといっても、  

[こういうの](https://github.com/frsyuki/fluent-plugin-multiprocess)  
とか  
[こういうの](http://orihubon.com/blog/2013/12/06/fluentd-multiprocess-input-plugin/)  
とは違う意味で。  

本当に、プロセスを別で起動したいんです！という話。  
ようは、片方のプロセス止めるときに、もう一方は止めたくないんだよね、的なアレ。  

<!-- more -->  

## 詳細
----------

まず、fluentd自体、ubuntuだとprecise/lucidしかサポートしていないので、  
binaryで入れたい方は対応osお使いください。  

### fluentdに限らず
fluentdに限らずな話ではあるんですが、なにかしらのアプリを複数起動する、  
ということ自体は、そんなに難しいことではないす。面倒な時はあるけど。  

気をつけるのは、以下。  

- IP address
- port
- other sharable resources(files/sockets)

大体のアプリは、上記を専有して起動しちゃうので、それが被らないように  
configをいじったりすれば概ね同時に起動して問題なす。  

### fluentdの場合
で、fluentd@ubuntu(precise pangolin)の場合にどうやったかね、というお話。  

まず、インストール後、今回触るファイルたちのリストは以下。  

- /etc/init.d/td-agent
- /etc/td-agent/td-agent.conf
- /etc/default/td-agent

それぞれ軽く解説をつけると・・・  

- /etc/init.d/td-agent
  - fluentdの起動スクリプト
  - このファイルいじって、PIDファイルが被らないようにする
- /etc/td-agent/td-agent.conf
  - fluentdのconfigファイル
  - これの設定をいじって、portが被らないようにする
- /etc/default/td-agent
  - fluentdの起動オプション設定ファイル
  - これの設定をいじって、起動時に読み込むconfigを切り替える

こんな感じ。実際のところは上記の３ファイルをコピって、二つずつ作るのだ。がっはっは。  

と、いうわけで、まずは各ファイルのコピーだけ作っておきましょ。  

```
% sudo cp /etc/init.d/td-agent /etc/init.d/td-agent_nginx
% sudo cp /etc/default/td-agent /etc/default/td-agent_nginx
% sudo cp /etc/td-agent/td-agent_nginx.conf /etc/td-agent/td-agent_nginx.conf
```

suffinxにnginxっていれてるのは、特に意味はないす。  
ただ、なんのためのfluentdなのか見分けつかなくなるようなファイル名はやめたほうがよいかと。  

### 各ファイルの詳細
#### /etc/init.d/td-agent_nginx

```
# Introduce the short server's name here
NAME=td-agent_nginx ★ここ修正

# Read configuration variable file if it is present
[ -r /etc/default/$NAME ] && . /etc/default/$NAME

# PATH should only include /usr/* if it runs after the mountnfs.sh script
PATH=/sbin:/usr/sbin:/bin:/usr/bin
USER=td-agent                   # Running user
GROUP=td-agent                  # Running group
DESC=td-agent_nginx             # Introduce a short description here★ここ修正
PIDFILE=/var/run/$NAME/$NAME.pid
DAEMON=/usr/lib/fluent/ruby/bin/ruby # Introduce the server's location here
# Arguments to run the daemon with
#DAEMON_ARGS="/usr/sbin/td-agent $DAEMON_ARGS --daemon $PIDFILE --log /var/log/td-agent/td-agent_nginx.log" ★ここ修正
DAEMON_ARGS="/usr/sbin/td-agent $DAEMON_ARGS $PIDFILE --log /var/log/td-agent/td-agent_nginx.log" ★ここ修正
echo $DAEMON_ARGS
SCRIPTNAME=/etc/init.d/$NAME
START_STOP_DAEMON_ARGS=""
```

#### /etc/td-agent/td-agent_nginx.conf

```
####
## Output descriptions:
##

<match td.*.*>
  type tdlog
  apikey YOUR_API_KEY

  auto_create_table
  buffer_type file
  buffer_path /var/log/td-agent_nginx/buffer/td ★ここ修正(同じログファイル握らないように)
</match>

<match debug.**>
  type stdout
</match>

####
## Source descriptions:
##

<source>
  type forward
  port 25224 ★defaultだと24224になってるので、明示的に変更しておく
</source>

# HTTP input
<source>
  type http
  port 18888 ★ここも明示的に変更しておく
</source>

## live debugging agent
<source>
  type debug_agent
  bind 127.0.0.1
  port 25235 ★ここも明示的に変更しておく
</source>
```
#### /etc/default/td-agent

```
# This file is sourced by /bin/sh from /etc/init.d/td-agent
# Options to pass to td-agent
DAEMON_ARGS="--config /etc/td-agent/td-agent_nginx.conf" ★これ追記。違う設定ファイル読み込むように
```

だめだわ。stopがうまくうごかん。

  
<a href="http://c.af.moshimo.com/af/c/click?a_id=442315&p_id=170&pc_id=185&pl_id=4157&guid=ON" target="_blank"><img src="http://image.moshimo.com/af-img/0068/000000004157.gif" width="300" height="250" style="border:none;"></a><img src="http://i.af.moshimo.com/af/i/impression?a_id=442315&p_id=170&pc_id=185&pl_id=4157" width="1" height="1" style="border:none;">
