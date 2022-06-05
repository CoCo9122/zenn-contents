---
title: "Reactでcreate-react-appを使わず、npm startを使いたい"
emoji: "💭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['react', 'nodejs', 'javascript', 'npm']
published: false
---

# Reactのcreate-react-appは便利

Node.jsを導入している状態で、`npx`コマンドを使用すれば、以下のコマンドで
```
npx create-react-app myapp
```
React環境が簡単に構築できます。そうするとmyappのフォルダーがカレントディレクトリに作成され、以下のようなフォルダー構成になります。
```
.myapp/
 ├── node_modules/
 ├── public
 │   ├── index.html
 │   :
 │   └── robots.txt
 ├── src
 │   ├── index.js
 │   ├── index.css
 │   :
 │   ├── App.js
 │   └── App.css
 ├── .gitignore
 ├── package.json
 ├── package-lock.json
 └── README.md
```

ここの中で`src/`フォルダの中身をいじって開発していくことになります。ほかのフォルダの解説をすこし行うと

`public/`は公開フォルダになります。
`node_modules`には実際にインストールしたモジュールが入っています。
`package.json`にはname, version, descriptionなどのデータは単なるパッケージのメタデータになり。インストールすべきパッケージの情報が書かれています。
`package-lock.json`に実際にインストールしたモジュールのバージョン等の情報が書かれています。

:::message
基本的にこの方法で作成をおすすめします。
:::

-----

## create-react-appを使わないでやってみた。

### Reactの導入する

基本的にReactで必要なパッケージは以下の二つです。
- react
- react-dom
```
npm install react react-dom
```

これらを以上のようにインストールする必要があります。そうするとフォルダの構成は
```
.myapp/
 ├── node_modules/
 ├── package.json
 └── package-lock.json
```
以下のようになります。この時点でReactは動かすことが可能になります。HTMLファイル及びjsファイルを作成することで開発が可能です。

-----
### `npm start`を導入する

ここに`react-scripts`をインストールすることで`npm start`が使用になります。

```
npm install react-scripts
```

`npm start`が使用できるように以下のようなフォルダ構成を作成します。

```diff
.myapp/
 ├── node_modules/
+├── public
+│   ├── index.html
+├── src
+│   ├── index.js
+│   ├── App.js
+│   └── App.css
 ├── package.json
 └── package-lock.json
```
続いて以下のファイルに以下のコードを書き込んできます。

```html:public/index.html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <div id="root"></div>
</body>
</html>
```

```jsx:src/index.js
import React from 'react'
import ReactDOM from 'react-dom'

import App from './App'

ReactDOM.render(
    <App />,
    document.getElementById('root')
)
```

```jsx:src/App.js
import React from 'react'

const App = () => {
    return (
        <h1>Hello World!</h1>
    )
}

export default App
```

```diff json:package.json
{
  "dependencies": {
    "react": "^17.0.2",
    "react-dom": "^17.0.2",
    "react-scripts": "^5.0.1"
  },
+ "scripts": {
+   "start": "react-scripts start",
+   "build": "react-scripts build",
+   "test": "react-scripts test",
+   "eject": "react-scripts eject"
  },
  :
  :
}
```

こうすることで`npm start`が使用可能になります。実行すると`Hello world!`が表示されると思います。

# まとめ

わざわざこうする必要もなく`create-react-app`をすることをお勧めします。ただ、一から設定するので、構成の理解や、いらないパッケージ等も入れなくて済むという点があります。
余談ですが、軽いものを作りたいならCodePenがお勧めです。Reactも簡単に導入できるので、node.js入れずにReactしたい人にお勧めです。
https://codepen.io/