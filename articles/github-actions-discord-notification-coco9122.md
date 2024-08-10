---
title: "[Github Actions]プルリクエスト時にレビューの確認依頼してませんか？"
emoji: "🌊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["github", "githubactions", "discord"]
published: true
publication_name: "dawnzlight"
---

開発者がプルリクエスト時にコードレビュアー(reviewer)に連絡していませんでしょうか？
コードレビュアー(reviewer)に連絡していないことにより、プロジェクトの円滑な進行を阻害する可能性があります。また、レビューの遅延や見落としが発生し、開発スピードの低下や品質の低下につながる恐れがあります。

そこで、開発者がプルリクエストを作成した際に、自動でコードレビュアーに通知する仕組みを導入することをおすすめします。これにより、開発者の手間を省きながら、レビュープロセスを回すことができます。

そこで、今回はGithub Actionsにて開発者がプルリクエスト時には自動で通知できるようにします。

# CI/CDについて

Github Actionsを作成する上で、CI/CDの技術は欠かせないので説明したいと思います。

CI/CDとは、継続的インテグレーション(Continuous Integration)と継続的デリバリー(Continuous Delivery)となります。ソフトウェア開発プロセスにおいて、頻繁に行われる統合、テスト、デプロイの自動化を指すものになります。

それぞれの特徴について解説していきます。

## CI(継続的インテグレーション)

CIとは開発者が頻繁にコードをメインリポジトリ等にマージする際に、自動でビルド、テスト、統合を実施するものになります。これにより、問題を早期に発見し、修正することができます。

実際は以下のような流れでCIを実施することが多いと思います。ただ、プロジェクトによってはこれ以外のプロセスを追加したりします。

1. 特定のブランチ(main, stg等)にプルリクエストする
2. プルリクエスト後、自動でビルドが行われる
3. ビルド後に自動でユニットテストやインテグレーションテストが実行される
4. テストに合格したコードが自動でプルリクエスト先のブランチにマージされる

### CIのメリット

CIの導入のメリットは以下があげられます。

- 問題の早期発見と修正
- 開発サイクルの短縮
- 品質の向上
- 開発者の生産性向上
  
特に一貫したプロセスでテストとビルドが行われるため、コードのレビューまでの確認項目数の短縮やバグの早期発見につながり、

## CD(継続的デリバリー)

CDとはCIで検証されたコードを自動でステージング環境にデプロイすること、手動承認を経て、本番環境にもデプロイするものになります。
これにより、自動で本番環境へリリースができ、リリースにかかる工数を削減することが可能です。

### CDのメリット

CDの導入のメリットは以下があげられます。

- リリースサイクルの短縮
- リリースの信頼性向上
- 迅速なフィードバック
- 開発者の生産性向上
  
特に、リリースごとに商用の作業を自動化することによって、本来であれば手動で実施していたことによる人為的なミスを削減することが可能です。

## 主なツール

主なツールとしては、Jenkins、CircleCI、GitHub Actionsなどがあります。
これらのツールを使ってCI/CDパイプラインを構築し、ソフトウェア開発の自動化を実現しています。

# プルリクエスト時にDiscord通知する

今回は様々なCI/CDツールがある中、github actionsを用いて、CI/CDの第一歩として開発者がプルリクエスト時には自動で通知できるように作成したいと思います。

これから実装する上でGithub Actionsは公式のDocsがありますのでここを確認しますと、より理解が深まると思います。
https://docs.github.com/ja/actions


## DiscordのWebhookの作成

まずはDiscordのWebhookを作成していきます。

はじめに任意のDiscordのサーバについてgithub actions通知用のgithub actionsのチャンネルを作成します。

![](/images/github-actions-discord-notification-coco9122/img-0001.png)
*図1.1 チャンネルの作成*

作成したチャンネルに図1.2のように移動しましょう。
つぎに赤線が引いてある「チャンネルの編集」をクリックします。

![](/images/github-actions-discord-notification-coco9122/img-0002.png)
*図1.2 Discordのチャンネル*

図1.3のような画面に移動すると思います。次に左の項目の連携サービスをクリックします。そうしますとウェブフックがあると思います。まだ一つも作成していませんので0件のウェブフックになっていると思っています。
ここの「ウェブフックの作成」をクリックします。

![](/images/github-actions-discord-notification-coco9122/img-0003.png)
*図1.3 連携サービス*

そうしますと図1.4のように「Captain Hook」が作成されます。

![](/images/github-actions-discord-notification-coco9122/img-0004.png)
*図1.4 Discordのチャンネル*

先ほど作成したウェブフックをクリックしますと図1.5のような画面に移行します。ここでは名前やアイコンの変更ができます。
今回はここにある「ウェブフックURLをコピー」を押下します。

