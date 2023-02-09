---
title: "sshでgit cloneする"
emoji: "👌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['github','git']
published: false
---

# Githubのリポジトリをクローン

Githubからクローンする場合は`HTTPS`と`SSH`と`Github CLI`があります。
この中でも今回は`SSH`によりクローンの方法になります。

-----

## Windows10の場合

まずはエクスプローラーあるいはコマンドプロントで`C:\User\<ご自身のユーザー名>`に移動します。
ここのフォルダー直下で`.shh`フォルダーを作成します。
```sh
mkdir .shh
```

次に.sshフォルダーに移動してください。
```sh
C:\Users\<ご自身のユーザー名>> cd .ssh
C:\Users\<ご自身のユーザー名>\.ssh>
```


