---
layout: post
title:  "Jekyllを使ってブログを書いていく"
date:   2015-03-20 15:45:16
category: blog
tags:
    - jekyll
    - github
---
なんとなく思い立って **github** でブログを書きはじめて見る。

いままでWordpress、MT、その他ブログサービスを使ってみましたが、今はgithubにjekyllなブログをpushすれば勝手にビルドされる時代。

これで十分な気がしております。

あと、markdownはたまにどう書くのか忘れてしまうときがあるので、それに慣れるというのも大事だなーと思ったのも **Jekyll** を使い始めた理由のひとつ。

簡単にJekyllを使った場合のブログの書き方をメモメモ。

## プログラムを貼る場合

### 簡単なsyntax-hightlightの場合

{% highlight js %}
function hoge(bar) {
  console.log(bar);
}
hoge('foo');
{% endhighlight %}

### gistを使った場合

こっちのほうがかっちょいいですね。

<script src="https://gist.github.com/hisasann/0f70abe01786ac1abd9f.js"></script>

### gistに素早くコードをアップする

普段Vimで開発やメモをしていので、[mattn/gist-vim](https://github.com/mattn/gist-vim) こちらを使わせてもらっている。

Vimを新規タブで開いて、gistにアップしたいのを貼り付けてあとは `:Gist` するだけ。

<script src="https://gist.github.com/hisasann/62dbdf756972ea86dffb.js"></script>

## Jekyllの使い方

### buildする

    bundle exec jekyll build

### serverを立ち上げてwatchする

    bundle exec jekyll serve --watch
    
## あると便利なブックマークレット

ぼくはブログを書くときに、見ているサイトのタイトルとURLをアンカータグ化するブックマークレットを使っているんですが、それのMarkdown版。

<script src="https://gist.github.com/hisasann/540105b37e474f803630.js"></script>
    
## TravisCIでビルドチェックしている

[travis-ci.org/hisasann/hisasann.github.io](https://travis-ci.org/hisasann/hisasann.github.io)

ビルドに成功したかどうかはimageとして取れるので、見やすいですね。

[![Build Status](https://travis-ci.org/hisasann/hisasann.github.io.svg)](https://travis-ci.org/hisasann/hisasann.github.io)

## 参考にした記事

こちらのリポジトリはたいへん参考にさせていただいた。

[Jekyllベースのブログに移行しました。Web Scratch](http://efcl.info/2014/07/06/new-blog/)

enjoy!

