---
layout: post
title: "RailsのViewの変数を全て表示する方法"
modified: 2014-06-29 18:53:04 +0900
tags: [Rails,variables,view]
image:
  feature:
  credit:
  creditlink:
comments:
share:
---

RailsでViewの変数を表示する方法は下記の方法で出来る。

~~~ ruby
<%= debug @posts %>
~~~

Controllerからの変数が全部わかっていればこれでいいのだけど、なんだか色々ViewHelperを含むGemを使っていて、
もう、Viewには何が来てるのか全部見たいよ！と言う時はこれで。

~~~ ruby
<%= debug self.inspect %>
~~~

さらに、これだと超見にくいので、これがベストだと思う。

~~~ ruby
<%= debug self.pretty_inspect %>
~~~

めったに使わないかもしれないけど、ちょっとした息抜きに是非。
