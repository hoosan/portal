# ローカル環境での開発

この _クイックスタート_ では Dfinity Canister SDK を初めてインストールする方が、スマートコントラクトを Internet Computer のブロックチェーンにデプロイするのではなく、 **ローカルの実行環境** にデプロイし、動作させることを想定しています。

まず最初に `greet` という関数を 1 つだけ持つシンプルな Hello Canister をビルドしデプロイしてみます。 `greet` 関数は 1 つの引数を受けとり、その結果を **Hello, everyone!** のように返します。コマンドラインで Canister を実行した場合はターミナルに、ブラウザで Canister にアクセスした場合は HTML で表示されます。

## 始める前に

DFINITY Canister SDK をダウンロードしてインストールする前に、以下のことを確認してください。

- インターネットに接続されている **macOS** または **Linux** コンピュータでターミナルにアクセスできること。

  現在 DFINITY Canister SDK は、macOS または Linux の OS を搭載したコンピュータでのみ動作します。

- フロントエンド開発用のテンプレートファイルをプロジェクトに含めたい場合は、 `node.js` がインストールされている必要があります。

## ダウンロードとインストール

最新バージョンの SDK を、コンピュータのターミナルから直接ダウンロードすることができます。

ダウンロードしてインストールするには以下のようにします。

1.  ローカルコンピュータでターミナルを開きます。

    例えば、「アプリケーション」から「ユーティリティ」を開いて、「ターミナル」をダブルクリックするか、<span class="keycombo">⌘+スペースキー</span> を押して「検索」を開き、 `terminal` と入力します。

2.  以下のコマンドを実行して、SDK パッケージをダウンロードしてインストールします。

        sh -ci "$(curl -fsSL https://smartcontracts.org/install.sh)"

    このコマンドは、DFINITY コマンドラインインターフェース（CLI）及びその依存関係をローカルコンピュータにインストールする前に、使用許諾契約書を読み同意するよう促します。

3.  `y` と入力し、Return を押すと、インストールが続行されます。

    このコマンドは、コンピュータにインストールされているコンポーネントに関する情報を表示します。

## SDK が使える状態になっていることの確認

インストールスクリプトでエラーがなければ、プログラム開発を始めるために必要なすべてのものが、ローカルコンピュータ上で利用可能になります。

SDK を使用できることを確認するには以下のようにします。

1.  ローカルコンピュータでターミナルを開きます（まだ開いていない場合）。

2.  次のコマンドを実行して、DFINITY コマンドラインインターフェース（CLI）がインストールされ `dfx` コマンドが実行可能であることを確認します。

        dfx --version

    このコマンドは、`dfx` コマンドのバージョン情報を表示します。最新バージョンについては[リリースノート](../updates/release-notes/release-notes.md) から確認することができます。

3.  他の `dfx` コマンドの使用方法は、以下のコマンドを実行してください。

        dfx --help

    このコマンドは、 `dfx` とそのオプション情報を表示します。

## 新しいプロジェクトの作成

Internet Computer の Dapps 開発は**プロジェクト**として始めます。 プロジェクトは dfx コマンドで作成できます。

このチュートリアルでは、プロジェクトのスターターファイルを使ったアプリケーション作成について説明するために、デフォルトのサンプルアプリケーションの作成から始めます。 新しいプロジェクトを作成すると、`dfx` コマンドラインインターフェイスは、ワークスペースにデフォルトのプロジェクトディレクトリを追加します。プロジェクトディレクトリを構成するテンプレートファイルについては、[デフォルトプロジェクトの確認](../build/backend/explore-templates) のチュートリアルで説明しています。

新しいプロジェクトを作成する手順は以下になります。

1.  ローカルコンピュータでターミナルを開きます。

2.  以下のコマンドを実行し、 `hello` という名前の新しいプロジェクトを作成します。

        dfx new hello

    `dfx new hello` コマンドは、新しい `hello` プロジェクトディレクトリ、テンプレートファイル、そして新しい Git リポジトリを作成します。

    もし別のプロジェクト名を使った場合は、使った名前をメモしておいてください。以下の手順では、`hello` プロジェクト名の代わりにそのプロジェクト名を使用してください。

