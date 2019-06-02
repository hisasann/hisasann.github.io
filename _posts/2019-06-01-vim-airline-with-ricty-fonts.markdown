---
layout: post
title:  "Vimでairlineを使うときに毎回Rictyのビルドしてるのでやり方メモ"
date:   2019-06-01 12:00:00
category: Vim
tags:
    - airline
    - powerline
    - ricty
---

## まえがき

何度も同じ作業をやっているんですが、フォント系は意外とややこしいので、メモしておきます。

[vim-airline/vim-airline: lean & mean status/tabline for vim that's light as air](https://github.com/vim-airline/vim-airline)


## Ricty を Homebrew からインストールする

```bash
$ brew tap sanemat/font
$ brew install ricty --with-powerline
```

インストールが終了すると、以下のようなコマンドがコンソールに表示されるのでフォントファイルをコピーします。

```bash
$ cp -f /usr/local/opt/ricty/share/fonts/Ricty*.ttf ~/Library/Fonts/
$ fc-cache -vf
```

via [Homebrewを使ってRictyをインストール - Qiita](https://qiita.com/mktktmr/items/5481eac60b96c80cc262)


## Vim に airline をインストールする

### .vimrc

<script src="https://gist.github.com/hisasann/bec700174a8c8e148442aadc0bb5d0f9.js"></script>

via [Powerlineは難しくないよ！ - Qiita](https://qiita.com/park-jh/items/557a9d5b470947aef2f5)

### .gvimrc

<script src="https://gist.github.com/hisasann/a673eefea8b9b1ffff60ce67fdc5c300.js"></script>


## あとがき

余談ですが、久々に入れなおしたんですが、 MacVim の右側の矢印が少し崩れてしまいました。

一旦は OK なので、このまま使います。

また、 iTerm のほうは、 [powerline/fonts: Patched fonts for Powerline users.](https://github.com/powerline/fonts) を使っています。
