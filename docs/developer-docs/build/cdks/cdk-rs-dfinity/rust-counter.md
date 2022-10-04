# カウンターのインクリメント

このチュートリアルでは、いくつかの簡単な関数によってカウンターの値をインクリメントする Dapp を作り、値の永続性について説明します。

この Dapp は、カウンターの値（自然数）を格納する可変型変数として、 `COUNTER` を宣言しています。 ここで以下の関数を考えます：

- `increment` : 現在の値を更新する関数です。値を１ずつ増加させ、戻り値はありません。

- `get` : カウンターの現在の値を返すシンプルなクエリ関数です。

- `set` : 引数で指定した数値にカウンターの値を更新する関数です。

このチュートリアルでは、Canister スマートコントラクトにデプロイされた Canister の関数を呼び出して、カウンターの値をインクリメントする簡単な例を示します。 値を増加させる関数を複数回呼び出すことで、呼び出しの間に変数の値が変わらないこと（値の永続性）を確認できます。

他のサンプル Dapps と同じように、このチュートリアルはシンプルでありながら現実的なワークフローのデモンストレーションとなっています:

- 新しいプロジェクトを作成します。

- WebAssembly モジュールにコンパイルされる Dapp を書きます。

- ローカル Canister 実行環境 に Canister をデプロイします。

- Canister スマートコントラクトのメソッドを呼び出して、カウンターの値をインクリメントしたり読んだりします。

## はじめる前に

プロジェクトをはじめる前に、以下を確認します:

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

2.  以下のコマンドを実行し、`rust_counter` という名前の新しいプロジェクトを作成します。

    ```bash
    dfx new --type=rust rust_counter
    ```

3.  以下のコマンドで、プロジェクトディレクトリに移動します。

    ```bash
    cd rust_counter
    ```

## デフォルトプロジェクトの変更

[Hello World! Rust CDK クイックスタート](rust-quickstart) では、Rust の Canister を使ったデフォルトのプロジェクトのファイルを確認しました。

このチュートリアルを完了するには、以下の手順を踏みます。

