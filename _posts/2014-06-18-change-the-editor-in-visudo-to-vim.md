---
layout: post
title: "visudoで使うエディタをvimに変更したい"
modified: 2014-06-18 10:04:34 +0900
tags: [sudo,visudo,vim]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: true
---
sudo出来るユーザーなどを編集するには、/etc/sudoersというファイルを編集することになるが、
これを直接編集するのはナンセンスで、普通はvisudoというコマンドを通して編集する。

[sudoersを変更する、よく使う設定例 - それマグで！](http://takuya-1st.hatenablog.jp/entry/20090806/1249554458)

これを、sudo visudoで編集すると、エディタがviで使いにくいので、vimにする方法を一つ考えた。

そもそもどこでviを指定しているのかというと、man visudoに書いてある。

> There is a hard-coded list of one or more editors that visudo will use set at compile-time that may be overridden
     via the editor sudoers Default variable.  This list defaults to /usr/local/bin/vi.  Normally, visudo does not
     honor the VISUAL or EDITOR environment variables unless they contain an editor in the aforementioned editors list.

とあるので、VISUALかEDITORにvimのパスを書けば良いようだ。
さらに、今のところvimじゃなくてどうしてもviを使いたい！ということがないので、システムグローバルに変更しちゃいたい。
これは、/etc/environmentを編集した。

~~~ bash
$ vim /etc/environment
EDITOR=/usr/local/bin/vim
~~~

これでsudo visudoしてみたらちゃんと色付けとかされてる。当然、suしてvisudoでも。

