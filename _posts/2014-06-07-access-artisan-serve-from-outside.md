---
layout: post
title: "laravelのartisan serveで起動したビルトインサーバに外部からアクセスする方法"
modified: 2014-06-07 17:20:02 +0900
tags: [laravel,php,virtualbox]
image:
  feature:
  credit:
  creditlink:
comments:
share: true
---
laravelのアプリを作成して、phpのビルトインサーバを使って起動するにはartisan(laravel.jpではアルチザンと読むことに決まった) serveと言うコマンドを使って起動する。

~~~ bash
# php artisan serve
Laravel development server started on http://localhost:8000
~~~

これをvagrantを使って仮想マシンに配置し、private_network(192.168.33.10がコメント化されているのでそれを使う)にして、
ホストOSをからアクセスするには、http://192.168.33.10:8000にブラウザでアクセスすれば良いかと思ったが出来なかった。

started on http://**localhost**:8000のlocalhostが気になったのでググってみたら

[apache 2.2 - why isn't laravel's php artisan serve server being accessible from the WWW on IIS - Server Fault](http://serverfault.com/questions/581529/why-isnt-laravels-php-artisan-serve-server-being-accessible-from-the-www-on-ii)

と言うのを見つけたので、下記のようにして見たら行けた、と言う話でした。

~~~ bash
# php artisan serve --host 0.0.0.0
Laravel development server started on http://0.0.0.0:8000
~~~

以上、よろしくお願いします。
