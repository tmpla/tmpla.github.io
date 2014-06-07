---
layout: post
title: "chefのレシピをgithubに置くときの名前付けの一案"
modified: 2014-06-07 18:02:48 +0900
tags: [chef,recipe,github]
image:
  feature:
  credit:
  creditlink:
comments:
share: true
---
Chefのレシピを色々githubで見ていると、こんな感じでリポジトリの名前をつけている事が多いような気がする。

> chef-aaaa_bbbb

なんでハイフンとアンダースコアが混ぜてるのか気になったが、やってみるとこれはこれでいいのかなと思った。
プレフィックス(chef-)は、chefのレシピであることを示すと言う感じで考える。

レシピ名は、当然、ハイフンより後を使って、とりあえずberkshelfでは下記のように使う。
例として、chef-capture\_web\_pageと言うレシピがあったとすると、

~~~ ruby
cookbook "capture_web_page", github: "kyokoshima/chef-capture_web_page"
~~~

こんな感じで。これはコミュニティレシピにしたとしてもこれでいいだろう。

じゃ、レシピ名はハイフン区切りでいいじゃん、と思うけども何故スネークケース（単語区切りがアンダースコア）を使っているかと言うと、
シンボルに出来るのが便利だと言うこと。ハイフンだとシンボルに出来ないと思うので。

レシピ名はシンボルでアクセス出来ると嬉しいこともある。