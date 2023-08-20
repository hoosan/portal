# 複数の Actor を使用する

このチュートリアルでは、複数の Actor を持つプロジェクトを作成します。現在、Motoko ファイルで定義できる Actor は1つだけで、1つの Actor は常に1つの Canister にコンパイルされます。しかし、複数の Actor を持つ**プロジェクト**を作成し、同じ`dfx.json` 設定ファイルから複数の Canister をビルドすることが可能です。

このチュートリアルでは、同じプロジェクトで3つの Actor に対して別々のプログラムファイルを作成することになります。このプロジェクトでは、以下の無関係な Actor を定義しています：

-   `assistant` Actor は、ToDo リストにタスクを追加、表示する機能を提供します。

    簡略化するために、このチュートリアルのコードサンプルには、ToDo 項目を追加する関数と、追加された ToDo 項目の現在のリストを表示する関数だけを含んでいます。この Canister のより完全なバージョン（項目を完了にしたり、リストから削除したりする追加機能）は [examples](https://github.com/dfinity/examples/) リポジトリの [Simple to-do checklist](https://github.com/dfinity/examples/tree/master/motoko/simple-to-do) として公開されています。

-   `rock_paper_scissors` Actor は、ハードコードされたジャンケン大会で勝者を決定するための関数を提供します。

    このコードサンプルは、ハードコードされたプレーヤーと選択肢を持つ Motoko プログラムにおける `switch` と `case` の基本的な使い方を説明するものです。

-   `daemon` Actor は、デーモンを起動したり停止したりするためのモック関数を提供します。

    このコードサンプルは、デモンストレーションのために、単に変数を代入し、メッセージを表示するものです。

## 始める前に

チュートリアルを始める前に、以下を確認してください：

-   [ダウンロードとインストール](../../quickstart/local-quickstart#download-and-install)で説明されているように、SDK パッケージをダウンロードしインストールする。

-   コンピューター上で動作しているローカル Canister の実行環境を停止しています。

このチュートリアルの所要時間は約 20分です。

## プロジェクトを作成する

このチュートリアルのための新しいプロジェクトを作成するには：

1.  ローカルコンピューターでターミナルシェルを開いてください（まだ開いていない場合）。

2.  Internet Computer のプロジェクトに使用しているフォルダがあれば、そこに移動します。

3.  以下のコマンドを実行して、新しいプロジェクトを作成します：

        dfx new multiple_actors

4.  以下のコマンドを実行して、プロジェクトディレクトリに移動してください：

        cd multiple_actors

## デフォルトコンフィグレーションを修正する

新しいプロジェクトを作成すると、デフォルトの `dfx.json` 設定ファイルがプロジェクトディレクトリに追加されることはすでにお分かりいただけたと思います。このチュートリアルでは、このファイルにセクションを追加して、ビルドしたい Actor を定義する各 Canister の場所を指定する必要があります。

デフォルトの設定ファイル `dfx.json` を変更するには：

1.  テキストエディタで `dfx.json` 設定ファイルを開き、デフォルトの `multiple_actors` Canister 名とソースディレクトリを `assistant` に変更します。

    例えば、`canisters` キーの下には以下のようになります：

            "assistant": {
              "main": "src/assistant/main.mo",
              "type": "motoko"
            },

    設定ファイルのこの `canisters` セクションに設定を追加するので、`assistant` のメインソースコードファイルの場所と Canister タイプを囲む中括弧の後に**コンマ**も追加する必要があります。

2.  ファイルから `multiple_actors_assets` セクションを削除します。

3.  Canister 名、ソースコードの場所、Canister タイプを `rock_paper_scissors` に、同じく Canister 名、ソースコードの場所、 Canister タイプを `daemon` プログラムファイルに追加し、 `assistant` Canister 定義の下に追加します。

    変更後、`dfx.json` ファイルの `canisters` セクションは、[この](../_attachments/multiple-actors-dfx.json)ような形になるはずです。

    その他の部分はそのままで大丈夫です。

4.  変更を保存し、`dfx.json` ファイルを閉じて続行します。

5.  以下のコマンドを実行して、デフォルトのソースファイルディレクトリの名前を `dfx.json` 設定ファイルで指定された名前と一致するように変更します：

        cp -r src/multiple_actors/ src/assistant/

6.  以下のコマンドを実行して `assistant` ソースファイルのディレクトリをコピーし、`rock_paper_scissors` Actor のメイン Canister ファイルを作成します：

        cp -r src/assistant/ src/rock_paper_scissors/

7.  以下のコマンドを実行して、`assistant` のソースファイルのディレクトリをコピーし、`daemon` Actor のメイン Canister ファイルを作成します：

        cp -r src/assistant/ src/daemon/

## デフォルト Canister を修正する

これで `src` ディレクトリに3つのディレクトリができ、それぞれに `main.mo` というテンプレートファイルができました。このチュートリアルでは、各テンプレート `main.mo` ファイルの内容を別の Actor で置き換えます。

デフォルトのソースコードを変更するには：

1.  テキストエディタで `src/assistant/main.mo` ファイルを開き、既存の内容を削除してください。

2.  [このコード](../_attachments/multiple-actors-assistant-main.mo)をコピーして、ファイルに貼り付けてください。

3.  変更を保存して、`main.mo` ファイルを閉じて続行します。

4.  テキストエディタで `src/rock_paper_scissors/main.mo` ファイルを開き、既存の内容を削除してください。

5.  [このコード](../_attachments/multiple-actors-rock-main.mo)をコピーして、ファイルに貼り付けてください。

6.  変更を保存して、`main.mo` ファイルを閉じて続行します。

7.  テキストエディタで `src/daemon/main.mo` ファイルを開き、既存の内容を削除してください。

8.  [このコード](../_attachments/multiple-actors-daemon-main.mo)をコピーして、ファイルに貼り付けてください。

9.  変更を保存して、`main.mo` ファイルを閉じて続行します。

## ローカル Canister 実行環境を起動する

ローカルに `multiple_actors` プロジェクトをインストールする前に、ローカル Canister 実行環境を起動する必要があります。もし、Internet Computer のブロックチェーンメインネットにデプロイするつもりであれば、このセクションはスキップします。

ローカル Canister の実行環境を起動するには：

1.  ローカルコンピューターで新しいターミナルウィンドウまたはタブを開きます。

2.  必要であれば、プロジェクトのルートディレクトリに移動します。

3.  次のコマンドを実行して、ローカルの Canister 実行環境を起動します。

        dfx start

4.   Canister の実行操作を表示しているターミナルは開いたままにして、フォーカスを新しいプロジェクトを作成した元のターミナルに切り替えます。

## マルチ Canister 型 Dapp をデプロイする

マルチ Canister Dapp をデプロイするには、ローカル Canister 実行環境または Internet Computer ブロックチェーンメインネットに Canister を登録、ビルド、インストールする必要があります。

ローカルに Dapp をデプロイするには：

1.  必要に応じて、まだプロジェクトのルートディレクトリにいることを確認します。

2.  以下のコマンドを実行して、アプリケーションを登録、ビルド、デプロイします：

        dfx deploy

Internet Computer ブロックチェーン・メインネットに Dapp をデプロイするには：

3.  必要に応じて、まだプロジェクトのルートディレクトリにいることを確認します。

4.  `dfx deploy` コマンドに `--network` オプションと `dfx.json` ファイルで設定されたネットワークエイリアスを指定して実行します。例えば、ネットワークエイリアス `ic` で指定された URL で Internet Computer メインネットに接続する場合、以下のようなコマンドを実行します：

        dfx deploy --network ic

`dfx deploy` コマンドの出力には、実行された操作に関する情報が表示されます。例えば、このコマンドは `dfx.json` 設定ファイルで定義された3つの Canister の識別子を表示します。

    Deploying all canisters.
    Creating canisters...
    Creating canister "assistant"...
    "assistant" canister created with canister id: "75hes-oqbaa-aaaaa-aaaaa-aaaaa-aaaaa-aaaaa-q"
    Creating canister "daemon"...
    "daemon" canister created with canister id: "cxeji-wacaa-aaaaa-aaaaa-aaaaa-aaaaa-aaaaa-q"
    Creating canister "rock_paper_scissors"...
    "rock_paper_scissors" canister created with canister id: "7kncf-oidaa-aaaaa-aaaaa-aaaaa-aaaaa-aaaaa-q"

##  Canister を呼び出してデプロイを確認する

これで、3つの **Canister** スマートコントラクトが、ローカル Canister 実行環境または Internet Computer ブロックチェーンメインネット上にデプロイされ、`dfx canister call` コマンドを使用してそれらをテストすることができるようになりました。

ローカルにデプロイした Canister をテストするには：

1.  `dfx canister call` コマンドを使用して、`addTodo` 関数を使って Canister  `assistant` を呼び出し、以下のコマンドを実行して追加したいタスクを渡します：

        dfx canister call assistant addTodo '("Schedule monthly demos")'

2.  以下のコマンドを実行して、`showTodos` 関数を使用して ToDo リストの項目が返されることを確認します：

        dfx canister call assistant showTodos

    このコマンドは、次のような出力を返します：

        ("
        ___TO-DOs___
        (1) Schedule monthly demos")

3.  `dfx canister call` コマンドを使用して、以下のコマンドを実行し、`contest` 機能を使用して Canister `rock_paper_scissors` を呼び出します：

        dfx canister call rock_paper_scissors contest

    コマンドは、以下のようなハードコードされたコンテストの結果を返します：

        ("Bob won")

4.  `dfx canister call` コマンドを使用して、以下のコマンドを実行し、 `launch` 機能を使用して、`daemon` Canister を呼び出します。

        dfx canister call daemon launch

5.  モック `launch` 関数が "The daemon process is running" というメッセージを返すことを確認します：

        (""The daemon process is running"")

Internet Computer ブロックチェーン・メインネット上で動作する Canister をテストするには、上記と同じコマンドを使用します。`—network` オプションと `dfx.json` ファイルで設定したネットワークエイリアスを指定します。

## ローカルネットワークを停止する

Dapp のテストが終わったら、ローカル Canister の実行環境を停止して、バックグラウンドで実行し続けないようにします。

ローカル Canister の実行環境を停止するには、以下のようにします：

1.  ネットワーク操作が表示されているターミナルで、Control-C キーを押して、ローカルネットワークの処理を中断します。

2.  以下のコマンドを実行して、ローカル Canister の実行環境を停止します：

        dfx stop

<!--
# Use multiple actors

In this tutorial, you are going to create a project with multiple actors. Currently, you can only define one actor in a Motoko file and a single actor is always compiled to a single canister. You can, however, create **projects** that have multiple actors and can build multiple canisters from the same `dfx.json` configuration file.

For this tutorial, you are going to create separate program files for three actors in the same project. This project defines the following unrelated actors:

-   The `assistant` actor provides functions to add and show tasks in a to-do list.

    For simplicity, the code sample for this tutorial only includes the functions to add to-do items and to show the current list of to-do items that have been added. A more complete version of this canister—with additional functions for marking items as complete and removing items from the list—is available in the [examples](https://github.com/dfinity/examples/) repository as [Simple to-do checklist](https://github.com/dfinity/examples/tree/master/motoko/simple-to-do).

-   The `rock_paper_scissors` actor provides a function for determining a winner in a hard-coded rock-paper-scissors contest.

    This code sample illustrates the basic use of `switch` and `case` in a Motoko program with hard-coded players and choices.

-   The `daemon` actor provides mock functions for starting and stopping a daemon.

    This code sample simply assigns a variable and prints messages for demonstration purposes.

## Before you begin

Before starting the tutorial, verify the following:

-   You have downloaded and installed the SDK package as described in [Download and install](../../quickstart/local-quickstart#download-and-install).

-   You have stopped any local canister execution environment running on your computer.

This tutorial takes approximately 20 minutes to complete.

## Create a new project

To create a new project for this tutorial:

1.  Open a terminal shell on your local computer, if you don’t already have one open.

2.  Change to the folder you are using for your Internet Computer projects, if you are using one.

3.  Create a new project by running the following command:

        dfx new multiple_actors

4.  Change to your project directory by running the following command:

        cd multiple_actors

## Modify the default configuration

You have already seen that creating a new project adds a default `dfx.json` configuration file to your project directory. For this tutorial, you need to add sections to this file to specify the location of each canister that defines an actor you want to build.

To modify the default `dfx.json` configuration file:

1.  Open the `dfx.json` configuration file in a text editor, then change the default `multiple_actors` canister name and source directory to `assistant`.

    For example, under the `canisters` key:

            "assistant": {
              "main": "src/assistant/main.mo",
              "type": "motoko"
            },

    Because you are going to add settings to this `canisters` section of the configuration file, you must also add a **comma** after the curly brace that encloses the location of the `assistant` main source code file and the canister type.

2.  Remove the `multiple_actors_assets` section from the file.

3.  Add a new canister name, source code location, and canister type for the `rock_paper_scissors` canister and a new canister name, source code location, and canister type for the `daemon` program files below the `assistant` canister definition.

    After making the changes, the `canisters` section of the `dfx.json` file should look similar to [this](../_attachments/multiple-actors-dfx.json).

    You can leave the other sections as-is.

4.  Save your changes and close the `dfx.json` file to continue.

5.  Change the name of the default source file directory to match the name specified in the `dfx.json` configuration file by running the following command:

        cp -r src/multiple_actors/ src/assistant/

6.  Copy the `assistant` source file directory to create the main canister file for the `rock_paper_scissors` actor by running the following command:

        cp -r src/assistant/ src/rock_paper_scissors/

7.  Copy the `assistant` source file directory to create the main canister file for the `daemon` actor by running the following command:

        cp -r src/assistant/ src/daemon/

## Modify the default canisters

You now have three separate directories in the `src` directory, each with a template `main.mo` file. For this tutorial, you will replace the content in each template `main.mo` file with a different actor.

To modify the default source code:

1.  Open the `src/assistant/main.mo` file in a text editor and delete the existing content.

2.  Copy and paste [this code](../_attachments/multiple-actors-assistant-main.mo) into the file.

3.  Save your changes and close the `main.mo` file to continue.

4.  Open the `src/rock_paper_scissors/main.mo` file in a text editor and delete the existing content.

5.  Copy and paste [this code](../_attachments/multiple-actors-rock-main.mo) into the file.

6.  Save your changes and close the `main.mo` file to continue.

7.  Open the `src/daemon/main.mo` file in a text editor and delete the existing content.

8.  Copy and paste [this code](../_attachments/multiple-actors-daemon-main.mo) into the file.

9.  Save your changes and close the `main.mo` file to continue.

## Start the local canister execution environment

Before you can install the `multiple_actors` project locally, you need to start your local canister execution environment. If you intend to deploy it to the Internet Computer blockchain mainnet, then you can skip this section.

To start the local canister execution environment:

1.  Open a new terminal window or tab on your local computer.

2.  Navigate to the root directory for your project, if necessary.

3.  Start the local canister execution environment by running the following command:

        dfx start

4.  Leave the terminal that displays canister execution operations open and switch your focus to your original terminal where you created your new project.

## Deploy your multi-canister dapp

To deploy the multi-canister dapp you need to register, build, and install the canisters either to the local canister execution environment or to the Internet Computer blockchain mainnet.

To deploy the dapp locally:

1.  Check that you are still in the root directory for your project, if needed.

2.  Register, build, and deploy the application by running the following command:

        dfx deploy

To deploy the dapp to the Internet Computer blockchain mainnet:

1.  Check that you are still in the root directory for your project, if needed.

2.  Run `dfx deploy` command specifying the `--network` option and the network alias configured in the `dfx.json` file. For example, if you are connecting to the Internet Computer mainnet using to the URL specified by the network alias `ic` you would run a command similar the following:

        dfx deploy --network ic

The `dfx deploy` command output displays information about the operations it performs. For example, the command displays the specific canister identifiers for the three canisters defined in the `dfx.json` configuration file.

    Deploying all canisters.
    Creating canisters...
    Creating canister "assistant"...
    "assistant" canister created with canister id: "75hes-oqbaa-aaaaa-aaaaa-aaaaa-aaaaa-aaaaa-q"
    Creating canister "daemon"...
    "daemon" canister created with canister id: "cxeji-wacaa-aaaaa-aaaaa-aaaaa-aaaaa-aaaaa-q"
    Creating canister "rock_paper_scissors"...
    "rock_paper_scissors" canister created with canister id: "7kncf-oidaa-aaaaa-aaaaa-aaaaa-aaaaa-aaaaa-q"

## Verify deployment by calling the canisters

You now have three **canisters** smart contracts deployed, either on your local canister execution environment or on the Internet Computer blockchain mainnet, and can test them by using `dfx canister call` commands.

To test the canisters you have deployed locally:

1.  Use the `dfx canister call` command to call the canister `assistant` using the `addTodo` function and pass it the task you want to add by running the following command:

        dfx canister call assistant addTodo '("Schedule monthly demos")'

2.  Verify that the command returns the to-do list item using the `showTodos` function by running the following command:

        dfx canister call assistant showTodos

    The command returns output similar to the following:

        ("
        ___TO-DOs___
        (1) Schedule monthly demos")

3.  Use the `dfx canister call` command to call the canister `rock_paper_scissors` using the `contest` function by running the following command:

        dfx canister call rock_paper_scissors contest

    The command returns the result of the hard-coded contest similar to the following:

        ("Bob won")

4.  Use the `dfx canister call` command to call the canister `daemon` using the `launch` function by running the following command:

        dfx canister call daemon launch

5.  Verify the mock `launch` function returns "The daemon process is running" message":

        (""The daemon process is running"")

To test your canisters running on the Internet Computer blockchain mainnet use the same command as above, specifying the `--network` option and the network alias configured in the `dfx.json` file.

## Stop the local network

After you finish experimenting with your dapp, you can stop the local canister execution environment so that it doesn’t continue running in the background.

To stop the local canister execution environment:

1.  In the terminal that displays network operations, press Control-C to interrupt the local network process.

2.  Stop the local canister execution environment by running the following command:

        dfx stop

-->