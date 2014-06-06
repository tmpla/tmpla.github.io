---
layout: post
title: "Cygwinのインストール2014年初夏"
modified: 2014-06-07 12:07:51 +0900
tags: [jekyll,cygwin]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: true
---
最近良く使っているWindowsマシンのハードディスクが壊れてしまったので、
またゼロから環境構築をしなくてはいけなくなった。

せっかくの機会なので、cygwinをインストールする手順をメモしておきたいと思った。
環境はWindows7(32bit)SP１である。

## cygwinのインストール

まずcygwinをインストールするところから始めよう。
[Cygwin](http://www.cygwin.com/)からsetup-x86.exeへのリンクが有ると思うので、
それをクリックしてダウンロードする。
現時点ではのバージョンは1.7.29のようである。
64bitマシンの人はsetup-x86_64.exeを使おう。

ダウンロードしたsetup-x86.exeを起動し、ミラーの選択やらプロキシやらの設定画面で「次へ」とどんどんやっていくと、
インストールするパッケージを選ぶ画面になる。
この画面で色々選択してインストールできるのだが、右上の検索窓からのインクリメンタルサーチがめちゃ遅くて使いにくい。
![Cygwinのパッケージ選択画面]({{ site.url }}/images/06/07/cygwin-package-select.png)

そこで、debianのapt-getみたいにこれらパッケージを入れることができるapt-cygというツールをまずは使えるようにする。

[apt-cyg - A command-line software installer for Cygwin - Google Project Hosting](https://code.google.com/p/apt-cyg/)

やり方はいろいろあると思うが、[githubに公開されている](https://github.com/transcode-open/apt-cyg)スクリプトをコピーすることにする。
スクリプトの中を見ると、gawk, bzip2, tar, wget, xzが必要なようなので、それはGUIのインストーラーで頑張ってインストールしておく。
そしたらcygwinを起動し、下記コマンドで/bin/以下にコピーする。あと実行権限をつける。

~~~ bash
$ wget --directory /bin https://raw.githubusercontent.com/transcode-open/apt-cyg/master/apt-cyg
$ chmod +x /bin/apt-cyg
$ apt-cyg --version
apt-cyg version 0.59
Written by Stephen Jungels

Copyright (c) 2005-9 Stephen Jungels.  Released under the GPL.

~~~

これでapt-cygを使える準備は出来た。
とりあえず、vimがないとすごく不便なので、vimを入れてみる。

~~~ bash
$ apt-cyg install vim
	:
	:
Package vim installed
$ vim
~~~

これでvimのスプラッシュ画面が出てきたのでOKぽい。

## homeディレクトリ問題
インストール直後では、ホームディレクトリは「/home/ユーザー名」の場所になっている。
これはWindowsから見ると、「C:\cygwin\home\ユーザー名」であり、Windows側と連携するときに何かと面倒くさい。
なので、このホームディレクトリをWindowsと合わせるように変更する。
普通、Windowsでのユーザー用ホームディレクトリは「C:\Users\ユーザー名」
になっているのではないだろうか。ここに統一しよう。
Cygwin側から見た上記ディレクトリは「/cygdrive/c/Users/ユーザー名」でアクセスできるのでこれをHOMEディレクトリとする。
これはWindowsのコントロールパネル>システムとセキュリティ>システム>システムの詳細設定>環境変数のユーザー環境変数へ下記の値で追加した。

	名前:HOME
	値:/cygdrive/c/Users/k-yokoshima/
	
これでcygwinを一度終了して再度起動すれば初期のディレクトリが変わっているはず。
ここで、もともとあったホームディレクトリのファイル（ドット付きファイルなど）もコピーしておこう。

~~~ bash
$ cp /home/k-yokoshima/.* ~
cp: ディレクトリ `/home/k-yokoshima/.' を省略しています
cp: ディレクトリ `/home/k-yokoshima/..' を省略しています
~~~

また、sshですごく影響があったのだけれど、cygwin内で見ているホームディレクトリの設定がある。
これは/etc/passwdというファイルに記載があるが、その中に/home/ユーザー名が記載されている列がある。
（IDっぽいところは***でマスクしてます)

~~~ bash
SYSTEM:*:****:****:,****::
    :
k-yokoshima:unused:****:****:****,****:/home/k-yokoshima:/bin/bash
~~~

この場合、/home/k-yokoshimaを/cygdrive/c/Users/k-yokoshimaへ変更すればOK。
これで、sshを試してみる。とりあえずはgithubの鍵設定をして、それでgithubへsshを試してみる。
githubへの秘密鍵は、github_id_rsaという名前で作成してある前提。

~~~ bash
$ vim ~/.ssh/config
Host github.com
	Hostname github.com
	IdentityFile ~/.ssh/github_id_rsa
$ chmod 600 ~/.ssh/github_id_rsa
~~~

現時点ではsshクライアントが入ってないので、apt-cygでインストールする。

~~~ bash
$ apt-cyg install openssh
	:
	:
Package openssh installed
~~~

これでとりあえずgithubへアクセスしてみよう！

~~~ bash
[k-yokoshima]% ssh -T git@github.com
Hi kyokoshima! You've successfully authenticated, but GitHub does not provide shell access.
~~~

これでとりあえず使えるようになったと思う。