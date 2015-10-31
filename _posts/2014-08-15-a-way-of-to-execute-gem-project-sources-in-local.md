---
layout: post
title: "Gemのソースをローカルで動かす方法"
modified: 2014-08-15 10:06:14 +0900
tags: [Gem,Github]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---

Rubyでプログラムを書いていて、[RubyGems](https://rubygems.org/)(以下、gem）を使わないということはあまりというか殆どないのではないでしょうか。
gemはRubyから使用できるライブラリを簡単に利用できるようにしたパッケージです。javaならjar, phpならpharみたいなもの。

これは世界中の多くの開発者が提供してくれており、ちょっと思いつきそうなものなら探せばだいたいあります。
また、多くはソースコードも公開してくれているのが多くて、githubなどで簡単に中身を確認する事もできます。

gemをローカル環境で使用できるようにするには、gem installコマンドで使えるようになるのですが、
例えば、あるgemをちょっとソースコードを変更して使用してみたい場合にどうやったかをメモします。

今回、[anemone](https://github.com/chriskite/anemone)というgemを使いたかったのですが、2年位メンテされてないので
一部変更して使ってみることにしました。
作業用にVirtualBox + vagrantのCentOSで、gem_localという名前のディレクトリで作業しました。

~~~ bash
/home/vagrant/workspace/gem_local
~~~

とりあえずは、本家からforkして、それをcloneしました。
そしてファイル構成はこんな感じ。

~~~ bash
$ git clone https://github.com/kyokoshima/anemone.git
Initialized empty Git repository in /home/vagrant/workspace/gem_local/anemone/.git/
remote: Counting objects: 1300, done.
remote: Compressing objects: 100% (15/15), done.
remote: Total 1300 (delta 4), reused 0 (delta 0)
Receiving objects: 100% (1300/1300), 263.59 KiB | 276 KiB/s, done.
Resolving deltas: 100% (587/587), done.
$ cd anemone
[vagrant@localhost anemone]$ ls -la
合計 60
drwxr-xr-x 6 vagrant vagrant 4096  8月 15 01:27 2014 .
drwxrwxr-x 3 vagrant vagrant 4096  8月 15 01:27 2014 ..
drwxrwxr-x 8 vagrant vagrant 4096  8月 15 01:27 2014 .git
-rw-rw-r-- 1 vagrant vagrant   47  8月 15 01:27 2014 .gitignore
-rw-rw-r-- 1 vagrant vagrant 2751  8月 15 01:27 2014 CHANGELOG.rdoc
-rw-rw-r-- 1 vagrant vagrant  203  8月 15 01:27 2014 CONTRIBUTORS
-rw-rw-r-- 1 vagrant vagrant   26  8月 15 01:27 2014 Gemfile
-rw-rw-r-- 1 vagrant vagrant 1056  8月 15 01:27 2014 LICENSE.txt
-rw-rw-r-- 1 vagrant vagrant 1304  8月 15 01:27 2014 README.rdoc
-rw-rw-r-- 1 vagrant vagrant  520  8月 15 01:27 2014 Rakefile
-rw-rw-r-- 1 vagrant vagrant    6  8月 15 01:27 2014 VERSION
-rw-rw-r-- 1 vagrant vagrant 1209  8月 15 01:27 2014 anemone.gemspec
drwxrwxr-x 2 vagrant vagrant 4096  8月 15 01:27 2014 bin
drwxrwxr-x 3 vagrant vagrant 4096  8月 15 01:27 2014 lib
drwxrwxr-x 2 vagrant vagrant 4096  8月 15 01:27 2014 spec
~~~

これで一度上(gem_local)に戻って、Gemfileにこのanemoneを使うように記述しました。

~~~ bash
$ cd ../
$ bundle init
$ vim Gemfile
# A sample Gemfile
source "https://rubygems.org"

# gem "rails"
gem "anemone", path: "anemone"
~~~

そして、本当それ使われているのかどうかを確認するためにgemspecのバージョンをちょっと変更してみます。

~~~ bash
spec = Gem::Specification.new do |s|
  s.name = "anemone"
- s.version = "0.7.2"
+ s.version = "0.7.2-dev"
~~~

bundleコマンドをやってみます。

~~~ bash
$ bundle
Resolving dependencies...
Using mini_portile 0.6.0
Using nokogiri 1.6.3.1
Using robotex 1.0.0
Using anemone 0.7.2.pre.dev from source at anemone
Using bundler 1.6.3
Your bundle is complete!
Use `bundle show [gemname]` to see where a bundled gem is installed.
~~~

もともとNokogiriとか依存関係のgemはすでにあるのでそれを使っていますが、anemoneはソースコードから参照されているように見えます。
これで下記のようなスクリプトを実行してみました。

そして、pryからrequireしてみます。

~~~ bash
$ pry
[1] pry(main)> require 'anemone'
LoadError: cannot load such file -- anemone
from /home/vagrant/.rbenv/versions/2.1.2/lib/ruby/2.1.0/rubygems/core_ext/kernel_require.rb:55:in `require'
~~~

見つからないようです。どうしたもんかと色々やってみたのですがダメでした。

~~~ bash
[2] pry(main)> require './anemone'
LoadError: cannot load such file -- ./anemone
from /home/vagrant/.rbenv/versions/2.1.2/lib/ruby/2.1.0/rubygems/core_ext/kernel_require.rb:55:in `require'
[3] pry(main)> require './anemone/lib'
LoadError: cannot load such file -- ./anemone/lib
from /home/vagrant/.rbenv/versions/2.1.2/lib/ruby/2.1.0/rubygems/core_ext/kernel_require.rb:55:in `require'
[4] pry(main)> require './anemone/lib/anemone'
LoadError: cannot load such file -- anemone/core
from /home/vagrant/.rbenv/versions/2.1.2/lib/ruby/2.1.0/rubygems/core_ext/kernel_require.rb:55:in `require'
[5] pry(main)> require './anemone/lib/anemone.rb'
LoadError: cannot load such file -- anemone/core
from /home/vagrant/.rbenv/versions/2.1.2/lib/ruby/2.1.0/rubygems/core_ext/kernel_require.rb:55:in `require'
~~~

./anemone/lib/anemoneとやると、anemone/coreが見つからない、みたいになっていますね。
これはどうやって認識させるのだろうと思って調べたら$LOAD\_PATHというのがあるようで、そこに本体があるパスを追加すれば良いみたいでした。

（閃いたのはこちらの記事を見た時[ RubyGemsはrequireの裏で何をやっているのか？ - ブログのおんがえし](http://ongaeshi.hatenablog.com/entry/20111214/1323830203)

$LOAD\_PATHを表示させてみたらこんな感じ。

~~~ bash
[7] pry(main)> $LOAD_PATH
=> ["/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/coderay-1.1.0/lib",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/method_source-0.8.2/lib",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/pry-0.10.0/lib",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/slop-3.6.0/lib",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/columnize-0.8.9/lib",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/debugger-linecache-1.2.0/lib",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/extensions/x86_64-linux/2.1.0-static/byebug-2.7.0",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/byebug-2.7.0/lib",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/pry-byebug-1.3.3/lib",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/pry-rails-0.3.2/lib",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/site_ruby/2.1.0",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/site_ruby/2.1.0/x86_64-linux",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/site_ruby",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/vendor_ruby/2.1.0",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/vendor_ruby/2.1.0/x86_64-linux",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/vendor_ruby",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/2.1.0",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/2.1.0/x86_64-linux"]
~~~

なるほど、gemの場合はlibディレクトリを追加すれば良さそうなので下記のようにしてみました。

~~~ bash
[8] pry(main)> $LOAD_PATH<<'./anemone/lib'
=> ["/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/coderay-1.1.0/lib",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/method_source-0.8.2/lib",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/pry-0.10.0/lib",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/slop-3.6.0/lib",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/columnize-0.8.9/lib",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/debugger-linecache-1.2.0/lib",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/extensions/x86_64-linux/2.1.0-static/byebug-2.7.0",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/byebug-2.7.0/lib",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/pry-byebug-1.3.3/lib",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/pry-rails-0.3.2/lib",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/site_ruby/2.1.0",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/site_ruby/2.1.0/x86_64-linux",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/site_ruby",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/vendor_ruby/2.1.0",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/vendor_ruby/2.1.0/x86_64-linux",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/vendor_ruby",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/2.1.0",
 "/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/2.1.0/x86_64-linux",
 "./anemone/lib"]
[9] pry(main)> require 'anemone'
=> true
~~~

追加したあとrequireしたら成功しているようでした。試しにクロールもしてみよう。

~~~ bash
[10] pry(main)> Anemone.crawl 'http://www.yahoo.co.jp', {depth_limit: 0} do |anemone|
[10] pry(main)*   anemone.on_every_page do | page |
[10] pry(main)*     puts page.doc.search('//head/title/text()').to_s
[10] pry(main)*   end
[10] pry(main)* end
Yahoo! JAPAN
=> #<Anemone::Core:0x007f0c31159430
 @after_crawl_blocks=[],
 @on_every_page_blocks=[#<Proc:0x007f0c31158dc8@(pry):11>],
 @on_pages_like_blocks={},
  :
  :
~~~

うまく動いているようでした。
ちなみに、$LOAD_PATHは$:とも書けるようなので、下記のようにするとちょっと短いです。

~~~ bash
$:<<'./anemone/lib'
~~~

小ネタでした。