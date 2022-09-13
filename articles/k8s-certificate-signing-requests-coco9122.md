---
title: "自宅で構築したk8sに別PCからkubectlをたたく！"
emoji: "👋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["kubernetes"]
published: false
---

# kubectlたたきたい

自宅でオンプレのk8s環境の作成を行う場合、直接Linuxのターミナルか別PCからLinuxにSSH接続してkubectlをたたくと思います。これを別PCからSSH接続せずにkubectlをたたくというのが今回の目的です。

# 環境

- master:1, worker:1のシンプルなk8s環境
- k8s version: v1.24.3
:::message
オンプレですでにk8s環境は構築済みでの話となります。
:::

---

## 使用される仕組み

- Certificate Signing Requests
- RBAC Authorization
- kubeconfig

以上の仕組みを少しだけ解説します。

---

### Certificate Signing Requests(CSR)/kubeconfig

公式サイトに以下のように説明されています。その下が和訳になります。

https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/

> The Certificates API enables automation of X.509 credential provisioning by providing a programmatic interface for clients of the Kubernetes API to request and obtain X.509 certificates from a Certificate Authority (CA).
> A CertificateSigningRequest (CSR) resource is used to request that a certificate be signed by a denoted signer, after which the request may be approved or denied before finally being signed

> Certificates APIは、Kubernetes APIのクライアントが認証局（CA）にX.509証明書を要求して取得するためのプログラム的インターフェイスを提供することで、X.509クレデンシャル・プロビジョニングの自動化を可能にします。
> CertificateSigningRequest (CSR) リソースは、証明書が指定された署名者によって署名されることを要求するために使用され、その後、要求は最終的に署名される前に承認または拒否される可能性があります。

簡単に言うとkube-apiserverにアクセスする際に認証が必要ということになります。

例えば、PC1でたかし君というユーザーでkube-apiserverにリクエストを行った際に、まずは本当にたかし君なのかの認証を行い、たかし君だと確定できたら、次にこのリクエストを許可するかしないかの認可を行う形になります。

```mermaid
sequenceDiagram
    PC1->>kubeapiserver: たかしというユーザーでpodのリストの確認してもいいですか？ 
    Note over PC1,kubeapiserver: kubectl get pods
    kubeapiserver->>PC1: たかし君本人だということの確認が取れました。
    kubeapiserver->>PC1: たかし君としてこのリクエストは実行可能です。
```

このなかでも「**認証**」機能を実装しているのがCertificateSigningRequest (CSR)とkubeconfigになります。実際に行う際に詳しく説明します。

### RBAC Authorization

https://kubernetes.io/docs/reference/access-authn-authz/rbac/

> Role-based access control (RBAC) is a method of regulating access to computer or network resources based on the roles of individual users within your organization.

> ロールベースアクセスコントロール（RBAC）とは、組織内の個々のユーザーの役割に基づいて、コンピューターやネットワークリソースへのアクセスを規制する手法のことである。

簡単に言うと、ユーザーに与えられる権利を規制することができる機能になります。


例えば、PC1でたかし君というユーザーはPodとDeploymentの作成のみ許可されているとします。そうするとPodを作成したいというリクエストを行うことができます。

```mermaid
sequenceDiagram
    PC1->>kubeapiserver: たかしというユーザーでPodを作成してもいいですか？ 
    Note over PC1,kubeapiserver: kubectl run nginx --image=nginx
    kubeapiserver->>PC1: たかし君本人だということの確認が取れました。
    kubeapiserver->>PC1: たかし君としてこのリクエストは実行可能です。
    Note over kubeapiserver,PC1: kubectl run nginx --image=nginxを実行しました
```

一方で、先ほど作成したPodを削除したいというリクエストとなると許可されていません。
```mermaid
sequenceDiagram
    PC1->>kubeapiserver: たかしというユーザーで先ほど作成したPodを削除してもいいですか？ 
    Note over PC1,kubeapiserver: kubectl delete pods nginx
    kubeapiserver->>PC1: たかし君本人だということの確認が取れました。
    kubeapiserver--XPC1: たかし君としてこのリクエストは実行不可です。
```

したがって、「**認可**」機能を実装しているのがRBACになります。実際に行う際に詳しく説明します。


以上の仕組みを使用し実装していきます。

# 実装したい目標

PC1を使用しているたかし君から以下のことを行いたいと要望がありました。

「PC1からDeploymentとServiceの管理したい」

とのことでした。というていでやっていきます。

今回はnamespaceに「takashi」を作成し、ここで以下のコマンドを打てるように実装します。これでたかしくんの要望は満たせると思います。

