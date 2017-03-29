---
layout: post
title:  "WebGLのwebglcontextlostイベントについての考察"
date:   2017-03-28 12:00:00
categories: electron webgl webcontextlost canvas
---

## はじめに

今回の検証は **electron 1.4.6** で WebGL を使った場合に webcontextlost が極稀にですが発生し、それをどう回避したかのメモになります。


## webglcontextlost イベントが発生するタイミング

The webglcontextlost event of the WebGL API is fired if the user agent detects that the drawing buffer associated with a WebGLRenderingContext object has been lost.

via [webglcontextlost | MDN](https://developer.mozilla.org/ja/docs/Web/Events/webglcontextlost)

Examples: Another page does something that takes the GPU too long and the browser or the OS decides to reset the GPU to get control back. 2 or more pages use too many resources and the browser decides to tell all the pages they lost the context and then restore it only to the front page for now. The user switches graphics cards (Turns on/off one or more in the control panel) or updates their driver (no reboot required on Windows7)

via [HandlingContextLost - WebGL Public Wiki](https://www.khronos.org/webgl/wiki/HandlingContextLost)

一見すると、少なくともフォアグラウンドで且つ現在アクティブなタブな場合は **webglcontextlost** イベントが発生しないように感じるんですが、 GPU まわりが貧弱な PC で試してみたところ、2週間に一回発生するかしないかの頻度で発生しました。

ちなみに、一日一度再起動しますが、ほぼつけっぱなしな場合です。


## webglcontextlost イベントが発生するとどうなるの？

このイベントを甘く見ていたのですが、 chrome で発生したタイミングを見たことがないので、 electron の場合の話になってしまいますが、一度このイベントが発生すると **webglcontextrestored** イベントが発生するまで context を get できません。

```js
let gl = canvas.getContext('webgl')
```

これはつまり、 location.reload を実行してもダメでした。

画面を **リロードしても動いてくれない** というなかなかハマりどころがある状態で、ログを仕込んでようやくそういった挙動をしたであろうという感じです。

three.js を使っている場合、以下のようにエラーが出てしまいます。

```js
Uncaught TypeError: Cannot read property 'getExtension' of null
```

該当の three.js のコードは、 THREE.WebGLExtensions の

```js
extension = gl.getExtension( name );
```

になります。


## webglcontextlost イベントが発生したらどうしたらいいの？

このイベントが発生するのが稀すぎて、実際に検証がしっかりできていないんですが、

1. location.reload する
  * ダメだった
1. BrowserWindow を起動し直す
  * OS の画面が見えないように真っ黒の別の BrowserWindow を用意して切り替えるなど必要
  * 試せていない
1. electron を起動し直す
  * 試せていない
1. gl.getError があるかどうかで、 canvas の処理が完全に動かないようにする
  * 結果的にこちらを落とし所にした

webglcontextlost イベント、 webglcontextrestored イベントはシミュレートができるので、その状態でも動くようにアプリケーションを作ることができます。


## webglcontextlost イベントが起きることを想定したコードを書く

```js
(() => {
  let extWebGLLoseContext;
  let webGLErrorCode;
  let gl;

  document.addEventListener('DOMContentLoaded', function(event) {
    document.querySelector('#lost').addEventListener('click', function() {
      loseContext();
    }, false);
    document.querySelector('#restore').addEventListener('click', function() {
      restoreContext();
    }, false);

    let canvas = document.querySelector('#canvas');
    gl = canvas.getContext('webgl');

    webGLErrorCode = gl.NO_ERROR;
    let initialGlError = gl.getError();
    if (initialGlError !== gl.NO_ERROR) {
      webGLErrorCode = initialGlError;
    }
    // 画面ロード時に webglcontextlost のままを想定するテストをする場合用
    // this.webGLErrorCode = this.gl.CONTEXT_LOST_WEBGL;
    console.log('isWebGLError: ' + isWebGLError());

    canvas.addEventListener('webglcontextlost', (event) => {
      console.error('---webglcontextlost---');
      event.preventDefault();

      // canvas まわりの dispose 処理をここで呼ぶ

      // webglcontextlost イベント時のエラーコードを保存しておく
      // gl.getError() は一回でも呼ぶと、その後は正常値を返す可能性があるので必要なタイミングで一回だけ呼ぶこと
      webGLErrorCode = gl.getError();
    }, false);

    canvas.addEventListener('webglcontextrestored', () => {
      console.error('---webglcontextrestored---');

      // webglcontextrestored イベントでリストアされた場合に限り画面をリストアする
      windowRestore();
    }, false);

    // 画面ロード時に gl にエラーがある場合は canvas の処理を実行しない
    if (webGLErrorCode !== gl.NO_ERROR) {
      console.log('画面ロード時に WebGL の Error が発生: ' + webGLErrorCode);
      return;
    }

    initCanvas(canvas);
  });

  function initCanvas(canvas) {
    let renderer = new THREE.WebGLRenderer({
      canvas: canvas,
      alpha: true,
      antialias: true
    });

    extWebGLLoseContext = renderer.context.getExtension('WEBGL_lose_context');
  }

  function windowRestore() {
    location.reload();
  }

  function isWebGLError() {
    let isError = false;

    if (webGLErrorCode !== gl.NO_ERROR) {
      isError = true;
      console.error('gl.getError(): ' + webGLErrorCode);
    } else {
      console.log('gl.getError(): ' + webGLErrorCode);
    }

    return isError;
  }

  function loseContext() {
    extWebGLLoseContext.loseContext();
    console.log('isWebGLError: ' + isWebGLError());
  }

  function restoreContext() {
    extWebGLLoseContext.restoreContext();
  }
})();
```

[webgl-webglcontextlost-event/index.js at master · hisasann/webgl-webglcontextlost-event](https://github.com/hisasann/webgl-webglcontextlost-event/blob/master/index.js)

不完全な部分もあるかもですが、だいたいこのような感じになりました。

画面をロード後に context がない場合あるので、そのあたりも考慮してます。

また、

```js
gl.getError()
```

はエラー後に一度だけ呼ばないと、二度目以降は正常が返ってきてしまった。

かつ、 **getError** は結構重い処理のようなので、なるべく変数にプールしておくのがよいみたいです。


## サンプルコード

[hisasann/webgl-webglcontextlost-event](https://github.com/hisasann/webgl-webglcontextlost-event)


## まとめ

[Error creating WebGL context · Issue #4927 · mrdoob/three.js](https://github.com/mrdoob/three.js/issues/4927) こちらの issue だと 2014 年から話が上がっていて、現在も進行形なので、結構やっかいな問題なんだなーと認識しました。

どうやら GPU まわりがどうのこうのという可能性が高そうです。

このイベントの細かい内容は [HandlingContextLost - WebGL Public Wiki](https://www.khronos.org/webgl/wiki/HandlingContextLost) こちらを見るとよいかもしれません。

そしてこちらも [Debugging - WebGL Public Wiki](https://www.khronos.org/webgl/wiki/Debugging) 。

[javascript - Webgl Context Lost and not restored - Stack Overflow](http://stackoverflow.com/questions/28135551/webgl-context-lost-and-not-restored/28137949#28137949)

[WebGLRenderingContext.getError() - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/WebGLRenderingContext/getError)
