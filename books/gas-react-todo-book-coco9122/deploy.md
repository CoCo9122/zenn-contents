---
title: "Webアプリケーションのデプロイ"
---

# はじめてのデプロイ

## ローカルサーバの立ち上げ
はじめにローカルのWebサーバを立ててみましょう。`zenn-react-gas`配下に移動し、以下のコマンドでローカルのWebサーバが立ち上がります。

```sh
npm run dev
```

デフォルトだとにhttp://127.0.0.1:5173/にWebサーバが立ち上がっています。図2.1のような画面になっていると思います。

![](/images/gas-react-todo-book-coco9122/deploy-0005.jpg)
*図2.1 初期のWebブラウザ画面*

`zenn-react-gas`ディレクトリで以下コマンドを実行します。

```sh
clasp open
```

コマンドを実行するとWebブラウザでGASが起動します。ここに今後はデプロイしていくことになります。

:::message
GASのurlのprojects/＜ランダムな文字列＞と.clasp.jsonのscriptIdが一致します。
:::

![](/images/gas-react-todo-book-coco9122/deploy-0006.jpg)
*図2.2 GASのコードエディタ*

## デプロイ

実際にデプロイします。まずはReactのコードをビルドします。以下のコマンドでビルドができます。
```sh
npm run build
```
ビルドに成功しますと、`./pred`配下に新たな`hosting`のディレクトリが作成され、中にindex.htmlが作成されていると思います。これが**GASにおけるWebサーバ**になります。

次に`./pred`配下のディレクトリを図2.2のGASコードエディタにプッシュをします。コマンドは以下になります。
```sh
clasp push
```

`? Manifest file has been updated. Do you want to push and overwrite?`と問われるので`y`と入力しEnterを押下します。

```sh
? Manifest file has been updated. Do you want to push and overwrite? (y/N)
```
プッシュに成功しますと以下の3つのファイルがGASのコードエディタにプッシュされています。図2.2の画面をリロードをすると図2.3のように追加されているのが分かると思います。

```
└─ pred/appsscript.json
└─ pred/code.js
└─ pred/hosting/index.html
```

:::message
code.gsとhosting/index.htmlの2つしかありませんがappsscript.jsonもプッシュされています。見えないだけです。
:::

![](/images/gas-react-todo-book-coco9122/deploy-0007.jpg)
*図2.3 GASのコードエディタ*

最後にデプロイをします。以下のコマンドでWebアプリケーションをデプロイします。

```sh
clasp deploy
```

デプロイが成功しますと`Created version 1.`とターミナルで表示されます。それでは実際にWebアプリケーションにアクセスしましょう。

先ほどの図2.3のWeb画面より右側上部にある図2.4のような青色のデプロイをクリックし、デプロイを管理をクリックします。

![](/images/gas-react-todo-book-coco9122/deploy-0008.jpg)
*図2.4 デプロイボタン*

そうしますと、図2.5のようなポップアップが画面真ん中に表示されます。ここのウェブアプリの下にあるURLをコピーしアクセスします。

![](/images/gas-react-todo-book-coco9122/deploy-0009.jpg)
*図2.5 デプロイの管理*

図2.6のように画像が入っていないWeb画面が表示されれば完成です。これでどの端末からもこのURLにアクセスしてしまえば、Webアプリケーションを利用することが可能になります。

![](/images/gas-react-todo-book-coco9122/deploy-0010.jpg)
*図2.6 GAS版初期のWebブラウザ画面*

図2.6のWeb画面をみると画像が読み込まれていないことが分かります。GASの場合はローカルにある画像は読み込むことができません。したがって、画像を使用する場合はインターネットにある画像のURLを指定する必要があります。

:::message alert
画像はGASにプッシュできません。
:::

以上が基本的なデプロイ手順になります。

### 補足

二回目の以降のデプロイには新しく以下のデプロイコマンドに`--deploymentId=<デプロイID>`を追加してください。デプロイIDとは図2.5のウェブアプリの上になるデプロイIDの文字列になります。これを指定せずにデプロイしますと**WebアプリケーションのURLがデプロイごとに変更されてしまう**のでお気をつけください。

```
clasp deploy --deploymentId=<デプロイID>
```

:::message
2回目以降のデプロイはデプロイIDを指定してデプロイする必要がある
:::

今後`デプロイする`といった場合は以下のコマンドを実行していることになりますのでご了承ください。
```sh
npm run build
clasp push
clasp deploy --deploymentId=<デプロイID>
```

## Hello world!のデプロイ

早速デフォルトのWeb画面からHello world!の画面に変更していきます。`./src/App.jsx`から編集します。画像の部分をHello world!に置き換えていきます。

```diff jsx:./src/App.jsx
import { useState } from 'react'
- import reactLogo from './assets/react.svg'
- import viteLogo from '/vite.svg'
import './App.css'

function App() {
  const [count, setCount] = useState(0)

  return (
    <>
      <div>
-       <a href="https://vitejs.dev" target="_blank">
-         <img src={viteLogo} className="logo" alt="Vite logo" />
-       </a>
-       <a href="https://react.dev" target="_blank">
-         <img src={reactLogo} className="logo react" alt="React logo" />
-       </a>
-       <p>Hello World!</p>
-     </div>
-     <h1>Vite + React</h1>
+     <h1>Hello World!</h1>  
      <div className="card">
        <button onClick={() => setCount((count) => count + 1)}>
          count is {count}
        </button>
-       <p>
-         Edit <code>src/App.jsx</code> and save to test HMR
-       </p>
      </div>
-     <p className="read-the-docs">
-       Click on the Vite and React logos to learn more
-     </p>
    </>
  )
}

export default App
```

画像は使用しないので`./pubilc/vite.svg`と`./src/assets`を削除します。
最後にデプロイします。そうすると先ほどのWebアプリケーションのURLアクセスしますとHello world!が表示されていると思います。

## Githubにプッシュ

:::message
Githubへのプッシュは任意になります。
:::

:::: details Githubにプッシュする
デプロイが終わりましたら最後にGithubにプッシュしたいと思います。

コミットする前に.gitignoreを編集します。最後の行に以下を追加します。

::: message
.gitignoreはgitの管理対象外ファイルしたいファイル名を記載します。
:::

```diff
+ .clasp.json
+ package-lock.json
```

.clasp.jsonにはスクリプトIDが記載されていますので、公開しないようにします。

次に`zenn-react-gas`配下に`README.md`ファイルを作成します。

最後に`zenn-react-gas`配下に移動しいコマンドを実行しステージングエリアに追加します。

```sh
git add ./
```

以下コマンドでコミットします。

```sh
git commit -m 初期構築
```

Githubにプッシュします。

```sh
git push
```

作成したGithubのリポジトリが更新されていれば成功です。

::::