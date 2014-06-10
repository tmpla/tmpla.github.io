---
layout: post
title: ".ssh/configおすすめ最小限設定"
modified: 2014-06-10 10:34:56 +0900
tags: [ssh,config]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: true
---
最近vagrantで色々なマシンへsshすることが多くなったので、private_networkで扱う時も.ssh/configに名前をつけて書いてることが多くなった。

## ssh_config

ちなみに.ssh/configとは、ssh接続する際に、接続するホストへの認証に必要な情報などを、ホストごとに分けて書いておけるファイルである。

[SSH_CONFIG (5)](http://www.unixuser.org/~euske/doc/openssh/jman/ssh_config.html)

よくある使い方としては、AホストとBホストへ接続する場合、Aホストではid_rsa_a、Bホストではid_rsa_bの秘密鍵を使ってログインしたいとすると
（IPアドレスは適当）

~~~ bash
Host A
	Hostname 192.168.33.10
	IdentityFile ~/.ssh/id_rsa_a
Host B
	Hostname 192.168.33.20
	IdentityFile ~/.ssh/id_rsa_b
~~~

で、ssh Aでid_rsa_aの鍵を、ssh Bでid_rsa_bの鍵を使って接続しようとしてくれる。
この設定について、今ん所これで困ってない、というところまでの段階に来たのでメモしておく。

## おすすめ設定

~~~ bash
Host [ホスト名]						... (1)
  Hostname [正引きまたは逆引きホスト名]   	... (2)
  User [接続ユーザ名]					... (3)
  IdentityFile [秘密鍵ファイル名]		... (4)
  StrictHostKeyChecking no 			... (5)
  UserKnownHostsFile /dev/null		... (6)
  ServerAliveInterval 15			... (7)
~~~

(1) SSH接続で使うホスト名と言うかエイリアス。短くすると楽。SSHを使わないところには影響なし。  
(2) 普通のネットワークホスト名。ping ホスト名で使えるものを指定すれば良い。  
(3) SSH接続するユーザ名。指定なければカレントユーザになる。  
(4) 接続に使う秘密鍵名。デフォルトでは~/.ssh/id\_rsaなどが使えるが、ここで任意のファイルを指定できる。  
(5) 初めて接続するホストの場合、このまま接続処理を続けて問題ないかをユーザーに確認するかどうかを聞かれるが、それを聞かれないようにする。  
(6) 初めて接続するホストへの接続を行った場合、デフォルトでは~/.ssh/known\_hostsにホスト側にある公開鍵？が記録されて、2度目以降はおそらくその鍵を使用して接続するようだが、それを/dev/nullへ捨ててknown\_hostsに記載しないようにする。こうすると、同じIPアドレスで別のマシンを設定した場合に、known\_hostsからその設定をいちいち消さなくていいので便利。  
(7) ここに指定した数値（例では15秒)ごとに、サーバへ応答確認をする。これにより放置しておいてもサーバが生きていれば放置しても接続が切れない。 

デフォルトでyesのTCPKeepAliveなども重要かと思うが、その辺りは書いていない。