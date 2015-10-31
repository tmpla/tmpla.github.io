---
layout: post
title: "Composerを使ってCakePHPをインストールする"
modified: 2014-06-27 10:38:01 +0900
tags: [Composer,CakePHP]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---

PHPでのライブラリ・パッケージ管理には昔から[PEAR](http://pear.php.net/manual/ja/)（ぴあ～？）、[PECL](http://pecl.php.net/)（ぴくる）
などがあるが、イマイチ便利に使えていなかった。

しかし最近、NodeJSの[npm](https://www.npmjs.org/)のような感じで割と簡単に使えるPHPのパッケージ管理ツールである[Composer](https://getcomposer.org/)というのが良さそうなのでこれを使ってCakePHPをインストールしてみた。

PHPの動作環境は

~~~ bash
$ php -v
PHP 5.5.13 (cli) (built: Jun  7 2014 15:38:22)
Copyright (c) 1997-2014 The PHP Group
Zend Engine v2.5.0, Copyright (c) 1998-2014 Zend Technologies
~~~

で、CakePHPをインストールするのにおそらく、php-mcrypt, php-xml、あとmysqlを使うのでphp-mysqlをインストール済みである。
OSはCentOS6.5で、VirtualBoxをVagrantのprivate_networkで動作させている。

## Composerインストール

[Getting Started](https://getcomposer.org/doc/00-intro.md)に記載があるのでそのままだが、

~~~ bash
$ curl -sS https://getcomposer.org/installer | php
~~~

で、composer.pharというファイルが作成されるので、それを

~~~ bash
$ php composer.phar --version
~~~

という感じで使用すれば良い。
システム全体で使用するのであれば

~~~ bash
$ sudo mv composer.phar /usr/local/bin/composer
~~~

のように、PATHの通っているところへ移動させて

~~~ bash
$ composer --version
~~~

と使うのもよし。今回はこうした。

## CakePHPのインストール

Composerで扱えるパッケージは、[Packagist](https://packagist.org/)というところからダウンロードして使うのがデフォルト設定である。
パッケージ名は、vendor名/パッケージ名で一つ、という感じのようだ。
これでCakePHPをインストールしてみる。

~~~ bash
$ composer create-project cakephp/cakephp
Installing cakephp/cakephp (2.5.2)
  - Installing cakephp/cakephp (2.5.2)
    Loading from cache

Created project in /home/vagrant/cakephp
Loading composer repositories with package information
Installing dependencies (including require-dev)
  - Installing composer/installers (v1.0.15)
    Loading from cache

  - Installing symfony/yaml (v2.5.0)
    Loading from cache

  - Installing phpunit/php-text-template (1.2.0)
    Loading from cache

  - Installing phpunit/phpunit-mock-objects (1.2.3)
    Loading from cache

  - Installing phpunit/php-timer (1.0.5)
    Loading from cache

  - Installing phpunit/php-token-stream (1.2.2)
    Loading from cache

  - Installing phpunit/php-file-iterator (1.3.4)
    Loading from cache

  - Installing phpunit/php-code-coverage (1.2.17)
    Loading from cache

  - Installing phpunit/phpunit (3.7.37)
    Loading from cache

  - Installing cakephp/debug_kit (2.2.3)
    Loading from cache

phpunit/phpunit-mock-objects suggests installing ext-soap (*)
phpunit/php-code-coverage suggests installing ext-xdebug (>=2.0.5)
phpunit/phpunit suggests installing phpunit/php-invoker (~1.1)
Writing lock file
Generating autoload files

~~~

これで、「cakephp」というディレクトリの中に、色々配置されたと思うので見てみる。

~~~ bash
$ ls -la cakephp/
total 104
drwxrwxr-x  8 vagrant vagrant  4096 Jun 27 03:10 .
drwx------  6 vagrant vagrant  4096 Jun 27 03:10 ..
drwxrwxr-x 14 vagrant vagrant  4096 Jun 27 03:10 app
-rw-rw-r--  1 vagrant vagrant   174 Jun 27 03:10 build.properties
-rw-rw-r--  1 vagrant vagrant 10315 Jun 27 03:10 build.xml
-rw-rw-r--  1 vagrant vagrant   705 Jun 27 03:10 composer.json
-rw-rw-r--  1 vagrant vagrant 19535 Jun 27 03:10 composer.lock
-rw-rw-r--  1 vagrant vagrant  3252 Jun 27 03:10 CONTRIBUTING.md
-rw-rw-r--  1 vagrant vagrant   311 Jun 27 03:10 .editorconfig
-rw-rw-r--  1 vagrant vagrant   730 Jun 27 03:10 .gitattributes
-rw-rw-r--  1 vagrant vagrant   333 Jun 27 03:10 .gitignore
-rw-rw-r--  1 vagrant vagrant   139 Jun 27 03:10 .htaccess
-rw-rw-r--  1 vagrant vagrant  1454 Jun 27 03:10 index.php
drwxrwxr-x  3 vagrant vagrant  4096 Jun 27 03:10 lib
drwxrwxr-x  3 vagrant vagrant  4096 Jun 27 03:10 Plugin
drwxrwxr-x  2 vagrant vagrant  4096 Jun 27 03:10 plugins
-rw-rw-r--  1 vagrant vagrant  1877 Jun 27 03:10 README.md
-rw-rw-r--  1 vagrant vagrant  4037 Jun 27 03:10 .travis.yml
drwxrwxr-x  6 vagrant vagrant  4096 Jun 27 03:10 vendor
drwxrwxr-x  2 vagrant vagrant  4096 Jun 27 03:10 vendors
~~~

せっかくなので動かしてみよう。
まず、timezoneが設定されていないとすごい勢いでNoticeが出るので、php.iniの下記値を設定する。

~~~ bash
date.timezone = Asia/Tokyo
~~~

また、app/Config/core.phpの下記値を初期値じゃないものに変更する。（適当にした）

~~~ bash
/**
 * A random string used in security hashing methods.
 */
        Configure::write('Security.salt', 'tekitou');

/**
 * A random numeric string (digits only) used to encrypt/decrypt strings.
 */
        Configure::write('Security.cipherSeed', 'tekitou');

~~~

続いて、app/Config/database.php.exampleをapp/Config/database.phpにリネームする。
また、mysql側でその設定に合わせて作成する。

~~~ bash
$ mysql -uroot
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 6
Server version: 5.5.38 MySQL Community Server (GPL)

Copyright (c) 2000, 2014, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.  

mysql> create database test_database_name default character set utf8;
Query OK, 1 row affected (0.00 sec)

mysql> grant all on database_name.* to user@localhost identified by 'password';
Query OK, 0 rows affected (0.00 sec)

mysql> grant all on test_database_name.* to user@localhost identified by 'password';
Query OK, 0 rows affected (0.01 sec)

~~~

この状態で、cakephp/app/webrootに移動してphpのビルトインサーバで動かしてみる。
VirtualBoxゲストの場合、実行時ホスト名を0.0.0.0にしないとホストからアクセス出来ないのでそうする。

~~~ bash
$ cd cakephp/app/webroot
$ php -S 0.0.0.0:8000
~~~

すると出た。

![CakephpTOP]({{ site_url }}/images/06/27/cakephp-top.png)

これだけだったら、git cloneとかでもいいじゃん、と思うかもしれないが、Composerがいいのはここからなのであった。