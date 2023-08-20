# ライブラリモジュールのインポート

このチュートリアルでは、電話番号の保存と検索を可能にする簡単な Dapp を作成します。このチュートリアルでは、いくつかの基本的な Motoko ライブラリ関数をインポートして使用する方法を説明します。

このチュートリアルでは、Motoko の基本ライブラリ関数は `List` と `AssocList` モジュールで定義されており、リンクされたキーとバリューのペアでリストを操作できるようにします。この例では、**key** は `name` で、**value** はその名前に関連付けられた `phone` というテキスト文字列です。

この Dapp は以下の関数コールをサポートしています：

-   `insert` 関数は、変数 `book` に格納された `name` と `phone` のキーとバリューのペアを入力として受け取ります。

-   `lookup` 関数は、指定された `name` キーを入力として、関連する電話番号を検索するクエリです。

## 始める前に

チュートリアルを始める前に、以下を確認してください：

-   [ダウンロードとインストール](../../quickstart/local-quickstart#download-and-install)で説明されているように、SDK パッケージをダウンロードしインストールする。

-   `dfx`が提供するローカル Canister の実行環境を停止する。

このチュートリアルの所要時間は約 10分です。

## 新しいプロジェクトを作成する

このチュートリアルのための新しいプロジェクトを作成するには：

1.  ローカルコンピューターでターミナルシェルを開いてください（まだ開いていない場合）。

2.  Internet Computer のプロジェクトに使用しているフォルダがあれば、そこに移動します。

3.  以下のコマンドを実行して、新しいプロジェクトを作成します：

        dfx new phonebook

4.  以下のコマンドを実行して、プロジェクトディレクトリに移動してください：

        cd phonebook

## デフォルトの Dapp を修正する

このチュートリアルでは、シンプルな電話番号検索 Dapp 用に新しい `main.mo` ファイルを作成しましょう。

デフォルトのテンプレートを変更するには：

1.  テキストエディタで `src/phonebook/main.mo` ファイルを開き、既存の内容を削除してください。

2.  [このコード](../_attachments/phonebook.mo) をコピーして `main.mo` ファイルに貼り付けてください。

    このサンプルアプリを見ると、次のような重要な要素に気づくかもしれません：

    -   このコードでは、`Name` と `Phone` をカスタムの Text タイプとして定義しています。ユーザー定義の型を作成することで、コードの可読性を向上させることができます。

    -   `insert` 関数はアップデートコールで、`lookup` 関数はクエリコールです。

    -   `Phone` 型は、`?Phone` 構文を使ってオプションの値として識別されます。

## ローカル Canister の実行環境を起動する

開発用に `dfx` はローカル Canister の実行環境を提供します。これには `dfx.json` ファイルが必要ですので、プロジェクトのルートディレクトリにいることを確認する必要があります。このチュートリアルでは、ターミナルシェルをふたつ用意して、ひとつのターミナルでローカル Canister 実行環境を起動して出力を確認し、もうひとつのターミナルでプロジェクトを管理するようにしてください。

ローカル Canister 実行環境を開始するには、以下の手順に従います：

1.  ローカルコンピューターで新しいターミナルウィンドウまたはタブを開きます。

2.  必要であれば、プロジェクトのルートディレクトリに移動します。

    -   これで、**ふたつのターミナル**が開いた状態になっているはずです。

    -   **プロジェクトディレクトリ**を、**現在の作業ディレクトリ**とする必要があります。

3.  ローカルコンピューターで以下のコマンドを実行して、ローカル Canister の実行環境を起動します：

        dfx start --clean

    このチュートリアルでは、`--clean` オプションを使用して、ローカル Canister 実行環境をクリーンな状態で起動します。

    このオプションは、通常の操作を中断させる可能性のある、孤立したバックグラウンドプロセスや Canister 識別子を削除します。例えば、プロジェクト間を移動する際に `dfx stop` を発行するのを忘れた場合、バックグラウンドや他のターミナルでプロセスが動作している可能性があります。`--clean` オプションを使用すると、実行中のプロセスを手動で見つけて終了させることなく、ローカル canister 実行環境を起動して次のステップに進むことができるようになります。

4.  ローカル Canister の実行環境の出力を表示しているターミナルを開いたまま、新しいプロジェクトを作成した元のターミナルにフォーカスを切り替えます。

## Dapp の登録、ビルド、デプロイ

開発環境において、ローカル Canister 実行環境が整えば、その上に Dapp を登録、ビルド、デプロイすることができます。

ローカルに Dapp をデプロイするには：

1.  必要に応じて、まだプロジェクトのルートディレクトリにいることを確認します。

2.  以下のコマンドを実行して、ローカルに Dapp を登録し、ビルド、デプロイします：

        dfx deploy phonebook

    `dfx.json` ファイルは、Dapp のフロントエンドエントリーポイントと `assets` Canister を作成するためのデフォルト設定を提供します。

    以前のチュートリアルでは、サンプルアプリのフロントエンドを追加しないため、アセット Canister のエントリーを削除しました。この変更により、プロジェクトのワークスペースが整理され、使われなくなるファイルがなくなりました。しかし、これを行う必要はなく、`dfx.json` ファイルにアセット Canister の記述を残しておいても問題はありません。例えば、後でフロントエンドアセットを追加するつもりなら、プレースホルダーとして使用することができます。

    このチュートリアルでは、`dfx deploy phonebook` コマンドを使用して電話帳のバックエンド Canister だけをデプロイすることができます。

    このチュートリアルでは、フロントエンド Canister のコンパイルを省略する方法を説明しますが、後で [examples](https://github.com/dfinity/examples) リポジトリの [phone-book](https://github.com/dfinity/examples/tree/master/motoko/phone-book) プロジェクトを調べれば、この Dapp に簡単なユーザーインターフェイスを追加することが可能です。

## insert 関数で名前や数字を追加する

これで、ローカル Canister 実行環境に **Canister** として Dapp がデプロイされ、`dfx canister call` コマンドを使用して Dapp をテストできるようになりました。

デプロイした Dapp をテストするには、以下のようにします：

1.  `dfx canister call` コマンドを使用して、`insert` 関数で Canister `phonebook` を呼び出し、以下のコマンドを実行して、名前と電話番号を渡します：

        dfx canister call phonebook insert '("Chris Lynn", "01 415 792 1333")'

2.  次のコマンドを実行して、2つ目の名前と番号のペアを追加します：

        dfx canister call phonebook insert '("Maya Garcia", "01 408 395 7276")'

3.  次のコマンドを実行して、`lookup` 関数を使って  "Chris Lynn “ に関連する番号が返されることを確認します：

        dfx canister call phonebook lookup '("Chris Lynn")'

    このコマンドは、次のような出力を返します：

        (opt "01 415 792 1333")

4.  以下のコマンドを実行して、"Maya Garcia “ に関連付けられた番号で `lookup` 関数を呼び出してみてください：

        dfx canister call phonebook lookup '("01 408 395 7276")'

    この場合、電話番号は "Maya Garcia" という名前のエントリに関連付けられたキーではないので、コマンドは `(null)` を返すことに注意してください。

5.  もう一度 `lookup` 関数を呼び出して、"Maya Garcia" と "Chris Lynn" の両方の電話番号を返すように、次のコマンドを実行してみてください：

        dfx canister call phonebook lookup '("Maya Garcia","Chris Lynn")'

    ひとつのキーに対してひとつの値を返すように書かれているので、コマンドは最初のキーに関連する情報、この例では `Maya Garcia` の電話番号のみを返します。

6.  Candid UI を使用してコードをテストしてください。

    コードをテストするには、[ここ](candid-ui.md)の指示に従います。

![電話帳機能](../_attachments/candid-phonebook.png)

## Dapp のソースコードを修正する

このチュートリアルで学んだことを発展させるために、ソースコードを修正して別の結果を返すようにしてみてはいかがでしょうか。

例えば、現在の Key-Value（名前-電話番号）のペアを挿入して検索するアプリの代わりに、データベースの「レコード」のように、主キーが複数のフィールドに関連付けられた連絡先情報を保存するアプリにするためにソースコードを変更したいと思うかもしれません。この例では、ユーザーや別のアプリが、自宅の電話番号、携帯電話番号、メールアドレス、住所などの情報を追加し、すべてのフィールドまたは特定のフィールドの値を選択的に返すことができるようにすることができます。

## ローカル Canister の実行環境を停止する

Dapp の実験が終わったら、ローカル Canister 実行環境を停止して、バックグラウンドで実行し続けないようにすることができます。

ローカル Canister 実行環境を停止するには、以下の手順に従います：

1.  ネットワーク操作が表示されているターミナルで、Control-C キーを押して、ローカルネットワークの処理を中断します。

2.  以下のコマンドを実行して、ローカル Canister 実行環境を停止します。

        dfx stop

<!--
# Import library modules

In this tutorial, you are going to write a simple dapp that enables you to store and look up telephone numbers. This tutorial illustrates how to import and use a few basic Motoko library functions.

For this tutorial, the Motoko base library functions are defined in the `List` and `AssocList` modules and enable you to work with lists as linked key-value pairs. In this example, the **key** is a `name` and the **value** is the `phone` text string associated with that name.

This dapp supports the following function calls:

-   The `insert` function accepts the `name` and `phone` key-value pair as input stored in the `book` variable.

-   The `lookup` function is a query that uses the specified `name` key as input to find the associated phone number.

## Before you begin

Before starting the tutorial, verify the following:

-   You have downloaded and installed the SDK package as described in [Download and install](../../quickstart/local-quickstart#download-and-install).

-   You have stopped the local canister execution environment provided by `dfx`.

This tutorial takes approximately 10 minutes to complete.

## Create a new project

To create a new project for this tutorial:

1.  Open a terminal shell on your local computer, if you don’t already have one open.

2.  Change to the folder you are using for your Internet Computer projects, if you are using one.

3.  Create a new project by running the following command:

        dfx new phonebook

4.  Change to your project directory by running the following command:

        cd phonebook

## Modify the default dapp

For this tutorial, let’s create a new `main.mo` file for the simple phone number lookup dapp.

To modify the default template:

1.  Open the `src/phonebook/main.mo` file in a text editor and delete the existing content.

2.  Copy and paste [this code](../_attachments/phonebook.mo) into the `main.mo` file.

    In looking at this sample dapp, you might notice the following key elements:

    -   The code defines `Name` and `Phone` as custom Text types. Creating user-defined types improves the readability of the code.

    -   The `insert` function is an update call and the `lookup` function is a query call.

    -   The `Phone` type is identified as an optional value by using the `?Phone` syntax.

## Start the local canister execution environment

For development purposes `dfx` provides a local canister execution environment. This requires a `dfx.json` file, so you should be sure you are in your project’s root directory. For this tutorial, you should have two separate terminal shells, so that you can start and see the output of the local canister execution environment in one terminal and manage your project in another.

To start the local canister execution environment:

1.  Open a new terminal window or tab on your local computer.

2.  Navigate to the root directory for your project, if necessary.

    -   You should now have **two terminals** open.

    -   You should have the **project directory** as your **current working directory**.

3.  Start the local canister execution environment on your local computer by running the following command:

        dfx start --clean

    For this tutorial, we’re using the `--clean` option to start the local canister execution environment in a clean state.

    This option removes any orphan background processes or canister identifiers that might disrupt normal operations. For example, if you forgot to issue a `dfx stop` when moving between projects, you might have a process running in the background or in another terminal. The `--clean` option ensures that you can start the local canister execution environment and continue to the next step without manually finding and terminating any running processes.

4.  Leave the terminal that displays the output of the local canister execution environment open and switch your focus to your original terminal where you created your new project.

## Register, build, and deploy the dapp

Once the local canister execution environment is up and running in your development environment, you can register, build, and deploy your dapp onto it.

To deploy the dapp locally:

1.  Check that you are still in the root directory for your project, if needed.

2.  Register, build, and deploy your dapp locally by running the following command:

        dfx deploy phonebook

    The `dfx.json` file provides default settings for creating a dapp frontend entry point and `assets` canister.

    In previous tutorials, we deleted the entries for the asset canister because we were not adding a frontend for the sample dapp. That change kept our project workspace tidy by eliminating files that would go unused. There is no requirement to do this, however, and there is no harm in leaving the asset canister description in the `dfx.json` file. For example, you might want to use it as a placeholder if you intend to add frontend assets later.

    For this tutorial, you can deploy just the phonebook backend canister using the `dfx deploy phonebook` command because the project doesn’t include any frontend assets and you will interact with it via the terminal.

    Although this tutorial illustrates how to skip compiling a frontend canister, you can add a simple user interface to this dapp later by exploring the [phone-book](https://github.com/dfinity/examples/tree/master/motoko/phone-book) project in the [examples](https://github.com/dfinity/examples) repository.

## Add names and numbers using the insert function

You now have a dapp deployed as a **canister** on your local canister execution environment and can test your dapp by using `dfx canister call` commands.

To test the dapp you have deployed:

1.  Use the `dfx canister call` command to call the canister `phonebook` using the `insert` function and pass it a name and phone number by running the following command:

        dfx canister call phonebook insert '("Chris Lynn", "01 415 792 1333")'

2.  Add a second name and number pair by running the following command:

        dfx canister call phonebook insert '("Maya Garcia", "01 408 395 7276")'

3.  Verify that the command returns the number associated with "Chris Lynn" using the `lookup` function by running the following command:

        dfx canister call phonebook lookup '("Chris Lynn")'

    The command returns output similar to the following:

        (opt "01 415 792 1333")

4.  Try to call the `lookup` function with the number associated with "Maya Garcia" by running the following command:

        dfx canister call phonebook lookup '("01 408 395 7276")'

    Note that, in this case, the command returns `(null)` because the phone number is not a key associated with the "Maya Garcia" name entry.

5.  Try to call the `lookup` function again to return the phone numbers for both "Maya Garcia" and "Chris Lynn" by running the following command:

        dfx canister call phonebook lookup '("Maya Garcia","Chris Lynn")'

    Because the dapp is written to return one value for one key, the command only returns information associated with the first key, in this example the phone number for `Maya Garcia`.

6.  Test your code using the candid ui.

    To test your code, follow the instructions [here](candid-ui.md).
![Phonebook functions](../_attachments/candid-phonebook.png)

## Revise the source code in your dapp

To extend what you have learned in this tutorial, you might want to try modifying the source code to return different results.

For example, you might want to change the source code so that instead of a dapp that inserts and looks up a current key-value (name-phone) pair to create a dapp that stores contact information similar to a database "record" in which a primary key is associated with multiple fields. In this example, your dapp might enable users or another dapp to add information—such as a home phone number, a cell phone number, an email address, and a street address—and selectively return all or specific field values.

## Stop the local canister execution environment

After you finish experimenting with your dapp, you can stop the local canister execution environment so that it does not continue running in the background.

To stop the local canister execution environment:

1.  In the terminal that displays network operations, press Control-C to interrupt the local network process.

2.  Stop the local canister execution environment by running the following command:

        dfx stop

-->