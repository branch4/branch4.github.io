---
layout: post
title: "RSpecでの基本的な書き方（前編）"
date: 2014-07-30 20:54:43 +0900
comments: true
published: true
author: adorechic
categories:
  - Ruby
  - RSpec
  - Rails
---

こんにちは。adorechicです。
[前回](http://blog.branch4.pw/blog/2014/07/27/first-rspec-with-rails/) ざっくりRSpecの概要みたいなところで終わってしまったので、今回は基本的な書き方みたいなところを書いてみます。
とはいえPOROなサンプルコードへのテスト、っていうのはよく見かけるので、Railsのモデルをテーマにやってみます。

RSpecって結構独特な感じあって、RSpec Wayみたいなものがある。
割と好みわかれたりもするのだけれど、べたべたな感じの書き方からはじめていきます。

<!-- more -->
# ファイル
まあよくあるUserモデルに対するテストを書いてみます。
app/models/user.rb ってやつ。

これに対応するspecは、spec/models/user_spec.rb です。

Rails使ってるとgeneratorが作ってくれたりする。

実はこのディレクトリ構造みてspecのタイプが判定されたりするのですがRSpec3からデフォルトオフっぽいのであんまり気にしなくてよいかも。

# find_byメソッドをテストしてみる
実際はActiveRecordのメソッドなんでいちいちテストする必要ないですが、わかりやすいので。

```ruby
require 'spec_helper'

describe User do
  before do
    User.create!(
      email: 'tom@example.com',
      password: 'password',
      name: 'tom'
    )
  end

  specify '.find_by returns tom' do
    user = User.find_by(email: 'tom@example.com')
    user.email.should eq 'tom@example.com'
    expect(user.name).to eq 'tom'
  end
end
```

どうでしょうか。tomというユーザーを作成して、find_byメソッドでそのモデルインスタンスが取得できることをテストしています。なんとなく雰囲気はつかめるのでは。

## specify
specifyブロックがひとつのテストになっていて、RSpecではexampleと呼ばれます。
ちなみにitとspecifyは同じです。itと書くほうが多い。理由はだんだんわかってくるはず。
specifyの引数には、検証内容を書く。たまにshould returnとか書く人もいるけど、returnsと書くのが一般的です。

内容としては、User.createしてfind_byしたもののメアドと名前が意図したものか検証している。
この検証してるところをexpectationとか言ったりします。

あえてshouldとexpectの2パターン書いたのですが、意味合い的には同じです。
もともとはshouldでしたが、RSpec3からは非推奨になってexpectになっています。（ただし例外あり）
RSpec2の途中からexpectは使える。

## before
beforeブロックは名前のとおり、exampleが実行される前に実行されるブロックです。
このコードのように事前にテストデータ作ったりとかの前処理に使えます。

## describe
一番外側のdescribeは、そういうexampleをまとめたもの。beforeブロックは同じdescribeブロックでは全てに適用されます。describeの引数にUserクラスをわたしているけど、文字列でもOK。ただそのspecファイルが対象としているクラスを引数にしたdescribeを一番外側に置く場合が多い。

## 実行してみる
-fdオプションつけるのオススメです。

```
rspec -fd spec/models/user_spec.rb
```

するとこういう出力になるはず。

```
User
  .find_by returns tom

Finished in 0.08805 seconds
1 example, 0 failures
```

describeの引数とspecifyの引数のメッセージが表示されている。
これだけでなんとなく何のテストをしているかわかりますね。

## 寄り道：ここで作ったデータってどうなるの？残るの？
まあそのまんまにしてたら残りますよね。
でも前回のテストで作られたゴミが残ってると意図せずテストこけたりして辛い。
毎回クリーンな状態でやりたい。

ということで、都度データをきれいにしてくれるツールがあります。
[DatabaseCleaner](https://github.com/DatabaseCleaner/database_cleaner) とかが有名ですが、
[DatabaseRewinder](https://github.com/amatsuda/database_rewinder) が速いしオススメです。

# テストデータを簡単に用意する
とりあえずはテストデータ普通にcreateメソッドで作ったわけですが、これ毎回必要なパラメータ設定してcreateするとかダルいですね。ここでもpasswordとかなんでもよいのに指定している。

そういうテストデータを用意するツールもあります。
fixturesというのがもともと仕組みとしてあるのですが、FactoryGirlなどのFactory系が主流になり、ただ最近はfixturesが見直されていたりしますが、ここではFactoryGirlを使ってみます。

```ruby
describe User do
  before do
    create :user, email: 'tom@example.com'
  end

  specify '.find_by returns tom' do
    user = User.find_by(email: 'tom@example.com')
    expect(user.email).to eq 'tom@example.com'
  end
end
```

おーシンプルになった。（nameの検証はもういらんので消した）
普通にcreateメソッドで作るとpasswordとか指定しないとvalidation errorになるのに・・・

Factory系は、設計図みたいなものを別途定義しておきます。

- spec/factories/users.rb
```ruby
FactoryGirl.define do
  sequence :email do |n|
    "test-#{n}@example.com"
  end

  factory :user do
    email
    password 'password'
    name 'Steve'
  end
end
```

これにそって必要な値をいれてくれる。適当な値でいいときは

```ruby
create :user
```

これだけでOK。テストに必要なら別途指定する感じです。

# 共通で使う値をまとめる
すっきりしてくると重複してるのが目立ちますね。'tom@example.com'が3箇所で書かれている。
後から値変更するとき3箇所弄るの辛いですね。まとめましょう。

```ruby
describe User do
  before do
    create :user, email: email
  end

  let(:email) { 'tom@example.com' }

  specify '.find_by returns tom' do
    user = User.find_by(email: email)
    expect(user.email).to eq email
  end
end
```

letが使えます。遅延評価される変数的なものです。

# describeをネストする
beforeとかletとかいろいろ出てきて要素がちょっとゴチャゴチャしてきましたね。

いまexampleがひとつだからよいけど、別メソッドのexampleとか書きたいとき混じってわけわからなくなりそう・・・

describeはexampleをまとめるものでした。そして実はいくらネストしてもOK。

```ruby
describe User do
  describe '.find_by' do
    before do
      create :user, email: email
    end

    let(:email) { 'tom@example.com' }

    it 'returns tom' do
      user = User.find_by(email: email)
      expect(user.email).to eq email
    end
  end
end
```

すっ飛ばしましたが、一番外側に対象のクラス名書いたら、個別のテスト対象メソッドごとにdescribeわけるとよいです。メソッドごとにbeforeブロックとかもわけて書きやすくなるし、テスト対象がわかりやすくなりましたね。

しれっとspecifyをitに変えました。処理的には同じなのですが、何のメソッドをテストしているかがdescribeに切りだされたので、重複して書く必要がありません。itの方が自然に読み下せますね。

ちなみにこの状態でrspec -fdすると

```
User
  .find_by
    returns tom

Finished in 0.10705 seconds
1 example, 0 failures
```

こんな感じ。


# まとめ
Railsのモデルspecを題材に、基礎的なRSpecの書き方でした。
いきなり最後の例が出てくると、どこから見ればよいのかちょっと混乱するかもですが、
慣れると結構わかりやすいのではないでしょうか。

本当はletとlet!だったりsubjectだったりいろいろなmatcherとかも書きたかったのですが、
思いの外長くなってきたので今回はこの辺で。