- Windows(PC1)から「takashi」というユーザーでk8s環境にnginx:1.16のDeploymentを作成
- Windows(PC1)から「takashi」というユーザーでk8s環境のDeploymentの一覧を確認
- Windows(PC1)から「takashi」というユーザーでk8s環境にService(NodePort)を作成し、Deploymentの公開
- Windows(PC1)から「takashi」というユーザーでk8s環境のServiceの一覧を確認
- Windows(PC1)からnginxのスタートページの閲覧
- Windows(PC1)から「takashi」というユーザーでk8s環境にある作成したDeploymentとServiceの削除

:::message
適当に決めたので、適宜行いたいロールに合わせて変更してください。
:::


# 流れ

## 今回行う流れ 

```mermaid
graph TB
    subgraph k8s環境
    1.秘密鍵,証明書署名要求の作成 --> 2.CSRの作成
    2.CSRの作成 --> 3.CSRの承認
    3.CSRの承認 --> 4.証明書の取得
    4.証明書の取得 --> 9.role,rlebindingの作成
    end
    4.証明書の取得 -.kubectlをたたきたいPCにコピー.-> 6.証明書とキーファイルをPCにコピー
    subgraph kubectlをたたきたいPC
    5.PCにkubectlのイントール --> 6.証明書とキーファイルをPCにコピー
    6.証明書とキーファイルをPCにコピー　--> 7.証明書とキーファイルをもとにkubeconfigにuserを設定
    6.証明書とキーファイルをPCにコピー　--> 8.kubeconfigにcluster,contextを設定
    end
    7.証明書とキーファイルをもとにkubeconfigにuserを設定 -.kubeconfigに設定したuserを使用.-> 9.role,rlebindingの作成
```

詳しい流れはこのリンクを参照下さい。
https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/

::::details 1. 秘密鍵,証明書署名要求の作成

CSRはSSL認証を使用しています。詳しくはご自身での勉強をお願いします（調べればすぐ出てくると思います）。

- まずはk8s環境の入っているマスターノードにSSH接続か直接ターミナルの移動をお願いします。
```sh
ssh <Linuxユーザー名>@<プライベートIPアドレス>
```

:::message
Linuxの操作になります。
:::

- 秘密鍵を保存するディレクトリの作成
```sh
mkdir takashi
```

- ディレクトリの移動
```sh
cd takashi
```

- 秘密鍵の作成
```sh
openssl genrsa -out takashi.key 2048
```

- 証明書署名要求(CSR)の作成
```sh
openssl req -new -days 3650 -key takashi.key -out takashi.csr
```
:::message
色々聞かれますがすべてEnterキーを押していれば大丈夫です。
:::

- 確認
```sh
ls
> takashi.csr  takashi.key
```
lsコマンドで以上の二つがあれば問題ありません。
::::

::::details 2. CSRの作成

- 先ほどと同じディレクトリに以下のようなyamlファイルを作成
```sh
vi takashi.yaml
```

```yaml:takashi.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: takashi
spec:
  request: 
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
```

- 作成したtakashi.csrのデコード
```sh
cat takashi.csr | base64 | tr -d "\n"
```
長い文字列がが表示されますがこれをコピーします。これを先ほど作成したyamlファイルのrequestの箇所にコピーします。

```diff yaml:takashi.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: takashi
spec:
- request:
+ request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ2lqQ0NBWElDQVFBd1JURUxNQWtHQTFVRUJoTUNRVlV4RXpBUkJnTlZCQWdNQ2xOdmJXVXRVM1JoZEdVeApJVEFmQmdOVkJBb01HRWx1ZEdWeWJtVjBJRmRwWkdkcGRITWdVSFI1SUV4MFpEQ0NBU0l3RFFZSktvWklodmNOCkFRRUJCUUFEZ2dFUEFEQ0NBUW9DZ2dFQkFMNWRocXNGNmNyNUs3OWJobHVzS0Jwc25EcmF4YkhnNkJCY3AvWDQKSDUvc1k1T1dsVmJBdGxWeE13d0xZV3l3N3dxQXA5TjZyMXpLUkVwRXRYWnNkQU9DNGE2WUwxMlhSeW9RMkE2Ywp4UDhwMEhXVEt1S044TEhDNDJna3kwcDlpM2FhYXlTRkRNbnJrM0hPMngwN2ppZ0RDa2UwbnlXOFlESjYzcEJpClZZeVk0SHZFdGYvOTdVNEtLS21oU3hrZXNQMUlYOWxLazRpZFB6M2lQYjE2SmJBa21aLzdueXRRUVFnMzdpWlAKTGlqc1NzZ2Rha3U5cDArRVJGRVdTMmt4SlBJWFA5V0lHQVl0bzcxcEZkVndDWlp6M1NmcHZUVk5udTFLWTlvVwpXL1dlNm4xMUZKYm9uNGpIWW1Rc2gydEZ1cWhLVGs4VktEdkVaWGpUNUYvYmw3TUNBd0VBQWFBQU1BMEdDU3FHClNJYjNEUUVCQ3dVQUE0SUJBUUJveExaR3RsQ09OQzFrcTJJcXAxSElqWDJNKzZxVWcySkJkRERxdVlNcElXQmcKc1J2aDR1b0tHSkdVRkJyMUhveE02WDZVazk3cTk0eFE5dk42ZURnYk55R2dNQmdQRzdKdG5uNlpRYnpLbG9HbApkUFRhdnI1bzNFSW9uVjF4U0tlUFMyWnBNcW1Rc3NaczNUaU5HdTFxL09NOHNxWTIycElqWmphZ0hVOW9YYTlMCkExVi9JZzdlSGt2UkFXUHphM1pCVGdCcXA3TkEwNU8vczd5SkNseFg0enV5SjFwd0xBaHBKVFpwbXE4Sk92YVAKUWJoRksvUVREVDFMNUVZQkV3MVNnaGNBVFJKeXBlREdQSStHSmxuWUg4UjExQXJ4NXlvZVU3WEYvb0Nlam4yNgpCdnNxQkVCLzh2TDdiYWdWdTFGc2xUNFAyZis4UzBaSWhXSS9saU1CCi0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
```