- [デフォルト Dapp の置き換え](#デフォルトの-dapp-を置き換え)

- [インターフェース記述ファイルの更新](#インターフェイス記述ファイルの更新)

### デフォルトの Dapp を置き換え

これで Rust Dapp 用のファイルが揃ったので、`lib.rs` Dapp を Rust Dapp に置き換えていきます。

既存の Dapp を置き換えるには以下のようにします。

1.  自分がルートディレクトリにいることを確認します。

2.  `src/rust_counter/lib.rs` ファイルをテキストエディタで開き、既存の内容を削除します。

    次に、`COUNTER` 変数の定義と `increment` ・ `get` ・ `set` の３つの関数の実装を行っていきます。

3.  [こちら](../../_attachments/counter.rs) のサンプルコードを `lib.rs` にコピー＆ペーストしてください。

4.  変更を保存して `lib.rs` ファイルを閉じ、次に進みます。

### インターフェイス記述ファイルの更新

Candid は、Internet Computer blockchain で動作する Canister と対話するためのインターフェース記述言語（IDL）です。 Candid ファイルは、Canister が定義する各関数の名前・引数・返し値のフォーマットやデータ型など、Canister のインターフェースを言語に依存しないように記述したものです。

Candid ファイルをプロジェクトに追加することで、Rust で定義されたデータが Internet Computer blockchain 上で安全に実行されるために適切に変換されることを保証します。

Candid インターフェース記述言語の構文の詳細は [_Candid ガイド_](../candid/candid-intro)か [Candid クレートのドキュメント](https://docs.rs/candid/)をご覧ください。

Candid ファイルを更新するには、以下のようにします:

1.  自分がプロジェクトのルートディレクトリにいることを確認します。

2.  `src/rust_counter/rust_counter.did` ファイルをテキストエディタで開き、 `increment` ・ `get` ・ `set` 関数のために以下の `service` の定義をコピー＆ペーストして書き込んでください。

    ```did
    service : {
      "increment": () -> ();
      "get": () -> (nat) query;
      "set": (nat) -> ();
    }
    ```

3.  変更を保存して `rust_counter.did` ファイルを閉じ、次に進んでください。

## ローカル Canister 実行環境 を立ち上げる

`rust_counter` プロジェクトをビルドする前に、ローカル Canister 実行環境 か、Internet Computer blockchain メインネットに接続する必要があります。

ローカル Canister 実行環境 を立ち上げるには、以下のようにします:

1.  自分がプロジェクトのルートディレクトリにいることを確認します。

2.  ローカル Canister 実行環境 をバックグラウンドで立ち上げるために、以下のコマンドを実行します:

    ```bash
    dfx start --background
    ```

    プラットフォームやローカルのセキュリティ設定によっては、警告が表示される場合があります。 ネットワーク接続を許可するかどうかの確認画面が表示された場合は、**Allow** をクリックします。

## プロジェクトの登録・ビルド・デプロイ

開発環境で立ち上がっている ローカル Canister 実行環境 に接続した後に、プロジェクトの登録・ビルド・デプロイをローカル環境で行うことができます。

登録・ビルド・デプロイを行うためには、以下のようにします:

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
        Creating canister "rust_counter"...
        "rust_counter" canister created with canister id: "rrkah-fqaaa-aaaaa-aaaaq-cai"
        Creating canister "rust_counter_assets"...
        "rust_counter_assets" canister created with canister id: "ryjl3-tyaaa-aaaaa-aaaba-cai"
        Building canisters...
        Executing: "cargo" "build" "--target" "wasm32-unknown-unknown" "--release" "-p" "rust_counter"
        ...
            Finished release [optimized] target(s) in 53.36s
        Building frontend...
        Installing canisters...
        Creating UI canister on the local network.
        The UI canister on the "local" network is "r7inp-6aaaa-aaaaa-aaabq-cai"
        Installing code for canister rust_counter, with canister_id rrkah-fqaaa-aaaaa-aaaaq-cai
        ...
        Deployed canisters.

## 関数を読んで Dapp をテストする

Canister のデプロイに成功したら、Canister の関数を呼び出すことでテストすることができます。 このチュートリアルでは、以下をテストします。

- `get` 関数を呼び、カウンターの値をクエリする。

- `increment` 関数を呼び、呼ぶたびにカウンターの値をインクリメントする。

- `set` 関数を呼び、引数として与えた任意の値でカウンターの値をアップデートする。

Dapp のテストのために、以下を実行します:

1.  以下のコマンドで `get` 関数を呼び、`COUNTER` 変数の現在の値を取得します；

    ```bash
    dfx canister call rust_counter get
    ```

    このコマンドは、`COUNTER` 変数の現在の値である０を返します。

        (0 : nat)

2.  `increment` 関数を呼び、`COUNTER` 変数の値を１つずつインクリメントします：

    ```bash
    dfx canister call rust_counter increment
    ```

    このコマンドは変数の値（ステート）をインクリメントしますが、返し値はありません：

3.  `get` 関数を呼ぶコマンドを再度実行し、`COUNTER` 変数の現在の値を確認します。

    ```bash
    dfx canister call rust_counter get
    ```

    このコマンドは、更新された `COUNTER` 変数の値である１を返します：

        (1 : nat)

4.  追加のコマンドを実行して、別の値を使った関数の呼び出しを試してみましょう。

    例えば、以下のようなコマンドを実行して、カウンターの値のセットや取得を試してみましょう：

    ```bash
    dfx canister call rust_counter set '(987)'
    dfx canister call rust_counter get
    ```

    このコマンドは、現在の値である 987 を返します。

    ```bash
    dfx canister call rust_counter increment
    dfx canister call rust_counter get
    ```

    このコマンドは、インクリメントされた値である 988 を返します。

## ローカル Canister 実行環境 を止める

アプリケーションのテストをした後は、ローカル Canister 実行環境 がバックグラウンドで稼働し続けないように、以下の手順で停止します：

1.  ネットワークの稼働状況が表示されている端末で、Control-C を押して ローカル Canister 実行環境 を止めてください。

2.  以下のコマンドを用いて ローカル Canister 実行環境 を停止してください：

    ```bash
    dfx stop
    ```

<!--
# Incrementing a counter

In this tutorial, you are going to write a dapp that provides a few basic functions to increment a counter and illustrates the persistence of a value.

For this tutorial, the dapp declares a `COUNTER` as a mutable variable to contain a natural number that represents the current value of the counter. This dapp supports the following functions:

-   The `increment` function updates the current value, incrementing by 1 with no return value.

-   The `get` function is a simple query that returns the current value of the counter.

-   The `set` function updates the current value to the numeric value you specify as an argument.

This tutorial provides a simple example of how you can increment a counter by calling functions on a deployed canister. By calling the function to increment a value multiple times, you can verify that the variable state—that is, the value of the variable between calls—persists.

Like the other sample dapps, this tutorial demonstrates a simple, but realistic, workflow in which you perform the following steps:

-   Create a new project.

-   Write a dapp that compiles into a WebAssembly module.

-   Deploy the canister on the local canister execution environment.

-   Invoke the canister methods to increment then read the value of a counter.

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
    dfx new --type=rust rust_counter
    ```

3.  Change to your project directory by running the following command:

    ``` bash
    cd rust_counter
    ```

## Modify the default project

In the [Hello, World! Rust CDK Quick Start](./rust-quickstart.md), you went through the files in a default project with Rust type canister.

To complete this tutorial, you’ll need to complete the following steps:

-   [Replace the default dapp](#_replace_the_default_dapp)

-   [Update interface description file](#_update_interface_description_file)

### Replace the default dapp

Now that you have the files in place for your Rust dapp, we can replace the template `lib.rs` dapp with the Rust dapp we want to deploy on the Internet Computer blockchain.

To replace the default dapp:

1.  Check that you are still in the root directory for your project, if needed.

2.  Open the template `src/rust_counter/lib.rs` file in a text editor and delete the existing content.

    The next step is to write a Rust dapp that declares the `COUNTER` variable and implements the `increment`, `get`, and `set` functions.

3.  Copy and paste [this code](../../_attachments/counter.rs) into the `lib.rs` file.

4.  Save your changes and close the `lib.rs` file to continue.

### Update interface description file

Candid is an interface description language (IDL) for interacting with canisters running on the Internet Computer. Candid files provide a language-independent description of a canister’s interfaces including the names, parameters, and result formats and data types for each function a canister defines.

By adding Candid files to your project, you can ensure that data is properly converted from its definition in Rust to run safely on the Internet Computer blockchain.

To see details about the Candid interface description language syntax, see the [*Candid Guide*](../../candid/candid-intro.md) or the [Candid crate documentation](https://docs.rs/candid/).

To update the Candid file for this tutorial:

1.  Check that you are still in the root directory for your project, if needed.

2.  Open the `src/rust_counter/rust_counter.did` file in a text editor, then copy and paste the following `service` definition for the `increment`, `get`, and `set` functions:

    ``` did
    service : {
      "increment": () -> ();
      "get": () -> (nat) query;
      "set": (nat) -> ();
    }
    ```

3.  Save your changes and close the `rust_counter.did` file to continue.

## Start the local canister execution environment

Before you can build the `rust_counter` project, you need to connect to the local canister execution environment running in your development environment or the decentralized Internet Computer blockchain mainnet.

To start the local canister execution environment:

1.  Check that you are still in the root directory for your project, if needed.

2.  Start the local canister execution environment on your computer in the background by running the following command:

    ``` bash
    dfx start --background
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
        Creating canister "rust_counter"...
        "rust_counter" canister created with canister id: "rrkah-fqaaa-aaaaa-aaaaq-cai"
        Creating canister "rust_counter_assets"...
        "rust_counter_assets" canister created with canister id: "ryjl3-tyaaa-aaaaa-aaaba-cai"
        Building canisters...
        Executing: "cargo" "build" "--target" "wasm32-unknown-unknown" "--release" "-p" "rust_counter"
        ...
            Finished release [optimized] target(s) in 53.36s
        Building frontend...
        Installing canisters...
        Creating UI canister on the local network.
        The UI canister on the "local" network is "r7inp-6aaaa-aaaaa-aaabq-cai"
        Installing code for canister rust_counter, with canister_id rrkah-fqaaa-aaaaa-aaaaq-cai
        ...
        Deployed canisters.

## Call functions and test the dapp

After successfully deploying the canister, you can test the canister by invoking the functions it provides. For this tutorial:

-   Call the `get` function to query the value of the counter.

-   Call the `increment` function to increment the counter each time it is called.

-   Call the `set` function to pass an argument to update the counter to an arbitrary value you specify.

To test the dapp:

1.  Call the `get` function to read the current value of the `COUNTER` variable by running the following command:

    ``` bash
    dfx canister call rust_counter get
    ```

    The command returns the current value of the `COUNTER` variable as zero:

        (0 : nat)

2.  Call the `increment` function to increment the value of the `COUNTER` variable by one:

    ``` bash
    dfx canister call rust_counter increment
    ```

    This command increments the value of the variable—changing its state—but does not return the result.

3.  Rerun the command to call the `get` function to see the current value of the `COUNTER` variable:

    ``` bash
    dfx canister call rust_counter get
    ```

    The command returns the updated value of the `COUNTER` variable as one:

        (1 : nat)

4.  Run additional commands to experiment with call the functions and using different values.

    For example, try commands similar to the following to set and return the counter value:

    ``` bash
    dfx canister call rust_counter set '(987)'
    dfx canister call rust_counter get
    ```

    Returns the current value of 987.

    ``` bash
    dfx canister call rust_counter increment
    dfx canister call rust_counter get
    ```

    Returns the incremented value of 988.

## Stop the local canister execution environment

After you finish experimenting with your dapp, you can stop the local Internet Computer network so that it doesn’t continue running in the background.

To stop the local canister execution environment:

1.  In the terminal that displays network operations, press Control-C to interrupt the local canister execution environment process.

2.  Stop the local canister execution environment by running the following command:

    ``` bash
    dfx stop
    ```

-->
