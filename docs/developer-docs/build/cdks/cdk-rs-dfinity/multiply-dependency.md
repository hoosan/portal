# 基本的な依存関係

Dapp デザインのよくあるアプローチは、ある Canister スマートコントラクト（あるいは単に Canister と呼ぶ）でデータを計算または保存し、それを別の Canister で使用するというものです。 たとえ基盤となるスマートコントラクトが異なる言語で書かれていても、異なる Canister のスマートコントラクトで定義された関数を共有して使用できることは、Internet Computer blockchain で動作する Dapps を構築する際の重要な戦略となります。 このチュートリアルでは、ある言語（今回の例では Motoko）で関数を書き、そのデータを別の言語（ここでは Rust）で使用する方法を紹介します。

両方の Canister スマートコントラクトは同じプロジェクトに含まれています。

- Motoko Canister は オペレーションの結果を保存するための `cell` 変数を持つ Actor を作成します。

- `mul` 関数は自然数を入力として受け取り、入力値を３倍にして、結果を `cell` 変数に格納します。

- Rust Canister は `cell` 変数の現在の値を返すシンプルな `read` 関数を持ちます。呼び出されるメソッドが `query` であっても、Canister 間の呼び出しを行う関数は `update` メソッドであるべきであることに注意してください。

## はじめる前に

プロジェクトをはじめる前に、以下を確認してください:

- インターネットに接続しており、ローカルの macOS または Linux コンピュータでターミナルにアクセスできること。

