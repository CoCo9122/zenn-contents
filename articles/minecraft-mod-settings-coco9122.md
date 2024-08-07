---
title: "[Minecraft Mod #2] Minecraft Mod開発の初期構築"
emoji: "🍣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["java", "minecraft"]
published: false
---

# はじめに

MinecraftのMod開発環境を作成するために、Forgeの公式サイトにある開発ファイルを使用したいと思います。

Minecraft Mod開発環境にはJavaとgradleのインストールが必要になりますので、環境がそろっていない場合は、以下の記事を参考にインストールをお願いします。

https://zenn.dev/dawnzlight/articles/java-gradle-install-coco9122

# Forgeのファイルのダウンロード

Forgeの公式サイトから開発環境の準備を行います。

## Forgeから開発用ファイルのダウンロード

今回は最新バージョン(2024年8月現在)である1.21用のforgeの開発ファイルを使用したいと思います。
まずは以下のサイトへアクセスしましょう。Forgeの公式サイトになります。

https://files.minecraftforge.net/net/minecraftforge/forge/index_1.21.html

図1.1のような画面になります。次に赤枠で囲われたMDKファイルをクリックします。

![](/images/minecraft-mod-settings-coco9122/img-0001.png)
*図1.1 Forgeのダウンロード画面*

そうすると図1.2のような画面に移行しますが、何もクリックせず、右上にある赤線の秒数のカウントが終わるまで待ちます。

![](/images/minecraft-mod-settings-coco9122/img-0002.png)
*図1.2 Forgeのダウンロード画面*

図1.3のように「SKIP」のボタンに代わりましたら、赤枠の「SKIP」クリックします。

![](/images/minecraft-mod-settings-coco9122/img-0003.png)
*図1.3 Forgeのダウンロード画面*


そうしますとzipファイルがダウンロードされると思います。図1.4のzipファイルがダウンロードされたファイルになります。

![](/images/minecraft-mod-settings-coco9122/img-0004.png)
*図1.4 zipファイル*

## Minecraft Mod開発環境の準備

次はMod開発準備です。先ほどダウンロードしたzipファイルを任意の場所に解凍しましょう。
解凍しますと、以下のディレクトリ構成になっていると思います。これで開発環境の準備は完了です。
```
.
└── forge-1.21-51.0.29-mdk/
    ├── gradle/warpper/
    ├── src/main
    ├── .gitattributes
    ├── .gitignore
    ├── CREDITS.txt
    ├── LICENSE.txt
    ├── README.md
    ├── README.txt
    ├── build.gradle
    ├── changelog.txt
    ├── gradle.properties
    ├── gradlew
    ├── gradlew.bat
    └── settings.gradle
```


# Minecraftの起動

次にマインクラフトの起動してみましょう。

## Minecraftの起動

まずは下記コマンド実行してみましょう。実行しますと、ビルドが開始されます。
```sh
gradlew build
```

次に下記コマンドを実行してみましょう。

```sh
gradlew runClient
```

たくさんのログが流れて図1.5のような画面がたちが上がり、最終的に図1.6のようにMinecraftが起動すると思います。

![](/images/minecraft-mod-settings-coco9122/img-0005.png)
*図1.5 Minecraft起動中の画面*

![](/images/minecraft-mod-settings-coco9122/img-0006.png)
*図1.6 Minecraft画面*

これでMinecraftの起動は完了です。

## VSCodeのタスクに追加

ついでにVSCodeにタスクを追加しましょう。`forge-1.21-51.0.29-mdk`配下に`.vscode/tasks.json`を作成します。

作成した、tasks.jsonファイルに以下のコードを書きこみます。(詳細な内容は省きます。)
```json:.vscode/tasks.json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "マインクラフトの起動",
            "type": "shell",
            "command": "gradlew build && gradlew runClient",
            "args": [],
            "problemMatcher": [],
        },
    ]
}
```
そうしますと、VSCodeのタスクの実行でマインクラフトの起動できるようなります。

# Modの初期設定

今のままだと図1.7のようにModsをクリックしModを確認するとExample Modという名前になっています。これをこれから作成するMod名に変更していきます。
![](/images/minecraft-mod-settings-coco9122/img-0007.png)
*図1.7 Minecraft上に見えるModの名前*

## gradle.propertiesの編集

基本的な事項の変更は主にgradle.propertiesを編集していきます。その中でもModに関係するところを編集していきます。

今回はTanuki Modを作成することにしましょう。

```diff txt:gradle.properties
# Modを特定するためにユニークなID。正規表現 [a-z][a-z0-9_]{1,63} に適合する必要がある。
- mod_id=examplemod
+ mod_id=tanukimod

# Mod名
- mod_name=Example Mod
+ mod_name=Tanuki Mod

# Modのバージョン
mod_version=1.0.0

# ModのグループIDになります。MODソースに使用されるベースパッケージと一致させる必要がある。
- mod_group_id=com.example.examplemod
+ mod_group_id=jp.dawnzlight.tanukimod

# Modの作成者の名前
- mod_authors=YourNameHere, OtherNameHere
+ mod_authors=CoCo9122
```

## ディレクトリ/ファイルの変更

次にMODソースに使用されるベースパッケージと一致させる必要がありますので、ここも変更します。

src配下のディレクトリ構成は以下のようになっていると思いますが、ここを変更します。
ディレクトリの構想とファイル名を変更します。
```
# 変更前
.
└── src/main/java/com/example/examplemod/
    ├── Config.java
    └── ExampleMod.java

# 変更後
.
└── src/main/java/jp/dawnzlight/tanukimod/
    ├── Config.java
    └── TanukiMod.java
```

次にディレクトリの変更とMod名の変更に伴ってコードを以下のように変更します。

```diff java:src/main/java/jp/dawnzlight/tanukimod/TanukiMod.java
- package com.example.examplemod;
+ package jp.dawnzlight.tanukimod;

- @Mod(ExampleMod.MODID)
- public class ExampleMod {
+ @Mod(TanukiMod.MODID)
+ public class TanukiMod {

-  public static final String MODID = "examplemod";
+  public static final String MODID = "tanukimod";

-  public ExampleMod() {
+  public TankukiMod() {
```

```diff java:src/main/java/jp/dawnzlight/tanukimod/Config.java
- package com.example.examplemod;
+ package jp.dawnzlight.tanukimod;


- @Mod.EventBusSubscriber(modid = ExampleMod.MODID, bus = Mod.EventBusSubscriber.Bus.MOD)
+ @Mod.EventBusSubscriber(modid = TanukiMod.MODID, bus = Mod.EventBusSubscriber.Bus.MOD)
```

変更が完了いたしましたら、Minecraftを起動してみましょう。図1.8のように`Tanuki Mod`に変更されていれば完了です。

![](/images/minecraft-mod-settings-coco9122/img-0008.png)
*図1.8 Minecraft上に見えるModの名前*

# まとめ

今回はMinecraftのMod開発構築を方法をまとめました。
意外に簡単に構築することができました。

---
何かあればコメントいただければと思います。