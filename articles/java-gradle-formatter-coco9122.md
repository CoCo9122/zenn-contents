---
title: "[java]VSCodeにてJavaのコードをフォーマットして、github Actionsでそれをチェックしたい"
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["java", "gradle"]
published: true
publication_name: "dawnzlight"
---


MinecraftのMod開発にJavaを使い始めて、コードの統一化（整合性）を保つためにフォーマットを実装しようとすると少し詰まってしましました。

PythonにはRuffやTypescriptにはprettierといった主流のライブラリ等の記事が簡単に見つかるのですが、私自身がJava初心者というもとこもあると思いますが、Javaではすんなり見つからず、、、

こんな苦戦した経緯もありまので、今回はJavaのフォーマット実装の解説になります。少しでも参考になれば幸いです。

# Javaのコードをフォーマットしたい

今回はフォーマットするにあたって、以下の条件は満たすようなものを探すことにします。

1. git add ./する前に、できるだけ簡単にコードを整形したい
2. pull requestの際にgithub actionsにてスタイルチェックをし、フォーマットされていなければ失敗としたい

これらの条件があれば、開発においてコードを最低限は整合性を保つことができると思います。

念のために以下に、私自身の動作環境を載せておきます。

|||
|-|-|
| OS | Windows10 home |
| Editor| VSCode |
| Java | 22.0.2 |
| Gradle | 8.7 |

それでは、実際に実装をしていきたいと思います。

# VScodeのEditorにJavaのコーディングルールの適用

## はじめに
コードをフォーマットする上で、様々なスタイルのフォーマット方法があると思いますが、今回はJavaで使用するものはGoogleの`google check`を使用します。（というか調べたら結果的にこれになりました。）

## Check for Javaのインストール

VSCodeの拡張機能にて`Checkstyle for Java`と検索し、図1.1のように一番上位に表示される`Checkstyle for Java`をインストールしましょう。

![](/images/java-gradle-formatter-coco9122/img-0001.png)
*図1.1 Check for Javaの検索画面*

## google_checksのスタイルの適用

CheckStyleのGithubにアクセスし、`google_checks.xml`のファイルをダウンロードし、プロジェクトのディレクトリに保存します。

https://github.com/checkstyle/checkstyle/blob/master/src/main/resources/google_checks.xml

最後に保存した`google_checks.xml`をVSCode上で右クリックし、図1.2のように`Set the Checkstyle Configuretion File`を押下し、ファイルを設定します。そうしますと、Javaにgoogle checkのスタイルが適用されます。

![](/images/java-gradle-formatter-coco9122/img-0002.png)
*図1.2 Checkスタイルの設定*

MinecraftのMod用のプロジェクトに適用すると大量にproblemが出ると思います。

以上にて、VScode上のコーディングルールの適用は完了です

# Spotlessの導入

次は指摘された内容を一つ一つ手動ではとてもめんどくさいと思いますので自動で直せるようにします。これが条件1の`git add ./する前に、できるだけ簡単にコードを整形したい`の内容になります。

今回は使用したものはSpotlessになります。以下がGithubになりますので詳細は以下をご覧ください。
https://github.com/diffplug/spotless

## Spotlessの設定

今回はGradleを使用してるため、build.gradleを以下のように修正します。pluginに先ほどの`spotless`の追加、spotlessにgooglejavaformatを追加する設定を書きます。

```diff txt:build.gradle
plugins {
    ・・・
+   id "com.diffplug.spotless" version "7.0.0.BETA1"
}


・・・

+ apply plugin: "com.diffplug.spotless"

+ spotless {
+   java {
+       googleJavaFormat()
+   }
+ }
```

記入後、プロジェクトをリロードするとspotlessのコマンドが実行可能になっています。
一度以下のようなコマンドを実行すると、Javaのファイルがフォーマットされたと思います。

```sh
gradlew spotlessApply
```

以上で設定は完了です。

## VScodeのタスクに追加する

追加で誰でも実行できるようにVScodeのタスクに追加しましょう。
新規に.vscodeのディレクトリとtasks.jsonを作成します。

![](/images/java-gradle-formatter-coco9122/img-0003.png)
*図1.3 .vscode/tasks.jsonの作成*

作成しましたら、以下の内容を書きこみましょう。ここではtasksの詳細は省きます。
```json:.vscode/tasks.json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "コードの整形",
            "type": "shell",
            "command": "gradlew spotlessApply",
            "args": [],
            "problemMatcher": [],
        },
    ]
}
```

記入が終わりましたら、VSCodeの上にある検索窓をクリックし、図1.4のようなセレクトが表示されますので、`Run Task`をクリックします。

![](/images/java-gradle-formatter-coco9122/img-0004.png)
*図1.4 VSCodeの検索窓のクリック時*

次に、tasks.jsonに記入した図1.5のように`コードの整形`という項目があると思いますので、クリックします。そうすると先ほどコマンドを実行したように、コードをフォーマットしたgradleのタスクが実行されると思います。

![](/images/java-gradle-formatter-coco9122/img-0005.png)
*図1.5 VScodeのタスク一覧*

図1.6のように成功したらOKです。

![](/images/java-gradle-formatter-coco9122/img-0006.png)
*図1.6 タスク実行完了画面*

以上にてSpotlessの導入は終了です。

# Github Actionsにスタイルチェックを導入

最後に条件2の`pull requestの際にgithub actionsにてスタイルチェックをし、フォーマットされていなければ失敗としたい`を満たすものを作成していきます。

## github actionsの作成

今回はnikitasavinovさんのCheckstyle GitHub Actionを使用していきましす。これはプルリク時にcheckstyleとreviewdogというものを表示してくれるものになります。

https://github.com/nikitasavinov/checkstyle-action

それではまずは、プロジェクトに.githubとworkflowsのディレクトリの作成をします。作成したらディレクトリにstylecheck.yamlを作成します。

作成しましたらGithub Actionsを書いていきます。(Github Actionsの詳細の解説は省きます。)

```yaml: .github/workflows/stylecheck.yaml
name: stylecheck
on:
  pull_request:
    types: [opened, synchronize, reopened]
    branches:
      - stg

jobs:
  test:
    runs-on: ubuntu-20.04
    steps:
      - name: Checkout
        uses: actions/checkout@v2
      - name: Run check style
        uses: nikitasavinov/checkstyle-action@master
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          reporter: 'github-pr-check'
          tool_name: 'reviewdog'
          fail_on_error: true
```

実際にプルリクエストしていますと、図1.7のように表示され、reviewdogが表示される際はgithub actionsが失敗しているのが分かります。

![](/images/java-gradle-formatter-coco9122/img-0007.png)
*図1.7 プルリクエスト時の動作*

以上にてgithub actionsの設定は完了です。

# まとめ
今回はJavaのコードのフォーマットからgithub ActionsでStyleチェックする内容を実装しました。

---
何かあればコメントいただければと思います。


# 参考文献

大変お世話になりました。ありがとうございます。

https://java.civic-apps.com/vscode-checkstyle/

https://progret.hatenadiary.com/entry/2019/12/09/165048