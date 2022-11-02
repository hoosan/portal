# ネットワークへのデプロイ

この _クイックスタート_ では SDK を初めてインストールする方が、デフォルトのプロジェクトを Internet Computer ブロックチェーンのメインネットにデプロイすることを想定しています。

もしローカルの開発環境にデプロイする場合は、[ローカル環境での開発](local-quickstart) シナリオを参照してください。

まず最初に `greet` という関数を 1 つだけ持つシンプルな Hello Canister をビルドしデプロイしてみます。 `greet` 関数は 1 つの引数を受けとり、その結果を **Hello, everyone!** のように返します。コマンドラインで Canister を実行した場合はターミナルに、ブラウザで Canister にアクセスした場合は HTML で表示されます。

## 始める前に

SDK をダウンロードしてインストールする前に、以下のことを確認してください。

- インターネットに接続されている **macOS** または **Linux** コンピュータでターミナルにアクセスできること。

  現在 SDK は、macOS または Linux の OS を搭載したコンピュータでのみ動作します。

- フロントエンド開発用のテンプレートファイルをプロジェクトに含めたい場合は、 `node.js` がインストールされている必要があります。

- 利用可能な ICP トークンまたは Cycle を所持していること。

  このチュートリアルを完了するには、**Cycle** が必要です。Cycle を入手するには ICP トークンを Cycle に変換するか、別の開発者が管理する Canister や第三者の Cycle プロバイダなど、他のソースから Cycle を入手する必要があります。このチュートリアルでは、ICP トークンを持つ台帳アカウントがあることを前提に、ICP トークンを Cycle に変換し、その Cycle を自分の **Cycle ウォレット**に転送する方法を説明します。

  ICP トークンの入手については、[ICP トークンの入手方法](../../concepts/tokens-cycles#get-cycles) を参照してください。 ICP トークンを管理する Network Nervous System アプリケーションの使用方法については、[Network nervous system アプリケーション クイックスタート](../../tokenomics/token-holders/nns-app-quickstart)を参照してください。 作成したデフォルトの Cycle ウォレットの使用方法については、[デフォルトの Cycle ウォレットを使う](../build/project-setup/cycles-wallet) を参照してください。

## ダウンロードとインストール

最新の SDK は、コンピュータのターミナルから直接ダウンロードすることができます。既に SDK をインストールしている場合は、このセクションをスキップして [新しいプロジェクトの作成](#net-new-project) から始めてください。

ダウンロードしてインストールするには:

1.  ローカルコンピュータでターミナルを開きます。

    例えば、「アプリケーション」から「ユーティリティ」を開いて、「ターミナル」をダブルクリックするか、<span class="keycombo">⌘+スペースキー</span> を押して「検索」を開き、 `terminal` と入力します。

2.  以下のコマンドを実行して、SDK パッケージをダウンロードしてインストールします。

        sh -ci "$(curl -fsSL https://internetcomputer.org/install.sh)"

    このコマンドは、DFINITY コマンドラインインターフェース（CLI）及びその依存関係をローカルコンピュータにインストールする前に、使用許諾契約書を読み同意するよう促します。

3.  `y` と入力し、Return を押すと、インストールが続行されます。

    このコマンドは、コンピュータにインストールされているコンポーネントに関する情報を表示します。

## SDK が使用できることの確認

インストールでエラーが無ければ、プログラム開発を始めるための全てがローカルコンピュータ上で利用可能になります。

SDK を使用できることを確認するには:

1.  ローカルコンピュータでターミナルを開きます（まだ開いていない場合）。

2.  次のコマンドを実行して、DFINITY コマンドラインインターフェース（CLI）がインストールされ `dfx` コマンドが実行可能であることを確認します。

        dfx --version

    このコマンドは `dfx` コマンドのバージョン情報を表示します。最新バージョンは[リリースノート](../updates/release-notes/release-notes.md) から確認することができます。

3.  他の `dfx` コマンドの使用方法は、以下のコマンドを実行してください。

        dfx --help

    このコマンドは、 `dfx` とそのオプション情報を表示します。

## 新しいプロジェクトの作成

Internet Computer の Dapps 開発は**プロジェクト**として始めます。 プロジェクトは dfx コマンドで作成できます。

このチュートリアルでは、プロジェクトのスターターファイルを使ったアプリケーション作成について説明するために、デフォルトのサンプルアプリケーションの作成から始めます。 新しいプロジェクトを作成すると、`dfx` コマンドラインインターフェイスは、ワークスペースにデフォルトのプロジェクトディレクトリを追加します。プロジェクトディレクトリを構成するテンプレートファイルについては、[デフォルトプロジェクトの確認](../build/backend/explore-templates) で説明しています。

新しいプロジェクトを作成する手順:

1.  ローカルコンピュータでターミナルを開きます。

2.  以下のコマンドを実行し、 `hello` という名前の新しいプロジェクトを作成します。

        dfx new hello

    `dfx new hello` コマンドは、新しい `hello` プロジェクトディレクトリ、テンプレートファイル、そして新しい Git リポジトリを作成します。

    もし別のプロジェクト名を使った場合は、使った名前をメモしておいてください。以下の手順では、`hello` プロジェクト名の代わりにそのプロジェクト名を使用してください。

3.  以下のコマンドでプロジェクトディレクトリに移動します。

        cd hello

## IC メインネットへの接続性を確認する

Internet Computer ブロックチェーンのメインネットにアクセスするためのに、予約済みのネットワークエイリアスが用意されています。ネットワークエイリアスはシステムで定義されており、プロジェクトで設定する必要はありません。

IC への接続性を確認するには:

1.  プロジェクトのルートディレクトリにいることを確認します。

2.  ネットワークエイリアス `ic` に対して以下のコマンドを実行して、IC の状態と、それへの接続性を確認します。

        dfx ping ic

3.  `dfx ping ic` コマンドが IC の情報を返すことを確認します。

    例えば、以下のような出力が表示されるはずです。

        {
          "ic_api_version": "0.18.0"  "impl_hash": "d639545e0f38e075ad240fd4ec45d4eeeb11e1f67a52cdd449cd664d825e7fec"  "impl_version": "8dc1a28b4fb9605558c03121811c9af9701a6142"  "replica_health_status": "healthy"  "root_key": [48, 129, 130, 48, 29, 6, 13, 43, 6, 1, 4, 1, 130, 220, 124, 5, 3, 1, 2, 1, 6, 12, 43, 6, 1, 4, 1, 130, 220, 124, 5, 3, 2, 1, 3, 97, 0, 129, 76, 14, 110, 199, 31, 171, 88, 59, 8, 189, 129, 55, 60, 37, 92, 60, 55, 27, 46, 132, 134, 60, 152, 164, 241, 224, 139, 116, 35, 93, 20, 251, 93, 156, 12, 213, 70, 217, 104, 95, 145, 58, 12, 11, 44, 197, 52, 21, 131, 191, 75, 67, 146, 228, 103, 219, 150, 214, 91, 155, 180, 203, 113, 113, 18, 248, 71, 46, 13, 90, 77, 20, 80, 95, 253, 116, 132, 176, 18, 145, 9, 28, 95, 135, 185, 136, 131, 70, 63, 152, 9, 26, 11, 170, 174]
        }

## 開発者 Identity と Ledger アカウントの確認

全ての ICP トークンの取引は、インターネットコンピュータブロックチェーン上で動作する [Ledger Canister](../glossary#ledger) に記録されます。 Ledger Canister は、すべての ICP トークン保有者の **アカウント識別子** と **残高** で構成されています。

Ledger アカウントに保持している ICP トークンを転送するためには、安全かつ適切に署名したメッセージを Ledger に送信し、あなたの開発者 ID に取引権限を与える必要があります。

Ledger に接続し取引を行うために必要なハードウェアやソフトウェア、および手順は、ICP トークンの保管方法によって異なります。 例えば、ハードウェアウォレットやハードウェアセキュリティモジュール (HSM) アプライアンス、 Network Nervous System (NNS) フロントエンドアプリケーション、または SDK の `dfx` コマンドラインインターフェイスなど、様々な方法で Ledger に接続して取引できます。 これらはそれぞれ、署名されたメッセージの Ledger への送信や、アカウントホルダーとして ID を提示するためのインターフェースが異なります。

### 開発者 Identity について

初めて SDK を使用したとき、 `dfx` コマンドラインツールはあなたの `default` の開発者 ID を作成します。この ID は、 **プリンシパル** データタイプと、**プリンシパル ID** とも呼ばれるテキスト表現で表されます。この ID は、ビットコインやイーサリアムのアドレスに似ています。

しかし、開発者 ID に関連するプリンシパルは、通常、台帳の **アカウント識別子** とは同じではありません。プリンシパル ID とアカウント識別子は関連しており、どちらもあなたの ID をテキストで表現しますが、異なるフォーマットを使用しています。

### Ledger に接続しアカウント情報を得る

このチュートリアルでは、ハードウェアウォレットや Ledger に接続する外部アプリケーションを使わないことを想定しています。開発者 ID で Ledger のアカウント識別子を取得し、ICP トークンを Ledger のアカウント識別子から開発者 ID が管理する Cycle ウォレット Canister に転送します。

Ledger で自分のアカウントを調べるには:

1.  以下のコマンドを実行して、現在の開発者 ID を確認します。

        dfx identity whoami

    ほとんどの場合、`default` の開発者 ID を使用しているはずです。 例えば、以下のようになります。

        default

2.  以下のコマンドを実行して、ID に紐づくプリンシパルを表示します。

        dfx identity get-principal

    このコマンドを実行すると、以下のような出力が表示されます:

        tsqwz-udeik-5migd-ehrev-pvoqv-szx2g-akh5s-fkyqc-zy6q7-snav6-uqe

3.  以下のコマンドを実行して、開発者 ID のアカウント識別子を表示します。

        dfx ledger account-id

    このコマンドは、あなたの開発者 ID に関連付けられた元帳のアカウント識別子を表示します。 例えば、以下のような出力が表示されるはずです。

        03e3d86f29a069c6f2c5c48e01bc084e4ea18ad02b0eec8fccadf4487183c223

4.  次のコマンドを実行して、口座残高を確認します。

        dfx ledger --network ic balance

    このコマンドは、元帳アカウントから ICP トークンの残高を表示します。 例えば、以下のような出力が表示されます。

        10.00000000 ICP

## ICP トークンを Cycle に変換する

アカウント情報から現在の ICP トークンの残高が確認できました。ICP トークンの一部を Cycle に変換して、Cycle ウォレットに移動させてみましょう。

ICP トークンを転送して Cycle ウォレットを作成するには:

1.  以下のコマンドを実行して、台帳アカウントから ICP トークンを転送し、新しい Canister を作成します。

        dfx ledger --network ic create-canister <principal-identifier> --amount <icp-tokens>

    このコマンドは、 `--amount` 引数に指定した ICP トークンを Cycle に変換し、指定したプリンシパルが制御する新しい Canister に Cycle を関連付けます。

    例えば、以下のコマンドは、0.25 ICP トークンを Cycle に変換し、新しい Canister のコントローラーとしてデフォルト ID のプリンシパル識別子を指定します。

        dfx ledger --network ic create-canister tsqwz-udeik-5migd-ehrev-pvoqv-szx2g-akh5s-fkyqc-zy6q7-snav6-uqe --amount .25

    取引が成功すると、台帳にイベントが記録され、以下のような出力が表示されます。

        Transfer sent at BlockHeight: 20
        Canister created with id: "gastn-uqaaa-aaaae-aaafq-cai"

2.  以下のコマンドを実行して、新たに作成した Canister プレースホルダーに Cycle ウォレットコードをインストールします。

        dfx identity --network ic deploy-wallet <canister-identifer>

    例えば、:

        dfx identity --network ic deploy-wallet gastn-uqaaa-aaaae-aaafq-cai

    このコマンドを実行すると、以下のような出力が表示されます。:

        Creating a wallet canister on the ic network.
        The wallet canister on the "ic" network for user "default" is "gastn-uqaaa-aaaae-aaafq-cai"

## Cycle ウォレットの検証

ICP トークンを Cycle に変換した後、Cycle ウォレットの Canister を検証し、現在の Cycle 残高を確認することができます。

Cycle ウォレットを検証するには:

1.  以下のコマンドを実行して、導入した Cycle ウォレットの Canister 識別子を確認します。

        dfx identity --network ic get-wallet

    このコマンドは、Cycle ウォレットの Canister 識別子を、以下のように表示します。

        gastn-uqaaa-aaaae-aaafq-cai

2.  次のコマンドを実行して、 Cycle ウォレット Canister が正しく設定され、Cycle の残高があることを確認します。

        dfx wallet --network ic balance

    コマンドは Cycle ウォレットの残高を表示します。 例えば:

        15430122328028812 cycles.

    また、Web ブラウザで以下のような URL を使用して、デフォルトの Cycle ウォレットにアクセスすることもできます。

        https://<WALLET-CANISTER-ID>.raw.ic0.app

    初めてアプリケーションにアクセスする場合、匿名のデバイスを使用しているという通知が表示され、本人認証、ウォレットへのアクセス許可、デバイスの登録が求められます。

3.  **Authenticate** をクリックして、Internet Identity サービスに進みます。

4.  以前に ID を登録したことがある場合は **ユーザー番号** を入力し、新しいユーザーとしてサービスに登録します。

    Internet Identity サービスの詳細や、複数の認証デバイスや認証登録については、[Internet Identity サービスの使い方](../../tokenomics/identity-auth/auth-how-to) を参照してください。

5.  ユーザー番号と登録した認証方法（セキュリティキーや指紋認証など）を使って認証します。

6.  **進む** をクリックすると、デフォルトの Cycle ウォレットアプリケーションにアクセスします。

7.  **デバイスの登録** ページに表示されているコマンドをコピーし、ターミナルで実行して、このセッションで使用するデバイスを登録します。

    例えば、以下のようなコマンドで、Cycle ウォレット Canister の `authorize` メソッドを呼び出します。

        dfx canister --network ic call "gastn-uqaaa-aaaae-aaafq-cai" authorize '(principal "ejta3-neil3-qek6c-i7rdw-sxreh-lypfe-v6hjg-6so7x-5ugze-3iohr-2qe")'

    コマンドに正しいネットワーク（`ic`）のエイリアスが含まれていることを確認してください。 Canister 識別子（この例では、`gastn-uqaaa-aaae-aaafq-cai`）が、あなたの ID に関連する Cycle ウォレットである必要があります。 ただし、これが IC 上の初めてのウォレットである場合、認証されているプリンシパルを認識できない可能性があります。このような場合には、別のプリンシパルを使用してください。

    `authorize` コマンドの実行後にブラウザを更新すると、プリンシパルアカウントの Cycle ウォレットが表示されます。

8.  ブラウザで Cycle の残高とアクティビティを確認できます。

    For example:

    ![cycles wallet](_attachments/cycles-wallet.png)

    デフォルトの Cycle ウォレットで使用できるコマンドやメソッドの詳細については、[デフォルトの Cycle ウォレットを使う](../build/project-setup/cycles-wallet) を参照してください。

## アプリケーションの登録、ビルド、およびデプロイ

Cycle ウォレットの残高を確認したら、サンプルアプリケーションを登録、ビルド、デプロイすることができます。

最初のアプリケーションを Internet Computer ブロックチェーンのメインネットにデプロイするには:

1.  ターミナルで、プロジェクトのルートディレクトリにいることを確認します。

2.  以下のコマンドを実行して、プロジェクトディレクトリで `node` モジュールが利用可能であることを確認します。

        npm install

    この手順の詳細については、[プロジェクトでノードが利用可能であることを確認する](../build/frontend/webpack-config#troubleshoot-node) を参照してください。

3.  以下のコマンドを実行して、最初のアプリケーションを登録、ビルド、デプロイします。

        dfx deploy --network ic

    `--network` オプションは、アプリケーションをデプロイするためのネットワークエイリアスまたは URL を指定します。 このオプションは、Internet Computer ブロックチェーンのメインネットにインストールするために必要です。

    `dfx deploy` コマンドの出力には、実行した結果が表示されます。

    例えば、このステップでは、`hello` メインプログラム用と `hello_assets` フロントエンドユーザーインターフェース用の 2 つの識別子を登録し、以下のようなインストール情報を表示します。

        Deploying all canisters.
        Creating canisters...
        Creating canister "hello"...
        "hello" canister created on network "ic" with canister id: "5o6tz-saaaa-aaaaa-qaacq-cai"
        Creating canister "hello_assets"...
        "hello_assets" canister created on network "ic" with canister id: "5h5yf-eiaaa-aaaaa-qaada-cai"
        Building canisters...
        Building frontend...
        Installing canisters...
        Installing code for canister hello, with canister_id 5o6tz-saaaa-aaaaa-qaacq-cai
        Installing code for canister hello_assets, with canister_id 5h5yf-eiaaa-aaaaa-qaada-cai
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

    操作を完了させるのに十分な ICP トークンを Cycle に変換しなかった場合は、以下のようなコマンドを実行することで、Cycle ウォレットに Cycle を追加することができます。

        dfx ledger --network ic top-up gastn-uqaaa-aaaae-aaafq-cai --amount 1.005

    このコマンドは、追加の `1.005` の ICP トークンを、`gastn-uqaaa-aaaae-aaafq-cai` の Cycle ウォレット識別子の Cycle に変換します。 このコマンドは以下のような出力を返します。

        Transfer sent at BlockHeight: 81520
        Canister was topped up!

4.  以下のコマンドを実行して、`hello` Canister と、定義済みの `greet` 関数を呼び出します。

        dfx canister --network ic call hello greet '("everyone": text)'

    例を詳しく見てみましょう。

    - `--network ic` オプションは、呼び出したい Canister が `ic` に展開されることを示します。ネットワークエイリアスの `ic` は、Internet Computer ブロックチェーンのメインネットにアクセスするためのエイリアスです。

    - `--network ic` オプションは操作サブコマンドの前になければならず、この場合は `dfx canister call` コマンドとなります。

    - `hello` 引数は呼び出したい Canister の名前を指定します。

    - `greet` 引数は、`hello` Canister で呼び出したい関数の名前を指定します。

    - テキスト文字列 `everyone` は、`greet` 関数に渡したい引数です。

5.  コマンドが `greet` 関数の戻り値を表示していることを確認してください。

    例えば、以下のようになります。

        ("Hello, everyone!")

6.  `dfx wallet balance` コマンドを再実行するか、ブラウザを更新すると、新しい Cycle の残高と最近のアクティビティが表示されます。

## フロントエンドアプリケーションのテスト

アプリケーションがデプロイされたことを確認し、コマンドラインを使って動作をテストした後は、Web ブラウザを使ってフロントエンドにアクセスできるかどうかを確認しましょう。

フロントエンドアプリケーションにアクセスするには、次のようにします。

1.  ブラウザを開きます。

2.  識別子の `hello_assets` と接尾辞の `boundary.ic0.app` を組み合わせた URL を使って、フロントエンドアプリケーションにアクセスします。

    Canister 識別子をメモしていなかった場合は、以下のコマンドを実行して調べることができます。

        dfx canister --network ic id hello_assets

    例えば、完全な URL は以下のようになります。

        https://gsueu-yaaaa-aaaae-aaagq-cai.raw.ic0.app

    この URL にアクセスすると、テンプレートアプリケーションの HTML エントリーページが表示されます。 例えば、以下のようになります。

    ![プロンプトが表示されるHTMLページ](_attachments/net-frontend-prompt.png)

3.  挨拶文を入力し、「Click Me」をクリックすると、挨拶文が返されます。

## 次のステップ

Internet Computer ブロックチェーンにアプリケーションを展開する方法を身につけたので、自分で開発したプログラムを展開する準備ができました。

Motoko の使い方や Internet Computer ブロックチェーン向けの Dapps の開発方法を学ぶための、より詳細な例やチュートリアルがドキュメントの随所に掲載されています。

次のステップに進むために以下も参考にしてください。

- [Build on the IC](../build/) ローカルの Canister 実行環境を使用して、シンプルな Dapps を構築するためのチュートリアルです。

- [What is Candid?](../build/candid/candid-concepts.md) インターフェース記述言語がどのようにサービスの相互運用性とコンポーザビリティを可能にするかを学びます。

- [Motoko Overview](../build/cdks/motoko-dfinity/overview.md) Motoko についての機能と構文について学ぶことができます。

<!--
# Network deployment

This *Quick Start* scenario assumes that you are installing the SDK for the first time and deploying the default project on the Internet Computer blockchain mainnet.

If you are only deploying projects in a local development environment, see the [Local development](local-quickstart) scenario.

To get started, let’s build and deploy a simple Hello dapp that has just one function—called `greet`. The `greet` function accepts one text argument and returns the result with a greeting similar to **Hello, everyone!** in a terminal if you run the dapp using the command-line or in an HTML page if you access the dapp in a browser.

## Before you begin

Before you download and install this release of the SDK, verify the following:

-   You have an internet connection and access to a shell terminal on your local **macOS** or **Linux** computer.

    Currently, the SDK only runs on computers with a macOS or Linux operating system.

-   You have `node.js` installed if you want to access the default frontend for the default project.

-   You have ICP tokens or cycles available for you to use.

    You must have **cycles** available to complete this tutorial. To get cycles, you must either convert ICP tokens to cycles or be provided cycles from another source, for example, from a canister controlled by another developer or from a third-party cycles provider. This tutorial assumes that you have an account with ICP tokens available and illustrates how to convert ICP tokens into cycles and transfer those cycles to a **cycles wallet** that you control.

    For information about how to get ICP tokens, see [How you can get ICP tokens](../../concepts/tokens-cycles#get-cycles). For an introduction to using the Network Nervous System application to manage ICP tokens, see [Network nervous system dapp quick start](../../tokenomics/token-holders/nns-app-quickstart). For information about using your default cycles wallet after you have created it, see [Use the default cycles wallet](../build/project-setup/cycles-wallet).

## Download and install

You can download the latest version of the SDK directly from within a terminal shell on your local computer. If you have previously installed the SDK, you can skip this section and start with [Create a new project](#net-new-project).

To download and install:

1.  Open a terminal shell on your local computer.

    For example, open Applications, Utilities, then double-click **Terminal** or press <span class="keycombo">⌘+spacebar</span> to open Search, then type `terminal`.

2.  Download and install the SDK package by running the following command:

        sh -ci "$(curl -fsSL https://internetcomputer.org/install.sh)"

    This command prompts you to read and accept the license agreement before installing the DFINITY execution command-line interface (CLI) and its dependencies on your local computer.

3.  Type `y` and press Return to continue with the installation.

    The command displays information about the components being installed on the local computer.

## Verify the SDK is ready to use

If the installation script runs without any errors, everything you need to start developing programs that run on the IC will be available on your local computer.

To verify the SDK is ready to use:

1.  Open a terminal shell on your local computer, if you don’t already have one open.

2.  Check that you have the DFINITY execution command-line interface (CLI) installed and the `dfx` executable is available in your PATH by running the following command:

        dfx --version

    The command displays version information for the `dfx` command-line executable. You can see the latest version in the [release notes](../updates/release-notes/release-notes.md).

3.  Preview usage information for the other `dfx` command-line sub-commands by running the following command:

        dfx --help

    The command displays usage information for the `dfx` parent command and its subcommands.

## Create a new project

Dapps for the Internet Computer start as **projects**. You create projects using the `dfx` parent command and its subcommands.

For this tutorial, we’ll start with the default sample dapp to illustrate creating a dapp using the starter files in a project. When you create a new project, the `dfx` command-line interface adds a default project directory structure to your workspace. We cover the template files that make up a project directory in the [Explore the default project](../build/backend/explore-templates) tutorial.

To create a new project for your first dapp:

1.  Open a terminal shell on your local computer, if you don’t already have one open.

2.  Create a new project named `hello` by running the following command:

        dfx new hello

    The `dfx new hello` command creates a new `hello` project directory, template files, and a new `hello` Git repository for your project.

    If you use a different project name instead of `hello`, make note of the name you used. You’ll need to use that project name in place of the `hello` project name throughout these instructions.

3.  Change to your project directory by running the following command:

        cd hello

## Check the connection to the IC mainnet

There is a reserved network alias that you can use to access the Internet Computer blockchain mainnet. The network alias is a system setting that’s defined internally, so there’s nothing you need to configure in your projects by default.

To check your connection to the IC:

1.  Check that you are in the root directory for your project, if needed.

2.  Check the current status of the IC and your ability to connect to it by running the following command for the network alias `ic`:

        dfx ping ic

3.  Verify that the `dfx ping ic` command returns information about the IC.

    For example, you should see output similar to the following:

        {
          "ic_api_version": "0.18.0"  "impl_hash": "d639545e0f38e075ad240fd4ec45d4eeeb11e1f67a52cdd449cd664d825e7fec"  "impl_version": "8dc1a28b4fb9605558c03121811c9af9701a6142"  "replica_health_status": "healthy"  "root_key": [48, 129, 130, 48, 29, 6, 13, 43, 6, 1, 4, 1, 130, 220, 124, 5, 3, 1, 2, 1, 6, 12, 43, 6, 1, 4, 1, 130, 220, 124, 5, 3, 2, 1, 3, 97, 0, 129, 76, 14, 110, 199, 31, 171, 88, 59, 8, 189, 129, 55, 60, 37, 92, 60, 55, 27, 46, 132, 134, 60, 152, 164, 241, 224, 139, 116, 35, 93, 20, 251, 93, 156, 12, 213, 70, 217, 104, 95, 145, 58, 12, 11, 44, 197, 52, 21, 131, 191, 75, 67, 146, 228, 103, 219, 150, 214, 91, 155, 180, 203, 113, 113, 18, 248, 71, 46, 13, 90, 77, 20, 80, 95, 253, 116, 132, 176, 18, 145, 9, 28, 95, 135, 185, 136, 131, 70, 63, 152, 9, 26, 11, 170, 174]
        }

## Confirm your developer identity and ledger account

All ICP token transactions are recorded in a [ledger canister](../glossary#ledger) running on the Internet Computer blockchain. The ledger canister consists of **account identifiers** and **balances** for all ICP token holders.

Before you can transfer any ICP tokens you hold in your ledger account, you need to send a secure and properly-signed message that verifies your identity to the ledger and authorizes your developer identity to complete the transaction.

Depending on how you have set up custody for holding your ICP tokens, the hardware, software, and steps required to connect to the ledger and complete a transaction can vary. For example, you might connect to the ledger and start a transaction from a hardware wallet, using a hardware security module (HSM) appliance, through the Network Nervous System (NNS) frontend application, or using the SDK `dfx` command-line interface. Each approach presents a different interface for signing and sending messages to the ledger and representing your identity as an account holder.

### About your developer identity

The first time you use the SDK, the `dfx` command-line tool creates a `default` developer identity for you. This identity is represented by a **principal** data type and a textual representation of the principal often referred to as your **principal identifier**. This representation of your identity is similar to a Bitcoin or Ethereum address.

However, the principal associated with your developer identity is typically not the same as your **account identifier** in the ledger. The principal identifier and the account identifier are related—both provide a textual representation of your identity—but they use different formats.

### Connect to the ledger to get account information

For the purposes of this tutorial—where there’s no hardware wallet or external application to connect to the ledger—we’ll use your developer identity to retrieve your ledger account identifier, then transfer ICP tokens from the ledger account identifier to a cycles wallet canister controlled by your developer identity.

To look up your account in the ledger:

1.  Confirm the developer identity you are currently using by running the following command:

        dfx identity whoami

    In most cases, you should see that you are currently using default\` developer identity. For example:

        default

2.  View the textual representation of the principal for your current identity by running the following command:

        dfx identity get-principal

    This command displays output similar to the following:

        tsqwz-udeik-5migd-ehrev-pvoqv-szx2g-akh5s-fkyqc-zy6q7-snav6-uqe

3.  Get the account identifier for your developer identity by running the following command:

        dfx ledger account-id

    This command displays the ledger account identifier associated with your developer identity. For example, you should see output similar to the following:

        03e3d86f29a069c6f2c5c48e01bc084e4ea18ad02b0eec8fccadf4487183c223

4.  Check your account balance by running the following command:

        dfx ledger --network ic balance

    This command displays the ICP token balance from the ledger account. For example, you should see output similar to the following:

        10.00000000 ICP

## Creating a Cycles Wallet

Now that you have confirmed your account information and current ICP token balance, you can convert some of those ICP tokens to cycles and move them into a cycles wallet.

To transfer ICP tokens to create a cycles wallet:

1.  Create a new canister with cycles by transferring ICP tokens from your ledger account by running a command similar to the following:

        dfx ledger --network ic create-canister <principal-identifier> --amount <icp-tokens>

    This command converts the number of ICP tokens you specify for the `--amount` argument into cycles, and associates the cycles with a new canister identifier controlled by the principal you specify.

    For example, the following command converts .25 ICP tokens into cycles and specifies the principal identifier for the default identity as the controller of the new canister:

        dfx ledger --network ic create-canister tsqwz-udeik-5migd-ehrev-pvoqv-szx2g-akh5s-fkyqc-zy6q7-snav6-uqe --amount .25

    If the transaction is successful, the ledger records the event and you should see output similar to the following:

        Transfer sent at BlockHeight: 20
        Canister created with id: "gastn-uqaaa-aaaae-aaafq-cai"

2.  Install the cycles wallet code in the newly-created canister placeholder by running a command similar to the following:

        dfx identity --network ic deploy-wallet <canister-identifer>

    For example:

        dfx identity --network ic deploy-wallet gastn-uqaaa-aaaae-aaafq-cai

    This command displays output similar to the following:

        Creating a wallet canister on the IC network.
        The wallet canister on the "ic" network for user "default" is "gastn-uqaaa-aaaae-aaafq-cai"

## Validate your cycles wallet

After you convert ICP tokens to cycles, you can validate the cycles wallet canister and check your current cycles balance.

To validate your cycles wallet:

1.  Verify the canister identifier for the cycles wallet you deployed by running the following command:

        dfx identity --network ic get-wallet

    The command displays the canister identifier for your cycles wallet with output similar to the following:

        gastn-uqaaa-aaaae-aaafq-cai

2.  Check that your cycles wallet canister is properly configured and holds a balance of cycles by running a command similar to the following:

        dfx wallet --network ic balance

    The command returns the balance for the your cycles wallet. For example:

        15430.122 TC (trillion cycles).

    You can also access your default cycles wallet in a web browser by using a URL similar to the following:

        https://<WALLET-CANISTER-ID>.raw.ic0.app

    The first time you access the application, you see a notice that you are using an Anonymous Device and are prompted to authenticate your identity, authorize access to the wallet, and register your device.

3.  Click **Authenticate** to continue to the Internet Identity service.

4.  Enter your **User Number** if you have previously registered an identity or register with the service as a new user.

    For more information about the Internet Identity service and how to register multiple authentication devices and methods, see [How to use the Internet Identity service](../../tokenomics/identity-auth/auth-how-to).

5.  Authenticate using your user number and the authentication method—for example, a security key or fingerprint—you have registered.

6.  Click **Proceed** to access to the default cycles wallet application.

7.  Register the device you are using for this session by copying the command displayed in the **Register Device** page and running it in a terminal.

    For example, call the `authorize` method for the cycles wallet canister with a command similar to the following:

        dfx canister --network ic call "gastn-uqaaa-aaaae-aaafq-cai" authorize '(principal "ejta3-neil3-qek6c-i7rdw-sxreh-lypfe-v6hjg-6so7x-5ugze-3iohr-2qe")'

    Be sure that the command you copy has the correct network (`ic`) alias. You should recognize the canister identifier—in this example, `gastn-uqaaa-aaaae-aaafq-cai`—as the cycles wallet associated with your identity. If this is your first wallet on the IC, however, you might not recognize the principal being authorized. The use of a different principal is the expected behavior in this case.

    When the browser refreshes after running the `authorize` command, the cycles wallet for your principal account is displayed.

8.  View your cycles balance and activity in the browser.

    For example:

    ![cycles wallet](_attachments/cycles-wallet.png)

    For more information about the commands and methods available for working with the default cycles wallet, see [Use the default cycles wallet](../build/project-setup/cycles-wallet).

## Register, build, and deploy the application

After you have validated your cycles wallet balance, you can register, build, and deploy your sample application.

To deploy your first application on the Internet Computer blockchain mainnet:

1.  In your terminal shell, check that you are still in the root directory for your project.

2.  Ensure that `node` modules are available in your project directory, if needed, by running the following command:

        npm install

    For more information about this step, see [Ensuring node is available in a project](../build/frontend/webpack-config#troubleshoot-node).

3.  Register, build, and deploy your first application by running the following command:

        dfx deploy --network ic

    The `--network` option specifies the network alias or URL for deploying the dapp. This option is required to install on the Internet Computer blockchain mainnet.

    The `dfx deploy` command output displays information about the operations it performs.

    For example, this step registers two identifiers—one for the `hello` main program and one for the `hello_assets` frontend user interface—and installation information similar to the following:

        Deploying all canisters.
        Creating canisters...
        Creating canister "hello"...
        "hello" canister created on network "ic" with canister id: "5o6tz-saaaa-aaaaa-qaacq-cai"
        Creating canister "hello_assets"...
        "hello_assets" canister created on network "ic" with canister id: "5h5yf-eiaaa-aaaaa-qaada-cai"
        Building canisters...
        Building frontend...
        Installing canisters...
        Installing code for canister hello, with canister_id 5o6tz-saaaa-aaaaa-qaacq-cai
        Installing code for canister hello_assets, with canister_id 5h5yf-eiaaa-aaaaa-qaada-cai
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

    If you didn’t convert enough ICP tokens to cycles to complete the operation, you can add cycles to your cycles wallet by running a command similar to the following:

        dfx ledger --network ic top-up gastn-uqaaa-aaaae-aaafq-cai --amount 1.005

    This command converts an additional `1.005` ICP tokens to cycles for the `gastn-uqaaa-aaaae-aaafq-cai` cycles wallet identifier. The command returns output similar to the following:

        Transfer sent at BlockHeight: 81520
        Canister was topped up!

4.  Call the `hello` canister and the predefined `greet` function by running the following command:

        dfx canister --network ic call hello greet '("everyone": text)'

    Let’s take a closer look at this example:

    -   Using the `--network ic` option indicates that the canister you want to call is deployed on the `ic`. The `ic` network alias is an internally-reserved alias for accessing the Internet Computer blockchain mainnet.

    -   Note that the `--network ic` option must precede the operation subcommand, which, in this case, is the `dfx canister call` command.

    -   The `hello` argument specifies the name of the canister you want to call.

    -   The `greet` argument specifies the name of the function you want to call in the `hello` canister.

    -   The text string `everyone` is the argument that you want to pass to the `greet` function.

5.  Verify the command displays the return value of the `greet` function.

    For example:

        ("Hello, everyone!")

6.  Rerun the `dfx wallet balance` command or refresh the browser to see your new cycle balance and recent activity.

## Test the dapp frontend

Now that you have verified that your dapp has been deployed and tested its operation using the command line, let’s verify that you can access the frontend using your web browser.

To access the dapp frontend:

1.  Open a browser.

2.  Navigate to the frontend for the dapp using a URL that consists of the `hello_assets` identifier and the `boundary.ic0.app` suffix.

    If you didn’t make a note of the canister identifier, you can look it up by running the following command:

        dfx canister --network ic id hello_assets

    For example, the full URL should look similar to the following:

        https://gsueu-yaaaa-aaaae-aaagq-cai.raw.ic0.app

    Navigating to this URL displays the HTML entry page for the template application. For example:

    ![HTML page with prompt](_attachments/net-frontend-prompt.png)

3.  Type a greeting, then click **Click Me** to return the greeting.

## Next steps

Now that you have seen how to deploy a dapp on the Internet Computer blockchain, you are ready to develop and deploy programs of your own.

You can find more detailed examples and tutorials to help you learn about how to use Motoko and how to develop dapps for the Internet Computer blockchain throughout the documentation.

Here are some suggestions for where to go next:

-   [Build on the IC](../build/) to explore building frontend and backend dapps in a local development environment.

-   [What is Candid?](../build/candid/candid-concepts.md) to learn how the Candid interface description language enables service interoperability and composability.

-   [Motoko overview](../build/cdks/motoko-dfinity/overview.md) to learn about the features and syntax for using Motoko.

-->