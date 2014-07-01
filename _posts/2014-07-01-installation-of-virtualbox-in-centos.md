---
layout: post
title: "CentOSに　VirtualBoxをインストールする"
modified: 2014-07-01 16:20:40 +0900
tags: [VirtualBox,CentOS]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---

VirtualBoxのゲストをCentOSにするという話ではなくて、
CentOSにVirtualBoxをインストールして見ました。

CentOSのバージョンは6.5の64bitです。
少なくとも、yum updateで各種ライブラリ等はアップデートされている状態とします。
インストールするVirtualBoxのバージョンは4.3.12です。

## まずrpmでインストール

~~~ bash
$ rpm -ivh http://download.virtualbox.org/virtualbox/4.3.12/VirtualBox-4.3-4.3.12_93733_el6-1.x86_64.rpm
http://download.virtualbox.org/virtualbox/4.3.12/VirtualBox-4.3-4.3.12_93733_el6-1.x86_64.rpm を取得中
警告: /var/tmp/rpm-tmp.5ep3Dt: ヘッダ V4 DSA/SHA1 Signature, key ID 98ab5139: NOKEY
エラー: 依存性の欠如:
        libGL.so.1()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libQtCore.so.4()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libQtGui.so.4()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libQtNetwork.so.4()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libQtOpenGL.so.4()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libSDL-1.2.so.0()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libX11.so.6()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libXcursor.so.1()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libXext.so.6()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libXinerama.so.1()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libXmu.so.6()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libXt.so.6()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libasound.so.2()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libpng12.so.0()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libpng12.so.0(PNG12_0)(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています

~~~

ということなので、上から片付けていきます。

## libGLインストール

~~~ bash
$ sudo yum install -y libGL
  :
  （略）
  :
Installed:
  mesa-libGL.x86_64 0:9.2-0.5.el6_5.2

Dependency Installed:
  libX11.x86_64 0:1.5.0-4.el6                          libX11-common.noarch 0:1.5.0-4.el6
  libXau.x86_64 0:1.0.6-4.el6                          libXdamage.x86_64 0:1.1.3-4.el6
  libXext.x86_64 0:1.3.1-2.el6                         libXfixes.x86_64 0:5.0-3.el6
  libXxf86vm.x86_64 0:1.1.2-2.el6                      libxcb.x86_64 0:1.8.1-1.el6
  mesa-dri-drivers.x86_64 0:9.2-0.5.el6_5.2            mesa-dri-filesystem.x86_64 0:9.2-0.5.el6_5.2
  mesa-dri1-drivers.x86_64 0:7.11-8.el6                mesa-private-llvm.x86_64 0:3.3-0.3.rc3.el6

Complete!

~~~

ちょっと減った。

## qtのインストール

~~~ bash
$ rpm -ivh http://download.virtualbox.org/virtualbox/4.3.12/VirtualBox-4.3-4.3.12_93733_el6-1.x86_64.rpm
http://download.virtualbox.org/virtualbox/4.3.12/VirtualBox-4.3-4.3.12_93733_el6-1.x86_64.rpm を取得中
警告: /var/tmp/rpm-tmp.Pvk7SO: ヘッダ V4 DSA/SHA1 Signature, key ID 98ab5139: NOKEY
エラー: 依存性の欠如:
        libQtCore.so.4()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libQtGui.so.4()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libQtNetwork.so.4()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libQtOpenGL.so.4()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libSDL-1.2.so.0()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libXcursor.so.1()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libXinerama.so.1()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libXmu.so.6()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libXt.so.6()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libasound.so.2()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libpng12.so.0()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libpng12.so.0(PNG12_0)(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
~~~

~~~ bash
$ yum install -y qt
Installed:
  qt.x86_64 1:4.6.2-28.el6_5

Complete!
~~~

続いて、libQtGuiをなんとかしたいが、これはqt-x11というのに入っているらしい。

~~~ bash
$ rpm -ivh http://download.virtualbox.org/virtualbox/4.3.12/VirtualBox-4.3-4.3.12_93733_el6-1.x86_64.rpm
http://download.virtualbox.org/virtualbox/4.3.12/VirtualBox-4.3-4.3.12_93733_el6-1.x86_64.rpm を取得中
警告: /var/tmp/rpm-tmp.gPn9dZ: ヘッダ V4 DSA/SHA1 Signature, key ID 98ab5139: NOKEY
エラー: 依存性の欠如:
        libQtGui.so.4()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libQtOpenGL.so.4()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libSDL-1.2.so.0()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libXcursor.so.1()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libXinerama.so.1()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libXmu.so.6()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libXt.so.6()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libasound.so.2()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libpng12.so.0()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libpng12.so.0(PNG12_0)(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています

~~~

~~~ bash
$ sudo yum install -y qt-x11
  :
  （略）
Installed:
  qt-x11.x86_64 1:4.6.2-28.el6_5

Dependency Installed:
  alsa-lib.x86_64 0:1.0.22-3.el6                                cairo.x86_64 0:1.8.8-3.1.el6
  cdparanoia-libs.x86_64 0:10.2-5.1.el6                         fontconfig.x86_64 0:2.8.0-3.el6
  freetype.x86_64 0:2.3.11-14.el6_3.1                           gstreamer.x86_64 0:0.10.29-1.el6
  gstreamer-plugins-base.x86_64 0:0.10.29-2.el6                 gstreamer-tools.x86_64 0:0.10.29-1.el6
  iso-codes.noarch 0:3.16-2.el6                                 lcms-libs.x86_64 0:1.19-1.el6
  libICE.x86_64 0:1.0.6-1.el6                                   libSM.x86_64 0:1.2.1-2.el6
  libXcursor.x86_64 0:1.1.13-6.20130524git8f677eaea.el6         libXft.x86_64 0:2.3.1-2.el6
  libXi.x86_64 0:1.6.1-3.el6                                    libXinerama.x86_64 0:1.1.2-2.el6
  libXrandr.x86_64 0:1.4.0-1.el6                                libXrender.x86_64 0:0.9.7-2.el6
  libXv.x86_64 0:1.0.7-2.el6                                    libgudev1.x86_64 0:147-2.51.el6
  libjpeg-turbo.x86_64 0:1.2.1-3.el6_5                          libmng.x86_64 0:1.0.10-4.1.el6
  libogg.x86_64 2:1.1.4-2.1.el6                                 liboil.x86_64 0:0.3.16-4.1.el6
  libpng.x86_64 2:1.2.49-1.el6_2                                libthai.x86_64 0:0.1.12-3.el6
  libtheora.x86_64 1:1.1.0-2.el6                                libtiff.x86_64 0:3.9.4-10.el6_5
  libvisual.x86_64 0:0.4.0-9.1.el6                              libvorbis.x86_64 1:1.2.3-4.el6_2.1
  mesa-libGLU.x86_64 0:9.2-0.5.el6_5.2                          pango.x86_64 0:1.28.1-7.el6_3
  phonon-backend-gstreamer.x86_64 1:4.6.2-28.el6_5              pixman.x86_64 0:0.26.2-5.1.el6_5
  qt-sqlite.x86_64 1:4.6.2-28.el6_5                             xml-common.noarch 0:0.6.3-32.el6

Complete!

~~~

## libSDLのインストール

~~~ bash
$ rpm -ivh http://download.virtualbox.org/virtualbox/4.3.12/VirtualBox-4.3-4.3.12_93733_el6-1.x86_64.rpm
http://download.virtualbox.org/virtualbox/4.3.12/VirtualBox-4.3-4.3.12_93733_el6-1.x86_64.rpm を取得中
警告: /var/tmp/rpm-tmp.Tniv5z: ヘッダ V4 DSA/SHA1 Signature, key ID 98ab5139: NOKEY
エラー: 依存性の欠如:
        libSDL-1.2.so.0()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libXmu.so.6()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libXt.so.6()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
~~~

~~~ bash
$ yum install -y SDL
  :
  (略)
Installed:
  SDL.x86_64 0:1.2.14-3.el6

Complete!

~~~

## libXmuのインストール

~~~ bash
$ rpm -ivh http://download.virtualbox.org/virtualbox/4.3.12/VirtualBox-4.3-4.3.12_93733_el6-1.x86_64.rpm
http://download.virtualbox.org/virtualbox/4.3.12/VirtualBox-4.3-4.3.12_93733_el6-1.x86_64.rpm を取得中
警告: /var/tmp/rpm-tmp.syb4iI: ヘッダ V4 DSA/SHA1 Signature, key ID 98ab5139: NOKEY
エラー: 依存性の欠如:
        libXmu.so.6()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
        libXt.so.6()(64bit) は VirtualBox-4.3-4.3.12_93733_el6-1.x86_64 に必要とされています
~~~

~~~ bash
$ yum install -y libXmu
  :
  (略)
Installed:
  libXmu.x86_64 0:1.1.1-2.el6

Dependency Installed:
  libXt.x86_64 0:1.1.3-1.el6

Complete!

~~~

もう、それぞれのライブラリが何なのかは不明で、キーワードで手動で依存関係を解消しているですね・・・。

## VirtualBoxのインストール

ついに本命のインストールが出来ました。

~~~ bash
$ sudo rpm -ivh http://download.virtualbox.org/virtualbox/4.3.12/VirtualBox-4.3-4.3.12_93733_el6-1.x86_64.rpm
http://download.virtualbox.org/virtualbox/4.3.12/VirtualBox-4.3-4.3.12_93733_el6-1.x86_64.rpm を取得中
警告: /var/tmp/rpm-tmp.iaBXkd: ヘッダ V4 DSA/SHA1 Signature, key ID 98ab5139: NOKEY
準備中...                ########################################### [100%]
   1:VirtualBox-4.3         ########################################### [100%]

Creating group 'vboxusers'. VM users must be member of that group!

No precompiled module for this kernel found -- trying to build one. Messages
emitted during module compilation will be logged to /var/log/vbox-install.log.

Stopping VirtualBox kernel modules [  OK  ]
Recompiling VirtualBox kernel modules [失敗]
  (Look at /var/log/vbox-install.log to find out what went wrong)

~~~

ですが、何やら失敗している模様でした。
ここでこれを見つけました。

[CentOS 上で Vagrant を導入するまでのメモ（CUI） - Qiita](http://qiita.com/Itomaki/items/9a6a314a853cdcd00f80)
[HowTos/Virtualization/VirtualBox - CentOS Wiki](http://wiki.centos.org/HowTos/Virtualization/VirtualBox)

ここで、VirtualBoxのyumリポジトリがあるのを知ったので追加しています。
あと、dkmsというものをインストールするために、RPMForgeのリポジトリを追加します。

~~~ bash
# cd /etc/yum.repos.d/
# curl -O http://download.virtualbox.org/virtualbox/rpm/rhel/virtualbox.repo
# rpm -ivh http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.3-1.el6.rf.x86_64.rpm

~~~

ここで、wikiに記載があった手順に切り替えます。

~~~ bash
$ sudo yum install dkms
$ sudo yum groupinstall "Development Tools"
$ sudo yum install kernel-devel
~~~

kernel-develでカーネルのソースコードがインストールされるが、/lib/modules/2.6.32-431.el6.x86_64/の「build」のシンボリックリンクがおかしな事になっており見つけられないようなので編集しました。

~~~ bash
# cd /lib/modules/2.6.32-431.el6.x86_64/
# rm build
# ln -s /usr/src/kernels/2.6.32-431.20.3.el6.x86_64/ build
~~~

これでセットアップスクリプトを起動します。

~~~ bash
# service vboxdrv setup
Stopping VirtualBox kernel modules                         [  OK  ]
Uninstalling old VirtualBox DKMS kernel modules            [  OK  ]
Trying to register the VirtualBox kernel modules using DKMS[  OK  ]
Starting VirtualBox kernel modules                         [  OK  ]

~~~

おお、なんかうまく行ったようなので確認したいがisoイメージをインポートしたりするのが面倒くさいので、次回vagrantでやる事にします。

これを踏まえて整理すると、

1. RPMForge, VirtualBoxのリポジトリを追加する。
2. yum groupinstall "Development Tools"を実行する
3. yum install -y libGL qt qt-x11 SDL libXmu dkms kernel-develをする
4. /lib/modules/2.6.32-431.el6.x86_64/buildのシンボリックリンクを/usr/src/kernels/2.6.32-431.20.3.el6.x86_64/に当てる(もちろん、2.6.32-431.20.3.el6の場合)
5. yum install -y VirtualBox-4.3

とやるのが良さそうです。