- [Rust のインストール方法](https://doc.rust-lang.org/book/ch01-01-installation.html) にあるように、Rust プログラミング言語と Cargo が OS にダウンロードされ、インストールされていること。

  ```bash
  curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
  ```

  Rust のバージョンは 1.46.0 より新しい必要があります。

- [ダウンロードとインストール](../../../quickstart/local-quickstart#download-and-install) の説明に従って、DFINITY Canister Software Development Kit (SDK) パッケージのダウンロードとインストールが済んでいること。

- `cmake` のインストールが済んでいること。例えば、macOS では Homebrew を使って以下のコマンドを実行します:

  ```bash
  brew install cmake
  ```

  Homebrew をインストールする方法については、[Homebrew のドキュメント](https://docs.brew.sh/Installation)を参照してください。

- コンピュータ上の ローカル Canister 実行環境 プロセスが停止していること。

このチュートリアルの所要時間は約 20 分です。

## 新しいプロジェクトの作成

新しいプロジェクトを作る手順は以下になります:

1.  ローカル PC でターミナルを開きます。

2.  以下のコマンドを実行し、新しいプロジェクトを作成します。

    ```bash
    dfx new --type=rust rust_deps
    ```

3.  以下のコマンドで、プロジェクトディレクトリに移動します。

    ```bash
    cd rust_deps
    ```

## デフォルトプロジェクトの変更

このチュートリアルを完了するためには、以下の手順を踏む必要があります。

- [Canister の初期設定の変更](#_edit_the_default_canister_settings)

- [Motoko Canister スマートコントラクトの実装](#_implement_motoko_canister_smart_contract)

- [デフォルトの Rust Canister スマートコントラクトの置き換え](#_replace_the_default_rust_canister_smart_contract)

- [インターフェース記述ファイルの更新](#_update_interface_description_file)

### Canister の初期設定の変更

このサンプルプロジェクトは、Motoko Canister と Rust Canister の２つの Canister で構成されているため、デフォルトの `dfx.json` 設定ファイルを変更し、Motoko Canister と Rust Canister の両方をビルドするための情報を含める必要があります。

`dfx.json` を変更するには、以下のようにします:

1.  自分がプロジェクトのルートディレクトリにいることを確認します。

2.  テキストエディタで `dfx.json` を開きます。

3.  `canisters.rust_deps` セクションの上に、Motoko プログラムをビルドするための設定を挿入します。

    例えば、`canisters` セクションの中で、新しい `multiply_deps` キーを以下のように追加します。

    ```json
    "multiply_deps": {
      "main": "src/multiply_deps/main.mo",
      "type": "motoko"
    }
    ```

4.  `rust_deps` に `dependencies` の設定を追加します。

    `dependencies` の設定では、ある Canister から関数をインポートし、別の Canister で使用することができます。 このチュートリアルでは、Motoko で書かれた `multiply_deps` Canister から関数をインポートし、Rust で書かれた `rust_deps` Canister から使用することを考えます。 `+rust_deps` のフィールドは以下のようになります。

    ```json
    "rust_deps": {
      "candid": "src/rust_deps/rust_deps.did",
      "package": "rust_deps",
      "type": "rust",
      "dependencies": [
        "multiply_deps"
      ]
    }
    ```

5.  ファイルから `rust_deps_assets` の設定をすべて削除します。

    このチュートリアルの Dapp では、フロントエンドの `asset` を使用していないので、設定ファイルからこれらの設定を削除することができます。

    また、このチュートリアルでは `defaults` と `dfx` のキーも削除することができます。

    これらの不要な設定を削除した後の設定ファイルは[こちら](../../_attachments/mul-deps-dfx.json) のようになります:

6.  変更内容を保存し、`dfx.json` ファイルを閉じて次に進みます。

### Motoko Canister スマートコントラクトの実装

次のステップでは、`src/multiply_deps/main.mo` にファイルを作成して、`mul` と `read` 関数を実装するコードを作成します。

Motoko のソースコードを書くには以下のようにします。

1.  Motoko Canister のためのディレクトリを作成します。

    ```bash
    mkdir multiply_deps
    ```

2.  `src/multiply_deps/main.mo` ファイルを作成し、テキストエディタで開きます。

3.  [こちら](../../_attachments/mul-deps.mo) のサンプルコードを `main.mo` ファイル内にコピー＆ペーストします。

4.  変更を保存してファイルを閉じ、次に進みます。

### デフォルトの Rust Canister スマートコントラクトの置き換え

これで Rust Canister が依存する Motoko Canister ができたので、Rust Canister をプロジェクトに追加してみましょう。

デフォルトの Rust Canister を置き換えるには以下のようにします：

1.  自分がルートディレクトリにいることを確認します。

2.  `src/rust_deps/lib.rs` ファイルをテキストエディタで開き、内容を削除します。

    次のステップは、Motoko Canister をインポートし、`read` 関数を実装した Rust プログラムを書くことです。

3.  `lib.rs` ファイルに[こちら](../../_attachments/mul-deps-main.rs) のコードをコピー＆ペーストします：

4.  `src/rust_deps/lib.rs` を保存して閉じ、先に進みます。

### インターフェース記述ファイルの更新

Candid は、Internet Computer blockchain で動作する Canister と対話するためのインターフェース記述言語（IDL）です。 Candid ファイルは、Canister が定義する各関数の名前・引数・返し値のフォーマットやデータ型など、Canister のインターフェースを言語に依存しないように記述したものです。

Candid ファイルをプロジェクトに追加することで、Rust で定義されたデータが Internet Computer blockchain 上で安全に実行されるために適切に変換されることを保証します。

Candid インターフェース記述言語の構文の詳細は [_Candid ガイド_](../candid/candid-intro) か [Candid クレートのドキュメント](https://docs.rs/candid/)をご覧ください。

Candid ファイルを更新するには、以下のようにします。

1.  自分がプロジェクトのルートディレクトリにいることを確認します。

2.  `src/rust_deps/rust_deps.did` ファイルをテキストエディタで開きます。

3.  `read` 関数のために、以下のように `service` を定義します:

    ```did
    service : {
      "read": () -> (nat);
    }
    ```

4.  変更を保存して `deps.did` ファイルを閉じ、次に進んでください。

## ローカル Canister 実行環境 を立ち上げる

プロジェクトをビルドする前に、ローカル Canister 実行環境 か、Internet Computer blockchain メインネットに接続する必要があります。

ローカル Canister 実行環境 を立ち上げるには、以下のようにします:

1.  自分がプロジェクトのルートディレクトリにいることを確認します。

2.  ローカル Canister 実行環境 をバックグラウンドで立ち上げるために、以下のコマンドを実行します:

    ```bash
    dfx start --clean --background
    ```

    プラットフォームやローカルのセキュリティ設定によっては、警告が表示される場合があります。 ネットワーク接続を許可するかどうかの確認画面が表示された場合は、**Allow** をクリックします。

## プロジェクトの登録・ビルド・デプロイ

開発環境で立ち上がっている ローカル Canister 実行環境 に接続した後に、プロジェクトの登録・ビルド・デプロイをローカル環境で行うことができます。 登録・ビルド・デプロイを行うためには、以下のようにします:

1.  プロジェクトのルートディレクトリにいることを確認します。

2.  `dfx.json` に指定されている Canister を以下のコマンドで、登録・ビルド・デプロイします:

    ```bash
    dfx deploy
    ```

    `dfx deploy` コマンドを実行すると、以下のような実行結果に関しての情報が表示されます。

        Creating a wallet canister on the local network.
        The wallet canister on the "local" network for user "default" is "rwlgt-iiaaa-aaaaa-aaaaa-cai"
        Deploying all canisters.
        Creating canisters...
        Creating canister "multiply_deps"...
        "multiply_deps" canister created with canister id: "rrkah-fqaaa-aaaaa-aaaaq-cai"
        Creating canister "rust_deps"...
        "rust_deps" canister created with canister id: "ryjl3-tyaaa-aaaaa-aaaba-cai"
        Building canisters...
        Executing: "cargo" "build" "--target" "wasm32-unknown-unknown" "--release" "-p" "rust_deps"
        ...
            Finished release [optimized] target(s) in 5.26s
        Executing: ic-cdk-optimizer -o target/wasm32-unknown-unknown/release/rust_deps.wasm target/wasm32-unknown-unknown/release/rust_deps.wasm
        Installing canisters...
        Creating UI canister on the local network.
        The UI canister on the "local" network is "r7inp-6aaaa-aaaaa-aaabq-cai"
        Installing code for canister multiply_deps, with canister_id rrkah-fqaaa-aaaaa-aaaaq-cai
        Installing code for canister rust_deps, with canister_id ryjl3-tyaaa-aaaaa-aaaba-cai
        Deployed canisters.

## デプロイした Canister の関数を呼ぶ

Canister のデプロイが成功すると、Canister の関数を呼ぶことでテストできるようになります。

このチュートリアルでは、以下のようにします：

- `mul` 関数を呼ぶたびに、`cell` 変数の値を 3 倍にします。

- `read` 関数を呼び出し、`cell` 変数の現在の値を返します。

デプロイした Canister をテストするために：

1.  デプロイした Canister 上の `cell` 変数の値を読み取るために Motoko Canister から `read` 関数を呼び出します。

    ```bash
    dfx canister call multiply_deps read
    ```

    このコマンドは、変数 `cell` の現在の値である１を返します。

        (1 : nat)

2.  次のコマンドを実行して、入力された引数を３倍する `mul` 関数を呼び出します。

    ```bash
    dfx canister call multiply_deps mul '(3)'
    ```

    このコマンドは、変数 `cell` の新しい値を返します。

        (9 : nat)

3.  `multiply_deps` Canister から関数をインポートしている `rust_deps` Canister の `read` 関数を呼び出します。

    ```bash
    dfx canister call rust_deps read
    ```

    このコマンドは、変数 `cell` の現在の値を返します。

        (9 : nat)

## ローカル Canister 実行環境 を止める

アプリケーションのテストをした後は、ローカル Canister 実行環境 がバックグラウンドで稼働し続けないように、以下の手順で停止します：

1.  ネットワークの稼働状況が表示されている端末で、Control-C を押して ローカル Canister 実行環境 のプロセスを止めてください。

2.  以下のコマンドを用いて ローカル Canister 実行環境 を停止してください：

    ```bash
    dfx stop
    ```

<!--
# Basic dependency

One common approach to dapp design is to calculate or store data in one canister – or canister for short – that you can then use in another canister. This ability to share and use functions defined in different canisters, even if the underlying smart contracts are written in different languages, is an important strategy for building dapps to run the Internet Computer blockchain. This tutorial provides a basic introduction to how you can write functions in one language—in the example, Motoko—then use the data in another—in this case, Rust.

For this tutorial, both canisters are in the same project.

-   The Motoko canister creates an actor with a `cell` variable to contain the current value that results from an operation.

-   The `mul` function takes a natural number as input, multiplies the input value by three and stores the result in the `cell` variable.

-   The Rust canister provides a simple `read` function that returns the current value of the `cell` variable. Notice that the function making inter-canister call should be a `update` method even though the method to be called is `query`.

## Before you begin

Before you start your project, verify the following:

-   You have an internet connection and access to a shell terminal on your local macOS or Linux computer.

-   You have downloaded and installed the Rust programming language and Cargo as described in the [Rust installation instructions](https://doc.rust-lang.org/book/ch01-01-installation.html) for your operating system.

    ``` bash
    curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
    ```

    The Rust tool chain must be at version 1.46.0, or later.

-   You have downloaded and installed the DFINITY Canister Software Development Kit (SDK) package as described in [Download and install](../../../quickstart/local-quickstart.md#download-and-install).

-   You have `cmake` installed. For example, use Homebrew with the following command:

    ``` bash
    brew install cmake
    ```

    For instructions on how to install Homebrew, see the [Homebrew Documentation](https://docs.brew.sh/Installation).

-   You have stopped any local canister execution environment processes running on your computer.

This tutorial takes approximately 20 minutes to complete.

## Create a new project

To create a new project directory for this tutorial:

1.  Open a terminal shell on your local computer, if you don’t already have one open.

2.  Create a new project by running the following command:

    ``` bash
    dfx new --type=rust rust_deps
    ```

3.  Change to your project directory by running the following command:

    ``` bash
    cd rust_deps
    ```

## Modify the default project

To complete this tutorial, you’ll need to complete the following steps:

-   [Edit the default canister settings](#_edit_the_default_canister_settings)

-   [Implement Motoko canister](#_implement_motoko_canister_smart_contract)

-   [Replace the default Rust canister](#_replace_the_default_rust_canister_smart_contract)

-   [Update interface description file](#_update_interface_description_file)

### Edit the default canister settings

Because this sample project is going to consist of two canisters-the Motoko canister and the Rust canister—you need to modify the default `dfx.json` configuration file to include information for building both the Motoko canister and a Rust canister.

To modify the `dfx.json` configuration file:

1.  Check that you are still in the root directory for your project, if needed.

2.  Open the `dfx.json` configuration file in a text editor.

3.  Insert a new section before the `canisters.rust_deps` settings with settings for building a Motoko program.

    For example, in the `canisters` section, add a new `multiply_deps` key with settings like these:

    ``` json
    "multiply_deps": {
      "main": "src/multiply_deps/main.mo",
      "type": "motoko"
    }
    ```

4.  Add a `dependencies` setting to the `rust_deps` .

    The `dependencies` setting enables you to import functions from one canisters for use in another canister. For this tutorial, we want to import a function from the `multiply_deps` canister—written in Motoko—and use it from the `rust_deps` canister written in Rust. Then the `+rust_deps` field will look like:

    ``` json
    "rust_deps": {
      "candid": "src/rust_deps/rust_deps.did",
      "package": "rust_deps",
      "type": "rust",
      "dependencies": [
        "multiply_deps"
      ]
    }
    ```

5.  Remove all of the `rust_deps_assets` configuration settings from the file.

    The sample dapp for this tutorial doesn’t use any frontend assets, so you can remove those settings from the configuration file.

    You can also remove the `defaults` and `dfx` version settings.

    For example, your configuration file might look like [this](../../_attachments/mul-deps-dfx.json) after you modify the settings.

6.  Save your change and close the `dfx.json` file to continue.

### Implement Motoko canister

The next step is to create a file at `src/multiply_deps/main.mo` with code that implements the `mul` and `read` functions.

To write the Motoko source code:

1.  Check that you are still in the root directory for your project, if needed.

2.  Create the directory for the Motoko canister.

    ``` bash
    mkdir multiply_deps
    ```

3.  Create and open the `src/multiply_deps/main.mo` file in a text editor.

4.  Copy and paste [this code](../../_attachments/mul-deps.mo) into the `main.mo` file.

5.  Save your changes and close the file to continue.

### Replace the default Rust canister

Now that we have the Motoko canister that the Rust canister depends upon, let’s add the Rust canister to the project.

To replace the default Rust canister:

1.  Check that you are still in the root directory for your project, if needed.

2.  Open the template `src/rust_deps/lib.rs` file in a text editor and delete the existing content.

    The next step is to write a Rust program that imports the Motoko canister and implements the `read` function.

3.  Copy and paste [this code](../../_attachments/mul-deps-main.rs) into the `lib.rs` file.

4.  Save your changes and close the `src/rust_deps/lib.rs` file to continue.

### Update interface description file

Candid is an interface description language (IDL) for interacting with canisters running on the Internet Computer blockchain. Candid files provide a language-independent description of a canister’s interfaces including the names, parameters, and result formats and data types for each function a canister defines.

By adding Candid files to your project, you can ensure that data is properly converted from its definition in Rust to run safely on the Internet Computer blockchain.

To see details about the Candid interface description language syntax, see the [*Candid Guide*](../../candid/candid-intro.md) or the [Candid crate documentation](https://docs.rs/candid/).

To update the Candid file for this tutorial:

1.  Check that you are still in the root directory for your project, if needed.

2.  Open the `src/rust_deps/rust_deps.did` file in a text editor.

3.  Copy and paste the following `service` definition for the `read` function:

    ``` did
    service : {
      "read": () -> (nat);
    }
    ```

4.  Save your changes and close the `deps.did` file to continue.

## Start the local canister execution environment

Before you can build the project, you need to connect to the local canister execution environment in your development environment or the Internet Computer blockchain mainnet.

To start the network locally:

1.  Check that you are still in the root directory for your project, if needed.

2.  Start the local canister execution environment on your computer in the background by running the following command:

    ``` bash
    dfx start --clean --background
    ```

    Depending on your platform and local security settings, you might see a warning displayed. If you are prompted to allow or deny incoming network connections, click **Allow**.

## Register, build, and deploy your project

After you connect to the local canister execution environment running in your development environment, you can register, build, and deploy your project locally.

To register, build, and deploy:

1.  Check that you are still in root directory for your project directory, if needed.

2.  Register, build, and deploy the canisters specified in the `dfx.json` file by running the following command:

    ``` bash
    dfx deploy
    ```

    The `dfx deploy` command output displays information about each of the operations it performs similar to the following excerpt:

        Creating a wallet canister on the local network.
        The wallet canister on the "local" network for user "default" is "rwlgt-iiaaa-aaaaa-aaaaa-cai"
        Deploying all canisters.
        Creating canisters...
        Creating canister "multiply_deps"...
        "multiply_deps" canister created with canister id: "rrkah-fqaaa-aaaaa-aaaaq-cai"
        Creating canister "rust_deps"...
        "rust_deps" canister created with canister id: "ryjl3-tyaaa-aaaaa-aaaba-cai"
        Building canisters...
        Executing: "cargo" "build" "--target" "wasm32-unknown-unknown" "--release" "-p" "rust_deps"
        ...
            Finished release [optimized] target(s) in 5.26s
        Executing: ic-cdk-optimizer -o target/wasm32-unknown-unknown/release/rust_deps.wasm target/wasm32-unknown-unknown/release/rust_deps.wasm
        Installing canisters...
        Creating UI canister on the local network.
        The UI canister on the "local" network is "r7inp-6aaaa-aaaaa-aaabq-cai"
        Installing code for canister multiply_deps, with canister_id rrkah-fqaaa-aaaaa-aaaaq-cai
        Installing code for canister rust_deps, with canister_id ryjl3-tyaaa-aaaaa-aaaba-cai
        Deployed canisters.

## Call functions on the deployed canister

After successfully deploying the canister, you can test the canister by invoking the functions it provides.

For this tutorial:

-   Call the `mul` function to multiply the value of the `cell` variable by three each time it is called.

-   Call the `read` function to return the current value of the `cell` variable.

To test the deployed canister:

1.  Call the `read` function from the Motoko canister, which reads the current value of the `cell` variable on the deployed canister:

    ``` bash
    dfx canister call multiply_deps read
    ```

    The command returns the current value of the `cell` variable as one:

        (1 : nat)

2.  Call the `mul` function to multiply the input argument by three by running the following command:

    ``` bash
    dfx canister call multiply_deps mul '(3)'
    ```

    The command returns the new value of the `cell` variable:

        (9 : nat)

3.  Call the `read` function using the `rust_deps` canister that imports functions from the `multiply_deps` canister:

    ``` bash
    dfx canister call rust_deps read
    ```

    The command returns the current value of the `cell` variable:

        (9 : nat)

## Stop the local canister execution environment

After you finish experimenting with your dapp, you can stop the local canister execution environment so that it doesn’t continue running in the background.

To stop the local canister execution environment:

1.  In the terminal that displays network operations, press Control-C to interrupt the local canister execution environment process.

2.  Stop the local canister execution environment by running the following command:

    ``` bash
    dfx stop
    ```

-->
