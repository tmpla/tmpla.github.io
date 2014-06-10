---
layout: post
title: ""
modified: 2014-06-06 18:48:15 +0900
tags: [berkshelf,chef,cygwin]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
published: false
status: draft
---
CYGWINでberkshelfをインストール出来ない！最新版が！

~~~ bash
[k-yokoshima]% gem i berkshelf
DL is deprecated, please use Fiddle
Building native extensions.  This could take a while...
ERROR:  Error installing berkshelf:
        ERROR: Failed to build gem native extension.

    /cygdrive/c/Users/k-yokoshima/.rbenv/versions/2.1.2/bin/ruby.exe extconf.rb
-> sh /cygdrive/c/Users/k-yokoshima/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/dep-selector-libgecode-1.0.1/ext/libgecode3/vendor/gecode-3.7.3/configure --prefix=/cygdrive/c/Users/k-yokoshima/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/dep-selector-libgecode-1.0.1/lib/dep-selector-libgecode/vendored-gecode --disable-doc-dot --disable-doc-search --disable-doc-tagfile --disable-doc-chm --disable-doc-docset --disable-qt --disable-examples --disable-flatzinc
checking for the host operating system... Windows
checking whether the C++ compiler works... no
configure: error: in `/cygdrive/c/Users/k-yokoshima/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/dep-selector-libgecode-1.0.1/ext/libgecode3/vendor/gecode-3.7.3':
configure: error: C++ compiler cannot create executables
See `config.log' for more details
extconf.rb:94:in `block in run': Failed to build gecode library. (GecodeBuild::BuildError)
        from extconf.rb:93:in `chdir'
        from extconf.rb:93:in `run'
        from extconf.rb:100:in `<main>'

extconf failed, exit code 1

Gem files will remain installed in /cygdrive/c/Users/k-yokoshima/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/gems/dep-selector-libgecode-1.0.1 for inspection.
Results logged to /cygdrive/c/Users/k-yokoshima/.rbenv/versions/2.1.2/lib/ruby/gems/2.1.0/extensions/x86-cygwin/2.1.0/dep-selector-libgecode-1.0.1/gem_make.out

~~~

https://github.com/opscode/dep-selector-libgecode/pull/23