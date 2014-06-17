---
layout: post
title: "Hadoop2.3を擬似分散モードで動かしてみる"
modified: 2014-06-16 18:52:42 +0900
tags: [hadoop,hdfs,yarn,mapreduce,hbase]
image:
  feature:
  credit:
  creditlink:
comments:
share: true
published: true
---

Hadoopを使いたいが、単語だけ知ってるレベルの状態からインストールするまでに苦労したので記録する。

[Hadoop](http://hadoop.apache.org/)は分散処理を行うためのライブラリである。
Googleが発表した[MapReduceの論文](http://research.google.com/archive/mapreduce.html)と[BigTableの論文](http://research.google.com/archive/bigtable.html)に触発された実装だそうである。

BigTableというのはファイルシステムを分散管理するもの、MapReduceは処理の分散化をするもの、だと思う。
BigTableはHDFS, MapReduceはそのままの名前だが、2.3.0ではYARN（やーーん？）というフレームワーク？の上に乗せて作成されているようだ。
HDFSは、ファイルがおいてある先が勝手に分散されてるということで、使用する側は意識しなくてい良い、くらいの認識で良いんじゃないかと。
MapReduceについてはあまり理解出来ているわけではないのだが、MapReduceの例では転置インデックスの話がある。
これはMap処理でカウント、Reduceでその集計という感じだが、なぜMapとReduceで処理を分けるのか？
これは、大規模なデータを扱うための知恵かと。
Mapにも巨大ファイルを分割して、かつ複数のプロセスで集計するが、その巨大な結果をReducerに渡し、
Reducerでもの複数かつ並行プロセスで処理して合理的に処理しよう、ということなのだろうと思う。
だからメモリに収まる程度のデータだともしかしたら全然意味はないのかも（並行処理は出来るとは思うが）

今回は、HDFS上にHBaseのストレージを構築するところまでやってみる。

## 環境
Mac上にVirtualBoxでCentOS 6.5(64bit)を作ってvagrantで構築した。WebUIの確認をしたいため、ネットワークはpublic\_network設定にしてある。
JavaはOracleの下記

~~~ bash
$ java -version
java version "1.7.0_51"
Java(TM) SE Runtime Environment (build 1.7.0_51-b13)
Java HotSpot(TM) 64-Bit Server VM (build 24.51-b03, mixed mode)
~~~

JAVA_HOMEはセットしておく。また、iptablesは切っておく。

## インストール

[Hadoopのページ](http://hadoop.apache.org/releases.html#Download)からダウンロードして展開すればそれで終わり、と思っていたのだが、どんなサービスがどう動くかを理解していないと意味がわからない。
最初は、バイナリを展開してみたのだが、設定が必要なのがどことどこなのかがわからない。
そこで[Apache Hadoop 2.3.0 - Hadoop MapReduce Next Generation 2.3.0 - Setting up a Single Node Cluster.](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/SingleCluster.html)というページを見てみたが、これはソースからビルドして、さらにHDFSの話は特に書いていなさそうだったがやってみた。

まずmavenが必要なのと、protobufが必要。
さらに、下記ページに書かれているような調整をしないとダメで、ビルドは出来たがやはりどのサービスが必要なのかがよくわからなかった。

[Geting Started With Hadoop 2.2.0 -- Building - recluze](http://csrdu.org/nauman/2014/01/23/geting-started-with-hadoop-2-2-0-building/)

ビルド後はこんな感じで、今のところ何が何やら、というとこでここから動かすのは断念した。

~~~ bash
$ ls -la
drwxr-xr-x 15 vagrant vagrant  4096 Jun 12 08:40 .
drwx------ 12 vagrant vagrant  4096 Jun 17 03:07 ..
-rw-r--r--  1 vagrant vagrant  9968 Oct  7  2013 BUILDING.txt
drwxr-xr-x  2 vagrant vagrant  4096 Oct  7  2013 dev-support
-rw-r--r--  1 vagrant vagrant   403 Oct  7  2013 .gitattributes
drwxr-xr-x  4 vagrant vagrant  4096 Jun 13 02:16 hadoop-assemblies
drwxr-xr-x  3 vagrant vagrant  4096 Jun 13 02:18 hadoop-client
drwxr-xr-x  9 vagrant vagrant  4096 Jun 13 02:17 hadoop-common-project
drwxr-xr-x  3 vagrant vagrant  4096 Jun 13 02:18 hadoop-dist
drwxr-xr-x  7 vagrant vagrant  4096 Jun 13 02:17 hadoop-hdfs-project
drwxr-xr-x 11 vagrant vagrant  4096 Jun 13 02:18 hadoop-mapreduce-project
drwxr-xr-x  4 vagrant vagrant  4096 Jun 13 02:16 hadoop-maven-plugins
drwxr-xr-x  3 vagrant vagrant  4096 Jun 13 02:18 hadoop-minicluster
drwxr-xr-x  4 vagrant vagrant  4096 Jun 13 02:16 hadoop-project
drwxr-xr-x  3 vagrant vagrant  4096 Jun 13 02:16 hadoop-project-dist
drwxr-xr-x 12 vagrant vagrant  4096 Jun 13 02:18 hadoop-tools
drwxr-xr-x  4 vagrant vagrant  4096 Jun 13 02:18 hadoop-yarn-project
-rw-r--r--  1 vagrant vagrant 15164 Oct  7  2013 LICENSE.txt
-rw-r--r--  1 vagrant vagrant   101 Oct  7  2013 NOTICE.txt
-rw-r--r--  1 vagrant vagrant 16569 Oct  7  2013 pom.xml
-rw-r--r--  1 vagrant vagrant  1366 Oct  7  2013 README.txt
~~~

ところで、clouderaというところが、Hadoopのyumリポジトリ（他のOSのも）を用意しているというのを見つけた。
しかも、ここにQuick Start的なテキストもあるようなので、これを使ってインストールした。

[CDH 5 Quick Start Guide ](http://www.cloudera.com/content/cloudera-content/cloudera-docs/CDH5/latest/CDH5-Quick-Start/CDH5-Quick-Start.html)

ここで、最新版ではMapReduceの部分がYARNというフレームワーク？を使って構築されているようなことがなんとなくわかる。
さらに見ると、pseudo-distributed mode(擬似分散モード）というのが出来るそうなのでそれを目指す。

yumリポジトリは[](http://www.cloudera.com/content/cloudera-content/cloudera-docs/CDH5/latest/CDH5-Installation-Guide/cdh5ig_cdh5_install.html)の中盤に書いてある、(http://archive.cloudera.com/cdh5/redhat/6/x86_64/cdh/cloudera-cdh5.repo)を使った。

~~~ bash
$ cd /etc/yum/repos.d/
$ sudo curl -O http://archive.cloudera.com/cdh5/redhat/6/x86_64/cdh/cloudera-cdh5.repo
~~~

これでhadoopを探してみると色んなのがあることがわかる。

~~~ bash
$ yum search hadoop
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: centos.mirror.secureax.com
 * extras: centos.mirror.secureax.com
 * updates: centos.mirror.secureax.com
======================================================== N/S Matched: hadoop ========================================================
hadoop.x86_64 : Hadoop is a software platform for processing vast amounts of data
hadoop-0.20-conf-pseudo.x86_64 : Hadoop installation in pseudo-distributed mode with MRv1
hadoop-0.20-mapreduce.x86_64 : Hadoop is a software platform for processing vast amounts of data
hadoop-0.20-mapreduce-jobtracker.x86_64 : Hadoop JobTracker
hadoop-0.20-mapreduce-jobtrackerha.x86_64 : Hadoop JobTracker High Availability
hadoop-0.20-mapreduce-tasktracker.x86_64 : Hadoop Task Tracker
hadoop-0.20-mapreduce-zkfc.x86_64 : Hadoop MapReduce failover controller
hadoop-client.x86_64 : Hadoop client side dependencies
hadoop-conf-pseudo.x86_64 : Hadoop installation in pseudo-distributed mode
hadoop-debuginfo.x86_64 : Debug information for package hadoop
hadoop-doc.x86_64 : Hadoop Documentation
hadoop-hdfs.x86_64 : The Hadoop Distributed File System
hadoop-hdfs-datanode.x86_64 : Hadoop Data Node
hadoop-hdfs-journalnode.x86_64 : Hadoop HDFS JournalNode
hadoop-hdfs-namenode.x86_64 : The Hadoop namenode manages the block locations of HDFS files
hadoop-hdfs-nfs3.x86_64 : Hadoop HDFS NFS v3 gateway service
hadoop-hdfs-secondarynamenode.x86_64 : Hadoop Secondary namenode
hadoop-hdfs-zkfc.x86_64 : Hadoop HDFS failover controller
hadoop-httpfs.x86_64 : HTTPFS for Hadoop
hadoop-libhdfs.x86_64 : Hadoop Filesystem Library
hadoop-mapreduce.x86_64 : The Hadoop MapReduce (MRv2)
	:
	:
~~~

擬似分散モードは、hadoop-conf-pseudo（hadoop-0.20-conf-pseudoもあるが、これは旧版のようだ）をインストールすると依存関係で色々インストールしてくれる。

~~~ bash
]$ sudo yum install -y hadoop-conf-pseudo
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: www.ftp.ne.jp
 * extras: www.ftp.ne.jp
 * updates: www.ftp.ne.jp
base                                                                                                          | 3.7 kB     00:00
cloudera-cdh5                                                                                                 |  951 B     00:00
cloudera-cdh5/primary                                                                                         |  40 kB     00:00
cloudera-cdh5                                                                                                                140/140
extras                                                                                                        | 3.4 kB     00:00
updates                                                                                                       | 3.4 kB     00:00
Setting up Install Process
Resolving Dependencies

	（略）

Installed:
  hadoop-conf-pseudo.x86_64 0:2.3.0+cdh5.0.2+579-1.cdh5.0.2.p0.25.el6

Dependency Installed:
  alsa-lib.x86_64 0:1.0.22-3.el6
  at.x86_64 0:3.1.10-43.el6_2.1
  atk.x86_64 0:1.30.0-1.el6
  avahi-libs.x86_64 0:0.6.25-12.el6
  avro-libs.noarch 0:1.7.5+cdh5.0.2+20-1.cdh5.0.2.p0.23.el6
  bc.x86_64 0:1.06.95-1.el6
  bigtop-jsvc.x86_64 0:0.6.0+cdh5.0.2+432-1.cdh5.0.2.p0.18.el6
  bigtop-utils.noarch 0:0.7.0+cdh5.0.2+0-1.cdh5.0.2.p0.23.el6
  cairo.x86_64 0:1.8.8-3.1.el6
  cdparanoia-libs.x86_64 0:10.2-5.1.el6
  cups.x86_64 1:1.4.2-50.el6_4.5
  cups-libs.x86_64 1:1.4.2-50.el6_4.5
  cvs.x86_64 0:1.11.23-16.el6
  dbus.x86_64 1:1.2.24-7.el6_3
  ed.x86_64 0:1.1-3.3.el6
  fontconfig.x86_64 0:2.8.0-3.el6
  foomatic.x86_64 0:4.0.4-3.el6
  foomatic-db.noarch 0:4.0-7.20091126.el6
  foomatic-db-filesystem.noarch 0:4.0-7.20091126.el6
  foomatic-db-ppds.noarch 0:4.0-7.20091126.el6
  freetype.x86_64 0:2.3.11-14.el6_3.1
  gettext.x86_64 0:0.17-16.el6
  ghostscript.x86_64 0:8.70-19.el6
  ghostscript-fonts.noarch 0:5.50-23.1.el6
  gnutls.x86_64 0:2.8.5-14.el6_5
  gstreamer.x86_64 0:0.10.29-1.el6
  gstreamer-plugins-base.x86_64 0:0.10.29-2.el6
  gstreamer-tools.x86_64 0:0.10.29-1.el6
  gtk2.x86_64 0:2.20.1-4.el6
  hadoop.x86_64 0:2.3.0+cdh5.0.2+579-1.cdh5.0.2.p0.25.el6
  hadoop-hdfs.x86_64 0:2.3.0+cdh5.0.2+579-1.cdh5.0.2.p0.25.el6
  hadoop-hdfs-datanode.x86_64 0:2.3.0+cdh5.0.2+579-1.cdh5.0.2.p0.25.el6
  hadoop-hdfs-namenode.x86_64 0:2.3.0+cdh5.0.2+579-1.cdh5.0.2.p0.25.el6
  hadoop-hdfs-secondarynamenode.x86_64 0:2.3.0+cdh5.0.2+579-1.cdh5.0.2.p0.25.el6
  hadoop-mapreduce.x86_64 0:2.3.0+cdh5.0.2+579-1.cdh5.0.2.p0.25.el6
  hadoop-mapreduce-historyserver.x86_64 0:2.3.0+cdh5.0.2+579-1.cdh5.0.2.p0.25.el6
  hadoop-yarn.x86_64 0:2.3.0+cdh5.0.2+579-1.cdh5.0.2.p0.25.el6
  hadoop-yarn-nodemanager.x86_64 0:2.3.0+cdh5.0.2+579-1.cdh5.0.2.p0.25.el6
  hadoop-yarn-resourcemanager.x86_64 0:2.3.0+cdh5.0.2+579-1.cdh5.0.2.p0.25.el6
  hicolor-icon-theme.noarch 0:0.11-1.1.el6
  iso-codes.noarch 0:3.16-2.el6
  jasper-libs.x86_64 0:1.900.1-15.el6_1.1
  lcms-libs.x86_64 0:1.19-1.el6
  libICE.x86_64 0:1.0.6-1.el6
  libSM.x86_64 0:1.2.1-2.el6
  libX11.x86_64 0:1.5.0-4.el6
  libX11-common.noarch 0:1.5.0-4.el6
  libXau.x86_64 0:1.0.6-4.el6
  libXcomposite.x86_64 0:0.4.3-4.el6
  libXcursor.x86_64 0:1.1.13-6.20130524git8f677eaea.el6
  libXdamage.x86_64 0:1.1.3-4.el6
  libXext.x86_64 0:1.3.1-2.el6
  libXfixes.x86_64 0:5.0-3.el6
  libXfont.x86_64 0:1.4.5-3.el6_5
  libXft.x86_64 0:2.3.1-2.el6
  libXi.x86_64 0:1.6.1-3.el6
  libXinerama.x86_64 0:1.1.2-2.el6
  libXrandr.x86_64 0:1.4.0-1.el6
  libXrender.x86_64 0:0.9.7-2.el6
  libXt.x86_64 0:1.1.3-1.el6
  libXtst.x86_64 0:1.2.1-2.el6
  libXv.x86_64 0:1.0.7-2.el6
  libXxf86vm.x86_64 0:1.1.2-2.el6
  libfontenc.x86_64 0:1.0.5-2.el6
  libgudev1.x86_64 0:147-2.51.el6
  libjpeg-turbo.x86_64 0:1.2.1-3.el6_5
  libmng.x86_64 0:1.0.10-4.1.el6
  libogg.x86_64 2:1.1.4-2.1.el6
  liboil.x86_64 0:0.3.16-4.1.el6
  libpng.x86_64 2:1.2.49-1.el6_2
  libthai.x86_64 0:0.1.12-3.el6
  libtheora.x86_64 1:1.1.0-2.el6
  libtiff.x86_64 0:3.9.4-10.el6_5
  libvisual.x86_64 0:0.4.0-9.1.el6
  libvorbis.x86_64 1:1.2.3-4.el6_2.1
  libxcb.x86_64 0:1.8.1-1.el6
  mailx.x86_64 0:12.4-7.el6
  man.x86_64 0:1.6f-32.el6
  mesa-dri-drivers.x86_64 0:9.2-0.5.el6_5.2
  mesa-dri-filesystem.x86_64 0:9.2-0.5.el6_5.2
  mesa-dri1-drivers.x86_64 0:7.11-8.el6
  mesa-libGL.x86_64 0:9.2-0.5.el6_5.2
  mesa-libGLU.x86_64 0:9.2-0.5.el6_5.2
  mesa-private-llvm.x86_64 0:3.3-0.3.rc3.el6
  nc.x86_64 0:1.84-22.el6
  openjpeg-libs.x86_64 0:1.3-10.el6_5
  pango.x86_64 0:1.28.1-7.el6_3
  parquet.noarch 0:1.2.5+cdh5.0.2+95-1.cdh5.0.2.p0.19.el6
  parquet-format.noarch 0:1.0.0+cdh5.0.2+7-1.cdh5.0.2.p0.30.el6
  patch.x86_64 0:2.6-6.el6
  pax.x86_64 0:3.4-10.1.el6
  perl-CGI.x86_64 0:3.51-136.el6
  perl-Test-Simple.x86_64 0:0.92-136.el6
  phonon-backend-gstreamer.x86_64 1:4.6.2-28.el6_5
  pixman.x86_64 0:0.26.2-5.1.el6_5
  poppler.x86_64 0:0.12.4-3.el6_0.1
  poppler-data.noarch 0:0.4.0-1.el6
  poppler-utils.x86_64 0:0.12.4-3.el6_0.1
  portreserve.x86_64 0:0.0.4-9.el6
  qt.x86_64 1:4.6.2-28.el6_5
  qt-sqlite.x86_64 1:4.6.2-28.el6_5
  qt-x11.x86_64 1:4.6.2-28.el6_5
  qt3.x86_64 0:3.3.8b-30.el6
  redhat-lsb.x86_64 0:4.0-7.el6.centos
  redhat-lsb-compat.x86_64 0:4.0-7.el6.centos
  redhat-lsb-core.x86_64 0:4.0-7.el6.centos
  redhat-lsb-graphics.x86_64 0:4.0-7.el6.centos
  redhat-lsb-printing.x86_64 0:4.0-7.el6.centos
  time.x86_64 0:1.7-37.1.el6
  tmpwatch.x86_64 0:2.9.16-4.el6
  urw-fonts.noarch 0:2.4-10.el6
  xml-common.noarch 0:0.6.3-32.el6
  xorg-x11-font-utils.x86_64 1:7.2-11.el6
  xz.x86_64 0:4.999.9-0.3.beta.20091007git.el6
  xz-lzma-compat.x86_64 0:4.999.9-0.3.beta.20091007git.el6
  zookeeper.x86_64 0:3.4.5+cdh5.0.2+33-1.cdh5.0.2.p0.31.el6

Complete!

~~~

こんなに依存関係あるのかよ！と驚く。

一応インストール確認する。

~~~ bash
$ rpm -ql hadoop-conf-pseudo
/etc/hadoop/conf.pseudo
/etc/hadoop/conf.pseudo/README
/etc/hadoop/conf.pseudo/core-site.xml
/etc/hadoop/conf.pseudo/hadoop-env.sh
/etc/hadoop/conf.pseudo/hadoop-metrics.properties
/etc/hadoop/conf.pseudo/hdfs-site.xml
/etc/hadoop/conf.pseudo/log4j.properties
/etc/hadoop/conf.pseudo/mapred-site.xml
/etc/hadoop/conf.pseudo/yarn-site.xml
~~~

## 設定

### JAVA\_HOMEの引き継ぎ

Javaはchefのjavaレシピで入れたOracleのもの。
インストールパスは「/usr/lib/jvm/jdk1.7.0\_51/」になっている。
sudoでJAVA\_HOMEが引き継がれないようなので下記の行を追加しておく。
HDFSのコマンド(hadoop-hdfs-*じゃなくてhdfsクライアント）の方は、こちらの設定を見ているようなのでこのように。

~~~ bash
$ sudo visudo
Defaults env_keep += "JAVA_HOME"
~~~

また、hadoop-hdfs-*のサービスはbigtop-utils/bigtop-detect-javahomeというのを使ってJAVA\_HOMEを設定しているようだ。
ここには環境変数にいくら設定してもJAVA\_HOMEが設定できないようなので、「/usr/lib/jvm/jdk1.7.0\_51/」を追加した。

~~~ diff
JAVA7_HOME_CANDIDATES='\
    /usr/java/jdk1.7* \
    /usr/java/jre1.7* \
    /usr/lib/jvm/j2sdk1.7-oracle \
    /usr/lib/jvm/j2sdk1.7-oracle/jre \
-   /usr/lib/jvm/java-7-oracle*' \
+   /usr/lib/jvm/java-7-oracle* \
+   /usr/lib/jvm/jdk1.7*'
~~~

[bigtop](http://bigtop.apache.org/)なんて全然知らなかった・・・。

### namenodeの初期化

今回は、最新版でYARNを使った方にしたいので、[Installing CDH 5 with YARN on a Single Linux Node in Pseudo-distributed mode ](http://www.cloudera.com/content/cloudera-content/cloudera-docs/CDH5/latest/CDH5-Quick-Start/cdh5qs_yarn_pseudo.html)
の通りにやってみてる。

まずnamenodeというのをフォーマットする必要があるらしい。namenodというのは一体・・・。
が、やってみる。

~~~ bash
$ sudo -u hdfs hdfs namenode -format
~~~

すると何やらドカッとログが出て処理完了したようだ。

### HDFSサービスの起動

HDFSを操作するためのサービスを起動する。/etc/init.dを見ると、hadoop-*というのが幾つかあるのがわかる。

~~~ bash
]$ cd /etc/init.d/
[vagrant@crawler2 init.d]$ ls -la
total 312
drwxr-xr-x.  2 root root  4096 Jun 17 03:33 .
drwxr-xr-x. 10 root root  4096 Mar 11 11:33 ..
-rwxr-xr-x   1 root root  2062 Jan 30  2012 atd
-rwxr-xr-x.  1 root root  3378 Jun 22  2012 auditd
-r-xr-xr-x.  1 root root  1340 Nov 23  2013 blk-availability
-rwxr-xr-x.  1 root root  2826 Nov 23  2013 crond
-rwxr-xr-x   1 root root  3034 Aug 17  2013 cups
-rwxr-xr-x   1 root root  1131 Nov  9  2013 dkms_autoinstaller
-rw-r--r--.  1 root root 18586 Oct 10  2013 functions
-rwxr-xr-x   1 root root  4350 Jun  9 16:42 hadoop-hdfs-datanode
-rwxr-xr-x   1 root root  4484 Jun  9 16:42 hadoop-hdfs-namenode
-rwxr-xr-x   1 root root  4217 Jun  9 16:42 hadoop-hdfs-secondarynamenode
-rwxr-xr-x   1 root root  4238 Jun  9 16:42 hadoop-mapreduce-historyserver
-rwxr-xr-x   1 root root  4236 Jun  9 16:42 hadoop-yarn-nodemanager
-rwxr-xr-x   1 root root  4196 Jun  9 16:42 hadoop-yarn-resourcemanager
-rwxr-xr-x.  1 root root  5866 Oct 10  2013 halt
-rwxr-xr-x.  1 root root 10804 Nov 23  2013 ip6tables
-rwxr-xr-x.  1 root root 10688 Nov 23  2013 iptables
-rwxr-xr-x.  1 root root  4535 Oct  8  2013 iscsi
-rwxr-xr-x.  1 root root  3990 Oct  8  2013 iscsid
-rwxr-xr-x.  1 root root   652 Oct 10  2013 killall
-r-xr-xr-x.  1 root root  2134 Nov 23  2013 lvm2-lvmetad
-r-xr-xr-x.  1 root root  2665 Nov 23  2013 lvm2-monitor
-rwxr-xr-x.  1 root root  2571 Oct 11  2013 mdmonitor
-rwxr-xr-x   1 root root  2200 Sep 13  2012 messagebus
-rwxr-xr-x.  1 root root  2523 Nov 22  2013 multipathd
-rwxr-xr-x.  1 root root  2989 Oct 10  2013 netconsole
-rwxr-xr-x.  1 root root  5428 Oct 10  2013 netfs
-rwxr-xr-x.  1 root root  6334 Oct 10  2013 network
-rwxr-xr-x   1 root root  6364 Nov 22  2013 nfs
-rwxr-xr-x   1 root root  3526 Nov 22  2013 nfslock
-rwxr-xr-x   1 root root  2023 Apr  3  2012 portreserve
-rwxr-xr-x.  1 root root  3852 Dec  3  2011 postfix
-rwxr-xr-x   1 root root  2649 Feb 18 23:23 puppet
-rwxr-xr-x.  1 root root  1513 Sep 17  2013 rdisc
-rwxr-xr-x.  1 root root  1822 Nov 22  2013 restorecond
-rwxr-xr-x   1 root root  2073 Feb 22  2013 rpcbind
-rwxr-xr-x   1 root root  2518 Nov 22  2013 rpcgssd
-rwxr-xr-x   1 root root  2305 Nov 22  2013 rpcidmapd
-rwxr-xr-x   1 root root  2464 Nov 22  2013 rpcsvcgssd
-rwxr-xr-x.  1 root root  2011 Aug 15  2013 rsyslog
-rwxr-xr-x.  1 root root  1698 Nov 22  2013 sandbox
-rwxr-xr-x.  1 root root  2056 Nov 20  2012 saslauthd
-rwxr-xr-x.  1 root root   647 Oct 10  2013 single
-rwxr-xr-x.  1 root root  4534 Nov 22  2013 sshd
-rwxr-xr-x.  1 root root  2294 Nov 22  2013 udev-post
-rwxr-xr-x   1 root root 15634 Mar 11 11:43 vboxadd
-rwxr-xr-x   1 root root  5378 Mar 11 11:44 vboxadd-service
-rwxr-xr-x   1 root root 20887 Mar 11 17:44 vboxadd-x11

~~~

これらのうち、hadoop-hdfsと付いているのを起動する。

~~~ bash
$ for x in `cd /etc/init.d ; ls hadoop-hdfs-*` ; do sudo service $x start ; done
Starting Hadoop datanode:                                  [  OK  ]
starting datanode, logging to /var/log/hadoop-hdfs/hadoop-hdfs-datanode-xxxx.out
Starting Hadoop namenode:                                  [  OK  ]
starting namenode, logging to /var/log/hadoop-hdfs/hadoop-hdfs-namenode-xxxx.out
Starting Hadoop secondarynamenode:                         [  OK  ]
starting secondarynamenode, logging to /var/log/hadoop-hdfs/hadoop-hdfs-secondarynamenode-xxxx.out
~~~

HDFS関係のサービスは、datanode, namenode, secondarynamenodeという3つが関わっているのがわかる。
ここで、このHDFSサービスのステータスをWebで確認できる。
ホストOSから

	http://ゲストOSのIPアドレス:50070

にアクセスすると下記の画面が見れるだろう。

![DFSHealth]({{ site.url }}/images/06/17/hdfs-webui.png)

### MapReduceの設定

MapReduceはYARNというフレームワーク？（なんなの？）上で動いているらしい。これも動かしておこう（よくわからないけど・・・）

まず、先ほど起動したHDFSにディレクトリ・ファイルを作成する。

~~~ bash
$ sudo -u hdfs hadoop fs -mkdir -p /tmp/hadoop-yarn/staging/history/done_intermediate
$ sudo -u hdfs hadoop fs -chown -R mapred:mapred /tmp/hadoop-yarn/staging
$ sudo -u hdfs hadoop fs -chmod -R 1777 /tmp
$ sudo -u hdfs hadoop fs -mkdir -p /var/log/hadoop-yarn
$ sudo -u hdfs hadoop fs -chown yarn:mapred /var/log/hadoop-yarn
$ sudo -u hdfs hadoop fs -ls -R /
drwxrwxrwt   - hdfs supergroup          0 2014-06-17 07:12 /tmp
drwxrwxrwt   - hdfs supergroup          0 2014-06-17 07:12 /tmp/hadoop-yarn
drwxrwxrwt   - mapred mapred              0 2014-06-17 07:12 /tmp/hadoop-yarn/staging
drwxrwxrwt   - mapred mapred              0 2014-06-17 07:12 /tmp/hadoop-yarn/staging/history
drwxrwxrwt   - mapred mapred              0 2014-06-17 07:12 /tmp/hadoop-yarn/staging/history/done_intermediate
drwxr-xr-x   - hdfs   supergroup          0 2014-06-17 07:12 /var
drwxr-xr-x   - hdfs   supergroup          0 2014-06-17 07:12 /var/log
drwxr-xr-x   - yarn   mapred              0 2014-06-17 07:12 /var/log/hadoop-yarn
~~~

コマンドは、sudoでユーザーを指定するのと、hadoop fsというプレフィックスを付けるのが面倒臭いが、unixのファイルを扱うような感じのコマンドになっている。

このファイル群を作った状態で、YARN(MapReduce)のサービスを起動する。

~~~ bash
$ sudo service hadoop-yarn-resourcemanager start
Starting Hadoop resourcemanager:                           [  OK  ]
starting resourcemanager, logging to /var/log/hadoop-yarn/yarn-yarn-resourcemanager-xxxx.out
 sudo service hadoop-yarn-nodemanager start
Starting Hadoop nodemanager:                               [  OK  ]
starting nodemanager, logging to /var/log/hadoop-yarn/yarn-yarn-nodemanager-xxxx.out
$ sudo service hadoop-mapreduce-historyserver start
Starting Hadoop historyserver:                             [  OK  ]
starting historyserver, logging to /var/log/hadoop-mapreduce/mapred-mapred-historyserver-xxxx.out
~~~

MapReduceの処理では、resourcemanager, nodemanager, historyserverの3つがあるようだ。

## hbaseのインストール

ここまでで十分良くやった気がするが、ここにhbaseを乗せるまで。
hbaseは前述で追加したclouderaのリポジトリにあるので、それを使う。

ここから先は[HBase Installation ](http://www.cloudera.com/content/cloudera-content/cloudera-docs/CDH5/latest/CDH5-Installation-Guide/cdh5ig_hbase_installation.html)の内容をやっただけ。

こちらも、[擬似分散モードの項](http://www.cloudera.com/content/cloudera-content/cloudera-docs/CDH5/latest/CDH5-Installation-Guide/cdh5ig_hbase_pseudo_configure.html)があったのでそれを元にしている。

まずはhbase-masterだけインストールしてみる。

~~~ bash
$ sudo yum install -y hbase-master
 (略)
Installed:
  hbase-master.x86_64 0:0.96.1.1+cdh5.0.2+74-1.cdh5.0.2.p0.21.el6

Dependency Installed:
  hbase.x86_64 0:0.96.1.1+cdh5.0.2+74-1.cdh5.0.2.p0.21.el6

Complete!
~~~

これで、/etc/hadoop/conf/hbase-site.xmlを修正する。

~~~ bash
$ sudo vim /etc/hbase/conf/hbase-site.xml
<configuration>
	<property>
    	<name>hbase.cluster.distributed</name>
        <value>true</value>
    </property>
    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://localhost:8020/hbase</value>
    </property>
</configuration>
~~~

今現在、localhostの8020でHDFSにアクセス出来るようになっているのでこの通り。
しかしこれで起動し、hbase shellでアクセスしてもZooKeeperがどうとかのExceptionでアクセス出来ない。
なので、ZooKeeperをインストールする。

~~~ bash
$ sudo yum install -y zookeeper-server
 (略)
Installed:
  zookeeper-server.x86_64 0:3.4.5+cdh5.0.2+33-1.cdh5.0.2.p0.31.el6

Complete!
$ sudo service zookeeper-server init
No myid provided, be sure to specify it in /var/lib/zookeeper/myid if using non-standalone
$ sudo service zookeeper-server start
JMX enabled by default
Using config: /etc/zookeeper/conf/zoo.cfg
Starting zookeeper ... STARTED
~~~

これで、hbase shellでアクセスしてもエラーメッセージが延々でる。

~~~ bash
$ hbase shell
2014-06-17 07:51:06,250 INFO  [main] Configuration.deprecation: hadoop.native.lib is deprecated. Instead, use io.native.lib.available
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 0.96.1.1-cdh5.0.2, rUnknown, Mon Jun  9 10:48:12 PDT 2014

hbase(main):001:0> status

... ERROR [main] client.HConnectionManager$HConnectionImplementation: The node /hbase is not in ZooKeeper. It should have been written by the master. Check the value configured in 'zookeeper.znode.parent'. There could be a mismatch with the one configured in the master.
	:
	:
~~~

おそらく、hbase-regionserverがないからなのだと思う。しかしregionserverが何であるかはわからないのだが、これをインストールしておく。

~~~ bash
$ sudo yum install -y hbase-regionserver
Installed:
  hbase-regionserver.x86_64 0:0.96.1.1+cdh5.0.2+74-1.cdh5.0.2.p0.21.el6

Complete!
~~~

そして、一度hbase-master, zookeeper, hbase-regionserverを一旦落として、zookeeper, hbase-master, hbase-regionserverの順で起動して見るとうまくいった。

~~~ bash
$ sudo service hbase-regionserver stop
$ sudo service zookeeper-server stop
$ sudo service hbase-master stop

$ sudo service zookeeper-server start
$ sudo service hbase-master start
$ sudo service hbase-regionserver start

$ hbase shell
2014-06-17 07:56:55,943 INFO  [main] Configuration.deprecation: hadoop.native.lib is deprecated. Instead, use io.native.lib.available
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 0.96.1.1-cdh5.0.2, rUnknown, Mon Jun  9 10:48:12 PDT 2014

hbase(main):001:0> status
1 servers, 0 dead, 2.0000 average load

hbase(main):002:0>
~~~

この時点では、

	http://ゲストOSのIPアドレス:60010

でhbase-masterのWebUIが見れる。

![HBase-Web]({{ site.url }}/images/06/17/hbase-webui.png)

ちなみに、regionserverは60030で見れるようである。
ということは、regionserverというのはhbaseのノードのことか・・・。

また、jps(Javaのpsかな？)というコマンドで、今まで上げたサービスのプロセスが見れる。

~~~ bash
$ sudo jps
32107 SecondaryNameNode
31869 DataNode
473 JobHistoryServer
32513 ResourceManager
2174 HRegionServer
1983 HMaster
31980 NameNode
1346 QuorumPeerMain
319 NodeManager
2943 Jps
~~~

WebUIまで見れるようになったので、とりあえずOKとする。

## キーワードまとめ

それぞれの意味は深くわかっていないが、まとめ。

* hdfs, namenode, datanode, YARN, resourecemanager, nodemanager, ZooKeeper, bigtop, jps