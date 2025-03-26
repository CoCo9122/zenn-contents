---
title: "MCPを活用して社内システムをClaudeから操作する"
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["mcp", "claude", "python"]
published: false
---

# はじめに

みなさん、MCPサーバーで遊んでいますでしょうか。すでにOSS用のMCPサーバや主要なサービスはMCPサーバを提供していますね。以下にMCPサーバのリストの一種ですが提示しておきます。
https://github.com/modelcontextprotocol/servers

ところで、主要なサービスにはMCPサーバが提供されていますが、社内のシステムにはまだMCPサーバが提供されていないことが多いと思います。そこで、今回は社内のシステムにMCPサーバを導入して、Claudeから社内のシステムを操作する方法を紹介します。

:::message
MCPサーバについての詳細は他の記事を参考にしてくださると幸いです。
:::

# 社内のシステム用のMCPの実装

実際に社内のシステム用のMCPサーバを実装するために、今回はPythonでMCPサーバを実装します。

## MCPサーバの実装に向けて

まずは社内のシステムに向けて、必要になってくるのがAPIになります。社内のシステムにAPIが提供されてない場合は、社内のシステム開発者にAPIを提供してくださいと依頼しましょう。

そのため、今回は社内のシステムにAPIが提供されていることを前提に進めます。

今回はホットペッパーグルメのAPIを例として、MCPサーバを実装します。ホットペッパーのAPIは以下のURLから取得できます。

