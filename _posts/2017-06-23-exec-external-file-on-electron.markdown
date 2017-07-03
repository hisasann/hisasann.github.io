---
layout: post
title:  "Windows上でelectronに内包したexeを実行する方法"
date:   2017-06-23 12:00:00
categories: electron nodejs exe
---

## まえがき

タイトルに書ききれていないのですがそもそもの発端が、 windows10 上で electron から別の electron のアップデートや、 electron とは関係ないアプリケーションのアップデーターを作ってみるというのを試していまして、そこでややこしい箇所が多々あったので、それをこちらにメモしておきます。


## electron から electron-builder でパッケージングした zip ファイルを解凍すると解凍されないファイルがあった

electronAppA から electronAppB をビルドした win-unpack ディレクトリを zip 化したファイルの解凍をレンダープロセスやメインプロセスで実行した場合にうまくいきませんでした。

正確には以下の2個のファイルを筆頭にいくつかのファイルが解凍されませんでした。

* resources/app.asar

* resources/app-update.yml

これ以外にも解凍できないファイルはあったのですが、このあとに書きます win-unpack ディレクトリ内を削除するときにもこの2個のファイルが残ってしまったので、こちらの2個をメインに書いておきます。

以下は解凍するのに試した npm モジュールたちです。

また、 electron 内部からではなくシンプルな node.js ファイルから実行すると問題なく削除されます。

zip の解凍モジュールはけっこうたくさんありどれもよかったのですが、 node-stream-zip は細かいイベントにフックでき、今後別で使うときはこちらを使ってみようと思っています。

### adm-zip

