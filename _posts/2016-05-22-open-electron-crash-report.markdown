---
layout: post
title:  "electronのcrashReport(minidump)をMacOSXで見てみるまでのメモ"
date:   2016-05-22 12:00:00
category: Electron
tags:
    - Electron
    - minidump
    - Mac
    - VMWare
    - node.js
    - ubuntu
---

いつの間にか electron も **v1** が出てきて、いつの間にか **crashReporter** のパラメータに必須なものが追加されていたので、その crashReporter で送信される中身を覗いてみました。

[electron/crash-reporter.md at master · electron/electron](https://github.com/electron/electron/blob/master/docs/api/crash-reporter.md)

```javascript
const {crashReporter} = require('electron');

crashReporter.start({
  productName: 'YourName',
  companyName: 'YourCompany',
  submitURL: 'https://your-domain.com/url-to-submit',
  autoSubmit: true
});
```

## crashReporter で送信されるファイルを保存する

いくつか送信されるパラメータがあるのですが、その中でファイルがあったのでこれを保存してみます。

    The crash reporter will send the following data to the submitURL as a multipart/form-data POST:
    
    upload_file_minidump File - The crash report in the format of minidump.

試したあとに知ったのですが、こういったレポートを保存してくれる node モジュールがあったのでこちらを使ってみるのもありかもしれません。

[Electron crash report server - electron - Atom Discussion](https://discuss.atom.io/t/electron-crash-report-server/20563)

[johnmuhl/electron-crash-report-server: server for collecting crash reports from your Electron based application](https://github.com/johnmuhl/electron-crash-report-server)

ぼくはサクッと express で作ってみました。

コードはだいたいこんな感じです。

express で **multipart/form-data** なデータを扱うために、以下のモジュールを使いました。

[expressjs/multer: Node.js middleware for handling `multipart/form-data`.](https://github.com/expressjs/multer)

こちらのモジュールは、バージョンによって使い方が結構変わっているので、ちょっとハマりましたがだいたい以下のように使います。

```javascript
var express = require('express');
var multer = require('multer');
var upload = multer({ dest: 'uploads/' });

var app = express();

app.post('/crashReporter', upload.single('upload_file_minidump'), function (req, res, next) {
  console.log(req.body);
  res.send(JSON.stringify('crash'));
});
```

これで、 electron が crash すると、 uploads ディレクトリにハッシュ値なバイナリファイルができます。

## electron を crash させる

[Electron Crash Reporter](https://electron-crash-reporter.appspot.com/)

やり方は、こちらを参考にしてみました。（こんなサイトもあったのねん）

メインプロセスで実行してます。

```javascript
setTimeout(() => {
  process.crash(); 
}, 3000);
```

タイマーを挟んでそれっぽく crash させてます。

## submit された minidump ファイルはどうやって見るのか

    Crash reports are super easy to collect and test! Every native crash (out of memory, segmentation faults) electron send a POST to conﬁgured submitUrl with: platform informations memory dump in minidump at not so easy to decode :)

[Electron? It works for us, and makes desktop fun and fast](https://pracucci.com/how-we-built-spreaker-studio-for-desktop.html)

と書いてあるので、なかなか decode するのは大変そうです。

[Decoding Crash Dumps - The Chromium Projects](http://www.chromium.org/developers/decoding-crash-dumps)

ここを読むと、どうやら **minidump** ファイルを見るためのツールがあるようです。

[Linux Breakpad Tools](https://drive.google.com/folderview?id=0B5yuieQYffwPfmYwNktxYUVxb2tobHlkS2hqcjlHMUplMTdndkVaMlU0QXlaa3cwSE9xZm8&usp=sharing)

さっそくここからダウンロードします。

```bash
minidump_stackwalk foo.dmp /tmp/my_symbols 2>&1 | grep my_symbols
```

こんなコマンドが書いてあるので、おそらく **minidump_stackwalk** を使うのでしょう。（名前もそれっぽい）

## minidump_stackwalk を使ってみる

Mac でさっそく、 **minidump_stackwalk** を使ってみたのですが、以下のようにエラーが出ていっこうに中身が覗けません。

```bash
$ ./minidump_stackwalk
zsh: exec format error: ./minidump_stackwalk
```

困っていたら、以下のような issue を発見！

[1.9.7 minidump_stackwalk doesn't work on OS X · Issue #12536 · ariya/phantomjs](https://github.com/ariya/phantomjs/issues/12536)

Linux だといけそうだと分かる。

## VirtualBox に Ubuntu を入れてみる

[VirtualBoxにUbuntu 14.04.2 LTSをインストールする方法 – OTTAN.XYZ](http://ottan.xyz/virtualbox-ubuntu-2418/)

あんまり触ってこなかった VirtualBox ですが、案の定、ファイルをうまいこと D&D できなくて、ムキーッってなって。

VMware Fusion で入れることにした。

## VMware Fusion に Ubuntu を入れてみる

[VMware Fusion 8にUbuntu 14.04.3 LTSをインストールして日本語化する – OTTAN.XYZ](http://ottan.xyz/vmware-fusion-8-ubuntu-iso-3446/)

完璧！

## やっと minidump_stackwalk が動く

```bash
$ ./minidump_stackwalk minidump_file
```

やっと動いたんですが、案の定中身は読み解くのが難しい。

    2016-05-23 01:20:15: minidump.cc:4518: INFO: Minidump closing minidump
    Operating system: Mac OS X
                      10.10.5 14F1509
    CPU: amd64
         family 6 model 70 stepping 1
         8 CPUs

    Crash reason:  EXC_BAD_INSTRUCTION / 0x00000001
    Crash address: 0x10ef2bc94

    Thread 0 (crashed)
     0  Electron Framework + 0xd0c94

しいて言うなら、このあたりがエラーのように読み取れる。

まずは、 crash report を開くところまでをメモしておきました。

enjoy!
