---
layout: post
title: Ansible入門 - Ad-hoc modeを使ってみる -
date: 2014-09-09 01:00:00 +0900
comments: true
published: true
author: xengineer
description:  "Provisioning tool, AnsibleのAd-hoc modeの使い方について"
keywords: "provisioning, ansible, linux, configuration management tool, continuous delivery, continuous deployment, chef, puppet, capistrano"
categories: 
  - Infra
  - Web Development
  - Provisioning
---

予定より少し遅れて Ansible エントリでございます。[@xengineer01](https://twitter.com/xengineer01)です。  

![ansible logo](http://blog.branch4.pw/images/2014/09/ANSI_logotype_web.png)  
![ansible badge](http://blog.branch4.pw/images/2014/09/ansible_badge.png)  

## Ansible とは？
---
Ansible は、configuration management tool というやつですわ。  
continuous delivery/continuous deployment とかに用いられますよ。chef/puppet/capistrano的な。

さて、私昔、chefを使おうとして挫折した組です。理由は沢山あるんですが、
まぁぶっちゃけ難しくてわからなかったですw

Ansibleの何がいいって、[techracho](http://techracho.bpsinc.jp/yamasita-taisuke/2014_05_29/17567)さんもいってらっさいますが、シンプルそうなんですよ。

なので触ってみます、勝つまでは。

## ざっくりagenda

1. システム構成
    * 今回の検証環境のシステム構成紹介
1. Ansible について
    * 簡単に紹介
1. Ansible の使い方
    * 簡単に紹介
2. Ansible 全体像
    * remote server の認証方法
3. Ad-hoc mode で使ってみる
    * 実際使ってみる

こんな感じ？

<!-- more -->  

## 写真紹介

まずは、唐突ながら、友人の写真紹介です(笑)
いつもいい写真撮るんですよね。

![nabechan kamiiso](http://blog.branch4.pw/images/2014/09/nabechan02_kamiiso.jpg)  
※ Hiroyuki Watanabeの写真で、[http://my-eyes.net/](http://my-eyes.net/)に元があります。

では本題に・・・

## システム構成
---
今回は、vagrant 使って、下記構成を作って使ってみました。
念のため最初に断っておくと、今回はまだ、ansible のad-hoc mode を試してみよう、
程度なので、vagrant + ansible の連携とか、vagrantの設定まで踏み込んでいくつもりはなく、
いつでも検証自体を再現できるように使ってるだけです。

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

### 環境について
- OS
  - Ubuntu 12.04 LTS \n \l
- ansible
  - ansible 1.8 (devel b6a30a7331) last updated 2014/08/27 14:24:50 (GMT +000)
- Python
  - Python 2.7.3
- OpenSSH/OpenSSL
  - OpenSSH_5.9p1 Debian-5ubuntu1, OpenSSL 1.0.1 14 Mar 2012

## Vagrantfile config
---
まず、ベースのところは、[前回のエントリ](http://blog.branch4.pw/blog/2014/08/11/setup-test-environment-with-vagrant2/)で書いた Vagrantfile からほぼ丸パクリ。

```
$ cat /vagrant/Vagrantfile
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
      bootstrap_path_user = "bootstrap/#{not_db_name}_user.sh"
      server_config.vm.box = "hashicorp/precise64"
      server_config.vm.hostname = not_db_name.to_s
      server_config.vm.network "private_network", ip: not_db_ip
      server_config.vm.provision :shell, path: bootstrap_path
      server_config.vm.provision :shell, path: bootstrap_path_user, privileged: false

      server_config.vm.provider :virtualbox do |v|
        v.customize ["modifyvm", :id, "--memory", "384"]
        v.customize ["modifyvm", :id, "--cpus", "1"]
      end
    end
  end
end
```

読めばわかる。大きく変えてるのは、provisioningのshell実行するとこで、rootで実行するパターンと、
通常ユーザで実行するパターンを分けてることかな。
もしわからなかったら[こちら](http://blog.branch4.pw/blog/2014/08/11/setup-test-environment-with-vagrant2/)へ。

bootstrap/ansible01.sh は下記。
```
$ cat /vagrant/bootstrap/ansible01.sh
#!/usr/bin/env bash

apt-get install -y git
apt-get install -y python-setuptools
apt-get install -y python-dev
apt-get install -y libyaml-dev

git clone git://github.com/ansible/ansible.git
cd ./ansible
source ./hacking/env-setup

echo "source ~/ansible/hacking/env-setup" >> ~/.bashrc

easy_install pip
pip install paramiko PyYAML jinja2 httplib2

mkdir -p /etc/ansible
cat <<EOF > /etc/ansible/hosts
[ansible]
192.168.101.101

[web]
192.168.102.101

[fluentd]
192.168.102.111
EOF
```

やってることは、ansibleのインストールとInventoryの設定。  
ansibleインストールのための必須条件は、下記。

- &gt;= Python 2.6(3系はNG)

管理される側のサーバは、

- &gt;= Python 2.4(3系はNG)
  - Python 2.5 < の場合は、python-simplejson も必須

Inventoryについては後述。

bootstrap/ansible01_user.shの内容はこれ。
```
$ cat /vagrant/bootstrap/ansible01_user.sh
#!/usr/bin/env bash

ssh-keygen -t rsa -f ~/.ssh/id_rsa -N ''

cp ~/.ssh/id_rsa.pub /vagrant/ansible01_publickey
echo "source ~/ansible/hacking/env-setup" >> ~/.bashrc
```

公開鍵と秘密鍵生成して、公開鍵を、共有ディレクトリに突っ込んで、
あとは、ansibleの環境変数をsourceするように.bashrcに書いてる、と。  

公開鍵の格納場所は、このあとの管理ノードがそこから読んで、authorized_keyに設定できるようにしてるわけです。
なんかもっとスマートな方法ないかなー、と思ったんだけど、とりあえずテスト用環境だからいっか。

次は、bootstrap/apserver01.sh。
```
$ cat /vagrant/bootstrap/apserver01.sh
#!/usr/bin/env bash

/vagrant/bootstrap/apserver.sh
/vagrant/bootstrap/not_db.sh
```

なんもしとらん！w  
apserver.shとnot_db.sh呼んでるだけなので、まずは、not_db.shから。

```
$ cat /vagrant/bootstrap/not_db.sh
#!/usr/bin/env bash

apt-get install -y curl
curl -L http://toolbelt.treasuredata.com/sh/install-ubuntu-precise.sh | sh

mkdir /var/log/td-agent/tmp/
```

ファイル名から察するに、DBサーバじゃなかったら、curlインストールするのと、fluentdインストールしてんだね。
んで、apserver.sh。

```
$ cat /vagrant/bootstrap/apserver.sh
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
```

apserverには、apacheインストールして、fluentdの細かい設定しております。
今回の、ansibleのエントリには毛ほども関係ない設定です。消してもいいです。

次は、bootstrap/apserver01_user.sh。

```
$ cat /vagrant/bootstrap/apserver01_user.sh
#!/usr/bin/env bash

mkdir -p ~/.ssh/
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
cat /vagrant/ansible01_publickey >> ~/.ssh/authorized_keys
```
ここで、公開鍵関連の設定をしております。  
なんか突然、/vagrant/ansible01_publickeyをauthorized_keysに入れてます。
Vagrantでは、/vagrantファイルに、Vagrantfileが存在してるディレクトリがマウントされるからっすね。
これで、上のほうで、ansible01.shがやってたことが理解できる、と。

では最後に、bootstrap/fluentd01.shと、bootstrap/fluentd01_user.shの中身を貼付けてだけおきます。
解説なし。

```
$ cat /vagrant/bootstrap/fluentd01.sh
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

$ cat /vagrant/bootstrap/fluentd01_user.sh
#!/usr/bin/env bash

mkdir -p ~/.ssh/
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
cat /vagrant/ansible01_publickey >> ~/.ssh/authorized_keys
```

あ、あと、bootstrap/all.shがあった。
これも解説なし。

```
$ cat /vagrant/bootstrap/all.sh
#!/usr/bin/env bash

apt-get update
apt-get install -y vim
apt-get install -y vim-common
```

この辺全部ぶっ込んで、vagrant upしたらうまくいくと思う。たぶん。

## Ansible について
---
[Ansible](http://www.ansible.com/home) に関しては、[公式ドキュメント](http://docs.ansible.com/)を読み進める形でエントリも書いていきます。
でも、それだと公式ドキュメントの日本語訳にしかならないので、
今回の構成にあった感じとか、わかりやすくしたり、僕だったら実践こんな感じで使うのかなー、
ってのをまじえつついく予定。

## Ansible の使い方
---
まず、Ansible の簡単な特徴を、下記イメージ図にて。

![ansible](http://blog.branch4.pw/images/2014/09/ansible_rserver_communication_image.png)  

図に書いてある感じなのかなー、と感じております。
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

## Ansible 全体像
それで、Ad-hoc mode で使う分にはこの辺を知ってればいいかな・・・という
イメージ図が下記。

![ad-hoc mode](http://blog.branch4.pw/images/2014/09/ansible_ad_hoc_image.png)  

これにさっきのイメージ図の認証があれば、大体全体像は理解できる。と思う。
認証の設定は、[先日書いたブログ](http://blog.branch4.pw/blog/2014/09/06/public-key-authentication/)を参照くだされ。
それぞれを簡単に説明すると・・・

- Inventory
  - 管理対象サーバをここで設定する
  - ノードの設定だけじゃなくて、グルーピング機能があったり、色々便利
- Ansible configuration
  - ansible全体の挙動の設定
    - 例えば、inventoryの設定ファイルの場所
    - 例えば、毎回パスワード確認のプロンプトを出すか
- Modules
  - 実際に実行する設定/deploy などが実装されてる単位
  - 例えば、commandを実行する場合は、Commands Modules
  - 例えば、file転送する場合は、Files Modules的な感じで、Module単位で実装されている

コマンド打つと、この辺の設定を読み込んで実行してくれる、という流れ。  

## Ad-hoc mode で使ってみる
まず、なんか動いてるのをみてテンションあげないとやってられないので、コマンド実行してみよう。  

ansible01にログインして今回登録した全サーバにpingを打ってみる。
vagrant関連のconfigで事前の設定は全部してあるから大丈夫。通ります。たぶん。

```
$ vagrant ssh ansible01
$ ansible all -m ping
192.168.101.101 | FAILED => SSH encountered an unknown error during the connection. We recommend you re-run the command using -vvvv, which will enable SSH debugging output to help diagnose the issue
192.168.102.101 | success >> {
    "changed": false,
    "ping": "pong"
}

192.168.102.111 | success >> {
    "changed": false,
    "ping": "pong"
}
```

あ、通ってない・・・w  
ansibleサーバ自身には、公開鍵登録してないからだわ。なので、登録。
(あんまり自分自身にコマンド打つ用途はないので、普段はやらないと思うけど)

```
$ cat .ssh/id_rsa.pub >> .ssh/authorized_keys
$ cat .ssh/authorized_keys #<- 確認してね
```

再度実行！

```
$ ansible all -m ping
192.168.101.101 | success >> {
    "changed": false,
    "ping": "pong"
}

192.168.102.111 | success >> {
    "changed": false,
    "ping": "pong"
}

192.168.102.101 | success >> {
    "changed": false,
    "ping": "pong"
}
```

おー。うまくいった。よしよし。では諸設定の説明をば。

### Inventory
<a name="inventory"></a>
全体像のイメージ図に出て来た３つの要素の一つ、Inventoryについて。

- Inventory
  - 管理対象サーバをここで設定する
  - ノードの設定だけじゃなくて、グルーピング機能があったり、色々便利

って上で書いてた。

まず、Inventoryファイルは、デフォルトの、/etc/ansible/hosts に以下のように作成します。
(ファイルの場所変更するには、Ansible Configuration に設定する。後述しますわ)

```
$ cat /etc/ansible/hosts
[ansible]
192.168.101.101

[web]
192.168.102.101

[fluentd]
192.168.102.111
```

四角い括弧のなかが、グループ名で、その下に書いてあるIPアドレスがそのグループに所属するホストになります。  
ホスト名解決はできないぽ。

apserver01 192.168.102.101  

みたいに書いたら、名前解決できんのかと思ったら、できまてん。なので、名前解決は、
いまのところの僕の知識だと、外だしソリューションしないといけないのかな。めんどくせ。
ただ、Dynamic Inventoryの項目がドキュメントにあったので、その辺深堀すればそれっぽいのがありそうな予感。  

Inventoryには、グルーピング以外にも、変数渡す機能とかもあるけど、
このエントリではここまでに留めて、別途Inventory単体でエントリ書く予定。
壮大になりすぎます。

ad-hoc mode で static Inventoryファイルを使ってコマンド送るときのベースはこんな感じで、
管理対象サーバ/グループの増減に従って、Inventoryファイルを編集しましょうね、な感じです。

### Ansible Configuration
２つ目の、Ansible Configurationについて。  

ansible全体の挙動を決定する設定ファイルのこと。
version1.6以降では、下記の優先順位で評価されます。

- ANSIBLE_CONFIG (環境変数)
- ansible.cfg (under current directory)
- ~/.ansible.cfg (under home directory)
- /etc/ansible/ansible.cfg

version1.5以前では、下記の優先順位。環境変数とcurrent directoryが入れ替わったのね。

- ansible.cfg (under current directory)
- ANSIBLE_CONFIG (環境変数)
- ~/.ansible.cfg (under home directory)
- /etc/ansible/ansible.cfg

ただこれも次のバージョンでは変わってるかもしれないので、[ここ](http://docs.ansible.com/intro_configuration.html#the-ansible-configuration-file)を正とするのがよーし。  

#### 何を設定できるの？
全体の挙動って何よ？ってなるので、例を下記に。  

項目|デフォルト値|備考
:---|:---|:---
hostfile       | /etc/ansible/hosts | Inventoryファイルの場所
library        | /usr/share/ansible | Moduleの格納場所
remote_tmp     | $HOME/.ansible/tmp | ansibleは、module丸ごとremoteに転送して実行されるので、remoteでの格納先
forks          | 5                  | remoteに同時接続する上限。デフォルトはかなり保守的
poll_interval  | 15                 | 非同期処理への終了確認ポーリング間隔の設定
sudo_user      | root               | playbookに sudo_userの指定がない場合のデフォルト設定
ask_sudo_pass  | False              | sudo時に、password確認のプロンプトをするか
ask_pass       | False              | password確認のプロンプトをするか
transport      | smart              | 転送に何を使うか。smart:ssh/local:localhost?/chroot:?/jail:?。基本ssh
remote_port    | 22                 | remote hostで使うポート番号。Inventoryファイルでホスト毎に上書きできる
module_lang    | C                  | 各モジュール間のやりとりで使う言語

こんな感じで、ほんとうに全体の挙動に関する設定ですわ。大体の設定は、
システムのデフォルト値をここで決めて、個別に、Playbook/Inventoryファイルなんかで決めるイメージ。
他にも項目が沢山あるので、それは別途エントリを作りますかね。  

そして設定項目全部みたいときは、[こちら！](https://github.com/ansible/ansible/blob/devel/examples/ansible.cfg)で、
大元をみたほうがはやいっす。もちろんgithubは最新なので、ansibleのバージョンによっては
使えないのも入ってると思うので、その辺は調べてみてくだされ。  

### Modules
最後の、Modules について。  

ここが、ansibleの実質上のコアなのかな。
ad-hoc modeでは、実行するmoduleを、コマンドラインから指定して、
playbookでは、playbookの中で設定して、各moduleをremoteサーバに転送した後、
各サーバで実行する方式です。  

色んなmoduleがあるので、実行したい内容に合わせて選択する形になります。
自作もできるようなので、ない場合は自分で作りましょう。  

本エントリは、ad-hoc modeのエントリなので、ad-hoc modeでどうやってmoduleを指定するかを下記に。

```
$ ansible <targets> -m <module> -a "<arguments to the module>"
```

まんまだわ。  

#### targets
[Inventory](#inventory)で書いてる何かを指定します。  

- サーバ単体を指定する場合
  - 192.168.102.101
- グループを指定する場合
  - web
- 全部を指定する場合
  - all

#### module
moduleはね、もう一杯あって、どれを使うのかわかんなくなるね。

例として、/tmp/test っていうファイルを、apserver01に作成して、permissionを777に変更するコマンドを実行してみよう。

##### まずは、apserver01 にファイルが存在しないことを確認。

ansible01で下記を実行する。

```
$ ansible 192.168.102.101 -m command -a "ls /tmp"
192.168.102.101 | success | rc=0 >>
vagrant-shell
```

うむ。vagrant-shellってファイルしかないので次に進もう。

##### 使うmodule 選択とざっくり流れ
今回は、ファイル作るので・・・command module で普通にtouch実行して、
そのあとに、file module でpermission変更します。

各moduleについては、下記参照。

- [Command module](http://docs.ansible.com/command_module.html)
  - 引数に、実行するコマンドを渡すと実行してくれる
  - pipe とか、＆はだめ、対応してない
  - pipe とか使う場合は、[shell module](http://docs.ansible.com/shell_module.html)を使う
- [File module](http://docs.ansible.com/file_module.html)
  - File/Directory/Symbolic link等を作成する
  - File/Directory等のowner設定変更
  - File/Directory等のgroup設定変更
  - File/Directory等のpermission設定変更

あ、見事にFile moduleでファイルも作成できる・・・でも今回はcommand で！w

##### apserver01上に、ファイル作成

下記をansible01上で実行してみる。

```
$ ansible 192.168.102.101 -m command -a "touch /tmp/test"
192.168.102.101 | success | rc=0 >>

$ ansible 192.168.102.101 -m command -a "ls -l /tmp"
192.168.102.101 | success | rc=0 >>
total 4
-rw-rw-r-- 1 vagrant vagrant   0 Sep  7 06:07 test
-rwx--x--x 1 vagrant vagrant 159 Aug 30 14:21 vagrant-shell
```

お、できている。スーパー！！！
で、permissionは、664。  

次は、permisssion変更します。

##### ファイルのpermission変更

644だったpermissionを777に変更してやります。
permission変更は、file moduleを利用するわけですが、
[公式ドキュメント](http://docs.ansible.com/file_module.html)曰く(ちょっと省略してるけど)、

parameter | required  | default | choices | comments
:---      | :---      | :---    | :---    | :---
force     |  no       | no      | yes     | ２つのケースについて、symlinkの作成を強制する。ひとつは、source fileが存在しない場合(linkを張っておいて、あとからファイル自体は作成する)。もうひとつは、destination が存在していて、かつファイルである場合。(この場合、pathに指定しているファイルを削除して、symlinkを作成する)
          |           |         | no      |
group     |  no       |         |         |
mode      |  no       |         |         | ファイル/ディレクトリのpermission設定。
owner     |  no       |         |         | ファイル/ディレクトリのowner設定。
path      |  yes      |         |         | 管理対象ファイルのpath。file module内で唯一必須項目。 Aliases: dest, name
recurse   |  no       | no      | yes     | ファイルの設定を再起的に実施するかどうか。(state=directoryにのみ適用する) (Ansible 1.1以降実装)
          |           |         | no      |
src       |  no       |         |         | state=linkにのみ適用する。linkするファイルのpathを指定。絶対path/相対path/存在しないpathを指定可能です。相対pathは展開されません。
state     |  no       | file    | file    | fileを指定して、srcが存在しない場合、ファイルは作成されません。作成したい場合は、copy/template moduleを使いましょう。はて？fileって必要なの？？
          |           |         | link    | linkを指定した場合、symbolick linkが作成されるか、変更される。
          |           |         | directory | directoryを指定した場合, 存在しない場合、直接の全サブディレクトリが作成されます。version1.7以降では、modeで指定したpermissionに従って作成される。
          |           |         | hard    | hardlinkを作成する場合は、hardを指定する。
          |           |         | touch   | touch(version1.4以降)を指定した場合、Linux コマンドのtouchと同じ挙動をする。
          |           |         | absent  | absentを指定した場合、ディレクトリは再起的に削除されて、ファイルとsymlinkは削除される。

この表からわかる通り、file moduleで必須なオプションは、"path" のみ。
pathで指定されたファイル/ディレクトリ等に対して、何をするかを、他のオプションで指定する寸法。
今回は、permissionを777に変更したいので、"mode=0777" を指定して、実行してみる。

```
$ ansible 192.168.102.101 -m file -a "path=/tmp/test mode=0777"
192.168.102.101 | success >> {
    "changed": true,
    "gid": 1000,
    "group": "vagrant",
    "mode": "0777",
    "owner": "vagrant",
    "path": "/tmp/test",
    "size": 0,
    "state": "file",
    "uid": 1000
}

$ ansible 192.168.102.101 -m command -a "ls -l /tmp"
192.168.102.101 | success | rc=0 >>
total 4
-rwxrwxrwx 1 vagrant vagrant   0 Sep  7 06:07 test
-rwx--x--x 1 vagrant vagrant 159 Aug 30 14:21 vagrant-shell
```

おーーーー、変わった変わった。ちゃんちゃん。
無事に、testファイルのpermissionが777になっておりました。
よかったよかった。  

というわけで、ansibleは、大量のサーバに任意のコマンドをばこばこ発行したり、色々できる
便利なツールでした。おしまい。




<script type="text/javascript" language="javascript">
  num = Math.floor( Math.random() * 6 );
  document.write( aff[ num ]);
</script>
