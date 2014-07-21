---
layout: post
title: fluentdを複数起動したい
date: 2014-07-22 22:30:00 +0900
comments: true
published: true
author: xengineer01
categories: 
  - Infra
  - Web Development
---

## 概要
---
タイトル通りなんだけど、「fluentdを複数起動したいなー」  
って、質問いただいたので、簡単に手順を書いておきまっせ。  

### fluentdとは？
---
知らない人はいないね。はい。  

[![fluentd logo](http://blog.branch4.pw/images/2014/07/fluentd-logo.png)](http://www.fluentd.org/)

[tresuredata](http://www.treasuredata.com)で開発されたオープンソースのデータコレクタですのん。  
詳しいことは、[fluentdのwebsite](http://www.fluentd.org/)にいけば大体書いてあるね。  

この前行った、AWSSummit Tokyo2014でも、どの会社も、  
「弊社では、ログはfluentdで処理しています！(ドヤ)」的な感じでした。  
使うのが普通ですね、はい。  

ちなみに僕はインストールするの今回初です(笑)  
弊社では使ってるんですけどね、もう現場仕事から離れること・・・200X年くらい。  

なのでね、今回の記事のイメージはこんな感じだけど、なんか間違って理解してたら誰か突っ込みよろ！  

![multiprocess with fluentd](http://branch4.blog.pw/images/2014/07/multiprocess.png)

### 前提
---
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
---
fluentdに限らずな話ではあるんですが、なにかしらのアプリを複数起動する、  
ということ自体は、そんなに難しいことではないす。面倒な時はあるけど。  

気をつけるのは、以下。  

- IP address
- port
- other sharable resources(files/sockets)

大体のアプリは、上記を専有して起動しちゃうので、それが被らないように  
configをいじったりすれば概ね同時に起動して問題なす。  

### fluentdの場合
---
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
---
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

インストール直後のconfigっす。コメントは邪魔なのでとっぱらってます。  

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
#### /etc/default/td-agent_nginx

起動時につけるオプションね。  
ここで読み込むconfig変更しますゆえお忘れなく。  

```
# This file is sourced by /bin/sh from /etc/init.d/td-agent
# Options to pass to td-agent
DAEMON_ARGS="--config /etc/td-agent/td-agent_nginx.conf" ★これ追記。違う設定ファイル読み込むように
```

このくらい修正をかけると、だ、下記コマンドで動くはず、だ。  
```
% sudo /etc/init.d/td-agent start
% sudo /etc/init.d/td-agent_nginx start
% ps aux|grep td-agent
td-agent 11314  0.0  3.5  98888 17912 ?        Sl   07:05   0:00 /usr/lib/fluent/ruby/bin/ruby /usr/sbin/td-agent --daemon /var/run/td-agent/td-agent.pid --log /var/log/td-agent/td-agent.log
td-agent 11317  2.4  5.2 125704 26260 ?        Sl   07:05   0:00 /usr/lib/fluent/ruby/bin/ruby /usr/sbin/td-agent --daemon /var/run/td-agent/td-agent.pid --log /var/log/td-agent/td-agent.log
td-agent 11348  0.0  3.5  98888 17908 ?        Sl   07:05   0:00 /usr/lib/fluent/ruby/bin/ruby /usr/sbin/td-agent --config /etc/td-agent/td-agent_nginx.conf --daemon /var/run/td-agent_nginx/td-agent_nginx.pid --log /var/log/td-agent/td-agent_nginx.log
td-agent 11351  5.5  5.2 125704 26224 ?        Sl   07:05   0:00 /usr/lib/fluent/ruby/bin/ruby /usr/sbin/td-agent --config /etc/td-agent/td-agent_nginx.conf --daemon /var/run/td-agent_nginx/td-agent_nginx.pid --log /var/log/td-agent/td-agent_nginx.log
```

ぱちぱちぱちぱちーーー。いやまぁそらそーだ。  
で、ここまでは結構すぐいきましたわい。ところが、、、、  

```
% sudo /etc/init.d/td-agent stop
```

つって、止めようとすると、、、なんと！両方のプロセスが止まる！  
ひぎぃ・・・・/etc/init.d/td-agentの中身としばらくにらめっこ。  

debian系のinit scriptの中では、start-stop-daemonってのがdaemonの起動・停止に  
まつわるetc をやっていて、そのあたりをちょっと調べてみることに。  

- fluentdのdo_stopでは、２回start-stop-daemonが呼ばれている
  - 1回目: start-stop-daemon --stop --quiet --retry=TERM/30/KILL/5 --pidfile $PIDFILE --name ruby
  - 2回目: start-stop-daemon --stop --quiet --oknodo --retry=TERM/30/KILL/5 --exec $DAEMON

と、いうのと、man start-stop-daemonの中身をみると。。。

```
Note: unless --pidfile is specified, start-stop-daemon behaves similar to killall(1).
start-stop-daemon will scan the process table looking for any processes which match
the process name, uid, and/or gid (if specified).

Any matching process will prevent --start from starting the daemon. All matching processes
will be sent the TERM signal (or the one specified via --signal or --retry)
if --stop is specified.

For  daemons  which  have  long-lived children which need to live through a --stop,
you must specify a pidfile.
```

つまり・・・--pidfileオプションが指定されてなければ、killallと同じように動く、と・・・  
おお・・・そりゃ両方のプロセス殺されるわけだ・・・  
逆に言えば、--pidfileを指定しとけばkillの動作なのかな・・・  

で、straceかけて、2パターン検証してみたYO。

#### 1回目: start-stop-daemon --stop --quiet --retry=TERM/30/KILL/5 --pidfile $PIDFILE --name ruby

やってることは、

1. $PIDFILEの中のprocessidをとってくる(ここでは、$PIDとしよう)
2. /proc/$PID/statが存在するか確認する
  - 存在しない場合は、終わり
  - 存在する場合は、3へ
3. killする
4. 2に戻る

をひたすら繰り返しております。  
なんで、基本的にはkillと同じ動きなのかな。retry付きで。  

#### 2回目: start-stop-daemon --stop --quiet --oknodo --retry=TERM/30/KILL/5 --exec $DAEMON

こっちは、まじkillallだったわ。  

1. /proc/<全ProcessのPID>/exe のsymbolic link先 == $DAEMON か確認
2. 同じだったprocessにkillでsignal送信
3. 同じのがなくなるまで1と2を繰り返す

killall。  
で、問題は、なんでこれを使う必要があるか・・・。  
きっと、本体殺しても、まだ生き残ってるプロセスがいる可能性があるから、  
なんだろうな・・・  

そこまでプロセスわけた上でやろうとすると、ちょいと今今時間がないので、  
一旦、各プロセスの本体を殺すinit scriptを書いてみたです。  
普通にkillコマンドで書いてます。start-stop-daemon、まだ使いこなせまてん。  

送るsignalは、[ここ](http://docs.fluentd.org/articles/signals)に書いてあったので、INT/TERM。  
今回はINTでお送りいたします。  

/etc/init.d/td-agentの、do_stopを下記に書き換えてもらって、それコピーして、  
/etc/init.d/td-agent_nginxつくってもらえれば、それぞれに殺すことができまっせ。  

```
#
# Function that stops the daemon/service
#
do_stop()
{
    # Return
    #   0 if daemon has been stopped
    #   1 if daemon was already stopped
    #   2 if daemon could not be stopped
    #   other if a failure occurred
    PID=`cat $PIDFILE`
    kill -INT $PID
    RETVAL="$?"
    if [ $RETVAL -ne 0 ]; then
        RETVAL="2"
    fi

    ps aux | grep $PIDFILE >/dev/null 2>&1
    RETVAL="$?"
    if [ $RETVAL -eq 0 ]; then
        rm -f $PIDFILE
        return "$RETVAL"
    fi

    return "2"
}
```

### 終わりに
---
これでなんか問題出たら、他のプロセスも殺せるように改変しようかな。  
(ご利用は各自の責任においてお願いします。Use at your own risk.)  

たぶん、改変自体は、ループ回して、待つ作戦＋ps 結果をgrepしてプロセス毎の  
PID取得する作戦かなー。  

ま、一旦はこれである程度まではいけるので、ブログはここまでで。  

  
<a href="http://c.af.moshimo.com/af/c/click?a_id=442315&p_id=170&pc_id=185&pl_id=4157&guid=ON" target="_blank"><img src="http://image.moshimo.com/af-img/0068/000000004157.gif" width="300" height="250" style="border:none;"></a><img src="http://i.af.moshimo.com/af/i/impression?a_id=442315&p_id=170&pc_id=185&pl_id=4157" width="1" height="1" style="border:none;">
