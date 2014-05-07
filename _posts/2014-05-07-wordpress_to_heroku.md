---
lauout: post
title: レンタルサーバのWordpressからHerokuへ移行に失敗した話
status: publish
---
## レンタルサーバからの脱却
ロケットネットのレンタルサーバももう５〜６年位になるが、先日６ヶ月まとめ払いの請求が来た。
主にWordpressを使っていたのだが、PHPのバージョンやmemory_limitの制限で苦しんだりしていたので、
何か他のないかなと思っていたらHerokuでWordpressを立ち上げる話を見つけた。

WordPressのブログをherokuで立ち上げて、何かメリットあるの？ | mah365
<http://blog.mah-lab.com/2013/05/01/wordpress-on-heroku/>

## Herokuアプリの作成
### Webアプリ作成
herokuのアカウントは既に持っていたので、あとは作ってみるだけ、の状態で行った。
なので、前述のリンクのStep 1から（ほとんど元記事のままですが・・・）

~~~ bash
$ heroku create [アプリ名] -s cedar -b git://github.com/iphoting/heroku-buildpack-php-tyler.git
~~~

### データベース作成
次に、データベースを使いたいので、ClearDBと言うものを使う。
ClearDBのように、HerokuではWebサーバとして動くもの以外ものもは、アドオンと言う形で使用する事になる。
ClearDBは、デフォルトではigniteと言う無料で5Mのストレージが使えるプランである。
他、Punch, Drift, Screamのように有償のプランがある模様。当然Igniteを使ってみる。

ClearDB MySQL Database | Add-ons | Heroku
<https://addons.heroku.com/cleardb>

使い方は下記の通り。

~~~ sh
$ heroku addons:add cleardb:ignite -a [アプリ名]
~~~

このデータベースの情報をWordpressに設定したいので下記コマンドで、ホスト名、データベース名、ユーザー名、パスワードを調べる。

~~~ sh
$ heroku config -a [アプリ名]
~~~

今回はこう打ってみた。

~~~ sh
$ heroku config -a tmpla
=== tmpla Config Vars
BUILDPACK_URL:        git://github.com/iphoting/heroku-buildpack-php-tyler.git
CLEARDB_DATABASE_URL: mysql://[ユーザ名]:[パスワード]@[ホスト名]/[データベース名]?reconnect=true
DATABASE_URL:         mysql://[ユーザ名]:[パスワード]@[ホスト名]/[データベース名]?reconnect=true
PATH:                 /app/vendor/bin:/app/local/bin:/app/vendor/nginx/sbin:/app/vendor/php/bin:/app/vendor/php/sbin:/usr/local/bin:/usr/bin:/bin
~~~

これは後でWordpressの設定で使用するものなのでどこかへ転記しておく。



