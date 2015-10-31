---
layout: post
title: "Cygwinでのrebaseのイケてる方法"
modified: 2014-06-06 17:42:40 +0900
tags: [cygwin,rebase,remap]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: true
---
cygwinでrubyを入れて、native extentionのコンパイルが走ると結構な確率で下記のようなメッセージが出る。

	[main] ruby 1268 child_info_fork::abort: unable to remap fctrl.so to same address as parent

これは、現在のシェル上で、gemで使用する*.soファイルをリストし、それをashというシェルでrebaseallというコマンドを打てば治るということを知った。

[Cygwin - rebaseall - PIB](http://seesaawiki.jp/w/kou1okada/d/Cygwin%20-%20rebaseall)

仕組みはあまりよくわかっていないが、実行ファイルのメモリ上のロード位置を調整するとかそんな感じなのだろうか。
ただ、findでリストをファイル化し、現在のシェルを終了させてash.exeをダブルクリックし・・・というのはちょっと面倒臭いので、
現在のシェル上で何とかならないかやってみた。
調べてみると、rebaseallはashかdashからしか使えないようだが、rebaseというコマンドはそうじゃなくても使えるらしい。

なので、これを使って~/.rbenv以下の*.soファイルをrebaseさせてみるコマンドを考えた。
ちなみに、今使っているマシンは32bitなので、64bitだとまた結果が変わってくるかもしれない。
ちなみに、バッククオートとシングルクオートはちゃんと区別してね。

~~~ bash
$ rebase -sv -T `find ~/.rbenv --name '*.so'` 
~~~

vはverbose, Tはファイルのリストで、sというオプションがポイントらしい。

~~~ bash
$ rebase --help
Usage: rebase [OPTIONS] [FILE]...
Rebase PE files, usually DLLs, to a specified address or address range.

  -4, --32                Only rebase 32 bit DLLs.  This is the default.
  -8, --64                Only rebase 64 bit DLLs.
  -b, --base=BASEADDRESS  Specifies the base address at which to start rebasing.
  -s, --database          Utilize the rebase database to find unused memory
                          slots to rebase the files on the command line to.
                          (Implies -d).
                          If -b is given, too, the database gets recreated.
  -O, --oblivious         Do not change any files already in the database
                          and do not record any changes to the database.
                          (Implies -s).
  -i, --info              Rather then rebasing, just print the current base
                          address and size of the files.  With -s, use the
                          database.  The files are ordered by base address.
                          A '*' at the end of the line is printed if a
                          collisions with an adjacent file is detected.

  One of the options -b, -s or -i is mandatory.  If no rebase database exists
  yet, -b is required together with -s.

  -d, --down              Treat the BaseAddress as upper ceiling and rebase
                          files top-down from there.  Without this option the
                          files are rebased from BaseAddress bottom-up.
                          With the -s option, this option is implicitly set.
  -n, --no-dynamicbase    Remove PE dynamicbase flag from rebased DLLs, if set.
  -o, --offset=OFFSET     Specify an additional offset between adjacent DLLs
                          when rebasing.  Default is no offset.
  -t, --touch             Use this option to make sure the file's modification
                          time is bumped if it has been successfully rebased.
                          Usually rebase does not change the file's time.
  -T, --filelist=FILE     Also rebase the files specified in FILE.  The format
                          of FILE is one DLL per line.
  -q, --quiet             Be quiet about non-critical issues.
  -v, --verbose           Print some debug output.
  -V, --version           Print version info and exit.
  -h, --help, --usage     This help.
~~~

なんだか今まで四苦八苦していたけど、rbenvのgemの*.soファイルに関してはこれで良さそう。