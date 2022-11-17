# シンプルなレコードの追加と検索

このチュートリアルでは、名前・説明・キーワードの配列で構成される、シンプルなプロフィールレコードを追加・取得するためのいくつかの基本的な関数を有する Dapp を書きます。

このプログラムは、以下の関数を有します：

- `update` 関数は、`name`、`description`、`keywords` からなるプロフィールを追加することができます。

- `getSelf` 関数は、関数の呼び出し元に関連づけられたプリンシパルのプロフィールを返します。

- `get` 関数は、渡された `name` の値にマッチするプロフィールを返す単純なクエリ関数です。 この関数では、指定された名前が `name` フィールドと完全に一致する必要があります。

- `search` 関数は、より複雑なクエリを実行して、任意のフィールドで指定されたテキストのすべてまたは一部に一致するプロフィールを返します。例えば、特定のキーワードを含むプロフィールや、名前や説明の一部のみにマッチするプロフィールを返すことができます。

このチュートリアルでは、Rust CDK のインターフェースとマクロを使用して、Rust で Internet Computer ブロックチェーン 用の Dapps を簡単に書けるようにするための簡単な例を紹介しています。

このチュートリアルでは以下を説明します：

- Candid インターフェース記述言語を使った、少し複雑なデータ（`record` とキーワードの `array`）の表現の仕方。

- 部分一致する文字列を照合するシンプルな検索機能の書き方。

- 特定のプリンシパルとプロフィールの関連付け方。

## はじめる前に

プロジェクトをはじめる前に、以下をご確認ください:

- インターネットに接続しており、ローカルの macOS または Linux コンピュータでターミナルにアクセスできること。

