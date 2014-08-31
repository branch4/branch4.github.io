---
layout: post
title: Ansible入門
date: 2014-08-26 00:06:30 +0900
comments: true
published: false
author: xengineer
categories: 
---

予定より少し遅れて Ansible エントリでございます。[@xengineer01](https://twitter.com/xengineer01)です。  

## Ansible とは？
---
Ansible は、configuration management tool というやつですわ。
continuous delivery/continuous deployment とかに用いられますよ。
chef/puppet/capistrano的な。

さて、私昔、chefを使おうとして挫折した組です。理由は沢山あるんですが、
まぁぶっちゃけ難しくてわからなかったですw

Ansibleの何がいいって、[techracho](http://techracho.bpsinc.jp/yamasita-taisuke/2014_05_29/17567)さんもいってらっさいますが、シンプルそうなんですよ。

なので触ってみます、勝つまでは。

## ざっくりagenda

1. Ansible の使い方
  - 簡単に紹介
2. Ansible の認証
  - remote server の認証方法
3. Ad-hoc mode
  - 実際使ってみる

こんな感じ？

<!-- more -->  

## システム構成
---
今回は、vagrant 使って、下記構成を作って使ってみまっせ。

```
                            +------------+
                            |            |
                        +---+ apserver01 |
    +-----------+       |   |            |
    |           |       |   +------------+
    | ansible01 +-------+
    |           |       |   +-----------+
    +-----------+       |   |           |
                        +---+ fluentd01 |
                            |           |
                            +-----------+
```

上のシステムで、ansible01 から２台のサーバに、deploy したり、
configuration managementしたり、ってのをやってみようかな、と。

## Vagrantfile config
---
まず、ベースのところは、[前回のエントリ](http://blog.branch4.pw/blog/2014/08/11/setup-test-environment-with-vagrant2/)で書いた Vagrantfile から丸パクリ。

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.provision :shell, path: "bootstrap/all.sh"

  not_dbs = { :ansible01 => '192.168.101.101',
              :apserver01 => '192.168.102.101',
              :fluentd01  => '192.168.102.111'
            }

  not_dbs.each do |not_db_name, not_db_ip|
    config.vm.define not_db_name do |server_config|
      bootstrap_path = "bootstrap/#{not_db_name}.sh"
      server_config.vm.box = "hashicorp/precise64"
      server_config.vm.hostname = not_db_name.to_s
      server_config.vm.network "private_network", ip: not_db_ip
      server_config.vm.provision :shell, path: bootstrap_path

      server_config.vm.provider :virtualbox do |v|
        v.customize ["modifyvm", :id, "--memory", "384"]
        v.customize ["modifyvm", :id, "--cpus", "1"]
      end
    end
  end
end
```

読めばわかる。
もしわからなかったら[こちら](http://blog.branch4.pw/blog/2014/08/11/setup-test-environment-with-vagrant2/)へ。

bootstrap/ansible01.sh は下記。
```
#!/usr/bin/env bash

apt-get install -y git

git clone git://github.com/ansible/ansible.git
cd ./ansible
source ./hacking/env-setup

easy_install pip
pip install paramiko PyYAML jinja2 httplib2

/vagrant/bootstrap/not_db.sh
```

## Ansible について
---
[Ansible](http://www.ansible.com/home) に関しては、[公式ドキュメント](http://docs.ansible.com/)を読み進める形でエントリも書いていきます。
でも、それだと公式ドキュメントの日本語訳にしかならないので、
今回の構成にあった感じとか、わかりやすくしたり、僕だったら実践こんな感じで使うのかなー、
ってのをまじえつついく予定。

## Ansible の使い方
---
まず、Ansible の特徴を、下記イメージ図にて。


書いてある通りの特徴がありますよ。
僕は、chef/puppet なんかに比べて比較的簡単に理解できそうなのでやってみようと思い立ちました。

認証については、今回は、public key 配布して、password なしで通す方向でやります。
きっとこの方法が一番需要が多いと思われますゆえ・・・

Ansible は、ドキュメントを読むと、大きく２通りの使い方があるみたい。
違いも込みであげると、

- Ad-hoc mode
  - リモートサーバ上でコマンドを実行するモード。
    コマンド自体は記録されないので、
    何度も反復はしない場合の使い方
- Playbook
  - Configuration Management Systemとしての使い方。
    定義したグループのサーバに対して、継続的に設定を反映したり、
    デプロイしたりする用途

今回のエントリは、Ad-hoc mode を使いながら紹介していきます。
それで、Ad-hoc mode で使う分にはこの辺を知ってればいいかな・・・という
イメージ図が下記。

これにさっきのイメージ図の認証があれば、大体全体像は理解できる。と思う。
それぞれを簡単に説明すると・・・

- Inventory
- Ansible configuration
- Modules




<script type="text/javascript" language="javascript">
  num = Math.floor( Math.random() * 6 );
  document.write( aff[ num ]);
</script>
