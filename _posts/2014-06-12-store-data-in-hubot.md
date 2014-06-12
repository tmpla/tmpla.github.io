---
layout: post
title: "Hubotでデータを永続化する"
modified: 2014-06-12 13:18:08 +0900
tags: [hubot,brain,persystent]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: true
---
[Hubot](https://hubot.github.com/)でスクリプトを書いているが、データを永続化させるのがよくわからないのでやってみた話。

## 参考サイト

* [hubotスクリプトの書き方とサンプル集 | mitc](http://blog.fumiz.me/2012/08/05/hubot-irc-bot-script/)
* [GitHubがOpsツールの中心として活用しているHubotを使ってみる～インストール、スクリプトの作成、Herokuへのデプロイ～ - Tech-Sketch](http://tech-sketch.jp/2013/12/hubot-install-heroku.html)
* [hubot/src/brain.coffee at master · github/hubot](https://github.com/github/hubot/blob/master/src/brain.coffee)

## robot.brain

永続化データにアクセスするには、robot.brainを通してアクセスするようだ。　　
試しに、下記のようなスクリプトを書いてみた。

~~~ coffee
## vim scripts/test.coffee
module.exports = (robot) ->
  robot.hear /test/, (msg) ->
    console.log robot.brain
~~~

これでHubotを起動して「test」という文字列を入力するとrobot.brainのインスタンスが表示された。

~~~ bash
Hubot> test
Hubot> { data:
   { users:
      { '1': [Object],
        hubot: [Object],
        kyokoshima: [Object]},
     _private: {},
  autoSave: true,
  saveInterval:
   { _idleTimeout: 5000,
     _idlePrev:
      { _idleNext: [Circular],
        _idlePrev: [Circular],
        msecs: 5000,
        ontimeout: [Function: listOnTimeout] },
     _idleNext:
      { _idleNext: [Circular],
        _idlePrev: [Circular],
        msecs: 5000,
        ontimeout: [Function: listOnTimeout] },
     _idleStart: 1402551612289,
     _onTimeout: [Function: wrapper],
     _repeat: true },
  _events: { save: [Function], close: [Function] } }
~~~

この中のdata、つまりrobot.brain.dataにアクセスすれば良さそうではある。
値を作って、robot.brain.saveすれば保存されるはず。

~~~ coffee
## vim scripts/test.coffee
module.exports = (robot) ->
  robot.hear /test/, (msg) ->
    robot.brain.data.test = {id: 'test', value: 0} unless robot.brain.data.test
    robot.brain.data.test.value++
    robot.brain.save
    console.log robot.brain.data.test
~~~

~~~ bash
Hubot> test
Hubot> { id: 'test', value: 1 }
~~~

これで一度hubotを再起動してもtest.valueの値はインクリメントされていくのがわかる。

## アクセッサ
ところで、brain.coffeeを見てみると、brain.set(key, value)やget(key)のようなメソッドがある。
これを使うのが正しいのじゃないかと思ってやってみた。

~~~ coffee
module.exports = (robot) ->
  robot.hear /test/, (msg) ->
    v = robot.brain.get('test') ? 0
    robot.brain.set('test', ++v)
    console.log robot.brain.data
~~~

~~~ bash
Hubot> test
Hubot> { users:
   { '1': { id: '1', name: 'Shell', room: 'Shell' },
     hubot: { id: 'hubot', name: 'hubot', room: '#test' },
     kyokoshima: { id: 'kyokoshima', name: 'kyokoshima', room: '#test' } },
  _private: { test: 7 },
  test: { id: 'test', value: 5 } }
~~~

robot.brain.data.\_privateに入っているようだ。
その他、mergeDataというメソッドもあり、名の通りマージするようだが、これは_private以下にあっても出来ないやつだと思う。
なので、深い階層データならdata直下に、お手軽に使うならアクセサを使えば良いのではなかろうか。


