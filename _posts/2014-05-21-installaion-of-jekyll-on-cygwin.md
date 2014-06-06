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
share: true
status: publish
published: true
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
bundlerでインストールしておきたいのだが、色々やってみたところグローバルgemとプロジェクトgem(vendor/bundleの)の両方のrebaseで
うまくいかないので、グローバルgemでインストールすることにした。
（グローバルgemでもインストール中にrebaseが必要なところはあるが、それはなんとかrebaseで解決した、というところからにする）

~~~ bash
$ gem i jekyll
~~~

rebaseはこの段階では下記の3つで解決できたようである。

~~~ bash
$ rebase -sv ~/.rbenv/versions/2.1.1/lib/ruby/2.1.0/i386-cygwin/digest.so
$ rebase -sv ~/.rbenv/versions/2.1.1/lib/ruby/2.1.0/i386-cygwin/fcntl.so
$ rebase -sv ~/.rbenv/versions/2.1.1/lib/ruby/2.1.0/i386-cygwin/date_core.so
~~~

## Pygmentsのインストール
[Pygments](http://pygments.org/)は、ざっくり言えばプログラムコードのキーワードを見やすいように強調表示させたいときに役に立つやつである。
こういうのはSyntax highlighter等というが、Jekyllではデフォルトでこれを使っているようである。
ただし、これはPythonのパッケージのようなのでこれを入れておく。
さらに、setuptoolsというPythonのパッケージマネージャ？からeasy_installというこれまたパッケージマネージャ？経由でPygmentsをインストールするのが作法のようだ。（ややこしい・・・）

~~~ bash
$ apt-cyg install python curl
$　curl 'https://bitbucket.org/pypa/setuptools/raw/bootstrap/ez_setup.py' | python
$ easy_install Pygments
~~~

## COMSPEC環境変数の設定
このシンタックスハイライト部分を処理するところでcmd.exeを使用するようなのだが、パスの形式がWindows形式で使用しようとするようである。
ビルド時に「 Liquid Exception: No such file or directory - C:\Windows\system32\cmd.exe」と、Windows形式のcmd.exeへのパスとなってしまうようだ。
これはCOMSPEC環境変数に、Unix式のcmd.exeのパスを指定すれば良いようだ。

~~~ bash
$ vim ~/.bashrc
export COMSPEC=/cygdrive/c/Windows/System32/cmd.exe
~~~

## Python実行ファイルの指定

さらに、「python2」というコマンドを使用したいらしいが、pythonをインストールした時点では「/usr/bin/python」しかないと思う。
なので、「python2」という名前でシンボリックリンクを貼る。

~~~ bash
$ ln -s /usr/bin/python /usr/local/bin/python2
~~~

## サイト作成
jekyllのサイトを新規作成してみる。

そして「site」という名前で作ってみる。もちろん適当な名前である。

~~~ bash
$ jekyll new site
DL is deprecated, please use Fiddle
New jekyll site installed in /cygdrive/c/Users/k-yokoshima/Work/jekyll_test/site.
~~~

そして一旦このまま生成してみる。
~~~ bash
$ jekyll build --trace
Configuration file: /cygdrive/c/Users/k-yokoshima/Work/jekyll_test/site/_config.yml
            Source: /cygdrive/c/Users/k-yokoshima/Work/jekyll_test/site
       Destination: /cygdrive/c/Users/k-yokoshima/Work/jekyll_test/site/_site
      Generating... done.
~~~

site/_siteに無事色々生成されたようであった。


元ネタは下記記事です。

[Liquid exception under Cygwin · Issue #1383 · jekyll/jekyll](https://github.com/jekyll/jekyll/issues/1383)
[Jekyll on Windows with Cygwin](http://nathanielstory.com/2013/12/28/jekyll-on-windows-with-cygwin.html)
