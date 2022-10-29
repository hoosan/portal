---
title: Hello World
---

# 20分でできる "Hello World” スマートコントラクトのデプロイ方法

このチュートリアルは、"hello world“ Canister スマートコントラクトを最小限ですが、20分以内の高速で IC にデプロイする方法です。このチュートリアルを実行するために必要なのは、ターミナルの使用に関する基本的な知識だけです。コードの編集は必要ありません。

## イントロダクション

このチュートリアルでは、シンプルな `hello` Canister スマートコントラクトをデプロイします。このスマートコントラクトは `greet` という名前の関数を1つだけ持っています。この `greet` という関数はひとつのテキスト引数を受け取り、その結果を greeting と一緒に返します。例えば、`greet` メソッドに `Alice` というテキスト引数を与えて呼び出したとします。

* コマンドラインから呼び出すと、端末に`Hello, Alice!` と表示されます。

* ブラウザからアクセスすると、ポップアップウィンドウに`Hello, Alice!`と表示されます。

このアプリはすぐに使えるコードですが、以下のような構成になっています。このアプリは、Internet Computer(IC) と対話するために設計されたプログラミング言語である Motoko で書かれたバックエンドのコードと、シンプルな webpack ベースのフロントエンドで構成されています。

### このチュートリアルに必要なコンセプト

* **Canister スマートコントラクト：** *Canister スマートコントラクト*、略して *Canister* は、コードとステートをバンドルしたスマートコントラクトの一種です。Canister は IC に配備され、そこで実行され、インターネット経由でアクセスすることができます。*Dapp* は分散型アプリケーションで、1つ以上の Canister で構成されています。

* **Cycle：** *Cycle* は、処理、メモリ、ストレージ、ネットワーク帯域幅など、IC 上で消費されるリソースの測定単位です。各 Canister には Cycle アカウントがあり、Canister によって消費されたリソースはこのアカウントに送られます。IC のユーティリティ・トークン（ICP）を Cycle に変換し、Canister に転送することができます。ICP から Cycle への変換は、常に現在の ICP の価格 \[SDR\] で行うことができます。1兆 Cycle が 1SDR に相当します。

このチュートリアルでは、Cycle フォーセットから無料 Cycle を取得します。

## 対応 OS

このチュートリアルでは、次のいずれかのオペレーティングシステムが必要です。
-   Linux
-   macOS (12.\* Monterey)
-   Windows (With [WSL installed](windows-wsl))

## ツールのインストール（5分）

Canister を構築しデプロイするには、ユーザーは以下をインストールする必要があります：

### Node.js

