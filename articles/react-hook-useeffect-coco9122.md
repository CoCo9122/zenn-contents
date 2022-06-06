---
title: "React Hooksの備忘録 ～useEffect()～"
emoji: "😺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['react', 'javascript']
published: true
---

# ReactのReact Hooks
フロントエンド開発で学習したものを忘れないようにReact Hooksの備忘録として投稿。

- [useState()](https://zenn.dev/coco9122/articles/react-hook-usestate-coco9122)

https://zenn.dev/coco9122/articles/react-hook-usestate-coco9122

- useEffect()

:::message
本記事はuseEffectについての内容になります。
:::

# useEffect()の基本的な使い方

Reactの公式のHookのAPI Refereneceより

>副作用を有する可能性のある命令型のコードを受け付けます。
>Accepts a function that contains imperative, possibly effectful code.

>デフォルトでは副作用関数はレンダーが終了した後に毎回動作しますが、特定の値が変化した時のみ動作させるようにすることもできます。
>By default, effects run after every completed render, but you can choose to fire them only when certain values have changed.

>もしも副作用とそのクリーンアップを 1 度だけ（マウント時とアンマウント時にのみ）実行したいという場合、空の配列 ([]) を第 2 引数として渡すことができます。
>If you want to run an effect and clean it up only once (on mount and unmount), you can pass an empty array ([]) as a second argument.

とのこと。ここでいう副作用はDOMの変更や、APIの通信等があげられます。

初回レンダー時および、アンマウント時、特定の処理（値の変化）を行った場合に、副作用を実行させるHook。

```jsx
import { useEffect } from 'react' 
```
以下のように記述します。
```jsx
useEffect(() => {/* */ return () => {}}, [])
```

`{/* */ return () => {}}`が副作用関数、そのうち`return () => {}`はアマウント時に実行されます。`[]`が特定の値が変化した時のみ動作させるための特定の値を配列にした物になります。

したがって、この場合だと空配列が指定されているため、初回レンダー時および、アンマウント時にしか実行することができません。

実際に行ってみます。今回は`useState`で定義した`count`に変更があると`useEffect`がよばれ、`count(Effect)`が加算されるというものです。

```jsx
useEffect(() => {
    setCountEffect((currentCount) => currentCount+1)
  },[count])
```

`count`および`countEffect`の初期値は同じゼロですが、`useEffect`は初回レンダー時に実行されるので`countEffect`は1になっています。
AddやRESETを押すと`count`が加算またリセットされることで、`count`が変更されたことになります。その変更を読み取って`useEffect`が実行され、、`count(Effect)`が加算されるという流れになります。

@[codepen](https://codepen.io/coco9122/pen/KKQBoLL?default-tab=js,result)

時間があればアマウント時のもかこうかなと

# まとめ

`useEffect`の要件をまとめると以下のようになります。
- 副作用関数を初回レンダー時、アマウント時、特定の値が変更された際に実行する
- 第二引数に特定の値を設定
- アマウント時に実行したいもの副作用関数内に`return () => {}`を記述
- `useEffect()`の引数で初期値を設定可能

# 参考文献
- React Hooks入門　著者「soarflat」[Link](https://www.amazon.co.jp/React-Hooks-%E5%85%A5%E9%96%80-%E3%83%95%E3%83%83%E3%82%AF%E3%81%AE%E5%9F%BA%E7%A4%8E%E3%82%84%E4%BD%BF%E3%81%84%E6%89%80%E3%82%92%E3%81%97%E3%81%A3%E3%81%8B%E3%82%8A%E7%90%86%E8%A7%A3%E3%81%97%E3%81%A6%E4%BD%BF%E3%81%84%E3%81%93%E3%81%AA%E3%81%99-soarflat-ebook/dp/B08JTXQ2M6)
- React公式サイト [Link](https://ja.reactjs.org/docs/hooks-reference.html)