3.  以下のコマンドでプロジェクトディレクトリに移動します。

        cd hello

## ローカル環境での開発を始める

最初のプロジェクトをビルドする前に、Canister 実行環境に接続する必要があります。 ベスト・プラクティスとして、**2 つのターミナル**を開いておいてください。これにより、1 つのターミナルで Canister の動作を確認し、別のターミナルでプロジェクトを管理することができます。

ローカルでの Canister 実行環境の準備:

1.  ローカルコンピュータで新しいターミナルを開きます。

2.  必要に応じて、プロジェクトのルートディレクトリに移動します。

    これで、2 つのターミナルが開き、両方のターミナルでプロジェクト・ディレクトリがカレント・ワーキング・ディレクトリになっているはずです。

3.  2 つ目のターミナルで以下のコマンドを実行して、ローカルな Canister 実行環境を開始します。

        dfx start

    お使いのプラットフォームのセキュリティ設定によっては、警告が表示される場合があります。 ネットワーク接続を許可するかどうかのプロンプトが表示された場合は、**Allow** をクリックします。

4.  Canister の実行が表示されているターミナルを開いたままにして、最初のターミナルにフォーカスを切り替えます。

    残りの手順は、Canister の実行操作が表示されていないこちらのターミナルで行います。

## アプリケーションの登録とビルド、デプロイ

ローカルの Canister 実行環境に接続すると、アプリケーションの登録とビルド、デプロイが可能になります。

最初のアプリケーションをローカル環境にデプロイするには

1.  プロジェクトのルート・ディレクトリにいることを確認します。

