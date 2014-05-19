---
layout: post
title: cygwinでchef環境を整える（難易度激高）
status: publish
---

Windows7にcygwinをインストールし、chefをインストールするのに苦労した話。

## 環境
環境はWindows7(32bit)で、cygwinは

{% highlight bash %}
$ cygcheck.exe -c cygwin
Cygwin Package Information
Package              Version        Status
cygwin               1.7.29-2       OK
{% endhighlight %}

である。ちなみにminttyを使っていて、rubyはrbenvで。2.1.2を入れた状態。

~~~ bash
$ rbenv versions
* 2.1.2 (set by /cygdrive/c/Users/k-yokoshima/.rbenv/version)
~~~

## chefインストール
gemでインストールすると、yajl-rubyのnative extensionのコンパイルのあたりで下記ようなメッセージが延々と出る。

~~~ bash
$ ... address space needed by 'fcntl.so' ...
~~~

ちなみに、yajl-rubyをfcntl.soもなんのことだかさっぱりわからないのでググってみると先人が居た。

[CygwinでRubyスナップショットをビルド - わさっき](http://d.hatena.ne.jp/takehikom/20130305/1362426096)

よく分かってないけど、rebaseということをすれば良さそうで、自分の場合は~/.rbenv以下のsoファイルをrebaseしてあげれば良さそうなのでこうした。

~~~ bash
$ find ~/.rbenv -name '*.so' > ~/rebase_so.lst
~~~


そして、エクスプローラからC:\cygwin\bin\ash.exeを起動して、下記コマンドを打つ。(ashというのはタブ補完とかそんな軟弱な機能はない)

~~~ bash
$ /bin/rebaseall -T rebase_so.lst -v
~~~

これでgem i chefは通るようになった。

### berkshelf問題
chefのレシピを実行する際に気づいた。berkshelfが入っていないことに。
なので入れてみたが無理だった。

~~~ bash
$ gem i berkshelf
DL is deprecated, please use Fiddle
Building native extensions.  This could take a while...
ERROR:  Error installing berkshelf:
        ERROR: Failed to build gem native extension.

    /cygdrive/c/Users/k-yokoshima/.rbenv/versions/2.1.2/bin/ruby.exe extconf.rb
-> sh /cygdrive/c/Users/k-yokoshima/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/dep-selector-libgecode-1.0.0/ext/libgecode3/vendor/gecode-3.7.3/configure --prefix=/cygdrive/c/Users/k-yokoshima/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/dep-selector-libgecode-1.0.0/lib/dep-selector-libgecode/vendored-gecode --disable-doc-dot --disable-doc-search --disable-doc-tagfile --disable-doc-chm --disable-doc-docset --disable-qt --disable-examples --disable-flatzinc
checking for the host operating system... Windows
checking whether the C++ compiler works... no
configure: error: in `/cygdrive/c/Users/k-yokoshima/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/dep-selector-libgecode-1.0.0/ext/libgecode3/vendor/gecode-3.7.3':
configure: error: C++ compiler cannot create executables
See `config.log' for more details
extconf.rb:89:in `block in run': Failed to build gecode library. (GecodeBuild::BuildError)
        from extconf.rb:88:in `chdir'
        from extconf.rb:88:in `run'
        from extconf.rb:95:in `<main>'

extconf failed, exit code 1

Gem files will remain installed in /cygdrive/c/Users/k-yokoshima/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/dep-selector-libgecode-1.0.0 for inspection.
Results logged to /cygdrive/c/Users/k-yokoshima/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/extensions/x86-cygwin/2.1.0/dep-selector-libgecode-1.0.0/gem_make.out

~~~

まずこれ。
どうやっても「dep-selector-libgecode-1.0.0」というのが入らない。
このメッセージを見ると、C++のコンパイラがないというメッセージだが、g++はcygwin側で入れている。

なのですごく面倒くさいがいろいろ調べてみると、
C:\Users\k-yokoshima\.rbenv\versions\2.1.2\lib\ruby\gems\2.1.0\gems\dep-selector-libgecode-1.0.0\ext\libgecode3\vendor\gecode-3.7.3\config.logにconfigureの結果が吐出されているようなので見ていた。

~~~ bash
## ----------- ##
## Core tests. ##
## ----------- ##

configure:2687: checking for the host operating system
configure:2707: result: Windows
configure:2832: checking for C++ compiler version
configure:2841: cl --version >&5
/cygdrive/c/Users/k-yokoshima/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/dep-selector-libgecode-1.0.0/ext/libgecode3/vendor/gecode-3.7.3/configure: line 2843: cl: command not found
~~~

cl: command not foundと言っているので、おそらくVisual Studioのcl.exeのことを言っているのだと思ってVisual Studioをインストールして「C:\Program Files\Microsoft Visual Studio 12.0\VC\bin」をPATHに追加して見たりしたがそこでまたlibなんとかがないと言うVC側のエラーっぽいのが出たのでこりゃダメだと思って諦めた。

この件も詳細は違うが偉大な先人がおられて、それを参考にローカルgemファイルを作ってやってみたりしたがやはりダメだった。

[BerkshelfとChefのインストールに苦労した話 - DQNEO起業日記](http://dqn.sakusakutto.jp/2013/12/berkshelf_chef_gem_ruby.html)

ちなみに、berkshelfをローカルgemをインストールする方法としては

~~~ bash
$ git clone https://github.com/berkshelf/berkshelf
$ cd berkshelf
$ gem build berkshelf.gemspec
$ gem i berkshelf-3.1.2.gem
~~~

で出来る（出来ないが）


でそもそも、先週まで普通に動いていたのがなぜ動かないのか。
先週まで使用していた環境は、4月に準備した環境である。
一応念のために、[berkshelfのリポジトリ](https://github.com/berkshelf/berkshelf/tree/v3.0.0)を見てみると、どうも4月中に3.0.0がリリースされているような感じである。ということは、4月中にberkshelfをインストールした可能性が高いので、2系の最終バージョン(現在2.0.16を指定して入れてみた

~~~ bash
$ gem i berkshelf -v '2.0.16'
~~~

するとなんとアッサリ入った。