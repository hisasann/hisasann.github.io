---
layout: post
title:  "[再入門]SpringBootで学ぶDI(Dependency injection)とAOP(アスペクト指向)"
date:   2019-10-31 12:00:00
category: Java
tags:
    - AOP
    - SpringBoot
---

## 🦑 まえがき

ぼくは別のブログで、10年前に `AOP` に関する記事を書いてましたが、

[最もシンプルにJavaのAOPを書いてみる→そしてJavaScriptへ at HouseTect, JavaScriptな情報をあなたに](http://hisasann.com/housetect/2009/04/javaaopjavascript.html)

久々に `SpringBoot` で再入門してみました。

理由としては、だいぶ時間もたち忘れてしまったのもあり、いまなりの書き方を再度覚えたいなーと思ったからになります。

検証しながら書いていきますが、もし間違っている点などありましたら、 [hisasann@DJ lemon-Sour🍌](https://twitter.com/hisasann) で指摘いただけますと幸いです。

また、サンプルコードがちょいちょい出てきますが、すべてを push したリポジトリと gradle を使ったビルド手順を後ろのほうに載せていますので、合わせてご覧ください。

## 🥦 DI（Dependency injection）とは

そもそもですが、 AOP を単体で使うことは少なく、この `DI + AOP` で採用されることが多いかと思います。

DI というのは、

    依存性の注入（いそんせいのちゅうにゅう、英: Dependency injection）とは、
    コンポーネント間の依存関係をプログラムのソースコードから排除するために、
    外部の設定ファイルなどでオブジェクトを注入できるようにするソフトウェアパターンである。
    英語の頭文字からDIと略される。

[依存性の注入 - Wikipedia](https://ja.wikipedia.org/wiki/%E4%BE%9D%E5%AD%98%E6%80%A7%E3%81%AE%E6%B3%A8%E5%85%A5)

文章で読むと難解なイメージがありますが、早い話自分で **new** をしてインスタンスを生成するのではなく、`誰か` にインスタンスを生成してもらって、そのインスタンスを使いたいクラスのメンバに **注入** してもらうことをいいます。

このときの `誰か` が **Springframework** になります。

Springframework が **DIコンテナ** から必要なインスタンスを取り出して、依存関係を注入してくれます。

`Dependency injection` という言葉ができる前は、 `IoCコンテナ[制御の反転(Inversion of Control)]` と呼ばれていたようですが、ぼくが依存性の注入を知ったときはすでに、 Dependency injection という言葉が主流でした。

このあたりに関しはては、 [Inversion of Control コンテナと Dependency Injection パターン](https://kakutani.com/trans/fowler/injection.html) こちらの記事がとても参考になりました。

では DI を使った場合と使わない場合のサンプルコードを見ていきましょう。

### 🥯 DIを使わないサンプルコード

DI を使わないと自分で `new` する必要が出てきます。

この場合、 `AOPMain` クラスは `AOPControllerImpl` クラスに依存しています。

```java
public class AOPMain {
  private IAOPController controller;

  public static void main(String[] args) {
    controller = new AOPControllerImpl();
    controller.uploadFile();
    controller.downloadFile();
  }
}
```

### 🌭 DIを使ったサンプルコード

DI を使うと自分で `new` する必要はなく、 controller メンバに勝手に `AOPControllerImpl` を Springframework が注入してくれます。

また、 `IAOPController` はインターフェイスなので、クラスには依存せず、あくまでもインターフェイスが提供するメンバを通じてアクセスすることになります。

こうすることで、疎結合となります。

またこの場合、 `AOPMain` クラスは `AOPControllerImpl` クラスに依存していないので、実際にどんなクラスが注入されるかはわかりません。

```java
@SpringBootApplication
public class AOPMain implements CommandLineRunner {
  @Autowired
  private IAOPController controller;

  public static void main(String[] args) {
    SpringApplication.run(AOPMain.class, args);
  }

  @Override
  public void run(String... args) throws Exception {
    controller.uploadFile();
    controller.downloadFile();
  }
}
```

### 🍣 Bean定義ファイルを使った場合のサンプルXML

今回は、 `Bean 定義ファイル` を使う方法と、コードに書く方法がありますが、今回はコードに書く方法で書いています。

もし、 Bean 定義ファイルのほうで書くなら、例ではありますが、以下のようになります。

```xml
<beans>
    <bean id="MovieLister" class="spring.MovieLister">
        <property name="finder">
            <ref local="MovieFinder"/>
        </property>
    </bean>
    <bean id="MovieFinder" class="spring.ColonMovieFinder">
        <property name="filename">
            <value>movies1.txt</value>
        </property>
    </bean>
</beans>
```

### 🍙 ここまでを踏まえて「制御の反転」を考える

冒頭で、デザインパターンの `制御の反転（IoCコンテナ）` について書きましたが、サンプルコードを見たあとだと理解がしやすいかもしれません。

DI を使わなければ、 `AクラスがBクラスをインスタん化するので、制御の方向はAクラス → Bクラス`、

DI を使えば `BクラスがクラスAに依存関係を注入するため、制御の方向はAクラス ← Bクラス`

という感じになり、反転します。

[SpringのDIとAOPについてまとめる。 - Qiita](https://qiita.com/hirooka0527/items/44f7e07114d7225e59ec)

### 🍥 AngularのDependency injection

実は `Angular` はできた当初から DI を採用していて、以下のページに詳細が書かれています。

[Angular - Angular の依存性の注入](https://angular.jp/guide/dependency-injection)

また、以前調べていたときに、仮引数を固定していたのが面白くてそれについて以下に書いてありますので、 angular の引数で依存オブジェクトを渡す仕組みがわかるかもしれません。（古い情報なので、いまは変わっているかもです）

[AngularJS's tutorial - あなたとともにAngularJS](http://lab.hisasann.com/AngularJSTutorial/)

の `$scopeなど、AngularJSがどうやって仮引数を固定しているか` が該当の部分です。

## 🍝 AOP（アスペクト指向）とは

DI だけでもまぁまぁなボリュームでしたが、ここからがいよいよ本命の `AOP` になります。

そもそも、アスペクト指向とは、

    アスペクト指向プログラミング（アスペクトしこうプログラミング、Aspect Oriented Programming、AOP）は、
    オブジェクト指向ではうまく分離できない特徴（クラス間を横断 (cross-cutting) するような機能）を「アスペクト」とみなし、
    アスペクト記述言語をもちいて分離して記述することでプログラムに柔軟性をもたせようとする試み。
    アスペクトの例としては、データ転送帯域の制限や例外の処理などがある。
    Java にアスペクト指向的要素を追加したAspectJ が実験的に実装されている。

    オブジェクト指向プログラミングとは直交するプログラミングパラダイムである。

    既存のプログラミングパラダイムを置き換えるものではなく、あくまで既存の言語の補助機能として利用されることを目的としている。

via [アスペクト指向プログラミング - Wikipedia](https://ja.wikipedia.org/wiki/%E3%82%A2%E3%82%B9%E3%83%9A%E3%82%AF%E3%83%88%E6%8C%87%E5%90%91%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0)

と、なっております。

この `オブジェクト指向ででうまくいかなかった部分を埋めるためにアスペクト指向が生まれた` という感じがぼくはすごく好きです。

### 🍜 横断的関心事（クラス間を横断 (cross-cutting) するような機能）

AOP を語る上で重要なのがこの **横断的関心事** です。

    開発者がひとつのまとまりとして考えるもののことで， 通常はソフトウェアの機能や品質など，
    開発者がソフトウェアに作り込む要素を指します． 
    「ひとつのまとまりとして考える」単位は，開発者の判断に依存します．

    たとえば「画面の描画」や「データベースへのアクセス」，「応答性能」など，開発者が名前を付けて取り扱おうとしているものは，
    それぞれ関心事であるといえます．

    関心事は，それぞれが明確に独立しているわけではなく，他の関心事と重複する部分があったり，
    ある関心事が他の関心事に依存していたりすることもあります．

[横断的関心事 - アスペクト指向なWiki](http://netail.net/aosdwiki/index.php?%B2%A3%C3%C7%C5%AA%B4%D8%BF%B4%BB%F6)

こうゆう言葉を聞くと、ウッとなりますが、意味としては、

`いろんなところに書かれた同じプログラム` を意味します。

ログ処理だったり、トランザクション処理だったりと様々ですが、たとえば、 100 個のメソッドがあった場合に、それぞれのメソッドの開始直後にトランザクション開始、メソッドの終わりにトランザクション終了を必ず書かないといけないとすると、もしも一個でもミスをして抜けてしまったら、データベースには commit されません。

このようの同じ処理を何回も書きたくないよね、というのもありますが、絶対にやらないといけない横断的処理があった場合にも効力を発揮します。

### 🧘🏻‍♂️ SpringframeworkとSpringBootの違い

*Springframework* は `DI` と `AOP` で一世風靡しましたが、その後現れた *SpringBoot* とはどう違うのか、

実は SpringBoot を触るのは今回が初めてだったので、ググった情報で補いますが、

    Spring FrameworkはXML形式の設定ファイルを定義することにより設定を外部化することができるが、
    Spring Frameworkではこの設定ファイルの設定の煩雑さがしばしば指摘されている。
    
    Spring Bootではこの設定ファイルの複雑さをできる限り排除しており、
    必要最低限の設定を行うだけでアプリケーションの起動・実行を行うことができる。

via [【Java】SpringBootとは？](https://eng-entrance.com/java-springboot#SpringBoot)

だそうです。

たしかに、 Springframework 時代は XML に結構書かなければいけなかったので、 SpringBoot はアノテーションベースになっているので、書きやすいです。

では DI を使った場合と使わない場合のサンプルコードを見ていきましょう。

### 🍦 AOPを使わないサンプルコード

このサンプルでの `横断的関心事` は、①と②になります。

ほぼほぼ同じようなコードですが、これが、仮に 100 以上のメソッドに書かないといけないとすると、煩雑ですしミスも起きそうです。

```java
public class AOPController implements IAOPController {
  public void uploadFile(jp) {
    // ①
    System.out.println("AOPControllerクラスのメソッドuploadFileを開始します");
    
    // 重要な処理と仮定
    System.out.println("uploadFile");
    
    // ②
    System.out.println("AOPControllerクラスのメソッドuploadFileを終了します");
  }

  public void downloadFile() {
    // ①
    System.out.println("AOPControllerクラスのメソッドdownloadFileを開始します");
    
    // 重要な処理と仮定
    System.out.println("downloadFile");
    
    // ②
    System.out.println("AOPControllerクラスのメソッドdownloadFileを終了します");
  }
}
```

### 🌮 AOPを使ったサンプルコード

AOP を使うと煩雑な処理が実装コードに含まれません。

そして、クラスが一つ増えて、 AOP の機能だけをもったクラスが増えています。

```java
@Component("aop.AOPController")
@Controller
public class AOPController implements IAOPController {
  public void uploadFile() {
    // 重要な処理と仮定
    System.out.println("uploadFile");
  }

  public void downloadFile() {
    // 重要な処理と仮定
    System.out.println("downloadFile");
  }
}
```

もともとあった煩雑な処理はこちらに移動しています。

このようにアノテーションをつけて、どこどこの `Component` のなんてメソッドが実行されたら何をしてほしいかを全く別ファイルに外だしすることができます。

```java
@Component
@Aspect
public class AOPComponent {
  @Before("execution(* aop.*Controller.*(..))")
  public void before(JoinPoint jp) { // メソッド名は何でもよい
    // ①
    System.out.println(jp.getSignature().getDeclaringType().getSimpleName() + "クラスの" + jp.getSignature().getName() + "メソッドを開始します");
  }

  @After("execution(* aop.*Controller.*(..))")
  public void after(JoinPoint jp) { // メソッド名は何でもよい
    // ②
    System.out.println(jp.getSignature().getDeclaringType().getSimpleName() + "クラスの" + jp.getSignature().getName() + "メソッドを終了します");
  }
}
```

これが、アスペクト指向です。

実行結果は以下のとおりです。

```bash
 ~/_/js-code/gs-gradle/initial   master ● ?  ./gradlew run                                                                                                                                                            ✔  1685  18:00:08
Starting a Gradle Daemon (subsequent builds will be faster)

> Task :run

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.1.6.RELEASE)

2019-10-30 18:00:15.127  INFO 60888 --- [           main] aop.AOPMain                              : Starting AOPMain on LAB-N1584.local with PID 60888 (/Users/hisamatsu/_/js-code/gs-gradle/initial/build/classes/java/main started by hisamatsu in /Users/hisamatsu/_/js-code/gs-gradle/initial)
2019-10-30 18:00:15.130  INFO 60888 --- [           main] aop.AOPMain                              : No active profile set, falling back to default profiles: default
2019-10-30 18:00:15.888  INFO 60888 --- [           main] aop.AOPMain                              : Started AOPMain in 1.129 seconds (JVM running for 1.438)
AOPController2クラスのuploadFileメソッドを開始します
uploadFile2
AOPController2クラスのuploadFileメソッドを終了します
AOPController2クラスのdownloadFileメソッドを開始します
downloadFile2
AOPController2クラスのdownloadFileメソッドを終了します
```

こうみると、たしかにオブジェクト指向ではカバーしきれなかった部分をうまくアスペクト指向が補っている感がありますよね。

## 🍩 SpringBootをgradleで動かすまでの道のり

以下を読みながらやっていきました。

[Getting Started · Building Java Projects with Gradle](https://spring.io/guides/gs/gradle/#initial)

ガイドリポジトリをクローン。

```bash
$ git clone https://github.com/spring-guides/gs-gradle.git
$ gs-gradle/initial
```

gradle を Homebrew でインストール。

```bash
$ brew install gradle
$ gradle build
$ ./gradlew build
```

gradle で Java コードを実行する。

```bash
$ ./gradlew run
```

また、今回作成した `build.gradle` は以下のようになりました。

一点ハマったのが、 `dependencies` のところに以下記述が必要でした。

[Maven Repository: org.springframework.boot » spring-boot-starter-aop » 2.1.6.RELEASE](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-aop/2.1.6.RELEASE)

```gradle
apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'application'

mainClassName = 'aop.AOPMain'

// tag::repositories[]
repositories {
    mavenCentral()
}
// end::repositories[]

// tag::jar[]
jar {
    baseName = 'gs-gradle'
    version =  '0.1.0'
}
// end::jar[]

// tag::dependencies[]
sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile "joda-time:joda-time:2.2"
    testCompile "junit:junit:4.12"
    // https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-aop
    compile group: 'org.springframework.boot', name: 'spring-boot-starter-aop', version: '2.1.6.RELEASE'
}
// end::dependencies[]
```

今回のサンプルコードは、 [springboot-aop-gs-gradle/initial/src/main/java/aop at master · hisasann/springboot-aop-gs-gradle](https://github.com/hisasann/springboot-aop-gs-gradle/tree/master/initial/src/main/java/aop) にまとめてあります。

## 🥙 JavaScriptでAOPを実現する

かなり前ですが、 JavaScript で AOP を実現するコードを書いたので、合わせて貼っておきます。

<script src="https://gist.github.com/hisasann/4523162.js"></script>

## 🍜 あとがき

細かい AOP 用語はほぼ取り上げませんでしたが、 `DI + AOP` の面白さは伝えられたのかなーと思っています。

また、オブジェクト指向は、何度勉強しても新しい発見があり、さらにそれプラス、アスペクト指向も再考すると関心しか生まれません。

賢い人の頭の中はどうなっているのかと。

ではでは！

## 🥪 参考記事

[テックノート – @Qualifierを使って任意のIDをつける方法（インタフェースと実クラスが1対nの場合の対処方法）](http://javatechnology.net/spring/qualifier/)

[CodeFlow](https://www.codeflow.site/ja/article/spring-primary)

[springの再入門 - DI（依存性注入） - Qiita](https://qiita.com/park-jh/items/4df5c67d895b2ea219d6)

[SpringのDIとAOPについてまとめる。 - Qiita](https://qiita.com/hirooka0527/items/44f7e07114d7225e59ec)

[Spring Boot入門① ～DI～ - Qiita](https://qiita.com/gksdyd88/items/7886f54ee8a22d311400)

[Spring FrameworkでDIする3つの方法 - Reasonable Code](https://reasonable-code.com/spring-injection-method/)

[[spring boot7]AOPを使ってログ出力してみよう - プログラミング　MEMO](https://programming-information.com/2018/12/25/post-218/)

[spring AOPについて - Qiita](https://qiita.com/hosochin/items/85bd761d6d3f6bb0c078)

[ざっくりとSpringで使うAOPの解釈をする - Qiita](https://qiita.com/ughirose/items/a7c66782f93cd1ae0d68)

[GradleでSpring Bootのプロジェクトを環境別にビルドする - Qiita](https://qiita.com/maroKanatani/items/9185fdf0ec83687c773a)

[Spring BootでAOP（アスペクト指向）を使うとコードが奇麗になる（@Aspect,@Before,@Afterなど） | 株式会社CONFRAGE ITソリューション事業部](https://confrage.jp/spring-boot%E3%81%A7aop%EF%BC%88%E3%82%A2%E3%82%B9%E3%83%9A%E3%82%AF%E3%83%88%E6%8C%87%E5%90%91%EF%BC%89%E3%82%92%E4%BD%BF%E3%81%86%E3%81%A8%E3%82%B3%E3%83%BC%E3%83%89%E3%81%8C%E5%A5%87%E9%BA%97/)
