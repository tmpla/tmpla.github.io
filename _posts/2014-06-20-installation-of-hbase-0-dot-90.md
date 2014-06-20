---
layout: post
title: "HBase 0.90のインストール"
modified: 2014-06-20 10:37:12 +0900
tags: [HBase,Hadoop,CDH3]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: true
---

[前回の記事]({{ site_url }}/installation-of-hadoop-0-dot-20)でCDH3を使ってHadoop 0.20をインストールしたが、
このCDH3に含まれるHBase 0.90をインストールしてみた。
Javaはインストール済みとして、/usr/lib/jvm/jdk1.7.0_51/がJAVA_HOMEの状態。

なぜHBaseの最新版ではなく0.90を使いたいかというと、[Apache Nutchのチュートリアル](http://wiki.apache.org/nutch/Nutch2Tutorial)に書いてあるし、
実際HBase 0.96で動かして見たらダメだったようなので。

CDH3が使えるようになっていれば、インストールは簡単である。

~~~ bash
$ cd /etc/yum.repos.d/
$ curl -O http://archive.cloudera.com/redhat/6/x86_64/cdh/cloudera-cdh3.repo
~~~

ところで、何をインストールしたらいいか迷うので、一度yum search hbaseしてみる。

~~~ bash
================================================= N/S Matched: hbase ==================================================
hadoop-hbase.noarch : HBase is the Hadoop database. Use it when you need random, realtime read/write access to your Big
                    : Data. This project's goal is the hosting of very large tables -- billions of rows X millions of
                    : columns -- atop clusters of commodity hardware.
hadoop-hbase-doc.noarch : Hbase Documentation
hadoop-hbase-master.noarch : The Hadoop HBase master Server.
hadoop-hbase-regionserver.noarch : The Hadoop HBase RegionServer server.
hadoop-hbase-rest.noarch : The Apache HBase REST gateway
hadoop-hbase-thrift.noarch : The Hadoop HBase Thrift Interface
hadoop-hive-hbase.noarch : Provides integration between Apache HBase and Apache Hive
~~~

色々調べて見ると、hbase-masterがまとめサーバーで、regionserverがデータを持つところのようである。
無印はHBaseのクライアント機能等なのだろう。
なので、hbase-masterとhbase-regionserverはインストールすることにする。（あとは依存関係解決におまかせ）

~~~ bash
$ sudo yum install -y hadoop-hbase-master hadoop-hbase-regionserver
~~~

## HDFSの設定
[前回の記事]({{ site_url }}/installation-of-hadoop-0-dot-20)で動かしたようにしておく。  
つまりhdfs://localhost:8020でアクセス出来るようにしておく。

そして、hbaseで使用するディレクトリを作成しておく。

~~~ bash
$ sudo -u hdfs hadoop fs -mkdir /hbase
$ sudo -u hdfs hadoop fs -chown hbase /hbase
~~~

## JAVA\_HOME解決

HBaseもなぜか自前でJAVA\_HOMEを解決しようとしているようだった。
なので、hbase-config.shというファイルのJAVA_HOMEを決めているところを変更した。

~~~ bash
$ vim /usr/lib/hbase/conf/hbase-config.sh
 86 if [ -z "$JAVA_HOME" ]; then
 87   for candidate in \
 88     /usr/lib/jvm/java-6-sun \
 89     /usr/lib/jvm/java-1.6.0-sun-1.6.0.*/jre \
 90     /usr/lib/jvm/java-1.6.0-sun-1.6.0.* \
 91     /usr/lib/jvm/j2sdk1.6-oracle \
 92     /usr/lib/jvm/j2sdk1.6-oracle/jre \
 93     /usr/lib/j2sdk1.6-sun \
 94     /usr/java/jdk1.6* \
 95     /usr/java/jre1.6* \
 96 +   /usr/lib/jvm/jdk1.7* \ 
 97     /Library/Java/Home ; do
 98     if [ -e $candidate/bin/java ]; then
 99       export JAVA_HOME=$candidate
100       break
101     fi
102   done
~~~

これで起動して、HBase shellでアクセスしてみる。

~~~ bash
$ sudo service hadoop-hbase-master start
$ sudo service hadoop-hbase-regionserver start
$ hbase shell
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 0.90.6-cdh3u6, r, Wed Mar 20 12:02:52 PDT 2013

hbase(main):001:0> status

ERROR: org.apache.hadoop.hbase.ZooKeeperConnectionException: HBase is able to connect to ZooKeeper but the connection closes immediately. This could be a sign that the server has too many connections (30 is the default). Consider inspecting your ZK server logs for that error and then make sure you are reusing HBaseConfiguration as often as you can. See HTable's javadoc for more information.
~~~

ZooKeeperConnectionExceptionというのが発生しているようだ。
薄々感づいてはいたが、ZooKeeperというのを起動させないとダメなようだ・・・。
CDHにこれも含まれているので、これもインストールしとく。

~~~ bash
$ yum install -y hadoop-zookeeper-server
~~~

また、/etc/zookeeper/の下に設定ファイルがあるはずなのになかったのでコピーしておく。
/etc/zookeeper.dist/にはなぜかあった。

~~~ bash
$ sudo cp /etc/zookeeper.dist/configuration.xsl /etc/zookeeper/
$ sudo cp /etc/zookeeper.dist/log4j.properties /etc/zookeeper/
$ sudo cp /etc/zookeeper.dist/zoo.cfg /etc/zookeeper/
~~~

中身はこのようなもの。

~~~ bash
$ vim /etc/zookeeper/zoo.cfg
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
dataDir=/var/zookeeper
# the port at which the clients will connect
clientPort=2181
server.0=localhost:2888:3888
~~~

これで起動してみる。

~~~ bash
$ sudo service hadoop-zookeeper-server start
$ sudo service hadoop-zookeeper-server status
hadoop-zookeeper-server is running
~~~

zookeeperのスクリプトは起動でコケてもSTDOUTに何も出さないのでステータスを確認している。
ところでzookeeperとは一体なんだろう。[ZooKeeper](http://oss.infoscience.co.jp/hadoop/zookeeper/docs/r3.3.1/zookeeperOver.html)を見ると　　

> 「分散アプリケーションのための分散コーディネーションサービス」

とある。プロセスを分割するためのそれぞれのプロセスの名前を解決するとかそんな感じなのだろうか。
ともあれ、zookeeperのクライアントがあるので見てみた。

~~~ bash
$ zookeeper-client
 (略)
[zk: localhost:2181(CONNECTED) 0]　help
ZooKeeper -server host:port cmd args
        connect host:port
        get path [watch]
        ls path [watch]
        set path data [version]
        delquota [-n|-b] path
        quit
        printwatches on|off
        create [-s] [-e] path data acl
        stat path [watch]
        close
        ls2 path [watch]
        history
        listquota path
        setAcl path acl
        getAcl path
        sync path
        redo cmdno
        addauth scheme auth
        delete path [version]
        setquota -n|-b val path

~~~

どうもファイルシステムっぽく定義にアクセス出来るらしい。
試しに、ls /としてみた

~~~ bash
[zk: localhost:2181(CONNECTED) 1] ls /
[zookeeper]
[zk: localhost:2181(CONNECTED) 2] 
~~~

タブ補完も出来るし、そのうちわかってくるだろう。

そしてとりあえずまたHBaseを起動してみる。

~~~ bash
$ sudo service hadoop-hbase-master start
$ sudo service hadoop-hbase-regionserver start
$ hbase shell
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 0.90.6-cdh3u6, r, Wed Mar 20 12:02:52 PDT 2013

hbase(main):001:0> status
14/06/20 06:17:54 FATAL zookeeper.ZKConfig: The server in zoo.cfg cannot be set to localhost in a fully-distributed setup because it won't be reachable. See "Getting Started" for more information.
1 servers, 0 dead, 7.0000 average load
~~~

エラーメッセージが出ているが、多分これはzoo.cfgにlocalhostと設定したらダメだよ、のようなエラーなんで大丈夫だろう。
このままhbase shellを終了して、zookeeper-clientを起動して見てみたかったのでやってみた。

~~~ bash
[zk: localhost:2181(CONNECTED) 0] ls /
[hbase, zookeeper]
~~~

おお、hbaseのエントリが追加されているっぽい。hbase自体にはあまり触れられていないが、zookeeperのことがちょっと分かった。
