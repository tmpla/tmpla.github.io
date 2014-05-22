---
layout: post
title: "Rails開発環境をクリーンに保つ2014年初夏"
modified: 2014-05-10 17:35:06 +0900
tags: [Rails,Ruby,開発]
image:
  feature:
  credit:
  creditlink:
comments:
share:
status: draft
published: false
---
Railsで開発を行う場合には当たり前だけど環境を用意しなければいけない。
ということはまずRailsでも環境を用意する必要があるし、トレンドもあるかと思う。

その辺りをまとめてみたのでメモる。
環境はVirtualBox上のCentOS6.5をMacのMervericsで動かすと言うことにする。
また、それをvagrantで動かしている。
なので、ユーザーvagrantで動かすことを前提としている。

## Ruby環境の準備
Rubyをインストールするのはそりゃソースコードを持ってきてコンパイルすればいいはずだが、
Rubyはバージョンが違えば話が違ってくるところが色々ある。
なので、ファイルシステムの切り替え作業でバージョンを切り替える事ができるrbenvと言うツールを使うことにする。

[sstephenson/rbenv](https://github.com/sstephenson/rbenv)

rbenvはhomebrewでインストールもできるが、その中身を知るために、マニュアルで設定してみる。

まずインストール

{% highlight bash %}
$ git clone https://github.com/sstephenson/rbenv.git ~/.rbenv
$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
$ echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
$ source ~/.bash_profile
$ rbenv -v
{% endhighlight %}

とりあえず、最後のrbenv -vで普通にバージョンが出ればOKである。
RVMはちょっと忘れた。

この作業でどういうクリーンな事ができたかというと、/usr/bin/rubyを呼ぶ前に、
/usr/local/bin/ruby（または任意のruby起動スクリプト）を呼ぶことでシステムのもの触らずに済むように回避しているようだ。

また、これを使ってrubyの実行ファイルを管理する場合、ruby_buildと言うrbenvのプラグイン？があるのだが、
これを使うとソースコードからビルドしてrbenv管理下に置く事ができる。
新しいrubyを使うならばこれも入れていこう。
下記公式サイトのコマンドからrbrnvのプラグインとして動かす事を考えている事がわかる

{% highlight bash %}
$ git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
{% endhighlight %}

正直、どういう動作原理なのかはよく見てないのだが、これで新Rubyをソースコードからコンパイルが出来る。
今何も入っていない状態なので、入れてみよう。

## gem環境の準備
