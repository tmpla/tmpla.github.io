---
layout: post
title: "JSONをコンソールで扱う"
modified: 2014-06-28 16:45:57 +0900
tags: [json,jq,ohai]
image:
  feature:
  credit:
  creditlink:
comments:
share:
---

Chefのレシピを作っていると、あれ、platform_familyはなんだっけな？と思い、調べるとするとohaiを使ってみたりします。
[ohai](http://docs.opscode.com/ohai.html)は、それを実行しているマシンの情報をかなり詳細に取得してくれます。
レシピでもこの情報を元に、例えばOS毎に条件分岐させて処理するための変数を用意しているようです。

これはいいのですが、ohaiコマンドを打つだけだと、全部の情報が出力されてしまうので、ちょっと見づらいし、目的のキーを探すのも大変です。普通にohaiとうつとこんな感じに。

~~~ bash
$ ohai
{
  "cpu": {
    "real": 2,
    "total": 4,
    "mhz": 1700,
    "vendor_id": "GenuineIntel\n",
    "model_name": "Intel(R) Core(TM) i5-2557M CPU @ 1.70GHz\n",
    "model": 42,
    "family": 6,
    "stepping": 7,
    "flags": [
      "fpu",
      "vme",
      "de",
      "pse",
      :
      :
      (以下延々と続く)
~~~

なので、jsonをコンソール上でフィルタしてくれるようなツールがないかなと思っていたら、下記ページで[jq](http://stedolan.github.io/jq/)と言うのを知りました。

[軽量JSONパーサー『jq』のドキュメント：『jq Manual』をざっくり日本語訳してみました ｜ Developers.IO](http://dev.classmethod.jp/tool/jq-manual-japanese-translation-roughly/)

とりあえずMacにはHomebrewでインストール出来るようなのでインストールして使ってみました。
一応インストールはこんな感じで。

~~~ bash
$ brew install jq
~~~

で早速やってみました！

~~~ bash
$ ohai | jq '.'
{
  "root_group": "wheel",
  "current_user": "kenichi",
  "etc": {
    "group": {
      "wheel": {
        "members": [
          "root"
        ],
        "gid": 0
      },
      :
      :
      (以下延々と続く)
~~~

以下延々と続くのは変わりませんが、まずキーと値に色が付いているのでちょっと見やすいです。
（この記事ではシンタックスハイライトが効いているのでohaiコマンドの方でも色付けされていますが…）
また、何かの順でソートされているような感じもします。

前述のplatform_familyだけフィルタリングするとしたらこんな風に。

~~~ bash
$ ohai | jq '.platform_family'
"mac_os_x"
~~~

おお、ちゃんと出来とる。当然ohaiにかぎらず、jsonを見やすくするにはこれいいですね。


