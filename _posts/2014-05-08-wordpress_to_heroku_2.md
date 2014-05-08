---
lauout: post
title: レンタルサーバのWordpressからHerokuへ移行に失敗した話その２
status: draft
published: false
---
このなかの、ユーザ名、パスワード、ホスト名、データベース名をwp-config.phpの下記の場所へ転記する。

{% highlight php %}
 :
// ** MySQL 設定 - こちらの情報はホスティング先から入手してください。 ** //
/** WordPress のためのデータベース名 */
define('DB_NAME', '[データベース名]');

/** MySQL データベースのユーザー名 */
define('DB_USER', '[ユーザ名]');

/** MySQL データベースのパスワード */
define('DB_PASSWORD', '[パスワード]');

/** MySQL のホスト名 */
define('DB_HOST', '[ホスト名]');
 :
 aaaa
{% endhilight %}