---
layout: post
title: "Capybara WebkitをCentOSで使えるようにする方法"
modified: 2014-07-30 10:07:46 +0900
tags: [Ruby,Capybara,Webkit]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---

# capybara-webkit

外部Webサイトをクロールする方法をどうしようか色々考えているのですが、クロールと、実はスクリーンショットも取れないかな？
と調べていたところ、[capybara-webkit](https://github.com/thoughtbot/capybara-webkit)というのでスクリーンショットが取れるような噂を聞きました。
webkitと付いているので何となく想像出来たのですが、これはJavaScriptも実行出来ちゃうようです。
[Capybara](https://github.com/jnicklas/capybara)はWebアプリケーションの主に画面上の操作をシミュレートするためのものですが、
capybara-webkitはcapybaraのページの操作をするドライバの一つのようです。
これを使ういつものトライ＆エラーのインストール手順メモです。

# 環境

環境はVirtualBoxをvagrantで使いCentOS6.5です。
rubyはrbenvで入れた2.1.2が使える状態です。
また、capybara-webkit-sampleというディレクトリを作ってそこで色々やってます。
また、bundlerだけ入れてます。bundler入れたあとはrbenv rehashを忘れずに。

# サンプルプロジェクトの作成

## gem準備

~~~ bash
$ mkdir capybara-webkit-sample
$ cd capybara-webkit-sample
$ bundle init
~~~

Gemfileにcapybara-webkitとだけ書いてやってみます。

~~~ ruby
# A sample Gemfile
source "https://rubygems.org"

# gem "rails"

gem "capybara-webkit"
~~~

そしてbundle

~~~ bash
$ bundle
Fetching gem metadata from https://rubygems.org/...........
Fetching additional metadata from https://rubygems.org/..
Resolving dependencies...
Installing mime-types 2.3
Installing mini_portile 0.6.0
Building nokogiri using packaged libraries. 
Building libxml2-2.8.0 for nokogiri with the following patches applied:
  - 0001-Fix-parser-local-buffers-size-problems.patch
  - 0002-Fix-entities-local-buffers-size-problems.patch
  - 0003-Fix-an-error-in-previous-commit.patch
  - 0004-Fix-potential-out-of-bound-access.patch
  - 0005-Detect-excessive-entities-expansion-upon-replacement.patch
  - 0006-Do-not-fetch-external-parsed-entities.patch
  - 0007-Enforce-XML_PARSER_EOF-state-handling-through-the-pa.patch
  - 0008-Improve-handling-of-xmlStopParser.patch
  - 0009-Fix-a-couple-of-return-without-value.patch
  - 0010-Keep-non-significant-blanks-node-in-HTML-parser.patch
  - 0011-Do-not-fetch-external-parameter-entities.patch
************************************************************************
IMPORTANT!  Nokogiri builds and uses a packaged version of libxml2.

If this is a concern for you and you want to use the system library
instead, abort this installation process and reinstall nokogiri as
follows:

    gem install nokogiri -- --use-system-libraries

If you are using Bundler, tell it to use the option:

    bundle config build.nokogiri --use-system-libraries
    bundle install

However, note that nokogiri does not necessarily support all versions
of libxml2.

For example, libxml2-2.9.0 and higher are currently known to be broken
and thus unsupported by nokogiri, due to compatibility problems and
XPath optimization bugs.
************************************************************************

Gem::Ext::BuildError: ERROR: Failed to build gem native extension.

    /home/vagrant/.rbenv/versions/2.1.2/bin/ruby extconf.rb
Building nokogiri using packaged libraries.
checking for iconv.h... yes
checking for iconv_open() in iconv.h... yes
Building libxml2-2.8.0 for nokogiri with the following patches applied:
  - 0001-Fix-parser-local-buffers-size-problems.patch
  - 0002-Fix-entities-local-buffers-size-problems.patch
  - 0003-Fix-an-error-in-previous-commit.patch
  - 0004-Fix-potential-out-of-bound-access.patch
  - 0005-Detect-excessive-entities-expansion-upon-replacement.patch
  - 0006-Do-not-fetch-external-parsed-entities.patch
  - 0007-Enforce-XML_PARSER_EOF-state-handling-through-the-pa.patch
  - 0008-Improve-handling-of-xmlStopParser.patch
  - 0009-Fix-a-couple-of-return-without-value.patch
  - 0010-Keep-non-significant-blanks-node-in-HTML-parser.patch
  - 0011-Do-not-fetch-external-parameter-entities.patch
************************************************************************
IMPORTANT!  Nokogiri builds and uses a packaged version of libxml2.

If this is a concern for you and you want to use the system library
instead, abort this installation process and reinstall nokogiri as
follows:

    gem install nokogiri -- --use-system-libraries

If you are using Bundler, tell it to use the option:

    bundle config build.nokogiri --use-system-libraries
    bundle install

However, note that nokogiri does not necessarily support all versions
of libxml2.

For example, libxml2-2.9.0 and higher are currently known to be broken
and thus unsupported by nokogiri, due to compatibility problems and
XPath optimization bugs.
************************************************************************
Extracting libxml2-2.8.0.tar.gz into tmp/x86_64-unknown-linux-gnu/ports/libxml2/2.8.0... OK
Running patch with /home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/nokogiri-1.6.3.1/ports/patches/libxml2/0001-Fix-parser-local-buffers-size-problems.patch...
Running 'patch' for libxml2 2.8.0... ERROR, review 'tmp/x86_64-unknown-linux-gnu/ports/libxml2/2.8.0/patch.log' to see what happened.
*** extconf.rb failed ***
Could not create Makefile due to some reason, probably lack of necessary
libraries and/or headers.  Check the mkmf.log file for more details.  You may
need configuration options.

Provided configuration options:
  --with-opt-dir
  --without-opt-dir
  --with-opt-include
  --without-opt-include=${opt-dir}/include
  --with-opt-lib
  --without-opt-lib=${opt-dir}/lib
  --with-make-prog
  --without-make-prog
  --srcdir=.
  --curdir
  --ruby=/home/vagrant/.rbenv/versions/2.1.2/bin/ruby
  --help
  --clean
  --use-system-libraries
  --enable-static
  --disable-static
  --with-zlib-dir
  --without-zlib-dir
  --with-zlib-include
  --without-zlib-include=${zlib-dir}/include
  --with-zlib-lib
  --without-zlib-lib=${zlib-dir}/lib
  --enable-cross-build
  --disable-cross-build
/home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/mini_portile-0.6.0/lib/mini_portile.rb:279:in `block in execute': Failed to complete patch task (RuntimeError)
  from /home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/mini_portile-0.6.0/lib/mini_portile.rb:271:in `chdir'
  from /home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/mini_portile-0.6.0/lib/mini_portile.rb:271:in `execute'
  from extconf.rb:282:in `block in patch'
  from extconf.rb:279:in `each'
  from extconf.rb:279:in `patch'
  from /home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/mini_portile-0.6.0/lib/mini_portile.rb:108:in `cook'
  from extconf.rb:253:in `block in process_recipe'
  from extconf.rb:154:in `tap'
  from extconf.rb:154:in `process_recipe'
  from extconf.rb:423:in `<main>'

extconf failed, exit code 1

Gem files will remain installed in /home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/nokogiri-1.6.3.1 for inspection.
Results logged to /home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/extensions/x86_64-linux/2.1.0-static/nokogiri-1.6.3.1/gem_make.out
An error occurred while installing nokogiri (1.6.3.1), and Bundler cannot continue.
Make sure that `gem install nokogiri -v '1.6.3.1'` succeeds before bundling.
~~~

うん、絶対nokogiriのインストールで出ると思った。これはおそらくpatchコマンドがないからじゃないかと思うのでpatchコマンドをインストールします。

~~~ bash
$ sudo yum install -y patch
$ bundle
Fetching gem metadata from https://rubygems.org/...........
Fetching additional metadata from https://rubygems.org/..
Resolving dependencies...
Using mime-types 2.3
Using mini_portile 0.6.0
Building nokogiri using packaged libraries.

  :

Installing nokogiri 1.6.3.1
Installing rack 1.5.2
Installing rack-test 0.6.2
Installing xpath 2.0.0
Installing capybara 2.3.0
Using json 1.8.1

Gem::Ext::BuildError: ERROR: Failed to build gem native extension.

    /home/vagrant/.rbenv/versions/2.1.2/bin/ruby extconf.rb
Command 'qmake -spec linux-g++ ' not available

Makefile not found

Gem files will remain installed in /home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/capybara-webkit-1.2.0 for inspection.
Results logged to /home/vagrant/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/extensions/x86_64-linux/2.1.0-static/capybara-webkit-1.2.0/gem_make.out
An error occurred while installing capybara-webkit (1.2.0), and Bundler cannot continue.
Make sure that `gem install capybara-webkit -v '1.2.0'` succeeds before bundling.
~~~

こんどはqmakeなんちゃらでエラーになっているようだった。
[capybara-webkitのwiki](https://github.com/thoughtbot/capybara-webkit/wiki/Installing-Qt-and-compiling-capybara-webkit)にqtというツールが必要との記載があるので、
それを入れてみます。

## qtインストール

まずは、yumリポジトリの設定

~~~ bash
$ vim /etc/yum.repos.d/qt48.repo
[epel-qt48]
name=Software Collection for Qt 4.8
baseurl=http://repos.fedorapeople.org/repos/sic/qt48/epel-$releasever/$basearch/
enabled=1
skip_if_unavailable=1
gpgcheck=0

[epel-qt48-source]
name=Software Collection for Qt 4.8 - Source
baseurl=http://repos.fedorapeople.org/repos/sic/qt48/epel-$releasever/SRPMS
enabled=0
skip_if_unavailable=1
gpgcheck=0
~~~

続いてインストール

~~~ bash
$ sudo yum install -y qt48-qt-webkit-devel
~~~

ヘッダファイルのリンク

~~~ bash
$ sudo ln -s /opt/rh/qt48/root/usr/include/QtCore/qconfig-64.h  /opt/rh/qt48/root/usr/include/QtCore/qconfig-x86_64.h
~~~

パス等の環境変数へのロード

~~~ bash
$ source /opt/rh/qt48/enable
$ export PATH=/opt/rh/qt48/root/usr/lib64/qt4/bin/${PATH:+:${PATH}}
~~~

再度実行

~~~ bash
$ bundle
Fetching gem metadata from https://rubygems.org/...........
Fetching additional metadata from https://rubygems.org/..
Resolving dependencies...
Using mime-types 2.3
Using mini_portile 0.6.0
Using nokogiri 1.6.3.1
Using rack 1.5.2
Using rack-test 0.6.2
Using xpath 2.0.0
Using capybara 2.3.0
Using json 1.8.1
Installing capybara-webkit 1.2.0
Using bundler 1.6.5
Your bundle is complete!
Use `bundle show [gemname]` to see where a bundled gem is installed.
~~~

うまく行ったようでした。

# スクリーンショットの取り方

こんなスクリプトを書いてみました。

~~~ ruby
require 'capybara-webkit'

d = Capybara::Webkit::Driver.new(nil)
d.visit 'http://www.yahoo.co.jp'
d.save_screenshot 'yahoo.png'
~~~

Driverのbrowserメンバを操作する手もあるのですが、DriverはほとんどbrowserのラッパーのようなのでDriverを操作しています。
実行してみると

~~~ bash
$ ruby screenshot.rb
webkit_server: cannot connect to X server
~~~

X Serverに接続出来ないとのこと。そんなの設定してないので入れてみます。

こちらを参考にさせて頂きました。

[RSpec Capybara Webkit Xvfb CentOS 要するにRailsでjavascriptをテストしたい | Workabroad.jp](http://www.workabroad.jp/posts/1138)

## Xvfbのインストール

~~~ bash
$ sudo yum install -y xorg-x11-server-Xvfb.x86_64
~~~

## headlessのインストール

~~~ ruby
# A sample Gemfile
source "https://rubygems.org"

# gem "rails"

gem "capybara-webkit"
gem "headless" # 追加
~~~

~~~ bash
$ bundle
Fetching gem metadata from https://rubygems.org/...........
Fetching additional metadata from https://rubygems.org/..
Resolving dependencies...
Using mime-types 2.3
Using mini_portile 0.6.0
Using nokogiri 1.6.3.1
Using rack 1.5.2
Using rack-test 0.6.2
Using xpath 2.0.0
Using capybara 2.3.0
Using json 1.8.1
Using capybara-webkit 1.2.0
Installing headless 1.0.2
Using bundler 1.6.5
Your bundle is complete!
Use `bundle show [gemname]` to see where a bundled gem is installed.
~~~

これで下記のようにソースを修正してみました。

~~~ ruby
require 'capybara-webkit'
require 'headless'

headless = Headless.new
headless.start

driver = Capybara::Webkit::Driver.new(nil)
driver.visit 'http://www.yahoo.co.jp'
driver.save_screenshot 'yahoo.png'

headless.destroy
~~~

~~~
$ ruby screenshot.rb
~~~

すると、スクリーンショットは無事に取得出来ていたのだが、日本語部分が四角になってしまっているようでした。

![Yahoo文字化け]({{ site_url }}/images/07/30/yahoo-garbled.png)

これは多分日本語フォントをインストールすれば直るのではないかと思って入れて再度やってみました。

~~~ bash
$ sudo yum groupinstall -y "Japanese Support"
~~~

![Yahoo]({{ site_url }}/images/07/30/yahoo.png)

これでうまく行ったようです。
本当は、左上のマウスポインタをなんとか消したいところですが、C++の処理の方でやっていそうなのであきらめましたが・・・。