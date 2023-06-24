---
title: "作成するアプリケーション「勤怠管理」"
---

# 作成するWebアプリケーション

今回作成するWebアプリケーションは**勤怠管理システム**になります。対象とするユーザは以下のように設定しましょう。

- 管理者
- 一般社員

次にそれぞれのユーザにどのような機能が必要なのか設定していきましょう。

まずは管理者の機能を考えていきます。管理者としては働いているユーザの勤怠履歴の閲覧と自分自身の勤怠登録ができればよいと考えられます。それに加えて、ユーザの追加、削除も機能として合った方がよさそうです。したがって機能は以下のようになります。

- 勤怠登録
- 全ユーザの勤怠記録の閲覧
- ユーザの閲覧、追加、削除

次に一般社員の機能を考えていきます。一般社員は勤怠の登録と自分自身の勤怠履歴が見れれば問題ないと思います。したがって機能は以下のようになります。

- 勤怠登録
- 勤怠記録の閲覧

それぞれのユーザと機能をまとめると以下の表になります。勤怠登録は管理者と一般社員は同じ機能、勤怠記録の閲覧は一般社員のみ一部機能制限、ユーザの閲覧、追加、削除は管理者のみ使用可能な機能になります。

|ユーザー|勤怠登録|勤怠記録の閲覧|ユーザの閲覧、追加、削除|
|:---:|:---:|:---:|:---:|
|管理者|〇|〇(全ユーザ)|〇|
|一般社員|〇|△(自分のみ)|×|

## Webアプリケーションの作成前に

早速、管理者と一般社員は同じ機能である勤怠登録画面の作成していきたいところですが、その前に作成に採用するUIデザインや必要なUIライブラリをセットアップをします。
使用するUIデザインはReactのコンポーネントとの相性が良い**アトミックデザイン**を参考にします。UIライブラリはデザインの簡略化のために**Chakra UI**を使用していきます。

### Chakra UIのセットアップ

先にChakra UIのセットアップしていきます。`zenn-react-gas`配下に移動し以下コマンドを実行します。

```sh
npm install -D @chakra-ui/react @emotion/react @emotion/styled framer-motion @chakra-ui/icons
```

これでChakra UIのインストールは終わりです。次はChakra UIを使用できるようにコードを修正していきます。まずは`./src/main.jsx`を以下のように修正します。

```diff jsx:./src/main.jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.jsx'
- import './index.css'

+ import { ChakraProvider } from '@chakra-ui/react'

ReactDOM.createRoot(document.getElementById('root')).render(   
    <React.StrictMode>
+       <ChakraProvider>
            <App />
+       </ChakraProvider>
    </React.StrictMode>,
)
```

次に`./src/App.jsx`を以下のように修正します。

```diff jsx:./src/App.jsx
import { useState } from 'react'
- import './App.css'

function App() {
  const [count, setCount] = useState(0)

  return (
    <>
      <h1>Hello World!</h1>
      <div className="card">
        <button onClick={() => setCount((count) => count + 1)}>
          count is {count}
        </button>
      </div>
    </>
  )
}

export default App
```

最後にindex.cssとApp.cssの削除します。これでChakra UIのセットアップは終わりになります。Chakara UIの公式サイトは以下を参照ください

@[card](https://chakra-ui.com/)


### アトミックデザインに適した設定

次にアトミックデザインに適したディレクトリの作成を行います。`./src`配下に以下の5つのディレクトリを作成します。

```
src:.
│  atoms
│  molecules
│  Organisms
│  Templates
└─ Pages
```

アトミックデザインのディレクトリを作成は以上になります。アトミックデザインの参考サイトは以下を参照ください。
@[card](https://spice-factory.co.jp/web/about-atmicdesign/)

以上の設定が終わりましたら一度Githubのプッシュしておきましょう。