---
layout: post
title:  "fishシェルに乗り換えてみた🔫"
date:   2020-02-09 01:00:00
category: Shell
tags:
    - Shell
---

## 🦑 まえがき

社内で、シェルは何を使っているのか、
framework 的なものはどういったものを使っているのかとか話して「 `fish` いいよ〜」という話になったので、使い始めてみました。

ぼくは、 zsh をずっと使っていて、

[ohmyzsh/ohmyzsh: 🙃 A delightful community-driven (with nearly 1,500 contributors) framework for managing your zsh configuration. Includes 200+ optional plugins (rails, git, OSX, hub, capistrano, brew, ant, php, python, etc), over 140 themes to spice up your morning, and an auto-update tool so that makes it easy to keep up with the latest updates from the community.](https://github.com/ohmyzsh/ohmyzsh)

や

[sorin-ionescu/prezto: The configuration framework for Zsh](https://github.com/sorin-ionescu/prezto)

を導入していました。

なので、 zsh -> fish という感じです。

## 🐶 fishをインストールする

```bash
$ brew install fish
$ fish -v
```

## 🍔 シェルを変更する

最後の行に追加します。

```bash
$ sudo vi /etc/shells
$ /usr/local/bin/fish
```

ログインシェルを fish に変更します。

```bash
$ chsh -s /usr/local/bin/fish
```

## 🔫 fisherをインストールする

パッケージマネージャーをインストールします。

[jorgebucaran/fisher: A package manager for the fish shell.](https://github.com/jorgebucaran/fisher)

```bash
$ curl https://git.io/fisher --create-dirs -sLo ~/.config/fish/functions/fisher.fish
```

## 🪓 テーマを変更する

```bash
$ fisher add oh-my-fish/theme-bobthefish
```

## 🏋🏻‍♂️ Powerlineフォントをインストールする

```bash
$ git clone https://github.com/powerline/fonts.git
$ cd fonts
$ ./install.sh
$ cd ../
$ rm -rf fonts
```

iTerm2 の Profiles -> Text で

**Source Code Pro for Powerline** を選び、 **Bold** にしました。

ぼくの環境では、 Bold をつけないと、 Powerline の矢印がズレてしまったので、これで対応しています。

## 🧛‍♂️ Draculaのテーマをインストールする

```bash
$ git clone https://github.com/dracula/iterm.git
$ cd iterm
$ open ./Dracula.itermcolors
```

## 🤼‍♀️ jethrokuan/zをインストールする

[jethrokuan/z: Pure-fish z directory jumping](https://github.com/jethrokuan/z)

z は過去に訪れたディレクトリの履歴検索ツールです。

```bash
$ fisher add jethrokuan/z
```

使い方は、

"z" + スペースを押して、行きたいディレクトリ名を入力し、タブで保管されます。

## 👮‍♀️ fzfをインストールする

z と少し被りますが、 fzf はコマンドを含め履歴検索できるツールです。

```bash
$ brew install fzf
$ fisher add jethrokuan/fzf

$ touch ~/.config/fish/config.fish
$ vim ~/.config/fish/config.fish

$ set -U FZF_LEGACY_KEYBINDINGS 0
$ set -U FZF_REVERSE_ISEARCH_OPTS "--reverse --height=100%"
```

    “FZF_LEGACY_KEYBINDINGS”はfishの過去バージョンとコンフリクトが発生していたため、
    新しいキーバインドを使用するのに必要だからです。

    “FZF_REVERSE_ISEARCH_OPTS”は履歴のフィルタリング時に入力欄をターミナル上部に表示するための”–reverse”オプションと、
    フルスクリーンで表示するための”–height”オプションを設定しています。

[【Mac】fish + fisher + peco + fzfで快適なターミナル環境を構築 Public Constructor](https://public-constructor.com/terminal-fish-setup/)

## 🥦 展開されるalias - abbrのすごさよ

zsh だと zshrc にこんな感じでエイリアスを定義して使うんですが、

```bash
alias gst='git status -sb'
```

fish の abbr は

```bash
abbr -a gst git git status -sb
```

これで、 `gst` で `git status -sb` が展開されます。

展開されるところがわかりやすくていいですねー

[[fish] 展開されるalias - abbr - Qiita](https://qiita.com/wataash/items/ab0a8b86b60e782f537f)

細かい使い方は以下を。

[abbr:fish流別名！入力後に展開される短縮コマンドを定義](http://fish.rubikitch.com/abbr/)

## 🍜 雑感

たまに自分のシェル環境を見直すと、いろいろ発見もあり、よりシンプルになるとうれしくなります。

毎日触るものなので、良くしていきたいですね！

それでは良いシェルライフを🎅🎄！

## 🧙 参考記事

[fishshellのインストール - Qiita](https://qiita.com/hirotatsuuu/items/74ab481e90066cce524f)

[fish shell のインストールと初期設定 - Qiita](https://qiita.com/1ain2/items/a63ea17320640cc37070)

[fish shell を使いたい人生だった ｜ Developers.IO](https://dev.classmethod.jp/etc/fish-shell-life/)

[macOSをCatalinaにしたらfish shellの補完が遅くなった話 - ウキウキ？ワクワク？ Yeah!! Happy!!](https://panaki.hatenablog.com/entry/2019/12/12/195249)
