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

{% highlight bash %}
$ heroku create [アプリ名] -s cedar -b git://github.com/iphoting/heroku-buildpack-php-tyler.git
{% endhighlight %}

### データベース作成
次に、データベースを使いたいので、ClearDBと言うものを使う。
ClearDBのように、HerokuではWebサーバとして動くもの以外ものもは、アドオンと言う形で使用する事になる。
ClearDBは、デフォルトではigniteと言う無料で5Mのストレージが使えるプランである。
他、Punch, Drift, Screamのように有償のプランがある模様。当然Igniteを使ってみる。

ClearDB MySQL Database | Add-ons | Heroku
<https://addons.heroku.com/cleardb>

使い方は下記の通り。

{% highlight bash %}
$ heroku addons:add cleardb:ignite -a [アプリ名]
{% endhighlight %}

このデータベースの情報をWordpressに設定したいので下記コマンドで、ホスト名、データベース名、ユーザー名、パスワードを調べる。

{% highlight bash %}
$ heroku config -a [アプリ名]
{% endhighlight %}

今回はこう打ってみた。

{% highlight bash %}
$ heroku config -a tmpla
=== tmpla Config Vars
BUILDPACK_URL:        git://github.com/iphoting/heroku-buildpack-php-tyler.git
CLEARDB_DATABASE_URL: mysql://[ユーザ名]:[パスワード]@[ホスト名]/[データベース名]?reconnect=true
DATABASE_URL:         mysql://[ユーザ名]:[パスワード]@[ホスト名]/[データベース名]?reconnect=true
PATH:                 /app/vendor/bin:/app/local/bin:/app/vendor/nginx/sbin:/app/vendor/php/bin:/app/vendor/php/sbin:/usr/local/bin:/usr/bin:/bin
{% endhighlight %}

これは後でWordpressの設定で使用するものなのでどこかへ転記しておく。

### Wordpressのインストール

インストールというか、ソースコードを取ってきてgitを使ってプッシュするということである。
ソースコードは下記からダウンロードした。ちなみに今現在は3.9-ja。

WordPress › 日本語
<http://ja.wordpress.org/>

ここのリンクからzipファイルがダウンロードできるので、それをダウンロードし、
解凍されたディレクトリ（この場合は、wordpress-3.9-ja）をアプリ名（この場合はtmpla）に変更する。
そして、その解凍してリネームしたディレクトリの中の下記ファイルに先ほどのデータベースの情報を書き込む。

{% highlight php %}
 :
$ vim tmpla/wp-config.php

// ** MySQL 設定 - こちらの情報はホスティング先から入手してください。 ** //
/** WordPress のためのデータベース名 */
define('DB_NAME', '[データベース名]');

/** MySQL データベースのユーザー名 */
define('DB_USER', '[ユーザー名]');

/** MySQL データベースのパスワード */
define('DB_PASSWORD', '[パスワード]');

/** MySQL のホスト名 */
define('DB_HOST', '[ホスト名]');
{% endhighlight %}


