# 電卓の機能で整数を使う

このチュートリアルでは、基本的な算術演算を行うためのいくつかのパブリックエントリポイント関数を持つ単一の Actor を作成して、簡単な電卓プログラムを書いていきます。

このチュートリアルでは、Actor は `Calc` と名付けます。このプログラムは `cell` 変数を使用して、電卓操作の現在の結果を表す整数を格納します。

このプログラムは以下の関数呼び出しをサポートしています：

-   `add` 関数は入力を受け取り、加算を実行する。

-   `sub` 関数は入力を受け取り、減算を実行する。

-   `mul` 関数は入力を受け取り、乗算を実行する。

-   `div` 関数は入力を受け取り、除算を実行する。

-   `clearall` 関数は、以前の操作の結果として保存された `cell` 値をクリアして、`cell` 値をゼロにリセットします。

また、`div`関数には、プログラムがゼロで割り切ろうとするのを防ぐためのコードも含まれています。

## 始める前に

チュートリアルを始める前に、以下を確認してください：

-   [ダウンロードとインストール](../../quickstart/local-quickstart#download-and-install)の説明に従って、SDK パッケージをダウンロードし、インストールする。

-   ローカルコンピューターで動作しているすべての Canister 実行環境を停止する。

このチュートリアルの所要時間は約 20分です。

## 新しいプロジェクトを作成する

このチュートリアルのための新しいプロジェクトを作成します：

1.  ローカルコンピューターでターミナルシェルを開きます（まだ開いていない場合）。

2.  Internet Computer のブロックチェーン・プロジェクトに使用しているフォルダがある場合は移動します。

3.  以下のコマンドを実行し、新しいプロジェクトを作成します：

        dfx new calc

4.  次のコマンドを実行して、プロジェクトディレクトリに移動します：

        cd calc

## デフォルトの設定を修正する

このチュートリアルでは、デフォルトの `dfx.json` 設定ファイルを修正して、メインプログラムにもっと具体的な名前を使えるようにしましょう。

デフォルトの設定ファイル `dfx.json` を修正します：

1.  設定ファイル `dfx.json` をテキストエディタで開いてください。

2.  `Main` キーの設定を、デフォルトの `main.mo` プログラム名から `calc_main.mo` に変更します。

    例：

        "main": "src/calc/calc_main.mo",

    このチュートリアルでは、ソースファイルの名前を `main.mo` から `calc_main.mo` に変更してどのように設定ファイル `dfx.json` の設定がコンパイルするソースファイルを決定するかを簡単に説明しています。

    より複雑な Dapp では、単一の `main` プログラムファイルではなく、複数のソースファイルを使用することがあります。さらに複雑なアプリケーションでは、複数のソースファイルの間に特定の依存関係があり、それを `dfx.json` 設定ファイルの設定を使って管理する必要があるかもしれません。このようなシナリオでは、複数の Canister とプログラムが `dfx.json` ファイルに定義されているので、複数のファイルをすべて `main.mo` という名前で管理すると、ワークスペースをナビゲートしづらくなります。各プログラムの名前は重要ではありませんが、`dfx.json` ファイルで設定した名前がファイルシステム内のプログラムの名前と一致することが重要です。

3.  変更を保存して、ファイルを閉じて続行します。

## デフォルトのプログラムを修正する

このチュートリアルでは、デフォルトのプログラムを基本的な算術演算を行うプログラムに置き換える必要があります。

デフォルトのプログラムを置き換えるには：

1.  必要であれば、プロジェクトディレクトリに留まっていることを確認します。

2.  以下のコマンドを実行して、テンプレートの `main.mo` ファイルをコピーし、 `calc_main.mo` という名前のファイルを新規に作成します：

        cp src/calc/main.mo src/calc/calc_main.mo

3.  テキストエディタで `src/calc/calc_main.mo` ファイルを開き、既存の内容を削除します。

4.  [このコード](../_attachments/calc_main.mo) をコピーして `calc_main.mo` ファイルに貼り付けてください。

    このサンプルコードでは、整数（`Int`）のデータ型を使用しているため、正負の数を使用できることにお気づきでしょう。もし、この電卓の関数が正の数しか使えないように制限したい場合は、データ型を変更して、自然数（`Nat`）のデータしか使えないようにすればよいでしょう。

5.  変更を保存して、ファイルを閉じて続行します。

## ローカルキャニスター実行環境の起動する

`calc` プロジェクトをビルドする前に、開発環境でローカルに動作している Canister 実行環境に接続するか、アクセス可能なサブネットに接続する必要があります。

ローカルでネットワークを起動するには `dfx.json` ファイルが必要なので、プロジェクトのルートディレクトリにいることを確認する必要があります。このチュートリアルでは、ターミナルシェルをふたつ用意し、一方のターミナルでネットワーク操作を開始・確認し、もう一方のターミナルでプロジェクトを管理できるようにします。

ローカル Canister の実行環境を起動するには、次のようにします：

1.  ローカルコンピューターで新しいターミナルウィンドウまたはタブを開きます。

2.  必要であれば、プロジェクトのルートディレクトリに移動します。

    -   これで、**2つのターミナル**が開いた状態になっているはずです。

    -   **プロジェクトディレクトリ**を、**現在の作業ディレクトリ**とする必要があります。

3.  以下のコマンドを実行して、お使いのマシンのローカル Canister 実行環境を起動します：

        dfx start

    ローカルネットワークを起動すると、ターミナルにネットワーク操作に関するメッセージが表示されます。

4.  ネットワーク操作を表示しているターミナルは開いたままにして、新しいプロジェクトを作成した元のターミナルをフォーカスしてください。

## Dapp の登録、ビルド、デプロイ

ローカルの Canister 実行環境に接続した後、ローカルで Dapp の登録、ビルド、デプロイができます。

ローカルに Dapp をデプロイするには：

1.  必要に応じて、まだプロジェクトのルートディレクトリにいることを確認します。

2.  以下のコマンドを実行して、Dappを登録、ビルド、デプロイしてください：

        dfx deploy

    `dfx deploy` コマンドの出力は、実行した操作に関する情報を表示します。

## Canister で電卓の機能を確認する

これで、プログラムがローカルのキャニスター実行環境上に **Canister** としてデプロイされました。このプログラムは `dfx canister call` コマンドを使用してテストすることができます。

デプロイしたプログラムをテストするには、以下のようにします：

1.  `dfx canister call` コマンドを使用して、`calc` キャニスター `add` 関数を呼び出し、次のコマンドを実行して入力引数 `10` を渡します：

        dfx canister call calc add '(10)'

    シングルクォーテーションと括弧で囲まれた引数を渡すと、インターフェース記述言語（IDL）がその引数の型を解析するので、引数の型を手動で指定する必要はありません。

    コマンドが `add` 関数に対して期待される値を返すことを確認します。例えば、プログラムは次のような出力を表示します：

        (10)

2.  以下のコマンドを実行して、`mul` 関数を呼び出し、入力引数`3`を渡します：

        dfx canister call calc mul '(3)'

    コマンドが `mul` 関数に対して期待される値を返すことを確認します。例えば、このプログラムでは次のような出力が表示されます：

        (30)

3.  以下のコマンドを実行して、`sub` 関数を呼び出し、入力引数として `number` 型の `5` を渡します：

        dfx canister call calc sub '(5)'

    コマンドが `sub` 関数に対して期待される値を返すことを確認します。例えば、このプログラムでは次のような出力が表示されます：

        (25)

4.  次のコマンドを実行して、`div`関数を呼び出し、入力引数`5`を渡します：

        dfx canister call calc div '(5)'

    コマンドが `div` 関数に対して期待される値を返すことを確認します。例えば、プログラムは次のような出力を表示します：

        (opt 5)

    `div` 関数がオプションの結果を返すことに気がついたかもしれません。このプログラムは、 `div` 関数がゼロ除算エラーの場合に `null` を返せるようにするために、結果をオプションにしています。

    このプログラムのセル変数は整数なので、その関数を呼び出して負の入力値を指定することも可能です。例えば、次のようなコマンドを実行します：

        dfx canister call calc mul '(-4)'

    すると、以下のように返されます：

        (-20)

5.  `clearall` 関数を呼び出して、`cell` の値を 0 にリセットすることを確認します：

        dfx canister call calc clearall

    すると、このプログラムでは次のような出力が表示されます：

        (0)


## ローカル Canister の実行環境を停止する

Dapp のテストが終了したら、ローカル Canister 実行環境を停止して、バックグラウンドで実行し続けないようにします。

ローカル Canister の実行環境を停止するには、以下の手順に従います：

1.  操作が表示されているターミナルで、Control-C キーを押して処理を中断します。

2.  以下のコマンドを実行して、ローカル Canister の実行環境を停止します：

        dfx stop

<!--
# Use integers in calculator functions

In this tutorial, you are going to write a simple calculator program that creates a single actor with several public entry-point functions to perform basic arithmetic operations.

For this tutorial, the actor is named `Calc`. The program uses the `cell` variable to contain an integer number that represents the current result of a calculator operation.

This program supports the following function calls:

-   The `add` function call accepts input and performs addition.

-   The `sub` function call accepts input and performs subtraction.

-   The `mul` function call accepts input and performs multiplication.

-   The `div` function call accepts input and performs division.

-   The `clearall` function clears the `cell` value stored as the result of previous operations, resetting the `cell` value to zero.

The `div` function also includes code to prevent the program from attempting to divide by zero.

## Before you begin

Before starting the tutorial, verify the following:

-   You have downloaded and installed the SDK package as described in [Download and install](../../quickstart/local-quickstart#download-and-install).

-   You have stopped any canister execution environment running on the local computer.

This tutorial takes approximately 20 minutes to complete.

## Create a new project

To create a new project for this tutorial:

1.  Open a terminal shell on your local computer, if you don’t already have one open.

2.  Change to the folder you are using for your Internet Computer projects, if you are using one.

3.  Create a new project by running the following command:

        dfx new calc

4.  Change to your project directory by running the following command:

        cd calc

## Modify the default configuration

For this tutorial, let’s modify the default `dfx.json` configuration file to use a more specific name for its main program.

To modify the default `dfx.json` configuration file:

1.  Open the `dfx.json` configuration file in a text editor.

2.  Change the `main` key setting from the default `main.mo` program name to `calc_main.mo`.

    For example:

        "main": "src/calc/calc_main.mo",

    For this tutorial, changing the name of the source file from `main.mo` to `calc_main.mo` simply illustrates how the setting in the `dfx.json` configuration file determines the source file to be compiled.

    In a more complex dapp, you might have multiple source files instead of a single `main` program file. More complex applications might also have specific dependencies between multiple source files that you need to manage using settings in the `dfx.json` configuration file. In a scenario like that—with multiple canisters and programs defined in your `dfx.json` file—having multiple files all named `main.mo` might make navigating your workspace more difficult. The name you choose for each program isn’t significant, but it is important that the name you set in the `dfx.json` file matches the name of your program in the file system.

3.  Save your changes and close the file to continue.

## Modify the default program

For this tutorial, you need to replace the default program with a program that performs basic arithmetic operations.

To replace the default program:

1.  Check that you are still in your project directory, if needed.

2.  Copy the template `main.mo` file to create a new file named `calc_main.mo` by running the following command:

        cp src/calc/main.mo src/calc/calc_main.mo

3.  Open the `src/calc/calc_main.mo` file in a text editor and delete the existing content.

4.  Copy and paste [this code](../_attachments/calc_main.mo) into the `calc_main.mo` file.

    You might notice that this sample code uses integer (`Int`) data types, enabling you to use positive or negative numbers. If you wanted to restrict the functions in this calculator code to only use positive numbers, you could change the data type to only allow natural (`Nat`) data.

5.  Save your changes and close the file to continue.

## Start the local canister execution environment

Before you can build the `calc` project, you need to connect to the canister execution environment running locally in your development environment, or you need to connect toa subnet that you can access.

Starting the network locally requires a `dfx.json` file, so you should be sure you are in your project’s root directory. For this tutorial, you should have two separate terminal shells, so that you can start and see network operations in one terminal and manage your project in another.

To start the local canister execution environment:

1.  Open a new terminal window or tab on your local computer.

2.  Navigate to the root directory for your project, if necessary.

    -   You should now have **two terminals** open.

    -   You should have the **project directory** as your **current working directory**.

3.  Start the local canister execution environment on your machine by running the following command:

        dfx start

    After you start the local network, the terminal displays messages about network operations.

4.  Leave the terminal that displays network operations open and switch your focus to your original terminal where you created your new project.

## Register, build, and deploy the dapp

After you connect to the local canister execution environment, you can register, build, and deploy your dapp locally.

To deploy the dapp locally:

1.  Check that you are still in the root directory for your project, if needed.

2.  Register, build, and deploy your dapp by running the following command:

        dfx deploy

    The `dfx deploy` command output displays information about the operations it performs.

## Verify calculator functions on the canister

You now have a program deployed as a **canister** on your local canister execution environment. You can test the program by using `dfx canister call` commands.

To test the program you have deployed:

1.  Use the `dfx canister call` command to call the `calc` canister `add` function and pass it the input argument `10` by running the following command:

        dfx canister call calc add '(10)'

    When you pass an argument enclosed by the single quotation marks and parentheses,the interface description language (IDL) parses the argument type, so you don’t need to specify the argument type manually.

    Verify that the command returns the value expected for the `add` function. For example, the program displays output similar to the following:

        (10)

2.  Call the `mul` function and pass it the input argument `3` by running the following command:

        dfx canister call calc mul '(3)'

    Verify that the command returns the value expected for the `mul` function. For example, the program displays output similar to the following:

        (30)

3.  Call the `sub` function and pass it the input argument `5` of type `number` by running the following command:

        dfx canister call calc sub '(5)'

    Verify that the command returns the value expected for the `sub` function. For example, the program displays output similar to the following:

        (25)

4.  Call the `div` function and pass it the input argument `5` by running the following command:

        dfx canister call calc div '(5)'

    Verify that the command returns the value expected for the `div` function. For example, the program displays output similar to the following:

        (opt 5)

    You might notice that the `div` function returns an optional result. The program makes the result optional to enable the `div` function to return `null` in the case of a division-by-zero error.

    Because the cell variable in this program is an integer, you can also call its functions and specify negative input values. For example, you might run the following command:

        dfx canister call calc mul '(-4)'

    which returns:

        (-20)

5.  Call the `clearall` function and verify it resets the `cell` value to zero:

        dfx canister call calc clearall

    For example, the program displays output similar to the following:

        (0)


## Stop the local canister execution environment

After you finish experimenting with the dapp, you can stop the local canister execution environment so that it does not continue running in the background.

To stop the local canister execution environment:

1.  In the terminal that displays the operations, press Control-C to interrupt the process.

2.  Stop the local canister execution environment by running the following command:

        dfx stop

-->
