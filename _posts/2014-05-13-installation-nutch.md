---
layout: post
title: "WebクローラNutchを動かしてみる"
modified: 2014-05-13 14:44:47 +0900
tags: [Solr,Nutch,Web Crawl]
image:
  feature:
  credit:
  creditlink:
comments:
share:
published: false
---
Apache Nutchと言うオープンソースのクローラがあるが、色々苦労して動かしたので書く。
元々は[Nutch2Tutorial - Nutch Wiki](http://wiki.apache.org/nutch/Nutch2Tutorial)に沿ってやろうとしたのだが、
HBaseとの連携で[このパッチ](https://issues.apache.org/jira/browse/NUTCH-1714)を適用しなければいけないようで、
ちょっと一筋縄ではいかないようなのでCassandoraを使っている。
とりあえずの環境ではあるが、最終的には一台でCassandra, Solr, Nutchを動かす環境となる。

[Nutch2Cassandra - Nutch Wiki](http://wiki.apache.org/nutch/Nutch2Cassandra)

をやることにした。

[Welcome to Apache Nutch™](https://nutch.apache.org/)

環境はMac Marvericks上のCentOS6.5(x86_64)をVirtualBoxで動かしている。


## Cassandraのインストール
現時点では、Cassandraの最新版は2.0.7なのでそれをインストールする。
まずはここからダウンロードする。

[Apache Download Mirrors](http://www.apache.org/dyn/closer.cgi?path=/cassandra/2.0.7/apache-cassandra-2.0.7-bin.tar.gz)

これをダウンロードし、展開して起動しておく

{% highlight bash %}
$ cd /tmp
$ wget http://ftp.riken.jp/net/apache/cassandra/2.0.7/apache-cassandra-2.0.7-bin.tar.gz
$ tar zxvf apache-cassandra-2.0.7-bin.tar.gz
$ cd apache-cassandra-2.0.7
$ sudo bin/cassandra
{% endhighlight %}

rootで何かやる必要があるようなので、sudoで起動します。これで起動しているはず。

## Nutchインストール
現時点では2.2.1が最新のようなので、それを使う。
好きなところからダウンロードしよう！

[Apache Download Mirrors](http://www.apache.org/dyn/closer.cgi/nutch/2.2.1/apache-nutch-2.2.1-src.tar.gz)

また、ビルドにはJavaとantが必要なので、それもインストールするが割愛する。バージョンだけ。
{% highlight bash %}
[vagrant@localhost example]$ java -version
java version "1.7.0_51"
Java(TM) SE Runtime Environment (build 1.7.0_51-b13)
Java HotSpot(TM) 64-Bit Server VM (build 24.51-b03, mixed mode)
[vagrant@localhost example]$ ant -version
Apache Ant(TM) version 1.8.2 compiled on December 20 2010
[vagrant@localhost example]$
{% endhighlight %}

ダウンロードしたらとりあえず展開して設定ファイルを編集する。
まずは、ビルドの設定を変える。
また、「http.agent.name」の値が必要なようなのでそれも一緒に追加しておく。

* /tmp/apache-nutch-2.2.1/conf/nutch-site.xml

{% highlight xml %}
<configuration>
+ <property>
+  <name>http.agent.name</name>
+  <value>My Nutch Spider</value>
+ </property>
+ <property>
+ 	<name>storage.data.store.class</name>
+ 	<value>org.apache.gora.cassandra.store.CassandraStore</value>
+ 	<description>Default class for storing data</description>
+ </property>
</configuration>
{% endhighlight %}

* gora.propertiesの編集

{% hightlight %}
$ vim conf/gora.properties
gora.datastore.default=org.apache.gora.cassandra.store.CassandraStore
gora.cassandrastore.servers=localhost:9160
{% endhighlight %}

* ivy/ivy.xmlの編集
ivyは多分、jar同士の依存関係をうまく解決してくれるツールのようだ。
xmlファイルに書いておけば、依存するjarファイルをしてそれをダウンロードしてくれるような想像。
これもcassandra用に編集する。

ivy/ivy.xmlにコメントとしてcassandraの設定があると思うので、それをコメントアウトする。

{% hightligh xml %}
<dependency org="org.apache.gora" name="gora-cassandra" rev="0.3" conf="*->default" />
{% enfhightlight %}

## Nutchの実行
あるディレクトリ以下のテキストファイルを読み込んでクロールするような仕組みなようで、
下記のようなファイルを作った

{% highlight %}
$ mkdir urls
$ vim urls/seed.txt
http://nutch.apacht.org/
{% endhighlight %}:wq