node.js をインストールするには、様々な方法があります。Linux ではお使いのシステムのパッケージマネージャを使用することをお勧めします。macOS では、[Homebrew](https://brew.sh) をお勧めします。また、[nodejs.org website](https://nodejs.org/en/download) でも説明されています。

このチュートリアルは `16.*.*` 以降の node.js のバージョンで最も効果的に動作します。

### DFX

このチュートリアルでは、DFINITY 財団が管理しているデフォルトの SDK である`dfx` という Canister SDK を使用します。この SDK はたくさんある SDK の中の一つです。

インストールするには、以下を実行します：

    $ sh -ci "$(curl -fsSL https://smartcontracts.org/install.sh)"

`dfx` が正しくインストールされていることを確認するために、以下を実行します：

    $ dfx --version

ターミナルには最新版の SDK（[SDKリリースノート](../updates/release-notes/release-notes.md)）が表示されるはずです。

## プロジェクトを生成する（2分）

`dfx` プロジェクトは、ソースコードや設定ファイルを含むアーティファクトのセットであり、Canister にコンパイルすることができます。以下を実行します：

    $ dfx new hello

`dfx` は `hello` という名前の新しいプロジェクトディレクトリを作成します。ターミナルの出力は以下のようになるはずです。

![dfx new](_attachments/dfx-new-hello-1.png)

![dfx new](_attachments/dfx-new-hello-2.png)


"hello world” Canister と新しい `hello` Git リポジトリに必要なアーティファクトは、`hello` プロジェクトディレクトリにあります。ディレクトリはこのようになっているはずです：

![cd new](_attachments/cd-hello.png)


## Canister をローカル環境で実行する（3分）

これで `hello` プロジェクトが作成されましたので、次のステップではそれをローカルにデプロイします。ローカルにデプロイするために `dfx` は実行環境のローカルインスタンスを開始することができます。この環境は完全な IC レプリカではなく、IC レプリカのステートをダウンロードすることもありません。Dapps のデプロイ専用に設計された軽量な環境です。

このため、ターミナルを2つ使用することを推奨します：

-   **ターミナル A** には、実行環境の出力が表示されます。これは、node.js の Express、python の Django、Ruby の Rails など、web2 プロジェクトでローカルサーバを起動するのと似ています。

-   **ターミナル B** は、実行環境で動作している Canister と対話するために使用されます。これは、ローカルで動作しているサーバ、例えば Postman や Panic に HTTP API メッセージを送信することに類似しています。

このチュートリアルでは、2つのターミナルを区別するために、ターミナル A の背景を紺色にしています...

![dfx new](_attachments/dfx-new-hello-2.png)

... ターミナル B の背景は黒色です。

![terminal b ls](_attachments/terminal-b-ls.png)

### 実行環境を起動する

ターミナル A で、プロジェクトのルートディレクトリ `hello` に移動し、次のように実行します：

    $ dfx start

![dfx start](_attachments/terminal-a-dfx-start.png)

NOTE: お使いのプラットフォームやローカルのセキュリティ設定によっては、警告が表示される場合があります。受信したネットワーク接続の許可または拒否を求めるプロンプトが表示された場合は、［許可］をクリックします。

これで、お使いのマシンでローカル Canister 実行環境が実行されるようになりました。ターミナル A を開いたまま、作業を続けてください。もしターミナルを閉じると、実行環境は停止し、チュートリアルの残りの部分は失敗します。

### Canister をローカルにデプロイする

NOTE: 今回は Canister 実行環境のみなので、Cycle を必要とするメインネットへのデプロイメントに比べ、ステップ数が少なくなっています。

ターミナル B で、プロジェクトのルートディレクトリ `hello` に移動します。必要なすべての node モジュールをインストールするには、以下のコマンドを実行します：

    $ npm install

![npm install](_attachments/terminal-b-npm-install.png)

`hello` Canister を登録、ビルドし、ローカルの実行環境にデプロイします。以下のコマンドを実行します：

    $ dfx deploy

![dfx deploy](_attachments/terminal-b-dfx-deploy.png)

### コマンドラインからローカルの Dapp をテストする

これで、Canister がローカルの実行環境にデプロイされたので、メッセージを送受信して Canister と対話できるようになります。Canister には `greet` というメソッド （パラメータとして文字列を受け取ります）があるので、メッセージを送信してみましょう。ターミナル B で、以下を実行します。

    $ dfx canister call hello_backend greet everyone

-   `dfx canister call` コマンドでは、Canister 名と呼び出す関数を指定する必要があります。
-   `hello_backend` は呼び出す Canister の名前を指定します。
-   `greet` は関数名を指定します。
-   `everyone` は `greet` 関数に渡す引数です。

### ブラウザでローカルの Dapp をテストする

Dapp がデプロイされ、コマンドラインで動作確認ができたので、Web ブラウザでフロントエンドにアクセスできることを確認しましょう。

ターミナル B で、開発用サーバーを起動します：

    $ npm start

ブラウザを開き、<http://localhost:8080/> に移動してください。

シンプルな HTML ページが表示され、サンプルアセット、画像ファイル、入力欄、ボタンを確認できます。

![Sample HTML page](_attachments/frontend-prompt.png)

greeting に入力し、**Click Me** をクリックすると、greeting が返されます。

![Hello](_attachments/frontend-result.png)

### ローカル Canister の実行環境を停止する

ブラウザでアプリケーションをテストした後、ローカル Canister の実行環境を停止して、バックグラウンドで実行し続けないようにします。オンチェーン・デプロイには、この実行環境は必要ありません。

ローカル・デプロイメントを停止するには、以下の手順に従います：

1.  ターミナル A で、Control-C を押して、ローカルネットワークの処理を中断します。
2.  ターミナル B で、Control-C を押して、開発サーバーのプロセスを中断します。
3.  ローカル・コンピュータ上で動作しているローカル Canister 実行環境を停止します。

        dfx stop

## オンチェーンにデプロイする（10分）

### サイクルに関する重要な注意事項

IC Dapps がオンチェーンで動作するためには、計算機とストレージのための Cycle が必要です。つまり、開発者は Cycle を獲得し、Canister を満たす必要があります。Cycle は ICP トークンから変換することができます。

このフローは、Web2 ソフトウェアに慣れている人にとっては驚くべきことかもしれません。ホスティングプロバイダーにクレジットカードを追加し、アプリをデプロイし、後で課金されます。Web3 では、ブロックチェーンはスマートコントラクトに以下を要求します。
Web3 ではブロックチェーンはスマートコントラクトが**何か**を消費することを要求します（それがイーサリアムのガスであろうと IC のサイクルであろうと）。
次のステップは、クリプトに携わる人にはおなじみのものでしょうが、新規参入者はなぜ Dapp デプロイメントの最初のステップが「トークンを取りに行く」なのか、混乱するかもしれません。


### Cycle を取得し、Canister に追加する（ターミナル B）

このチュートリアルでは、Cycle フォーセットから `Hello` Dapp 用の無料 Cycle を取得します。こちらの手順に従ってください。[Cycle フォーセット](cycles-faucet.md)

Cycle に関するいくつかの注意点：

-   Cycle は計算とストレージに支払われます。

-   Cycle のフォーセットは、開発者に20兆 Cycle を付与します。

-   Canister をデプロイするのに4兆 Cycle が必要です。

-   計算とストレージのコストの表はこちらで確認できます。[Computation and storage costs](../deploy/computation-and-storage-costs) をご覧ください。

### Internet Computer ブロックチェーン・メインネットへの接続をチェックする（ターミナル B）

サニティチェックとして IC との接続が安定していることを確認してください：

Internet Computer ブロックチェーンの現状と、それに接続できるかどうかを確認するには、次のコマンドを ネットワークエイリアス ic として実行します。

    $ dfx ping ic

成功すると、次のような出力が表示されます：

    $ {
      "ic_api_version": "0.18.0"  "impl_hash": "d639545e0f38e075ad240fd4ec45d4eeeb11e1f67a52cdd449cd664d825e7fec"  "impl_version": "8dc1a28b4fb9605558c03121811c9af9701a6142"  "replica_health_status": "healthy"  "root_key": [48, 129, 130, 48, 29, 6, 13, 43, 6, 1, 4, 1, 130, 220, 124, 5, 3, 1, 2, 1, 6, 12, 43, 6, 1, 4, 1, 130, 220, 124, 5, 3, 2, 1, 3, 97, 0, 129, 76, 14, 110, 199, 31, 171, 88, 59, 8, 189, 129, 55, 60, 37, 92, 60, 55, 27, 46, 132, 134, 60, 152, 164, 241, 224, 139, 116, 35, 93, 20, 251, 93, 156, 12, 213, 70, 217, 104, 95, 145, 58, 12, 11, 44, 197, 52, 21, 131, 191, 75, 67, 146, 228, 103, 219, 150, 214, 91, 155, 180, 203, 113, 113, 18, 248, 71, 46, 13, 90, 77, 20, 80, 95, 253, 116, 132, 176, 18, 145, 9, 28, 95, 135, 185, 136, 131, 70, 63, 152, 9, 26, 11, 170, 174]
    }

### オンチェーンデプロイ（ターミナル B）

これで、オンチェーンに Dapp をデプロイする準備が整いました。

    $ npm install

    $ dfx deploy --network ic

`--network` オプションは Dapp をデプロイするためのネットワークエイリアスまたは URL を指定します。このオプションは、Internet Computer ブロックチェーンメインネット上でインストールすることを要求されます。

もし成功すれば、以下のように表示されるはずです：

    Deploying all canisters.
    Creating canisters...
    Creating canister "hello_backend"...
    "hello_backend" canister created on network "ic" with canister id: "5o6tz-saaaa-aaaaa-qaacq-cai"
    Creating canister "hello_frontend"...
    "hello_frontend" canister created on network "ic" with canister id: "5h5yf-eiaaa-aaaaa-qaada-cai"
    Building canisters...
    Building frontend...
    Installing canisters...
    Installing code for canister hello_backend, with canister_id 5o6tz-saaaa-aaaaa-qaacq-cai
    Installing code for canister hello_frontend, with canister_id 5h5yf-eiaaa-aaaaa-qaada-cai
    Authorizing our identity (default) to the asset canister...
    Uploading assets to asset canister...
      /index.html 1/1 (472 bytes)
      /index.html (gzip) 1/1 (314 bytes)
      /index.js 1/1 (260215 bytes)
      /index.js (gzip) 1/1 (87776 bytes)
      /main.css 1/1 (484 bytes)
      /main.css (gzip) 1/1 (263 bytes)
      /sample-asset.txt 1/1 (24 bytes)
      /logo.png 1/1 (25397 bytes)
      /index.js.map 1/1 (842511 bytes)
      /index.js.map (gzip) 1/1 (228404 bytes)
      /index.js.LICENSE.txt 1/1 (499 bytes)
      /index.js.LICENSE.txt (gzip) 1/1 (285 bytes)
    Deployed canisters.

4.4 節で発生する可能性のある一般的なエラーに注意してください。
`Error: The replica returned an HTTP Error: Http Error: status 403 Forbidden`
このエラーは、Canister にデプロイするための十分な Cycle がないことを意味します。

### コマンドライン（ターミナル B）でオンチェーン Dapp をテストする

これで Canister はオンチェーンに配備されたので、メッセージを送ることができます。
Canister には `greet` というメソッドがあるので （これはパラメータとして文字列を受け取ります）それにメッセージを送ります。

    $ dfx canister --network ic call hello_backend greet '("everyone": text)'

メッセージの組み立て方にもご注目ください。
`dfx canister --network ic call` は、IC 上の Canister を呼び出すための設定です。
`hello_backend` という名前の Canister にメッセージを送り、その `greet` メソッドを呼び出すということです。 `'("everyone": text)’` は `greet` に送るパラメータです （このパラメータは `Text` だけを入力として受け取ります）。

## トラブルシューティング

### リソース

-   問題に当たった開発者は、検索するか、次の場所に投稿することをお勧めします。
    [IC 開発者フォーラム](https://forum.dfinity.org)

<!--
---
title: Hello World
---

# How to Deploy a "Hello World" Smart Contract in 20 Minutes

This is a fast and minimalist tutorial for deploying a "hello world"
canister smart contract to the IC in 20 minutes or less. All that is
necessary to run this tutorial is basic knowledge of using a terminal. No code editing is required.

## Introduction

In this tutorial, we will deploy a simple `hello` canister smart
contract that has just one function called `greet`. The `greet` function
accepts one text argument and returns the result with a greeting. For
example, if you call the `greet` method with the text argument `Alice`.

* If you call it via the command-line, dapp will return `Hello, Alice!` in a terminal

* If you access the dapp in a browser, it will alert pop-up window reading `Hello, Alice!`

While the code comes ready out of the box for you, this dapp consists of
backend code written in Motoko, a programming language specifically
designed for interacting with the Internet Computer (IC), and a simple
webpack-based frontend.

### Concepts necessary for this tutorial

* **Canister smart contract:** A *canister smart contract*, or *canister* for short, is a type of smart contract that bundles code and state. A canister is deployed to the IC, where it gets executed and can be accessed over the Internet. A *dapp* is a distributed application, composed of one or more canisters.

* **Cycles:** A *cycle* is the unit of measurement for
resources consumed on the IC in the form of processing, memory, storage, and
network bandwidth. Every canister has a cycles account to which
resources consumed by the canister are charged. The IC’s
utility token (ICP) can be converted to cycles and transferred to a
canister. ICP can always be converted to cycles using the current price
of ICP measured in \[SDR\] using the convention that one trillion cycles
correspond to one SDR.

In this tutorial, you will get free cycles from the cycles faucet.

## Supported operating systems

This tutorial, requires one of the following operating systems
-   Linux
-   macOS (12.\* Monterey)
-   Windows (With [WSL installed](windows-wsl))

## Installing tools (5 min)

To build and deploy a canister, users need to install the following:

### Node.js

There are many ways of installing node.js. On Linux, we recommend using your system's package manager. On macOS, we recommend [Homebrew](https://brew.sh). Alternatively, you find instructions on the [nodejs.org website](https://nodejs.org/en/download).

This tutorial works best with a node.js version higher than `16.*.*`.

### DFX

In this tutorial, we use a Canister SDK called `dfx`, which is the default SDK maintained by the DFINITY foundation. There are many other SDK’s so this is
just one.

To install, run:

    $ sh -ci "$(curl -fsSL https://internetcomputer.org/install.sh)"

To verify that `dfx` is properly installed, run:

    $ dfx --version

The terminal should show you the most recent version ([See SDK release notes](../updates/release-notes/release-notes.md)).

## Create a project (2 min)

A `dfx` project is a set of artifacts, including source code and configuration files, that can be compiled to a canister. By running 

    $ dfx new hello

`dfx` creates a new project directory called `hello`. The terminal output should look like this:

![dfx new](_attachments/dfx-new-hello-1.png)

![dfx new](_attachments/dfx-new-hello-2.png)


The `hello` project directory includes the artifacts required for a "hello world" canister and a new `hello` Git repository. Your directory should look like this:

![cd new](_attachments/cd-hello.png)


## Run your canister locally (3 min)

Now that the code is there, the next step is to spin up a local execution environment that allows you to run your canister locally. For this, we recommend using two terminals:

-   **Terminal A** shows the output of the execution environment. This is analogous to starting a local server in a web2 project, e.g. node.js’s Express, python’s Django, or Ruby’s Rails.

-   **Terminal B** is used to interact with the canister running in the execution environment. This is analogous to sending HTTP API messages to
        servers running locally, e.g. Postman or Panic.

To distinguish the two terminals in this tutorial, terminal A has a dark blue background... 

![dfx new](_attachments/dfx-new-hello-2.png)

... and terminal B has a black background. 

![terminal b ls](_attachments/terminal-b-ls.png)

### Start the execution environment

In terminal A, navigate to the root directory `hello` of our project and run

    $ dfx start

![dfx start](_attachments/terminal-a-dfx-start.png)

Note: Depending on your platform and local security settings, you might
see a warning displayed. If you are prompted to allow or deny incoming
network connections, click Allow.

That's it, there is now a local canister execution environment running on your
machine. Leave terminal A open and running while you continue. If you
close the terminal, the execution environment will stop the rest of the tutorial will fail.

### Deploy the canister locally

Note: since this is only a canister execution environment, this deployment has fewer steps than a deployment to mainnet, which requires cycles.

In terminal B, navigate to the root directory `hello` of our project. Install all the necessary node modules by running

    $ npm install

![npm install](_attachments/terminal-b-npm-install.png)

Register, build, and deploy the `hello` canister to the local execution environment by running

    $ dfx deploy

![dfx deploy](_attachments/terminal-b-dfx-deploy.png)

### Test the canister locally via the command line

Now that the canister is deployed to the local execution environment, you can interact with the canister by sending and receiving messages. Since the canister has a method called `greet` (which accepts a string as a parameter), we will send it a message. In terminal B, run

    $ dfx canister call hello_backend greet everyone

-   The `dfx canister call` command requires you to specify a canister
    name and a function to call.

-   `hello_backend` specifies the name of the canister you call.

-   `greet` specifies the function name.

-   `everyone` is the argument that you pass to the `greet` function.

### Test the canister locally via the browser

Now that you have verified that your dapp has been deployed and tested
its operation using the command line, let’s verify that you can access
the frontend using your web browser.

In terminal B, start the development server by running

    $ npm start

Open a browser and navigate to <http://localhost:8080/>.

You should see a simple HTML page with a sample asset
image file, an input field, and a button.

![Sample HTML page](_attachments/frontend-prompt.png)

Type a greeting, then click **Click Me** to return the greeting.

![Hello](_attachments/frontend-result.png)

## Stop the local canister execution environment

After testing the application in the browser, you can stop the local
canister execution environment so that it does not continue running in
the background.

To stop the local deployment:

1.  In the terminal A, press Control-C to interrupt the local network
    process.

2.  In the terminal B, press Control-C to interrupt the development
    server process.

3.  Stop the local canister execution environment running on your local
    computer by running the following command:

        dfx stop

## Deploying on-chain (10 min)

### Important note about cycles

In order to run on-chain,ICdapps require cycles to pay for compute and
storage. This means that the developer needs to acquire cycles and fill
their canister with them. Cycles can be converted from ICP token.

This flow may be surprising to people familiar with Web2 software where
they can add a credit card to a hosting provider, deploy their apps, and
get charged later. In Web3, blockchains require their smart contracts
consume **something** (whether it is Ethereum’s gas or the IC’s cycles).
The next steps will likely be familiar to those in crypto, but new
entrants may be confused as to why first step of deploying a dapp is
often "go get tokens."

### Acquiring cycles and adding them to your canister (Terminal B)

For the purposes of this tutorial, you can acquire free cycles for your
"hello world" dapp from the cycles faucet. Follow the instructions here:
[Claim your free cycles](cycles-faucet.md).

Few notes about cycles:

-   Cycles pay for computation and storage

-   Cycles faucet will grant developers 20 trillion cycles

-   It takes 4 trillion cycles to deploy a canister.

-   You can see a table of compute and storage costs here: [Computation
    and storage
    costs](../deploy/computation-and-storage-costs).

### Check the connection to the Internet Computer blockchain mainnet (Terminal B)

As sanity check, it is good practice to check your connection to the IC
is stable:

Check the current status of the Internet Computer blockchain and your
ability to connect to it by running the following command for the
network alias ic:

    $ dfx ping ic

If successful you should see output similar to the following:

    $ {
      "ic_api_version": "0.18.0"  "impl_hash": "d639545e0f38e075ad240fd4ec45d4eeeb11e1f67a52cdd449cd664d825e7fec"  "impl_version": "8dc1a28b4fb9605558c03121811c9af9701a6142"  "replica_health_status": "healthy"  "root_key": [48, 129, 130, 48, 29, 6, 13, 43, 6, 1, 4, 1, 130, 220, 124, 5, 3, 1, 2, 1, 6, 12, 43, 6, 1, 4, 1, 130, 220, 124, 5, 3, 2, 1, 3, 97, 0, 129, 76, 14, 110, 199, 31, 171, 88, 59, 8, 189, 129, 55, 60, 37, 92, 60, 55, 27, 46, 132, 134, 60, 152, 164, 241, 224, 139, 116, 35, 93, 20, 251, 93, 156, 12, 213, 70, 217, 104, 95, 145, 58, 12, 11, 44, 197, 52, 21, 131, 191, 75, 67, 146, 228, 103, 219, 150, 214, 91, 155, 180, 203, 113, 113, 18, 248, 71, 46, 13, 90, 77, 20, 80, 95, 253, 116, 132, 176, 18, 145, 9, 28, 95, 135, 185, 136, 131, 70, 63, 152, 9, 26, 11, 170, 174]
    }

### Deploying on-chain (Terminal B)

You are now ready to deploy your dapp on-chain.

    $ npm install

    $ dfx deploy --network ic

The `--network` option specifies the network alias or URL for deploying
the dapp. This option is required to install on the Internet Computer
blockchain mainnet.

If succesful, your terminal should look like this:

    Deploying all canisters.
    Creating canisters...
    Creating canister "hello_backend"...
    "hello_backend" canister created on network "ic" with canister id: "5o6tz-saaaa-aaaaa-qaacq-cai"
    Creating canister "hello_frontend"...
    "hello_frontend" canister created on network "ic" with canister id: "5h5yf-eiaaa-aaaaa-qaada-cai"
    Building canisters...
    Building frontend...
    Installing canisters...
    Installing code for canister hello_backend, with canister_id 5o6tz-saaaa-aaaaa-qaacq-cai
    Installing code for canister hello_frontend, with canister_id 5h5yf-eiaaa-aaaaa-qaada-cai
    Authorizing our identity (default) to the asset canister...
    Uploading assets to asset canister...
      /index.html 1/1 (472 bytes)
      /index.html (gzip) 1/1 (314 bytes)
      /index.js 1/1 (260215 bytes)
      /index.js (gzip) 1/1 (87776 bytes)
      /main.css 1/1 (484 bytes)
      /main.css (gzip) 1/1 (263 bytes)
      /sample-asset.txt 1/1 (24 bytes)
      /logo.png 1/1 (25397 bytes)
      /index.js.map 1/1 (842511 bytes)
      /index.js.map (gzip) 1/1 (228404 bytes)
      /index.js.LICENSE.txt 1/1 (499 bytes)
      /index.js.LICENSE.txt (gzip) 1/1 (285 bytes)
    Deployed canisters.

Note, a common error one may get in section 4.4:
`Error: The replica returned an HTTP Error: Http Error: status 403 Forbidden`.
This error means that the canister does not have enough cycles to
deploy.

### Testing the on-chain dapp via command line (Terminal B)

Now that the canister is deployed on-chain, you can send it a message.
Since the canister has a method called `greet` (which accepts a string
as a parameter), we will send it a message.

    $ dfx canister --network ic call hello_backend greet '("everyone": text)'

Note the way the message is constructed: \*
`dfx canister --network ic call` is setup for calling a canister on the
IC \* `hello_backend greet` means we are sending a message to a canister named
`hello_backend` and evoking its `greet` method \* `'("everyone": text)'` is the
parameter we are sending to `greet` (which accepts `Text` as its only
input).

## Troubleshooting

### Resources

-   Developers who hit any blockers are encouraged to search or post in
    [IC developer forum](https://forum.dfinity.org).

-->
