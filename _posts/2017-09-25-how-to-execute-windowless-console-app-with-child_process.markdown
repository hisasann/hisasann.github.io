---
layout: post
title:  "ウィンドウレスなコンソールアプリをnode.jsのchild_process経由で実行し結果を受け取る方法"
date:   2017-09-25 12:00:00
categories: node.js windows c# visualstudio console
---

## やりたいこと

「node.js の child_process 経由で、自作した C# で書かれたプログラムの exe を実行し、その戻り値を node.js 側に返す。」

これはたとえば **win32 API** などを呼びたいが node.js からだとそのまま書くことは厳しいので、 C# などネイティブコードを書ける環境で exe を作成し、それを実行したかったからです。

このときに、いろいろとハマりポイントがあったのでこちらにメモしておきます。


## コンソールアプリをウィンドウレスにする方法

通常 Visual Studio でプロジェクトを作るときは、 Windows アプリケーションかコンソールアプリケーションか選ぶと思いますが、コンソールアプリケーションをそのまま選んでしまうと、 exe を実行したときにコンソールが一瞬でも立ち上がってしまいます。

Windows アプリケーション ならなおさら UI が存在してしまいます。

ですが、これらを合わせるとウィンドウレスにすることができます。

1. コンソールアプリケーションとしてプロジェクトを作成する

1. プロジェクトのプロパティから **出力の種類** を **Windowsアプリケーション** にする

これで、コンソールアプリケーションとして作られたプロジェクトですが、出力の種類が Windows アプリケーションなので UI 側に出力しそうですが、 UI が存在しないので何も起動しない。

ということになるようです。

ちょっとトリッキーですが、どうやらこの方法が調べた結果スタンダードのようです。


## C# での標準出力の挙動について

今回ここが一番のハマりポイントだったんですが、まずは以下の三種類があり、それぞれ役割が違うのでそこから書いておきます。

* System.Diagnostics.Debug.Write()

* Console.Write()

* AttachConsole

### System.Diagnostics.Debug.Write()

あくまでも Visual Studio の **出力** ペインに出力するための出力方法です。

### Console.Write()

コンソールアプリケーションを作成したときに、そのコンソールアプリケーションを実行したときに出力される方法です。

### AttachConsole

これが今回採用した出力なんですが、コンソールアプリケーションを実行するのをダブルクリックとはではなく、コマンドプロンプトから実行した場合に呼び出し元に出力するのがこの方法です。

書いててもややこしいんですが、つまり対象となる exe （コンソールアプリケーション）には出力せず、呼び出し元のコンソールに出力する方法です。

[WPF アプリでもコンソールにログしたい](http://var.blog.jp/archives/67899331.html)

[[C#] WinFormのプログラムからコンソール(標準出力)に文字を出力する](http://nanoappli.com/blog/archives/2363)

<script src="https://gist.github.com/hisasann/2dcf55566ff78891ad03ed6322157c54.js"></script>

今回検証してて気がついたのですが、 node.js の child_process 経由で実行する場合、 Console.Write() でも受け取ることができました。

ただし、コマンドプロンプトから対象の exe を実行しても受け取ることができなかったので、こちらの AttachConsole を使っておくことが今後を考えてベターだと思います。


## AttachConsole を使ったシンプルな C# プロジェクトを作成する

ここではあくまでもサンプルなので、シンプルにしますが、最終的に結果を node.js 側に返したいので、出力だけしています。

<script src="https://gist.github.com/hisasann/11565cf2be47e71f18fa8882328d3a27.js"></script>


## node.js 側のコード

```javascript
const childProcess = require('child_process');

const path = 'path/to/console-application.exe';

// この方法では動かなかった
// let p = childProcess.spawn(path, [], {
// });
// p.on('exit', function (code) {
//   console.log('child process exited with code ' + code);
// });

// この方法では動かなかった
// childProcess.exec(path, (err, stdout, stderr) => {
//   if (err) {
//     console.log(err);
//     return;
//   }
//
//   console.log(stdout);
// });

childProcess.execFile(path, [], (err, stdout, stderr) => {
  if (err) {
    console.log(err);
    return;
  }

  console.log(stdout);  // 出力！
});
```

結果的に AttachConsole の値は childProcess.execFile でしか取れませんでした。


## あとがき

これにて目的の **ウィンドウレスなコンソールアプリをnode.jsのchild_process経由で実行し結果を受け取る** ができたのですが、もっとシンプルに取れる方法がないか未だ調べております。

また、呼び出し元を考慮するとエラーがあったら err に入るようにしたいのですが、 C# 側で Exception を発行してもダメで、今はエラーがあった場合には prefix をつけて出力することでエラーと node.js 側に認識させています。

こうゆう言語を跨ぐというかそうゆうコードを書くのは楽しいですね。

ではでは！
