---
layout: post
title: "Rails開発はじめの一歩"
modified: 2015-10-27 23:36:49 +0900
tags: [rails,]
image:
  feature:
  credit:
  creditlink:
comments:
share:
---

「あ、いいこと思いついた」と思って何かWebサービスを作る時、じゃあ具体的にアプリはどうやって作るのかと言う事を書きたいと思います。

# 開発環境の構築
最近開発環境開発環境言ってますけど、細かいことは置いといて開発環境はdockerを使いましょう！
Rails公式のdocker imageがあるのでそれをセットアップします。
Dockerを使えるようにするのは、[Dockerで開発環境を準備してみる]({% post_url 2015-10-20-getting-started-with-docker %})を見てみてください

## docker imageの選び方
[Docker hub](https://hub.docker.com/)と言うところに色んなimageが配置されているので、それをキーワードで検索して返してくれるようなコマンドがあります。
そこに思いつくキーワードを指定して上げれば検索して返してくれます。

~~~bash
$ docker search rails
NAME                       DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
rails                      Rails is an open-source web application fr...   325       [OK]
redmine                    Redmine is a flexible project management w...   40        [OK]
finnlabs/rails             Base image for Rails applications followin...   6                    [OK]
 :
 :
~~~

この場合は一番上のがスターも多いし、OFFICIALにもマークが付いているのでこれを使っておけば良さそうですね。
名前は「rails」と言うimageを使用するのですが、これを指定してdocker runコマンドを実行します。

~~~bash
$ docker run -it --name rails_container -P rails /bin/bash
~~~

これで起動したら、まずはvimが入っていないのでvimをインストールして置きます。
このrailsと言うimageは、debianのようなのでdebianでのパッケージマネージャを使って入れておきます。

~~~bash
$ apt-get update && apt-get install vim
~~~

自分の場合は、これいつもやることになるのでこの状態のコンテナをイメージとして保存しちゃいます。
コンテナを抜けてホストから

~~~bash
$ docker commit rails_container rails:my
~~~

今のrails_containerと言うコンテナをrails:myと言うimageとして保存しておくと。

# アプリ作成
これでもう使える状態となっているので、まずはアプリを作成してみましょう！
サンプルとして適当なのは何が良いのか考えたのですが、やはりブログが結局一番良いのかなと思いました。
理由は

- モデル(post)がすぐ思いつくこと
- 機能として認証を使用するのが自然
- 機能として権限を使用するのが自然
- 画像を使うのが自然
- モデル間のリレーションのバリエーションがある（投稿と投稿者、投稿とカテゴリとか色々思いつく）

なのでブログを作ってみます。
ホームディレクトリ等に移動して、下記コマンドで最初の一歩は終わりですね。

~~~bash
$ rails new blog
~~~

これだけ！あとは最初の画面を確認してみましょう！
rails newでは色んなファイルをどんどん作成してくれるのですが、使いたくない場合もあります。
オプションではそれらファイルの作成をスキップしてくれるような指定をしますが今回は特にしません。

~~~bash
$ cd blog
$ rails server -b 0.0.0.0
~~~

![rails]({{ site_url }}/images/2015/10/27/rails_top.png)

この画面が見れたらまずは完了です。