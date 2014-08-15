---
layout: post
title: DynamoDBの本番とテスト環境の分け方
date: 2014-08-15 01:30:00 +0900
comments: true
published: true
author: risterlab
categories: 
 - AWS
 - DynamoDB
---
  
はい、こんばんみー。最近カレーにますますハマってる[risterlab](http://diary.risterlab.com)です。  
南インドより北インドカレー派ってことが最近わかりました。  
もはやナンいらなくてカレーだけでも飲めます。うそです。チャパティ派です。  
  
戯言はその辺にして、今回はDynamoDBでの本番環境とテスト環境の分け方を。  
![test-and-production-mistaken](http://blog.branch4.pw/images/2014/08/test-and-production-mistaken.gif)  
  
うちではテスト環境としてDynamoが使えるかは下記２つの条件がありました。  
  
1. 同じテーブル名のものを複数作成できること  
1. テスト環境へのクエリが本番環境へ発行されてしまう危険性がない、またはとても低いこと
  
<!-- more --> 
  
### 同じテーブル名を複数つくるには  
----------
  
これがDynamoDBでは意外にやっかい。  
結論申しますと、同じテーブル名については、別リージョンという制限付きで可能。  
  
オンプレだと本番DBとテストDBは大体違う筐体、DB、スキーマにのってるからテーブル名は同じにできるんだけど  
DynamoDBの場合は  
１つのリージョンが１つのDBで、AWSのアカウントがスキーマ。  
というイメージだとわかりやすい。  
つまりAWSのあるアカウントでTokyoリージョンにDynamoDBのテーブルを作るということは  
ひとつのDBのあるスキーマ内にテーブルを作るということ。  
  
つまりもちろん同じテーブル名はだめですね。  
じゃあ別のアカウントにするかって、そんなにアカウント作りたくないし、それは現実的ではなし。  
一番簡単なのはリージョンを変えることでしょう。  
    
本番をもちろん東京に。テストは適当に一番安いVirginiaってのがよい。  
  
|AWSアカウントID | リージョン | テーブル | 環境 |  
|:------------:|:------:|:-----:|:------:|  
|432156781234   | TOKYO     | TBL_HOGEHOGE | 本番 |  
|432156781234   | VIRGINA   | TBL_HOGEHOGE | テスト |  
  
これでそれぞれのスループット数を設定すればOK  
  
### 本番とテストのアクセス制限   
----------
  
DynamoDBへのアクセスにはリソースネーム（ARN）を使います。  
たとえばこんなの。  
`arn:aws:dynamodb:us-east-1:432156781234:table/TBL_HOGEHOGE`  

これにはリージョンが含まれているから、アクセス先の設定ミスなど人為的なミスを除けば、間違える危険性は低い。  
とはいえ、その人為的ミスが起こっちゃう訳ですね。  
だからそもそもアクセス制限をしたいところですが、そこで使うのがIAM。  
  
IAMでアプリが使うユーザーをテスト用と本番用で作って  
そこでさっきのリソースネームを使ってアクセス制限をかけましょう。  
そうすれば、さらにACCESSKEYとSECRETKEYがそれぞれ違うので、  
本当に最初のKEYの設定さえ間違えなければ、アプリが間違えて本番用のARNで本番用のテーブルに繋ごうとしても、権限なしではじいてくれるというわけです。  
  
IAMで設定するDynamoDBのPolicyはこんな感じ。  
さっきの表の例でいくと  
#####テスト用のユーザーにつけるPolicy　　
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1399538126123",
      "Effect": "Allow",
      "Action": [
        "dynamodb:BatchGetItem",
        "dynamodb:BatchWriteItem",
        "dynamodb:CreateTable",
        "dynamodb:DeleteItem",
        "dynamodb:DescribeTable",
        "dynamodb:GetItem",
        "dynamodb:ListTables",
        "dynamodb:PutItem",
        "dynamodb:Query",
        "dynamodb:Scan",
        "dynamodb:UpdateItem",
        "dynamodb:UpdateTable"
      ],
      "Resource": [
        "arn:aws:dynamodb:us-east-1: 432156781234:table/TBL_HOGEHOGE"
      ]
    }
  ]
}
```
#####本番用のユーザーにつけるPolicy　　
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1399538126123",
      "Effect": "Allow",
      "Action": [
        "dynamodb:BatchGetItem",
        "dynamodb:BatchWriteItem",
        "dynamodb:CreateTable",
        "dynamodb:DeleteItem",
        "dynamodb:DescribeTable",
        "dynamodb:GetItem",
        "dynamodb:ListTables",
        "dynamodb:PutItem",
        "dynamodb:Query",
        "dynamodb:Scan",
        "dynamodb:UpdateItem",
        "dynamodb:UpdateTable"
      ],
      "Resource": [
        "arn:aws:dynamodb:ap-northeast-1: 432156781234:table/TBL_HOGEHOGE"
      ]
    }
  ]
}
```
  
### まとめ  
----------
  
DynamoDBでテスト環境と本番環境を作る時、  
下記のようにすればよし。  
  
1. 同じテーブル名を複数作成する  
別リージョンで同名のテーブルを作成することが可能。  
1. テスト環境へのクエリが本番環境へ発行されてしまわないようにする  
ARNだけの区別では不十分。  
ユーザーをわけてIAMでPolicyを設定しアクセス制限をすべし。  　
  
もちろん性能評価とか負荷試験には別リージョンのテスト環境は使ってはだめですよ。レイテンシー違いますからね。  
  
<script type="text/javascript" language="javascript">
  num = Math.floor( Math.random() * 6 );
  document.write( aff[ num ]);
</script>
  
