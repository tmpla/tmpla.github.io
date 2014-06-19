---
layout: post
title: "Hadoop 0.20をインストールする"
modified: 2014-06-19 13:50:09 +0900
tags: [Hadoop,HBase,MapReduce,HDFS]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: true
published: true
---
先日苦労して最新版Hadoopを動かせるようにしたのだが、最終目的であるNutch 2.2.1がHBase 0.90系じゃないとダメだった。  
先日のインストールは[CDH5](http://www.cloudera.com/content/cloudera-content/cloudera-docs/CDH5/latest/CDH-Version-and-Packaging-Information/cdhvd_cdh_package_tarball.html)でHBaseもインストール出来たのだが、バージョンは0.96系。

よくよく調べてみると、[CDH3](http://www.cloudera.com/content/cloudera-content/cloudera-docs/CDH4/4.6.0/CDH-Version-and-Packaging-Information/cdhvd_topic_7.html?scroll=topic_6_2_unique_9)を使えばHBase0.90系が使えそうなので、CDH3のyumリポジトリからインストールすることにする。  
とりあえずは、HBaseは置いといて、Hadoopだけ使えるようにしてみる。  

ちなみに、javaは[前回]({{ site_url }}/installation-hadoop-pseudo-destributed-mode)でも使ったchefのjavaレシピで入れたJDK1.7である。

~~~ bash
$ java -version
java version "1.7.0_51"
Java(TM) SE Runtime Environment (build 1.7.0_51-b13)
Java HotSpot(TM) 64-Bit Server VM (build 24.51-b03, mixed mode)
~~~

## yumリポジトリの追加

リポジトリファイルをダウンロードする。

~~~ bash
$ cd /etc/yum.repos.d/
$ curl -O http://archive.cloudera.com/redhat/6/x86_64/cdh/cloudera-cdh3.repo
~~~

## 擬似分散モードの設定インストール
よく理解しているなら個別にパッケージを指定するのでいいと思うが、よく理解していないので、擬似分散モードの設定の依存パッケージに頼ることにする。

~~~ bash
$ sudo yum install -y hadoop-0.20-conf-pseudo
$ yum deplist hadoop-0.20-conf-pseudo
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: centos.mirror.secureax.com
 * extras: centos.mirror.secureax.com
 * jpackage-generic: ftp.heanet.ie
 * jpackage-generic-updates: ftp.heanet.ie
 * rpmforge: ftp.kddilabs.jp
 * updates: centos.mirror.secureax.com
Finding dependencies:
package: hadoop-0.20-conf-pseudo.noarch 0.20.2+923.479-1
  dependency: hadoop-0.20-jobtracker = 0.20.2+923.479-1
   provider: hadoop-0.20-jobtracker.noarch 0.20.2+923.479-1
  dependency: hadoop-0.20 = 0.20.2+923.479-1
   provider: hadoop-0.20.noarch 0.20.2+923.479-1
  dependency: hadoop-0.20-tasktracker = 0.20.2+923.479-1
   provider: hadoop-0.20-tasktracker.noarch 0.20.2+923.479-1
  dependency: hadoop-0.20-namenode = 0.20.2+923.479-1
   provider: hadoop-0.20-namenode.noarch 0.20.2+923.479-1
  dependency: hadoop-0.20-datanode = 0.20.2+923.479-1
   provider: hadoop-0.20-datanode.noarch 0.20.2+923.479-1
  dependency: hadoop-0.20-secondarynamenode = 0.20.2+923.479-1
   provider: hadoop-0.20-secondarynamenode.noarch 0.20.2+923.479-1
  dependency: /bin/sh
   provider: bash.x86_64 4.1.2-15.el6_4
~~~

見ると、jobtracker, tasktracker, namenode, datanode, secondarynamenodeと本体(hadoop-0.20)が使われていることがわかる。
jobtracker, tasktrackerはMapReduceの、namenode, datanode, secondarynamenodeはHDFSで使うんじゃないかと思う。

ところで、最新版では、JAVA_HOMEを特定するのにbigtopというユーティリティを使っていた。  
今回はそれは依存関係でインストールされていないので、どこでやってるのかを見てみたら、hadoop-config.shというファイルでやっているようだ。  
service hadoop-0.20-*を実行するとすると、呼び順は、

1. /etc/init.d/hadoop-0.20-*
2. /usr/lib/hadoop-0.20/hadoop-daemon.sh
3. /usr/lib/hadoop-0.20/bin/hadoop-config.sh

のようになって、3番目のとこでセットしている。ここに/usr/lib/jvm/jdk1.7*というのを入れた。  
しかしなぜ普通にJAVA_HOMEを認識してくれないのか・・・。

~~~ bash
67 # attempt to find java
68 if [ -z "$JAVA_HOME" ]; then
69   for candidate in \
70     /usr/lib/jvm/java-6-sun \
71     /usr/lib/jvm/java-1.6.0-sun-1.6.0.*/jre/ \
72     /usr/lib/jvm/java-1.6.0-sun-1.6.0.* \
73     /usr/lib/jvm/j2sdk1.6-oracle \
74     /usr/lib/jvm/j2sdk1.6-oracle/jre \
75     /usr/lib/j2sdk1.6-sun \
76     /usr/java/jdk1.6* \
77     /usr/java/jre1.6* \
78     /Library/Java/Home \
79     /usr/java/default \
80     /usr/lib/jvm/jdk1.7* \ # ここ
81     /usr/lib/jvm/default-java ; do
82     if [ -e $candidate/bin/java ]; then
83       export JAVA_HOME=$candidate
84       break
85     fi
86   done
~~~

設定ファイル等は、/etc/hadoop-0.20/confにあるのでそれを修正することになるが、今回修正はしなくても良かった。  
これで起動して、WebUIで確認できればOK。  
serviceスクリプトが多くて面倒くさい。  

~~~ bash
$ sudo service hadoop-0.20-namenode start
$ sudo service hadoop-0.20-datanode start
$ sudo service hadoop-0.20-secondarynamenode start
$ sudo service hadoop-0.20-jobtracker start
$ sudo service hadoop-0.20-tasktracker start
~~~

WebUIのアクセスポートはどこで定義すれば良いのかわからなかったので調べてみたが、
[r2.3.0のhdfs-default.xml](http://hadoop.apache.org/docs/r2.3.0/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml)と
[r2.3.0のmapred-default.xml](http://hadoop.apache.org/docs/r2.3.0/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml)
を見つけたのでそこから推測するに

datanode: 50075
namenode: 50070
jobtracker: 50030
tasktracker: 50060

という具合のようだ。ゲストのIPアドレス:上記のポートでアクセス出来るのを確認して完了。  
自分の環境では、namenodeのアクセスをすると/dfshealth.htmlにリダイレクトして404エラーになってしまうが、  
dfshealth.jspに修正すれば画面が出るようになった。
