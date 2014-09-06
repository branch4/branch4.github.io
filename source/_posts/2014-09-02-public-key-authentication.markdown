---
layout: post
title: "公開鍵を用いたssh認証"
date: 2014-09-02 19:56:06 +0900
comments: true
published: false
author: xengineer
categories: 
---

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

## sshとは？
---
このエントリにおける ssh(Secure SHell)は、ssh client のことをいいます。
つまり、rlogin/rsh/telnetと同じような通信アプリケーションで、
リモートのサーバと通信することができるツールのことをさすことにします。

ssh と、telnet/rsh等を比較すると、、、

- よりセキュアな認証
- よりセキュアな通信(暗号化通信)

こんな機能が追加で提供されているので、、、まぁセキュアってこった。

本エントリでは、上記２点を実現するために利用されている方式の一つである、
公開鍵と秘密鍵について紹介します。

## 認証フロー
まずはパスワード認証方式のフロー。

![password authentication](http://blog.branch4.pw/images/2014/09/password_authentication.png)  

やっていることは、パスワードが正しいかの検証のみ。  

次に公開鍵認証方式のフロー(すごい簡易バージョン。実際に実装されているのとは違うけど、イメージはこんな感じ)。

![password authentication](http://blog.branch4.pw/images/2014/09/publickey_authentication.png)  

こっちはなんだか色々やっている。  

    |クライアント側|サーバ側
:---|:---|:---
事前準備|公開鍵をサーバに渡す|公開鍵をクライアントから受け取る
ログイン処理|ログインしたいです、という|
||ランダム文字列をクライアントの公開鍵で暗号化してクライアントに渡す
|受け取った暗号化文字列を秘密鍵で復号化してサーバに送り返す|
||クライアントから受け取った復号後文字列と、最初の文字列を突き合わせて、一致していれば認証する

ざっくりこんな感じ。(実際とはちょっと異なるイメージだけど、まぁわかりやすく特徴はあらわせてるはず)

## 暗号化フロー
暗号化については、さっきの公開鍵認証方式フローの、ランダム文字列部分が、任意のメッセージに変更に
なるだけで、それ以外はなんも変わらない感じ。

## なぜ公開鍵認証が安全？
公開鍵を盗まれたら終わりでは？と、思うんだけど、実は違うんだね。  


## 認証

### Lists
----------

1. list1
   - list1
   - list2
   - list3
   - list4
1. list2
1. list3
1. list4

### Table
----------

左揃え | 中央揃え | 右揃え
:----- | :------: | -----:
1111   | 2222     | 3333 
4444   | 5555     | 6666
7777   | 8888     | 9999  

<!-- more -->  

### Codes
----------
`$ command !!`

    $ ls -l
    $ ls -ltr
    $ sl -l
    $ sl -al

    param1 :
    説明  

### Links
************

[google test link](http://google.com "google")  
[google test link wo title](http://google.com)

### Images
************

![Root4Logo](http://root04.github.com/images/email.png)  
source/images配下にcommitしてpushした画像は、http://branch4.github.com/images/xxx.xxx  
でみれる。


### Emphasize
----------
*EMPHASIZE*  
_EMPHASIZE_  
**EMPHASIZE**  
__EMPHASIZE__  

### Quote
************
> a said a  
>> b said ba  
>>> c said ca  
  
<script type="text/javascript" language="javascript">
  num = Math.floor( Math.random() * 6 );
  document.write( aff[ num ]);
</script>