2.  以下のコマンドを実行して、プロジェクト・ディレクトリで `node` モジュールが利用可能であることを確認します。

        npm install

    このステップについてさらに情報が欲しい場合は [node がプロジェクトで利用可能であることを確認する](../build/frontend/webpack-config#troubleshoot-node) を参照してください。

3.  以下のコマンドで、あなたの最初のアプリケーションを登録、ビルドそしてデプロイします。

        dfx deploy

    `dfx deploy` コマンドの出力には、実行した結果が表示されます。 例えば、このステップでは `hello` メインプログラムと `hello_assets` フロントエンド UI 用の 2 つの Canister 識別子を登録し、以下のようなインストール情報を表示します。

        Creating a wallet canister on the local network.
        The wallet canister on the "local" network for user "default" is "rwlgt-iiaaa-aaaaa-aaaaa-cai"
        Deploying all canisters.
        Creating canisters...
        Creating canister "hello"...
        "hello" canister created with canister id: "rrkah-fqaaa-aaaaa-aaaaq-cai"
        Creating canister "hello_assets"...
        "hello_assets" canister created with canister id: "ryjl3-tyaaa-aaaaa-aaaba-cai"
        Building canisters...
        Building frontend...
        Installing canisters...
        Creating UI canister on the local network.
        The UI canister on the "local" network is "r7inp-6aaaa-aaaaa-aaabq-cai"
        Installing code for canister hello, with canister_id rrkah-fqaaa-aaaaa-aaaaq-cai
        Installing code for canister hello_assets, with canister_id ryjl3-tyaaa-aaaaa-aaaba-cai
        Authorizing our identity (default) to the asset canister...
        Uploading assets to asset canister...
          /index.html 1/1 (573 bytes)
          /index.html (gzip) 1/1 (342 bytes)
          /index.js 1/1 (605692 bytes)
          /index.js (gzip) 1/1 (143882 bytes)
          /main.css 1/1 (484 bytes)
          /main.css (gzip) 1/1 (263 bytes)
          /sample-asset.txt 1/1 (24 bytes)
          /logo.png 1/1 (25397 bytes)
          /index.js.map 1/1 (649485 bytes)
          /index.js.map (gzip) 1/1 (149014 bytes)
        Deployed canisters.

    別の名前でプロジェクトを作成した場合、 `hello` や `hello_assets` ではなく、プロジェクトの名前になります。

    また、最初のデプロイ時には、 `dfx` は `default` Identity と、`default` Identity が管理するローカルの Cycle ウォレットを作成することに注意してください。 Cycle ウォレットは特別なタイプの Canister で、[Cycle](../../concepts/tokens-cycles) を他の Canister に転送することができます。

    **このサンプルアプリケーションをローカルにデプロイするためには**、デフォルトの開発者 Identity 、Cycle ウォレットの使用、Cycle の管理などについて特に知る必要はありません。これらのトピックについては後ほど説明しますが、今のところ、これらが自動的に作成されることを覚えておいてください。

4.  以下のコマンドを実行して、 `hello` Canister と定義済みの `greet` 関数を呼び出します。

        dfx canister call hello greet everyone

    このコマンドを詳しく見てみましょう。

    - `dfx canister call` コマンドでは、Canister 名と、呼び出すメソッドまたは関数を指定する必要があります。

    - `hello` は呼び出したい **Canister** の名前を指定します。

    - `greet` は `hello` Caniser で呼び出したい関数の名前を指定します。

    - `everyone` は `greet` 関数に渡したいテキスト型の引数です。

    ただし、別の名前でプロジェクトを作成した場合は、Canister 名がプロジェクト名と一致するので、コマンドラインを `hello` の代わりに使用した名前に合わせて変更する必要があります。

5.  コマンドが `greet` 関数の戻り値を表示することを確認してください。

    例:

        ("Hello, everyone!")

## フロントエンドアプリケーションのテスト

アプリケーションのデプロイとコマンドラインを使った動作テストが終わったので、Web ブラウザを使ってフロントエンドにアクセスできるかどうかを確認してみましょう。

1.  開発サーバーを `npm start` で起動します。

2.  ブラウザを開きます。

3.  <http://localhost:8080/> にアクセスします。

この URL にアクセスすると、サンプルのアセット画像ファイル、入力フィールド、ボタンを含むシンプルな HTML ページが表示されます。 例えば、以下のようになります。

\+ ![Sample HTML page](_attachments/frontend-prompt.png)

1.  挨拶を入力し、 **Click Me** をクリックすると挨拶が返ってきます。

    例:

    ![Hello](_attachments/frontend-result.png)

## ローカルの Canister 実行環境の停止

ブラウザでアプリケーションをテストした後は、ローカルの Canister 実行環境を停止して、バックグラウンドで実行し続けないようにします。

ローカルの実行環境を停止するには、以下の手順に従います。

1.  開発サーバが表示されているターミナルで、Control-C を押して開発サーバのプロセスを中断します。

2.  Canister 実行操作を表示しているターミナルで、Control-C を押してローカル・ネットワーク・プロセスを中断します。

3.  以下のコマンドを実行して、ローカル・コンピュータ上で動作している Canister 実行環境を停止します。

        dfx stop

## 次のステップ

このクイックスタートでは、独自の Dapps を開発するための基本的な流れを紹介するために、いくつかの重要なステップにのみ触れています。 他のドキュメントには、Motoko の使い方や Internet Computer ブロックチェーン上で動作する Dapps の開発方法を学ぶための、より詳細な例やチュートリアルなどもあります。

次のステップに進むために以下も参考にしてください。

- [Building on the IC](../build/) ローカルの Canister 実行環境を使用して、シンプルな Dapps を構築するためのチュートリアルです。

- [Convert ICP tokens to cycles](network-quickstart#convert-icp) Internet Computer ブロックチェーンへのアプリケーションのデプロイを可能にするために、ICP トークンを Cycle に変換します。

- [On-chain deployment](network-quickstart) Cycle を所持し、Internet Computer ブロックチェーンのメインネットにアプリケーションをデプロイします。

- [What is Candid?](../build/candid/candid-concepts.md) Candid インターフェース記述言語がどのようにサービスの相互運用性とコンポーザビリティを可能にするかを学びます。

- [Motoko at-a-glance](../build/cdks/motoko-dfinity/overview.md) Motoko についての機能と構文について学ぶことができます。

<!--
# Local development

This *Quick Start* scenario assumes that you are installing the SDK for the first time and want to run a canister in a **local canister execution environment** instead of deploying it to the Internet Computer blockchain.

To get started, let’s build and deploy a simple Hello canister that has just one function—called `greet`. The `greet` function accepts one text argument and returns the result with a greeting similar to **Hello, everyone!** in a terminal if you run the canister using the command-line or in an HTML page if you access the canister in a browser.

## Before you begin

Before you download and install this release of the SDK, verify the following:

-   You have an internet connection and access to a shell terminal on your local **macOS** or **Linux** computer.

    Currently, the SDK only runs on computers with a macOS or Linux operating system.

-   You have `node.js` installed if you want to include the default template files for frontend development in your project.

## Download and install

You can download the latest version of the SDK directly from within a terminal shell on your local computer.

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

Dapps on the Internet Computer start as **projects**. You create projects using the `dfx` parent command and its subcommands.

For this tutorial, we’ll start with the default sample dapp to illustrate creating dapp using the starter files in a project. When you create a new project, the `dfx` command-line interface adds a default project directory structure to your workspace. We cover the template files that make up a project directory in the [Explore the default project](../build/backend/explore-templates.md) tutorial.

To create a new project for your first application:

1.  Open a terminal shell on your local computer, if you don’t already have one open.

2.  Create a new project named `hello` by running the following command:

        dfx new hello

    The `dfx new hello` command creates a new `hello` project directory, template files, and a new `hello` Git repository for your project.

    If you use a different project name instead of `hello`, make note of the name you used. You’ll need to use that project name in place of the `hello` project name throughout these instructions.

3.  Change to your project directory by running the following command:

        cd hello

## Start the local deployment

Before you can build your first project, you need to connect to the local canister execution environment. As a best practice, this step requires you to have **two terminal shells** open, so that you can start and see canister execution operations in one terminal and manage your project in another.

To prepare the local canister execution environment:

1.  Open a new second terminal window or tab on your local computer.

2.  Navigate to the root directory for your project, if necessary.

    You should now have **two terminals** open with your **project directory** as your **current working directory** in both terminals.

3.  Start the local canister execution environment on your computer in your second terminal by running the following command:

        dfx start

    Depending on your platform and local security settings, you might see a warning displayed. If you are prompted to allow or deny incoming network connections, click **Allow**.

4.  Leave the terminal window that displays canister execution operations open and switch your focus to the first terminal window where you created your new project.

    You perform the remaining steps in the terminal that doesn’t display canister execution operations.

## Register, build, and deploy the application

After you connect to the local canister execution environment you can register, build, and deploy your dapp locally.

To deploy your first dapp locally:

1.  Check that you are still in the root directory for your project, if needed.

2.  Ensure that `node` modules are available in your project directory, if needed, by running the following command:

        npm install

    For more information about this step, see [Ensuring node is available in a project](../build/frontend/webpack-config.md#troubleshoot-node).

3.  Register, build, and deploy your first dapp by running the following command:

        dfx deploy

    The `dfx deploy` command output displays information about the operations it performs. For example, this step registers two identifiers—one for the `hello` main program and one for the `hello_assets` frontend user interface—and installation information similar to the following:

        Creating a wallet canister on the local network.
        The wallet canister on the "local" network for user "default" is "rwlgt-iiaaa-aaaaa-aaaaa-cai"
        Deploying all canisters.
        Creating canisters...
        Creating canister "hello"...
        "hello" canister created with canister id: "rrkah-fqaaa-aaaaa-aaaaq-cai"
        Creating canister "hello_assets"...
        "hello_assets" canister created with canister id: "ryjl3-tyaaa-aaaaa-aaaba-cai"
        Building canisters...
        Building frontend...
        Installing canisters...
        Creating UI canister on the local network.
        The UI canister on the "local" network is "r7inp-6aaaa-aaaaa-aaabq-cai"
        Installing code for canister hello, with canister_id rrkah-fqaaa-aaaaa-aaaaq-cai
        Installing code for canister hello_assets, with canister_id ryjl3-tyaaa-aaaaa-aaaba-cai
        Authorizing our identity (default) to the asset canister...
        Uploading assets to asset canister...
          /index.html 1/1 (573 bytes)
          /index.html (gzip) 1/1 (342 bytes)
          /index.js 1/1 (605692 bytes)
          /index.js (gzip) 1/1 (143882 bytes)
          /main.css 1/1 (484 bytes)
          /main.css (gzip) 1/1 (263 bytes)
          /sample-asset.txt 1/1 (24 bytes)
          /logo.png 1/1 (25397 bytes)
          /index.js.map 1/1 (649485 bytes)
          /index.js.map (gzip) 1/1 (149014 bytes)
        Deployed canisters.

    If you created a project with a different name, however, your canister names will match your project name instead of `hello` and `hello_assets`.

    You should also note that the **first time you deploy**, `dfx` creates a `default` identity and a local cycle wallet controlled by your `default` identity. A cycles wallet is a special type of canister that enables you to transfer [cycles](../../concepts/tokens-cycles.md) to other canisters.

    **To deploy this sample dapp locally**, you don’t need to know anything about your default developer identity, using a cycles wallet, or managing cycles. We’ll cover these topics later, but for now, just note that these are created for you automatically.

4.  Call the `hello` canister and the predefined `greet` function by running the following command:

        dfx canister call hello greet everyone

    Let’s take a closer look at this example command:

    -   The `dfx canister call` command requires you to specify a canister name and a method—or function—to call.

    -   `hello` specifies the name of the **canister** you want to call.

    -   `greet` specifies the name of the **function** you want to call in the `hello` canister.

    -   `everyone` is the text data type argument that you want to pass to the `greet` function.

    Remember, however, that if you created a project with a different name, the canister name will match your project name and you’ll need to modify the command line to match the name you used instead of `hello`.

5.  Verify the command displays the return value of the `greet` function.

    For example:

        ("Hello, everyone!")

## Test the dapp frontend

Now that you have verified that your dapp has been deployed and tested its operation using the command line, let’s verify that you can access the frontend using your web browser.

1.  Start the development server with `npm start`

2.  Open a browser.

3.  Navigate to <http://localhost:8080/>

Navigating to this URL displays a simple HTML page with a sample asset image file, an input field, and a button. For example:

\+ ![Sample HTML page](_attachments/frontend-prompt.png)

1.  Type a greeting, then click **Click Me** to return the greeting.

    For example:

    ![Hello](_attachments/frontend-result.png)

## Stop the local canister execution environment

After testing the application in the browser, you can stop the local canister execution environment so that it doesn’t continue running in the background.

To stop the local deployment:

1.  In the terminal that displays the development server, press Control-C to interrupt the development server process.

2.  In the terminal that displays canister execution operations, press Control-C to interrupt the local network process.

3.  Stop the local canister execution environment running on your local computer by running the following command:

        dfx stop

## Next steps

This *Quick Start* touched on only a few key steps to introduce the basic workflow for developing dapps of your own. You can find more detailed examples and tutorials to help you learn about how to use Motoko and how to develop dapps to run on the Internet Computer blockchain throughout the documentation.

Here are some suggestions for where to go next:

-   [Building on the IC](../build/) to explore building simple dapps using a local canister execution environment.

-   [Convert ICP tokens to cycles](./network-quickstart.md#convert-icp) if you have ICP tokens that you want to convert to cycles to enable you to deploy dapp to the Internet Computer blockchain.

-   [On-chain deployment](./network-quickstart.md) if you have cycles and are ready to deploy an application to the Internet Computer blockchain mainnet.

-   [What is Candid?](../build/candid/candid-concepts.md) to learn how the Candid interface description language enables service interoperability and composability.

-   [Motoko overview](../build/cdks/motoko-dfinity/overview.md) to learn about the features and syntax for using Motoko.

-->
