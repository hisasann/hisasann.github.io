---
layout: post
title:  "Mac上でelectronをビルドしCentOSでFullscreen表示するまで"
date:   2017-05-21 12:00:00
categories: electron centos nodejs virtualbox
---

## まえがき

electron を CentOS 上で Fullscreen 表示する、つまりサイネージ用として使い物になるかを確認するための作業をここにメモしておきます。

結果として、 Fullscreen は可能でしたが、画面のチラツキがあり、最終的に CentOS 上で electron の Fullscreen はやめることにしました。


## VirtualBox をインストールする

[Oracle VM VirtualBox Downloads Oracle Technology Network Oracle](http://www.oracle.com/technetwork/server-storage/virtualbox/downloads/index.html?ssSourceSiteId=otnjp)

こちらから VirtualBox をインストールします。

インストール作業は基本的に次へ次へで OK です。


## VirtualBox 上に仮想マシンを作成する

新規ボタンをクリックし、ウィザードを起動します。

### 仮想マシンの作成

* 名前: CentOS 7.0
* タイプ: Linux
* バージョン: Red Hat (64-bit)

仮想メモリは 2GB ぐらいにしました。

仮想ハードディスクを作成する を選択し、作成をクリックします。

ハードディスクのファイルタイプは、 VDI を選択します。

物理ハードディスクにあるストレージは、可変サイズを選択します。

ファイルの場所とサイズは、 8GB にしました。


## CentOS のダウンロードとインストール

[CentOS Project](https://www.centos.org/)

より、 CentOS をダウンロードします。

ダウンロードするタイプは、 DVD ISO です。

VirtualBox の画面の右のペインから、ストレージの IDE セカンダリマスターの部分をクリックします。

ここでダウンロードした CentOS のファイルを選択します。


## CentOS を設定・起動する

VirtualBox の Start ボタンより CentOS を起動します。

Install CentOS 7 をクリックします。

インストールの概要で、ソフトウェアの選択をクリックします。

**GNOME Desktop** と **GNOME アプリケーション** にチェックを入れます。

あとは、ネットワークとホスト名、のところで Ethernet をオンにしておきましょう。

ここまでやって、インストール開始、をクリックし CentOS をインストールします。

インストール中、ユーザー作成ができるのでやってきましょう。

インストール後にログインすると、 GUI がある状態の CentOS が起動すると思います。

via: [仮想化ソフトウェアVirtualBoxにCentOS7.0をインストールする方法を紹介 | 株式会社エンライズコーポレーション](http://www.enrise-corp.co.jp/2185)


## Mac 上で CentOS 用に electron をビルドする

Mac 上で Linux ビルドするには、以下のパッケージが必要になるので brew 経由でインストールしておきましょう。

[Multi Platform Build · electron-userland/electron-builder Wiki](https://github.com/electron-userland/electron-builder/wiki/Multi-Platform-Build)

To build app for Linux on macOS:

```bash
$ brew install gnu-tar graphicsmagick xz
```

また、ぼくは electorn のビルドには electron-builder を使っているので、 target を AppImage にしてビルドします。

[Options · electron-userland/electron-builder Wiki](https://github.com/electron-userland/electron-builder/wiki/Options#LinuxBuildOptions-target)

ぼくは以下のようにしています。

```json
"build": {
  "productName": "",
  "appId": "",
  "files": [
    "dist/",
    "node_modules/",
    "assets/",
    "app.html",
    "main.js",
    "main.js.map",
    "package.json"
  ],
  "dmg": {
    "contents": [
      {
        "x": 130,
        "y": 220
      },
      {
        "x": 410,
        "y": 220,
        "type": "link",
        "path": "/Applications"
      }
    ]
  },
  "win": {
    "target": [
      "nsis"
    ]
  },
  "linux": {
    "target": [
      "AppImage"
    ]
  },
  "directories": {
    "buildResources": "resources",
    "output": "release"
  }
},
```

ちなみに AppImage は [AppImage | Linux apps that run anywhere](http://appimage.org/) にあるように、 Linux 向けのアプリケーションを簡単に作れるアーキテクチャのようです。


## Mac と CentOS とでファイルを渡せるように共有フォルダの設定をする

### Guest Additions をインストールする

VirtualBox の仮想マシンのメニューから Devices → Install Guest Additions CD Images を選択します。

次に、挿入したイメージをマウントします。

```bash
$ mkdir /mnt/cdrom

$ mount -r /dev/cdrom /mnt/cdrom
```

マウントしたら、 VBoxLinuxAdditions.run を実行します。

```bash
$ cd /mnt/cdrom/

$ ./VBoxLinuxAdditions.run
```

エラーが出てしまったので、必要なものをインストールしておきます。

```bash
$ yum install gcc

$ yum install perl

$ yum install kernel-devel-$(uname -r) -y
```

再度インストールを実行します。

```bash
$ ./VBoxLinuxAdditions.run
```

ここまででフォルダ共有の下準備は完了です。

### Mac と CentOS のフォルダを共有する

VirtualBox の 仮想マシンの設定 → 共有フォルダ から Mac のフォルダを追加します。

このとき、 自動マウント と 永続化する にチェックを入れます。

CentOS 側でシェアしたいフォルダを作成しマウントします。

```bash
$ mkdir /mnt/share

$ mount -t vboxsf share /mnt/share
```

ここまでやって、やっと Mac 側と CentOS 側でファイルを渡すことができるようになりました。

via:

[VirtualBoxの共有フォルダ設定 – MacとCentOSでファイル共有](http://kepachu.com/post-55)

[macにVirtualBoxをインストールしてCentOSを起動してホストとssh接続、最後に共有ディレクトリの設定まで](http://djangoapplab.com/34/)


## CentOS（Linux）上で electron を fullscreen 表示するには

fullscreen は Mac だととても簡単にできるのですが、 Windows ・ Linux だとちょっとややこしかったです。

```javascript
const bw = new BrowserWindow({
  left: 0,
  top: 0,
  transparent: false,
  show: false,
  frame: false,
  resizable: true,
  kiosk: true,
  'always-on-top': true
});

// 画面の最大化
bw.maximize();
// 画面をリサイズできないようにする
bw.setResizable(false);
```

最終的にいきついたのが、↑の BrowserWindows のセットアップです。

resizable を true の状態にしてしまうと、 maximize メソッドを呼んでも最大化してくれません。

そして、 maximize メソッドを呼んだあとに、 setResizable(false) を呼んでリサイズできないようにしています。

結果的に、 maximize で事足りてしまったので fullscreen プロパティは使っていません。


## あとがき

はじめ、 electron 側でうまいこといかず、 CentOS 側の設定でなんとかヘッダーとフッターの UI を消せないか検証していました。

そのあたりの議論は、

[Auto hide top panel in Gnome 3 - FedoraForum.org](http://forums.fedoraforum.org/showthread.php?t=264623)

[Removing bottom bar in Gnome 3.8.4 - CentOS](https://www.centos.org/forums/viewtopic.php?t=50815)

[How to remove the bottom panel in gnome 3 classic? - linux - SysTutorials QA](https://www.systutorials.com/qa/2326/how-to-remove-the-bottom-panel-in-gnome-3-classic)

ここらでされていて、ちょっと古いんですが、色々試してみたんですが、うまいこといかなかったです。

[Hide Top Bar - GNOME Shell Extensions](https://extensions.gnome.org/extension/545/hide-top-bar/)

[Hide top panel - GNOME Shell Extensions](https://extensions.gnome.org/extension/740/hide-top-panel/)

ここいらのプラグインを使ってみたりしてみたんですが、うまいこと消えてくれませんでした。

[1.3. GNOME クラシックとは](https://access.redhat.com/documentation/ja-JP/Red_Hat_Enterprise_Linux/7/html/Desktop_Migration_and_Administration_Guide/what-is-gnome-classic.html)

[Smiling Life : CentOS7におけるGNOME3のカスタマイズ](http://blog.livedoor.jp/rootan2007/archives/52090677.html)

GNOME3 のカスタマイズというのもしてみたんですが、消えないですね。

ということで、現状は electron 側でうまいこと最大化してあげるのが一番楽でした。

ではでは。
