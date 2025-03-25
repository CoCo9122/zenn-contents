---
title: "ClaudeでPlaywright MCPを使う（Windows）"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["mcp", "playwright", "claude"]
published: true
---

# はじめに

WindowsでClaude Desktopを使ってPlaywright MCPを使う際に少しハマったので、その解決方法をまとめます。
Playwright MCPについての詳細は他の記事を参考にしてくださると幸いです。

# Claude Desktopのインストール

まずはClaude Desktopをインストールします。
https://claude.ai/download

Claude Desktopをインストールしたら、起動します。アカウントが必要なので適宜登録してください。

# Playwright MCPの準備

初めにPlaywright MCPを使うための準備をします。

## リポジトリのクローン

まずはPlaywright MCPのリポジトリをクローンします。ついでに、リポジトリに移動します。
```bash
git clone git@github.com:microsoft/playwright-mcp.git

cd playwright-mcp
```

## パッケージのインストール

次に、Playwright MCPのパッケージをインストールします。
```bash
npm install
```

## Google Chromeの設定

すでにGoogle Chromeがインストールされている方が多いと思いますので、Pathの設定を行います。

まずはTypeScriptの型定義の定義を変更します。
```diff ts: src/server.ts
// コードの一部を抜粋

export type LaunchOptions = {
  headless?: boolean;
+ executablePath?: string;
};

```

それに伴ってLaunchOptionsに`executablePath`を追加します。

```diff ts: src/program.ts
// コードの一部を抜粋

program
    .version('Version ' + packageJSON.version)
    .name(packageJSON.name)
    .option('--headless', 'Run browser in headless mode, headed by default')
    .option('--vision', 'Run server that uses screenshots (Aria snapshots are used by default)')
    .action(async options => {
      const launchOptions: LaunchOptions = {
        headless: !!options.headless,
+       // 自身のchrome.exeのパスを指定してください
+       executablePath: "C:/Program Files/Google/Chrome/Application/chrome.exe"
      };
      const tools = options.vision ? screenshotTools : snapshotTools;
      const server = new Server({
        name: 'Playwright',
        version: packageJSON.version,
        tools,
        resources,
      }, launchOptions);
      setupExitWatchdog(server);
      await server.start();
    });

```

これで、Google Chromeのパスを指定することができるようになります。

# MCPの設定

次に、MCPの設定を行います。
Claude Desktopを開いて、MCPの設定を行います。

Claude Desktopの右上のハンバーガーメニューをクリックします。
![](/images/claude-playwright-mcp-coco9122/img-0001.png)

ファイル > 設定をクリックします。
![](/images/claude-playwright-mcp-coco9122/img-0002.png)

`開発者`タブを選択して、構成の編集をクリックします。
![](/images/claude-playwright-mcp-coco9122/img-0003.png)

ディレクトリが開かれるので、`claude_desktop_config.json`を開きます。
そして以下のように設定を追加します。
```json:claude_desktop_config.json
{
    "mcpServers": {
        "playwright": {
            "command": "npx",
            "args": [
                "D:/Github/playwright-mcp/", // ここにCloneした自身のplaywright-mcpのパスを指定してください
                "--vision" //今回はUIが観たいのでVisionを有効
            ]
        }
    }
}
```

設定が完了したら、Claude Desktopを再起動します。

# 実行

実際に動作確認してみましょう。
![](/images/claude-playwright-mcp-coco9122/img-0004.png)

右下にあるハンマーアイコンをクリックして以下が表示されたらMCPが読み込めています。
![](/images/claude-playwright-mcp-coco9122/img-0006.png)


今回はせっかくになのでZennさん主催の第２回 AI Agent Hackathon with Google Cloudのページを見てもらうことにします。

プロンプトは以下のようにしました。
```txt
browser_screenshotを用いて以下のURLのスクリーンショットをとってきてください
https://zenn.dev/hackathons/google-cloud-japan-ai-hackathon-vol2
```

結果としては以下のようになりました。
![](/images/claude-playwright-mcp-coco9122/img-0005.png)

ちゃんとスクリーンショットが取得できていることが確認できました。

# おわりに
今回は、Windows環境でClaude Desktopを使いながらPlaywright MCPを設定・実行する方法を解説しました。Google Chromeのパス設定やMCPの構成変更など、いくつかの手順を踏む必要がありますが、無事に動作確認ができ、スクリーンショットも取得できました。