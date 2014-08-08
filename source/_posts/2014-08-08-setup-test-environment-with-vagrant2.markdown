--- layout: post title: fluentdのテスト環境をvagrantでセットアップしてみるべ(複数サーバ管理) date: 2014-08-08 22:00:00 +0900
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
- vagrant commandの使い方
- networkについて
- provisioningについて

こんなところになる予定。

## 各種バージョン
---
前回も書いたけど、このとおり。  
今回は fluentdのバージョンも追記しておこう。

- vagrant version 1.6.3
- Windows PC + VirtualBox 4.3.8r92456
- MintLinux17 + VirtualBox 4.3 ?
- MacOSX(Mavericks) + VirtualBox 4.3 ?

## システム構成図
---
Appサーバのapacheのログを fluentd(HA) に投げて、postgres insertくらいを一旦の目処にしようかね。  
最終的には、HTTP(RESTなのかは不明)で別apに投げるのと、S3でのバックアップ、くらいまでをやるか、
と思ってはいるけどどこまで書けるやら・・・。

一応構成図はこれ。

![multi server01](https://blog.branch4.pw/images/2014/08/vagrant-multi-server01.png)

<!-- more -->

## Vagrantfile複数サーバ対応
----------

### １台のときは？
何はなくとも予習と復習は大切です。  
なので、前回の復習から。

```
$ cat Vagrantfile
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "hashicorp/precise64"
  config.vm.hostname = "testserver"
  config.vm.network "public_network", bridge: 'en0: Wi-Fi (AirPort)'
  config.vm.provision :shell, path: "bootstrap.sh"

  # ここから下を追記した
  config.vm.provider :virtualbox do |v|
    v.customize ["modifyvm", :id, "--memory", "512"]
    v.customize ["modifyvm", :id, "--cpus", "1"]
    v.customize ["modifyhd", "b5fc9c57-f008-4118-a03f-e535f25deea4", "--resize", "1024"]
  end
end
```

### Projectディレクトリと、Vagrantfile を作る！

下記コマンドを実行してね。  
(本Entryでは、以降Projectのルートディレクトリは、PROJECT_ROOTとします)
```
$ mkdir <PROJECT_ROOT>
$ cd <PROJECT_ROOT>
$ vagrant init
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.

```

やってることは、

1. Projectディレクトリを作成
2. Projectディレクトリに移動
3. Projectを初期化(Vagrantfileが生成される)

でございます。コマンドで生成された Vagrantfileから、コメントの行を消すと、、、

```
$ cat Vagrantfile

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "base"
end
```
こんな感じ。APIのバージョンは、"2" のようです。  
config.vm.box = "base"は、よくわからないけど、今はいいや。  
Vagrantfile は、git,svn等の、VCSにcommitすべきものらしいので、commit。  

### Box とは？

生成された、Vagrantfile中にも出てくるけど、Boxとは？  
Vagrant では、box っていうのが、ひとつのベースイメージになるんだと。  
仮想イメージの呼び方をVagrant風に言うと "Box" なのです。  

### Box のインストール
まずは、自分のマシンに、Box(仮想イメージ)を追加します。  
下記コマンドを、実行。  

```
$ vagrant box add hashicorp/precise64
==> box: Loading metadata for box 'hashicorp/precise64'
    box: URL: https://vagrantcloud.com/hashicorp/precise64
    This box can work with multiple providers! The providers that it
    can work with are listed below. Please review the list and choose
    the provider you will be working with.

    1) hyperv
    2) virtualbox
    3) vmware_fusion

    Enter your choice: 2

    ==> box: Adding box 'hashicorp/precise64' (v1.1.0) for provider: virtualbox
        box: Downloading: https://vagrantcloud.com/hashicorp/precise64/version/2/provider/virtualbox.box
        ==> box: Successfully added box 'hashicorp/precise64' (v1.1.0) for 'virtualbox'!
```

今回使用してる、hashicorp さん謹製の box、precise64 は、  

- hyperv
- virtualbox
- vmware_fusion

に対応してるみたいだけど、2番の virtualbox を選択。  
そうすると、下記ディレクトリ配下に、VirtualBox のイメージがダウンロードされたり、  
Vagrantfileのような諸情報が格納されます。(結構時間かかる)  

${HOME}/.vagrant.d  
${HOME}/.vagrant.d/boxes  
${HOME}/.vagrant.d/data  
${HOME}/.vagrant.d/gems  
${HOME}/.vagrant.d/rgloader  
${HOME}/.vagrant.d/tmp  

たぶん大事なのは、boxes 配下なのかな？きっとそうだろう。  
イメージのダウンロード元は、[ここ](https://vagrantcloud.com/)からみたい。  

初期化完了した状態で、

```
$ vagrant up
```
実行すると、今追加したほやほやのboxがすぐ起動。  
ただ、諸々未設定なので、一旦落として設定しましょ。

- ネットワーク設定
- ホスト名の設定
- 起動時にインストールするアプリがあるのかどうか

などなど、設定していきます。  
まずは、停止。  

```
$ vagrant destroy
```

### Vagrantfile設定
設定自体は、初期化時に生成された、Vagrantfileを編集していく。

Vagrantfile では、下記なんかを定義できる  

- 起動するマシンスペック
- インストールするアプリケーション
- どうやってアクセスするか

今回は、こんなマシンにしようかな。

![guestserver](http://blog.branch4.pw/images/2014/08/guestserver01.png)

- CPU x 1 個
- Memory 512 MB
- HDD 15 GB
- Ubuntu12.04
- Network(DHCP/public)
- hostname: testserver
- apache pre-install

[ここ](http://docs.vagrantup.com/v2/getting-started/index.html)とか、
[ここ](http://docs.vagrantup.com/v2/virtualbox/configuration.html)を参照して、書いていく。  
ハードのスペック関連は、VirtualBoxのAPI経由なので、[ここ](http://www.virtualbox.org/manual/ch08.html)
からやりたいことを探すんだーね。  
そして出来上がったVagrantfile。(まだスペック関連入れてない版)
```
$ cat Vagrantfile

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "hashicorp/precise64"
  config.vm.hostname = "testserver"
  config.vm.network "public_network"
  config.vm.provision :shell, path: "bootstrap.sh"
end
```
順番に説明。
#### config.vm.box = "hashicorp/precise64"
このBoxは、このイメージですよ！と、いうこと。

#### config.vm.hostname = "testserver"
Host名は、testserverですよ！と、いうこと。

#### config.vm.network "public_network"
Public、といっても、グローバルIPが必ず振られるわけではない。  
たぶん、下記環境だったらグローバルが来るんじゃないか？

- Network IF が 1 つ
- DHCP でグローバルが割り当てられる

VirtualBox では、NAT になる。

Network IF が複数ある場合は、こんな感じに指定するそうな。
```
config.vm.network "public_network", bridge: 'en1: Wi-Fi(AirPort)'
```

#### config.vm.provision :shell, path: bootstrap.sh
ゲストサーバ起動時に、PROJECT_ROOT/bootstrap.sh を実行しろ、ということ。  
なので、ここに、

```
#!/usr/bin/evn bash

apt-get update
apt-get install -y apache2
```

って書いておくと、起動時に、apache2 が入った状態になります。  
さて、準備は整ったはずなので、いざ起動！！

```
$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'hashicorp/precise64'...
==> default: Matching MAC address for NAT networking...
==> default: Checking if box 'hashicorp/precise64' is up to date...
==> default: Setting the name of the VM: project_blog_default_1407340802923_76317
==> default: Fixed port collision for 22 => 2222. Now on port 2204.
==> default: Clearing any previously set network interfaces...
==> default: Available bridged network interfaces:
1) en0: Wi-Fi (AirPort)
2) en1: Thunderbolt 1
3) en2: Thunderbolt 2
4) bridge0
5) p2p0

```
あ、NIC だけじゃなくて色々あるから指定しないとだめなんですね・・・  
一旦とめて、Vagrantfile を編集。

```
$ cat Vagrantfile

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "hashicorp/precise64"
  config.vm.hostname = "testserver"
  config.vm.network "public_network"
  config.vm.network "public_network", bridge: 'en0: Wi-Fi (AirPort)'
  config.vm.provision :shell, path: "bootstrap.sh"
end
```

気を取り直して、再度実行。

```
$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Checking if box 'hashicorp/precise64' is up to date...
==> default: Fixed port collision for 22 => 2222. Now on port 2205.
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
    default: Adapter 2: bridged
==> default: Forwarding ports...
    default: 22 => 2205 (adapter 1)
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2205
    default: SSH username: vagrant
    default: SSH auth method: private key
==> default: Machine booted and ready!
==> default: Checking for guest additions in VM...
    default: The guest additions on this VM do not match the installed version of
    default: VirtualBox! In most cases this is fine, but in rare cases it can
    default: prevent things such as shared folders from working properly. If you see
    default: shared folder errors, please make sure the guest additions within the
    default: virtual machine match the version of VirtualBox you have installed on
    default: your host and reload your VM.
    default:
    default: Guest Additions Version: 4.2.0
    default: VirtualBox Version: 4.3
==> default: Setting hostname...
==> default: Configuring and enabling network interfaces...
==> default: Mounting shared folders...
    default: /vagrant => /Users/nemoto_hideaki/work/vagrant/project_blog
==> default: Running provisioner: shell...
    default: Running: /var/folders/kn/k_t_9_cs0yjd5q44m9w8b8wh0000gn/T/vagrant-shell20140807-3962-19neoc2.sh
==> default: stdin: is not a tty
==> default: bash: /tmp/vagrant-shell: /usr/bin/evn: bad interpreter: No such file or directory
The following SSH command responded with a non-zero exit status.
Vagrant assumes that this means the command failed!

chmod +x /tmp/vagrant-shell && /tmp/vagrant-shell

Stdout from the command:



Stderr from the command:

stdin: is not a tty
bash: /tmp/vagrant-shell: /usr/bin/evn: bad interpreter: No such file or directory
```

はい、再度失敗orz  
なんだなんだ・・・bootstrap.shを確認確認・・・  

&#35;!/usr/bin/evn bash  
&#35;!/usr/bin/evn bash  
&#35;!/usr/bin/evn bash  
&#35;!/usr/bin/evn bash  
&#35;!/usr/bin/evn bash  

evn ね・・・修正いたしまして・・・
```
&#35;!/usr/bin/env bash

apt-get update
apt-get install -y apache2
```
修正して、再度実行。  

```
$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Checking if box 'hashicorp/precise64' is up to date...
==> default: VirtualBox VM is already running.
```
あ、さっき実行してるから実行中なのか。もしかして今回は bootstrap.sh が実行されてないかも。  
なので、下記コマンドどっちかで強制実行。  

```
$ vagrant reload --provision
$ vagrant provision
```

違いは、stdoutみる限り、たぶん・・・
#### $ vagrant reload --provision
こっちは一旦マシン再起動かけてからの強制実行。  

#### $ vagrant provision
こっちは起動したままbootstrap.shだけ強制実行。  

どっちか実行すると、ちゃんと apache がインストールされます。  

### ログイン！
次は実際にログインアクセスしますよと。
```
$ vagrant ssh
Welcome to Ubuntu 12.04 LTS (GNU/Linux 3.2.0-23-generic x86_64)

 * Documentation:  https://help.ubuntu.com/
Welcome to your Vagrant-built virtual machine.
Last login: Fri Sep 14 06:23:18 2012 from 10.0.2.2
vagrant@testserver:~$
```

おーいえす。ちゃんとホスト名も設定されてる。  

ちなみに起動時の標準出力じっくりみると色々わかるんだけど、起動時に、guestのssh port(22)と、  
hostの何番かを紐づけてくれてるので、そこにアクセスしてもいいのかも。  
```
Hideaki-no-MacBook-Pro:project_blog nemoto_hideaki$ vagrant reload --provision
==> default: Attempting graceful shutdown of VM...
==> default: Checking if box 'hashicorp/precise64' is up to date...
==> default: Clearing any previously set forwarded ports...
==> default: Fixed port collision for 22 => 2222. Now on port 2204.
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
    default: Adapter 2: bridged
==> default: Forwarding ports...
    default: 22 => 2204 (adapter 1) <--ここ！！★
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2204
    default: SSH username: vagrant
    default: SSH auth method: private key
==> default: Machine booted and ready!
==> default: Checking for guest additions in VM...
```

というわけで、localhostの 2204 番にアクセスしてみるも・・・
```
$ ssh -p 2204 vagrant@127.0.0.1
vagrant@127.0.0.1's password:
```
private key 設定すればよさげだけど・・・どの key だ？面倒なんでパス。  
つまり、こんな設定をして、vagrant up/vagrant ssh すればつながるよ！  
という話でした。

はっ！マシンスペック！変えるんだった。とりあえず、default のままのスペックは下記。  
```
$ cat /proc/cpuinfo | grep -E 'processor|model name'
processor: 0
model name: Intel(R) Core(TM) i5-4258U CPU @ 2.40GHz
processor: 1
model name: Intel(R) Core(TM) i5-4258U CPU @ 2.40GHz

$ free -m
             total       used       free     shared    buffers     cached
             Mem:           365        321         43          0         11        248
             -/+ buffers/cache:         61        303
             Swap:          767          0        767

$ df -h
Filesystem                  Size  Used Avail Use% Mounted on
/dev/mapper/precise64-root   79G  2.3G   73G   4% /
udev                        174M  4.0K  174M   1% /dev
tmpfs                        74M  288K   73M   1% /run
none                        5.0M     0  5.0M   0% /run/lock
none                        183M     0  183M   0% /run/shm
/dev/sda1                   228M   25M  192M  12% /boot
vagrant                     233G   92G  142G  40% /vagrant
```
CPU : 2個
Memory : 384MB
HDD : 80GB
(VirtualBox の GUI から引っ張って来てるスペック)

- CPU : 1個
- Memory 512MB
- HDD 15GB

やることは、

- CPUを一個に減らす
- Memoryを512MBに増やす
- HDDを15GBに減らす

[参考サイト１](http://docs.vagrantup.com/v2/virtualbox/configuration.html)  
[参考サイト２](http://www.virtualbox.org/manual/ch08.html)  
上記２サイトを見比べた結果・・・
```
$ cat Vagrantfile
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "hashicorp/precise64"
  config.vm.hostname = "testserver"
  config.vm.network "public_network", bridge: 'en0: Wi-Fi (AirPort)'
  config.vm.provision :shell, path: "bootstrap.sh"

  # ここから下を追記した
  config.vm.provider :virtualbox do |v|
    v.customize ["modifyvm", :id, "--memory", "512"]
    v.customize ["modifyvm", :id, "--cpus", "1"]
    v.customize ["modifyhd", "b5fc9c57-f008-4118-a03f-e535f25deea4", "--resize", "1024"]
  end
end
```
こう。こんな感じでいけるはず。  
modifyhd の横の、"b5fc9c57-f008-4118-a03f-e535f25deea4"は、VirtualBox イメージのイメージファイルの UUID。  
下記コマンド実行すればみれるです。
```
$ VBoxManage list -l vms
Name:            project_blog_default_1407343357247_93821
Groups:          /
Guest OS:        Ubuntu (64 bit)
UUID:            7819729d-541f-47b8-8607-ec50965f4901
...
(中略)
...
SATA Controller (0, 0): /Users/nemoto_hideaki/VirtualBox VMs/project_blog_default_1407343357247_93821/box-disk1.vmdk (UUID: b5fc9c57-f008-4118-a03f-e535f25deea4) <-- これ！！
...
(中略)
...
```

よし！実行！
```
$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
...
(中略)
...
A customization command failed:

["modifyhd", "b5fc9c57-f008-4118-a03f-e535f25deea4", "--resize", "1024"]

The following error was experienced:

#<Vagrant::Errors::VBoxManageError: There was an error while executing `VBoxManage`, a CLI used by Vagrant
for controlling VirtualBox. The command and stderr is shown below.

Command: ["modifyhd", "b5fc9c57-f008-4118-a03f-e535f25deea4", "--resize", "1024"]

Stderr: 0%...
Progress state: VBOX_E_NOT_SUPPORTED
VBoxManage: error: Resize hard disk operation for this format is not implemented yet!
>

Please fix this customization and try again.
```

はい、まただめ・・・orz  
VBoxManage: error: Resize hard disk operation for this format is not implemented yet!  
まだ実装してねーってよ。あきらめよう。  

HDD の件を削って実行して、スペック確認した結果。
```
vagrant@testserver:~$ cat /proc/cpuinfo | grep -E 'processor|model name'
processor   : 0
model name  : Intel(R) Core(TM) i5-4258U CPU @ 2.40GHz
vagrant@testserver:~$ free -m
             total       used       free     shared    buffers     cached
Mem:           491        338        153          0         15        260
-/+ buffers/cache:         61        429
Swap:          767          0        767
vagrant@testserver:~$ df -h
Filesystem                  Size  Used Avail Use% Mounted on
/dev/mapper/precise64-root   79G  2.3G   73G   4% /
udev                        237M  4.0K  237M   1% /dev
tmpfs                        99M  288K   99M   1% /run
none                        5.0M     0  5.0M   0% /run/lock
none                        246M     0  246M   0% /run/shm
/dev/sda1                   228M   25M  192M  12% /boot
vagrant                     233G   92G  142G  40% /vagrant
```

HDD リサイズとかは、たぶん Box 定義からいじる、とかなのかな？  
その辺の深追いはまた今度。  
まず今日の課題はクリアで。  

![guestserver_hddstay](http://blog.branch4.pw/images/2014/08/guestserver01_nohdd.png)

## 次回予告
複数サーバをぼこぼこあげるとき。

<script type="text/javascript" language="javascript">
  num = Math.floor( Math.random() * 6 );
  document.write( aff[ num ]);
</script>
