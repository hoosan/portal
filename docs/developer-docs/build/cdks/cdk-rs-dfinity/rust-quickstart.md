# Hello World! Rust CDK クイックスタート

DFINITY Canister Development Kit (CDK) for Rust は、分散型の Internet Computer ブロックチェーンメインネット上で動作する Dapps を作成するためのツールとサンプルコードとドキュメントを提供しています。 _Hello World! Rust CDK クイックスタート_ では、DFINITY Rust CDK を初めてインストールすることを想定しています。

初めての方の助けとなるように、このチュートリアルでは、よくある "Hello World" という最初の Dapp を DFINITY Rust CDK を使うように修正する方法を説明します。 このシンプルな Dapp は、ターミナルにテキストを出力する関数が１つ定義されているだけですが、 Internet Computer ブロックチェーンにデプロイする Dapp を Rust で書く際のワークフローを理解するための良いモデルとなります。

## はじめる前に

プロジェクトをはじめる前に、以下を確認してください:

- インターネットに接続しており、ローカルの macOS または Linux コンピュータでターミナルにアクセスできること。

- [Rust のインストール方法](https://doc.rust-lang.org/book/ch01-01-installation.html) にあるように、Rust プログラミング言語と Cargo が OS にダウンロードされ、インストールされていること。

  ```bash
  curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
  ```

  Rust のバージョンは 1.46.0 より新しい必要があります。

- [ダウンロードとインストール](../../../quickstart/hello10mins.md) の説明に従って、DFINITY Canister Software Development Kit (SDK) パッケージのダウンロードとインストールが済んでいること。

- `ic-cdk-optimizer` ユーティリティのインストールが済んでいること。以下のコマンドでインストールすることができます。

  ```bash
  cargo install ic-cdk-optimizer
  ```

- `cmake` のインストールが済んでいること。例えば、macOS では Homebrew を使って以下のコマンドを実行します:

  ```bash
  brew install cmake
  ```

  Homebrew をインストールする方法については、[Homebrew のドキュメント](https://docs.brew.sh/Installation)を参照してください。

- コンピュータ上の ローカル実行環境 プロセスが停止していること。

## 新しいプロジェクトの作成

Internet Computer ブロックチェーン用のアプリケーション開発は **プロジェクト** の作成から始まります。 DFINITY Canister SDK を使用して Internet Computer ブロックチェーンの新しいプロジェクトを作成することができます。 Internet Computer ブロックチェーンにデプロイするプロジェクトを作成するため、本チュートリアルでは、親コマンドである `dfx` とそのサブコマンドを使って Rust プログラムを作成・ビルド・デプロイする方法を中心に説明します。

DFINITY Canister SDK を使って新しいプロジェクトを作成する手順は以下になります:

1.  ローカル PC でターミナルを開きます。

2.  以下のコマンドを実行し、`rust_hello` という名前の Rust Canister を持つ新しいプロジェクトを作成します。

    ```bash
    dfx new --type=rust rust_hello
    ```

    `dfx new --type=rust rust_hello` コマンドは `rust_hello` プロジェクトのディレクトリとテンプレートファイルと `rust_hello` の Git リポジトリを新たに生成します。

3.  以下のコマンドで、プロジェクトディレクトリに移動します。

    ```bash
    cd rust_hello
    ```

## プロジェクトファイルを一通り確認する

プロジェクトをコンパイルし、Internet Computer ブロックチェーンにデプロイする準備ができました。 デプロイする前に、プロジェクトファイルを見てみましょう。

### `dfx.json`

プロジェクトディレクトリに含まれるテンプレートファイルの一つに、`dfx.json` 設定ファイルがあります。 このファイルには Internet Computer ブロックチェーンで動かすプロジェクトをビルドするために必要な設定が含まれています。 これは `Cargo.toml` ファイルが Rust プログラムのビルドやパッケージ管理の設定を含んでいるのと同じです。

設定ファイルは[こちら](../../_attachments/rust-quickstart-dfx.json) のようなものです。

`canisters` キーの下に、`rust_hello` Canister のデフォルト設定があることに注意してください。

1.  `"type": "rust "` は `rust_hello` が `rust` タイプの Canister であることを指定します。

2.  `"candid": "src/rust_hello/rust_hello.did "` は、Canister に使用する Candid インターフェース記述ファイルの場所を指定します。

3.  `"package": "rust_hello "` は、Rust クレートのパッケージ名を指定します。`Cargo.toml` ファイルに記述されているものと同じである必要があります。

### `Cargo.toml`

ルートディレクトリには、`Cargo.toml` というファイルがあります。

これは、各 Rust クレートのパスを指定することで、Rust ワークスペースを定義しています。 Rust タイプの Canister は、WebAssembly にコンパイルされた Rust クレートにすぎません。 ここでは、`src/rust_hello` に一つのメンバーを持っており、これが唯一の Rust Canister です。

```toml
[workspace]
members = [
    "src/rust_hello",
]
```

### `src/rust_hello/`

さて、Rust Canister を見ていきましょう。標準的な Rust クレートと同様に `Cargo.toml` ファイルがあり、Rust クレートをビルドするための詳細な設定をしています。

#### `src/rust_hello/Cargo.toml`

```toml
[package]
name = "rust_hello"
version = "0.1.0"
edition = "2018"

[lib]
path = "lib.rs"
crate-type = ["cdylib"]

[dependencies]
ic-cdk = "0.5"
ic-cdk-macros = "0.5"
```

`crate-type = ["cdylib"]` の行に注目してください。これは、この Rust プログラムを WebAssembly モジュールにコンパイルするために必要なものです。

#### `src/rust_hello/lib.rs`

デフォルトのプロジェクトでは、DFINITY Rust CDK `query` マクロを使用したシンプルな `greet` 関数が用意されています。

```rust
#[ic_cdk_macros::query]
fn greet(name: String) -> String {
    format!("Hello, {}!", name)
}
```

#### `src/rust_hello/rust_hello.did`

Candid は、Internet Computer ブロックチェーンで動作する Canister と対話するためのインターフェース記述言語（IDL）です。 Candid ファイルは、Canister が定義する各関数の名前・引数・返し値のフォーマットやデータ型など、Canister のインターフェースを言語に依存しないように記述したものです。

Candid ファイルをプロジェクトに追加することで、Rust で定義されたデータが Internet Computer ブロックチェーン上で安全に実行されるために適切に変換されることを保証します。

Candid インターフェース記述言語の構文の詳細は [_Candid ガイド_](../candid-guide/candid-intro)か [Candid クレートのドキュメント](https://docs.rs/candid/)をご覧ください。

```did
service : {
    "greet": (text) -> (text) query;
}
```

これは、`greet` 関数が `text` データを入力として受け取り `text` データを返す `query` メソッドであることを指定します。

## ローカル実行環境 を立ち上げる

プロジェクトをビルドする前に、ローカル実行環境 か、Internet Computer ブロックチェーンメインネットに接続する必要があります。

ローカル実行環境 を立ち上げるには、以下のようにします:

1.  自分がプロジェクトのルートディレクトリにいることを確認します。

2.  ローカル実行環境 をバックグラウンドで立ち上げるために、以下のコマンドを実行します:

    ```bash
    dfx start --background
    ```

    プラットフォームやローカルのセキュリティ設定によっては、警告が表示される場合があります。 ネットワーク接続を許可するかどうかの確認画面が表示された場合は、**Allow** をクリックします。

## プロジェクトの登録・ビルド・デプロイ

開発環境で立ち上がっている ローカル実行環境 に接続した後に、プロジェクトの登録・ビルド・デプロイをローカル環境で行うことができます。 登録・ビルド・デプロイを行うためには、以下のようにします:

1.  プロジェクトのルートディレクトリにいることを確認します。

2.  以下のコマンドを実行し、`wasm32-unknown-unknown` がインストールされていることを確認します：

    ```bash
    rustup target add wasm32-unknown-unknown
    ```

3.  `dfx.json` に指定されている Canister を以下のコマンドで、登録・ビルド・デプロイします:

    ```bash
    dfx deploy
    ```

    `dfx deploy` コマンドを実行すると、以下のような実行結果に関しての情報が表示されます。

        Creating a wallet canister on the local network.
        The wallet canister on the "local" network for user "default" is "rwlgt-iiaaa-aaaaa-aaaaa-cai"
        Deploying all canisters.
        Creating canisters...
        Creating canister "rust_hello"...
        "rust_hello" canister created with canister id: "rrkah-fqaaa-aaaaa-aaaaq-cai"
        Creating canister "rust_hello_assets"...
        "rust_hello_assets" canister created with canister id: "ryjl3-tyaaa-aaaaa-aaaba-cai"
        Building canisters...
        Executing: "cargo" "build" "--target" "wasm32-unknown-unknown" "--release" "-p" "rust_hello"
            Updating crates.io index
           Compiling unicode-xid v0.2.2
           ...
        Building frontend...
        Installing canisters...
        Creating UI canister on the local network.
        The UI canister on the "local" network is "r7inp-6aaaa-aaaaa-aaabq-cai"
        Installing code for canister rust_hello, with canister_id rrkah-fqaaa-aaaaa-aaaaq-cai
        ...
        Deployed canisters.

## Dapp のテスト

ローカルでデプロイした Dapp をテストするには、以下のようにします:

1.  プロジェクトのルートディレクトリにいることを確認します。

2.  Dpp の `greet` 関数を以下のコマンドで呼び出します:

    ```bash
    dfx canister call rust_hello greet world
    ```

3.  `rust_hello` Canister の `greet` 関数の呼び出しがテキストメッセージである `("Hello, world!")` を返すことを確認します。

## ローカル実行環境 を止める

アプリケーションのテストをした後は、ローカル実行環境 がバックグラウンドで稼働し続けないように、以下の手順で停止します:

1.  ネットワークの稼働状況が表示されている端末で、Control-C を押して ローカル実行環境 を止めてください。

2.  以下のコマンドを用いて ローカル実行環境 を停止してください:

    ```bash
    dfx stop
    ```

<!--
# Hello, World! Rust CDK Quick Start

The DFINITY Canister Development Kit (CDK) for Rust provides tools, sample code, and documentation to help you create dapps to run on the decentralized Internet Computer blockchain mainnet. This *Hello, World! Rust CDK Quick Start* assumes that you are installing the DFINITY Rust CDK for the first time.

To help you get started, this tutorial illustrates how to modify the traditional "Hello World" first dapp to use the DFINITY Rust CDK. This simple dapp has just one function that prints text to a terminal, but it provides a good model for understanding the workflow when writing dapps in Rust that you want to deploy on the Internet Computer blockchain.

## Before you begin

Before you start your project, verify the following:

-   You have an internet connection and access to a shell terminal on your local macOS or Linux computer.

-   You have downloaded and installed the Rust programming language and Cargo as described in the [Rust installation instructions](https://doc.rust-lang.org/book/ch01-01-installation.html) for your operating system.

    ``` bash
    curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
    ```

    The Rust tool chain must be at version 1.46.0, or later.

-   You have downloaded and installed the DFINITY Canister Software Development Kit (SDK) package as described in [Download and install](../../../quickstart/hello10mins.md).

-   You have installed the `ic-cdk-optimizer` utility. You can install it with:
    ``` bash
    cargo install ic-cdk-optimizer
    ```

-   You have `cmake` installed. For example, use Homebrew with the following command:

    ``` bash
    brew install cmake
    ```

    For instructions on how to install Homebrew, see the [Homebrew Documentation](https://docs.brew.sh/Installation).

-   You have stopped any local execution environment processes running on your computer.

## Create a new project

Applications for the Internet Computer blockchain start as **projects**. You can create new projects for the Internet Computer blockchain using the DFINITY Canister SDK. Because you are building this project to be deployed on the Internet Computer blockchain, this tutorial focuses on how to create, build, and deploy a Rust program by using the `dfx` parent command and its subcommands.

To create a new project using the DFINITY Canister SDK:

1.  Open a terminal shell on your local computer, if you don’t already have one open.

2.  Create a new project with rust canister named `rust_hello` by running the following command:

    ``` bash
    dfx new --type=rust rust_hello
    ```

    The `dfx new --type=rust rust_hello` command creates a new `rust_hello` project directory, template files, and a new `rust_hello` Git repository for your project.

3.  Change to your project directory by running the following command:

    ``` bash
    cd rust_hello
    ```

## Take a look at the project

The project is ready to be compiled and deployed to the Internet Computer blockchain. Before that, let’s go through the project files.

### `dfx.json`

One of the template files included in your project directory is a default `dfx.json` configuration file. This file contains settings required to build a project for the Internet Computer blockchain much like the `Cargo.toml` file provides build and package management configuration details for Rust programs.

The configuration file should look like [this](../../_attachments/rust-quickstart-dfx.json).

Notice that under the `canisters` key, you have some default settings for the `rust_hello` canister.

1.  `"type": "rust"` specifies that `rust_hello` is a `rust` type canister.

2.  `"candid": "src/rust_hello/rust_hello.did"` specifies the location of the Candid interface description file to use for the canister.

3.  `"package": "rust_hello"` specifies the package name of the Rust crate. It should be the same as in the crate `Cargo.toml` file.

### `Cargo.toml`

In the root directory, there is a `Cargo.toml` file.

It defines a Rust workspace by specifying paths to each Rust crate. A Rust type canister is just a Rust crate compiled to WebAssembly. Here we have one member at `src/rust_hello` which is the only Rust canister.

``` toml
[workspace]
members = [
    "src/rust_hello",
]
```

### `src/rust_hello/`

Now we are in the Rust canister. As any standard Rust crate, it has a `Cargo.toml` file which configures the details to build the Rust crate.

#### `src/rust_hello/Cargo.toml`

``` toml
[package]
name = "rust_hello"
version = "0.1.0"
edition = "2018"

[lib]
path = "lib.rs"
crate-type = ["cdylib"]

[dependencies]
ic-cdk = "0.5"
ic-cdk-macros = "0.5"
```

Notice the `crate-type = ["cdylib"]` line which is necessary to compile this rust program into WebAssembly module.

#### `src/rust_hello/lib.rs`

The default project has a simple `greet` function that uses the DFINITY Rust CDK `query` macro.

``` rust
#[ic_cdk_macros::query]
fn greet(name: String) -> String {
    format!("Hello, {}!", name)
}
```

#### `src/rust_hello/rust_hello.did`

Candid is an interface description language (IDL) for interacting with canisters running on the Internet Computer. Candid files provide a language-independent description of a canister’s interfaces including the names, parameters, and result formats and data types for each function a canister defines.

By adding Candid files to your project, you can ensure that data is properly converted from its definition in Rust to run safely on the Internet Computer blockchain.

To see details about the Candid interface description language syntax, see the [*Candid Guide*](../../candid/candid-intro.md) or the [Candid crate documentation](https://docs.rs/candid/).

``` did
service : {
    "greet": (text) -> (text) query;
}
```

This definition specifies that the `greet` function is a `query` method which takes `text` data as input and returns `text` data.

## Start the local execution environment

Before you can build your project, you need to connect to the local execution environment running in your development environment or the decentralized Internet Computer blockchain mainnet.

To start the local execution environment:

1.  Check that you are still in the root directory for your project, if needed.

2.  Start the local execution environment on your computer in the background by running the following command:

    ``` bash
    dfx start --background
    ```

    Depending on your platform and local security settings, you might see a warning displayed. If you are prompted to allow or deny incoming network connections, click **Allow**.

## Register, build, and deploy your project

After you connect to the local execution environment running in your development environment, you can register, build, and deploy your project locally.

To register, build, and deploy:

1.  Check that you are still in root directory for your project directory, if needed.

2.  Make sure you have `wasm32-unknown-unknown` installed by running the command:

    ``` bash
    rustup target add wasm32-unknown-unknown
    ```

3.  Register, build, and deploy the canisters specified in the `dfx.json` file by running the following command:

    ``` bash
    dfx deploy
    ```

    The `dfx deploy` command output displays information about each of the operations it performs similar to the following excerpt:

    ``` bash
    Creating a wallet canister on the local network.
    The wallet canister on the "local" network for user "default" is "rwlgt-iiaaa-aaaaa-aaaaa-cai"
    Deploying all canisters.
    Creating canisters...
    
    Creating canister rust_hello_backend...
    rust_hello_backend canister created with canister id: rrkah-fqaaa-aaaaa-aaaaq-cai
    
    Creating canister rust_hello_frontend...
    rust_hello_frontend canister created with canister id: ryjl3-tyaaa-aaaaa-aaaba-cai
    
    Building canisters...

    ...

    Deployed canisters.
    URLs:

    Frontend canister via browser
        rust_hello_frontend: http://127.0.0.1:8000/?canisterId=ryjl3-tyaaa-aaaaa-aaaba-cai

    Backend canister via Candid interface:
        rust_hello_backend: http://127.0.0.1:8000/?canisterId=r7inp-6aaaa-aaaaa-aaabq-cai&id=rrkah-fqaaa-aaaaa-aaaaq-cai
    ```


## Test the dapp

To test the deployed dapp locally:

1.  Check that you are still in root directory for your project directory, if needed.

2.  Call the `greet` function in the dapp by running the following command:

    ``` bash
    dfx canister call rust_hello_backend greet world
    ```

3.  Verify that the call to the `rust_hello_backend` canister `greet` function returns a text message `("Hello, world!")`.

## Stop the local execution environment

After testing the application, you can stop the local execution environment so that it doesn’t continue running in the background.

To stop the local execution environment:

1.  In the terminal that displays network operations, press Control-C to interrupt the local execution environment process.

2.  Stop the local execution environment running on your computer by running the following command:

    ``` bash
    dfx stop
    ```

-->
