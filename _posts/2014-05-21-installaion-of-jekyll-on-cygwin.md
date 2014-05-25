---
layout: post
title: "cygwinでjekyllを動かす"
modified: 2014-05-21 13:54:21 +0900
tags: [jekyll,cygwin,ruby]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
status: draft
published: false
---
[jekyll](http://jekyllrb.com/)は、Markdownのように、テキスト＋いくつかのシンタックスルールを使って書いたものを、
Webブラウザで見れるようにHTMLへ変換してくれるものである。

ブログのソフトウェアでWordpressとMovableTypeがメジャーどこで、Wordpressがリクエスト毎にHTMLを生成するが、
MovableTypeは記事作成時にHTMLを生成する、後者の方に似ているといえば似ているのかも。
ただし、記事を書く管理画面などは当然なく、自分で記事やカテゴリ、タグなどを管理する。
好きなテキストエディタと、Markdown記法、あとHTMLをちょっと知っていれば簡単にブログのようなものを作成できる。
（ローカルで動かす場合は、jekyllのインストールは必要だが）

細かいことは省くが、[Github Pages](https://pages.github.com)にそれなりのルールでjekyllの記事や設定をプッシュすれば簡単に記事サイトを作ることが出来る。
それをローカルPCで動かして確認する事も当然できる。それを今回はcygwin上で行ってみた話。

## jekyllのインストール

cygwinは継続して使っているもので、今rbenvのカレントが2.1.2なので、2.1.1をrbenvで入れた状態でやってみる。
今まっさらな状態でruby2.1.1が入っている状態。

~~~ bash
$ cygcheck.exe -c cygwin
Cygwin Package Information
Package              Version        Status
cygwin               1.7.29-2       OK

$ rbenv versions
* 2.1.1 (set by /cygdrive/c/Users/k-yokoshima/.rbenv/version)
  2.1.2
~~~

で、とりえあずはjekyllのテスト用ディレクトリを作って、そこで作業することにする。
グローバルなgemでもいいのだが、とりあえずbundlerでjekyllをインストールすることにする。
~~~ bash
$ mkdir jekyll_test
$ cd jekyll_test/
$ bundle init
Writing new Gemfile to /cygdrive/c/Users/k-yokoshima/Work/jekyll_test/Gemfile
~~~

この時点でのGemfileはこう
~~~ ruby
# A sample Gemfile
source "https://rubygems.org"

gem "jekyll"
~~~

ここからbundle installするとみんな大好きashを使ってrebaseallすれば解決のunable to remap...のエラーが出る。

~~~ bash
$ bundle install --path vendor/bundle
DL is deprecated, please use Fiddle
Fetching gem metadata from https://rubygems.org/........
Fetching additional metadata from https://rubygems.org/..
Resolving dependencies...
Installing blankslate 2.1.2.4
Installing timers 1.1.0
Installing celluloid 0.15.2
      0 [main] ruby 2520 child_info_fork::abort: unable to create interim mapping for C:\Users\k-yokoshima\.rbenv\versions\2.1.1\lib\ruby\2.1.0\i386-cygwin\openssl.so, Win32 error 8
      0 [main] ruby 9104 child_info_fork::abort: unable to create interim mapping for C:\Users\k-yokoshima\.rbenv\versions\2.1.1\lib\ruby\2.1.0\i386-cygwin\openssl.so, Win32 error 8
      1 [main] ruby 9620 child_info_fork::abort: unable to create interim mapping for C:\Users\k-yokoshima\.rbenv\versions\2.1.1\lib\ruby\2.1.0\i386-cygwin\openssl.so, Win32 error 8
      0 [main] ruby 5076 child_info_fork::abort: unable to create interim mapping for C:\Users\k-yokoshima\.rbenv\versions\2.1.1\lib\ruby\2.1.0\i386-cygwin\openssl.so, Win32 error 8
      0 [main] ruby 9792 child_info_fork::abort: address space needed by 'fcntl.so' (0x540000) is already occupied
      0 [main] ruby 8208 child_info_fork::abort: address space needed by 'fcntl.so' (0x540000) is already occupied
      :
      :
~~~

これは以前解決したように、rebaseallをashからやれば解決できた。
が、rebaseallはashかdashから行わないといけないようである。なぜならbashから起動するとそういうメッセージが出るからである。

~~~ bash
$ rebaseall
rebaseall: only ash or dash processes are allowed during rebasing
    Exit all Cygwin processes and stop all Cygwin services.
    Execute ash (or dash) from Start/Run... or a cmd or command window.
    Execute '/bin/rebaseall' from ash (or dash).
~~~

しかし、rebaseという方は特にそんな警告も出ないのでそれでこれはbashのまま行けるのではと思ってやってみたら行けた。
helpを見ると、ファイル名と「-s」オプションと、詳細を出力したいので「v」でやってみた。

~~~ bash
$ rebase --help
Usage: rebase [OPTIONS] [FILE]...
Rebase PE files, usually DLLs, to a specified address or address range.

  -4, --32                Only rebase 32 bit DLLs.  This is the default.
  -8, --64                Only rebase 64 bit DLLs.
  -b, --base=BASEADDRESS  Specifies the base address at which to start rebasing.
  -s, --database          Utilize the rebase database to find unused memory
                          slots to rebase the files on the command line to.
                          (Implies -d).
                          If -b is given, too, the database gets recreated.
  -O, --oblivious         Do not change any files already in the database
                          and do not record any changes to the database.
                          (Implies -s).
  -i, --info              Rather then rebasing, just print the current base
                          address and size of the files.  With -s, use the
                          database.  The files are ordered by base address.
                          A '*' at the end of the line is printed if a
                          collisions with an adjacent file is detected.

  One of the options -b, -s or -i is mandatory.  If no rebase database exists
  yet, -b is required together with -s.

  -d, --down              Treat the BaseAddress as upper ceiling and rebase
                          files top-down from there.  Without this option the
                          files are rebased from BaseAddress bottom-up.
                          With the -s option, this option is implicitly set.
  -n, --no-dynamicbase    Remove PE dynamicbase flag from rebased DLLs, if set.
  -o, --offset=OFFSET     Specify an additional offset between adjacent DLLs
                          when rebasing.  Default is no offset.
  -t, --touch             Use this option to make sure the file's modification
                          time is bumped if it has been successfully rebased.
                          Usually rebase does not change the file's time.
  -T, --filelist=FILE     Also rebase the files specified in FILE.  The format
                          of FILE is one DLL per line.
  -q, --quiet             Be quiet about non-critical issues.
  -v, --verbose           Print some debug output.
  -V, --version           Print version info and exit.
  -h, --help, --usage     This help.

~~~

とりあえず、openssh.soがダメらしいのでそれを。

~~~ bash
$ rebase -sv ~/.rbenv/versions/2.1.1/lib/ruby/2.1.0/i386-cygwin/openssl.so
/usr/lib/sasl2_3/cygscram-3.dll: new base = 662b0000, new size = 10000
/usr/lib/sasl2_3/cygsasldb-3.dll: new base = 662c0000, new size = 10000
  :
  :
/cygdrive/c/Users/k-yokoshima/.rbenv/versions/2.1.2/lib/ruby/2.1.0/i386-cygwin/openssl.so: new base = 6bc00000, new size = 1400000
  :
  The following DLLs couldn't be rebased because they were in use:
  /usr/bin/cygreadline7.dll
  /usr/bin/cygncursesw-10.dll
  /usr/bin/cygintl-8.dll
  /usr/bin/cygiconv-2.dll
  /usr/bin/cyggcc_s-1.dll
  :
~~~

最後の方で、cygwinのdllがrebase出来なかったというメッセージが出ているが、まあ多分問題ないだろう。
またこれで、bundle installして警告が出てjekyllを入れる場合は下記のsoをrebaseすればよかったようだ。

~~~ bash
$ rebase -sv ~/.rbenv/versions/2.1.1/lib/ruby/2.1.0/i386-cygwin/digest.so
$ rebase -sv ~/.rbenv/versions/2.1.1/lib/ruby/2.1.0/i386-cygwin/fcntl.so
$ rebase -sv ~/.rbenv/versions/2.1.1/lib/ruby/2.1.0/i386-cygwin/date_core.so
~~~

これでbundle installが通った。と言うかやはり、~/.rbenv/**/*.soをリスト化して-Tオプションで渡して出来るようなので、全部やっておいたほうがいいかもしれない。

## サイト作成
jekyllのサイトを新規作成してみる。
最近、bundle install --binstubsがお気に入りなので、それもしておく。
~~~ bash
$ bundle install --binstubs
~~~

そして「site」という名前で作ってみる。もちろん適当な名前である。

~~~ bash
$ bin/jekyll new site
$ bin/jekyll new site
DL is deprecated, please use Fiddle
New jekyll site installed in /cygdrive/c/Users/k-yokoshima/Work/jekyll_test/site.
~~~

そして一旦このまま生成してみる。
~~~ bash
$ cd site
$ ../bin/jekyll build
DL is deprecated, please use Fiddle
Configuration file: /cygdrive/c/Users/k-yokoshima/Work/jekyll_test/site/_config.yml
            Source: /cygdrive/c/Users/k-yokoshima/Work/jekyll_test/site
       Destination: /cygdrive/c/Users/k-yokoshima/Work/jekyll_test/site/_site
      Generating...
      0 [main] ruby 6512 child_info_fork::abort: address space needed by 'yajl.so' (0x480000) is already occupied
      0 [main] ruby 9992 child_info_fork::abort: address space needed by 'stringio.so' (0x430000) is already occupied
      0 [main] ruby 9824 child_info_fork::abort: address space needed by 'yajl.so' (0x480000) is already occupied

~~~

また来た！しかしもう対処策はあるのでrebaseしよう。今度は、~/.rbenvではなく、vendor/bundleにあるものをやる。
せっかくのなのでリスト化してやってみよう。
vendor/bundleがあるディレクトリに移動して、

~~~ bash
$ rebase -sv -T `find vendor/bundle/ -name *.so`
~~~
rebaseall yaji-ruby

https://github.com/jekyll/jekyll/issues/1383
export COMSPEC=/cygdrive/c/Windows/System32/cmd.exe

http://nathanielstory.com/2013/12/28/jekyll-on-windows-with-cygwin.html