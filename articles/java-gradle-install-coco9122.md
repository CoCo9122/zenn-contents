---
title: "[Minecraft Mod #1] Java,Gradleのインストールまで"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["java", "gradle"]
published: true
publication_name: "dawnzlight"
---

# Minecraft Mod作成の開発環境
MinecraftのMod開発環境にはJavaの開発環境が必要になります。そこで今回はMod開発に入る前のJavaとそのビルドツールであるGradleのインストールおよび、環境構築まで解説します。

# Java,Gradle をインストールする上で
## 動作環境
|||
|-|-|
| OS | Windows10 home |
| CPU | Intel core i7-10700 |

:::message
本記事はWindowsを対象としています。
:::

## 対象
以下のバージョンをインストールしていきます。
|item|version|
|-|-|
| Java | 22.0.2 |
| Gradle | 8.7 |

# 1.Javaのインストール

## Java実行ファイルのダウンロード
OracleのJava downloadsサイトにアクセスします。

https://www.oracle.com/jp/java/technologies/downloads/

アクセスしますと、図1.1のようなサイトが表示されると思います。
ここの`JDK 22`のタブを選択、さらに`Windows`のタブを選択します。

テーブルの`profile/file description`列のレコードがx64 Installerに当たる`Donwload`列をクリックします。
そうしましたら、ダウンロードが始まりますので、完了するまで待ちます。

![](/images/java-gradle-install-coco9122/img-0001.png)
*図1.1 Javaのダウンロード画面*

## Java実行ファイルの実行

ダウンロードしたexeファイルを実行しましょう。そうすると以下のような図1.2が表示されます。
そのまま「次 > 」をクリックします。
![](/images/java-gradle-install-coco9122/img-0002.png)
*図1.2 Javaの実行画面①*

次の画面は図1.3が表示されます。これはインストール先の宛先を決めることができます。今回は変更はないのでそのままにします。それと後程、このディレクトリパスを使用するため、メモ等に控えておいてください。

~~~
C:\Program Files\Java\jdk-22\
~~~

問題なければ「次」をクリックします。

![](/images/java-gradle-install-coco9122/img-0003.png)
*図1.3 Javaの実行画面②*

次の画面は図1.4になります。これはダウンロード中の画面になりますので、そのままバーがたまるまでお待ちください。

![](/images/java-gradle-install-coco9122/img-0004.png)
*図1.4 Javaの実行画面③*

図1.5の画面が表示されていれば、完了です。「閉じる」を押下して、ウィンドウを閉じてください・

![](/images/java-gradle-install-coco9122/img-0005.png)
*図1.5 Javaの実行画面③*

念のため以下のディレクトリを確認してみましょう。
~~~
C:\Program Files\Java
~~~

図1.6のようにjdk-22があれば成功です。

![](/images/java-gradle-install-coco9122/img-0006.png)
*図1.6 インストール後のディレクトリ*

## 環境変数の設定
次はインストールしたJavaのPathを通すために環境変数の設定を行います。

まずは図1.7のようにWindowsアイコンの検索窓に、「システム環境変数」と検索して、表示されるシステム環境変数の編集をクリックします。
![](/images/java-gradle-install-coco9122/img-0007.png)
*図1.7 環境変数の設定画面ソフトウェアの検索*

先ほどのソフトウェアをクリックすると図1.8の画面が表示されます。
ここの「環境変数」をクリックします。

![](/images/java-gradle-install-coco9122/img-0008.png)
*図1.8 環境変数の設定画面*

クリック後、図1.9の画面が表示されます。ここの「Path」を選択し「編集」を押下します。

![](/images/java-gradle-install-coco9122/img-0009.png)
*図1.9 環境変数の設定画面*

図1.10の画面に遷移しましたら、「新規」をクリックし先ほど、控えたJavaのディレクトリパスにbinを加えたものを設定します。完了しましたら「OK」をクリックします。これで環境変数の登録は完了です。
~~~
C:\Program Files\Java\jdk-22\bin
~~~

![](/images/java-gradle-install-coco9122/img-0010.png)
*図1.10 環境変数の設定画面*

## Javaの確認

最後に実際にインストールできたかを以下のコマンドで確認します。
```
java -version
```

以下の表示されましたら、インストール完了です。
```
java version "22.0.2" 2024-07-16
Java(TM) SE Runtime Environment (build 22.0.2+9-70)
Java HotSpot(TM) 64-Bit Server VM (build 22.0.2+9-70, mixed mode, sharing)
```

# 2.Gradleのインストール

次にGradleのインストールをします。

## GradleのZipファイルのダウンロード

Gradle downloadsサイトにアクセスします。

https://gradle.org/releases/

アクセスしますと、図2.1のようなサイトが表示されると思います。
ここの`v8.7`の`complete`のをクリックします。そうすると、zipファイルがダウンロードされます。

![](/images/java-gradle-install-coco9122/img-0011.png)
*図2.1 Gradleのダウンロードサイト画面*

## GradleのZipファイルの解凍

ダウンロードしたzipファイルを解凍します。今回はjavaと同じ`C:\Program files\`に解凍します。次にGradleのパスを通すために同様にディレクトリをメモ等で控えておきましょう。

~~~
C:\Program files\gradle-8.7
~~~

![](/images/java-gradle-install-coco9122/img-0012.png)
*図2.2 Gradleのzipファイル*

![](/images/java-gradle-install-coco9122/img-0013.png)
*図2.3 Gradleの解凍後ディレクトリ*

## 環境変数の設定

最後にJavaと同様に環境変数にPathの登録を行います。Javaと同様になりますので、「Path」先ほど設定した`C:\Program files\gradle-8.7`にbinを追加したものを保存します。設定したものが図2.4になります。

~~~
C:\Program files\gradle-8.7\bin
~~~

![](/images/java-gradle-install-coco9122/img-0014.png)
*図2.4 Gradleの設定後のPath*

## Gradleの確認

最後に実際にインストールできたかを以下のコマンドで確認します。
```
gradle -version
```

以下の表示されましたら、インストール完了です。
```
------------------------------------------------------------
Gradle 8.7
------------------------------------------------------------

Build time:   2024-03-22 15:52:46 UTC
Revision:     650af14d7653aa949fce5e886e685efc9cf97c10

Kotlin:       1.9.22
Groovy:       3.0.17
Ant:          Apache Ant(TM) version 1.10.13 compiled on January 4 2023
JVM:          22.0.2 (Oracle Corporation 22.0.2+9-70)
OS:           Windows 10 10.0 amd64
```

# まとめ

今回はMod開発の前提となるJavaとGradleのインストール方法をまとめました。
次回はForge環境の構築をしていきたいと思います。

---
何かあればコメントいただければと思います。