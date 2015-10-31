---
layout: post
title: "Ubuntuに最新PHPをインストールする方法"
modified: 2014-06-13 19:40:04 +0900
tags: [php,ubuntu,apt-get]
image:
  feature:
  credit:
  creditlink:
comments:
share: true
---

php最新版をubuntuにインストールする方法を忘れたのでメモ。
オプションで、なぜか5.5.9が/usr/local/bin/phpにあたってる状態です。なんでこれ覚えてないのかな…。

OSバージョンはhashicorp/precise64のvagrantのboxで以下

~~~ bash
vagrant@precise64:/etc/apt$ cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=12.04
DISTRIB_CODENAME=precise
DISTRIB_DESCRIPTION="Ubuntu 12.04.4 LTS"
~~~

laravelのサンプルをubuntu上で作っているのだが、laravelはphp5.4以降じゃないとダメなのである。
一方、apt-cache show php5で見るバージョンは5.3.10なので、何とかしないといけない。
今のところ入っているphpは5.5.9なので、どうにかして入れたんだろうが、どう入れたかが思い出せないので初心に帰って調べてみた。

というか、ググったらドンピシャの記事があったのでそれにインスパイア(パクリ)と言うことですいません。

[How to install/setup latest version of PHP 5.5 on Ubuntu 12.04 LTS (Precise Pangolin) - Dev Metal](http://www.dev-metal.com/how-to-setup-latest-version-of-php-5-5-on-ubuntu-12-04-lts/)

まず、apt-getで入れたいところだが、debianパッケージにもそのパッケージを置くリポジトリというものがあるようだ。
通常は標準リポジトリで良いのだと思うが、前衛的な奴はリポジトリを追加してあげれば良い（rhelのepelみたいな感じだと思う）

これは、add-apt-repositoryと言うコマンドがあるようなのだが、このコマンドの補完が効かないので入っていないのだろう。
python-software-propertiesと言うパッケージのコマンドのようなのでそれをまず入れる。
あと、apt-get自体の更新もしとく。

~~~ bash
$ sudo apt-get update
$ sudo apt-get install python-software-properties
~~~

これでadd-apt-repositoryが使えるようになったので、ppa:ondrej/php5と言うリポジトリをありがたく使わせて頂く。ドイツの人かな？

[Ondřej Surý in Launchpad](https://launchpad.net/~ondrej)

~~~ bash
$ sudo add-apt-repository ppa:ondrej/php5
You are about to add the following PPA to your system:
 This branch follows latest PHP packages as maintained by me & rest of the Debian pkg-php team.

You can get more information about the packages at https://sury.org

If you need to stay with PHP 5.4 you can use the oldstable PHP repository:

    ppa:ondrej/php5-oldstable

PLEASE READ: If you like my work and want to give me a little motivation, please consider donating: https://deb.sury.org/pages/donate.html
 More info: https://launchpad.net/~ondrej/+archive/php5
Press [ENTER] to continue or ctrl-c to cancel adding it

gpg: keyring `/tmp/tmpsqW8Kd/secring.gpg' created
gpg: keyring `/tmp/tmpsqW8Kd/pubring.gpg' created
gpg: requesting key E5267A6C from hkp server keyserver.ubuntu.com
gpg: /tmp/tmpsqW8Kd/trustdb.gpg: trustdb created
gpg: key E5267A6C: public key "Launchpad PPA for Ond\xc5\x99ej Sur?" imported
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1)
OK
~~~

お、おう。なんかよくわかんないけどリポジトリ追加されたのだろう…。

これでapt-get updateをする。

~~~ bash
$ sudo apt-get update
~~~

そんで、apt-cache show php5をやってみると、新しいphpのバージョンが見れる。

~~~ bash
vagrant@precise64:/etc/apt$ apt-cache show php5
Package: php5
Priority: optional
Section: php
Installed-Size: 29
Maintainer: Debian PHP Maintainers <pkg-php-maint@lists.alioth.debian.org>
Architecture: all
Version: 5.5.12+dfsg-2+deb.sury.org~precise+1
Depends: libapache2-mod-php5 (>= 5.5.12+dfsg-2+deb.sury.org~precise+1~) | libapache2-mod-php5filter (>= 5.5.12+dfsg-2+deb.sury.org~precise+1~) | php5-cgi (>= 5.5.12+dfsg-2+deb.sury.org~precise+1~) | php5-fpm (>= 5.5.12+dfsg-2+deb.sury.org~precise+1~), php5-common (>= 5.5.12+dfsg-2+deb.sury.org~precise+1~)
Filename: pool/main/p/php5/php5_5.5.12+dfsg-2+deb.sury.org~precise+1_all.deb
Size: 1234
MD5sum: 936bef445630d9daf1074504af969d06
SHA1: e8d90e1da4032ad4e083a51166cf06d88737e349
SHA256: f43fd17b01302ffff8b65ea948eb23fe745d91340044ebf4d7ad513ac95bd7f2
Description: server-side, HTML-embedded scripting language (metapackage)
 This package is a metapackage that, when installed, guarantees that you
 have at least one of the four server-side versions of the PHP5 interpreter
 installed. Removing this package won't remove PHP5 from your system, however
 it may remove other packages that depend on this one.
 .
 PHP (recursive acronym for PHP: Hypertext Preprocessor) is a widely-used
 open source general-purpose scripting language that is especially suited
 for web development and can be embedded into HTML.

Package: php5
Priority: optional
Section: php
Installed-Size: 21
Maintainer: Ubuntu Developers <ubuntu-devel-discuss@lists.ubuntu.com>
Original-Maintainer: Debian PHP Maintainers <pkg-php-maint@lists.alioth.debian.org>
Architecture: all
Version: 5.3.10-1ubuntu3.11
Depends: libapache2-mod-php5 (>= 5.3.10-1ubuntu3.11) | libapache2-mod-php5filter (>= 5.3.10-1ubuntu3.11) | php5-cgi (>= 5.3.10-1ubuntu3.11) | php5-fpm (>= 5.3.10-1ubuntu3.11), php5-common (>= 5.3.10-1ubuntu3.11)
Filename: pool/main/p/php5/php5_5.3.10-1ubuntu3.11_all.deb
Size: 1076
MD5sum: 27093858ac6d275faaea06bad0484303
SHA1: a2ca78bbeec85581aefdd6c2d3a2d214cdf410f9
SHA256: d723656ba17e2bb7020ef36c11c2a888265a080b7ebc6e87782cc40b50bbdd1d
Description-en: server-side, HTML-embedded scripting language (metapackage)
 This package is a metapackage that, when installed, guarantees that you
 have at least one of the four server-side versions of the PHP5 interpreter
 installed. Removing this package won't remove PHP5 from your system, however
 it may remove other packages that depend on this one.
 .
 PHP5 is a widely-used general-purpose scripting language that is
 especially suited for Web development and can be embedded into HTML.
 The goal of the language is to allow web developers to write
 dynamically generated pages quickly.
Homepage: http://www.php.net/
Description-md5: b826cf1de0b3c891e4e48313d300ade9
Bugs: https://bugs.launchpad.net/ubuntu/+filebug
Origin: Ubuntu
Supported: 5y
Task: mythbuntu-frontend, mythbuntu-desktop, mythbuntu-backend-slave, mythbuntu-backend-master, mythbuntu-backend-master

Package: php5
Priority: optional
Section: php
Installed-Size: 21
Maintainer: Ubuntu Developers <ubuntu-devel-discuss@lists.ubuntu.com>
Original-Maintainer: Debian PHP Maintainers <pkg-php-maint@lists.alioth.debian.org>
Architecture: all
Version: 5.3.10-1ubuntu3
Depends: libapache2-mod-php5 (>= 5.3.10-1ubuntu3) | libapache2-mod-php5filter (>= 5.3.10-1ubuntu3) | php5-cgi (>= 5.3.10-1ubuntu3) | php5-fpm (>= 5.3.10-1ubuntu3), php5-common (>= 5.3.10-1ubuntu3)
Filename: pool/main/p/php5/php5_5.3.10-1ubuntu3_all.deb
Size: 1072
MD5sum: 143c4760cf3c27d54592f77a3f634d31
SHA1: f9c4faa06c9dd983652855db6f55426ffccd56fa
SHA256: e003bb86860d7b3f1be2f7aae1878e6382b293be5d5c8a7c1a1d40c6d386ed07
Description-en: server-side, HTML-embedded scripting language (metapackage)
 This package is a metapackage that, when installed, guarantees that you
 have at least one of the four server-side versions of the PHP5 interpreter
 installed. Removing this package won't remove PHP5 from your system, however
 it may remove other packages that depend on this one.
 .
 PHP5 is a widely-used general-purpose scripting language that is
 especially suited for Web development and can be embedded into HTML.
 The goal of the language is to allow web developers to write
 dynamically generated pages quickly.
Homepage: http://www.php.net/
Description-md5: b826cf1de0b3c891e4e48313d300ade9
Bugs: https://bugs.launchpad.net/ubuntu/+filebug
Origin: Ubuntu
Supported: 5y
Task: mythbuntu-frontend, mythbuntu-desktop, mythbuntu-backend-slave, mythbuntu-backend-master, mythbuntu-backend-master
~~~

いざ、インストールしてみる。

~~~ bash
$ sudo apt-get install php5
 :
 :
Setting up php5 (5.5.12+dfsg-2+deb.sury.org~precise+1) ...
Processing triggers for libc-bin ...
ldconfig deferred processing now taking place
~~~

これでバージョン確認してみると…。

~~~ bash
$ php -v
PHP 5.5.9 (cli) (built: Jun  9 2014 14:45:47)
Copyright (c) 1997-2014 The PHP Group
Zend Engine v2.5.0, Copyright (c) 1998-2014 Zend Technologies
~~~

あれー、5.5.12じゃないのかね。どうも、PATHが上手く更新されないようだ。
とりあえずは、/usr/local/bin/phpをリネームとかしてログアウトしてログインしたら行けた。

~~~ bash
$ sudo mv /usr/local/bin/php /usr/local/bin/_____php
vagrant@precise64:/etc/apt$ php -v
-bash: /usr/local/bin/php: No such file or directory
~~~

でログアウトしてログイン後

~~~ bash
$ php -v
PHP 5.5.12-2+deb.sury.org~precise+1 (cli) (built: May 12 2014 13:46:35)
Copyright (c) 1997-2014 The PHP Group
Zend Engine v2.5.0, Copyright (c) 1998-2014 Zend Technologies
    with Zend OPcache v7.0.4-dev, Copyright (c) 1999-2014, by Zend Technologies
~~~

まあ、Virtualbox+vagrantなのでゴミが気になるならもっかいやれば良い。