こちら次のセクションに使用しますのでメモ帳等で控えておきましょう。

![](/images/github-actions-discord-notification-coco9122/img-0005.png)
*図1.5 DiscordのWebhookの編集画面*

## Github Secretsの登録

続いて、先ほどのWebhookのURLをGithub Secretsに登録をします。まずは対象のリポジトリにアクセスします。
アクセスしましたら、「Settings」をクリックします。そうしますと図1.6のような画面へ遷移します。

遷移いたしましたら、赤線の引いてある「Secrets and variables」をクリックします。

![](/images/github-actions-discord-notification-coco9122/img-0006.png)
*図1.6 GithubのSettings画面*

クリックしますと図1.7のようにセレクト画面が表示されますので「Actions」をクリックします。

![](/images/github-actions-discord-notification-coco9122/img-0007.png)
*図1.7 GithubのSecrets and variablesの選択画面*

そうしますと、図1.8のような画面に移行します。次に「New repogistory secret」をクリックします。

![](/images/github-actions-discord-notification-coco9122/img-0008.png)
*図1.8 GithubのActions Secrets and variables画面*

図1.9のような画面に移動します。ここでSecrestsの新規の登録ができます。
Nameには`DISCORD_WEBHOOK`を入力しSecretsには先ほどメモ帳で控えたWebhookのURLを書き込みます。

図1.9はすでに入力済みのものになります。入力が完了しましたら「Add secret」をクリックします。

![](/images/github-actions-discord-notification-coco9122/img-0009.png)
*図1.9 Secretsの登録画面*

登録が完了しましたら、図1.10のように`DISCORD_WEBHOOK`が追加されたら完了です。

![](/images/github-actions-discord-notification-coco9122/img-0010.png)
*図1.10 Secretsの一覧*

以上でGithub Secretsの登録は完了です。これでGithub ActionsにてDISCORD_WEBHOOKが使用可能になります。

## Github Actionsの作成

最後はGithub Actionsの作成を行います。対象のリポジトリにディレクトリ`.github/workflows/`を作成します。

作成したディレクトリに`discord_notification.yaml`を作成します。


作成する上で、今回は以下の要件に沿ったものを作ろうと思います。

- 開発者がプルリクエストするブランチは必ずstgとする
- 商用リリース時はstgからmainにプルリクエストとする

上記の要件を沿ったコードは以下になります。

```yaml:/.github/workflows/discord_notification.yaml
name: discord-notification 
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - stg
      - main
jobs:
  command:
    name: Use Actions Status Discord 
    runs-on: ubuntu-20.04
    steps:
      - name: Dicord Notification
        uses: sarisia/actions-status-discord@v1
        if: always()
        with:
          webhook: ${{ secrets.DISCORD_WEBHOOK }}
```
---

一つ一つ解説していきます。

```yaml
name: discord-notification
```
これはActionsに表示される名前になります。機能が分かる分かりやすい名前を書くのをお勧めします。今回はdiscordの通知になりますので`discord-notification`としました。


```yaml
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - stg
      - main
```
次はいつActionsを実行するかを書いてるものになります。今回の場合だと、mainにpush時に実行、mainかstgにプルリクエスト時に実行となります。
ここを変更することで、随時実行やクーロンによるスケジュール設定による実行ができます。

```yaml
jobs:
  command:
    name: Use Actions Status Discord # ジョブ名
    runs-on: ubuntu-20.04 # ジョブを実行する環境
    steps:
      - name: Dicord Notification # ステップ名
        uses: sarisia/actions-status-discord@v1 # sarisiaさんが作成しているActionです
        if: always()
        with:
          webhook: ${{ secrets.DISCORD_WEBHOOK }} # 先ほど登録したDISCORD_WEBHOOKをここで読み込みます。
```

続いては、何を実行しているかの内容になります。それぞれコメントアウトにて説明しております。
特にDiscord通知には以下のActionsを使用させていただきました。ありがとうございます。
https://github.com/sarisia/actions-status-discord

これにより、作成は完了です。

## 実際にプルリクエストをする

実際にプルリクエストをしますと図1.11のようにActionsが実行されると思います。問題なければ実行が完了すれば緑色のチェックマークになると思います。

![](/images/github-actions-discord-notification-coco9122/img-0011.png)
*図1.11 Secretsの一覧*

上記の確認が取れましたら、先ほど設定したDisordのチャンネルに見に行ってみましょう。
図1.12のような画面になっていましたら通知が成功しています。

![](/images/github-actions-discord-notification-coco9122/img-0012.png)
*図1.12 Secretsの一覧*

# おわりに

以上で、プルリクエスト時にDisordの通知をするgithub actionsを実装しました。

---

何かあればコメントいただければと思います。