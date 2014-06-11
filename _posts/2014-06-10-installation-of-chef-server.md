---
layout: post
title: "Chef Serverセットアップ"
modified: 2014-06-10 15:04:15 +0900
tags: [chef,server,node]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
status: 
published: 
---
## 環境準備
環境はすべてMacの中のVirtualboxをvagantで用意したもの。  
boxはCentos6.5-x64で下記のように構成  
IPアドレスは説明のため実際とは違う妄想。

* Chef Server 1台 192.168.33.10
* Chef Workstation 1台 192.168.33.20
* Chef Node 3台 192.168.33.31~33

操作ユーザーは「vagrant」で行っている。なので、vagrantのホストの~/.vagrant.d/insecure\_pryvate\_keyを使って鍵認証でssh出来る前提。  
[Opscodeのドキュメント](http://docs.opscode.com/install_server.html)を見ると、hostnameを付与したほうが良さそうな感じで書いてあるが、どこらへんに効いてくるかというと、
nodeを追加した時に、変更不可能なattributesで「fqdn」「ipaddress」などがあるのだが、ここの値はnode側でohaiを使って取られているということ。  
そのohaiは多分、hostnameコマンドでそれらを取得しているようなので、node側でhostname --fqdnでとれた値を使わないといけない。  
これはDNSなどがあれば良いが、hostnameコマンドで直接変えたのと今回はnode側のマシンの/etc/hostsにエントリを追加して解決した。  
追加しないと、すべて「localhost」になってしまうのでアクセス出来ない。

またWorkstationにはvagrantは入っていないので、ホストマシンの~/.vagrant.d/insecure_private_keyをVagrantfileがおいてあるパスへコピーして使いまわしている。（ゲストでは/vagrant/insecure_private_keyでアクセス出来るので）

~~~ bash
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.33.31 node1
~~~

また､Workstation側でknife sshというコマンドをあとで使うが、これはnode情報をChef Serverから取得しそのマシンに対してfqdn(かIPアドレス)を使って接続するようなのでWorkstation側でもnodeで設定したfqdnから名前解決出来るようにしなくてはならない。　　
ちなみに、すべてのマシンはvagrantのpublic_networkを設定しているが、ohaiで取れるIPアドレスはNAT用のIPアドレス（10.0.2.15だった)が最初で、それを使用されてしまうためIPアドレス情報は使うのを断念した。
まとめると、

* node側マシンではhostname --fqdnコマンドを使えるようにするため/etc/hostsにグローバルIP用のエントリを追加する
* Workstationマシンでは、与えられたnodeの名前を使ってsshするため/etc/hostsにnodeへのエントリを追加する。

DNSがあればそれが一番ラクかもしれない。ちなみに、Chef Serverは特に名前解決出来なくても大丈夫だと思う。（多分・・・）