:::message alert
[ホットペッパーグルメのAPI](https://webservice.recruit.co.jp/)は無料で利用できますが、APIキーが必要です。使用する際にはプライバシーポリシー及びリクルートWEBサービス 利用規約をご確認ください。
:::

## APIリファレンスの確認

ホットペッパーグルメのAPIのリファレンスを確認して、MCPサーバの実装に必要なエンドポイントを確認します。
https://webservice.recruit.co.jp/doc/hotpepper/reference.html

ホットペッパーのAPIには、様々なクエリパラメータが用意されていますが、今回は以下のパラメータを使ってMCPサーバを実装します。

| パラメータ | 項目名 | 説明 |
| --- | --- | --- |
| key | APIキー | リクルートWEBサービスのAPIキー |
| format | レスポンス形式 | レスポンス形式を指定します。jsonまたはxmlを指定できます。 |
| keyword | キーワード | 検索キーワードを指定します。 |
| name | 店舗名 | 店舗名を指定します。 |

次にリスポンスを確認します。リファレンスによると以下のようなレスポンスが返ってくるようです。

```json
{
  "results": {
    "api_version": "1.26",
    "results_available": 1,
    "results_returned": "1",
    "results_start": 1,
    "shop": [
      {
        "id": "J000000000",
        "name": "店舗名",
        "address": "住所",
        "urls": {
          "pc": "https://...",
          "mobile": "https://..."
        },
      }
    ]
  }
}
```

:::message
一部抜粋しています。
:::


実際に実装する際は、社内のシステムリファレンスを確認してください。


## MCPサーバの実装

MCPサーバの実装には、PythonのMCPライブラリを使います。MCPライブラリは以下のURLから取得できます。

https://pypi.org/project/mcp/

各自の環境に合わせてインストールしてください。（私はconda環境で実施します。）

```bash
pip install mcp mcp[cli]
```

MCPライブラリをインストールしたら、MCPサーバを実装します。

まずはライブラリのインポートから始めます。
```python: main.py
import os
import asyncio
import requests

from typing import Any, List
from mcp.server import Server
from mcp.types import Tool, TextContent
from mcp.server.stdio import stdio_server
```

次に、APIキーを取得します。APIキーは環境変数から取得します。
```python: main.py
# トークンの取得
HOTPEPPER_API_KEY = os.getenv("HOTPEPPER_API_KEY")
if not HOTPEPPER_API_KEY:
    raise ValueError("HOTPEPPER_API_KEY environment variable is required")
```

次に、MCPサーバを作成します。
```python: main.py
# MCPサーバーの作成 
mcp = Server("HOTPEPPER-Server")
```

次に、レスポンスを整形する関数を作成します。これは先ほど確認したレスポンスを整形する関数です。今回は店の名前、住所、最寄り駅、メニューURLをClaudeに返却します。
```python: main.py
def formatted_response(shops: Any) -> str:
    return [TextContent(
        type="text", 
        text=f"Results: {len(shops)} shops found:\n\n" +
            "\n".join([f"Shop Name: {shop['name']}, Address: {shop['address']}, Station: {shop['station_name']}, URL: {shop['coupon_urls']['pc'].replace("map", "food")}\n" for shop in shops])
        )
    ]
```

次に、MCPサーバにツールを登録します。今回は店名、最寄り駅、キーワードで検索するツールを登録します。Toolクラスの引数には、ツール名、ツールの説明、入力スキーマを指定します。入力スキーマは、Claudeからツールを呼び出す際に必要なパラメータを指定します。

```python: main.py
@mcp.list_tools()
async def list_tools() -> List[Tool]:
    """List available HOTPEPPER tools."""
    return [
        Tool(
            name="search_shops_by_name",
            description="Search shops by name",
            inputSchema={
                "type": "object",
                "properties": {
                    "name": {"type": "string"},
                    "count": {
                        "type": "number",
                        "description": "Maximum number of shops to fetch",
                        "minimum": 1,
                        "maximum": 100
                    }
                },
                "required": ["name"],
            }
        ),
        Tool(
            name="search_shops_by_station",
            description="Search shops by station",
            inputSchema={
                "type": "object",
                "properties": {
                    "station": {"type": "string"},
                    "count": {
                        "type": "number",
                        "description": "Maximum number of shops to fetch",
                        "minimum": 1,
                        "maximum": 100
                    }
                },
                "required": ["station"],
            }
        ),
        Tool(
            name="search_shops_by_keyword",
            description="Search shops by keyword",
            inputSchema={
                "type": "object",
                "properties": {
                    "keyword": {"type": "string"},
                    "count": {
                        "type": "number",
                        "description": "Maximum number of shops to fetch",
                        "minimum": 1,
                        "maximum": 100
                    }
                },
                "required": ["keyword"],
            }
        )
    ]
```

次に、ツールを呼び出された際に実行される関数を実装します。今回は店名、最寄り駅、キーワードで検索する関数を実装します。クエリパラメータを設定して、リクエストを送信し、レスポンスを整形して返却します。nameでツール名を判別して、処理を分岐させます。

```python: main.py
@mcp.call_tool()
async def call_tool(name: str, arguments: Any) -> List[TextContent]:

    REQUEST_URL = "http://webservice.recruit.co.jp/hotpepper/gourmet/v1/"

    """Handle HOTPEPPER tool calls."""
    if name == "search_shops_by_name":

        # リクエストヘッダの設定
        headers = {
            "Content-Type": "application/json"
        }

        # リクエストパラメータの設定
        query = {
            "key": HOTPEPPER_API_KEY,
            "name": arguments["name"],
            "count": arguments.get("count", 10),
            "format": "json"
        }

        # リクエストの送信
        response = requests.get(REQUEST_URL, headers=headers, params=query)

        # レスポンスの取得
        data = response.json()

        # レスポンスの処理
        shops = data["results"]["shop"]
        return formatted_response(shops)
    elif name == "search_shops_by_station":
        # リクエストヘッダの設定
        headers = {
            "Content-Type": "application/json"
        }

        # リクエストパラメータの設定
        query = {
            "key": HOTPEPPER_API_KEY,
            "keyword": arguments["station"],
            "count": arguments.get("count", 10),
            "format": "json"
        }

        # リクエストの送信
        response = requests.get(REQUEST_URL, headers=headers, params=query)

        # レスポンスの取得
        data = response.json()

        # レスポンスの処理
        shops = data["results"]["shop"]
        return formatted_response(shops)
    
    elif name == "search_shops_by_keyword":

        # リクエストヘッダの設定
        headers = {
            "Content-Type": "application/json"
        }

        # リクエストパラメータの設定
        query = {
            "key": HOTPEPPER_API_KEY,
            "keyword": arguments["keyword"],
            "count": arguments.get("count", 10),
            "format": "json
        }
    
        # リクエストの送信
        response = requests.get(REQUEST_URL, headers=headers, params=query)

        # レスポンスの取得
        data = response.json()

        # レスポンスの処理
        shops = data["results"]["shop"]
        return formatted_response(shops)
    
    else:
        raise ValueError(f"Unknown tool name: {name}")
```

最後に、MCPサーバを起動します。stdio_server()関数を使って、標準入出力を使ってMCPサーバを起動します。
```python: main.py
async def main():
    async with stdio_server() as (read_stream, write_stream):
        await mcp.run(
            read_stream, 
            write_stream,
            mcp.create_initialization_options()
        )

if __name__ == "__main__":
    asyncio.run(main())
```

上記にてMCPサーバの実装は完了です。次に、デスクトップ版のClaudeにMCPサーバを組み込みます。

以下に実装の全容を示します。
:::: details コードの全容
```python: main.py
import os
import asyncio
import requests

from typing import Any, List
from mcp.server import Server
from mcp.types import Tool, TextContent
from mcp.server.stdio import stdio_server

# トークンの取得
HOTPEPPER_API_KEY = os.getenv("HOTPEPPER_API_KEY")
if not HOTPEPPER_API_KEY:
    raise ValueError("HOTPEPPER_API_KEY environment variable is required")

# MCPサーバーの作成
mcp = Server("HOTPEPPER-Server")

def formatted_response(shops: Any) -> str:
    return [TextContent(
        type="text", 
        text=f"Results: {len(shops)} shops found:\n\n" +
            "\n".join([f"Shop Name: {shop['name']}, Address: {shop['address']}, Station: {shop['station_name']}, URL: {shop['coupon_urls']['pc'].replace("map", "food")}\n" for shop in shops])
        )
    ]


@mcp.list_tools()
async def list_tools() -> List[Tool]:
    """List available HOTPEPPER tools."""
    return [
        Tool(
            name="search_shops_by_name",
            description="Search shops by name",
            inputSchema={
                "type": "object",
                "properties": {
                    "name": {"type": "string"},
                    "count": {
                        "type": "number",
                        "description": "Maximum number of shops to fetch",
                        "minimum": 1,
                        "maximum": 100
                    }
                },
                "required": ["name"],
            }
        ),
        Tool(
            name="search_shops_by_station",
            description="Search shops by station",
            inputSchema={
                "type": "object",
                "properties": {
                    "station": {"type": "string"},
                    "count": {
                        "type": "number",
                        "description": "Maximum number of shops to fetch",
                        "minimum": 1,
                        "maximum": 100
                    }
                },
                "required": ["station"],
            }
        ),
        Tool(
            name="search_shops_by_keyword",
            description="Search shops by keyword",
            inputSchema={
                "type": "object",
                "properties": {
                    "keyword": {"type": "string"},
                    "count": {
                        "type": "number",
                        "description": "Maximum number of shops to fetch",
                        "minimum": 1,
                        "maximum": 100
                    }
                },
                "required": ["keyword"],
            }
        )
    ]

@mcp.call_tool()
async def call_tool(name: str, arguments: Any) -> List[TextContent]:

    REQUEST_URL = "http://webservice.recruit.co.jp/hotpepper/gourmet/v1/"

    """Handle HOTPEPPER tool calls."""
    if name == "search_shops_by_name":

        # リクエストヘッダの設定
        headers = {
            "Content-Type": "application/json"
        }

        # リクエストパラメータの設定
        query = {
            "key": HOTPEPPER_API_KEY,
            "name": arguments["name"],
            "count": arguments.get("count", 10),
            "format": "json"
        }

        # リクエストの送信
        response = requests.get(REQUEST_URL, headers=headers, params=query)

        # レスポンスの取得
        data = response.json()

        # レスポンスの処理
        shops = data["results"]["shop"]
        return formatted_response(shops)
    elif name == "search_shops_by_station":
        # リクエストヘッダの設定
        headers = {
            "Content-Type": "application/json"
        }

        # リクエストパラメータの設定
        query = {
            "key": HOTPEPPER_API_KEY,
            "keyword": arguments["station"],
            "count": arguments.get("count", 10),
            "format": "json"
        }

        # リクエストの送信
        response = requests.get(REQUEST_URL, headers=headers, params=query)

        # レスポンスの取得
        data = response.json()

        # レスポンスの処理
        shops = data["results"]["shop"]
        return formatted_response(shops)
    
    elif name == "search_shops_by_keyword":

        # リクエストヘッダの設定
        headers = {
            "Content-Type": "application/json"
        }

        # リクエストパラメータの設定
        query = {
            "key": HOTPEPPER_API_KEY,
            "keyword": arguments["keyword"],
            "count": arguments.get("count", 10),
            "format": "json"
        }

        # リクエストの送信
        response = requests.get(REQUEST_URL, headers=headers, params=query)

        # レスポンスの取得
        data = response.json()

        # レスポンスの処理
        shops = data["results"]["shop"]
        return formatted_response(shops)
    
    else:
        raise ValueError(f"Unknown tool name: {name}")
    

async def main():
    async with stdio_server() as (read_stream, write_stream):
        await mcp.run(
            read_stream, 
            write_stream,
            mcp.create_initialization_options()
        )

if __name__ == "__main__":
    asyncio.run(main())
```
::::

# デスクトップ版のClaudeにMCPを組み込む

MCPサーバを実装したら、次にデスクトップ版のClaudeにMCPサーバを組み込みます。
Claude Desktopを開いて、MCPの設定を行います。

Claude Desktopの右上のハンバーガーメニューをクリックします。
![](/images/mcp-server-coco9122/img-0001.png)

ファイル > 設定をクリックします。
![](/images/mcp-server-coco9122/img-0002.png)

`開発者`タブを選択して、構成の編集をクリックします。
![](/images/mcp-server-coco9122/img-0003.png)

ディレクトリが開かれるので、`claude_desktop_config.json`を開きます。
そして以下のように設定を追加します。

```json:claude_desktop_config.json
{
    "mcpServers": {
        "hotpepper": {
            "command": "<conda.exeの絶対パス>",
            "args": [
                "run",
                "-n",
                "<仮想環境名>",
                "--no-capture-output",
                "python",
                "<main.pyの絶対パス>"
            ],
            "env": {
                "HOTPEPPER_API_KEY": "<hotpepper_api_key>"
            }
        }
    }
}
```

:::message
conda以外の仮想環境を使用している場合は、適宜コマンドを変更してください。
:::

設定が完了したら、Claude Desktopを再起動します。

# 動作確認

Claude Desktopを開いて、MCPサーバを呼び出してみます。右のハンマーアイコンをクリックして、MCPサーバを選択します。
![](/images/mcp-server-coco9122/img-0004.png)


下記のように作成したツールが表示されていれば、MCPサーバの設定が成功しています。
![](/images/mcp-server-coco9122/img-0005.png)


今回は、浜松町駅周辺の店舗を検索してみます。以下のプロンプトを入力します。
```txt
HotPepperを用いて浜松町駅周辺の店舗を5件検索してください。
```

以下のように店舗情報が表示されれば成功です。search_shops_by_stationツールが呼び出されて、店舗情報が取得されていることがわかります。
![](/images/mcp-server-coco9122/img-0006.png)

このように、MCPサーバを活用することで、社内のシステム（今回はホットペッパーを例にしました）をClaudeから操作することが可能です。

# おわりに

今回は、社内のシステムにMCPサーバを導入して、Claudeから社内のシステムを操作する方法を紹介しました。MCPサーバを使うことで、社内のシステムをClaudeから操作することができます。ぜひ、MCPサーバを作成してみてください。