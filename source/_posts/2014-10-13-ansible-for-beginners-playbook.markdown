---
layout: post
title: "Ansible入門 - Playbookを使ってみる -"
date: 2014-10-13 5:00:00 +0900
comments: true
published: true
author: xengineer
description:  "Provisioning tool, Ansible の Playbook の使い方について"
keywords: "provisioning, ansible, linux, configuration management tool, continuous delivery, continuous deployment, chef, puppet, capistrano"
categories: 
  - Infra
  - Web Development
  - Provisioning
---

またしても Ansible エントリでございます。[@xengineer01](https://twitter.com/xengineer01)です。  

![ansible logo](http://blog.branch4.pw/images/2014/10/ansible_wordlogo_black.png)  

ansible の基本的なとことか、ansible の ad-hoc mode については、
[以前の記事](http://blog.branch4.pw/blog/2014/09/09/ansible-for-beginners/)
をみていただきまして・・・  

今日は、ansible で playbook 使ってってみようかね。  
include とか、vars とか、role とか沢山便利機能があるけど、  
今回は playbook を使う！以上のことはしませぬ。  
なぜなら僕は ansible 初級。  

## ざっくりagenda

1. システム構成
    * 今回の検証環境のシステム構成紹介
2. Ansible playbookについて
    * 簡単に紹介
3. やってること詳細
    * 今回やったことの詳細

こんな感じ。

<!-- more -->  

## 写真紹介

まずは、いつも通り唐突ながら、なべちゃんの写真紹介。  
いつもいい写真撮るんですよね。今回は秩父にある名もない滝だそうです。

![nabechan chihibu](http://blog.branch4.pw/images/2014/10/nabechan03_chichibu.jpeg)  
※ Hiroyuki Watanabeの写真で、[http://my-eyes.net/](http://my-eyes.net/)に元があります。

では本題に・・・

## システム構成
---
今回も、vagrant 使って、下記構成を作って使ってみました。
今回も、vagrant + ansible の連携とか、vagrantの設定まで踏み込んでいくつもりはなく、
いつでも検証自体を再現できるように使ってるだけです。  
あ、あと、packageのインストール的なのは今回はansibleではなくvagrantでやってます。
この辺は後述するけど、思想の問題。

![wordpress](http://blog.branch4.pw/images/2014/10/ansible_wordpress_system.png)  

ざっくり、こんなシステム作りまっせ。
デフォ設定んとこはなんも書いてなし。MySQLも、言語設定追記してる程度なので
構成図上はなんも記載してないす。

WordPressは、Wordpressを、git clone/git checkout refs/tags/xxxxして
nginxのrootに格納するのと、 pluginのインストールまで。

### 環境について
```
$ cat /etc/issue
Ubuntu 12.04 LTS \n \l
$ ansible-playbook --version
ansible-playbook 1.7.2
```

## Ansible playbookについて
[以前の記事](http://blog.branch4.pw/blog/2014/09/09/ansible-for-beginners/)
では、ad-hoc mode つって、対象サーバ上で実行したいことを一つずつコマンドとして
実行していく形式を紹介しましたよ、と。  

でもそれだと明らかに毎回沢山打ち込まないといけなくて面倒なので、
設定ファイルに起こして実行できるようにしましょう。としてるとこの
設定ファイルの部分が playbook ですよ。  
なので、対象サーバに対して実施したいことを書いておくファイルのことです。

Playbookを使った場合の ansible の挙動のざっくりイメージ図を以下に！

![ansible-playbook image](http://blog.branch4.pw/images/2014/10/ansible_playbook_image.png)  

ad-hoc mode で書いたときとほとんど変わっていないのは内緒。
Modules が Playbook に、ansible が ansible-playbook になっただけです。
しかも、実際のところは Modules も呼び出されるから、
なんかちょっと違うんだけど。なんとなくなイメージだから。

## やってること詳細
### 各種install/configurationの分類
前述したけど、packageのインストールにはvagrant使ってるよ、ということもあるので、
何をvagrantでやってて、なにをansibleでやってんだい、ってのを書いておきましょ。

- Vagrant でやってること
  - Update package repositories
  - Install many packages
  - Create user/group
    - ansible/ansible
  - Set authorized_keys
  - Execute ansible
- ansible でやってること
  - nginxの設定
  - php/php-fpmの設定
  - MySQLの設定
  - MySQLのユーザ作成
  - Wordpress の checkout とインストール
  - Wordpress plugin の インストール
  - Wordpressの設定
  - nginx/php-fpm/mysql の再起動

つまり、package のインストールとか、ansible の設定関係は全部 vagrant でやってて、
Wordpress の実行環境に必要な設定系は全部 ansibleでやってるよ、と。  

packageのインストールは時間かかるし、頭に一回だけ走らせればいいので、
vagrant 側でお世話しようと思った次第で。ansible とか chef 的なものは、
何度も何度も実行すると思うので、

- 何かのインストールみたいな時間がかかる処理
- 何かの設定みたいな時間がかからない処理
- ユーザ作成的な単発の処理

などなど種別を考えて、どこに記述すんのか分けたほうがええべよ。
(僕はそうしたいだけ)

今回は、とりあえずさっき書いた感じにわけてみた。
なので、ここから ansible の playbook だけ引っこ抜いて実行しても
全然 Wordpress は動きません。

### 各種ansisble設定解説
----------

まずは、vagrant 関連のファイルを git clone しておきます。
さすがにペタペタ貼って動作確認するの面倒な量になってきたので github にあげたわい。

```
$ git clone https://github.com/xengineer/ansible_playbooks.git
```

これ clone して、vagrant up すると、きっとすぐに Wordpress 使えるようになるはず。
でもそれは重要ではなく、今回は解説が重要である。

なので、以下諸々解説。なお、解説は全部 ansible の解説となっております。
vagrant の解説もしはじめるとながーくなるので、vagrant については、

[ここ](http://blog.branch4.pw/blog/2014/08/07/setup-test-environment-with-vagrant/)とか、
[ここ](http://blog.branch4.pw/blog/2014/08/11/setup-test-environment-with-vagrant2/)を
参照くださいませ。

vagrant up後、wordpress02 っていうインスタンスができてるので、
そこに vagrant ssh して、仮想インスタンスの中と合わせて
下記を眺めてもらえるとわかりやすいよ。
(ファイルのパスとかは全部仮想インスタンスにあわせて書いてるので)

### config 格納場所
ansible 自体の config は、/etc/ansible 配下にまとめて格納してあるので、
tree するとこんな感じ。
(tree はインストールしてないので、sudo apt-get install tree してちょ)

```
$ tree  /etc/ansible/
/etc/ansible/
|-- ansible.cfg
|-- hosts
`-- playbooks
    `-- wordpress
        `-- wordpress-playbook.yml
```

各ファイルを説明すると、

- ansible.cfg
  - ansible 自体の挙動の設定
  - [以前の記事](http://blog.branch4.pw/blog/2014/09/09/ansible-for-beginners/)参照
- hosts
  - 上のほうのざっくりイメージ図上でいうところの、Inventory file
  - ようは設定対象サーバを指定するファイル
  - [以前の記事](http://blog.branch4.pw/blog/2014/09/09/ansible-for-beginners/)参照
- wordpress-playbook.yml
  - 今回作った playbook ファイル
  - 今回メインで説明するやつ
  - ファイル名はなんでもいいよ

という感じ。

### wordpress-playbook.yml

さて、今回作ったのはこれ一つ。本来は、色んな便利機能があって、もっとわかりやすく、
簡単に書けたりもするんだけど、それについてのお勉強は次回以降ってことで。
今回は愚直にひたすら書いてみるのね。

勝海舟さんもこういっております。  

**"事を成し遂げる者は、愚直であれ、才走ってはならない。"**

問題ない。持ってたらもう走っている。  
菜箸くらいは台所にあるけどね。

で、下記が今回書いた playbook。どん！

```
$ cat /etc/ansible/playbooks/wordpress/wordpress-playbook.yml
- name: Configure/install applications needed to run wordpress(nginx/php/php-fpm/mysql/wordpress)
  hosts: web
  sudo: yes
  tasks:
  - name: Setup nginx.conf file
    template: src=/vagrant/conf/nginx/nginx.conf
              dest=/etc/nginx/nginx.conf
  - name: Setup fastcgi_params file
    template: src=/vagrant/conf/nginx/fastcgi_params
              dest=/etc/nginx/fastcgi_params
  - name: Setup nginx virtualhost config file for wordpress
    template: src=/vagrant/conf/nginx/sites-available/wordpress
              dest=/etc/nginx/sites-available/wordpress
  - name: remove default nginx site
    file:     path=/etc/nginx/sites-enabled/default
              state=absent
  - name: remove default php-fpm site
    file:     path=/etc/php5/fpm/pool.d/www.conf
              state=absent
  - name: Setup nginx virtualhost symbolic link for wordpress
    file:     src=/etc/nginx/sites-available/wordpress
              dest=/etc/nginx/sites-enabled/wordpress
              state=link
  - name: Create nginx root directory for wordpress
    file:     path=/var/www/paralympics
              state=directory
  - name: Setup php-fpm.conf file
    template: src=/vagrant/conf/php5/fpm/php-fpm.conf
              dest=/etc/php5/fpm/php-fpm.conf
  - name: Setup php.ini
    template: src=/vagrant/conf/php5/fpm/php.ini
              dest=/etc/php5/fpm/php.ini
  - name: Setup fpm pool config file for wordpress
    template: src=/vagrant/conf/php5/fpm/pool.d/wordpress.conf
              dest=/etc/php5/fpm/pool.d/wordpress.conf
  - name: Create php5-fpm socket file directory
    file:     path=/var/run/php5-fpm/
              state=directory owner=www-data group=www-data
  - name: Create php5-fpm log directory
    file:     path=/var/log/php5-fpm/log/
              state=directory

- name: Configure database server
  hosts: db
  sudo: yes
  tasks:
  - name: Setup mysql
    template: src=/vagrant/conf/mysql/my.cnf
              dest=/etc/mysql/my.cnf
  - name: Add user 'wp-user' to mysql
    mysql_user: name=wp-user password=""

- name: Configure wordpress
  hosts: wordpress
  sudo: yes
  handlers:
    - name: stop apache2
      service: name=apache2 state=stopped
    - name: restart nginx
      service: name=nginx state=restarted
    - name: restart php5-fpm
      service: name=php5-fpm state=restarted
    - name: restart mysql
      service: name=mysql state=restarted
  tasks:
  - name: Create wordpress config directory
    file: path=/etc/wordpress/ state=directory
  - name: Setup wordpress
    git: repo=git://github.com/WordPress/WordPress.git
         dest=~/downloads/
         version=4.0
         accept_hostkey=yes
    sudo: no
  - name: Download wordpress plugin staticpress
    git: repo=https://github.com/megumiteam/staticpress
         dest=~/downloads/wp-content/plugins/staticpress
         accept_hostkey=yes
    sudo: no
  - name: Download wordpress plugin The Event Calendar
    get_url: url=https://downloads.wordpress.org/plugin/the-events-calendar.3.8.zip dest=/tmp/
  - name: Unzip the-events-calendar plugin
    shell: unzip -o /tmp/the-events-calendar.3.8.zip -d ~/downloads/wp-content/plugins/
    sudo: no
  - name: Fetch random salts for Wordpress config
    local_action: command curl https://api.wordpress.org/secret-key/1.1/salt/
    register: "wp_salt"
    sudo: no
  - name: Create Wordpress database
    mysql_db: name=mywp_db state=present
  - name: Create Wordpress database user
    mysql_user: name=wp-user password=""
                priv=*.*:SELECT,INSERT,UPDATE,DELETE,CREATE,ALTER,DROP,INDEX
                host='localhost' state=present
  - name: Copy Wordpress config file
    template: src=/vagrant/conf/wp-config.php dest=/etc/wordpress/
  - name: Install wordpress files to nginx root directory for wordpress
    synchronize: src=~/downloads/
                 dest=/var/www/paralympics/
                 recursive=yes
  - name: Change ownership of Wordpress installation
    file: path=/var/www/paralympics/ owner=www-data group=www-data state=directory recurse=yes
    notify:
      - stop apache2
      - restart mysql
      - restart nginx
      - restart php5-fpm
```

な、ながい・・・ほんとにこれを解説するのか・・・いいえ、しません。
ポイントだけね。

- フォーマット
  - yaml で書きましょう
  - あとは推して図りましょう
- handlers と notify だけちょっとわからない気がする
  - handlers は task と同じなんだけど、特別な task と思うのがいい
    - 何が特別って、notify からしか呼び出せない
  - つまり、notify を、どっかの task(task A とする)の中に書いておいて、
    handlers の name に書いてある task を呼び出すと、
    その task A が終了したときに handlers の task を呼び出せる(daemon 再起動とかに便利)
- なんかよくわからないところで説明してないところがある気がする
  - 僕もよくわからないからだと思います

モジュールの詳細については、ansible の公式ドキュメントをみたほうが圧倒的にいいので、
[ここ](http://docs.ansible.com/modules_by_category.html)参照。

その他、この Playbook で使ってるモジュールの説明は以下につらつらと。
みててわかんないとこはコメントなり twitter なり自分で考えるなりしてちょ。

### config file のテンプレをコピーするだけの時はどうしているか
#### template モジュールを使ってます
```
- name: Setup nginx.conf file
  template: src=/vagrant/conf/nginx/nginx.conf
            dest=/etc/nginx/nginx.conf
```
こんな感じに。  
src  : テンプレファイルを指定  
dest : テンプレのコピー先を指定  

**dest**だよ。  
僕はずっと、**dst**って書いてて、
動かない現象に勝手に悩まされていました。

### 既存ファイルを削除するときはどうしているか
#### file モジュールを使ってます
```
- name: remove default nginx site
  file:     path=/etc/nginx/sites-enabled/default
            state=absent
```

path  : 削除したいファイルのパスを指定  
state : absent って書いておけば消してくれる

### Symbolic link を作成するときはどうしているか
#### file モジュールを使ってます
```
- name: Setup nginx virtualhost symbolic link for wordpress
  file:     src=/etc/nginx/sites-available/wordpress
            dest=/etc/nginx/sites-enabled/wordpress
            state=link
```

↓ ↓ ↓ ↓ こういうシンボリックリンクになる。  
dest --> src 

### Directory を作成するときはどうしているか
#### file モジュールを使ってます
```
- name: Create nginx root directory for wordpress
  file:     path=/var/www/paralympics
            state=directory
```
path  : 作成したい Directory のパスを指定  
state : directory って書いておけば作ってくれる

### MySQL のユーザを作るときはどうしているか
#### mysql_user モジュールを使ってます
```
- name: Configure database server
  hosts: db
  sudo: yes
  tasks:
  - name: Setup mysql
    template: src=/vagrant/conf/mysql/my.cnf
              dest=/etc/mysql/my.cnf
  - name: Add user 'wp-user' to mysql
    mysql_user: name=wp-user password=""
```

上記で作って、もう一カ所、下記でも作っている・・・
これはたぶん上のほうを消すの忘れただけなので、上はなくても動くと思う。たぶん。
みたまんま、wp-userを、パスなしで、色々grantして作ってます。

```
- name: Create Wordpress database user
  mysql_user: name=wp-user password=""
              priv=*.*:SELECT,INSERT,UPDATE,DELETE,CREATE,ALTER,DROP,INDEX
```

### MySQL のtable 操作をしたい場合はどうしているか
#### mysql_db モジュールを使ってます
```
- name: Create Wordpress database
  mysql_db: name=mywp_db state=present
```

name  : 作りたい Database 名  
state : present で作る  

### git 操作したい場合はどうしているか
#### git モジュールを使ってます
```
- name: Download wordpress plugin staticpress
  git: repo=https://github.com/megumiteam/staticpress
       dest=~/downloads/wp-content/plugins/staticpress
       accept_hostkey=yes
  sudo: no
- name: Setup wordpress
  git: repo=git://github.com/WordPress/WordPress.git
       dest=~/downloads/
       version=4.0
       accept_hostkey=yes
  sudo: no
```

repo : clone したいレポジトリ  
dest : どこに clone するか  
accept_hostkey : yes で サーバ認証を無視してくれます(sshのStrictHostKeyChecking off と同じ)  
sudo : no にすると、実行時は一般ユーザで  
version : 指定してる場合はそのタグを checkout  

### ファイルを download する場合はどうしているか
#### get_url を使ってます
```
- name: Download wordpress plugin The Event Calendar
  get_url: url=https://downloads.wordpress.org/plugin/the-events-calendar.3.8.zip dest=/tmp/
```

get_url : ダウンロードしたい URL を指定する  
dest    : download 先を指定する

### download したファイルを unzip したい場合はどうしているか
#### shell モジュールを使ってます
```
- name: Unzip the-events-calendar plugin
  shell: unzip -o /tmp/the-events-calendar.3.8.zip -d ~/downloads/wp-content/plugins/
  sudo: no
```

一見、unarchive モジュールで出来るように見えるんだけど、ローカルファイルを、
リモートに送信してから展開するモジュールなので、ちょっと違う。
今回は、shell モジュールで直接 unzip コマンドを実行しましたよ。

### ファイルを remote サーバ内でコピーする場合はどうしているか
#### synchronize モジュールを使ってます
```
- name: Install wordpress files to nginx root directory for wordpress
  synchronize: src=~/downloads/
               dest=/var/www/paralympics/
               recursive=yes
```

src       : 送信するファイル or ディレクトリ  
dest      : 送信先のパス  
recursive : yes にすると、再起的にディレクトリを送ってくれる

### file/directory の owner を変更する場合はどうしているか
#### file モジュールを使ってます
```
- name: Change ownership of Wordpress installation
  file: path=/var/www/paralympics/ owner=www-data group=www-data state=directory recurse=yes
```

path : 対象パス
owner : owner 誰にするか
group : どの group にするか
state : directory にすると、directory の設定をする？のかな？
recurse : yes を設定すると、再起的にowner/group を設定してくれる

ざっくり全体感はこんな感じ。
何も説明してない感満載だけど、playbook 書いて実行するだけなら、
そんなに勉強することもない、ってことな気がする。role とか vars を使うときに
少し説明したり、勉強したり、がでてくるんじゃないかな。

そんなわけで、その辺は次回のお楽しみ！

## その他
### 今回一番大変だったこと
まぁいつもそうだけど、動くところまで大変でした。

#### ansible で ImportError って出てる場合は、実は、ansible のログ出力先に permission がない場合もある
/var/log/ansible.log にしてて、permission、めんどくさくて 777にしちゃった　

#### ansible で ImportError って出てる場合は、必要なpackageがインストールされていない場合もある
動作確認するのに、何度もvagrant destroy/vagrant upを繰り返したくて、
一旦 apt-get update を削除したわけです。
そしたら、なんか急に yaml 関連のエラーが出てきて大変でした。
出てくるエラーが、ログ出力先のpermission問題の時と似てるので、
「あれ？なんかpermissionのところいじったっけ？？？」ってなって、泥沼。

しばらくしてから、ふと、そういえば一旦 update 止めよう、
ってくらいから変な気がする・・・と思って戻したら直ったわけで。
update しないとインストールできないpackageもあるってことですな・・・

こんな感じに結構簡単に使える ansible です。
ただ、playbook は、こんな雑に書いてるると、実運用ではカオスすぎて
使い物にならないと思うので、次回は、include/role/vars辺りについて
書いて、もっとスマート運用を目指します。  

でもあれなんだよな、僕なんも運用するもの持ってないんだよな・・・

![ansible badge](http://blog.branch4.pw/images/2014/10/ansible_circleA_red.png)  

  
<script type="text/javascript" language="javascript">
  num = Math.floor( Math.random() * 6 );
  document.write( aff[ num ]);
</script>