## 参考ページ
[Install Chef Server 11.x — Chef Docs](http://docs.opscode.com/install_server.html)  
[【Chef 11版】Chef Server環境セットアップ手順の紹介 | IDC Frontier Engineers' Blog](http://www.idcf.jp/blog/cloud/chef-11/)  
[サーバー構築自動化ツール Chef 最新版のインストール方法 - Hive Color](http://hivecolor.com/id/46)
[Getting Started with Chef Server. Part 2](http://leopard.in.ua/2013/09/01/chef-server-getting-started-part-2/)


## Chef Serverインストール

[Install Chef | Chef](http://www.getchef.com/chef/install/)

からChef Serverタブを選んでrpmをダウンロード、そしてインストールする。

~~~ bash
$ curl -O https://opscode-omnibus-packages.s3.amazonaws.com/el/6/x86_64/chef-server-11.1.1-1.el6.x86_64.rpm
	：
$ sudo rpm -ivh chef-server-11.1.1-1.el6.x86_64.rpm                             
警告: chef-server-11.1.1-1.el6.x86_64.rpm: ヘッダ V4 DSA/SHA1 Signature, key ID 83ef826a: NOKEY
準備中...                ########################################### [100%]           
   1:chef-server            ########################################### [100%]       
Thank you for installing Chef Server!                                                
                                                                                     
The next step in the install process is to run:                                      
                                                                                     
sudo chef-server-ctl reconfigure
~~~

## Chef Server設定

「sudo chef-server-ctl reconfigure」を実行しよう！とのことなので実行してみる。

~~~ bash
$ sudo chef-server-ctl reconfigure
~~~

すると何やら色々なレシピが実行される。  
この時点で443ポートでWebUIがアクセス可能になっているはず。  
終わったら下記コマンドでtestをしてみて成功することを確認する。  

~~~ bash
$ sudo chef-server-ctl test
~~~

## Chef Workstation設定
何はともあれ、chef-clientをインストールする。

~~~ bash
$ curl -L https://www.opscode.com/chef/install.sh | sudo bash
 :
$ chef-client -v
Chef: 11.12.8
~~~

### gitインストール
よしなに

~~~ bash
$ yum install -y git
~~~

### chef-repoひな形コピー

~~~ bash
$ git clone git://github.com/opscode/chef-repo.git 
~~~

とりあえず~/chef-repo以下に全部収めたかったので、それを指定する。  
chef-serverで作成したadmin.pemとchef-validator.pemは、~/chef-repo/.chef/以下にコピーしておく。
下記コマンド中の(変更)は変更したところ。

~~~ bash
$ cd chef-repo
$ knife configure --initial
WARNING: No knife configuration file found
Where should I put the config file? [/home/vagrant/.chef/knife.rb] ~/chef-repo/.chef/knife.rb (変更)
Please enter the chef server URL: [https://localhost:443] https://192.168.33.10:443 (変更)
Please enter a name for the new user: [vagrant]
Please enter the existing admin name: [admin]
Please enter the location of the existing admin's private key: [/etc/chef-server/admin.pem] ~/chef-repo/.chef/admin.pem　(変更)
Please enter the validation clientname: [chef-validator]
Please enter the location of the validation key: [/etc/chef-server/chef-validator.pem] ~/chef-repo/.chef/chef-validator.pem　(変更)
Please enter the path to a chef repository (or leave blank): ~/chef-repo　(変更)
Creating initial API user...
Please enter a password for the new user:
Created user[vagrant]
Configuration file written to /home/vagrant/chef-repo/.chef/knife.rb
~~~

この時点で~/chef-repo/.chef/knife.rbを見てみると、相対パスで指定してもちゃんと絶対パスに変換してくれているようだ。

~~~ bash
log_level                :info
log_location             STDOUT
node_name                'vagrant'
client_key               '/home/vagrant/chef-repo/.chef/vagrant.pem'
validation_client_name   'chef-validator'
validation_key           '/home/vagrant/chef-repo/.chef/chef-validator.pem'
chef_server_url          'https://192.168.33.10:443'
syntax_check_cache_path  '/home/vagrant/chef-repo/.chef/syntax_check_cache'
cookbook_path [ '~/chef-repo/cookbooks' ]
~~~

## Chef Node追加
Workstationで行う。  
面倒臭いのでvagrantの秘密鍵を使う

~~~ bash
$ knife bootstrap -x vagrant 192.168.33.31 --sudo -i /vagrant/insecure_private_key -N node1
$ knife bootstrap -x vagrant 192.168.33.32 --sudo -i /vagrant/insecure_private_key -N node2
$ knife bootstrap -x vagrant 192.168.33.33 --sudo -i /vagrant/insecure_private_key -N node3
~~~

この「node1」というマシン名でWorkstationからsshが出来るようになっていないといけない。
多分推測なのだけれど、このbootstrapという処理は、knife node create & node側でchef-clientを実行しているのだと思う。

## レシピ実行
コミュニティレシピを実行してみたいので設定する。　　
最近は[java](https://github.com/socrata-cookbooks/java)レシピがお気に入りなのでそれを使ってみる。
まずは、berkshelfをインストールするが、これはCYGWINではなくLinux上なので全然問題なくストレートにインストール出来た。(autoconfがなかったのでyumでインストールしたけど)

~~~ bash
$ gem i berkshelf
Successfully installed berkshelf-3.1.3
~~~

続いてberkshelfの初期化をする。
Berkshelfのバージョンは

~~~ bash
$ berks -v
3.1.3
~~~

である。CYGWINじゃないので最新を使えた。
まずBerksfileを作るために

~~~ bash
$ berks init
The resource at '/home/vagrant/chef-repo/metadata.rb' does not appear to be a valid cookbook. Does it have a metadata.rb?
~~~

とやると怒れられるので、下記内容でBerksfileを作成する。

~~~ bash
$ vim Berksfile
source "https://api.berkshelf.com/"

cookbook "java"
~~~

この環境では/home/vagrant/chef-repoにいる状態で、

~~~ bash
$  berks install --path cookbooks
DEPRECATED: `berks install --path [PATH}` has been replaced by `berks vendor`.
DEPRECATED: Re-run your command as `berks vendor [PATH]` or see `berks help vendor`.
~~~

と怒られたので、

~~~ bash
$ berks vendor cookbooks
destination already exists /home/vagrant/chef-repo/cookbooks. Delete it and try again or use a different filepath.
~~~

とまた怒られるのでcookbooksの中はどうせ空なので消してからやったらうまく行った（なんなんだよ）

~~~ bash
$ berks vendor cookbooks
Resolving cookbook dependencies...
Using java (1.22.0)
Vendoring java (1.22.0) to /home/vagrant/chef-repo/cookbooks/java
~~~

続いて、Chef Serverにcookbookをアップロードする。
berks uploadというのも本当は使えるようなのだが、この環境だと使えなかったのでknifeコマンドでやる。
どうせレシピも一つなので・・・

~~~ bash
$ knife cookbook upload java
Uploading java           [1.22.0]
Uploaded 1 cookbook.
~~~

これでChef ServerのWebUIを見てみると、レシピが入っていると思う。

![レシピアップロード後画面]({{ site.url }}/images/06/11/uploaded-recipe.png)

続いて、このjavaレシピはコミュニティレシピだが、OpenJDKじゃなくてOracleの正規な？ものもインストールすることが出来る。  
なのでattributeを指定したいので下記の様にした。  
Chefにはenvironmentという環境の切り替えに対応するための概念があるのだが、普段は「\_default」という環境のようだ。  
ただ、この\_defaultという環境にattributesを追加することが出来ないようなので、「stating」という環境で動かすようにし、
そこにattributesを追加するようにした。

~~~ bash
$ knife environment create staging
~~~

これでエディタが立ち上がると思うので、ここにattributesを記載する。

~~~ json
{
  "name": "staging",
  "description": "",
  "cookbook_versions": {
  },
  "json_class": "Chef::Environment",
  "chef_type": "environment",
  "default_attributes": {
    	"java": {									//追加
    		"install_flavor": "oracle", 			//追加
    		"jdk_version": "7",						//追加
    		"oracle": {								//追加
      		"accept_oracle_download_terms": true	//追加
    	}											//追加
    }												//追加
  },
  "override_attributes": {
  }
}

~~~

しかしこれはstagint全体のattributesをこれにしちゃうということなので、nodeとかroleごとに設定するほうがいいのかもしれない。

これは、編集が終われば勝手にアップロードされるようだ（多分）　　
![environmentアップロード後画面]({{ site.url }}/images/06/11/uploaded-environment.png)

## レシピ実行
ここまで来たら、各ノードへレシピを適用させる。
今回はnodeごとにrecipeを書くが、roleにしたほうが良さそうなのを痛感した。

~~~ bash
$ knife node edit node1
{
  "name": "node1",
  "chef_environment": "staging", //_default→stagingへ
  "normal": {
    "tags": [

    ]
  },
  "run_list": [
    "recipe[java]"	// 追加
  ]
}
~~~

これを、node1~3までやった。

それには、各ノードでchef-clientコマンドを実行する必要があるが、Workstation側でそれをnodeの情報を使ってやってくれるコマンドがある。

~~~ bash
$ knife ssh 'name:*' 'sudo chef-client' -i /vagrant/insecure_private_key
~~~

ユーザーはすべてvagrantユーザーなので省略したが、本当は-x vagrantとして指定するのがいいと思う。
これで各ノードにログインしてみてjava -versionが出た時の嬉しい事！