- CSRのアプライ
```sh
kubectl apply -f takashi.yaml
> certificatesigningrequest.certificates.k8s.io/takashi created
```

::::

:::: details 3. CSRの承認

CSRの確認をしましょう。
```sh
kubectl get csr
```
以上のコマンドを打つとたかし君が追加されていると思います。しかしCONDITIONはPENDINGなのでまだ承認されていません。以下のコマンドを打ちます。

```sh
kubectl certificate approve takashi
```

再びCSRの確認を行うとたかし君のCONDITIONはApprovedになっていると思います。これで承認は完了です。

::::

:::: details 4. 証明書の取得

```sh
kubectl get csr takashi -o yaml
```
このコマンドによりtakashiのyamlファイルを見ることができます。

この中のstatus.certificateにある文字列が認証書のコードになります。

したがってjsonpathで抽出し、crtファイルで保存します。
```sh
kubectl get csr takashi -o jsonpath='{.status.certificate}'| base64 -d > takashi.crt
```

確認を行います。

```sh
ls
> takashi.crt  takashi.key takashi.csr  takashi.yaml
```
lsコマンドで以下の4つがあれば大丈夫です。
::::

:::: details 5. PCにkubectlのイントール

以下のリンクが参考になります。
https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/#install-kubectl-binary-with-curl-on-windows

- SSH接続を切断

```sh
exit
```

:::message alert
Windows(kubectlをたたきたいPC)の操作になります。
:::

- 任意の場所にフォルダーを作成します
```sh
mkdir takashi
```
- フォルダ下に移動
```sh
cd takashi
```

- curlを用いてkubectlをインストール
```sh
curl -LO "https://dl.k8s.io/release/v1.24.3/bin/windows/amd64/kubectl.exe"
```

- ダウロードしたファイルを環境変数のPathに登録

以下が参考になります。

https://proengineer.internous.co.jp/content/columnfeature/5205

:::message
ダウンロードした先のフォルダーの絶対パスを指定します。いまはtakashiフォルダにインストールされていると思います。
:::

- kubectlの実行可能か確認
```sh
kubectl version --client
```
versionがv1.24.3と表示されれば大丈夫です。
::::

:::: details 6. kubectlをたたきたいPCにコピー

:::message alert
引き続きWindows(kubectlをたたきたいPC)の操作になります。
:::

- keyファイルとcrtファイルをWindowsのtakashiフォルダーにコピー
```sh
scp <Linuxユーザー名>@<プライベートIPアドレス>:~/takashi/takashi.key
scp <Linuxユーザー名>@<プライベートIPアドレス>:~/takashi/takashi.crt
```

- dirコマンドで確認

```sh
dir
```

kubectl.exeとtakashi.keyとtakashi.crtファイルがあれば大丈夫です。

::::

:::: details 7. 証明書とキーファイルをもとにkubeconfigにuserを設定

:::message alert
引き続きWindows(kubectlをたたきたいPC)の操作になります。
:::

- userの登録
```sh
kubectl config set-credentials takashi --client-key=takashi.key --client-certificate=takashi.crt --embed-certs=true
```
::::

:::: details 8. kubeconfigにcluster,contextを設定
:::message
Linuxの操作になります。
:::
::::

:::: details 9. role,rlebindingの作成

:::message
Linuxの操作になります。
:::

::::