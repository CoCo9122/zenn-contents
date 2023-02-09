---
title: "はじめてのFlask"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python","flask"]
published: false
---

# Flaskとは

FlaskはPythonのWebアプリケーションフレームワークです。とってもシンプルに記述できます。

気になる方は公式サイトを参照してください。
https://msiz07-flask-docs-ja.readthedocs.io/ja/latest/

# 今回の目的

http://localhost:8080にアクセスしたら"Hello World!"と表示できるようにしよう！

# 使用するもの

dockerを使用します。

dockerについて知りたい方は以下を参照してください。大変参考になります。
https://zenn.dev/suzuki_hoge/books/2022-03-docker-practice-8ae36c33424b59

https://www.amazon.co.jp/%E4%BB%95%E7%B5%84%E3%81%BF%E3%81%A8%E4%BD%BF%E3%81%84%E6%96%B9%E3%81%8C%E3%82%8F%E3%81%8B%E3%82%8B-Docker-Kubernetes%E3%81%AE%E3%81%8D%E3%81%BB%E3%82%93%E3%81%AE%E3%81%8D%E3%81%BB%E3%82%93-%E5%B0%8F%E7%AC%A0%E5%8E%9F%E7%A8%AE%E9%AB%98/dp/4839972745/ref=sr_1_5?__mk_ja_JP=%E3%82%AB%E3%82%BF%E3%82%AB%E3%83%8A&crid=2GFHHLB073ZNT&keywords=docker&qid=1675943776&sprefix=docker%2Caps%2C347&sr=8-5

# プロジェクトフォルダーの作成

まずは、フォルダー構成の説明になります。先に必要なファイルをすべて作ってしまいます。

任意のフォルダに`flask-startup`と名のフォルダーを作成します。

その配下に`docker-compose.yaml`のファイルと`app`、`docker`のフォルダを作成します。

`docker`フォルダの配下に`Dockerfile`のファイルを、`app`フォルダの配下に`main.py`のファイルを作成します。

最終的には以下のフォルダ構成になっています。

```
.flask-startup/
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

# docker-compose.yamlの作成

```yaml
version: "3"
services:
  flask:
    bluid:
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

公式サイトから引用しています。

```py
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello_world():
    return "<p>Hello World!</p>"

app.run(host='0.0.0.0', port=8080)
```

最後の一行だけ追加いたしました。これはポートの設定を追加しています。

# Webアプリケーションの起動

まずはコンテナの中に入っていきます。
```sh
docker exec -it /bin/bash
```

flaskのアプリを起動します。
```sh
python main.py
```

ここにアクセスすると`Hello World!`と表示されるはずです。

# Flaskのシンプルさ
環境などの設定が必要ですが、Pythonのコードは6行で完結しています。とてもシンプルです。また、コードからどのような挙動をしているかシンプルで分かりやすいのが特徴です。webアプリケーションの流れと実際のコードの流れを学びたい初学者にはよいwebアプリケーション（主にバックエンド）フレームワークになっていると思います。

有名なフレームワークであるDjangoなどありますが、初学者が始めるには少し閾値が高いです（その代わり本格的なwebアプリケーションが作れます）。webアプリケーションの理解の上でFlaskはおすすめです。


# まとめ

今回はFlaskの基本の基本を作成致しました。

# 参考文献
https://msiz07-flask-docs-ja.readthedocs.io/ja/latest/

# Github

リポジトリになります。

# 変更履歴
・2023年02月09日 投稿