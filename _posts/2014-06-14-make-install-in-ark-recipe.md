---
layout: post
title: "arkでmake installするレシピを楽にする方法"
modified: 2014-06-14 16:46:37 +0900
tags: [chef,recipe,ark]
image:
  feature:
  credit:
  creditlink:
comments:
share:
---
下記記事でarkと言うレシピがあることを知った。

[Chefでソースアーカイブ取得→展開→ビルド→インストールするならark cookbookを使わない手はない - Qiita](http://qiita.com/soundTricker/items/3ffabb746d38eb083c45)

Chefのレシピで、package(yum, apt-getとか)でインストール出来るのはそれで良いが、
ソースコードを持ってきてconfigure, make, make installを行う処理は自分で書いていた。
remote_fileで持ってきて、executeで展開、configure, makeするなど。
この辺のところを汎用的に使えるようにしたレシピのようだ。

[burtlo/ark](https://github.com/burtlo/ark)

arkと言うリソースでは下記のようなactionがある。

> Actions
>
:install: extracts the file and creates a 'friendly' symbolic link to the extracted directory path
>
:configure: configure ahead of the install action
>
:install\_with\_make: extracts the archive to a path, runs make, and make install. It does not run the configure step at this time
>
:dump: strips all directories from the archive and dumps the contained files into a specified path
>
:cherry\_pick: extract a specified file from an archive and places in specified path
>
:put: extract the archive to a specified path, does not create any symbolic links
>
:remove: removes the extracted directory and related symlink #TODO
>
:setup\_py\_build: runs the command "python setup.py build" in the extracted directory
>
:setup\_py\_install: runs the comand "python setup.py install" in the extracted directory


今回、defaultのactionは:installのようだが、これはアーカイブを取得して展開し、/usr/local/(リソース名)のところに置くまでしかしない。
make installまでやってくれるのは、:install\_with\_makeを指定する。

今回、Hadoopのビルドに必要だったので、[protobuf](https://code.google.com/p/protobuf/)と言うものをarkレシピでインストールしてみた。
ちなみにchef solo環境で、レシピ名は「protobuf」で、site-cookbooks/protobufに配置し、
Berkfileには

~~~ ruby
cookbook "protobuf", path: "site-cookbooks/protobuf"
~~~

とやっておく。
このレシピは[githubに上げた](https://github.com/kyokoshima/chef-protobuf)。

~~~ ruby
# metadata.rb
name             'protobuf'
maintainer       'YOUR_COMPANY_NAME'
maintainer_email 'YOUR_EMAIL'
license          'All rights reserved'
description      'Installs/Configures protobuf'
long_description IO.read(File.join(File.dirname(__FILE__), 'README.md'))
version          '0.1.0'

%w[ark].each do |dep|
  depends dep
end
~~~

~~~ ruby
# recipe/default.rb
%w[g++ make].each do |pkg|
  package pkg
end

ark "protobuf" do
  url "https://protobuf.googlecode.com/files/protobuf-2.5.0.tar.gz"
  action :install_with_make
end
~~~

g++とmakeはpackageで入れる。ゲストがprotobufだとしたら、knife solo cook protobufでレシピが実行される。
このprotobufは、「protoc」と言うコマンドを生成するので確認してみるとちゃんと入っている。

vagrant sshして

~~~ bash
vagrant@precise64:~$ which protoc
/usr/local/bin/protoc
~~~

また便利なレシピを知った。