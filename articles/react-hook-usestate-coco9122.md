---
title: "React Hooksの備忘録 ～useState()～"
emoji: "📌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['react', 'javascript']
published: false
---

# ReactのReact Hooks
フロントエンド開発で学習したものを忘れないようにReact Hooksの備忘録として投稿。

- useState()
- useEffect()
- useRef()

:::message
本記事はuseStateについての内容になります。
:::

# useState()の基本的な使い方

Reactの公式のHookのAPI Refereneceより

>ステートフルな値と、それを更新するための関数を返します。
>Returns a stateful value, and a function to update it.

とのこと。データの保持と更新することのできるHook。

importは以下の文になります。

```jsx
import { useState } from 'react' 
```
以下のように記述します。
```jsx
const [state, setState] = useState(initilaState)
```
ここでいう`state`は変数名、`initialState`は初期値になります。

初回レンダー時に`state`に`initialState`が代入されます。

`setState`は関数であり、`state`を更新するために使用されます。それとともにコンポーネントを再レンダーを行います。

実際に行ってみます。

今回はシンプルに`initialState`には`true`代入し、TRUEボタンを押したらTRUEにFALSEボタンを押したらFALSEとなります。実際にボタンを押すことで、stateに値が代入され、`<p>`が再レンダーされています。

@[codepen](https://codepen.io/coco9122/pen/poaZEmR?default-tab=js,result)

### これだと再レンダーされない

上記の書き方だと再レンダーされますが、変数を更新する関数を使わず、変数を直接更新すると、再レンダーされません。Reactの公式のHookのAPI Refereneceより

>現在値と同じ値で更新を行った場合、React は子のレンダーや副作用の実行を回避して処理を終了します。
>If you update a State Hook to the same value as the current state, React will bail out without rendering the children or firing effects.

したがって、変数を更新する関数によって、更新する必要があります。

## useState()はオブジェクトも使用可

`initialState`の初期値にオブジェクトを代入することもできます。

```jsx
const [state, setState] = useState({
    fizz: 0,
    buzz: 0,
})
```

実際に行ってみます。

Add Fizzボタンを押すとFizzが加算され、Add Buzzボタンを押すとBuzzが加算されます。RESETボタンで加算された値がゼロに戻ります。

@[codepen](https://codepen.io/coco9122/pen/YzejNBr?default-tab=js,result)

ここで使用される三点リーダーに関しましてはここのリンクを参照ください。
https://pgmemo.tokyo/data/archives/2067.html

# まとめ

`useState`の要件をまとめると以下のようになります。
- 第一引数に変数を定義（オブジェクトも可）
- 第二引数に変数を更新する関数を定義
- `useState()`の引数で初期値を設定可能

# 参考文献
- React Hooks入門　著者「soarflat」[Link](https://www.amazon.co.jp/React-Hooks-%E5%85%A5%E9%96%80-%E3%83%95%E3%83%83%E3%82%AF%E3%81%AE%E5%9F%BA%E7%A4%8E%E3%82%84%E4%BD%BF%E3%81%84%E6%89%80%E3%82%92%E3%81%97%E3%81%A3%E3%81%8B%E3%82%8A%E7%90%86%E8%A7%A3%E3%81%97%E3%81%A6%E4%BD%BF%E3%81%84%E3%81%93%E3%81%AA%E3%81%99-soarflat-ebook/dp/B08JTXQ2M6)
- React公式サイト [Link](https://ja.reactjs.org/docs/hooks-reference.html)

-----
## 気になったこと
useStateを定義する際、
```jsx
const [state, setState] = useState(initilaState)
```
となり、`[◇◇, set◇◇]`の形で書かれますが、
```jsx
const [state, noSetState] = useState(initilaState)
```
と書いても`noSetState`関数は問題なく動作しますが、set◇◇の方が良いと思われます。

