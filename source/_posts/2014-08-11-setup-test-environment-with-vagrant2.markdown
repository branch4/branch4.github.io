---
layout: post
title: fluentdのテスト環境をvagrantでセットアップしてみるべ(複数サーバ管理)
date: 2014-08-11 2:00:00 +0900
comments: true
published: false
author: xengineer01
categories: 
  - Infra
  - Web Development
  - Virtualization
---

こんにちは。前回の続きで、またVagrantのエントリ書いてる[@xengineer01](https://twitter.com/xengineer01)です。  

![vagrant logo](http://blog.branch4.pw/images/2014/08/logo_vagrant.png)

## 前回のあらすじ
[前回の記事](http://blog.branch4.pw/blog/2014/08/07/setup-test-environment-with-vagrant/)
は、下記あたりの流れをざっくり紹介しました。

- Vagrantってなんですか？
- Vagrantのinstall
- Vagrantfileの書き方
- vagrantでソフトウェア関連設定
- vagrantでサーバのハードウェアスペック指定

前回は、サーバ１台の場合についてだったんで、今回は複数台管理する場合について。  
この記事、本来の目的は、fluentdの検証をすることなんで、当然複数サーバあげたいわけで。

## 概要
---
今回はたぶん、  

1 Projectで複数サーバを管理する場合の、

- Vagrantfileの書き方
- networkについて
- provisioningについて
- vagrant commandの使い方

こんなところになる予定。

## 各種バージョン
---
前回も書いたけど、このとおり。  
今回は fluentdのバージョンも追記しておこう。

- vagrant version 1.6.3
- Windows PC + VirtualBox 4.3.8r92456
- MintLinux17 + VirtualBox 4.3 ?
- MacOSX(Mavericks) + VirtualBox 4.3.14r95030

## システム構成図
---
Appサーバのapacheのログを fluentd(HA) に投げて、postgres insertくらいを一旦の目処にしようかね。  
最終的には、HTTP(RESTなのかは不明)で別apに投げるのと、S3でのバックアップ、くらいまでをやるか、
と思ってはいるけどどこまで書けるやら・・・。

今回の簡易システム構成図はこれ。

![multi server01](https://blog.branch4.pw/images/2014/08/vagrant-multi-server01.png)

<!-- more -->

## Vagrantfile複数サーバ対応
----------

### １台のときは？
何はなくとも予習と復習は大切です。  
なので、前回の復習から。単体のサーバの場合の Vagrantfile はこんな感じでした。

```
$ cat Vagrantfile
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "hashicorp/precise64"
  config.vm.hostname = "testserver"
  config.vm.network "public_network", bridge: 'en0: Wi-Fi (AirPort)'
  config.vm.provision :shell, path: "bootstrap.sh"

  config.vm.provider :virtualbox do |v|
    v.customize ["modifyvm", :id, "--memory", "512"]
    v.customize ["modifyvm", :id, "--cpus", "1"]
  end
end
```

ほむほむ。思い出しました。

### 複数台のときは？
では、引き続き複数の場合はどうすりゃいいんかね・・・ vagrant の公式 website を物色。  

[ここ](https://docs.vagrantup.com/v2/multi-machine/index.html)ですよ。ドンピシャ。
ではこれを参考に、システム構成図ぽく書いてみると・・・  

```
$ cat Vagrantfile
# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.define "ap_server01" do |ap_server01|
    ap_server01.vm.box = "hashicorp/precise64"
    ap_server01.vm.hostname = "apserver01"
    ap_server01.vm.network "public_network", bridge: 'en0: Wi-Fi (AirPort)'
    ap_server01.vm.provision :shell, path: "bootstrap.sh"

    ap_server01.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--memory", "384"]
      v.customize ["modifyvm", :id, "--cpus", "1"]
    end
  end

  config.vm.define "ap_server02" do |ap_server02|
    ap_server02.vm.box = "hashicorp/precise64"
    ap_server02.vm.hostname = "apserver02"
    ap_server02.vm.network "public_network", bridge: 'en0: Wi-Fi (AirPort)'
    ap_server02.vm.provision :shell, path: "bootstrap.sh"

    ap_server02.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--memory", "384"]
      v.customize ["modifyvm", :id, "--cpus", "1"]
    end
  end

  config.vm.define "fluentd01" do |fluentd01|
    fluentd01.vm.box = "hashicorp/precise64"
    fluentd01.vm.hostname = "fluentd01"
    fluentd01.vm.network "public_network", bridge: 'en0: Wi-Fi (AirPort)'
    fluentd01.vm.provision :shell, path: "bootstrap.sh"

    fluentd01.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--memory", "384"]
      v.customize ["modifyvm", :id, "--cpus", "1"]
    end
  end

  config.vm.define "fluentd02" do |fluentd02|
    fluentd02.vm.box = "hashicorp/precise64"
    fluentd02.vm.hostname = "fluentd02"
    fluentd02.vm.network "public_network", bridge: 'en0: Wi-Fi (AirPort)'
    fluentd02.vm.provision :shell, path: "bootstrap.sh"

    fluentd02.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--memory", "384"]
      v.customize ["modifyvm", :id, "--cpus", "1"]
    end
  end

  config.vm.define "postgres" do |postgres|
    postgres.vm.box = "hashicorp/precise64"
    postgres.vm.hostname = "postgres"
    postgres.vm.network "public_network", bridge: 'en0: Wi-Fi (AirPort)'
    postgres.vm.provision :shell, path: "bootstrap.sh"

    postgres.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--memory", "384"]
      v.customize ["modifyvm", :id, "--cpus", "1"]
    end
  end
end
```

な、、、ながいすな・・・これはさすがに短くできる予感しかしないけども、
一旦起動するかどうか確認をば・・・

```
$ vagrant up
```

ってうつと・・・おお、、、起動した。結構時間かかった。
あと今更だけど、network "public_network"ってどういう IP 振られてるんだろう？  

ログインして確認しよう。ぬ、複数サーバあがってるときに、vagrant ssh つっても、
どのサーバに ssh するかわからんだろ・・・とりあえずやってみる。

```
$ vagrant ssh
This command requires a specific VM name to target in a multi-VM environment.
```

あ、、、やっぱり怒られるのね・・・。どうやら引数に、ssh したいサーバの識別子渡すそうです。  

config.vm.define "ap_server01" do |ap_server01| <- "ここの中の文字列"

```
$ vagrant ssh ap_server01
Welcome to Ubuntu 12.04 LTS (GNU/Linux 3.2.0-23-generic x86_64)

 * Documentation:  https://help.ubuntu.com/
Welcome to your Vagrant-built virtual machine.
Last login: Fri Sep 14 06:23:18 2012 from 10.0.2.2
vagrant@apserver01:~$ /sbin/ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:88:0c:a6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global eth0
    inet6 fe80::a00:27ff:fe88:ca6/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:01:26:5a brd ff:ff:ff:ff:ff:ff
    inet 172.16.17.10/24 brd 172.16.17.255 scope global eth1
    inet6 2408:27:bfff:28d:411a:c61a:be37:db7c/64 scope global temporary dynamic
       valid_lft 604272sec preferred_lft 85272sec
    inet6 2408:27:bfff:28d:a00:27ff:fe01:265a/64 scope global dynamic
       valid_lft 2591957sec preferred_lft 604757sec
    inet6 fe80::a00:27ff:fe01:265a/64 scope link
       valid_lft forever preferred_lft forever
vagrant@apserver01:~$ logout
Connection to 127.0.0.1 closed.
```

IP は、そうか、うちの自宅内の DHCP が振られてるのね。
検証環境には、完全 Private振りたいので、あとで変更しよっと。
その前に、Vagrantfile の簡略化をしてみよう。これは長過ぎる・・・  

### Vagrantfile 短く！
さて、どうしたものか、まずはぐぐってみる。  

_"vagrantfile 簡略化"_  

はそれっぽいのがヒットせず。

_"vagrantfile 複数"_  

で、[こんな記事](http://nmbr8.com/blog/2014/05/22/how-to-define-and-control-multiple-guest-machines-per-Vagrantfile/)が。
これだ！ruby で書けばいいからこう書けるのね。

```
$ cat Vagrantfile
# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.provision :shell, path: "bootstrap/all.sh"             #1

  not_dbs = { :apserver01 => '192.168.101.1',
              :apserver02 => '192.168.101.2',
              :fluentd01  => '192.168.102.1',
              :fluentd02  => '192.168.102.2'
            }                                                      #2

  not_dbs.each do |not_db_name, not_db_ip|                         #3
    config.vm.define not_db_name do |server_config|
      bootstrap_path = "bootstrap/#{not_db_name}.sh"
      server_config.vm.box = "hashicorp/precise64"
      server_config.vm.hostname = not_db_name.to_s
      server_config.vm.network "private_network", ip: not_db_ip    #4
      server_config.vm.provision :shell, path: bootstrap_path

      server_config.vm.provider :virtualbox do |v|
        v.customize ["modifyvm", :id, "--memory", "384"]
        v.customize ["modifyvm", :id, "--cpus", "1"]
      end
    end
  end

  config.vm.define "postgres01" do |postgres01|
    postgres01.vm.box = "hashicorp/precise64"
    postgres01.vm.hostname = "postgres01"
    postgres01.vm.network "private_network", ip: '192.168.103.1'
    postgres01.vm.provision :shell, path: "bootstrap/postgres01.sh"

    postgres01.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--memory", "384"]
      v.customize ["modifyvm", :id, "--cpus", "1"]
    end
  end
end
```

なんかまとまってる感あり！
行末に、コメントで番号付けてるとこを軽く説明しておこうかな。

1. config.vm.provision :shell, path: "bootstrap/all.sh"
  - あれだわ。全部のサーバで実行する provisioning の shell を指定してます
  - これとは別に、各サーバ毎に実行する shell もループ中で指定しております
2. not_dbs = { :apserver01 => '192.168.101.1', ... }
  - 今回は、DBサーバとそれ以外で事前処理が似てたので、それ以外を共通化してループにしてみた
3. not_dbs.each do |not_db_name, not_db_ip|
  - そのループ。中の処理はみればわかる
4. server_config.vm.network "private_network", ip: not_db_ip
  -  networkはね、今回はプライベートにしました。IP も指定で

結構簡略化できてると思うけどどうでしょう。
次は、provisioning 用の shellを覗いてみよう。  

shellは、apserver/fluentdは、複数台あるけど、今全部中身一緒なので、1台分だけみる。

#### 全部のサーバで実行されるやーつ
```
$ cat bootstrap/all.sh
#!/usr/bin/env bash

apt-get update
apt-get install -y vim
apt-get install -y vim-common
```

#### ap server でだけ実行されるやーつ
```
$ cat bootstrap/apserver01.sh
#!/usr/bin/env bash

/vagrant/bootstrap/apserver.sh
/vagrant/bootstrap/not_db.sh
```

#### fluentd でだけ実行されるやーつ
```
$ cat bootstrap/fluentd01.sh
#!/usr/bin/env bash

/vagrant/bootstrap/not_db.sh

cat << EOF > /etc/td-agent/td-agent.conf
# Input
<source>
  type forward
  port 24224
</source>

# Output
<match apache.**>
  type file
  path /var/log/td-agent/out_apachelog
  time_slice_format %Y%m%d
  time_slice_wait 10m
  time_format %Y%m%dT%H%M%S%z
  compress gzip
  utc
</match>
EOF
```

#### postgres でだけ実行されるやーつ
```
$ cat bootstrap/postgres01.sh
#!/usr/bin/env bash

apt-get install -y postgresql
```

大体見ればわかると思うので、一個だけ説明。

```
/vagrant/bootstrap/apserver.sh
/vagrant/bootstrap/not_db.sh
```
この二つだけ。

/vagrant っていうのは、Hostサーバでいうところの、PROJECT_ROOTのディレクトリなのです。  
つまり、  

PROJECT_ROOT/bootstrap/apserver.sh  
PROJECT_ROOT/bootstrap/not_db.sh

をそれぞれ実行しろよ、と。
今回でいうと、apserverが一番実行される shell が多いのかな。  

1. まず、全体で、all.shを実行
2. 次に、apserver01.sh に書いてある何かを実行(今回は実は中身ほとんどないんだけど)
3. で、bootstrap/apserver.sh と bootstrap/not_db.sh を実行

なるほど。こんな感じにすれば、共通化もできるのねん。一旦、簡略化はこんなもんかな。

apserver.sh と not_db.sh については、下記内容が書いてありますよ。  

```
$ cat bootstrap/apserver.sh
#!/usr/bin/env bash

apt-get install -y apache2
rm -rf /var/www
ln -fs /vagrant /var/www

chmod o+x /var/log/apache2/
chmod o+r /var/log/apache2/access.log

cat << EOF > /etc/td-agent/td-agent.conf
# tail input
<source>
  type tail
  path /var/log/apache2/access.log
  pos_file /var/log/td-agent/tmp/httpd-access.log.pos
  tag apache.access
  format apache2
</source>

# Log Forwarding
<match apache.**>
  type forward

  # primary host
  <server>
    host 192.168.102.1
    port 24224
  </server>
  # use secondary host
  <server>
    host 192.168.102.2
    port 24224
    standby
  </server>

  # use longer flush_interval to reduce CPU usage.
  # note that this is a trade-off against latency.
  flush_interval 60s
</match>
EOF
```

```
$ cat bootstrap/not_db.sh
#!/usr/bin/env bash

apt-get install -y curl
curl -L http://toolbelt.treasuredata.com/sh/install-ubuntu-precise.sh | sh

mkdir /var/log/td-agent/tmp/
chown td-agent. /var/log/td-agent/tmp
```

#### apserver.shの内容
見ればわかるけど、apache インストールと、fluentd の HA 設定してるくらい。  

#### not_db.shの内容
ほぼ fluentd インストールしてるだけ。  

### Network について
network は、public/private で書き方だけわかったからいいや。

#### public(DHCP)
config.vm.network "public_network", bridge: 'en0: Wi-Fi (AirPort)'

#### private(static)
config.vm.network "private_network", ip: 'IP address'

たぶん、private で DHCP 振りたい場合は、ip: 以降削ればいける。

### Provisioning について
shell でも、ここまで共通化したり、分けたりできれば結構楽にはいけると思うけど、
そもそも、shell でやってるのはいかがなものか、という意見は認める。なので次は、ansible やろうかな。  

### vagrant command について

まずは、リストアップ。  
- box
  - manages boxes: installation, removal, etc.
- connect
  - connect to a remotely shared Vagrant environment
- destroy
  - stops and deletes all traces of the vagrant machine
- global-status
  - outputs status Vagrant environments for this user
- halt
  - stops the vagrant machine
- help
  - shows the help for a subcommand
- init
  - initializes a new Vagrant environment by creating a Vagrantfile
- login
  - log in to Vagrant Cloud
- package
  - packages a running vagrant environment into a box
- plugin
  - manages plugins: install, uninstall, update, etc.
- provision
  - provisions the vagrant machine
- rdp
  - connects to machine via RDP
- reload
  - restarts vagrant machine, loads new Vagrantfile configuration
- resume
  - resume a suspended vagrant machine
- share
  - share your Vagrant environment with anyone in the world
- ssh
  - connects to machine via SSH
- ssh-config
  - outputs OpenSSH valid configuration to connect to the machine
- status
  - outputs status of the vagrant machine
- suspend
  - suspends the machine
- up
  - starts and provisions the vagrant environment
- version
  - prints current and latest Vagrant version

### おわりに
はい、postgresに突っ込むところまではいきませんでした。
なぜならpostgres使ったことなくてちょっとめんどくさくなったから。

なので、現状だと、apache の log を tail で引っ張ってきて、fluentd01/02にファイル出力する、 まで。
次はどこまでいけるかな・・・

## 次回予告
ansible 使った provisioning 。

<script type="text/javascript" language="javascript">
  num = Math.floor( Math.random() * 6 );
  document.write( aff[ num ]);
</script>
