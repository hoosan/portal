# "Hello World" Dapp を10分間でデプロイする

"Hello World" Dapp を Internet Computer (IC) に 10 分以内でデプロイするためのクイックチュートリアルです。アプリのデプロイに必要な知識はターミナルの使い方のみです。
<!-- Code editing knowledge is not necessary. -->

始める前に、オンチェーンで動作しているこの Dapp のバージョンを見てみましょう：<https://6lqbm-ryaaa-aaaai-qibsa-cai.ic0.app/>

このチュートリアルでは、次のことを学びます：

1.  Canister SDK をインストールする
2.  ローカルで Dapp をビルドしてデプロイする
3.  Dapp の動力（ガス）となる無料 Cycle を収集する
4.  Cycle ウォレットを作成し、そこから他の Dapp に Cycle を転送する
5.  Dapp をオンチェーンにデプロイする

このシンプルな `Hello` Dapp は、2つの [Çanister スマートコントラクト](https://wiki.internetcomputer.org/wiki/Glossary#C)（バックエンド用とフロントエンド用）で構成されています。このアプリはテキストを入力として受け取り、greeding を返します。例えば、Canister SDK（Canisterのインストール方法は後述）を使ってコマンドラインで `greet` メソッドに `Everyone` というテキスト引数を与えて呼び出すと、ターミナルに `Hello, Everyone!` と返されます：

``` bash
$ dfx canister call hello_backend greet Everyone
$ "Hello, Everyone"
```

またはブラウザで Dapp を経由すると、メッセージとともにポップアップウィンドウが表示されます。 `Hello, Everyone!`

![Hello](_attachments/frontend-result.png)

"Hello World" Dapp は、IC とのやりとりに特化したプログラミング言語である [Motoko](../build/cdks/motoko-dfinity/motoko.md) で書かれたバックエンドコードと、webpack ベースのシンプルなフロントエンドから構成されていることに注意してください。

このチュートリアルは、Linux、macOS 12.\* Monterey 以降、または [Windows Subsystem for Linux (WSL)](windows-wsl.md) がインストールされた Windows で実施します。

## このチュートリアルで扱うトピック

-   ** Canister **は、IC にインストールされたスマートコントラクトです。実行されるコードと、その結果生成されるステートが格納されます。"Hello World” アプリのように、複数の Canister で構成されることが一般的です。

-   **[Cycle](../../concepts/tokens-cycles.md)** は、リソースの消費量を表す単位で、通常は IC 上で消費される処理（メモリ、ストレージ、ネットワーク帯域幅）を指します。このチュートリアルでは、Cycle はイーサリアムのガスに似ています。Cycle は Dapps を実行するために必要ですが、ガスと違って安定しており、価格も安くなります。各 Canister には Cycle アカウントがあり、そこから Canister で消費されたリソースがチャージされます。IC のユーティリティ・トークン（ICP）は、Cycle に変換して Canister に転送することができます。ICP は常に [SDR ](https://en.wikipedia.org/wiki/Special_drawing_rights)(通貨バスケット)で測定された ICP の現在価格を使って Cycle に変換することができ、1兆サイクルが 1SDR に対応するという慣例があります。**Cycle faucet（無料）** 

-   **[Cycle ウォレット](../build/project-setup/cycles-wallet.md)**は Cycle を入れて、Dapps をパワーアップする Canister です。

## 1. ツールをインストールする

`Hello` Dapp をビルドしてデプロイするには、以下のツールをインストールする必要があります。

### DFX

このチュートリアルでは、DFINITY 財団が管理しているデフォルトのSDKである `dfx` という Canister SDK を使用します。 

`dfx` をインストールするには、以下を実行します：

``` bash
sh -ci "$(curl -fsSL https://smartcontracts.org/install.sh)"
```

`dfx` が正しくインストールされたか確認するには以下を実行します：

``` bash
dfx --version
```

ターミナルには最新版（参照: [SDK リリースノート](../updates/release-notes/release-notes.md)）が表示されるはずです。

その他のインストールオプションや `dfx` のアンインストール方法については、[SDKのインストール](../build/install-upgrade-remove.mdx)で説明されています。

### Node.js

Node.js はフロントエンドのアセットをレンダリングするために必要なので、このチュートリアルを完了させるために必要です。しかし、一般的な Canister の開発には Node.js は必要ありません。

Node.js は12から始まるすべての安定したバージョンをサポートしています。12、14、16 のいずれかをインストールすることができます。Node 17は Webpack の API プロキシツールをサポートしていないため、`npm start` が正しく動作しない可能性があることに注意してください。

node.js をインストールする方法はたくさんあります。Linux では、お使いのシステムのパッケージマネージャを使用することをお勧めします。macOS では、[Homebrew](https://brew.sh) をお勧めします。また、[nodejs.org website](https://nodejs.org/en/download) に説明があります。

このチュートリアルは `16.*.*` よりも高いバージョンの node.js で最適に動作します。

<!-- Besides installing node.js, users need to also install: 
* Node Package Manager (NPM). (This comes packaged with Node, but you may want to upgrade with `npm i -g npm`) 
* Node Version Manager (NVM), see [installing NVM](https://github.com/nvm-sh/nvm#installing-and-updating).
* Once you have NVM, install the latest stable build with `nvm install --lts` -->

## 2. プロジェクトを生成する（１分）

<!-- After the SDK is installed, create the default "Hello World" project with two canisters (backend and frontend).

The SDK comes with a starter default project that has a backend in Motoko and frontend code in HTML, CSS, and JS. Developers can use this default project to start their own dapps. In this tutorial, we will build and deploy this bundled project, so there is no need to download any other dapp code. -->

<!-- The `dfx new hello` command uses the template code to create a new project directory named `hello`, template files, and a new `hello` Git repository for your project. You can create many projects with many names. -->

<!-- This is roughly analogous in Web2 to Rail’s `rails new`, React.js’s `create-react-app`, or Rust’s `cargo new`. -->

<!-- To create a new project for your first application: -->

<!-- ### 2.1 Open a Terminal and Create a new project named "hello" -->

`dfx` プロジェクトは、ソースコードや設定ファイルを含むアーティファクトのセットであり、Canister にコンパイルすることができます。以下を実行します： 

``` bash
dfx new hello
```

`dfx` は `hello` という名前の新しいプロジェクトディレクトリを作成します。ターミナルの出力は以下のようになるはずです：

![dfx new](_attachments/dfx-new-hello-1.png)

![dfx new](_attachments/dfx-new-hello-2.png)

<!-- ### 2.2 Move to your project directory

``` bash
cd hello
```

Your directory should look like this: -->

"hello world” Canister と新しい `hello` Git リポジトリに必要なアーティファクトは、`hello` プロジェクトディレクトリにあります。ディレクトリはこのようになっているはずです：

![cd new](_attachments/cd-hello.png)

<!-- ### 2.3 Understanding your dapp project -->

`hello`プロジェクトは、2つの Canister で構成されています：

-   `hello_backend` Canister、 テンプレートバックエンドロジックを含みます
-   `hello_frontend` Canister、 Dapp アセット（画像や html ファイルなど）を含みます

![hello Dapp](_attachments/2-canisters-hello-dapp.png)

「なぜ Canister が2つあるのか」と思われるかもしれません。2つの Canister は、プロジェクトを整理するために作成されます。アセットとバックエンドのロジックを1つの Canister に収めることもできますが、IC 開発者は2つの Canister（バックエンド用とフロントエンド用）を作成することが有用であると理解しています。

## 3. Dapp をローカル環境で実行する（3分）

これで `hello` プロジェクトが作成されましたので、次のステップではそれをローカルにデプロイします。ローカルにデプロイするために `dfx` は実行環境のローカルインスタンスを開始することができます。この環境は完全な IC レプリカではなく、IC レプリカのステートをダウンロードすることもありません。Dapps のデプロイ専用に設計された軽量な環境です。

このため、ターミナルを2つ使用することを推奨します：

-   **ターミナル A** には、実行環境の出力が表示されます。これは、node.js の Express、python の Django、Ruby の Rails など、web2 プロジェクトでローカルサーバを起動するのと似ています。

-   **ターミナル B** は、実行環境で動作している Canister と対話するために使用されます。これは、ローカルで動作しているサーバ、例えば Postman や Panic に HTTP API メッセージを送信することに類似しています。

このチュートリアルでは、2つのターミナルを区別するために、ターミナル A の背景を紺色にしています...

![dfx new](_attachments/dfx-new-hello-2.png)

... ターミナル B の背景は黒色です。

![terminal b ls](_attachments/terminal-b-ls.png)

### 実行環境の起動する

<!-- Navigate to the root directory for your project, if necessary. In this tutorial, you should be in the folder `hello` because that is the name of the project created in section 2 above.

Start the local canister execution environment in Terminal A: -->

ターミナル A で、プロジェクトのルートディレクトリ `hello` に移動し、次のように実行します：

``` bash
dfx start
```

![dfx start](_attachments/terminal-a-dfx-start.png) 

:::note

-   お使いのプラットフォームやローカルのセキュリティ設定によっては、警告が表示されることがあります。着信ネットワーク接続の許可または拒否を求めるメッセージが表示された場合は、「許可 」をクリックしてください。
-   8000番台のポートの競合を引き起こすような他のネットワークプロセスが実行されていないことを確認します。

:::

**おめでとうございます！ これであなたのマシン上でICの実行環境のローカルインスタンスが動作しています！ このウィンドウ/タブを開いたままにしておいてください。** ウィンドウ/タブを閉じると、IC の実行環境のローカルインスタンスが動作しなくなり、このチュートリアルの続きができなくなります。

### Dappをローカルにデプロイする

:::note
これは Canister 実行環境のみなので、このデプロイはメインネットへのデプロイよりも手順が少ないですが、[Cycle](../../concepts/tokens-cycles.md) が必要です。
:::

<!-- To deploy your first dapp locally:

#### 3.2.1 Check that you are still in the root directory for your project, if needed.

Ensure that node modules are available in your project directory, if needed, by running the following command (it does not hurt to run this many times): -->

ターミナル B で、プロジェクトのルートディレクトリ `hello` に移動します。必要なすべての node モジュールをインストールするには、以下のコマンドを実行します：

``` bash
npm install
```

![npm install](_attachments/terminal-b-npm-install.png)

`hello` Canister を登録、ビルドし、ローカルの実行環境にデプロイします。以下のコマンドを実行します：

``` bash
dfx deploy
```

![dfx deploy](_attachments/terminal-b-dfx-deploy.png) 

Dapp は2つの Canister で構成されています。（ターミナル B から）以下のコピーで確認できます：

``` bash
Installing code for canister hello_backend, with canister_id rrkah-fqaaa-aaaaa-aaaaq-cai
Installing code for canister hello_frontend, with canister_id ryjl3-tyaaa-aaaaa-aaaba-cai
```

1.  `hello_backend` Canister `rrkah-fqaaa-aaaaa-aaaaq-cai` バックエンドロジックを含みます。

2.  `hello_frontend` Canister `yjl3-tyaaa-aaaaa-aaaba-cai` フロントエンドのアセット（HTML、JS、CSSなど）を含みます。

### コマンドラインからローカルの Dapp をテストする

これで、Canister がローカルの実行環境にデプロイされたので、メッセージを送受信して Canister と対話できるようになります。Canister には `greet` というメソッド （パラメータとして文字列を受け取ります）があるので、メッセージを送信してみましょう。ターミナル B で、以下を実行します。

``` bash
dfx canister call hello_backend greet everyone
```

-   `dfx canister call` コマンドでは、Canister 名と呼び出す関数を指定する必要があります。
-   `hello_backend` は呼び出す Canister の名前を指定します。
-   `greet` は関数名を指定します。
-   `everyone` は `greet` 関数に渡す引数です。

### ブラウザでローカルの Dapp をテストする

Dapp がデプロイされ、コマンドラインで動作確認ができたので、Web ブラウザでフロントエンドにアクセスできることを確認しましょう。

ターミナル B で、開発用サーバーを起動します：

``` bash
npm start
```

<!-- ### 3.4.2 Test the dapp locally in the browser

To see your dapp running locally in the browser on http://localhost:8080. -->

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

    ``` bash
    dfx stop
    ```

### トラブルシューティング

### Node.js が正しくインストールされない

ブラウザでアプリが表示されない場合、Node.js がインストールされていない可能性があります。インストールされているかどうか、実行して確認してください。

``` bash
node --version
```

### 以前の dfx インストール

2022年2月以前に IC Dapp を作成したことがある場合、クリーンインストールが必要な場合があります。SDK と関連するプロファイルを削除して、再インストールすることができます。こちらの手順に従ってください。[SDK のインストール](../build/install-upgrade-remove.mdx) に従ってください。

## 4. オンチェーン・デプロイのための Cycle 取得（5分）

IC Dapps がオンチェーンで動作するためには、計算と保存のための費用として Cycle が必要です。つまり、開発者は Cycle を獲得し、Canister に充填する必要があります。Cycle は ICP トークンから生成されます。

このフローは、ホスティングプロバイダーにクレジットカードを追加してアプリをデプロイし、後で課金される Web2 ソフトウェアに慣れている人にとっては驚くべきことかもしれません。Web3 では、ブロックチェーンはスマートコントラクトが**何か**を消費することを要求します（それがイーサリアムのガスであろうとICの Cycle であろうとです）。次のステップは、クリプトやブロックチェーンの関係者には馴染み深いもので、アプリをデプロイする最初のステップは「トークンを取りに行く」というものです。

さらに、なぜ ICP トークンではなく Cycle で Dapp が動くのか不思議に思うかもしれません。ICP トークンのコストは暗号市場によって変動しますが、Cycle は予測可能で比較的安定したトークンであり、[SDR](https://en.wikipedia.org/wiki/Special_drawing_rights) にペグされるからです。1兆 Cycle は、ICP の価格に関係なく、常に 1SDR のコストになります。

Cycle に関する実用的な注意点：

-   新しい開発者は20 兆 Cycle を与えられます。 [Cycle フォーセット](cycles-faucet.md)
-   Canister をデプロイするには1000億 Cycle が必要ですが、十分な Cycle で Canister をロードするために、`dfx` は Canister を作成すると3兆 Cycle を注入します（これは変更可能なパラメータです）。
-   計算と保存のコストの表はこちらで見ることができます。[計算とストレージのコスト](../deploy/computation-and-storage-costs.md)
-   ICP の取得と管理については、[Acquiring and managing ICP tokens](https://wiki.internetcomputer.org/wiki/Tutorials_for_acquiring,_managing,_and_staking_ICP)で詳しく説明しています。

このチュートリアルでは、Cycle を取得する2つの方法を紹介します：

-   **オプション1：** [無料 Cycle フォーセットから Cycle を取得する](#オプション1-無料の-cycle-フォーセットによる-cycle-の取得2分)は、Cycle フォーセットから Cycle を取得する方法です（新しい開発者に最も一般的です）。
-   **オプション2：** [ICP トークンを Cycle に変換する](#オプション2-icp-トークンを-cycle-に変換する5分)は ICP トークンで Cycle を取得する方法です（多くの Cycle を求める開発者に最も一般的です）。

このセクションが終わるころには、Canister が3つになっていることでしょう：

-   `hello_backend` Canister（IC に未デプロイ）
-   `hello_frontend` プロジェクトの Canister（IC に未デプロイ）
-   Cycle を収納するウォレット Canister（ICにデプロイ済み）

![hello dapp and cycles wallet](_attachments/3-canisters-hello-dapp.png)

<!-- ### 4.2 Check the connection to the Internet Computer (terminal B) -->

サニティチェックとして、Internet Computer のブロックチェーンの現状と接続の可否を確認することで、IC への接続が安定しているかどうかを確認するとよいでしょう：

``` bash
dfx ping ic
```

成功すると、次のような出力が表示されます：

``` bash
$ {
  "ic_api_version": "0.18.0"  "impl_hash": "d639545e0f38e075ad240fd4ec45d4eeeb11e1f67a52cdd449cd664d825e7fec"  "impl_version": "8dc1a28b4fb9605558c03121811c9af9701a6142"  "replica_health_status": "healthy"  "root_key": [48, 129, 130, 48, 29, 6, 13, 43, 6, 1, 4, 1, 130, 220, 124, 5, 3, 1, 2, 1, 6, 12, 43, 6, 1, 4, 1, 130, 220, 124, 5, 3, 2, 1, 3, 97, 0, 129, 76, 14, 110, 199, 31, 171, 88, 59, 8, 189, 129, 55, 60, 37, 92, 60, 55, 27, 46, 132, 134, 60, 152, 164, 241, 224, 139, 116, 35, 93, 20, 251, 93, 156, 12, 213, 70, 217, 104, 95, 145, 58, 12, 11, 44, 197, 52, 21, 131, 191, 75, 67, 146, 228, 103, 219, 150, 214, 91, 155, 180, 203, 113, 113, 18, 248, 71, 46, 13, 90, 77, 20, 80, 95, 253, 116, 132, 176, 18, 145, 9, 28, 95, 135, 185, 136, 131, 70, 63, 152, 9, 26, 11, 170, 174]
}
```

### オプション1: 無料の Cycle フォーセットによる Cycle の取得（2分）

このオプションは、最小限の時間しか投資したくない人、Cycle フォーセットを使ったことがない人に最適です（フォーセットは1度だけ使用できます）。

このチュートリアルでは、Cycle フォーセットから `Hello` Dapp 用の無料 Cycle を取得します。こちらの手順に従ってください。[Cycle フォーセット](cycles-faucet.md)

#### Cycle 残高をチェックする

Cycle フォーセットを使用した後は、ターミナル B で Cycle 残高を確認することができます：

``` bash
dfx wallet --network ic balance
```

Cycle ウォレットを使用した後にこれを実行すると、20兆 Cycle が表示されるはずです。その場合は、[5.オンチェーンデプロイ（1分）](#5-オンチェーンデプロイ1分) のセクションにスキップしてください。

Cycle が表示されない場合、このチュートリアルの残りの部分でオンチェーンをデプロイしてもうまくいきません。[オプション2: ICP トークンを Cycle に変換する（5分）](#オプション2-icp-トークンを-cycle-に変換する5分) を試す必要があります。

### オプション2: ICP トークンを Cycle に変換する（5分）

このオプションは、すでに Cycle ウォレットを使い果たしてしまった人や、将来的に Cycle を追加するために環境を整えたい人に最適です。

ICP トークンを Cycle に変換するには、まず ICP をいくつか入手し、適切なアカウントに転送する必要があります。ICP トークンは取引所で入手するか、知り合いに頼んで送ってもらうことができます。ICP トークンをどのアカウントに転送するかは、以下を実行してください。

``` bash
dfx ledger account-id
```

これにより、ICP 台帳にアカウント番号が表示されます。これと似た文字列です：

```
e213184a548871a47fb526f3cba24e2ee2fbbc8129c4ab497ef2ce535130a0a4
```

このアカウントに ICP トークンを送金したら（5～10ドル分あれば十分）、このコマンドで残高を確認することができます：

``` bash
dfx ledger --network ic balance
```

すると、このような出力が得られます：

```
12.49840000 ICP
```

ICP トークンの準備ができたら、Cycle ウォレットの作成に取りかかりましょう。まず、ウォレットとなる Cycle を作成する必要があります。そのための基本コマンドは以下の通りです：

``` bash
dfx ledger --network ic create-canister <your-principal-identifier> --amount <icp-tokens>
```

代入する値は、自分の Principal と、変換したいトークン数の2つです。自分の Principal を知るには、`dfx identity get-principal` の出力を使用します。私の Principal が `tsqwz-udeik-5migd-ehrev-pvoqv-szx2g-akh5s-fkyqc-zy6q7-snav6-uqe` で、 2.3 ICPを Cycle に変換したい場合、コマンドは以下のようになります：

``` bash
dfx ledger --network ic create-canister tsqwz-udeik-5migd-ehrev-pvoqv-szx2g-akh5s-fkyqc-zy6q7-snav6-uqe --amount 2.3
```

このコマンドは少し時間がかかり、次のようなものが出力されます：

```
Transfer sent at BlockHeight: 351220
Canister created with id: "gastn-uqaaa-aaaae-aaafq-cai"
```

この出力にある ID は、あなたのウォレットの Canister アドレスです。この例では、`gastn-uqaaa-aaaae-aaafq-cai`となります。

これで Canister が作成されたので、このコマンドを使用してウォレットコードをインストールすることができます：

``` bash
dfx identity --network ic deploy-wallet <canister-identifer>
```

ここでは、前のコマンドの出力で受け取った ID を使用して、Canister の識別子を代入する必要があります。つまり、この例では次のようになります：

``` bash
dfx identity --network ic deploy-wallet gastn-uqaaa-aaaae-aaafq-cai
```

そして、このような出力になるはずです：

```
Creating a wallet canister on the IC network.
The wallet canister on the "ic" network for user "default" is "gastn-uqaaa-aaaae-aaafq-cai"
```

これで、ウォレットの設定が完了し、使用できるようになりました。すべてがうまくいったかどうかを確認するには、以下を実行して、設定したウォレットの識別子を確認します：

``` bash
dfx identity --network ic get-wallet
```

これで、先ほどのコマンドで使用した Canister ID が表示されるはずです。

また、新しい Cycle ウォレットの残高を確認することもできます：

``` bash
dfx wallet --network ic balance
```

これで、次のようなものが表示されるはずです：
```
6.951 TC (trillion cycles).
```


## 5. オンチェーン・デプロイ（1分）

[Cycle](../../concepts/tokens-cycles.md) と `dfx` が Cycle を転送するように設定されたので、`hello` Dapp をオンチェーンにデプロイする準備ができています。ターミナル B で、以下を実行します：

``` bash
npm install
```

``` bash
dfx deploy --network ic --with-cycles 1000000000000
```

`—network` オプションは、Dapp をデプロイするためのネットワークエイリアスまたは URL を指定します。このオプションは、Internet Computer のブロックチェーンメインネットにインストールするために必要です。`—with-cycles` は `dfx` に使用する Cycle 数を明示的に指定します。そうでなければ、デフォルトの3兆 Cycle が使用されます。

成功すると、ターミナルはこのようになります：

``` bash
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
URLs:
  Frontend:
    hello_frontend: https://5h5yf-eiaaa-aaaaa-qaada-cai.ic0.app/
  Candid:
    hello_backend: https://a4gq6-oaaaa-aaaab-qaa4q-cai.raw.ic0.app/?id=5o6tz-saaaa-aaaaa-qaacq-cai
```

メッセージの一番下に、Canister のフロントエンドがオンチェーンにデプロイされているのを見ることができる URL が返されることに注意してください：<https://5h5yf-eiaaa-aaaaa-qaada-cai.ic0.app/>

上記の例では、`hello` という Dapp を作成し、以下を構成しています：

1.  `hello_backend` Canister、 `5o6tz-saaaa-aaaaa-qaacq-cai` バックエンドロジックを含みます。

2.  `hello_frontend` Canister、 `5h5yf-eiaaa-aaaaa-qaada-cai` フロントエンドのアセット（HTML、JS、CSSなど）を含みます。

### ブラウザで デプロイされた Dapp オンチェーンを確認する

こちらに移動し、名前を入力します：<https://5h5yf-eiaaa-aaaaa-qaada-cai.ic0.app/>

Dapp がロードされる前に、ブラウザにはすぐに次のようなメッセージが表示されます。Installing "Internet Computer Validating Service Worker “ この [service worker](https://developer.chrome.com/docs/workbox/service-worker-overview/) は IC から来るもので、ユーザーが見る Web アプリが改ざんされていない正しいフロントエンドであることを確認するために使用されます。一度読み込まれると、ブラウザはサービスワーカーをキャッシュし、Web アプリはより速く読み込まれるようになります。

![Hello](_attachments/service-worker.png)

サービスワーカーをロードした後、Dapp がロードされます：

![Hello](_attachments/frontend-result.png)

### コマンドラインからオンチェーン Dapp をテストする

Canister には `greet` というメソッド（パラメータとして文字列を受け取ります）があるので、`dfx` を介してメッセージを送ることができます：

``` bash
dfx canister --network ic call hello_backend greet '("everyone": text)'
```

メッセージの構成に注目してください：

-   `dfx canister --network ic call` は IC 上の Canister を呼び出すためのセットアップです。

-   `hello_backend greet` は、`hello` という名前の Canister にメッセージを送信し、その `greet` メソッドを呼び出すことを意味します。これは、`hello` とCanister ID のマッピングがローカルの `.dfx/local/canister_ids.json` に保存されているためです。

-   `'("everyone": text)'` は `greet` に送るパラメータです （このパラメータは `Text` を唯一の入力として受け取ります）。

### トラブルシューティング

#### 403 Error

403 Error が表示される場合、使用している ID に十分な Cycle がない可能性があります。デバッグのために以下を試してみてください：

#### 1. あなたが想定している ID を使用していることを確認する

``` bash
dfx identity whoami
```

#### 2. 使用する ID がオンチェーンに十分な Cycle を有していることを確認する

``` bash
dfx wallet --network ic balance
```

#### 3. ウォレット経由のプロキシを試す

時々（特に0.9.0より前のバージョンの `dfx` で Canister を作成した場合）あなたのウォレットが Canister のコントローラとして設定されていることがあります。あなたのウォレットをデプロイ命令のソースにするには、デプロイコマンドまたはコールコマンドに `--wallet <insert-your-wallet-id-here>` というフラグを追加してください。

これがうまくいって、自分の Principal を Canister のコントローラとして追加したい場合（そうすれば、もう `--wallet` オプションを使う必要はありません）、次のように実行できます：

``` bash
dfx canister --wallet "$(dfx identity get-wallet)" update-settings --all --add-controller "$(dfx identity get-principal)"
```

## 完成

おめでとうございます。あなたは10分以内に完全にオンチェーン（バックエンドとフロントエンドの両方）の Dapp を構築しました。

チュートリアルの要約
-   Dapps は複数の Canister で構成することができます。
-   Dapps はローカルにもオンチェーンにもデプロイできます。
-   Dapps を動かすには Cycle が必要です。
-   Cycle ウォレットから無料 Cycle を取得します。
-   無料 Cycle を使って Dapps を追加することができます。

### 原点に立ち返る

ゼロから始める場合は、SDK と関連するプロファイルを削除して、再インストールしてください。こちらの手順に従ってください。[SDK のインストール](../build/install-upgrade-remove.mdx)

**Dapps や ICP に紐づく ID は必ず保存してください。**

### 困ったときは？

もし、問題が発生した場合は、[開発者フォーラム](https://forum.dfinity.org)や [Discord](https://discord.com/invite/cA7y6ezyE2) で解決策を探したり、質問を投稿してください。

### 次の挑戦の準備はできていますか？

DAO や NFT などの構築などさらに進むには [こちら](https://smartcontracts.org/samples)。

### もっと詳しく知りたい方は？

始める前にもっと情報を知りたい、自分で試す前にデプロイ方法のデモを見たい、という方は、以下の関連リソースをご覧ください：

-   [How you can use ICP tokens (overview)](../../concepts/tokens-cycles#using-tokens)

-   [Building on the Internet Computer: Fundamentals (video)](https://www.youtube.com/watch?v=jduSMHxdYD8)

-   [What is the DFINITY Canister SDK (video)](https://www.youtube.com/watch?v=60uHQfoA8Dk)

-   [Deploying your first dapp (video)](https://www.youtube.com/watch?v=yqIoiyuGYNA)

<!--575,580,652,666,686,692,725,754,808,877行はコメントアウトが入れ子にできないために追加しました-->

<!--
# Deploy a "Hello World" Dapp in 10 Minutes

This is a quick tutorial to deploy a "Hello World" dapp to the Internet Computer (IC) in 10 minutes or less. Deployment of the dapp only requires basic knowledge of using a terminal.
<!-- Code editing knowledge is not necessary. -->
<!--
Before starting, take a look at a version of this dapp running on-chain: <https://6lqbm-ryaaa-aaaai-qibsa-cai.ic0.app/>

In this tutorial, you will learn how to:

1.  Install the Canister SDK
2.  Build and deploy a dapp locally
3.  Collect free cycles to power your dapp
4.  Create a "cycles wallet" from which you can transfer cycles to any other dapps you want to power
5.  Deploy a dapp on-chain

This simple `Hello` dapp is composed of two [canister smart contracts](https://wiki.internetcomputer.org/wiki/Glossary#C) (one for backend and one for frontend). The dapp accepts a text argument as input and returns a greeting. For example, if you call the `greet` method with the text argument `Everyone` on the command-line via the canister SDK (see instructions below on how to install the canister SDK), the dapp will return `Hello, Everyone!` either in your terminal:

``` bash
$ dfx canister call hello_backend greet Everyone
$ "Hello, Everyone"
```

Or via the dapp in a browser, a pop-up window will appear with the message: `Hello, Everyone!`

![Hello](_attachments/frontend-result.png)

Note that the "Hello World" dapp consists of backend code written in [Motoko](../build/cdks/motoko-dfinity/motoko.md), a programming language specifically designed for interacting with the IC, and a simple webpack-based frontend.

This tutorial requires Linux, macOS 12.\* Monterey or later, or Windows with a [Windows Subsystem for Linux (WSL)](windows-wsl.md) installation.

## Topics Covered in this Tutorial

-   **Canisters** are the smart contracts installed on the IC. They contain the code to be run and a state, which is produced as a result of running the code. As is the case of the "Hello World" dapp, it is common for dapps to be composed of multiple canisters.

-   **[Cycles](../../concepts/tokens-cycles.md)** refer to a unit of measurement for resource consumption, typically for processing, memory, storage, and network bandwidth consumed on the IC. For the sake of this tutorial, cycles are analogous to Ethereum’s gas: cycles are needed to run dapps, but unlike gas they are stable and less expensive. Every canister has a cycles account from which the resources consumed by the canister are charged. The IC’s utility token (ICP) can be converted to cycles and transferred to a canister. ICP can always be converted to cycles using the current price of ICP measured in [SDR](https://en.wikipedia.org/wiki/Special_drawing_rights) (a basket of currencies) using the convention that one trillion cycles correspond to one SDR. **Get free cycles from the cycles faucet.**

-   A **[cycles wallet](../build/project-setup/cycles-wallet.md)** is a canister that holds cycles and powers up dapps.

## 1. Installing Tools

To build and deploy the `Hello` dapp, you need to install the following tools.

### DFX

In this tutorial, we use a Canister SDK called `dfx`, which is the default SDK maintained by the DFINITY foundation. 

To install `dfx`, run:

``` bash
sh -ci "$(curl -fsSL https://smartcontracts.org/install.sh)"
```

To verify that `dfx` properly installed, run:

``` bash
dfx --version
```

The terminal should show you the most recent version ([See SDK release notes](../updates/release-notes/release-notes.md)).

More installation options and instructions for uninstalling `dfx` are covered in [Installing the SDK](../build/install-upgrade-remove.mdx).

### Node.js

Node.js is necessary for rendering the frontend assets and so is necessary to complete this tutorial. Note however that Node.js is not needed for canister development in general.

We support all stable versions of Node.js starting with 12. You can install 12, 14, or 16. Please note that Node 17 does not support Webpack’s api proxy tool, so `npm start` may not work correctly.

There are many ways of installing node.js. On Linux, we recommend using your system's package manager. On macOS, we recommend [Homebrew](https://brew.sh). Alternatively, you find instructions on the [nodejs.org website](https://nodejs.org/en/download).

This tutorial works best with a node.js version higher than `16.*.*`.

<!-- Besides installing node.js, users need to also install: 
* Node Package Manager (NPM). (This comes packaged with Node, but you may want to upgrade with `npm i -g npm`) 
* Node Version Manager (NVM), see [installing NVM](https://github.com/nvm-sh/nvm#installing-and-updating).
* Once you have NVM, install the latest stable build with `nvm install --lts` -->
<!--
## 2. Create a project (1 min)

<!-- After the SDK is installed, create the default "Hello World" project with two canisters (backend and frontend).

The SDK comes with a starter default project that has a backend in Motoko and frontend code in HTML, CSS, and JS. Developers can use this default project to start their own dapps. In this tutorial, we will build and deploy this bundled project, so there is no need to download any other dapp code. -->

<!-- The `dfx new hello` command uses the template code to create a new project directory named `hello`, template files, and a new `hello` Git repository for your project. You can create many projects with many names. -->

<!-- This is roughly analogous in Web2 to Rail’s `rails new`, React.js’s `create-react-app`, or Rust’s `cargo new`. -->

<!-- To create a new project for your first application: -->

<!-- ### 2.1 Open a Terminal and Create a new project named "hello" -->
<!--
A `dfx` project is a set of artifacts, including source code and configuration files, that can be compiled to a canister. By running 

``` bash
dfx new hello
```

`dfx` creates a new project directory called `hello`. The terminal output should look similar to this:

![dfx new](_attachments/dfx-new-hello-1.png)

![dfx new](_attachments/dfx-new-hello-2.png)

<!-- ### 2.2 Move to your project directory

``` bash
cd hello
```

Your directory should look like this: -->
<!--
The `hello` project directory includes the artifacts required for a "hello world" canister and a new `hello` Git repository. Your directory should look like this:

![cd new](_attachments/cd-hello.png)

<!-- ### 2.3 Understanding your dapp project -->
<!--
The `hello` project is composed of two canisters:

-   `hello_backend` canister, which contains the template backend logic
-   `hello_frontend` canister, which contains the dapp assets (images, html files, etc)

![hello Dapp](_attachments/2-canisters-hello-dapp.png)

You may wonder "why two canisters?" Two canisters are created to help you organize your project. It could be that assets and backend logic live in one canister, but IC developers have found that it's useful to create two canisters (one for backend and one for frontend).

## 3. Run your dapp locally (3 min)

Now that your `hello` project is created, the next step is to deploy it locally. To deploy locally, `dfx` can start a local instance of the execution environment. This environment is not a full IC replica, nor does it download any of the state of an IC replica. It is a lightweight environment designed exclusively for deploying dapps.

For this, we recommend using two terminals:

-   **Terminal A** shows the output of the execution environment. This is analogous to starting a local server in a web2 project, e.g. node.js’s Express, python’s Django, or Ruby’s Rails.

-   **Terminal B** is used to interact with the canister running in the execution environment. This is analogous to sending HTTP API messages to servers running locally, e.g. Postman or Panic.

To distinguish the two terminals in this tutorial, terminal A has a dark blue background...

![dfx new](_attachments/dfx-new-hello-2.png)

... and terminal B has a black background.

![terminal b ls](_attachments/terminal-b-ls.png)

### Start the execution environment

<!-- Navigate to the root directory for your project, if necessary. In this tutorial, you should be in the folder `hello` because that is the name of the project created in section 2 above.

Start the local canister execution environment in Terminal A: -->
<!--
In terminal A, navigate to the root directory `hello` of our project and run

``` bash
dfx start
```

![dfx start](_attachments/terminal-a-dfx-start.png) 

:::note

-   Depending on your platform and local security settings, you might see a warning displayed. If you are prompted to allow or deny incoming network connections, click "Allow."
-   Check no other network process is running that would create a port conflict on 8000.

:::

**Congratulations - there is now a local Instance of the execution environment of the IC running on your machine! Leave this window/tab open and running while you continue.** If the window/tab is closed, the local instance of the execution environment of the IC will not be running and the rest of the tutorial will fail.

### Deploy the dapp locally

:::note
Since this is only a canister execution environment, this deployment has fewer steps than a deployment to mainnet, which requires [cycles](../../concepts/tokens-cycles.md)).
:::

<!-- To deploy your first dapp locally:

#### 3.2.1 Check that you are still in the root directory for your project, if needed.

Ensure that node modules are available in your project directory, if needed, by running the following command (it does not hurt to run this many times): -->
<!--
In terminal B, navigate to the root directory `hello` of our project. Install all the necessary node modules by running

``` bash
npm install
```

![npm install](_attachments/terminal-b-npm-install.png)

Register, build, and deploy the `hello` canister to the local execution environment by running

``` bash
dfx deploy
```

![dfx deploy](_attachments/terminal-b-dfx-deploy.png) 

Your dapp is now composed of two canisters as you can see in the copy below (from terminal B):

``` bash
Installing code for canister hello_backend, with canister_id rrkah-fqaaa-aaaaa-aaaaq-cai
Installing code for canister hello_frontend, with canister_id ryjl3-tyaaa-aaaaa-aaaba-cai
```

1.  `hello_backend` canister `rrkah-fqaaa-aaaaa-aaaaq-cai` which contains the backend logic.

2.  `hello_frontend` canister `yjl3-tyaaa-aaaaa-aaaba-cai` which contains the frontend assets (e.g. HTML, JS, CSS).

### Test the dapp locally via the command line

Now that the canister is deployed to the local execution environment, you can interact with the canister by sending and receiving messages. Since the canister has a method called `greet` (which accepts a string as a parameter), we will send it a message. In terminal B, run

``` bash
dfx canister call hello_backend greet everyone
```

-   The `dfx canister call` command requires you to specify a canister name and function to call.
-   `hello_backend` specifies the name of the canister you call.
-   `greet` specifies the function name.
-   `everyone` is the argument that you pass to the `greet` function.

### Test the dapp locally via the browser

Now that you have verified that your dapp has been deployed and tested its operation using the command line, let’s verify that you can access the frontend using your web browser.

In terminal B, start the development server by running:

``` bash
npm start
```

<!-- ### 3.4.2 Test the dapp locally in the browser

To see your dapp running locally in the browser on http://localhost:8080. -->
<!--
Open a browser and navigate to <http://localhost:8080/>.

You should see a simple HTML page with a sample asset
image file, an input field, and a button.

![Sample HTML page](_attachments/frontend-prompt.png)

Type a greeting, then click **Click Me** to return the greeting.

![Hello](_attachments/frontend-result.png)

### Stop the local canister execution environment

After testing the application in the browser, you can stop the local canister execution environment so that it does not continue running in the background. We will not need it running to deploy on-chain.

To stop the local deployment:

1.  In terminal A, press Control-C to interrupt the local network process.
2.  In terminal B, press Control-C to interrupt the development server process.
3.  Stop the local canister execution environment running on your local computer:

    ``` bash
    dfx stop
    ```

### Troubleshooting

### Node.js is not properly installed

If your dapp does not show in the browser, it is possible that Node.js is not installed. Confirm it is installed by running:

``` bash
node --version
```

### Prior installations of dfx

If you have previously created IC dapps before February 2022, you may need to do a clean install. You can delete the SDK and associated profiles and re-install it. Follow the instructions here: [Install, upgrade, or remove software](../build/install-upgrade-remove.mdx).

## 4. Acquiring cycles to deploy on-chain (5 min)

In order to run on-chain, IC dapps require cycles to pay for computation and storage. This means that the developer needs to acquire cycles and fill their canister with them. Cycles are created from ICP tokens.

This flow may be surprising to people familiar with Web2 software where they can add a credit card to a hosting provider, deploy their apps, and get charged later. In Web3, blockchains require their smart contracts consume **something** (whether it is Ethereum’s gas or the IC’s cycles). The next steps will likely be familiar to those in crypto or blockchain, who grow used to the first step of deploying a dapp being "go get tokens."

You may further wonder why dapps run on cycles rather than ICP tokens. The reason is that the cost of ICP tokens fluctuate with the crypto market, but cycles are predictable and relatively stable tokens which are pegged to [SDR](https://en.wikipedia.org/wiki/Special_drawing_rights). One trillion cycles will always cost one SDR, regardless of the price of ICP.

Practical notes about cycles:

-   There is a [free cycles faucet](cycles-faucet.md) that grants new developers 20 trillion cycles
-   It takes 100 billion cycles to deploy a canister, but in order to load up the canister with sufficient cycles, `dfx` injects 3 trillion cycles for any canister created (this is a parameter that can be changed).
-   You can see a table of compute and storage costs here: [Computation and storage costs](../deploy/computation-and-storage-costs.md).
-   You can learn more about acquiring and managing ICP in [Acquiring and managing ICP tokens](https://wiki.internetcomputer.org/wiki/Tutorials_for_acquiring,_managing,_and_staking_ICP).

In this tutorial, we present two ways of acquiring cycles:

-   **Option 1:** [Acquiring cycles via the free cycles faucet](#option-1-acquiring-cycles-via-the-free-cycles-faucet-2-min) shows one how to get cycles via the cycles faucet (most common for new developers).
-   **Option 2:** [Converting ICP tokens into cycles](#option-2-converting-icp-tokens-into-cycles-5-min) shows one how to get cycles via ICP tokens (most common for developers who want more cycles).

By the end of this section, you will now have three canisters:

-   `hello_backend` canister (not yet deployed to the IC)
-   `hello_frontend` canister in your project (not yet deployed to the IC)
-   Your cycles wallet canister that holds your cycles (deployed on the IC)

![hello dapp and cycles wallet](_attachments/3-canisters-hello-dapp.png)

<!-- ### 4.2 Check the connection to the Internet Computer (terminal B) -->
<!--
As a sanity check, it is good practice to check if your connection to the IC is stable by verifying the current status of the Internet Computer blockchain and your ability to connect to it:

``` bash
dfx ping ic
```

If successful you will see an output resembling the following:

``` bash
$ {
  "ic_api_version": "0.18.0"  "impl_hash": "d639545e0f38e075ad240fd4ec45d4eeeb11e1f67a52cdd449cd664d825e7fec"  "impl_version": "8dc1a28b4fb9605558c03121811c9af9701a6142"  "replica_health_status": "healthy"  "root_key": [48, 129, 130, 48, 29, 6, 13, 43, 6, 1, 4, 1, 130, 220, 124, 5, 3, 1, 2, 1, 6, 12, 43, 6, 1, 4, 1, 130, 220, 124, 5, 3, 2, 1, 3, 97, 0, 129, 76, 14, 110, 199, 31, 171, 88, 59, 8, 189, 129, 55, 60, 37, 92, 60, 55, 27, 46, 132, 134, 60, 152, 164, 241, 224, 139, 116, 35, 93, 20, 251, 93, 156, 12, 213, 70, 217, 104, 95, 145, 58, 12, 11, 44, 197, 52, 21, 131, 191, 75, 67, 146, 228, 103, 219, 150, 214, 91, 155, 180, 203, 113, 113, 18, 248, 71, 46, 13, 90, 77, 20, 80, 95, 253, 116, 132, 176, 18, 145, 9, 28, 95, 135, 185, 136, 131, 70, 63, 152, 9, 26, 11, 170, 174]
}
```

### Option 1: Acquiring cycles via the free cycles faucet (2 min)

This option is best for people who want minimal time investment and have never used cycles faucet (faucet can be used only once).

For the purposes of this tutorial, you can acquire free cycles for your `Hello` dapp from the cycles faucet. Follow the instructions here: [Claim your free cycles](cycles-faucet.md).

#### Check your cycles balance

Now that you have used the cycles faucet, in terminal B you can check your cycles balance:

``` bash
dfx wallet --network ic balance
```

You should see around 20 trillion cycles if you run this after using the cycles wallet. If so, skip to section [5. Deploying on-chain](#5deploy-on-chain-1-min).

If you do not see any cycles, deploying on-chain in the rest of the tutorial will not work. You should try [Option 2: Converting ICP token into cycles](#option-2-converting-icp-tokens-into-cycles-5-min).

### Option 2: Converting ICP tokens into cycles (5 min)

This option is best for people who have already exhausted the cycles wallet or who want to set up their environment to add more cycles in the future.

To convert ICP tokens into cycles, you first need to obtain some ICP and transfer to the right account. You can get ICP tokens on exchanges, or ask someone you know to send you some. To figure out which account to transfer the ICP tokens to, run the following:

``` bash
dfx ledger account-id
```

This will display your account number on the ICP ledger. It looks similar to this:

```
e213184a548871a47fb526f3cba24e2ee2fbbc8129c4ab497ef2ce535130a0a4
```

Once you have transferred some ICP tokens into this account (5-10$ worth should be plenty to get going), you can see the balance using this command:

``` bash
dfx ledger --network ic balance
```

This will output something like this:

```
12.49840000 ICP
```

With those ICP tokens ready, you can start creating your cycles wallet. To start, you have to create a canister which will become your wallet. The base command for this is as follows:

``` bash
dfx ledger --network ic create-canister <your-principal-identifier> --amount <icp-tokens>
```

The two values you have to substitute are your own principal and the amount of tokens you want to convert. To figure out your own principal, use the output of `dfx identity get-principal`. If my principal is `tsqwz-udeik-5migd-ehrev-pvoqv-szx2g-akh5s-fkyqc-zy6q7-snav6-uqe` and I want to convert 2.3 ICP into cycles, the command looks like this:

``` bash
dfx ledger --network ic create-canister tsqwz-udeik-5migd-ehrev-pvoqv-szx2g-akh5s-fkyqc-zy6q7-snav6-uqe --amount 2.3
```

This command will take some time and output something similar to the following:

```
Transfer sent at BlockHeight: 351220
Canister created with id: "gastn-uqaaa-aaaae-aaafq-cai"
```

The id in this output is the address of the canister where your wallet will live. In this example, it would be `gastn-uqaaa-aaaae-aaafq-cai`.

Now that the canister is created, you can install the wallet code using this command:

``` bash
dfx identity --network ic deploy-wallet <canister-identifer>
```

Here, you have to substitute the canister identifier using the id you received in the output of the previous command. So, in the example this would look like this:

``` bash
dfx identity --network ic deploy-wallet gastn-uqaaa-aaaae-aaafq-cai
```

And the output should look like this:

```
Creating a wallet canister on the IC network.
The wallet canister on the "ic" network for user "default" is "gastn-uqaaa-aaaae-aaafq-cai"
```

Now your wallet should be configured and ready to go. To check if everything went right, run this to see the identifier of your configured wallet:

``` bash
dfx identity --network ic get-wallet
```

This should print the canister id you used in the commands earlier.

You can also check the balance of your new cycles wallet:

``` bash
dfx wallet --network ic balance
```

This should print something looking like this:

```
6.951 TC (trillion cycles).
```


## 5. Deploy on-chain (1 min)

Now that you have your [cycles](../../concepts/tokens-cycles.md) and your `dfx` is configured to transfer cycles, you are now ready to deploy your `hello` dapp on-chain. In terminal B, run:

``` bash
npm install
```

``` bash
dfx deploy --network ic --with-cycles 1000000000000
```

The `--network` option specifies the network alias or URL for deploying the dapp. This option is required to install on the Internet Computer blockchain mainnet. `--with-cycles` explicitly tells `dfx` how many cycles to use, otherwise it will use the default of 3 trillion.

If successful, your terminal should look like this:

``` bash
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
URLs:
  Frontend:
    hello_frontend: https://5h5yf-eiaaa-aaaaa-qaada-cai.ic0.app/
  Candid:
    hello_backend: https://a4gq6-oaaaa-aaaab-qaa4q-cai.raw.ic0.app/?id=5o6tz-saaaa-aaaaa-qaacq-cai
```

Note the bottom of the message which returns the URL where you can see your canister’s frontend deployed on-chain: <https://5h5yf-eiaaa-aaaaa-qaada-cai.ic0.app/>

In the example above, we created a `hello` dapp that is composed of:

1.  `hello_backend` canister `5o6tz-saaaa-aaaaa-qaacq-cai` which contains the backend logic.

2.  `hello_frontend` canister `5h5yf-eiaaa-aaaaa-qaada-cai` which contains the frontend assets (e.g. HTML, JS, CSS).

### See your dapp live on-chain via a browser

Navigate to and enter a name: <https://5h5yf-eiaaa-aaaaa-qaada-cai.ic0.app/>

Before your dapp loads, your browser will quickly show a message that reads: Installing "Internet Computer Validating Service Worker". This [service worker](https://developer.chrome.com/docs/workbox/service-worker-overview/) comes from the IC and it is used to make sure the web app the user sees is the correct, untampered frontend. Once loaded, your browser will cache the service worker and your web app will load much quicker.

![Hello](_attachments/service-worker.png)

After loading the service worker, your dapp will load:

![Hello](_attachments/frontend-result.png)

### Testing the on-chain dapp via the command line

Since the canister has a method called `greet` (which accepts a string as a parameter), we can send it a message via `dfx`.

``` bash
dfx canister --network ic call hello_backend greet '("everyone": text)'
```

Note the way the message is constructed:

-   `dfx canister --network ic call` is the setup for calling a canister on the IC.

-   `hello_backend greet` means we are sending a message to a canister named `hello` and evoking its `greet` method. `dfx` knows which `hello` canister (out of the many in the IC), one refers to because a mapping of `hello` to a canister id is stored locally in `.dfx/local/canister_ids.json`.

-   `'("everyone": text)'` is the parameter we are sending to `greet` (which accepts `Text` as its only input).

### Troubleshooting

#### 403 Error

If you receive a 403 error, it is possible the identity you are using does not have enough cycles. You should try the following to debug:

#### 1. Confirm you are using the identity you assume are using

``` bash
dfx identity whoami
```

#### 2. Confirm the identity you are using has enough cycles on-chain

``` bash
dfx wallet --network ic balance
```

#### 3. Try proxying through your wallet

Sometimes (especially when you created the canisters with `dfx` versions before 0.9.0) your wallet is set as your canister’s controller. To have your wallet be the source of the deployment instruction, add the flag `--wallet <insert-your-wallet-id-here>` to the deploy or call command.

If this works and you would like to add your own principal as a controller of the canister (so you don’t have to use the `--wallet` option anymore), you can run this:

``` bash
dfx canister --wallet "$(dfx identity get-wallet)" update-settings --all --add-controller "$(dfx identity get-principal)"
```

## Wrap-up

Congratulations! You have built a dapp fully on-chain (both backend and frontend) within 10 minutes.

Tutorial takeaways:
-   Dapps can be composed of multiple canisters
-   Dapps can be deployed locally and on-chain
-   Cycles are needed power dapps
-   Get free cycles from the cycles wallet
-   Free cycles can be used to power additional dapps

### Starting from scratch

If you wish to start from scratch, delete the SDK and associated profiles and re-install it. Follow the instructions here: [Install, upgrade, or remove software](../build/install-upgrade-remove.mdx).

**Be sure to save any identities linked to dapps or ICP.**

### Getting Stuck?

If you get stuck or run into problems search for solutions or post questions in the [Developer forum](https://forum.dfinity.org) or [DISCORD](https://discord.com/invite/cA7y6ezyE2).

### Ready for the next challenge?

Build DAOs, NFTs and more [here](https://smartcontracts.org/samples).

### Want to learn more?

If you are looking for more information before getting started or want to view a demonstration of how to deploy before you try it for yourself, check out the following related resources:

-   [How you can use ICP tokens (overview)](../../concepts/tokens-cycles#using-tokens)

-   [Building on the Internet Computer: Fundamentals (video)](https://www.youtube.com/watch?v=jduSMHxdYD8)

-   [What is the DFINITY Canister SDK (video)](https://www.youtube.com/watch?v=60uHQfoA8Dk)

-   [Deploying your first dapp (video)](https://www.youtube.com/watch?v=yqIoiyuGYNA)

-->
