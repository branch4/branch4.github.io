---
layout: post
title: "公開鍵を用いたssh認証 - ノンパスでログインしちゃうYo ! -"
date: 2014-09-06 21:00:00 +0900
comments: true
published: true
author: xengineer
categories: 
  - Infra
  - Web Development
---

こんにちは。公開鍵認証について興味を持った、[@xengineer01](https://twitter.com/xengineer01)です。  
なんの特徴も変哲もないブログですわ。  
まずはいい写真撮るなべちゃんの作品紹介から。

![Big Sky](http://blog.branch4.pw/images/2014/09/nabechan01_bigsky.jpg)  
※ Hiroyuki Watanabeの写真で、[http://my-eyes.net/](http://my-eyes.net/)に元があります。

## 概要
---
本エントリでは、公開鍵と秘密鍵を用いた認証方式について、さらっと紹介します。

認証と言っても色々ですな。

- 古き良きパスワード認証
- 詰めちゃったら終わりの指紋認証
- パスワードの一種だけど、OTP(One Time Password)
- 鍵認証

などなど他にもありますが、今回は、パスワード認証と、鍵認証(の中でも公開鍵/秘密鍵)について。
特に、sshの認証に使う場合の違いを絡めて紹介します。

<!-- more -->

## sshとは？
---
このエントリにおける ssh(Secure SHell)は、ssh client のことをいいます。
つまり、rlogin/rsh/telnetと同じような通信アプリケーションで、
リモートのサーバと通信することができるツールのことをさすことにします。

ssh と、telnet/rsh等を比較すると、、、

- よりセキュアな認証
- よりセキュアな通信(暗号化通信)

こんな機能が追加で提供されているので、、、まぁセキュアってこった。

本エントリでは、上記のうち、「セキュアな認証」を実現するために利用されている方式の一つである、
公開鍵と秘密鍵について紹介します。

## 認証フロー
まずはパスワード認証方式のフロー。

![password authentication](http://blog.branch4.pw/images/2014/09/password_authentication.png)  

やっていることは、パスワードが正しいかの検証のみ。  

次に公開鍵認証方式のフロー(すごい簡易バージョン。実際に実装されているのとは違うけど、イメージはこんな感じ)。

![password authentication](http://blog.branch4.pw/images/2014/09/publickey_authentication.png)  

こっちはなんだか色々やっている。  

フロー|クライアント側|サーバ側
:---|:---|:---
事前準備    |公開鍵を生成する    |
            |公開鍵をサーバに渡す|公開鍵をクライアントから受け取る
ログイン処理|サーバにアクセスする|
            |                    |ランダム文字列をクライアントの公開鍵で暗号化してクライアントに渡す
            |受け取った暗号化文字列を秘密鍵で復号化してサーバに送り返す|
            |                    |クライアントから受け取った復号後文字列と、最初の文字列を突き合わせて、一致していれば認証する

ざっくりこんな感じ。(実際とはちょっと異なるイメージだけど、まぁわかりやすく特徴はあらわせてるはず)

## なぜ公開鍵認証が安全？
公開鍵を盗まれたら終わりでは？と、思うんだけど、実はちがうんだね。  

この方式で重要なのは、下記２点。

- 公開鍵でかけた暗号化は、秘密鍵でしか復号化できない
- 秘密鍵でかけた暗号化は、公開鍵でしか復号化できない

つまり、公開鍵を入手しても、サーバにログインできるわけではないのです。
安全安心。

## 設定
ではでは、実際の設定やら鍵の生成やらをやります。
ざっくり、クライアント側とサーバ側でやることのリスト。

クライアント側

- ふたつの鍵(公開鍵と秘密鍵)を生成する
- 公開鍵をサーバに渡す
- アクセスする

サーバ側

- sshサーバの設定をする(公開鍵認証用設定)
- 公開鍵をクライアントから受け取って、設定する
- アクセスを待つ

以下詳細へと続く。

### ふたつの鍵の生成
では、公開鍵と秘密鍵の生成手順です。ふたつの鍵はふたつでひとつなので、忘れないようにしましょう。

- 環境：MacOSX 10.9.4
- ssh-keygen はバージョンの調べ方がわからん・・・

まず、クライアント側で鍵生成。以下コマンドを、お好みの暗号化方式を選択して実行します。

```
#RSA鍵を作成(ssh version2)
$ ssh-keygen -b 4096 -t rsa -C "comment"
# RSA1鍵を作成(ssh version1)
$ ssh-keygen -t rsa1 -C "comment"
#DSA鍵を作成(ssh version2)
$ ssh-keygen -t dsa -C "comment"
```

で、RSA/RSA1/DSAってなんやねん。ってなるわけですが、
それぞれちょっと調べてみた。

- RSA1
  - ssh version1 で使われていた暗号化方式
  - 強度はそんなにないぽい。当然今は使われていない
  - ぐぐってもあんまり詳細が出てこない
- [RSA](http://ja.wikipedia.org/wiki/RSA%E6%9A%97%E5%8F%B7)
  - 発明者３名の名字の頭文字を取って命名。(ron Rivest/adi Shamir/len Adleman)
  - wikiにある通り、768bitまでは解読されちゃってる
  - 使うなら2048bit以上にしよう
  - ssh version2 で使える
- [DSA](http://ja.wikipedia.org/wiki/Digital_Signature_Algorithm)
  - Digital Signature Algorithmの略
  - 同じ符合長なら、RSA方式と同程度の暗号化強度らしい
  - ssh version2 で使える
- [ECDSA](http://ja.wikipedia.org/wiki/%E6%A5%95%E5%86%86%E6%9B%B2%E7%B7%9ADSA)
  - あ、一個増えた。調べてるうちに新しいのが出て来た。そしてこれが最新らしい
  - DSAの亜種で、短い鍵で高い暗号化強度を担保できる
  - つまーり、計算量を少なく抑えられて、パフォーマンスもいい

RSA1は使うな。で、RSA vs DSA だと、暗号化強度云々より、パフォーマンスで選ぶみたい。
普通に使う分にはどっちでもいいレベルなので、本エントリでは、RSA を使いますわ。
しかも、OpenSSH で生成できる DSA は、1024bitまでらしいので。
でも個人的には、ECDSAに移行しようかな。

そんなわけで、RSA鍵を生成するコマンドを実行。

```
#RSA鍵を作成(ssh version2)
$ ssh-keygen -b 4096 -t rsa -C "comment"
```

僕は既に既存 key がデフォルトディレクトリにあるので、色々指定して実行しまっせ。
(以降、秘密鍵:rsa、公開鍵:rsa.pubになってるけど、-f オプションなしの場合は、id_rsa/id_rsa.pubに読み替えてちょ)

```
#RSA鍵を作成(ssh version2)
$ ssh-keygen -b 4096 -t rsa -C "Test key for blog" -f "~/.ssh2/rsa"
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
passphrase too short: have 4 bytes, need > 4
Saving the key failed: /Users/nemoto_hideaki/.ssh2/rsa.
```

あら、passphraseに、4文字しかいれなかったら、5文字以上！って怒られました。
ちなみに、passphrase は、パスワードみたいなもん。何も設定しなければ、パスワードを要求されないす。
そして、5文字未満なのに普通に生成されます。(謎)

一旦まともにpassphrase を設定して生成。

```
$ ssh-keygen -b 4096 -t rsa -C "Test key for blog" -f ~/.ssh2/rsa
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/nemoto_hideaki/.ssh2/rsa.
Your public key has been saved in /Users/nemoto_hideaki/.ssh2/rsa.pub.
The key fingerprint is:
a0:53:a2:0a:cd:c1:7b:05:2a:14:d6:eb:08:34:09:de Test key for blog
The key's randomart image is:
+--[ RSA 4096]----+
|o++ .            |
|+=.o .           |
|o.=Eo +          |
|.+ = = .         |
|o B +   S        |
|.o o .           |
|.                |
|                 |
|                 |
+-----------------+
```

鍵ができてるかみてみる。  
*** 秘密鍵は公開しちゃだめですよ。この鍵は使わないので公開してますが ***

```
$ ls -l .ssh2/
total 16
-rw-------  1 nemoto_hideaki  staff  3326  9  6 18:32 rsa
-rw-r--r--  1 nemoto_hideaki  staff   743  9  6 18:32 rsa.pub
```

rsaのほうが、秘密鍵。  
rsa.pubのほうが、公開鍵。  
めでたく鍵が生成できました〜。中身も覗いてみます。何書いてあるか全然わからんけどな。

```
$ cat .ssh2/rsa
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,860352EBF26B86E25C86C2AB88D4DF4F

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
-----END RSA PRIVATE KEY-----
$ cat .ssh2/rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC3STy6YZelz5z0zcQkxYx2QMu8X+SCMBlMgxGbmoORBFB18sckyo9vPAjBXyR6XfEViXRUHLo07u6S/FqaLuBSSlwi8GPs8cux8e+KnbQ1Cik6lL829O++rgEFyMRPEvM/bB/GjPsni4Y2aBstU4xA8D4ZpqaN5rgKG2U9ZsovjhVQsIWHa8a4/W/z2oOASqss0/p+LHOYzmjHLDwaGCqqJNqglcdJrrNYxEUZApS5RT8JrkW7874+KbghCdN5L6n27NNblsusP6HJQknTC0ufKE340J3dEUmxpf/vErN6P7bKNDFlzgVfJehLW62n+u1XCj5akHBrfMEKOaS4y90u/pALUoYWoAhIcGou7Xwc6EhtTYYg3gSWqDbk8vDE+A4JVDYl3RnsYptqRqmkJZL9RE1MT67+5yqY5bY74KjuohKeBgAptiEecLb3ABlhPJCvLX1zTOnHQaXa6cC4Ln8DH+2gnFdC3EgDr/sRHonLpAMqO8y/tGNc9digefdxCfr8N3/eX0UPXCyjaPwpN/DjC4MlWDFnqMQGfXUsxCHLl/n/MY/MThYYFUPDe0e8dV0D3oq0l4NoYu+aVEeBIc7/7g01WrQWLHoGYZSPlezCmtclClJzSzOeWk1ghYKVSfWflXBfHgdyNpL3vPyAVeArFg1UDi8MHCHNkZDfLkJHLw== Test key for blog
```

では次の手順へ。

### 公開鍵をサーバに渡す/公開鍵をクライアントから受け取って、設定する
では鍵をサーバに登録するところをやります。

#### クライアント側作業
クライアントでやるのは、さっき生成した公開鍵をサーバに転送するだけの簡単なお仕事。
今回は、scp で公開鍵を丸っと転送。しゅわっち！

```
$ scp .ssh2/rsa.pub <Server IP>:/tmp/
```

#### サーバ側作業

サーバ側では、下記をやるわけです。

- authorized_keys への公開鍵の登録
- authorized_keys のアクセス権限の確認/変更
- sshサーバの設定確認/変更/再起動

まずは、
##### authorized_keys への公開鍵の登録

```
$ cat /tmp/rsa.pub >> .ssh/authorized_keys
$ cat .ssh/authorized_keys # <- ちゃんと入ってるか確認
```

##### authorized_keys のアクセス権限の確認/変更

permission をしっかり設定しないと、他が正しくても永遠にログインできねーので、下記参照の上、ちゃんと設定します。

クライアント側

- .ssh/rsa : 600

```
$ chomod 600 .ssh/rsa
```

サーバ側

- authorized_keys : 600
- ~/.ssh ディレクトリ : 700
- /home/hogehoge ディレクトリ : 755

```
$ chomod 700 .ssh
$ chomod 600 .ssh/authorized_keys
$ chomod 755 ~
```

ようは、他人が鍵関連のファイルに書き込めない状態にしとけと。
これでOK。

次。

##### sshサーバの設定確認/変更/再起動

sshサーバの設定で、公開鍵認証OKにしないといかんのです。  
/etc/sshd_configを開いて、下記３行が入ってなければ追記します。

```
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeyFile   .ssh/authorized_keys
```

で、再起動して、クライアントから接続して、ログインできればOK !  
段々めんどくさくなって手抜きになってきたけど、なんとなく伝わるはず。
間違いなど、ご指摘いただけるとありがたいっす。

参考URL :  
  [reference 1](http://www-net.nifs.ac.jp/lnas/manual/man-sshrsa.html)  
  [reference 2](http://oshiete.goo.ne.jp/qa/8356280.html)  
  [reference 3](http://stackoverflow.com/questions/2841094/what-is-the-difference-between-dsa-and-rsa)  

<script type="text/javascript" language="javascript">
  num = Math.floor( Math.random() * 6 );
  document.write( aff[ num ]);
</script>
