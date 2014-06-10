---
layout: post
title: "Githubのリポジトリをhubコマンドで作る方法"
modified: 2014-06-08 15:44:03 +0900
tags: [github,hub]
image:
  feature:
  credit:
  creditlink:
comments:
share: true
---
[hubコマンド](https://github.com/github/hub)というのがあるが、これを使うとgithubのリポジトリの操作が大分楽になる。

githubに新しいリポジトリを作る場合は、これがなければgithubのwebページでリポジトリを作成し、
ローカルリポジトリのremoteとこれをひもづける様な作業が必要だったが、hubコマンドがあればgithub側のリモートリポジトリも作成できる。

~~~ bash
% mkdir test_repo
% cd test_repo
% git init .
% hub create
~~~

ユーザー名等が設定してあれば（なければhttpsでのアクセスで聞かれるが）、これでtest_repoと言うリモートリポジトリが出来ちゃう。
Chefのレシピ等で、リモートリポジトリ名をローカルリポジトリと違うのにしたい場合は下記のようにする。

~~~ bash
% mkdir test_repo
% cd test_repo
% git init .
% hub create chef-test_repo
~~~

ただ名前が変更出来るのですよ、ということを言いたいだけでした。
