---
layout: post
title: "PostgreSQLのコミュニティレシピを使ってみた"
modified: 2014-07-04 10:57:47 +0900
tags: [Chef,Postgresql]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---
[PostgreSQLのChefのコミュニティレシピ](http://community.opscode.com/cookbooks/postgresql)を試してみたのでメモします。

## 環境
VirtualBoxのゲスト環境でCentOS6.5です。
ホストもCentOS6.5でChef、knife-solo、Berkshelfが動く状態です。

## 準備
ゲストの設定はこうなっています。

~~~ bash
$ vagrant ssh-config
Host default
  HostName 127.0.0.1
  User vagrant
  Port 2200
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /home/vagrant/.vagrant.d/insecure_private_key
  IdentitiesOnly yes
  LogLevel FATAL
~~~

なのでこう準備します。

~~~ bash
$ knife solo init .
$ knife solo prepare localhost -p 2200 -i ~/.vagrant.d/insecure_private_key
~~~

そして今、nodes/localhost.jsonはこうなってます。

~~~ json
$ vim nodes/localhost.json
{
  "run_list": [

  ],
  "automatic": {
    "ipaddress": "localhost"
  }
}
~~~

Berksfileはこう。

~~~ ruby
source "https://api.berkshelf.com"

cookbook "postgresql"
~~~

で、PostgreSQLのサーバーを入れたいので、run_listにpostgresql::serverを実行させます。
なので、nodes/localhost.jsonはこう書き換えます。

~~~ json
{
  "run_list": [
    "recipe[postgresql::server]"
  ],
  "automatic": {
    "ipaddress": "localhost"
  }
}

~~~

これで実行してみると・・・

~~~ bash
$ knife solo cook localhost -p 2200 -i ~/.vagrant.d/insecure_private_key
Running Chef on localhost...
    :
    (略)
    :
Compiling Cookbooks...
[2014-07-04T14:59:58+00:00] FATAL: You must set node['postgresql']['password']['postgres'] in chef-solo mode. For more information, see https://github.com/opscode-cookbooks/postgresql#chef-solo-note
    :
~~~

パスワードを設定しないとダメらしい。なのでnodes/localhost.jsonに追記します。

~~~ json
{
  "postgresql":{
    "password": "postgres"
  },
  "run_list": [
    "recipe[postgresql::server]"
  ],
  "automatic": {
    "ipaddress": "localhost"
  }
}

~~~

実行してみると。インストールはうまく行ったようですが、いくつか気になる点がログから見れました。

~~~ bash
$ knife solo cook localhost -p 2200 -i ~/.vagrant.d/insecure_private_key
Running Chef on localhost...
  :
  (略)
  :
Recipe: postgresql::client
  * package[postgresql-devel] action install
    - install version 8.4.20-1.el6_5 of package postgresql-devel
  :

  * package[postgresql-server] action install
    - install version 8.4.20-1.el6_5 of package postgresql-server
  :
        -#------------------------------------------------------------------------------
        -# CUSTOMIZED OPTIONS
        -#------------------------------------------------------------------------------
        -
        -#custom_variable_classes = ''          # list of custom variable class names
        +lc_messages = 'en_US.UTF-8'
        +lc_monetary = 'en_US.UTF-8'
        +lc_numeric = 'en_US.UTF-8'
        +lc_time = 'en_US.UTF-8'
  :
~~~

まず、バージョンは9シリーズにしたかったのでした。あと、ロケールっぽい値がen\_US.UTF-8じゃなくて、ja\_JP.UTF-8にしたいところです。

まずはバージョンだけ変更して実行してみました。

~~~ json
{
  :
  "postgresql":{
    "password": "postgres",
    "version": "9.3"
  },
  :
}
~~~

~~~ bash
Recipe: postgresql::client
  * package[postgresql93-devel] action install
    * No version specified, and no candidate version available for postgresql93-devel
================================================================================
Error executing action `install` on resource 'package[postgresql93-devel]'
================================================================================
~~~

まあそうでしょう。そんなパッケージはデフォルトyumリポジトリにないので。

## PostgreSQLのパッケージリポジトリの使用

そこで、rhelのPostgreSQL用のパッケージリポジトリを使ってインストールしてくれるenable\_pgdg\_yumというオプションがあるのでそれを指定します。
(debian系にもenable\_pgdg\_aptというのがあります。)

~~~ json
{
  "postgresql":{
    "enable_pgdg_yum": true,
    "password": "postgres",
    "version": "9.3"
  },
  :
}
~~~

これで大丈夫なはずですが、今回は直接レシピを書いているわけではないので前回の8.4のPostgerSQL関係のものが入った状態になったままです。
なので一度ここでvagrant destroy & upしてからやってみます。
ここで使うためだろう！というところだと思いますが、knife solo bootstrapというのがprepare & cookをやってくれるのでせっかくなので使いましょう。

~~~ bash
$ knife solo bootstrap localhost -p 2200 -i ~/.vagrant.d/insecure_private_key
  : 
  (略)
Recipe: postgresql::server_redhat
  * service[postgresql] action restart
    - restart service service[postgresql]

Running handlers:
Running handlers complete

Chef Client finished, 17/17 resources updated in 41.209762026 seconds
~~~

うまく行きました。ちなみに、8.4が入ったままだと8.4が起動しっぱなしとか、pg_hba.confのauth_methodのpeerが9.1以上と未満だと対応状況が変わってて失敗したりしました。

## ロケール関係の設定

先ほどen\_US.UTF-8になってたところは少なくともja\_JP.UTF-8にしたいので、その設定をlocalhost.jsonにします。


~~~ json
  "postgresql":{
    "enable_pgdg_yum": true,
    "password": "postgres",
    "version": "9.3",
    "config": {
      "lc_messages": "ja_JP.UTF-8",
      "lc_monetary": "ja_JP.UTF-8",
      "lc_numeric": "ja_JP.UTF-8",
      "lc_time": "ja_JP.UTF-8"
    }
  },
~~~

これでもう一回チャレンジしてみました。

~~~ bash
$ knife solo cook localhost -p 2200 -i ~/.vagrant.d/insecure_private_key
~~~

実際のデータベースのシステムカタログを見てみてどうだろ、と思ってみてみましたが、

~~~ bash
# sudo -u postgres psql
psql (9.3.4)
Type "help" for help.

postgres=# SELECT name, setting, context FROM pg_settings WHERE name LIKE 'lc%';
    name     |   setting   |  context
-------------+-------------+-----------
 lc_collate  | en_US.UTF-8 | internal
 lc_ctype    | en_US.UTF-8 | internal
 lc_messages | ja_JP.UTF-8 | superuser
 lc_monetary | ja_JP.UTF-8 | user
 lc_numeric  | ja_JP.UTF-8 | user
 lc_time     | ja_JP.UTF-8 | user
(6 rows)
~~~

まだ甘かった。lc\_collateとlc\_ctypeは直接指定してもpostgresql.confで変更出来ないエラーになります。
そこで、initdb\_localeというオプションを指定します。
現在のnodes/localhost.jsonはこう。

~~~ bash
{
  "postgresql":{
    "enable_pgdg_yum": true,
    "password": "postgres",
    "version": "9.3",
    "config": {
      "lc_messages": "ja_JP.UTF-8",
      "lc_monetary": "ja_JP.UTF-8",
      "lc_numeric": "ja_JP.UTF-8",
      "lc_time": "ja_JP.UTF-8"
    },
    "initdb_locale": "ja_JP.UTF-8"
  },
  "run_list": [
    "recipe[postgresql::server]"
  ],
  "automatic": {
    "ipaddress": "localhost"
  }
}

~~~

すでに前回initdbを一度されているので、面倒くさいのでまたdestroy & up & bootstrapでやります。
その後、システムカタログを見てみると、

~~~ bash
$ sudo -u postgres psql
psql (9.3.4)
Type "help" for help.

postgres=# SELECT name, setting, context FROM pg_settings WHERE name LIKE 'lc%';
    name     |   setting   |  context
-------------+-------------+-----------
 lc_collate  | ja_JP.UTF-8 | internal
 lc_ctype    | ja_JP.UTF-8 | internal
 lc_messages | ja_JP.UTF-8 | superuser
 lc_monetary | ja_JP.UTF-8 | user
 lc_numeric  | ja_JP.UTF-8 | user
 lc_time     | ja_JP.UTF-8 | user
(6 rows)
~~~

おお、狙ったとおりになった。



