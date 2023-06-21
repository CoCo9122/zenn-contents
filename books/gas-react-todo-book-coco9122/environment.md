---
title: "開発環境を構築"
---

ここでは開発環境を構築していきます。環境はWindows10で作成していきます。

# 事前確認
## Node.jsのインストール

Nodeが必要なのでコマンドプロンプトで以下のコマンドで確認してください。

```sh
node -v
```
```sh
npm -v
```
バージョンが表示されれば、インストール済みとなります。表示されない場合は以下サイトを参考にインストールしてください。
@[card](https://prog-8.com/docs/nodejs-env-win)

# GASにデプロイ可能なローカルの環境構築
Githubでリポジトリ作成、ローカルの環境構築の順で作成します。

## Githubによるリポジトリ作成
:::message
Githubのリポジトリ作成は任意になります。
:::
::::details Githubでリポジトリ作成
Githubで新規リポジトリの作成を行います。(アカウントがない方は登録お願いします。)

Githubのリポジトリ一覧に移動し図1.1のように左側にある`New`ボタンを押下します。
![](/images/gas-react-todo-book-coco9122/deploy-0001.jpg)
*図1.1 githubのリポジトリ一覧画面*

そうすると図1.2のような作成画面に移動します。今回はZenn向けのリポジトリとして使用するため、リポジトリの名前を`zenn-react-gas`にします。加えてREADME.mdの作成もチェックにしています。設定が終わりましたら`Create repository`をクリックしリポジトリを作成します。
:::message alert
外部公開したくない場合はprivateでの作成をお願いします。
:::
![](/images/gas-react-todo-book-coco9122/deploy-0002.jpg)
*図1.2 githubのリポジトリ作成画面*

図1.3のような画面に遷移していれば、リポジトリの作成完了になります。
![](/images/gas-react-todo-book-coco9122/deploy-0003.jpg)
*図1.3 zenn-react-gasのリポジトリ*

次にローカルの環境にクローンします。図1.3の左上部にある`<> Code`をクリックすると図1.4のようなポップオーバーが表示されます。HTTPSでクローンする場合はHTTPSをクリックしURL(http方式)をコピーします。SSHでクローンする場合はSSHをクリックし、git@github.com(ssh方式)をコピーします。

![](/images/gas-react-todo-book-coco9122/deploy-0004.jpg)
*図1.4 <>Codeクリック時に表示されるポップオーバー*
:::message
githubでssh接続するためには設定が必要なので以下手順を参考にしてください。
:::
@[card](https://qiita.com/shizuma/items/2b2f873a0034839e47ce)

続いてgitコマンドを実行する必要があるのでgitが実行可能かどうか以下のコマンドを実行します。
```sh
git --version
```
バージョンが表示されれば、インストール済みとなります。表示されない場合は以下サイトを参考にインストールしてください。
@[card](https://qiita.com/taiponrock/items/632c117220e57d555099)

最後に任意のディレクトリに移動し以下のコマンドを実行しクローンしてください。
```sh
git clone <先ほどコピーしたURLまたはgit@github.com>
```

これでGithubでリポジトリ作成は完了になります。

::::

## Viteを使用した環境構築

ビルドツール`Vite`を使用して環境構築を行います。Githubによるリポジトリ作成を行った方は以下のコマンドでディレクトリを移動してください。`zenn-react-gas`をカレントディレクトリにします。
```sh
cd zenn-react-gas
```
作成していない方は新しく`zenn-react-gas`のディレクトリを作成し、`zenn-react-gas`をカレントディレクトリにします。

### プロジェクトを以下のコマンドで作成

```sh
npm create vite@latest ./
```
コマンドを実行すると`Ok to proceed?`と問われるので`y`を入力し、Enterを押下します。
また、`? Current directory is not empty. Remove existing files and continue?`と問われるので`y`を入力し、Enterを押下します。
```sh
Need to install the following packages:
  create-vite@latest
Ok to proceed? (y) y
? Current directory is not empty. Remove existing files and continue? » (y/N)
```
次に`Select a framework: » - Use arrow-keys. Return to submit.`と問われるので十字キーを使い`React`を選択しEnterを押下します。

```sh
Need to install the following packages:
  create-vite@latest
Ok to proceed? (y) y
√ Current directory is not empty. Remove existing files and continue? ... yes
? Select a framework: » - Use arrow-keys. Return to submit.
    Vanilla
    Vue
>   React
    Preact
    Lit
    Svelte
    Others
```

最後に`? Select a variant: » - Use arrow-keys. Return to submit.`と問われるので十字キーを使い`JavaScript`を選択しEnterを押下します。

```sh
Need to install the following packages:
  create-vite@latest
Ok to proceed? (y) y
√ Current directory is not empty. Remove existing files and continue? ... yes
√ Select a framework: » React
? Select a variant: » - Use arrow-keys. Return to submit.
    TypeScript
    TypeScript + SWC
>   JavaScript
    JavaScript + SWC
```

作成が完了すると`zenn-react-gas`配下は以下のようなディレクトリ構成になっています。
```
D:.
│  .eslintrc.cjs
│  .gitignore
│  index.html
│  package.json
│  vite.config.js
│
├─public
│      vite.svg
│
└─src
    │  App.css
    │  App.jsx
    │  index.css
    │  main.jsx
    │
    └─assets
            react.svg
```


### 全てのリソースをビルド時にインライン化してくれるViteのプラグインライブラリの導入

以下のコマンドを実行します。
```sh
npm install vite-plugin-singlefile -D
```

インストールが完了すると`zenn-react-gas`配下に`package_lock.json`と`node_modules`が追加されます。


### claspのインストール
以下のコマンドを実行します。
```sh
npm install -D @google/clasp
```

### Googleログイン
以下のコマンドを実行します。
```sh
clasp login 
```
そうしますと、Googleサイト（環境によっては他検索サイト）が開きアカウント選択画面に移行しますので作成したいアカウントを選択します。`clasp – The Apps Script CLI が Google アカウントへのアクセスをリクエストしています`と問われますので許可を押下します。
Web画面に`Logged in! You may close this page.`と表示されたらログイン完了になりますのでウィンドウを閉じてください。

### GASのWEBアプリケーションの作成

次に以下のコマンドを実行します。
```sh
clasp create --type webapp
```
新たに`zenn-react-gas`配下に`appsscript.json`と`.clasp.json`が追加されます。

### ファイル作成、移動

- serverフォルダーの作成、appsscript.jsonをserverフォルダーに移動
- serverフォルダー配下にcode.jsの作成

7. .clasp.jsonの修正

```diff json
{
    "scriptId":"XXX...XXX",
-   "rootDir":"C:\\ ... \\incidet-gas"
+   "rootDir":"./server"
}
```

8. vite.config.jsの修正

```js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import { viteSingleFile } from 'vite-plugin-singlefile'

export default defineConfig({
  plugins: [react(), viteSingleFile()],
  build: {
    outDir: "server/hosting",
  },
});
```

9. server/code.jsの修正

```js
function doGet() {
    return HtmlService.createHtmlOutputFromFile("hosting/index.html")
      .addMetaTag("viewport", "width=device-width, initial-scale=1")
      .setTitle("勤怠管理")
}
```

10. server/appsscript.jsonの修正

```diff js
{
- "timeZone": "...",
+ "timeZone": "Asia/Tokyo",
  "dependencies": {
  },
+ "webapp":  {
+   "access": "MYSELF",
+   "executeAs": "USER_DEPLOYING"
+ },
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8"
}
```

11. GASにプッシュ、デプロイ

```sh
clasp push
```
```sh
clasp deploy
```
```sh
clasp deploy --deploymentId=...
```

環境構築は以下サイトを参考にしています。
@[card](https://engineer.retty.me/entry/2022/12/22/150035)



# Hello Worldのデプロイ