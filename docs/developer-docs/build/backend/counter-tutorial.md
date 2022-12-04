# 自然数をインクリメントする

このチュートリアルでは、カウンターをインクリメントするひとつの Actor を作成し、いくつかの基本的な関数を提供するプログラムを書くことで値の永続性を説明します。

このチュートリアルでは、Actor は `Counter` と名付けます。このプログラムは `currentValue` 変数を使用して、カウンターの現在値を表す自然数を格納します。このプログラムは以下の関数呼び出しをサポートしています：

-   `increment` 関数の呼び出しは、現在の値を更新し、1 だけ増加させます （返り値はありません）。

-   `get` 関数呼び出しは、カウンタの現在値を照会して返します。

-   `set` 関数呼び出しは、現在の値を引数として指定した任意の数値に更新します。

このチュートリアルでは、配置された Canister で関数を呼び出してカウンタをインクリメントする方法を簡単に説明します。カウンタ値をインクリメントしてクエリする関数を複数回呼び出すことで、変数のステート、つまり呼び出し間の変数値が永続していることを確認できます。

## 始める前に

チュートリアルを始める前に、以下を確認してください：

-   [ダウンロードとインストール](../../quickstart/local-quickstart#download-and-install)の説明に従って、SDK パッケージをダウンロードし、インストールする。

-   ローカルコンピューターで動作しているすべての Canister 実行環境を停止する。

このチュートリアルの所要時間は約 20分です。

## 新しいプロジェクトを作成する

このチュートリアルのための新しいプロジェクトを作成します：

1.  ローカルコンピューターでターミナルシェルを開きます（まだ開いていない場合）。

2.  Internet Computer のブロックチェーン・プロジェクトに使用しているフォルダがある場合は、そこに移動します。

3.  以下のコマンドを実行し、新しいプロジェクトを作成します：

        dfx new my_counter

    このコマンドは、あなたのプロジェクトに新しい `my_counter` プロジェクトを作成します。

4.  次のコマンドを実行して、プロジェクトディレクトリに移動します：

        cd my_counter

## デフォルトの設定を修正する

新しいプロジェクトを作成すると、デフォルトの `dfx.json` 設定ファイルがプロジェクトディレクトリに追加されることはすでにおわかりいただけたと思います。このチュートリアルでは、デフォルトの設定を変更して、プロジェクトのメインプログラムに別の名前を使用するようにします。

設定ファイル `dfx.json` を修正するには、以下の手順に従います：

1.  テキストエディタで `dfx.json` 設定ファイルを開き、デフォルトの `main` 設定を `main.mo` から `increment_counter.mo` に変更します。

    例：

        "main": "src/my_counter/increment_counter.mo",

    このチュートリアルでは、ソースファイルの名前を `main.mo` から `increment_counter.mo` に変更してどのように設定ファイル `dfx.json` の設定がコンパイルするソースファイルを決定するかを簡単に説明しています。

    より複雑な Dapp では、複数のソースファイルがあり、その依存関係を `dfx.json` 設定ファイルの設定で管理する必要があるかもしれません。このようなシナリオでは、複数の Canister とプログラムが `dfx.json` ファイルに定義されているため、複数のファイルに `main.mo` という名前を付けると混乱する可能性があります。

    それ以外の初期設定はそのままで問題ありません。

2.  変更を保存し、`dfx.json` ファイルを閉じて続行します。

3.  ソースコードディレクトリ `src` にあるメインプログラムのファイル名を、設定ファイル `dfx.json` で指定された名前に合わせます。以下のコマンドを実行します：

        mv src/my_counter/main.mo src/my_counter/increment_counter.mo

## デフォルトのプログラムを修正する

これまでのところ、プロジェクトのメインプログラムの名前を変更しただけです。次のステップは `src/my_counter/increment_counter.mo` ファイルにあるコードを修正して、 `Counter` という名前の Actor を定義し、 `increment`、`get`、`set` という関数を実装することです。

デフォルトのテンプレートのソースコードを変更するには、以下のようにします：

1.  必要であれば、プロジェクトディレクトリに留まっていることを確認します。

2.  `src/my_counter/increment_counter.mo` ファイルをテキストエディタで開き、既存の内容を削除してください。

3.  [このコード](../_attachments/counter.mo) をコピーして `increment_counter.mo` ファイルに貼り付けてください。

    このサンプルプログラムを詳しく見てみましょう：

    -   この例では、変数 `currentValue` の宣言に `stable` キーワードが含まれており、ステート（set、increment、retrieved が可能な値）が永続することを示していることがわかります。

        このキーワードは、プログラムがアップグレードされたときに、変数の値が変更されないようにするためのものです。

    -   変数 `currentValue` の宣言では、その型が自然数（`Nat`）であることも指定されています。

    -   このプログラムには、2つの更新メソッド（`increment` と `set` 関数）とひとつのクエリメソッド（`get` 関数）が含まれています。

    ステーブル変数とフレキシブル変数については、[*Motoko Programming Language Guide*](../cdks/motoko-dfinity/about-this-guide.md) の [Stable variables and upgrade methods](../cdks/motoko-dfinity/upgrades.md) を参照してください。

    クエリとアップデートの違いについては、[Canisters include both program and state](../../../concepts/canisters-code.md#canister-state) にある [Query and update methods](../../../concepts/canisters-code.md#query-update) を参照してください。

4.  変更を保存して、ファイルを閉じて続行します。

## ローカル canister 実行環境を起動する

`my_counter` プロジェクトをビルドする前に、Internet Computer ブロックチェーンをシミュレートするローカル Canister 実行環境か、Internet Computer ブロックチェーンメインネットに接続する必要があります。

ローカル Canister 実行環境を起動するには `dfx.json` ファイルが必要なので、プロジェクトのルートディレクトリにいることを確認する必要があります。このチュートリアルでは、ターミナルシェルをふたつ用意して、ひとつのターミナルでネットワーク操作を開始・確認し、もうひとつのターミナルでプロジェクトを管理できるようにします。

ローカル Canister の実行環境を起動するには、次のようにします：

1.  ローカルコンピューターで新しいターミナルウィンドウまたはタブを開きます。

2.  必要であれば、プロジェクトのルートディレクトリに移動します。

    -   これで、**ふたつのターミナル**が開いた状態になっているはずです。

    -   **プロジェクトディレクトリ**を、**現在の作業ディレクトリ**とする必要があります。

3.  以下のコマンドを実行して、お使いのマシンのローカル Canister 実行環境を起動します：

        dfx start

    ローカルネットワークを起動すると、ターミナルにネットワーク操作に関するメッセージが表示されます。

4.  ネットワーク操作を表示しているターミナルは開いたままにして、新しいプロジェクトを作成した元のターミナルをフォーカスしてください。

## Dapp の登録、ビルド、デプロイ

ローカルの Canister 実行環境に接続した後、ローカルで Dapp の登録、ビルド、デプロイができます。

ローカルに Dapp をデプロイするには：

1.  必要に応じて、まだプロジェクトのルートディレクトリにいることを確認します。

2.  以下のコマンドを実行して、Dapp を登録、ビルド、デプロイしてください：

        dfx deploy

    `dfx deploy` コマンドの出力は、実行した操作に関する情報を表示します。


## デプロイされた Canister メソッドを呼び出す

Canister のデプロイに成功したら、エンドユーザーが Canister が提供するメソッドを呼び出す様子をシミュレートしてみましょう。このチュートリアルでは、カウンターの値を問い合わせる `get` メソッド、呼び出されるたびにカウンターを増加させる `increment` メソッド、そしてカウンターを指定した任意の値に更新するための引数を渡す `set` メソッドを呼び出します。

デプロイされた Canister でメソッドを呼び出すテストを行うには、以下のようにします：

1.  以下のコマンドを実行して `get` 関数を呼び出し、デプロイされた Canister の `currentValue` 変数の現在値を読み取ります：

        dfx canister call my_counter get

    このコマンドは、変数 `currentValue` の現在値を 0 として返します：

        (0 : nat)

2.  以下のコマンドを実行して `increment` 関数を呼び出し、デプロイされた Canister の `currentValue` 変数の値をひとつ増やします：

        dfx canister call my_counter increment

    このコマンドは変数の値を増加させ、そのステートを変化させるが、結果は返しません。

3.  以下のコマンドを再実行して、デプロイされた canister の `currentValue` 変数の現在値を取得します：

        dfx canister call my_counter get

    このコマンドは、更新された `currentValue` 変数の値、1 を返します：

        (1 : nat)

4.  他のコマンドを実行して、他のメソッドを呼び出したり、異なる値を使用したりテストをしてください。

    例えば、次のようなコマンドで、カウンターの値を設定したり、戻したりしてみてください：

        dfx canister call my_counter set '(987)'
        dfx canister call my_counter get

    これにより、`currentValue`の更新された値は987となります。追加コマンドの実行：

        dfx canister call my_counter increment
        dfx canister call my_counter get

    インクリメントされた `currentValue` の 988 を返します。

5.  Candid UI を使用してコードをテストしてください。

    コードをテストするには、[こちら](candid-ui.md)に従ってください。

## ローカル Canister の実行環境を停止する

Dapp のテストが終了したら、ローカル Canister 実行環境を停止して、バックグラウンドで実行し続けないようにします。

ローカル Canister の実行環境を停止するには、以下の手順に従います：

1.  操作が表示されているターミナルで、Control-C キーを押して処理を中断します。

2.  以下のコマンドを実行して、ローカル Canister の実行環境を停止します：

        dfx stop

<!--
# Increment a natural number

In this tutorial, you are going to write a program that creates a single actor and provides a few basic functions to increment a counter and illustrate persistence of a value.

For this tutorial, the actor is named `Counter`. The program uses the `currentValue` variable to contain a natural number that represents the current value of the counter. This program supports the following function calls:

-   The `increment` function call updates the current value, incrementing it by 1 (no return value).

-   The `get` function call queries and returns the current value of the counter.

-   The `set` function call updates the current value to an arbitrary numeric value you specify as an argument.

This tutorial provides a simple example of how you can increment a counter by calling functions on a deployed canister. By calling the functions to increment and query the counter value multiple times, you can verify that the variable state—that is, the value of the variable between calls—persists.

## Before you begin

Before starting the tutorial, verify the following:

-   You have downloaded and installed the SDK package as described in [Download and install](../../quickstart/local-quickstart#download-and-install).

-   You have stopped any local canister execution environments running on the computer.

This tutorial takes approximately 20 minutes to complete.

## Create a new project

To create a new project directory for this tutorial:

1.  Open a terminal shell on your local computer, if you don’t already have one open.

2.  Change to the folder you are using for your Internet Computer projects, if you are using one.

3.  Create a new project by running the following command:

        dfx new my_counter

    The command creates a new `my_counter` project for your project.

4.  Change to your project directory by running the following command:

        cd my_counter

## Modify the default configuration

You have already seen that creating a new project adds a default `dfx.json` configuration file to your project directory. In this tutorial, you will modify the default settings to use a different name for the main program in your project.

To modify the `dfx.json` configuration file:

1.  Open the `dfx.json` configuration file in a text editor and change the default `main` setting from `main.mo` to `increment_counter.mo`.

    For example:

        "main": "src/my_counter/increment_counter.mo",

    For this tutorial, changing the name of the source file from `main.mo` to `increment_counter.mo` simply illustrates how the setting in the `dfx.json` configuration file determines the source file to be compiled.

    In a more complex dapp, you might have multiple source files with dependencies that you need to manage using settings in the `dfx.json` configuration file. In a scenario like that—with multiple canisters and programs defined in your `dfx.json` file—having multiple files all named `main.mo` might be confusing.

    You can leave the rest of the default settings as they are.

2.  Save your change and close the `dfx.json` file to continue.

3.  Change the name of the main program file in the source code directory `src` to match the name specified in the `dfx.json` configuration file by running the following command

        mv src/my_counter/main.mo src/my_counter/increment_counter.mo

## Modify the default program

So far, you have only changed the name of the main program for your project. The next step is to modify the code in the `src/my_counter/increment_counter.mo` file to define an actor named `Counter` and implement the `increment`, `get`, and `set` functions.

To modify the default template source code:

1.  Check that you are still in your project directory, if needed.

2.  Open the `src/my_counter/increment_counter.mo` file in a text editor and delete the existing content.

3.  Copy and paste [this code](../_attachments/counter.mo) into the `increment_counter.mo` file.

    Let's take a closer look at this sample program:

    -   You can see that the `currentValue` variable declaration in this example includes the `stable` keyword to indicate the state—the value that can be set, incremented, and retrieved—persists.

        This keyword ensures that the value for the variable is unchanged when the program is upgraded.

    -   The declaration for the `currentValue` variable also specifies that its type is a natural number (`Nat`).

    -   The program includes two public update methods—the `increment` and `set` functions—and one a query method-the `get` function.

    For more information about stable and flexible variables, see [Stable variables and upgrade methods](../cdks/motoko-dfinity/upgrades.md) in the [*Motoko Programming Language Guide*](../cdks/motoko-dfinity/about-this-guide.md).

    For more information about the differences between a query and an update, see [Query and update methods](../../../concepts/canisters-code.md#query-update) in [Canisters include both program and state](../../../concepts/canisters-code.md#canister-state).

4.  Save your changes and close the file to continue.

## Start the local canister execution environment

Before you can build the `my_counter` project, you need to either connect to a local canister execution environment simulating the Internet Computer blockchain or to the Internet Computer blockchain mainnet.

Starting the local canister execution environment requires a `dfx.json` file, so you should be sure you are in your project’s root directory. For this tutorial, you should have two separate terminal shells, so that you can start and see network operations in one terminal and manage your project in another.

To start the local canister execution environment:

1.  Open a new terminal window or tab on your local computer.

2.  Navigate to the root directory for your project, if necessary.

    -   You should now have **two terminals** open.

    -   You should have the **project directory** as your **current working directory**.

3.  Start the local canister execution environment on your computer by running the following command:

        dfx start

    After you start the local canister execution environment, the terminal displays messages about network operations.

4.  Leave the terminal that displays network operations open and switch your focus to your original terminal where you created your new project.

## Register, build, and deploy the dapp

After you connect to the local canister execution environment running in your development environment, you can register, build, and deploy your dapp locally.

To deploy the dapp locally:

1.  Check that you are still in the root directory for your project, if needed.

2.  Register, build, and deploy your dapp by running the following command:

        dfx deploy

    The `dfx deploy` command output displays information about the operations it performs.

## Invoke methods on the deployed canister

After successfully deploying the canister, you can simulate an end-user invoking the methods provided by the canister. For this tutorial, you invoke the `get` method to query the value of a counter, the `increment` method that increments the counter each time it is called, and the `set` method to pass an argument to update the counter to an arbitrary value you specify.

To test invoking methods on the deployed canister:

1.  Run the following command to invoke the `get` function, which reads the current value of the `currentValue` variable on the deployed canister:

        dfx canister call my_counter get

    The command returns the current value of the `currentValue` variable as zero:

        (0 : nat)

2.  Run the following command to invoke the `increment` function to increment the value of the `currentValue` variable on the deployed canister by one:

        dfx canister call my_counter increment

    This command increments the value of the variable—changing its state—but does not return the result.

3.  Rerun the following command to get the current value of the `currentValue` variable on the deployed canister:

        dfx canister call my_counter get

    The command returns the updated value of the `currentValue` variable as one:

        (1 : nat)

4.  Run additional commands to experiment with invoking other methods and using different values.

    For example, try commands similar to the following to set and return the counter value:

        dfx canister call my_counter set '(987)'
        dfx canister call my_counter get

    This returns the updated value of the `currentValue` to be 987. Running the additional commands

        dfx canister call my_counter increment
        dfx canister call my_counter get

    returns the incremented `currentValue` of 988.

5.  Test your code using the candid ui.

    To test your code, follow the instructions [here](candid-ui.md).

## Stop the local canister execution environment

After you finish experimenting with your dapp, you can stop the local canister execution environment so that it doesn’t continue running in the background.

To stop the local canister execution environment:

1.  In the terminal that displays network operations, press Control-C to interrupt the local canister execution environment.

2.  Stop the local canister execution environment by running the following command:

        dfx stop

-->
