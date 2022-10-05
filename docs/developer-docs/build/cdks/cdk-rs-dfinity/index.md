# さまざまなプログラミング言語を用いた開発

このドキュメントに掲載されている Dapps の例のほとんどは、Motoko という IC で動作するように特別に設計されたプログラミング言語を使用しています。 しかし、IC 上に Dapps をデプロイするために、WebAssembly にコンパイル可能な任意のプログラミング言語で Dapps を書くことも可能です。 この章では、様々なプログラミング言語で Dapps を書き、IC 上にデプロイするためのハイレベルなガイダンスを用意しています。

## Rust を使用する

Cargo を使用し、Dapp を WebAssembly にコンパイルすることで、IC 上で動作する Rust プロジェクトを作成することができます。

この章では、Rust プログラムを Canister スマートコントラクトとして IC 上にデプロイする際の主要な手順をまとめています。 ただし、ここで説明している手順は、1 つのアプローチを示しているに過ぎないことに注意してください。 他のアプローチも可能です。

なお、[Rust canister development kit (Rust CDK)](https://github.com/dfinity/cdk-rs) では、クエリコールやアップデートコールの関数を書きやすくするためのショートカットが用意されており、Rust ベースのプロジェクトを作り始めるためのいくつかの [例](https://github.com/dfinity/cdk-rs/tree/next/examples) が含まれていますが、Rust CDK を使わなくても IC 用の Dapps を開発することができます。

### プロジェクトの作成

ほとんどの Rust プログラマーは、Cargo を使って Dapp が依存しているライブラリのダウンロードやコンパイルなどのビルドやパッケージ管理のタスクを処理しています。そのため、最初のステップは Cargo のコマンドラインインターフェイスを使って新しい Rust プロジェクトを作成することです。

Cargo の代わりに {sdk-long-name} を使って新しいプロジェクトを作成することもできますが、Cargo を使ってプロジェクトを作成するのが Rust プロジェクト作成の典型的なワークフローです。

新しい Rust プロジェクトを作成するには、以下のようにします：

1.  ローカル PC でターミナルを開きます。

2.  以下のコマンドで Cargo がインストールされているかどうかを確認します：

        cargo --version

3.  IC か Rust サンプルプロジェクトのために使用しているディレクトリに移動してください。

4.  以下のコマンドを実行して、新しいプロジェクトを作成します：

        cargo new my_rust_dapp

    このコマンドは、`Cargo.toml` ファイルが入った `my_rust_dapp` ディレクトリと、`main.rs` ファイルが入った `src` ディレクトリを新たに作成します。

5.  以下のコマンドを実行してプロジェクトディレクトリに移動します：

        cd my_rust_dapp

    このディレクトリの中身を表示すると、`Cargo.toml` ファイルと `src` ディレクトリだけが含まれていることがわかります。 このプロジェクトを IC 上で動作するようにコンパイルするには、いくつかの追加ファイルが必要になります。

### Cargo 設定ファイルの修正

`Cargo.toml` ファイルでは、それぞれの Rust パッケージごとに **マニフェスト** を用意しています。 マニフェストには、パッケージの設定の詳細を指定するセクションが含まれています。 Rust プロジェクトを IC 上で実行できるようにするために、デフォルトの `Cargo.toml` ファイルをコピーして、プロジェクトの設定の詳細をいくつか変更します。

`Cargo.toml` ファイルを修正するには、以下のようにします：

1.  必要に応じて `pwd` コマンドを使い、自分がプロジェクトのルートディレクトリにいることを確認します。

2.  デフォルトの `Cargo.toml` ファイルを `src` ディレクトリに以下のコマンドでコピーします：

        cp Cargo.toml src/Cargo.toml

    IC 上で実行されるプロジェクトでは通常、プロジェクトのルートディレクトリにある `Cargo.toml` ファイルを使用してプロジェクトの Canister 群のワークスペースを設定し、ソースコードのディレクトリにある 2 つ目の `Cargo.toml` ファイルを使用して、各 Canister の設定を行います。

3.  プロジェクトのルートディレクトリにある `Cargo.toml` ファイルをテキストエディタで開きます。

    デフォルトでは、`[package]` と `[dependencies]` のセクションが含まれています。

4.  `[package]` セクションを、以下のように `[workspace]` セクションに置き換えます：

        [workspace]
        members = [
            "src/my_rust_dapp",
        ]

    `[workspace]` のセクションで使用するキーについては、[Workspaces](https://doc.rust-lang.org/cargo/reference/workspaces.html) を参照してください。 `Cargo.toml` ファイルで設定できるその他のセクションやキーについては、[The Manifest Format](https://doc.rust-lang.org/cargo/reference/manifest.html) を参照してください。

5.  `[dependencies]` セクションを削除します。

6.  変更を保存しファイルを閉じて次に進んでください。

7.  `src/Cargo.toml` ファイルをテキストエディタで開きます。

8.  以下のように、main のソースコードへのパスを指定する `[lib]` セクションを追加します：

        [lib]
        path = "main.rs"

9.  パッケージの依存関係については適宜 `[dependencies]` セクションに追加してください。

10. 変更を保存しファイルを閉じて次に進んでください。

### Canister 設定ファイルの追加

SDK を使用して新しいプロジェクトを作成すると、`dfx new` コマンドが自動的にデフォルトの設定ファイルである `dfx.json` をプロジェクトディレクトリに追加します。 ここでは Cargo を使って Rust プロジェクトを作成したので、`dfx.json` をプロジェクトディレクトリに手動で作成する必要があります。

`dfx.json` を追加するには、以下のようにします：

1.  必要に応じて `pwd` コマンドを使い、自分がプロジェクトのルートディレクトリにいることを確認します。

2.  `dfx.json` 設定ファイルをプロジェクトのルートディレクトリに新しく作成します。

3.  `dfx.json` ファイルをテキストエディタで開きます。

4.  `version` と　`canisters` キーを以下のように追加します：

        {
          "version": 1,
          "canisters": {
            "my_rust_dapp": {
              "type": "custom",
              "candid": "src/my_rust_dapp.did",
              "wasm": "target/wasm32-unknown-unknown/debug/my_rust_dapp.wasm",
              "build": "cargo build --target wasm32-unknown-unknown --package my_rust_dapp"
            }
          }
        }

    この設定について詳しく見てみましょう。

    - `version` の設定は、プロジェクトの作成に使用したソフトウェアのバージョンを確認するために使用されます。

    - `canisters` セクションでは、プロジェクトの Canister の名前を指定します。 この例では、`my_rust_dapp` という名前の Canister が 1 つ指定されています。

    - `type` キーが `custom` なのは、この Canister が現在認識されている Canister タイプ (`motoko` か `assets`) ではないからです。

    - `candid` キーは、このプロジェクトで使用する Candid インターフェース記述ファイルの名前と場所を指定します。

    - `wasm` キーは、`cargo build` コマンドで生成される WebAssembly ファイルのパスを指定します。

    - `build` キーは、コンパイルに使用する `cargo` コマンドを指定します。

    これらは必要最小限の設定です。 より複雑なプログラムを作成する際には、`Cargo.toml` ファイル、`dfx.json` ファイル、あるいはその両方に、追加の設定情報を含める必要が出る可能性があります。

5.  変更を保存し、ファイルを閉じて次に進んでください。

### Candid インターフェース記述ファイルの作成

設定ファイルである `dfx.json` に加えて、Candid インターフェース記述ファイル（例えば、`my_rust_dapp.did`）を用意する必要があります。このファイルは、Dapp の引数や返り値のフォーマットを、Candid での言語にとらわれない表現にマッピングするために必要です。

Candid インターフェース記述ファイルを追加するには、以下のようにします：

1.  必要に応じて `pwd` コマンドを使い、自分がプロジェクトのルートディレクトリにいることを確認します。

2.  プロジェクトの `src` ディレクトリに、Candid インターフェース記述ファイル（例えば、`my_rust_dapp.did`）を新たに作成します。

3.  Candid インターフェース記述ファイルをテキストエディタで開き、Dapp が定義する各関数に対する記述を追加します。

    例えば、`my_rust_dapp` が `increment`、`read`、`write` 関数を使ってカウンタをインクリメントするシンプルな dapp である場合、`my_rust_dapp.did` ファイルは以下のようになります：

        service : {
          "increment": () -> ();
          "read": () -> (nat) query;
          "write": (nat) -> ();
        }

4.  変更を保存しファイルを閉じて次に進んでください。

### デフォルトの Dapp の修正

新しいプロジェクトを作成すると、"Hello, World!" プログラムのテンプレートファイルである `main.rs` ファイルが `src` ディレクトリに作られます。

このテンプレートのソースコードを修正するには以下のようにします：

1.  `src/main.rs` ファイルをテキストエディタで開き、中身を削除します。

2.  IC にデプロイしたいプログラムを書きます。

    プログラムを書く際には、呼び出しには「アップデートコール」と「クエリコール」の 2 種類があることと、アップデート関数は非同期メッセージングを行うことに注意してください。

3.  変更を保存して、`main.rs` ファイルを閉じます。

### Dapp のデプロイ

Dapp をデプロイしてテストする前に、以下を行う必要があります。

- ローカルの Canister 実行環境、または IC ブロックチェーンのメインネットのいずれかに接続します。

- アプリケーションにネットワーク固有の識別子を登録します。

- WebAssembly をターゲット出力として Dapp をコンパイルします。

WebAssembly にコンパイルする `cargo build` コマンドを `dfx.json` ファイルに設定したので、`dfx` コマンドラインインターフェイスと標準的なワークフローによって残りのすべてのステップを実行することができます。

Dapp をローカルでビルドとデプロイするには以下のようにします：

1.  必要に応じて `pwd` コマンドを使い、自分がプロジェクトのルートディレクトリにいることを確認します。

2.  新しいターミナルの窓あるいはタブを開き、プロジェクトディレクトリへ移動します。

    例えば、macOS で Terminal を使用している場合は、以下のどちらかを行います：

    - **Shell** をクリックし、**New Tab** を選択して、現在の作業ディレクトリに新しいターミナルを開きます。

    - **Shell** をクリックし、 **New Window** を選択して、`cd ~/ic-projects/location_hello` を新しいターミナルで実行してください（`ic-projects` フォルダの中に `location_hello` プロジェクトがある場合）。

    今手元では、プロジェクトディレクトリを作業ディレクトリとした、2 つのターミナルが開かれているはずです。

3.  以下のコマンドを実行して、ローカルの Canister 実行環境を起動します：

        dfx start

    使用しているプラットフォームやローカルのセキュリティ設定によっては、警告が表示される場合があります。 受信するネットワーク接続を許可するか拒否するかを選択する操作画面が表示された場合は、**Allow** をクリックしてください。

4.  ネットワーク操作を表示している端末を開いたまま、プロジェクトを作成した元の端末にフォーカスを切り替えます。

5.  以下のコマンドを実行して、アプリケーションに固有の Canister ID を登録します：

        dfx canister create --all

6.  以下のコマンドを実行して Dapp をビルドします：

        dfx build

7.  以下のコマンドを実行して Dapp をローカルの Canister 実行環境にデプロイします：

        dfx canister install --all

8.  コマンドライン、あるいはブラウザから、Dapp の関数をテストしてください。

<!--
# Develop using different languages

Rust is a powerful and type sound modern programming language with an active developer community. Because Rust compiles to WebAssembly, it offers a rich development environment for writing dapps to run on the Internet Computer blockchain. To help pave the way for writing dapps in Rust that can be deployed on the Internet Computer blockchain, DFINITY provides some tools to simplify the process.

Collectively, these tools are referred to as the DFINITY Canister Development Kit (CDK) for Rust and consist of the following main libraries:

| Package            | Description                                                                                                                                                                                                             |
|--------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `ic-types`         | The `ic-types` crate defines the types used to communicate with the decentralized Internet Computer blockchain, and when building dapps to be deployed as canisters on the Internet Computer blockchain. |
| `ic-agent`         | The `ic-agent` library enables direct communication with the Internet Computer blockchain.                                                                                                                              |
| `ic-utils`         | The `ic-utils` library provides utilities for managing calls and dapps deployed as canisters.                                                                                                            |
| `ic-cdk`           | The `ic-cdk` provides the core methods that enable Rust programs to interact with the Internet Computer blockchain system API. This library serves as the runtime core of the Rust CDK.                                 |
| `ic-cdk-macros`    | The `ic-cdk-macros` library defines the procedural macros that facilitate building operation endpoints and APIs. This library includes macros for `update`, `query`, `import` and other important operations.           |
| `ic-cdk-optimizer` | The `ic-cdk-optimizer` is a helper library used to reduce the size of WebAssembly modules.                                                                                                                              |

The following diagram provides a simplified view of the Rust Canister Development Kit (CDK) building blocks from the lowest to highest level of abstraction.

![Rust building blocks](../_attachments/../../_attachments/Rust-building-blocks.svg)

## Using Rust without SDK

You can create Rust projects to run on the IC by using Cargo and compiling your dapp to use WebAssembly as the target output.

This section provides a detailed summary of the key steps involved in deploying a Rust program as a canister on the IC. You should note, however, that the steps described here only illustrate one approach. Other implementation approaches are also possible.

Note that the [Rust canister development kit (Rust CDK)](https://github.com/dfinity/cdk-rs) provides some shortcuts to make it easier to write functions as query and update calls and includes several [examples](https://github.com/dfinity/cdk-rs/tree/next/examples) to get you started building Rust-based projects, but you can also develop dapps for the IC without using the Rust CDK. Also, it is much easier to use `dfx` instead of setting up the project from scratch. See the [Rust Quickstart](./rust-quickstart.md) for a much simpler walkthrough.

### Create a project

Because most Rust programmers use Cargo to handle build and package management tasks, such as downloading and compiling the libraries your dapp depends on, your first step is to create a new Rust project using the Cargo command-line interface.

Alternatively, you could create a new project using SDK instead of Cargo, but creating a project using Cargo represents the typical workflow for creating Rust projects.

To create a new Rust project:

1.  Open a terminal shell on your local computer, if you don’t already have one open.

2.  Verify that you have Cargo installed by running the following command:

        cargo --version

3.  Change to the folder you are using for your IC or Rust sample projects.

4.  Create a new project by running a command similar to the following:

        cargo new my_rust_dapp

    This command creates a new `my_rust_dapp` directory with a default `Cargo.toml` file and a `src` directory with a default `main.rs` file.

5.  Change to your project directory by running the following command:

        cd my_rust_dapp

    If you list the contents of this directory, you’ll see that it only contains the `Cargo.toml` file and `src` directory. To compile this project to run on the IC, you’ll need some additional files.

### Modify the Cargo configuration file

The `Cargo.toml` file provides a **manifest** for each Rust package. The manifest contains sections that specify configuration details for the package. To prepare the Rust project to run on the IC, we’ll copy the default `Cargo.toml` file then modify some of the configuration details for the project.

To modify the `Cargo.toml` file:

1.  Check that you are in the root directory for your project by running the `pwd` command, if necessary.

2.  Copy the default `Cargo.toml` file to the `src` directory by running the following command:

        cp Cargo.toml src/Cargo.toml

    Projects that run on the IC typically use one project-level `Cargo.toml` file to set up a workspace for the canister members of the project and a second `Cargo.toml` file in the source code directory to configure settings for each canister.

3.  Open the `Cargo.toml` file that is the root directory of your project in a text editor.

    By default, the file contains the `[package]` and the `[dependencies]` sections.

4.  Replace the `[package]` section with a `[workspace]` section similar to the following:

        [workspace]
        members = [
            "src/my_rust_dapp",
        ]

    For information about the `[workspace]` section and `[workspace]` keys, see [Workspaces](https://doc.rust-lang.org/cargo/reference/workspaces.html). For information about the other sections and keys you can configure in the `Cargo.toml` file, see [The Manifest Format](https://doc.rust-lang.org/cargo/reference/manifest.html).

5.  Remove the `[dependencies]` section.

6.  Save your changes and close the file to continue.

7.  Open the `src/Cargo.toml` file in a text editor.

8.  Add a `[lib]` section with the path to the main source code similar to the following:

        [lib]
        path = "main.rs"

9.  Update the `[dependencies]` section with any package dependencies.

10. Save your changes and close the file to continue.

### Add a canister configuration file

When you create a new project using the SDK, the `dfx new` command automatically adds a default `dfx.json` configuration file to the project directory. Because we created the Rust project using Cargo, you need to manually create this file in your project directory.

To add the `dfx.json` configuration file:

1.  Check that you are still in your project directory by running the `pwd` command, if necessary.

2.  Create a new `dfx.json` configuration file in the root directory for your project.

3.  Open the `dfx.json` file in a text editor.

4.  Add the `version` and `canisters` keys with settings similar to the following to the `dfx.json` file:

        {
          "version": 1,
          "canisters": {
            "my_rust_dapp": {
              "type": "custom",
              "candid": "src/my_rust_dapp.did",
              "wasm": "target/wasm32-unknown-unknown/debug/my_rust_dapp.wasm",
              "build": "cargo build --target wasm32-unknown-unknown --package my_rust_dapp"
            }
          }
        }

    Let’s take a closer look at these settings.

    -   The `version` setting is used to identify the version of the software used to create the project.

    -   The `canisters` section specifies the name of the project’s canisters. In this case, there’s only one canister and it is named `my_rust_dapp`.

    -   The `type` key is set to `custom` because this canister is not one of the currently recognized (`motoko` or `assets`) canister types.

    -   The `candid` key specifies the name and location of the Candid interface description file to use for this project.

    -   The `wasm` key specifies the path to the WebAssembly file generated by the `cargo build` command.

    -   The `build` key specifies the `cargo` command used to compile the output.

    These are the minimum settings required. As you build more complex programs, you might need to include additional configuration details in the `Cargo.toml` file, the `dfx.json` file, or both files.

5.  Save your changes and close the file to continue.

### Create a Candid interface description file

In addition to the `dfx.json` configuration file, you need to have a Candid interface description file—for example, `my_rust_dapp.did`—to map your dapp’s input parameters and return value formats to their language-agnostic representation in Candid.

To add the Candid interface description file:

1.  Check that you are still in your project directory by running the `pwd` command, if necessary.

2.  Create a new Candid interface description file—for example, `my_rust_dapp.did`—in the `src` directory for your project.

3.  Open the Candid interface description file in a text editor and add a description for each function the dapp defines.

    For example, if the `my_rust_dapp` is a simple dapp that increments a counter using the `increment`, `read`, and `write` functions, the `my_rust_dapp.did` file might look like this:

        service : {
          "increment": () -> ();
          "read": () -> (nat) query;
          "write": (nat) -> ();
        }

4.  Save your changes and close the file to continue.

### Modify the default dapp

When you create a new project, your project `src` directory includes a template `main.rs` file with the "Hello, World!" program.

To modify the template source code:

1.  Open the template `src/main.rs` file in a text editor and delete the existing content.

2.  Write the program you want to deploy on the IC.

    As you write your program, keep in mind that there are two types of calls—update calls and query calls—and that update functions use asynchronous messaging.

3.  Save your changes and close the `main.rs` file.

### Deploy the dapp

Before you can deploy and test your dapp, you need to do the following:

-   Connect to either the local canister execution environment, or to the IC blockchain mainnet.

-   Register a network-specific identifier for the application.

-   Compile the dapp with a target output of WebAssembly.

Because you configured the custom `dfx.json` file with a `cargo build` command that compiles to WebAssembly, you can use the `dfx` command-line interface and standard work flow to perform all of the remaining steps.

To build and deploy the dapp locally:

1.  Check that you are still in your project directory by running the `pwd` command, if necessary.

2.  Open a new terminal window or tab on your local computer and navigate to your project directory.

    For example, you can do either of the following if running Terminal on macOS:

    -   Click **Shell**, then select **New Tab** to open a new terminal in your current working directory.

    -   Click **Shell** and select **New Window**, then run `cd ~/ic-projects/location_hello` in the new terminal if your `location_hello` project is in the `ic-projects` working folder.

    You should now have two terminals open with your project directory as your current working directory\*\*.

3.  Start the local canister execution environment by running the following command:

        dfx start

    Depending on your platform and local security settings, you might see a warning displayed. If you are prompted to allow or deny incoming network connections, click **Allow**.

4.  Leave the terminal that displays network operations open and switch your focus to your original terminal where you created your project.

5.  Register a unique canister identifier for the application by running the following command:

        dfx canister create --all

6.  Build the dapp by running the following command:

        dfx build

7.  Deploy the dapp on the local canister execution environment by running the following command:

        dfx canister install --all

8.  Test functions in the dapp from the command-line or in a browser.

-->
