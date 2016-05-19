---
layout: post
title:  "BitBarにtrelloのcardを表示してタスク管理する"
date:   2016-05-18 12:00:00
categories: node.js bitbar trello
---

タスク管理は **trello** を使っていて、 iOS アプリもあるので、すごく使いやすいんですが、 Today のタスクも稀にやり忘れるときがあるので、 **BitBar** に表示することにしました。

そのやり方をメモメモ。

## API key を取得する

[Developer API Keys](https://trello.com/app-key)

## API token を取得する

[https://trello.com/1/authorize?key=＜APIkey＞&name=&expiration=never&response_type=token&scope=read,write](https://trello.com/1/authorize?key=＜API key＞&name=&expiration=never&response_type=token&scope=read,write)

## Board の id を取得する

[https://trello.com/1/members/＜name＞/boards?key=＜API key＞&token=＜Token key＞&fields=name](https://trello.com/1/members/＜name＞/boards?key=＜API key＞&token=＜Token key＞&fields=name)

## List の id を取得する

[https://trello.com/1/boards/＜borad id＞/lists?key=＜API key＞&token=＜Token key＞&fields=name](https://trello.com/1/boards/＜borad id＞/lists?key=＜API key＞&token=＜Token key＞&fields=name)

## Card の一覧を取得する

[https://api.trello.com/1/lists/＜listid＞?fields=name&cards=open&card_fields=name&key=＜API key＞&token=＜Token key＞](https://api.trello.com/1/lists/＜listid＞?fields=name&cards=open&card_fields=name&key=＜API key＞&token=＜Token key＞)

## BitBar 用の node.js コード

上記で取得した情報をもとに、 api 用の URL を組み立ててリクエストするだけ。

ただ、それだけです。
楽ちん！

<script src="https://gist.github.com/hisasann/975221cfdca3737077bcd582c4b7907e.js"></script>

enjoy!

