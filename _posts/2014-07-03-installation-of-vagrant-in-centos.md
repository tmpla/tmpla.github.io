---
layout: post
title: "CentOSにVagrantをインストールする"
modified: 2014-07-03 09:58:30 +0900
tags: [Vagrant,CentOS]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---

vagrant「で」CentOSをインストールする話ではなくて、CentOS「に」vagrantをインストールする話です。

前回の[CentOSにVirtualBoxをインストールする]({{ site_url }}/installation-of-virtualbox-in-centos)で、VirtualBoxをインストールした後に
Vagrantをインストールして仮想マシンを動かして見た話。
前回同様、OSはCentOS6.5の64bitである。
ちなみに、ハードウェアは[ PowerEdge R320 rack server details  | Dell ](http://www.dell.com/us/business/p/poweredge-r320/fs)というやつだと思う。

## Vagrantのインストール

vagrantのインストール自体は全然問題なく出来た。

[Download Vagrant - Vagrant](http://www.vagrantup.com/downloads)からrpmのリンクをコピペし、

~~~ bash
$ sudo rpm -ivh https://dl.bintray.com/mitchellh/vagrant/vagrant_1.6.3_x86_64.rpm
~~~

でインストール出来る。

~~~ bash
$ vagrant version
Installed Version: 1.6.3
Latest Version: 1.6.3

You're running an up-to-date version of Vagrant!
~~~

この後、vagrant boxをインポートしつつvmを作ってみる。

~~~ bash
$ vagrant init chef/centos-6.5
  :
  (略)
  :
$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Checking if box 'chef/centos-6.5' is up to date...
==> default: Clearing any previously set forwarded ports...
==> default: Fixed port collision for 22 => 2222. Now on port 2200.
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 22 => 2200 (adapter 1)
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2200
    default: SSH username: vagrant
    default: SSH auth method: private key
    default: Warning: Connection timeout. Retrying...
    default: Warning: Connection timeout. Retrying...
    default: Warning: Connection timeout. Retrying...
    default: Warning: Connection timeout. Retrying...
    default: Warning: Connection timeout. Retrying...
    :
    (このあとずーっと続く)
~~~

どうも起動はされていそうだけどssh接続が出来ないように思える。
が、CLIからだと、どうしても起動プロセスがどうなっているのかがわからない。
しょうがないので、CentOSにDesktopをインストールし、起動してみた。

~~~ bash
$ sudo yum groupinstall -y "Desktop"
~~~

これで、startxすれば良いのかと思ったが、デスクトップは起動するもハードに直接挿しているUSBのキーボードとマウスが全然反応しなくなってしまった・・・。
で色々調べて見ると、haldaemonというサービスが起動していないとダメなようだ。

[CentOS • View topic - Keyboard & Mouse Stop Working When Starting Gnome or KDE](https://www.centos.org/forums/viewtopic.php?t=4661)

ちなみに、haldaemonと言うのはハードウェアのプラグ＆プレイを実現するためのサービスのようである。
messagebusというサービスも起動したほうが良さそうなので、起動してからstartxでうまく行ったようだ。
(別端末でssh繋ぎっぱなしだったのでそっちからやった)

~~~ bash
$ sudo service messagebus start
$ sudo service haldaemon start
$ startx
~~~

この状態で、起動したいvmのVagrantfileに下記を追加（コメントアウトされているところを外すでもいい）して起動する。

config.vm.provider "virtualbox" do |vb|
  # Don't boot with headless mode
  vb.gui = true
end

するとデスクトップにVirtualBoxのGUIで起動プロセスが表示される。
すると、起動時に

>
VT-x/AMD-V hardware acceleration has been enabled, but is not operational. Your 64-bit guest will fail to detect a 64-bit CPU and will not be able to boot.

なんて出ている。あれ、これ64bitなのは間違いないのになんでだろうと思って色々調べてみると、BIOSでVirtuarization Technologyというのが有効化されていないとダメらしい。

このラックの場合は、Dellの起動画面のプログレスの最後のほうでF2を押すとBIOS設定画面に入れるが、そこでProcessor SettingsでIntel Virtualization Technologyを「Enabled」に変更すればOK。

これで普通に起動できた。