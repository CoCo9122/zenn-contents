---
title: "flaskでHTTPリクエストを受け取る"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python"]
published: false
---

# HTTPリクエストとは

以下のサイトを参考に簡単に説明します。

https://developer.mozilla.org/ja/docs/Web/HTTP/Methods


# 今回の目的

flaskのアプリケーションのhttpリクエストしよう。

# 使用するもの

dockerを使用します。

dockerについて知りたい方は以下を参照してください。大変参考になります。
https://zenn.dev/suzuki_hoge/books/2022-03-docker-practice-8ae36c33424b59

https://www.amazon.co.jp/%E4%BB%95%E7%B5%84%E3%81%BF%E3%81%A8%E4%BD%BF%E3%81%84%E6%96%B9%E3%81%8C%E3%82%8F%E3%81%8B%E3%82%8B-Docker-Kubernetes%E3%81%AE%E3%81%8D%E3%81%BB%E3%82%93%E3%81%AE%E3%81%8D%E3%81%BB%E3%82%93-%E5%B0%8F%E7%AC%A0%E5%8E%9F%E7%A8%AE%E9%AB%98/dp/4839972745/ref=sr_1_5?__mk_ja_JP=%E3%82%AB%E3%82%BF%E3%82%AB%E3%83%8A&crid=2GFHHLB073ZNT&keywords=docker&qid=1675943776&sprefix=docker%2Caps%2C347&sr=8-5

postmanを使用します。

httpリクエストを送信できるアプリケーションです。以下からダウンロードできます。
https://www.postman.com/

:::message
flaskの基本は以下の記事に作成しています。
https://zenn.dev/coco9122/articles/flask-startup-coco9122
:::

# プロジェクトフォルダーの作成

まずは、フォルダー構成の説明になります。先に必要なファイルをすべて作ってしまいます。

任意のフォルダに`flask-request`と名のフォルダーを作成します。

その配下に`docker-compose.yaml`のファイルと`app`、`docker`のフォルダを作成します。

`docker`フォルダの配下に`Dockerfile`のファイルを、`app`フォルダの配下に`main.py`のファイルを作成します。

最終的には以下のフォルダ構成になっています。

```
.flask-request/
 ├── docker
 │   └── Dockerfile
 ├── app
 │   └── main.py
 └── docker-compose.yaml
```

# Dockerfileの作成

Dockerfileを作成してきます。

```dockerfile
FROM python:3.9-slim-buster

WORKDIR /usr/src/app

RUN pip install --upgrade pip
RUN pip install flask
```

コンテナの中に入って、pythonを実行するのでコマンドは書いていません。

# docker-compose.yamlの作成

docker-compose.yamlを作成していきます。

```yaml
version: "3"
services:
  flask:
    build:
      context: .
      dockerfile: ./docker/Dockerfile
    restart: always
    tty: true
    ports:
    - 8080:8080
    volumes:
    - type: bind
      source: ./app
      target: /usr/src/app
```

# main.pyの作成

