---
layout: post
title: "Dockerで開発環境を準備してみる"
modified: 2015-10-20 21:14:43 +0900
tags: [docker]
image:
  feature:
  credit:
  creditlink:
comments:
share:
---

# 開発用仮想環境

ローカルに開発環境を整える場合、どんな感じで準備するでしょうか。
普通は個人が主に使用しているPCは1台とかで、そのマシンで動作確認ができるように環境を作成してもいいと思いますが、
開発内容が変われば開発環境も切り替える必要がありそのたびにローカル開発環境に気を使うのももったいないかもしれません。
特にサーバサイドのアプリケーションの場合は色んなところに気を使いますね。

なので最近では使用しているパソコンの中に仮想マシンを起動させて、その中で開発をして色んな開発環境を切り替える手段があります。
VirtualBoxやVMWareが有名ですが、その仲間といえば仲間にあたるDockerと言うのを使って環境を作ってみることにしました。
VirtualBox等と何が違うかと言うのはあるのですが、とりあえず仮想環境として使えるようなモノで軽い！と言う風に思っておけばいいと思います。
そんで、これはおそらくなんですけどDockerはLinuxのLXCと言う機能をベースにしてるのですがMacにはなく、なのでVirtualBoxでLinuxを立ち上げてその中にあるDockerコンテナを使い、Mac版はそのDockerのラッパーみたいなもんだと思うんですけどあまり詳しいことはまだ調べてません。
セットアップする環境はMac（El Capitan)で下記のような環境です。

~~~ bash
$ sw_vers
ProductName:	Mac OS X
ProductVersion:	10.11
BuildVersion:	15A284
~~~

# Dockerインストール

