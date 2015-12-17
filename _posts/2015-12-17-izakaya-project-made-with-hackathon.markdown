---
layout: post
title:  "ハッカソンでWebSocketを使ったガチャサイトを作ってみた"
date:   2015-12-17 15:45:16
categories: node.js es6 react gulp pixi.js create.js hackathon
---

# 居酒屋プロジェクト

<p align="center">
  <img src="https://raw.githubusercontent.com/izakaya-project/izakaya-project-web/master/image.png">
</p>

[居酒屋プロジェクト](https://github.com/izakaya-project)

## はじめに

ちょっとしたハッカソンで、WebSocketを使った連動サイトを作ってみました。

デザイナー ＋ エンジニア（ぼく）の二人構成です。

## 目的

呑みたい！、居酒屋行きたい！の欲求をガチャを通して、可視化する。

## ゲーム概要

スマホでガチャを引くと、居酒屋のカウンターに、ガチャで引いたアイテムが置かれます。

PCサイトの舞台は居酒屋さんです。

そこには、SPサイトにて引いた料理（ガチャ）が並びます。

## ガチャ画面

某白なんとかというアプリのガチャ画面をそれっぽく作ってみたかったので、参考にしました。

## Web

[izakaya-project/izakaya-project-web](https://github.com/izakaya-project/izakaya-project-web)

### おかみさん側（みんなのガチャビューアー）

https://izakaya-project.herokuapp.com/

居酒屋のカウンターに、みんなが引いたガチャのアイテムが降ってきます。

### おじさん側（ガチャ画面）

https://izakaya-project.herokuapp.com/ojisan

ガチャは **星2〜星4** まであります。

もちろんレモンサワーは **星4** です。

## Electronアプリ

単にWebViewを使っただけですが、おかみさん側をElectronでアプリ化してみました。

[izakaya-project/izakaya-project-electron](https://github.com/izakaya-project/izakaya-project-electron)

## iOSアプリ

iPhone6sを買ったので、3Dタッチの部分をswiftでどう書くのか試したかったので作りました。

iOSアプリのほうはおじさん側です。

<script src="https://gist.github.com/hisasann/9043c96b0a91bd67ff1e.js"></script>

[izakaya-project/izakaya-project-ios](https://github.com/izakaya-project/izakaya-project-ios)

### 使った技術一覧

* node.js
* React
* Express
* gulp
* ES6
* WebSocket
* Pixi.js
* Create.js

## おかみさん側バックサウンド

あらかじめ決められた恋人たちへ - ディスタンス [あらかじめ決められた恋人たちへの「ブレ」を iTunes で](https://itunes.apple.com/jp/album/bure/id101805991)

[居酒屋プロジェクト](https://github.com/izakaya-project)

( ・∀・)ｲｲ!!