- [Rust のインストール方法](https://doc.rust-lang.org/book/ch01-01-installation.html) にあるように、Rust プログラミング言語と Cargo が OS にダウンロードされ、インストールされていること。

  ```bash
  curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
  ```

  Rust のバージョンは 1.46.0 より新しい必要があります。

- [ダウンロードとインストール](../../../quickstart/hello10mins.md) の説明に従って、DFINITY Canister Software Development Kit (SDK) パッケージのダウンロードとインストールが済んでいること。

- `cmake` のインストールが済んでいること。例えば、macOS では Homebrew を使って以下のコマンドを実行します:

  ```bash
  brew install cmake
  ```

  Homebrew をインストールする方法については、[Homebrew のドキュメント](https://docs.brew.sh/Installation)を参照してください。

- コンピュータ上の ローカル実行環境 プロセスが停止していること。

このチュートリアルの所要時間は約 20 分です。

## 新しいプロジェクトの作成

新しいプロジェクトを作る手順は以下になります:

1.  ローカル PC でターミナルを開きます。

2.  以下のコマンドを実行して新しいプロジェクトを作成します。

    ```bash
    dfx new --type=rust rust_profile
    ```

3.  以下のコマンドで、プロジェクトディレクトリに移動します。

    ```bash
    cd rust_profile
    ```

## デフォルトプロジェクトの変更

[Hello World! Rust CDK クイックスタート](rust-quickstart)では、Rust Canister のデフォルトプロジェクトのファイルを確認しました。

このチュートリアルを完了するには、以下の手順を踏みます。

- [デフォルト Dapp の置き換え](#デフォルト-dapp-の置き換え)

- [インターフェース記述ファイルの更新](#インターフェース記述ファイルの更新)

### デフォルト Dapp の置き換え

Rust Dapp 用のファイルが揃ったので、テンプレートの `lib.rs` Dapp を Internet Computer ブロックチェーン にデプロイしたい Rust Dapp に置き換えていきます。

デフォルトのプログラムを置き換えるには、以下のようにします。

1.  自分がプロジェクトのルートディレクトリにいることを確認します。

2.  テキストエディタで `src/rust_profile/Cargo.toml` ファイルを開き、依存関係に `serde` を追加します。

    ```toml
    [dependencies]
    ic-cdk = "0.5"
    ic-cdk-macros = "0.5"
    serde = "1.0"
    ```

3.  `src/rust_profile/lib.rs` ファイルをテキストエディタで開き、既存の内容を削除します。

    次に、`getSelf`、`update`、`get`、`search` 関数を Rust プログラムで実装していきます。

4.  [こちら](../../_attachments/profile.rs) のサンプルコードを `profile.rs` にコピー＆ペーストしてください。

5.  変更を保存してファイルを閉じ、次に進みます。

## インターフェイス記述ファイルの更新

Candid は、Internet Computer ブロックチェーン で動作する Canister と対話するためのインターフェース記述言語（IDL）です。 Candid ファイルは、Canister が定義する各関数の名前・引数・返し値のフォーマットやデータ型など、Canister のインターフェースを言語に依存しないように記述したものです。

Candid ファイルをプロジェクトに追加することで、Rust で定義されたデータが Internet Computer ブロックチェーン 上で安全に実行されるために適切に変換されることを保証します。

Candid インターフェース記述言語の構文の詳細は [_Candid ガイド_](../../candid/candid-intro.md))か [Candid クレートのドキュメント](https://docs.rs/candid/)をご覧ください。

Candid ファイルを更新するには、以下のようにします:

1.  自分がプロジェクトのルートディレクトリにいることを確認します。

2.  `src/rust_profile/rust_profile.did` ファイルをテキストエディタで開いてください。

3.  `getSelf`、`update`、`get`、`search` 関数に対する以下の `Profile` 型宣言と `service` 定義をコピー＆ペーストしてください：

    ```did
    type Profile = record {
        "name": text;
        "description": text;
        "keywords": vec text;
    };
    service : {
        "getSelf": () -> (Profile) query;
        "get": (text) -> (Profile) query;
        "update": (Profile) -> ();
        "search": (text) -> (opt Profile) query;
    }
    ```

4.  変更を保存してファイルを閉じ、次に進んでください。

## ローカル実行環境 を立ち上げる

`rust_profile` プロジェクトをビルドする前に、ローカル実行環境 か、Internet Computer ブロックチェーン メインネットに接続する必要があります。

ローカル実行環境 を立ち上げるには、以下のようにします:

1.  自分がプロジェクトのルートディレクトリにいることを確認します。

2.  ローカル実行環境 をバックグラウンドで立ち上げるために、以下のコマンドを実行します:

    ```bash
    dfx start --background --clean
    ```

    プラットフォームやローカルのセキュリティ設定によっては、警告が表示される場合があります。 ネットワーク接続を許可するかどうかの確認画面が表示された場合は、**Allow** をクリックします。

## プロジェクトの登録・ビルド・デプロイ

開発環境で立ち上がっている ローカル実行環境 に接続した後に、プロジェクトの登録・ビルド・デプロイをローカル環境で行うことができます。

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
        Creating canister "rust_profile"...
        "rust_profile" canister created with canister id: "rrkah-fqaaa-aaaaa-aaaaq-cai"
        Creating canister "rust_profile_assets"...
        "rust_profile_assets" canister created with canister id: "ryjl3-tyaaa-aaaaa-aaaba-cai"
        Building canisters...
        Executing: "cargo" "build" "--target" "wasm32-unknown-unknown" "--release" "-p" "rust_profile"
        ...
            Finished release [optimized] target(s) in 6.31s
        Building frontend...
        Installing canisters...
        Creating UI canister on the local network.
        The UI canister on the "local" network is "r7inp-6aaaa-aaaaa-aaabq-cai"
        Installing code for canister rust_profile, with canister_id rrkah-fqaaa-aaaaa-aaaaq-cai
        ...
        Deployed canisters.

## デプロイした Canister の関数を呼ぶ

Canister のデプロイが成功すると、Canister の関数を呼ぶことでテストできるようになります。

このチュートリアルでは、以下のようにします：

- `update` 関数を呼び、プロフィールを追加する。

- `getSelf` 関数を呼び、プリンシパル ID のプロフィールを表示する。

- `search` 関数を呼び、キーワードを使ってプロフィールを検索する。

デプロイした Canister をテストするために：

1.  `update` 関数を呼び、プロフィールレコードを作成するには以下のコマンドを実行します：

    ```bash
    dfx canister call rust_profile update '(record {name = "Luxi"; description = "mountain dog"; keywords = vec {"scars"; "toast"}})'
    ```

2.  `getSelf` 関数を呼び、プロフィールレコードを取得するには以下のコマンドを実行します：

    ```bash
    dfx canister call rust_profile getSelf
    ```

    このコマンドは、`update` 関数を使って追加したプロフィールを返します。 例えば、以下のようになります：

        (  record {
            name = "Luxi";
            description = "mountain dog";
            keywords = vec { "scars"; "toast" };
          },
        )

    いまのところ、この Dapp は１つのプロフィールを保存して返すだけです。 次のコマンドを実行して、`update` 関数を使って２つ目のプロフィールを追加すると、`Luxi` のプロフィールが `Dupree` のプロフィールに置き換わります：

    ```bash
    dfx canister call rust_profile update '(record {name = "Dupree"; description = "black dog"; keywords = vec {"funny tail"; "white nose"}})'
    ```

    `get`、`getSelf`、`search` 関数を使っても、`Dupree` のプロフィールが返ってくるだけです。

3.  `search` 関数を以下のように実行してみましょう。

    ```bash
    dfx canister call rust_profile search '("black")';
    ```

    このコマンドは `description` との一致を見つけ、以下のプロフィールを返します：

        (
          opt record {
            name = "Dupree";
            description = "black dog";
            keywords = vec { "funny tail"; "white nose" };
          },

## 新しい ID に対するプロフィールの追加

いまのところ、この Dapp はコマンドを実行したプリンシパルに関連付けられた１つのプロフィールしか保存しません。 関数 `get`, `getSelf`, `search` が思った通りに動作することをテストするために、異なるプロフィールを持つことができる新しい ID を追加する必要があります。

テストのために ID を追加します：

1.  以下のコマンドで新しいユーザー ID を作成します。

    ```bash
    dfx identity new Miles
    ```

        Creating identity: "Miles".
        Created identity: "Miles".

2.  `update` 関数を呼び、新しい ID に対応したプロフィールを追加します。

    ```bash
    dfx --identity Miles canister call rust_profile update '(record {name = "Miles"; description = "Great Dane"; keywords = vec {"Boston"; "mantle"; "three-legged"}})'
    ```

3.  `getSelf` 関数を呼び、`default` のユーザー ID に紐づけられたプロフィールを確認します。

    ```bash
    dfx canister call rust_profile getSelf
    ```

    このコマンドはデフォルトの ID に現在紐づけられているプロフィールを表示ます。この例では、Dupree のプロフィールです：

        (
          record {
            name = "Dupree";
            description = "black dog";
            keywords = vec { "funny tail"; "white nose" };
          },
        )

4.  `Miles` のユーザー ID を使って `getSelf` 関数を呼ぶには以下のコマンドを実行します：

    ```bash
    dfx --identity Miles canister call rust_profile getSelf
    ```

    このコマンドは Miles の ID に現在紐づいているプロフィールを表示します。この例では、以下のようになります：

        (
          record {
            name = "Miles";
            description = "Great Dane";
            keywords = vec { "Boston"; "mantle"; "three-legged" };
          },
        )

5.  正しいプロフィールが返されるかどうかをテストするために、説明の一部やキーワードを使って `search` 関数を呼び出しましょう。

    例えば、`Miles` のプロフィールが返されることを検証するために、以下のコマンドを実行します：

    ```bash
    dfx canister call rust_profile search '("Great")'
    ```

    このコマンドは `Miles` のプロフィールを返します：

        (
          opt record {
            name = "Miles";
            description = "Great Dane";
            keywords = vec { "Boston"; "mantle"; "three-legged" };
          },
        )

6.  正しいプロフィールが返されるかどうかをさらにテストするために、`search` 関数を呼びます。

    例えば、`Dupree` のプロフィールが返されることを検証するために、以下のコマンドを実行します：

    ```bash
    dfx canister call rust_profile search '("black")'
    ```

    このコマンドは `Dupree` のプロフィールを返します：

        (
          opt record {
            name = "Dupree";
            description = "black dog";
            keywords = vec { "funny tail"; "white nose" };
          },
        )

## サンプル Dapp の拡張

このサンプル Dapp は、ユーザー ID ごとに１つのプロフィールを保存するだけです。 もし各ユーザーのプロフィールにソーシャルコネクションへのリンクを追加するような拡張を行えば、LinkedUp サンプルアプリケーションを Rust で再現することができるでしょう。

## ローカル実行環境 を止める

アプリケーションのテストをした後は、ローカル実行環境 がバックグラウンドで稼働し続けないように、以下の手順で停止します：

1.  ネットワークの稼働状況が表示されている端末で、Control-C を押して ローカル実行環境 のプロセスを止めてください。

2.  以下のコマンドを用いて ローカル実行環境 を停止してください：

    ```bash
    dfx stop
    ```

<!--
# Adding and searching simple records

In this tutorial, you are going to write a dapp that provides a few basic functions to add and retrieve simple profile records that consist of a name, description, and an array of keywords.

This program supports the following functions:

-   The `update` function enables you to add a profile that consists of a `name`, a `description`, and `keywords`.

-   The `getSelf` function returns the profile for the principal associated with the function caller.

-   The `get` function performs a simple query to return the profile matching the `name` value passed to it. For this function, the name specified must match the `name` field exactly to return the record.

-   The `search` function performs a more complex query to return the profile matching all or part of the text specified in any profile field. For example, the `search` function can return a profile containing a specific keyword or that matches only part of a name or description.

This tutorial provides a simple example of how you can use the Rust CDK interfaces and macros to simplify writing dapps in Rust for the Internet Computer blockchain.

This tutorial demonstrates:
-   How to represent slightly more complex data—in the form of a profile as a `record` and an `array` of keywords—using the Candid interface description language.
-   How to write a simple search function with partial string matching.
-   How profiles are associated with a specific principal.

## Before you begin

Before you start your project, verify the following:

-   You have an internet connection and access to a shell terminal on your local macOS or Linux computer.

-   You have downloaded and installed the Rust programming language and Cargo as described in the [Rust installation instructions](https://doc.rust-lang.org/book/ch01-01-installation.html) for your operating system.

    ``` bash
    curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
    ```

    The Rust tool chain must be at version 1.46.0, or later.

-   You have downloaded and installed the DFINITY Canister Software Development Kit (SDK) package as described in [Download and install](../../../quickstart/hello10mins.md).

-   You have `cmake` installed. For example, use Homebrew with the following command:

    ``` bash
    brew install cmake
    ```

    For instructions on how to install Homebrew, see the [Homebrew Documentation](https://docs.brew.sh/Installation).

-   You have stopped any local execution environment processes running on your computer.

This tutorial takes approximately 20 minutes to complete.

## Create a new project

To create a new project directory for this tutorial:

1.  Open a terminal shell on your local computer, if you don’t already have one open.

2.  Create a new project by running the following command:

    ``` bash
    dfx new --type=rust rust_profile
    ```

3.  Change to your project directory by running the following command:

    ``` bash
    cd rust_profile
    ```

## Modify the default project

In the [Hello, World! Rust CDK Quick Start](./rust-quickstart.md), you went through the files in a default project with Rust type canister.

To complete this tutorial, you’ll need to complete the following steps:

-   [Replace the default dapp](#_replace_the_default_dapp)

-   [Update interface description file](#_update_interface_description_file)

### Replace the default dapp

Now that you have the files in place for your Rust dapp, we can replace the template `lib.rs` dapp with the Rust dapp we want to deploy on the Internet Computer blockchain.

To replace the default program:

1.  Check that you are still in the root directory for your project, if needed.

2.  Open the `src/rust_profile_backend/Cargo.toml` file in a text editor and add `serde` to dependencies.

    ``` toml
    [dependencies]
    ic-cdk = "0.5"
    ic-cdk-macros = "0.5"
    serde = "1.0"
    ```

3.  Open the template `src/rust_profile_backend/src/lib.rs` file in a text editor and delete the existing content.

    The next step is to add a Rust program that implements the `getSelf`, `update`, `get`, and `search` functions.

4.  Copy and paste [this code](../../_attachments/profile.rs) into the `lib.rs` file.

5.  Save your changes and close the file to continue.

## Update interface description file

Candid is an interface description language (IDL) for interacting with canisters running on the Internet Computer. Candid files provide a language-independent description of a canister’s interfaces including the names, parameters, and result formats and data types for each function a canister defines.

By adding Candid files to your project, you can ensure that data is properly converted from its definition in Rust to run safely on the Internet Computer blockchain.

To see details about the Candid interface description language syntax, see the [*Candid Guide*](../../candid/candid-intro.md) or the [Candid crate documentation](https://docs.rs/candid/).

To update Candid file for this tutorial:

1.  Check that you are still in the root directory for your project, if needed.

2.  Open the `src/rust_profile_backend/rust_profile_backend.did` file in a text editor.

3.  Copy and paste the following `Profile` type declaration and `service` definition for the `getSelf`, `update`, `get`, and `search` functions.
    ```did
    type Profile = record {
        "name": text;
        "description": text;
        "keywords": vec text;
    };

    service : {
        "getSelf": () -> (Profile) query;
        "get": (text) -> (Profile) query;
        "update": (Profile) -> ();
        "search": (text) -> (opt Profile) query;
    }
    ```

4.  Save your changes and close the file to continue.

## Start the local execution environment

Before you can build the `rust_profile` project, you need to connect to the local execution environment running in your development environment or the decentralized Internet Computer blockchain mainnet.

To start local execution environment:

1.  Check that you are still in the root directory for your project, if needed.

2.  Start the local execution environment on your computer in the background by running the following command:

    ``` bash
    dfx start --background --clean
    ```

    Depending on your platform and local security settings, you might see a warning displayed. If you are prompted to allow or deny incoming network connections, click **Allow**.

## Register, build, and deploy your project

After you connect to the local execution environment running in your development environment, you can register, build, and deploy your project locally.

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
        Creating canister "rust_profile_backend"...
        "rust_profile_backend" canister created with canister id: "rrkah-fqaaa-aaaaa-aaaaq-cai"
        Creating canister "rust_profile_frontend"...
        "rust_profile_frontend" canister created with canister id: "ryjl3-tyaaa-aaaaa-aaaba-cai"
        Building canisters...
        Executing: "cargo" "build" "--target" "wasm32-unknown-unknown" "--release" "-p" "rust_profile_backend"
        ...
            Finished release [optimized] target(s) in 6.31s
        Building frontend...
        Installing canisters...
        Creating UI canister on the local network.
        The UI canister on the "local" network is "r7inp-6aaaa-aaaaa-aaabq-cai"
        Installing code for canister rust_profile_backend, with canister_id rrkah-fqaaa-aaaaa-aaaaq-cai
        ...
        Deployed canisters.

## Call functions on the deployed canister

After successfully deploying the canister, you can test the canister by calling the functions it provides.

For this tutorial:

-   Call the `update` function to add a profile.

-   Call the `getSelf` function to display the profile for the principal identity.

-   Call the `search` function to look up the profile using a keyword.

To test the deployed canister:

1.  Call the `update` function to create a profile record by running the following command:

    ``` bash
    dfx canister call rust_profile_backend update '(record {name = "Luxi"; description = "mountain dog"; keywords = vec {"scars"; "toast"}})'
    ```

2.  Call the `getSelf` function to retrieve a profile record by running the following command:

    ``` bash
    dfx canister call rust_profile_backend getSelf
    ```

    The command returns the profile you used the `update` function to add. For example:

        (  record {
            name = "Luxi";
            description = "mountain dog";
            keywords = vec { "scars"; "toast" };
          },
        )

    In its current form, the dapp only stores and returns one profile. If you run the following command to add a second profile using the `update` function, the command replaces the `Luxi` profile with the `Dupree` profile:

    ``` bash
    dfx canister call rust_profile_backend update '(record {name = "Dupree"; description = "black dog"; keywords = vec {"funny tail"; "white nose"}})'
    ```

    You can use the `get`, `getSelf`, and `search` functions, but they will only return results for the `Dupree` profile.

3.  Run the following command to call the `search` function:

    ``` bash
    dfx canister call rust_profile_backend search '("black")';
    ```

    This command finds the matching profile using the `description` and returns the profile:

        (
          opt record {
            name = "Dupree";
            description = "black dog";
            keywords = vec { "funny tail"; "white nose" };
          },

## Adding profiles for new identities

In its current form, the dapp only stores one profile—the one associated with the principal invoking the commands. To test that the `get`, `getSelf`, and `search` functions do what we want them to, we need to add some new identities that can have different profiles.

To add identities for testing:

1.  Create a new user identity by running the following command:

    ``` bash
    dfx identity new Miles
    ```

        Creating identity: "Miles".
        Created identity: "Miles".

2.  Call the `update` function to add a profile for the new identity.

    ``` bash
    dfx --identity Miles canister call rust_profile_backend update '(record {name = "Miles"; description = "Great Dane"; keywords = vec {"Boston"; "mantle"; "three-legged"}})'
    ```

3.  Call the `getSelf` function to view the profile associated with the `default` user identity.

    ``` bash
    dfx canister call rust_profile_backend getSelf
    ```

    The command displays the profile currently associated with the default identity, in this example, the Dupree profile:

        (
          record {
            name = "Dupree";
            description = "black dog";
            keywords = vec { "funny tail"; "white nose" };
          },
        )

4.  Call the `getSelf` function using the `Miles` user identity by running the following command:

    ``` bash
    dfx --identity Miles canister call rust_profile_backend getSelf
    ```

    The command displays the profile currently associated with the Miles identity, in this example:

        (
          record {
            name = "Miles";
            description = "Great Dane";
            keywords = vec { "Boston"; "mantle"; "three-legged" };
          },
        )

5.  Call the `search` function using part of the description or a keyword to further test the whether the correct profile is returned.

    For example, to verify the `Miles` profile is returned, you might run the following command:

    ``` bash
    dfx canister call rust_profile_backend search '("Great")'
    ```

    The command returns the `Miles` profile:

        (
          opt record {
            name = "Miles";
            description = "Great Dane";
            keywords = vec { "Boston"; "mantle"; "three-legged" };
          },
        )

6.  Call the `search` function to further test the whether the correct profile is returned.

    For example, to verify the `Dupree` profile is returned, you might run the following command:

    ``` bash
    dfx canister call rust_profile_backend search '("black")'
    ```

    The command returns the `Dupree` profile:

        (
          opt record {
            name = "Dupree";
            description = "black dog";
            keywords = vec { "funny tail"; "white nose" };
          },
        )

## Extending the sample dapp

This sample dapp only stores one profile for each unique user identity. If you were to extend this dapp by adding a second method for linking social connections to each users profile, you would be well on your way to recreating the LinkedUp sample application using Rust.

## Stop the local execution environment

After you finish experimenting with your program, you can stop the local execution environment so that it doesn’t continue running in the background.

To stop the local execution environment:

1.  In the terminal that displays network operations, press Control-C to interrupt the local execution environment process.

2.  Stop the local execution environment by running the following command:

    ``` bash
    dfx stop
    ```

-->