[cthackers/adm-zip: A Javascript implementation of zip for nodejs. Allows user to create or extract zip files both in memory or to/from disk](https://github.com/cthackers/adm-zip)

```javascript
var zip = new AdmZip(zipFile);
zip.extractAllTo(/*target path*/C.tempPath, /*overwrite*/true);
```

### extract-zip

[maxogden/extract-zip: Zip extraction written in pure JavaScript. Extracts a zip into a directory.](https://github.com/maxogden/extract-zip)

```javascript
extract(zipFile, {dir: C.tempPath}, function (err) {
  resolve();
});
```

### node-stream-zip

[antelle/node-stream-zip: node.js library for fast reading of large ZIPs](https://github.com/antelle/node-stream-zip)

```javascript
var zip = new StreamZip({
  file: zipFile,
  storeEntries: true
});
zip.on('error', function(err) {
  console.error(err);
});
zip.on('ready', function() {
  console.log('Entries read: ' + zip.entriesCount);
  // extract folder
  zip.extract('win-unpacked/locales', './temp/', function(err, count) {
      console.log('Extracted ' + count + ' entries');
  });
  // extract all
  zip.extract(null, './temp/', function(err, count) {
      console.log('Extracted ' + count + ' entries');
  });
});
zip.on('extract', function(entry, file) {
  console.log('Extracted ' + entry.name + ' to ' + file);
});
zip.on('entry', function(entry) {
  console.log('Read entry ', entry.name);
});
```

### node-unzip

[EvanOxfeld/node-unzip: node.js cross-platform unzip using streams](https://github.com/EvanOxfeld/node-unzip)

```javascript
fs.createReadStream(zipFile).pipe(unzip.Extract({path: C.tempPath}));
```


## node.js で解凍するのは諦めて 7zip というソフトをキックする方法に切り替えてみた

[インストール不要のZIP、7z、RARファイルの解凍方法 | 7-Zip](https://sevenzip.osdn.jp/howto/non-install-extract.html)

コマンドラインから使える Windows 用解凍ソフトとして **7zip** というのがあり、こちらを使ってみたのですが、 electron を dev モードで試したときは問題がなかったのですが、 electron-builder で exe 化したときに、うまくいかなかったです。

child_process.exec や execSync で試してたんですが、 exe の中に内包している exe をうまく実行できなく悩んでいました。

[Executing a script as a child_process.exec inside an asar archive · Issue #3512 · electron/electron](https://github.com/electron/electron/issues/3512)

こちらの issue を見ていたら、 2015 年くらいからこの話がされていてつい最近もコメントがついているぐらい議論されていました。

[Electronのアプリ内部に配置されたファイルを実行する - Qiita](http://qiita.com/nyamogera/items/0aaff641c07c9b5aec51)

悶々としてたらこちらの記事を発見し execFile を使ってみたらすんなり動いてくれました。

```javascript
childProcess.execFile('7zipのpath', ['x', '-y', '-o' + tempPath, zipFile], (err, stdout, stderr) => {
  if (err) {
    console.log(err);
  }
  console.log(stdout);
});
```

強引ではありますが、こちらで一旦落ち着きました。


## electron から electron-builder でパッケージングした win-unpack ディレクトリを削除したら削除されないファイルがあった

↑までに書いたのは、 **解凍** ですが、以下のファイルを含むディレクトリも electron 上から **削除** がなかなかうまくいきませんでした。

どうゆうパターンを試したのかをメモしておきます。

* resources/app.asar

* resources/app-update.yml

### rimraf でレンダープロセスで削除を試してみる

```javascript
var rimraf = require('rimraf');
rimraf('C:\\marksman-output\\electron1', {}, function (err) {
  if (err) {
    console.log(err);
  }
  console.log('done');
});
```

エラーが出ずうまくいかなかった。

### rimraf をメインプロセスで削除を試してみる

レンダープロセス

```javascript
const ipcRenderer = window.ipcRenderer;
ipcRenderer.on('delete-directory2', (event, message) => {
  console.log('done');
});
ipcRenderer.send('delete-directory1');
```

メインプロセス

```javascript
ipcMain.on('delete-directory1', function (event) {
  var rimraf = require('rimraf');
  rimraf('C:\\marksman-output\\electron1', {}, function (err) {
    if (err) {
      console.log(err);
    }
    console.log('rimraf done');
    event.sender.send('delete-directory2');
  });
});
```

以下のエラーが出てうまくいかなかった。

    { Error: ENOTEMPTY: directory not empty, rmdir 'C:\marksman-output\electron1\win-unpacked\resources'
      errno: -4051,
      code: 'ENOTEMPTY',
      syscall: 'rmdir',
      path: 'C:\\marksman-output\\electron1\\win-unpacked\\resources' }
    rimraf done

### 外部ファイルに切り出してレンダープロセスで exec で呼び出し削除を試してみる

レンダープロセス

```javascript
var child_process = require('child_process');
child_process.exec('node ' + C.rootPath + '\\delete-directory.js', (err, stdout, stderr) => {
  if (err) {
    console.log(err);
  }
  console.log(stdout);
});
```

delete-directory.js

```javascript
const log = require("electron-log");
log.info('child');
console.log('child');

var rimraf = require('rimraf');

rimraf('C:\\marksman-output\\electron1', {}, function (err) {
  if (err) {
    log.info(err);
    console.log(err);
  }

  log.info('rimraf done');
  console.log('rimraf done');
});
```

以下のエラーが出てうまくいかなかった。

    [17:03:26:0546] [info] Error: EBUSY: resource busy or locked, unlink 'C:\marksman-output\electron1\win-unpacked\resources\app.asar'
        at Error (native)

    { Error: EBUSY: resource busy or locked, unlink 'C:\marksman-output\electron1\win-unpacked\resources\app.asar'
        at Error (native)
      errno: -4082,
      code: 'EBUSY',
      syscall: 'unlink',
      path: 'C:\\marksman-output\\electron1\\win-unpacked\\resources\\app.asar' }
    [17:03:26:0552] [info] rimraf done
    rimraf done

### 外部ファイルに切り出してレンダープロセスで fork で呼び出し削除を試してみる

レンダープロセス

```javascript
var child_process = require('child_process');
let child = child_process.fork(C.rootPath + '/delete-directory');
child.send({ message: "from parent" });
```

delete-directory.js

```javascript
const log = require("electron-log");
log.info('child');
console.log('child');

var rimraf = require('rimraf');

process.on("message", function (msg) {
  rimraf('C:\\marksman-output\\electron1', {}, function (err) {
    if (err) {
      log.info(err);
      console.log(err);
    }

    log.info('rimraf done');
    console.log('rimraf done');
  });
});

process.on("exit", function () {
    console.log("child exit");
});
```

エラーが出ないが削除がされなかった。

### 外部ファイルに切り出してメインプロセスで fork で呼び出し削除を試してみる

レンダープロセス

```javascript
const ipcRenderer = window.ipcRenderer;
ipcRenderer.on('delete-directory2', (event, message) => {
  console.log('done');
});
ipcRenderer.send('delete-directory1');
```

メインプロセス

```javascript
ipcMain.on('delete-directory1', function (event, returnEvent, imagePath) {
  var child_process = require('child_process');

  var child_process = require('child_process');
  let child = child_process.fork(`${__dirname}/delete-directory`);
  child.send({ message: "from parent" });
});
```

削除処理は上と同じ

以下のエラーが出てうまくいかなかった。

    [17:22:29:0945] [info] child
    child
    [17:22:42:0862] [info] Error: ENOTEMPTY: directory not empty, rmdir 'C:\marksman-output\electron1\win-unpacked\resources'

    { Error: ENOTEMPTY: directory not empty, rmdir 'C:\marksman-output\electron1\win-unpacked\resources'
      errno: -4051,
      code: 'ENOTEMPTY',
      syscall: 'rmdir',
      path: 'C:\\marksman-output\\electron1\\win-unpacked\\resources' }
    [17:22:42:0867] [info] rimraf done
    rimraf done

### 外部ファイルに切り出してメインプロセスで spawn で呼び出し削除を試してみる

```javascript
let p = child_process.spawn('node', [`${__dirname}\\delete-directory.js`]);
p.on('exit', function (code) {
  console.log('child process exited with code ' + code);
});
```

問題なく削除できた

### 外部ファイルに切り出してメインプロセスで execFile で呼び出し削除を試してみる

```javascript
child_process.execFile('node', [`${__dirname}\\delete-directory.js`], (err, stdout, stderr) => {
  if (err) {
    console.log(err);
  }
  console.log(stdout);
});
```

問題なく削除できた


## レンダープロセスで windows のコマンドを実行して削除した

↑で検証はしたが electron-builder で exe に内包した .js ファイルをレンダープロセス・メインプロセスで spawn, exec, execFile を使ってみたのですが実行できませんでした。

なので、苦渋ですが、 windows のコマンドを使って対応しました。

```javascript
var child_process = require('child_process');
child_process.exec('rd /s /q C:\\marksman-output\\electron1', (err, stdout, stderr) => {
  if (err) {
    console.log(err);
  }

  console.log(stdout);
});
```

dev モードでできるが exe 化するとできないことがこれで2点目なので、なるべく小さなプログラムに書いて実際に exe 化して試すのが大切だなーと思いました。


## 外部プログラムを child_process 経由で実行するときに親が閉じても子が閉じないように起動する方法

これは、 electronAppA が electronAppB をアップデートし electronAppB の exe を起動するときのお話です。

はじめ、 exec や execFile で試していたのですが親が閉じると子が閉じてしまい electronAppA だけを閉じることができませんでした。

この件は、 Mac だと

```javascript
exec('open /Applications/hoge.app');
```

は親が閉じても子は閉じないようです。

ではどうやって親と子を干渉させないようにするかは、

[Forked child process not detaching on windows · Issue #5146 · nodejs/node](https://github.com/nodejs/node/issues/5146)

こちらの issue に解決策が書かれていました。

**{ detached: true }** です。

```javascript
let p = childProcess.spawn(outputPath + '\\win-unpacked\\hoge.exe', [], {
  detached: true,
  stdio: ['ignore']
});
p.unref();
```

さらに **unref** メソッドで親プロセスが子を待機するのも防いでいます。


## child_process のメソッドたち

少し **exec** **execFile** **spawn** **fork** についておさらいをメモしておきます。

細かい詳細は以下の document ページをご覧ください。

[Child Process Node.js v0.8.26 Manual & Documentation](http://nodejs.jp/nodejs.org_ja/docs/v0.8/api/child_process.html)

また、 child_process のソースは以下から見ることができます。

[node/child_process.js at master · nodejs/node](https://github.com/nodejs/node/blob/master/lib/child_process.js)

### exec

exec は第一引数にコマンドとその引数を文字列として渡せる **非同期** メソッドです。

callback 関数には、 err・標準出力文字列・標準エラー出力文字列 が渡されてきます。

同期的に呼び出したい場合は、 **execSync** を使います。ただし、 標準出力文字列 の内容が Buffer で返ってきます。

```javascript
child_process.exec('ls -la /hoge', function(error, stdout, stderr){
  console.log(stdout);
});
```

exec は execFile のラッパーのようです。

```javascript
exports.exec = function(command /*, options, callback*/) {
  var opts = normalizeExecArgs.apply(null, arguments);
  return exports.execFile(opts.file,
                          opts.options,
                          opts.callback);
};
```

via [node/child_process.js at master · nodejs/node](https://github.com/nodejs/node/blob/master/lib/child_process.js#L134)

exec を使うときは、主にシェルコマンドを実行するときに使っています。

これは、

> execとexecFileの最大の違いは、execは/bin/shを経由してプロセスを起動することです

via [node.jsのchild_processのexecとexecFileとspawnの違い - SundayHacking](http://www.axlight.com/mt/sundayhacking/2014/03/nodejschild-processexecexecfilespawn.html)

> 子シェルで実行する代わりに指定されたファイルを直接実行することを除いて child_process.exec() と同様です。 これは child_process.exec より若干効率的で、同じオプションを持ちます。

via [Child Process Node.js v0.8.26 Manual & Documentation](http://nodejs.jp/nodejs.org_ja/docs/v0.8/api/child_process.html#child_process_child_process_execfile_file_args_options_callback)

のとおりです。

### execFile

基本的には exec と同じですが、コマンドとその引数を別々に書かなければなりません。

今回 exe を起動するときに execFile を使っていますが、こうゆう exe を起動などは execFile を使い、ただコマンドを実行したいだけの場合は exec を使っています。

callback 関数には、 err・標準出力文字列・標準エラー出力文字列 が渡されてきます。

```javascript
child_process.execFile('ls', ['-la', '/hoge'], function(error, stdout, stderr){
  console.log(stdout);
});
```

### fork

fork は JavaScript ファイルを子プロセスとして起動して使いたい場合に使います。

fork のメリットは親と子のプロセスが emitter 経由でメッセージのやりとりができる点です。

parent.js

```javascript
var child = child_process.fork("./child");

child.on("message", function (msg) {
  console.log(msg);

  setTimeout(function () {
    child.send({ message: "from parent" });
  }, 1000);
});
```

child.js

```javascript
child.send({ message: "from parent" });

process.on("message", function (msg) {
    console.log(msg);

    setTimeout(function() {
        process.send({ message: "from child!" });
    }, 1000);
});

process.on("exit", function () {
    console.log("child exit");
});
```

### spawn

spawn は exec と fork の機能を兼ね備えたいい子ちゃんです。

```javascript
child = child_process.spawn('ls', ['-la', '/hoge']);

child.stdout.on('data', function (data) {
  console.log('stdout: ' + data);
});

child.stderr.on('data', function (data) {
  console.log('stderr: ' + data);
});

child.on('exit', function (code) {
  console.log('child process exited with code ' + code);
});
```

とくに今回助かったのが、

```javascript
var fs = require('fs'),
  spawn = require('child_process').spawn,
  out = fs.openSync('./out.log', 'a'),
  err = fs.openSync('./out.log', 'a');

var child = spawn('prg', [], {
  detached: true,
  stdio: [ 'ignore', out, err ]
});

child.unref();
```

**detached: true** で親と子のプロセス関係を断ちけれる点です。


## あとがき

普段あまり child_process を使わないのでたいへん勉強になりました。

また、 electron から electron のディレクトリを解凍や削除をする場合は少し注意が必要という点が分かりました。

しかし、 node.js は楽しいですね！

ではでは。
