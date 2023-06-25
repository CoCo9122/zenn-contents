---
title: "勤怠登録画面の作成"
---

# 勤怠登録画面の作成

管理者と一般社員は同じ機能である勤怠登録画面の作成していきます。

## フロントエンド(Web画面)の作成

まずはユーザが利用するフロント画面を作成していきます。

骨組みとなるメインページとメインテンプレートを作成していきます。`./src/components/templates/`に`MainTemplate.jsx`を作成し以下のコードをコピーします。

```jsx:./src/components/templates/MainTemplate.jsx
import React from 'react'

const MainTemplate = () => {

    return (
        <h1>メインテンプレート</h1>
    )
}

export default MainTemplate
```

次に`./src/components/pages/`に`MainPage.jsx`を作成し以下のコードをコピーします。その際に先ほど作成した`MainTemplate`コンポーネントをインポートし使用します。

```jsx:./src/components/pages/MainPage.jsx
import React from 'react'

// コンポーネントのimport
import MainTemplate from '../templates/MainTemplate'

const MainPage = () => {

    return (
        <MainTemplate />
    )
}

export default MainPage
```

最後に`./src/App.jsx`のファイルを以下のように`MainPage`コンポーネントのインポート、`Hello World!`を先ほど作成した`MainPage`コンポーネントと入れ替えます。

```diff jsx:./src/App.jsx
import React from 'react'

+ // コンポーネントのimport
+ import MainPage from './components/pages/MainPage'

function App() {

  return (
-   <>
-       <h1>Hello World!</h1>
-   </>
+   <MainPage />
  )
}

export default App
```
骨組みとしては完成となります。ローカルサーバを立ち上げ、Webブラウザを確認すると画像左端に`メインテンプレート`と表示されているはずです。

### ヘッダーの作成

まずはHeaderの作成を行います。以下の機能を持たせることにします。
- ヘッダーに「勤怠管理」の表示
- カラーモードのボタン（ライトモードとダークモードをボタンで入れ替える機能）

カラーモードのボタンのコンポーネントを作成します。コンポーネントは最小であるため`./src/components/atoms/`に`ColorModeButton.jsx`として作成します。

```jsx:./src/components/atoms/ColorModeButton.jsx
import React from "react"

// Chakra UIのコンポーネントのImport
import { useColorMode, IconButton } from "@chakra-ui/react"
import { SunIcon, MoonIcon } from "@chakra-ui/icons"

const ColorModeButton = () => {
    const { colorMode, toggleColorMode } = useColorMode()

    return (
        <IconButton aria-label='ColorModeSwitch' icon={colorMode==='light'? <MoonIcon />:<SunIcon />} onClick={toggleColorMode} variant='outline'/>
    )
}

export default ColorModeButton
```

次にヘッダーのコンポーネントを作成します。`./src/components/molecules/`に`MainHeader.jsx`を作成します。勤怠管理と先ほどの作成した`ColorModeButton`をインポートし使用します。

```jsx:./src/components/molecules/MainHeader.jsx
import React from "react"

// Chakra UIのコンポーネントのImport
import { Flex, Text, Spacer, useColorModeValue } from "@chakra-ui/react"

// コンポーネントのImport
import ColorModeButton from "../atoms/ColorModeButton"

const MainHeader = () => {

    const headerBgColor = useColorModeValue('teal.200', 'teal.600')

    return(
        <Flex h={"50px"} justify='center' align='center' bgColor={headerBgColor} pl={10} pr={10}>
            <Text fontSize={'4xl'}>
                勤怠管理
            </Text>
            <Spacer />
            <ColorModeButton />
        </Flex>
    )
}

export default MainHeader
```

最後に作成した`MainHeader`コンポーネントをインポートし使用するために`MainTemplate`コンポーネントを以下のように修正します。これでヘッダーの作成は終わりです。

```diff jsx:./src/components/templates/MainTemplate.jsx
import React from 'react'

+ // コンポーネントのImport
+ import MainHeader from '../molecules/MainHeader'

const MainTemplate = () => {

    return (
-       <h1>メインテンプレート</h1>
+       <>
+          <MainHeader />
+       </>
    )
}

export default MainTemplate
```

これでヘッダーの作成は終わりです。

### 勤怠登録(ボディ)の作成