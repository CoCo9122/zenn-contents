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

それでは本題の勤怠登録のWeb画面を作成しています。まずはアバターとオンライン表示可能なアイコンのコンポーネントの作成します。`./src/components/molecules`配下に`AttendanceAvatar.jsx`を作成し、以下を入力します。これでアバターコンポーネントの作成完了です。

```jsx:./src/components/molecules/AttendanceAvatar.jsx
import React from "react"

// Chakra UIのコンポーネントのImport
import { Avatar, AvatarBadge, Flex, Box, Text } from '@chakra-ui/react'

const AttendanceAvatar = () => {
    return(
        <Flex textAlign={'center'} mb={3}>
            <Avatar border={'solid #A0AEC0'} bg='teal.500'>
                <AvatarBadge boxSize='1em' bg='green.300' borderColor={'gray.400'}/>
            </Avatar>
        </Flex>
    )
}

export default AttendanceAvatar
```

次に勤怠登録を行うボタンを作成します。`./src/components/molecules`配下に`AttendanceButtons.jsx`を作成します。今回ボタンは開始、終了ボタンにいたします。以下を入力しますとボタンコンポーネントの作成完了です。

```jsx:./src/components/molecules/AttendanceButtons.jsx
import React from "react"

// Chakra UIのコンポーネントのImport
import { Button, ButtonGroup } from '@chakra-ui/react'

const AttendanceButtons = () => {
    return(
        <ButtonGroup gap='4' m={2}>
            <Button colorScheme='blackAlpha'>開始</Button>
            <Button colorScheme='blackAlpha'>終了</Button>
        </ButtonGroup>
    )
}

export default AttendanceButtons
```

次はロール（役職）の表示を行うコンポーネントを作成します。`./src/components/molecules`配下に`AttendanceRole.jsx`を作成します。ここに管理者かそうでないかが分かるコンポーネントを表示するものになります。以下を入力しますとロールコンポーネントの作成完了です。

```jsx:./src/components/molecules/AttendanceRole.jsx
import React from "react"

// Chakra UIのコンポーネントのImport
import { Stack, Text, Tag, TagLabel, TagRightIcon } from '@chakra-ui/react'

const AttendanceRole = () => {
    return(
        <Stack m={2}>
            <Text fontSize={'2xl'}>Role</Text>
            <Tag size={'md'} w={'min'}>
                <TagLabel>Admin</TagLabel>
            </Tag>
        </Stack>
    )
}

export default AttendanceRole
```

先ほど作成したコンポーネントをまとめるコンポーネントを作成します。これが勤怠登録画面のメインになります。`./src/components/organisms`配下に`AttendanceBody.jsx`を作成します。以下のコードをコピーします。これで作成完了になります。

```jsx:./src/components/organisms/AttendanceBody.jsx
import React from "react"

// Chakra UIのコンポーネントのImport
import { Box, Text, Divider } from '@chakra-ui/react'

// コンポーネントのImport
import AttendanceButtons from "../molecules/AttendanceButtons"
import AttendanceAvatar from "../molecules/AttendanceAvatar"
import AttendanceRole from "../molecules/AttendanceRole"

const AttendanceBody = () => {
    return(
        <Box bg={'linear-gradient( #C4F1F9 0px, #C4F1F9 45px, #A0AEC0 30px, #A0AEC0 100% )'} w={'50%'}  borderRadius={'10'} p={3}> 
            <AttendanceAvatar />
            <Box bg={'gray.600'} borderRadius={'10'} p={2} color={'white'}>
                <Text fontSize={'4xl'} ml={2}>CoCo9122</Text>
                <Text fontSize={'xl'} ml={2}>abcdefg@admin.com</Text>
                <Divider w={'98%'} ml={'1%'}/>
                <AttendanceRole />
                <Box align='right'>
                    <AttendanceButtons />
                </Box>
            </Box>
        </Box>
    )
}

export default AttendanceBody
```

全てのボディを集結させるコンポーネントを作成します。`./src/components/organisms`配下に`MainBody.jsx`を作成します。以下のコードをコピーします。これで作成完了になります。

```jsx:./src/components/organisms/MainBody.jsx
import React from "react"

// Chakra UIのコンポーネントのImport
import { Box, } from "@chakra-ui/react"

// コンポーネントのImport
import AttendanceBody from "./AttendanceBody"

const MainBody = () => {
    return(
        <Box mt={5} w={'70%'} ml={'15%'} pb={5} pt={5} color={'black'}>
            <AttendanceBody />
        </Box>
    )
}

export default MainBody
```

最後に`./src/components/templates/MainTemplate.jsx`に`MainBody`コンポーネントを読み込むためにいかのように修正します。

```diff jsx:./src/components/templates/MainTemplate.jsx
import React from 'react'

// コンポーネントのImport
import MainHeader from '../molecules/MainHeader'
+ import MainBody from '../organisms/MainBody'

const MainTemplate = () => {

    return (
        <>
           <MainHeader />
+          <MainBody />
        </>
    )
}

export default MainTemplate
```

作成が完了しますと、図3.1のようになります。

![](/images/gas-react-todo-book-coco9122/attendance01-0001.jpg)
*図3.1 勤怠登録画面*