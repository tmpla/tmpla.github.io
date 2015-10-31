---
layout: post
title: "Capistranoのデフォルトdeployタスクを確認してみる"
modified: 2014-06-26 18:12:55 +0900
tags: [Capistrano,task,default]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---

[Capistrano](http://capistranorb.com/)でデプロイしてるでしょうか。
インストールしてちょっと設定を変えれば結構そのまま使えたりするのも良いですが、色々カスタマイズして使いたくなってきます。

デフォルトのままならば内部をそんなに意識せずともいいのですが、カスタマイズするにはどのような思想なのかを勉強する必要が出てきます。
これが面倒くさいからシェルか、手デプロイで、というのは悲しいです。

細かいところは色々覚える必要があるのですが、まずはデフォルトではどんな事をやっているのかを知りたい。

これは[framework.rake](https://github.com/capistrano/capistrano/blob/master/lib/capistrano/tasks/framework.rake)の最後の方にあるように、

~~~ ruby
desc 'Deploy a new release.'
task :deploy do
  set(:deploying, true)
  %w{ starting started
      updating updated
      publishing published
      finishing finished }.each do |task|
    invoke "deploy:#{task}"
  end
end
~~~

で、

1. deploy:starting
2. deploy:started
3. deploy:updating
4. deploy:updated
5. deploy:publishing
6. deploy:published
7. deploy:finishing
8. deploy:finished

の順にタスクを呼び出しています。そしてそのタスクのデフォルトの内容は[deploy.rake](https://github.com/capistrano/capistrano/blob/master/lib/capistrano/tasks/deploy.rake)に書いてあります。
例えばstartingを見てみると

~~~ ruby
 task :starting do
    invoke 'deploy:check'
    invoke 'deploy:set_previous_revision'
  end
~~~

ディレクトリのチェック等をして、前回のデプロイ済みリビジョンを内部変数にセットする、みたいなことしてますね。
タスク名からも大体わかるしこの辺りは一目瞭然かと思います。

しかし、ほんとにそうなの？実行時に書き換えたりしてるんじゃないの？と思ったのでこれらタスクをオーバーライドして見てみました。
config/deploy.rbに

~~~ ruby
namespace :deploy do |ns|
  require 'pp'
  ns.tasks.each do |t|
    pp t
    t.clear_actions
    t.enhance do
      pp "Override #{t}"
    end
  end
  :
  （略）
~~~

:deployのTaskの名前を表示し、clear_actionsで事前タスクはそのままに、actionをクリアし、そのタスクではタスク名を出力するだけの処理をしてます。
普通にtaskを定義するだけだと、もともとあったactionはそのままに、新しいタスクが追加されるようなので一度clearしています。
するとこんな感じに出ます。

~~~ bash
<Rake::Task deploy:check => []>
<Rake::Task deploy:check:directories => []>
<Rake::Task deploy:check:linked_dirs => []>
<Rake::Task deploy:check:linked_files => []>
<Rake::Task deploy:check:make_linked_dirs => []>
<Rake::Task deploy:cleanup => []>
<Rake::Task deploy:cleanup_rollback => []>
<Rake::Task deploy:failed => []>
<Rake::Task deploy:finished => []>
<Rake::Task deploy:finishing => []>
<Rake::Task deploy:finishing_rollback => []>
<Rake::Task deploy:log_revision => []>
<Rake::Task deploy:new_release_path => []>
<Rake::Task deploy:published => []>
<Rake::Task deploy:publishing => []>
<Rake::Task deploy:restart => []>
<Rake::Task deploy:revert_release => [rollback_release_path]>
<Rake::Task deploy:reverted => []>
<Rake::Task deploy:reverting => []>
<Rake::Task deploy:rollback => []>
<Rake::Task deploy:rollback_release_path => []>
<Rake::Task deploy:set_current_revision => []>
<Rake::Task deploy:set_previous_revision => []>
<Rake::Task deploy:started => []>
<Rake::Task deploy:starting => []>
<Rake::Task deploy:symlink:linked_dirs => []>
<Rake::Task deploy:symlink:linked_files => []>
<Rake::Task deploy:symlink:release => []>
<Rake::Task deploy:symlink:shared => []>
<Rake::Task deploy:updated => []>
<Rake::Task deploy:updating => [new_release_path]>
"Override deploy:starting"
"Override deploy:started"
"Override deploy:new_release_path"
"Override deploy:updating"
"Override deploy:updated"
"Override deploy:publishing"
"Override deploy:restart"
"Override deploy:published"
"Override deploy:finishing"
"Override deploy:finished"
~~~

これは実行結果なので、ちゃんとその順番に実行されていることが見えます。
この順番を意識して、:after, :before等で定義していけば良いですね。
多分普通はこんな完全オーバーライドはやらないんじゃないかと思いますが・・・。