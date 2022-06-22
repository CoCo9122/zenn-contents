---
title: "replitにおいてNode.jsでテストする"
emoji: "🐡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['nodejs']
published: false
---

# 準備
replitでNode.jsを選択してプロジェクトを作成します。

index.jsが作成されたプロジェクトが作られます。

ここに以下のコードを書きます。
```js:index.html
const sum = (x, y) => {
    return x+y
}

const mean = (arr) => {
    let total = arr.reduce(function(sum, element){
        return sum + element;
    }, 0)
    return Math.floor(total/arr.length)
}

exports.sum = sum
exports.mean = mean
```
`sum`は入力されたふたつの和を返す関数です。
`mean`は配列を入力し、その配列の平均値を返す関数です。

これに対してテストをおこないます。

# テストの実行
いかの赤い枠で囲われた✓マークをクリック
![](/images/replit-nodejs-test-01.png)
そうする以下の画面になる。
![](/images/replit-nodejs-test-02.png)
ここでAdd testをクリックするとテストコートを書く画面がひらきます。
ここで以下のコードを書きます。
```js
test("1＋1は2です。", async function() {
    expect(index.sum(1, 1)).toBe(2)
})
```
![](/images/replit-nodejs-test-03.png)
saveを押すといかに関数名が追加される。
![](/images/replit-nodejs-test-04.png)
この状態でRun testを行うとconsoleに以下の画面になり、テストが成功する。
![](/images/replit-nodejs-test-05.png)

つぎに以下のテストコードを二つ追加します。
```js
test("1＋1は3です。", async function() {
    expect(index.sum(1, 1)).toBe(3)
})
```

```js
test("配列1-2-3-4-5の平均値は3です。", async function() {
    expect(index.mean([1, 2, 3, 4, 5])).toBe(3)
})
```

Run testを行うと「1＋1は3です。」は失敗し、「配列1-2-3-4-5の平均値は3です。」は成功する。

テストができていることが分かる。

具体的なAPIは以下に

https://jestjs.io/ja/docs/api#testname-fn-timeout

https://jestjs.io/ja/docs/expect#expectvalue