Mac版のインストールは[Get Started with Docker](https://docs.docker.com/mac/started/)に書いてある通りに実行します。
自分の環境の場合は、VirtualBoxは既に入っているのでわからないのですが、多分VirtualBoxも一緒にインストールされるのかな？
もし必要なら[VirtualBox](https://www.virtualbox.org/)もインストールしときましょう。

まず[Docker Toolbox | docker](https://www.docker.com/toolbox)から「Download(Mac)」の方のボタンを押してDockerToolbox-x.x.x.pkgファイルをダウンロード・インストールします。
インストールが終わったら、「Docker Quickstart Terminal」をポチッと押してコンソールを開きます。
コンソールを開く時に、自分の場合はiTermを使用しているのでそれを使うかどうかを聞いてきますが使いたい場合はそれを選択します。
このQuickstart Terminalは何してくれてるかというと、Dockerで使用するVirtualBoxのVMの設定とコンソール起動時の環境変数の設定などをしてくれているようです。
今回はとりあえず使いたいだけなので深くは触れないです。
Kitematicと言うのは、GUIでDockerの操作を出来るツールで便利そうですがとりあえず今は触れません。

ちなみに、これらボタンはインストールすればLaunchpadにありますんで次からはそれを使いましょう。

![installer]({{ site_url }}/images/20151020/docker-installer.png)

起動したら、こんな感じでDockerのクジラ？のAAが表示されるのでそれを噛み締めます。
![whale]({{ site_url }}/images/20151020/docker-whale.png)

あとは、これからよく使うdocker-machine、dockerコマンドでバージョンでも確認してみましょう。

~~~bash
$ docker-machine -v
docker-machine version 0.4.1 (e2c88d6)
$ docker -v
Docker version 1.8.3, build f4bf5c7
~~~

# imageの取得
これでめでたくdockerが使える環境になったので、早速仮想環境を使いましょう。
dockerはイメージとコンテナと言うものがありますが、その違いはなんでしょうか。

イメージはもう使用できる状態になった環境を保存したもので、まずはこれをどこからか調達してきます。殆どの場合は、[Docker Hub](https://hub.docker.com/)と言うところに置いてあるものを使えばいいと思います。
ここには色んな人達や色んな団体が作成したdocker imageが置いてあります。

そしてコンテナと言うものですが、これはイメージを持ってきてローカルで起動したものです。
このコンテナにログインして開発やら何やらを行うことになります。
このコンテナをイメージとして保存したりする事もできます。

docker hubに起動確認用？のimageとしてhello worldと言うのがあるのでそれを起動してみます。

~~~bash
$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world

b901d36b6f2f: Pull complete
0a6ba66e537a: Pull complete
Digest: sha256:517f03be3f8169d84711c9ffb2b3235a4d27c1eb4ad147f6248c8040adb93113
Status: Downloaded newer image for hello-world:latest

Hello from Docker.
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker Hub account:
 https://hub.docker.com

For more examples and ideas, visit:
 https://docs.docker.com/userguide/
~~~

このhello-worldと言うイメージは単に上記のようなデスクリプションを表示するだけのようなのですが、
コマンドの使い方は

~~~bash
$ docker (サブコマンド) (オプション)
~~~

みたいな感じでやります。

# dockerコマンド抜粋
dockerコマンドは「docker」と打つとヘルプが表示されるので詳しくはそちらを見ると良いですが、よく使うやつを抜粋します。

## コンテナ起動
~~~bash
$ docker run -it --name (コンテナ名) (イメージ名) (起動シェル)
~~~
-itはおまじないみたいなもんです。--nameはコンテナに名前をつけとくと後々操作が楽なので。
起動シェルと言うのは起動シェルというか起動時に起動するプログラムというのが正しいのかも。まあほぼほぼシェルを起動すると思います。

あと、サーバサイドのアプリケーションを開発する場合なんかはポート番号の変換の指定をここでします。
今回使用しているDocker Toolboxの場合はVirtualBoxで起動しているマシンを仮想的なDockerのホストとして使ってます。
本当のホスト（Mac)からは仮想ホストへ向けてアクセスしますが、その仮想ホストの中にあるDockerコンテナのポートへアクセスするための指定です。
-Pで勝手にポートのマッピングがされてくれるはずなんですが、うまくいかないようでしたので直接ポートを指定する方法を書きます。
例えばRailsの開発の場合で3000番ポートを使うとすると

~~~bash
$ docker run -it --name (コンテナ名) -p 3000:3000 (イメージ名) (起動シェル)
~~~

のように、-p (仮想ホストのポート番号):(コンテナのポート番号)みたいに指定します。
そして、仮想ホストのIPアドレスが「192.168.99.100」だったとするならばブラウザでは
http://192.168.99.100:3000
にアクセスして動作を確認することが出来ます。

仮想ホストのIPアドレスは起動時にも出ますし、下記コマンドで確認することも出来ます。

~~~bash
$ docker-machine ip default
~~~

## コンテナ一覧
~~~bash
$ docker ps -a
~~~
「-a」オプションを付けると終了したコンテナも見ることが出来ます。
dockerのコンテナに入ってexitするとそのコンテナは終わってしまうのでよく使うかも。

## コンテナ起動
~~~bash
$ docker start (コンテナ名|コンテナID)
~~~
終了してしまったコンテナを又使う場合は起動しないといけないです。そのためのコマンドです。
ここでコンテナ名をつけとくと楽ですが、docker psでコンテナIDも見れるのでそれをお好みで。

## コンテナへアタッチ
~~~bash
$ docker attach (コンテナ名|コンテナID)
~~~
起動中のコンテナへまたログインしたい場合に使います。起動してないと使えませんので終了しているコンテナならdocker startした後に。

## コンテナ削除
~~~bash
$ docker rm (コンテナ名|コンテナID)
~~~
スコンとコンテナが作成できるので、場合によってはどんどん同じイメージからコンテナを作ってしまう時があります。
それを削除するためのコマンドです。起動中だと削除できません。

## イメージ一覧確認
~~~bash
$ docker images
~~~
何もない状態でdocker runした場合には、まずimageがローカル環境にダウンロードされ保存されています。
その保存されたイメージの一覧を確認できます。

## イメージの保存
~~~bash
$ docker pull (イメージ名)
~~~
明示的にイメージをローカルに持ってくる時に使います。

# railsの環境を試してみる。
では、rails公式のイメージがあるのでそれを持ってきてアプリケーションを作ってみて繋がるか確認してみましょう！

~~~bash
$ docker run -it --name rails_container -p 3000:3000 rails /bin/bash                                                                                                                             2.2.3
Unable to find image 'rails:latest' locally
latest: Pulling from library/rails
7a42f1433a16: Pull complete
3d88cbf54477: Pull complete
41d087dfc152: Pull complete
e2c9f36da890: Pull complete
c410d64f4834: Pull complete
072795d5b574: Pull complete
f9985046e6d0: Pull complete
dddb08b35f8f: Pull complete
b5d817cabfa8: Pull complete
60fe69178855: Pull complete
c7713e831deb: Pull complete
8f75e2913294: Pull complete
20ca91477a26: Pull complete
7193f9b09dfe: Pull complete
d4ff40e1f510: Pull complete
8979ed397e25: Pull complete
d8c20d2a1c09: Pull complete
acdbf8c0b76d: Pull complete
5210f37ce86e: Pull complete
4df3ae0ce68b: Pull complete
aa314ec8dd11: Pull complete
Digest: sha256:d354b7c0b339105f9a4747e3eb4d27514232a86c9d3c8df9785c87cd040637ac
Status: Downloaded newer image for rails:latest
root@f7e7a95c05d3:/#
~~~

ダウンロードがそれなりに時間掛かりますが（と言っても5分くらいだった）これでもうログインされちゃってる！
何のOSなのか確認してみよう。

~~~bash
root@f7e7a95c05d3:/# cat /etc/issue
Debian GNU/Linux 8 \n \l
~~~

Debianみたいですな。あとRubyとRailsとMySQLは入ってるかな〜？

~~~bash
root@f7e7a95c05d3:/# rails -v
Rails 4.2.4
root@f7e7a95c05d3:/# ruby -v
ruby 2.2.3p173 (2015-08-18 revision 51636) [x86_64-linux]
root@f7e7a95c05d3:/# mysql --version
mysql  Ver 14.14 Distrib 5.5.44, for debian-linux-gnu (x86_64) using readline 6.3
~~~

うむバッチリのようです。ではサンプル的にアプリを作ってみる。

~~~bash
root@f7e7a95c05d3:/# cd ~
root@f7e7a95c05d3:~# rails new example
      create
      create  README.rdoc
      create  Rakefile
      create  config.ru
      create  .gitignore
      :
      :
~~~

ここもbundle installのとこがそれなりに時間かかるけど起動して動作確認してみよう。

~~~bash
root@f7e7a95c05d3:~# cd example/
root@f7e7a95c05d3:~/example# rails s -b0.0.0.0
=> Booting WEBrick
=> Rails 4.2.4 application starting in development on http://0.0.0.0:3000
=> Run `rails server -h` for more startup options
=> Ctrl-C to shutdown server
[2015-10-20 14:04:32] INFO  WEBrick 1.3.1
[2015-10-20 14:04:32] INFO  ruby 2.2.3 (2015-08-18) [x86_64-linux]
[2015-10-20 14:04:32] INFO  WEBrick::HTTPServer#start: pid=279 port=3000
~~~

これで、
http://192.168.99.100:3000
へアクセスしてみると。

![rails]({{ site_url }}/images/20151020/rails.png)

おお、凄い。環境作るのに10分かかってない。

あとはdockerのイメージとコンテナを確認してみよう。
起動させたままコンテナから抜けるにはctrl+p - ctrl+qで抜けれるのでそれで抜けて、ホスト側から確認。

~~~bash
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
rails               latest              aa314ec8dd11        5 days ago          825 MB
hello-world         latest              0a6ba66e537a        6 days ago          960 B
~~~
~~~bash
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES
f7e7a95c05d3        rails               "/bin/bash"         12 minutes ago      Up 12 minutes       0.0.0.0:3000->3000/tcp   rails_container
~~~

うむ、完璧である。
というわけで、開発環境にdocker、オススメです。