---
lauout: post
title: レンタルサーバのWordpressからHerokuへ移行に失敗した話
status: publish
---
## レンタルサーバからの脱却

ロケットネットのレンタルサーバももう５〜６年位になるが、先日６ヶ月まとめ払いの請求が来た。
主にWordpressを使っていたのだが、PHPのバージョンやmemory_limitの制限で苦しんだりしていたので、
何か他のないかなと思っていたらHerokuでWordpressを立ち上げる話を見つけた。

[WordPressのブログをherokuで立ち上げて、何かメリットあるの？ \| mah365](http://blog.mah-lab.com/2013/05/01/wordpress-on-heroku/)

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
[ClearDB MySQL Database \| Add-ons \| Heroku](https://addons.heroku.com/cleardb)

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
そして、その解凍してリネームしたディレクトリを初期コミットとしておこう。

{% highlight bash %}
$ cd tmpla
$ git init .
$ git add .
$ git commit -m "First commit"
{% endhighlight %}

また、リモートリポジトリの設定もするが、これはherokuのコマンドでラップしてくれているようなのでそれを使う。

{% highlight bash %}
$ heroku git:remote -a tmpla
$ git remote -v
heroku	git@heroku.com:tmpla.git (fetch)
heroku	git@heroku.com:tmpla.git (push)
{% endhighlight %}

こうなっていればOK

### Database設定

中のwp-config.phpと言うファイルがWordpressの設定ファイルである。
一番最初は存在しないかと思うが、wp-config-sample.phpと言う例があるのでそれをコピーして使う。
これに先ほどのデータベースの情報を書き込む。

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
 :
{% endhighlight %}

また、認証キーの情報も生成しておく。
wp-config.phpのコメントに書いてあるが、
https://api.wordpress.org/secret-key/1.1/salt/

にアクセスすれば、ランダムな値を設定した該当部分のテキストが生成されるのでそれを使うのが楽だろう。

ここまで
[mah365](http://blog.mah-lab.com/2013/05/01/wordpress-on-heroku/)さんのブログと同じすぎるがあちらが本筋です。

### Webサーバの設定
元記事でもnginxを使っているので、それを真似る。
ちょっとだけ調べたところ、このbuildpackでは、conf/nginx.conf.erbと言うファイルがあればそれを使うようで、
なければデフォルトのを使うようである。
これは作っておけばカスタマイズできるので作っておいたほうが良さそう。

{% highlight yaml %}
# setting worker_processes to CPU core count
worker_processes      1;
daemon off;

events {
  worker_connections  1024;
}

http {
  include             mime.types;
  default_type        application/octet-stream;
  sendfile            on;
  server_tokens       off;
  keepalive_timeout   65;
  access_log          off;
  error_log           logs/error.log;
  proxy_max_temp_file_size  0;
  #fastcgi_max_temp_file_size  0;
  limit_conn_zone $binary_remote_addr zone=phplimit:1m; # define a limit bucket for PHP-FPM
  # don't use server listen port in redirects.
  port_in_redirect off;

  # set $https only when SSL is actually used.
  map $http_x_forwarded_proto $my_https {
    default off;
    https on;
  }

  upstream php_fpm {
    server unix:/tmp/php-fpm.socket;
  }

  root              /app/;
  index             index.php index.html index.htm;

  server {
    listen            <%= ENV['PORT'] %>;
    server_name       _;

    # Some basic cache-control for static files to be sent to the browser
    location ~* \.(?:ico|css|js|gif|jpeg|jpg|png)$ {
      expires         max;
      add_header      Pragma public;
      add_header      Cache-Control "public, must-revalidate, proxy-revalidate";
    }

    # Deny hidden files (.htaccess, .htpasswd, .DS_Store).
    location ~ /\. {
      deny            all;
      access_log      off;
      log_not_found   off;
    }

    # Deny /favicon.ico
    location = /favicon.ico {
      access_log      off;
      log_not_found   off;
    }

    # Deny /robots.txt
    location = /robots.txt {
      allow           all;
      log_not_found   off;
      access_log      off;
    }

    # Status. /status.html uses /status
    location ~ ^/(status|ping)$ {
      include         fastcgi_params;
      fastcgi_param   HTTPS $my_https if_not_empty;
      fastcgi_pass    php_fpm;
      fastcgi_param   SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    location /server-status {
      stub_status on;
      access_log   off;
    }

    location / {
      # wordpress fancy rewrites
      if (-f $request_filename) {
        break;
      }
      if (-d $request_filename) {
        break;
      }

      rewrite         ^(.+)$ /index.php?q=$1 last;

      # Add trailing slash to */wp-admin requests.
      rewrite         /wp-admin$ $scheme://$host$uri/ permanent;

    #  # redirect to feedburner.
    #  # if ($http_user_agent !~ FeedBurner) {
    #  #   rewrite ^/feed/?$ http://feeds.feedburner.com/feedburner-feed-id last;
    #  # }
    }

    include /app/conf/nginx.d/*.conf;

    location ~ .*\.php$ {
      try_files $uri =404;
      limit_conn phplimit 5; # limit to 5 concurrent users to PHP per IP.
      include         fastcgi_params;
      fastcgi_param   HTTPS $my_https if_not_empty;
      fastcgi_pass    php_fpm;
      fastcgi_param   SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
  }
}
{% endhighlight %}

ここまでで、コミットしてWordpressを起動する

{% highlight bash %}
$ git add .
$ git commit -m "Wordpress初期設定完了！"
$ git push heroku master
$ heroku open -a tmpla
{% endhighlight %}

ふう・・・（長い）

### 既存データのインポート

Wordpressにはデフォルト（か、デファクトスタンダード）でインポート／エクスポートプラグインが付いているはず。
これを使って既存サイトからデータをエクスポートし、新しい（herokuの）サイトへインポートしたい。

ただし、エクスポートされるのはテキストのものだけなので、記事内で写真を使ったりしている場合はまたそれも考えなければいけないが、
それは単にwp-content/uploadsの内容をコピーすればいいだけで済むはずなのでそれは置いとく。
ちょっと日本語だと記事が古いが、エクスポートの方法はこちらに書いてある。

[管理画面/ツール/エクスポート - WordPress Codex 日本語版](http://wpdocs.sourceforge.jp/Tools_Export_SubPanel)

詳細な手順はこれの通りエクスポートし、heroku側のWordpressでインポートしていたのだが、
途中で「このページは見つかりません」のようなページへ飛んでしまっている。
なんでかなぁ、とログを見てみるとこんな記述を見つけた。
ちなみにherokuでログをみるには

{% highlight bash %}
$ heroku logs
{% endhighlight %}

で見れる。

{% highlight bash %}
client intended to send too large body: 2037815 bytes,
{% endhighlight %}

これはどうもnginxの設定で、リクエスト内容がデカすぎるのが問題だそうでした。
ちなみに、エクスポートしたWXR(Wordpress eXtended RSS)は2M位でぴったり。

[nginxで"client intended to send too large body"が発生した時の対策方法 - Qiita](http://qiita.com/notanota/items/4816ad71b90a9967fa18)

なので、先ほど作っておいた、tmpla/conf/nginx.conf.erbを変更してプッシュします。

{% highlight bash %}
 :
  server {
    listen            <%= ENV['PORT'] %>;
    server_name       _;

+   client_max_body_size 100M;
 :
{% endhighlight %}

じゃあ、これで大丈夫だね！と思って再度実行したら、今度は「ページが見つかりません」には行かず、
順調に進捗している様子。だが、途中で処理が止まっているようにも見えるが…。
そこでまた、ログを確認してみると。

{% highlight bash %}
User '[ユーザ名]' has exceeded the 'max_questions' resource
{% endhighlight %}

と出ていてそれ以上処理が出来ていないようだった。
そこで調べると

[mysql - User 'root' has exceeded the 'max_questions' resource - Stack Overflow](http://stackoverflow.com/questions/13836772/user-root-has-exceeded-the-max-questions-resource)

と言うのがあったが、さすがにClearDB:igniteでそんなに設定出来るもんかなと思って調べたらできなさそうだった。
なのでエクスポート／インポートではここであきらめた。

### mysqldumpでのリストア

ツールでは無理だが、直接MySQLに入れてしまえば良いかなと思って考えたが、現在使っているレンタルサーバはシェルを許可していなかった。
が、phpMyAdminが使えるようなので、それを使ってmysqldumpは出来そうだ。
でやってみたが、サイズが110Mもある。なぜ個人のブログでこんなサイズになるのかレコード数を調べてみたら、
どうもfirestatsと言うWordpressでのアクセス解析ツールのログが凄いことになっているようだった。(PV毎のレコードなので)
この時知らなかったが、ClearDB:igniteのDBサイズは5Mなのでそもそも入る理由がなかったが、
ClearDBへのホスト、アカウント情報などは前述で調べていたので一応やってみた。

結果、ClearDBのquotaのアラートがずっと来るようになった。今は後悔している。
