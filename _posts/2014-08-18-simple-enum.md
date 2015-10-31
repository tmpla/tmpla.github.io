---
layout: post
title: "Railsでコード値等を扱いやすく出来るsimple_enumを使ってみる"
modified: 2014-08-18 20:47:19 +0900
tags: [ruby,activerecord,simple_enum]
image:
  feature:
  credit:
  creditlink:
comments:
share:
---

データベースを使うアプリケーションで、データ的には0, 1等の数値で持っているけど、
実際に使うときには、男、女みたいな感じで使う場合はよくあります。
データベースのコメントに区分値の内訳を書いておいたりしたりしますね。

これをRails（Ruby）で扱うのに良さそうな、[simple_enum](https://github.com/lwe/simple_enum)と言うのをひょんなトコから知ったのでこれを使ってみようと思いました。

どんなのが例として良いかと思ったのですが、wordpressには投稿ステータスと言うのがあるので、
それをこれで扱ってみたいと思います。

とりあえずblogと言うRailsアプリを作ってみます。
まずはRailsアプリのひな形を作って、Gemfileに「simple_enum」を追記します。

~~~ bash
$ mkdir blog
$ cd blog
$ bundle init
$ vim Gemfile
# A sample Gemfile
source "https://rubygems.org"

gem "rails"
$ bundle install
$ bundle exec rails new .
$ bundle update
$ echo "gem 'simple_enum'" >> Gemfile
$ bundle
~~~


そして、Wordpressのwp_postsのステータスを見てみます。

[データベース構造 - WordPress Codex 日本語版](http://wpdocs.sourceforge.jp/%E3%83%87%E3%83%BC%E3%82%BF%E3%83%99%E3%83%BC%E3%82%B9%E6%A7%8B%E9%80%A0#.E3.83.86.E3.83.BC.E3.83.96.E3.83.AB.EF.BC.9A_wp_posts)

これを見ると、下記のようなステータスがあります。

- 'publish': 公開済み
- 'pending': ペンディング
- 'draft': 草稿
- 'private': プライベート（非公開）
- 'static':（2.0.x 以前はページ）
- 'object':
- 'attachment':
- 'inherit': 継承（添付ファイル、改訂履歴・自動保存のとき）
- 'future': 予約投稿

これを0から順に振っていくようにします。
よくよく見ると、privateとかstaticとか予約語ぽいのもあるのですがとりあえず動いたので置いておいて、
simple_enumで扱うカラムは、「xxxx_cd」と言う風に、_cdと言うプレフィックスをつけとくのがデフォルト設定になっているようなので、status_cdというフィールドを作る事にします。
テーブル自体は、title, content, status（とあとid, timestamp系のやつ）とします。
なのでscaffoldで

~~~ bash
$ bin/rails g scaffold posts title:string content:string status_cd:integer
$ bin/rakd db:migrate
~~~

そして一旦起動して見てみます。

~~~ bash
$ bin/rails s                                                                                                                                                               2.1.2
=> Booting WEBrick
   :
   :
~~~

するとこんな画面が出ます。

![Blog First]({{ site_url }}/images/08/18/blog_first.png)

試しにこんな記事を作ってみたけど

![Blog Test Post First]({{ site_url }}/images/08/18/blog_first_post.png)

もちろんこんな感じになります。

![Blog List First]({{ site_url }}/images/08/18/blog_first_list.png)

これを、一覧では文字列で表示したいのでModelをこんな感じに修正してみます。

まずModelそのコード値を書いちゃいます。

~~~ ruby
class Post < ActiveRecord::Base
+	as_enum :status,
+		publish: 	0, # 公開済み
+		pending: 	1, # ペンディング
+		draft: 		2, #草稿
+		private: 	3, # プライベート（非公開）
+		static:   4, #（2.0.x 以前はページ）
+		object:   5, #
+		attachment: 6,
+		inherit: 	7, # 継承（添付ファイル、改訂履歴・自動保存のとき）
+		future: 	8  # 予約投稿
end
~~~

続いてViewを

~~~ ruby
  <tbody>
    <% @posts.each do |post| %>
      <tr>
        <td><%= post.title %></td>
        <td><%= post.content %></td>
-       <td><%= post.status_cd %></td>
+       <td><%= post.status %></td>
        <td><%= link_to 'Show', post %></td>
        <td><%= link_to 'Edit', edit_post_path(post) %></td>
        <td><%= link_to 'Destroy', post, method: :delete, data: { confirm: 'Are you sure?' } %></td>
      </tr>
    <% end %>
  </tbody>
~~~

すると、リスト表示画面では「publish」に変わっているのが見えます。

![Blog List 1]({{ site_url }}/images/08/18/blog_list_1.png)

次に、更新画面もそのリスト選択にしたいとしたら、posts/_form.html.erbを下記のように変更してみます。

~~~ ruby
  <div class="field">
    <%= f.label :status_cd %><br>
-   <%= f.number_field :status_cd %>
+   <%= f.select :status, enum_option_pairs(Post, :status) %>
  </div>
~~~

するとこんな簡単にリストが！

![Blog New 1]({{ site_url }}/images/08/18/blog_new_1.png)

また、更新用に更新OKのパラメータとしてControllerに登録します。（これはstrong_parametersに登録する、と言う言い方でいいのかな？）

~~~ ruby
    # Never trust parameters from the scary internet, only allow the white list through.
    def post_params
-     params.require(:post).permit(:title, :content, :status_cd)
+     params.require(:post).permit(:title, :content, :status)
    end
~~~

これでウマイこと更新出来ているのもわかります。

![Blog List 2]({{ site_url }}/images/08/18/blog_list_2.png)

あとは、この場合は

~~~ ruby
p = Post.find_by_title('記事１')
p.draft!
p.save
~~~

とやって更新出来たり、

~~~ ruby
Post.publishes
~~~

とやって多分公開一覧が取ってこれたり出来るようです。
コード値が多くなってきたら、こんなのを使ってみるといいですね。