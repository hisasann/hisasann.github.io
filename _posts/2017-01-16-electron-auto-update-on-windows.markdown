---
layout: post
title:  "electronのautoUpdaterをWindows上で試したときのメモ"
date:   2017-01-15 12:00:00
categories: node.js electron windows
---

## はじめに

今回試したかったのが、 **Windows** 環境でアプリをどこかに登録などせずに簡単に **auto upadte** ができるかどうかです。

これは、世の中に公開せずにクローズドな環境で auto update ができるかという検証になります。

**Mac** 環境だとおそらく code sign が必要そうですが、 Windows だと証明書などなくてもアプリケーションの auto update ができました。

[Electron - Signing a Mac Application](https://pracucci.com/atom-electron-signing-mac-app.html)

ただ、これもいろんな記事によってはできるできないと分かれていましたので、そういった記事のリンクも貼りつつぼくが試したのをメモしていきます。


## electron に付属している autoUpdater を使う場合

[Electronアプリをプロダクトとして「正しく」リリースするために必要な3つのこと - ヌーラボ [Nulab Inc.]](https://nulab-inc.com/ja/blog/typetalk/3-points-for-releasing-production-electron-app/)

[Auto-updating apps for Windows and OSX using Electron: The complete guide – Medium](https://medium.com/@svilen/auto-updating-apps-for-windows-and-osx-using-electron-the-complete-guide-4aa7a50b904c#.9x4g8zpvg)

上記のリンク先のように、自前でサーバーを用意して、以下のようなスクリプトでサーバーサイドで exe ファイルを管理するやりかたです。

<script src="https://gist.github.com/svileng/0d61efa7a071b0ea1dec.js"></script>

ただ、こちらのパターンだと手順が結構複雑で、とくに [electron-archive/grunt-electron-installer: Grunt plugin to build Windows installers for Electron apps](https://github.com/electron-archive/grunt-electron-installer) この grunt のモジュールを使ってビルドしたり、或いは、 [electron/windows-installer: Build Windows Installers for Electron apps](https://github.com/electron/windows-installer) を使うなど、個人的に複雑でした。

かつ、 code sign が必要な感じがあったので、別の方法に切り替えてみました。


## electron-builder を使ってみる

[electron-userland/electron-builder: A complete solution to package and build a ready for distribution Electron app with “auto update” support out of the box](https://github.com/electron-userland/electron-builder)

**electron-builder** を使った場合は、こちらの記事が参考になりました。

[Publishing with electron-builder](http://electron.rocks/electron-builder-explained/)


### ディレクトリ構成

```
[electron-auto-update]
├── [app]
    ├── ...
    ├── [dist]          <- generated distributions
    ├── main.js
    ├── package.json    <- application package.json
├── ...
├── [build]             <- build resources
├── package.json        <- development package.json
```

このように、アプリケーションと開発環境を別に作成します。


### development package.json

```json
"build": {
    "productName": "ElectronAutoUpdate",
    "appId": "com.hisasann.ElectronAutoUpdate",
    "category": "public.app-category.tools",
    "files": [
        "dist/",
        "node_modules/",
        "index.html",
        "main.js",
        "renderer.js",
        "package.json"
    ],
    "win": {
        "target": "nsis",
        "publish": [
        {
            "provider": "generic",
            "url": "https://s3-ap-northeast-1.amazonaws.com/electron-auto-update/latest"
        }
        ]
    }
},
```

開発環境用の package.json にこのように Windows でのビルド方法を指定します。

今回は、 AWS を使った場合の provider: generic を指定しています。

これは、 [Publishing Artifacts · electron-userland/electron-builder Wiki](https://github.com/electron-userland/electron-builder/wiki/Publishing-Artifacts) の wiki を見ながらやってみました。

また、 AWS を使わずに自前のサーバーでも auto update は可能のようなのですが、 **https** は必須のようです。


そして、ビルド用の script は、

```json
"scripts": {
    "start": "electron ./app",
    "dev": "npm run start",
    "release": "build --ia32"
},
```

このように、 build を実行するだけになります。

今回は、 32ビット版のビルドをしたので   --ia32 と引数を渡しています。

これで releases ディレクトリに exe ファイルが書き出されます。


### Webpack などで .js ファイルを書き出す場合

今回はトランスパイル系は使っていないので、 dist ディレクトリの中は空っぽですが、 Babel などを使った場合はここに書き出されるようにします。


## electron のメインプロセスで autoUpdater を使う

よくあるコードの例として、以下のようにバージョンをセットして、 **setFeedURL** のがありますが、 electron-builder は自動的に setFeedURL を呼んでくれるので、自前で呼んではいけません。

これは [Auto Update · electron-userland/electron-builder Wiki](https://github.com/electron-userland/electron-builder/wiki/Auto-Update) にさらっと書いてあります。

それと、 **electron.autoUpdater** を使うのではなく、 electron-auto-updater モジュールに内包されている autoUpdater を使うます。

```javascript
const autoUpdater = electron.autoUpdater;
const appVersion = require('./package.json').version;
const os = require('os').platform();

const updateFeed = os === 'darwin' ?
 'https://localhost:3000/updates/latest?os=darwin&v=' :
 'https://localhost:3000/updates/latest?os=win32&v=';

console.log(updateFeed + appVersion);
autoUpdater.setFeedURL(updateFeed + appVersion);

autoUpdater.on('update-downloaded', function (event, releaseNotes, releaseName, releaseDate, updateUrl, quitAndUpdate) {
 console.log('update-downloaded');
 autoUpdater.quitAndInstall();
});
```

なので、上記のコードではなく以下のようにします。

また、イベントに入ったかどうかわかりにくいので、 [megahertz/electron-log: Just a very simple logging module for your Electron application](https://github.com/megahertz/electron-log) を使っています。

このモジュールはとても使いやすく、ぼくは日付でローテートする版を作って、使っています。

それらを加味すると以下のような感じになります。

```javascript
const autoUpdater = require('electron-auto-updater').autoUpdater;

const log = require("electron-log");
autoUpdater.logger = log;
autoUpdater.logger.transports.file.level = "info";

autoUpdater.addListener("update-available", function (event) {
 log.info("update-available");
});
autoUpdater.addListener("update-downloaded", function (event, releaseNotes, releaseName, releaseDate, updateURL) {
 log.info("update-downloaded" + releaseName);
 autoUpdater.quitAndInstall();
 return true
});
autoUpdater.addListener("error", function (error) {
 log.info(error);
});
autoUpdater.addListener("checking-for-update", function (event) {
 log.info("checking-for-update");
});
autoUpdater.addListener("update-not-available", function () {
 log.info("update-not-available");
});

autoUpdater.checkForUpdates();
```

ログファイルには以下のように出力されました。

    [2017-01-16 15:57:30:0746] [info] checking-for-update
    [2017-01-16 15:57:31:0093] [info] update-available
    [2017-01-16 15:57:32:0452] [info] update-downloaded1.0.2

ちゃんと動いてそうです！


## auto update が動くために必要な latest.yml

今回は AWS を使っていますが、自前サーバーでも同じで、 **latest.yml** が必須になります。

このファイルは中身を見ると、

    version: 1.0.2
    githubArtifactName: electron-auto-update-Setup-1.0.2.exe
    path: ElectronAutoUpdate Setup 1.0.2.exe
    sha2: xxxxxxxxxxxxxxxxxxxxxx

最新がどのバージョンなのかを持っています。

これで、 electron が毎回起動する度にここに問い合わせをしてバージョンが上がっていれば、 auto update が動くという仕組みのようです。

ちなみに Mac のほうで、同じようにビルドしても latest-mac.xml は自動的に作成されませんでした。

もしかしたら、 code sign をせずにビルドしているからかもしれないです。


## 今回のサンプルリポジトリ

[electron-various-tutorials/electron-auto-update at master · hisasann/electron-various-tutorials](https://github.com/hisasann/electron-various-tutorials/tree/master/electron-auto-update)

[electron-various-tutorials/electron-auto-update-server at master · hisasann/electron-various-tutorials](https://github.com/hisasann/electron-various-tutorials/tree/master/electron-auto-update-server)


## 参考リンク

### electron-builder を使っているリポジトリ

[salomvary/soundcleod: SoundCloud for macOS and Windows](https://github.com/salomvary/soundcleod)

[develar/onshape-desktop-shell: Onshape desktop app (web application shell). Unofficial.](https://github.com/develar/onshape-desktop-shell)

[Publishing with electron-builder](http://electron.rocks/electron-builder-explained/)

[Auto Update · electron-userland/electron-builder Wiki](https://github.com/electron-userland/electron-builder/wiki/Auto-Update)

[Auto-updating apps for Windows and OSX using Electron: The complete guide – Medium](https://medium.com/@svilen/auto-updating-apps-for-windows-and-osx-using-electron-the-complete-guide-4aa7a50b904c#.9x4g8zpvg)

[electron/auto-updater.md at master · electron/electron](https://github.com/electron/electron/blob/master/docs-translations/jp/api/auto-updater.md)

[Oneteam アプリのビルド + 配信自動化 #meguroes - Atsushi Nagase](https://ja.ngs.io/2016/02/11/how-oneteam-deliver/)

[Electron - Signing a Mac Application](https://pracucci.com/atom-electron-signing-mac-app.html)

[Get that damn Windows Auto Update working on Electron](https://blog.avocode.com/blog/get-that-damn-windows-auto-update-working-on-electron)

