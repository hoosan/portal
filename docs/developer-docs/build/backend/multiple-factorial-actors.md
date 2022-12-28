# 複数の Actor を利用する

February 2020 (Alpha) :proglang: Motoko :IC: Internet Computer :company-id: DFINITY

このチュートリアルでは、複数の Actor を持つプロジェクトを作成します。現在、Motoko のファイルではひとつの Actor しか定義できず、ひとつの Actor は常にひとつの Canister にコンパイルされます。また、あるキャニスタの Actor で定義された関数を別の Canister で定義された Actor から呼び出したり、複数の Actor インスタンスをサポートする Actor クラスを Motoko プログラムで定義することはまだできません。しかし、複数の Actor を持つ **プロジェクト** を作成し、同じ `dfx.json` 設定ファイルから複数の Canister をビルドすることはできます。

このチュートリアルでは、同じプロジェクトで3つの Actor に対して別々のプログラムファイルを作成します。このプロジェクトでは、以下の無関係な Actor を定義しています：

-   `assistant`  Actor は、ToDo リストにタスクを追加したり、表示したりする機能を提供します。

    簡略化するために、このチュートリアルのコードサンプルには、ToDo 項目を追加する関数と、追加された ToDo 項目の現在のリストを表示する関数だけを含んでいます。この Canister のより完全なバージョン（項目を完了にしたり、リストから削除したりする追加機能）は [examples](https://github.com/dfinity/examples/) リポジトリの [Simple to-do checklist](https://github.com/dfinity/examples/tree/master/motoko/simple-to-do) として公開されています。

-   `factorial`  Actor は、指定された数の階乗を求める関数を提供します。

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

1.  テキストエディターで `dfx.json` 設定ファイルを開き、デフォルトの `multiple_actors` Canister 名とソースディレクトリを `assistant` に変更します。

    例：

        {
          "canisters": {
            "assistant": {
              "frontend": {
                "entrypoint": "src/multiple_actors/public/index.js"
              },
              "main": "src/assistant/main.mo"
            },

    設定ファイルのこの `canisters` セクションに設定を追加するので、`assistant` のメインソースコードファイルの場所と Canister タイプを囲む中括弧の後に**コンマ**も追加する必要があります。

2.  `assistant` のソースファイルの場所の下に、`factorial` プログラム用の新しい Canister 名とソースファイルの場所、`daemon` プログラムファイル用の新しい Canister 名とソースファイルの場所を追加してください。

    例：

            "factorial": {
              "main": "src/factorial/main.mo"
            },
            "daemon": {
              "main": "src/daemon/main.mo"
                }
          },

    その他の部分はそのままで大丈夫です。

3.  以下のコマンドを実行して、デフォルトのソースファイルディレクトリの名前を `dfx.json` 設定ファイルで指定された名前と一致するように変更します。

        cp -r src/multiple_actors/ src/assistant/

4.  以下のコマンドを実行して、`assistant` のソースファイルのディレクトリをコピーし、`factorial` Actor のメインプログラムファイルを作成します。

        cp -r src/assistant/ src/factorial/

5.  以下のコマンドを実行して、`assistant` のソースファイルのディレクトリをコピーし、`daemon` Actor のメインプログラムファイルを作成します。

        cp -r src/assistant/ src/daemon/

## デフォルトのテンプレートプログラムを変更する

これで `src` ディレクトリに3つのディレクトリができ、それぞれに `main.mo` というテンプレートファイルができました。このチュートリアルでは、各テンプレート `main.mo` ファイルの内容を別の Actor で置き換えます。

デフォルトのテンプレートのソースコードを変更するには：

1.  テキストエディタで `src/assistant/main.mo` ファイルを開き、既存のコンテンツを削除してください。

2.  以下のサンプルコードをコピーして、ファイルに貼り付けてください：

        include::example$multiple-actors/assistant/main.mo

3.  テキストエディタで `src/factorial/main.mo` ファイルを開き、既存のコンテンツを削除してください。

4.  以下のサンプルコードをコピーして、ファイルに貼り付けてください：

        include::example$multiple-actors/factorial/main.mo

5.  テキストエディタで `src/daemon/main.mo` ファイルを開き、既存のコンテンツを削除してください。

6.  以下のサンプルコードをコピーして、ファイルに貼り付けてください：

        include::example$multiple-actors/daemon/main.mo

## プロジェクトですべての Canister をビルドする

これで、ローカル・レプリカ・ネットワークにデプロイを実行可能な WebAssembly モジュールにコンパイルできるプログラムができました。

プロジェクト内の各 Actor の実行ファイルをビルドするには、次のようにします：

1.  必要であれば、プロジェクトのルートディレクトリ `~/ic-projects/multiple_actors` に変更します。

2.  以下のコマンドを実行して、各プログラムの WebAssembly 実行ファイルをビルドしてください：

        dfx build --all

    If the command is successful, it builds all of the canisters you have specified in the `dfx.json` file.

        Building canister assistant
        Building canister factorial
        Building canister daemon

## プロジェクトに Canister をデプロイする

これで、 Actor ーごとにコンパイルされた3つのプログラムができあがり、デプロイの準備が整いました。

 Canister をデプロイするには：

1.  以下のコマンドを実行して、ローカルコンピューターで IC ネットワークを起動します：

        dfx start

2.  新しいターミナルシェルを開き、プロジェクトのルートディレクトリ `~/ic-projects/multiple_actors` に移動します。

    例：

        cd ~/ic-projects/multiple_actors

3.  以下のコマンドを実行して、ローカルネットワーク上に Canister をデプロイします。

        dfx canister install --all

## 関数を呼び出してデプロイメントを検証する

これで、3つのプログラムがローカルのレプリカネットワーク上に ** Canister ** として配置され、`dfx canister call` コマンドを使用して各プログラムをテストすることができるようになりました。

ローカル・レプリカ・ネットワークにデプロイしたプログラムをテストするには：

1.  `dfx canister call` コマンドを使用して、`addTodo` 関数を使って Canister `assistant` を呼び出し、以下のコマンドを実行して追加したいタスクを渡します：

        dfx canister call assistant addTodo '("Schedule monthly demos")'

2.  以下のコマンドを実行して、`showTodos` 関数を使用して ToDo リストの項目が返されることを確認します：

        dfx canister call assistant showTodos

    このコマンドは、次のような出力を返します：

        ("
        ___TO-DOs___
        (1) Schedule monthly demos

3.  `dfx canister call` コマンドを使用して、以下のコマンドを実行し、`fac` 関数を使用して Canister の `factorial` を呼び出します：

        dfx canister call factorial fac '(8)'

    このコマンドは、以下のような結果を返します：

        (40320)

4.  `dfx canister call` コマンドを使用して、以下のコマンドを実行し、`launch` 機能を使用して、canister `daemon` を呼び出します：

        dfx canister call daemon launch

5.  モック `launch` 関数が "The daemon process is running" というメッセージを返すことを確認します：

        (""The daemon process is running"")

6.  以下のコマンドを実行して、ローカルコンピューターで動作している IC プロセスを停止します：
        dfx stop

<!--
# Using multiple actors

February 2020 (Alpha) :proglang: Motoko :IC: Internet Computer :company-id: DFINITY

In this tutorial, you are going to create a project with multiple actors. Currently, you can only define one actor in a Motoko file and a single actor is always compiled to a single canister. In addition, you cannot yet call functions defined in an actor in one canister from an actor defined in another canister or define an actor class to support multiple actor instances in your Motoko programs. You can, however, create **projects** that have multiple actors and can build multiple canisters from the same `dfx.json` configuration file.

For this tutorial, you are going to create separate program files for three actors in the same project. This project defines the following unrelated actors:

-   The `assistant` actor provides functions to add and show tasks in a to-do list.

    For simplicity, the code sample for this tutorial only includes the functions to add to-do items and to show the current list of to-do items that have been added. A more complete version of this program-with additional functions for marking items as complete and removing items from the list—is included in [Sample code, applications, and microservices](../sample-apps).

-   The `factorial` actor provides a function for determining the factorial for a specified number.

-   The `daemon` actor provides mock functions for starting and stopping a daemon.

    This code sample simply assigns a variable and prints messages for demonstration purposes.

## Before you begin

Before starting the tutorial, verify the following:

-   You have downloaded and installed the SDK as described in [the Quick Start](../../quickstart/hello10mins.md).

-   You have stopped any network replica processes running on the local computer.

This tutorial takes approximately 20 minutes to complete.

## Create a new project

To create a new project for this tutorial:

1.  Open a terminal shell on your local computer, if you don’t already have one open.

2.  Change to the folder you are using for your IC projects, if you are using one.

3.  Create a new project by running the following command:

        dfx new multiple_actors

4.  Change to your project directory by running the following command:

        cd multiple_actors

## Modify the default configuration

You have already seen that creating a new project adds a default `dfx.json` configuration file to your project directory. For this tutorial, you need to add sections to this file to specify the location of each program that defines an actor you want to build.

To modify the default `dfx.json` configuration file:

1.  Open the `dfx.json` configuration file in a text editor, then change the default `multiple_actors` canister name and source directory to `assistant`.

    For example:

        {
          "canisters": {
            "assistant": {
              "frontend": {
                "entrypoint": "src/multiple_actors/public/index.js"
              },
              "main": "src/assistant/main.mo"
            },

    Because you are going to add settings to this `canisters` section of the configuration file, you must also add a **comma** after the curly brace that encloses the location of the `assistant` main source code file.

2.  Add a new canister name and source file location for the `factorial` program and a new canister name and source file location for the `daemon` program files below the `assistant` source file location.

    For example:

            "factorial": {
              "main": "src/factorial/main.mo"
            },
            "daemon": {
              "main": "src/daemon/main.mo"
                }
          },

    You can leave the other sections as-is.

3.  Change the name of the default source file directory to match the name specified in the `dfx.json` configuration file by running the following command:

        cp -r src/multiple_actors/ src/assistant/

4.  Copy the `assistant` source file directory to create the main program file for the `factorial` actor by running the following command:

        cp -r src/assistant/ src/factorial/

5.  Copy the `assistant` source file directory to create the main program file for the `daemon` actor by running the following command:

        cp -r src/assistant/ src/daemon/

## Modify the default template programs

You now have three separate directories in the `src` directory, each with a template `main.mo` file. For this tutorial, you will replace the content in each template `main.mo` file with a different actor.

To modify the default template source code:

1.  Open the `src/assistant/main.mo` file in a text editor and delete the existing content.

2.  Copy and paste the following sample code into the file:

        include::example$multiple-actors/assistant/main.mo

3.  Open the `src/factorial/main.mo` file in a text editor and delete the existing content.

4.  Copy and paste the following sample code into the file:

        include::example$multiple-actors/factorial/main.mo

5.  Open the `src/daemon/main.mo` file in a text editor and delete the existing content.

6.  Copy and paste the following sample code into the file:

        include::example$multiple-actors/daemon/main.mo

## Build all of the canisters in the project

You now have a program that you can compile into an executable WebAssembly module that you can deploy on your local replica network.

To build the executable for each actor in the project:

1.  Change to the `~/ic-projects/multiple_actors` root directory for your project, if needed.

2.  Build the WebAssembly executable for each program by running the following command:

        dfx build --all

    If the command is successful, it builds all of the canisters you have specified in the `dfx.json` file.

        Building canister assistant
        Building canister factorial
        Building canister daemon

## Deploy the canisters in the project

You now have three separate compiled programs—one for each actor—ready for deployment.

To deploy the canisters:

1.  Start the IC network on your local computer by running the following command:

        dfx start

2.  Open a new terminal shell, then change the `~/ic-projects/multiple_actors` root directory for your project.

    For example:

        cd ~/ic-projects/multiple_actors

3.  Deploy your canisters on the local network by running the following command:

        dfx canister install --all

## Verify deployment by calling functions

You now have three programs deployed as a **canisters** on your local replica network and can test each program by using `dfx canister call` commands.

To test the programs you have deployed on the local replica network:

1.  Use the `dfx canister call` command to call the canister `assistant` using the `addTodo` function and pass it the task you want to add by running the following command:

        dfx canister call assistant addTodo '("Schedule monthly demos")'

2.  Verify that the command returns the to-do list item using the `showTodos` function by running the following command:

        dfx canister call assistant showTodos

    The command returns output similar to the following:

        ("
        ___TO-DOs___
        (1) Schedule monthly demos

3.  Use the `dfx canister call` command to call the canister `factorial` using the `fac` function by running the following command:

        dfx canister call factorial fac '(8)'

    The command returns the result of the function:

        (40320)

4.  Use the `dfx canister call` command to call the canister `daemon` using the `launch` function by running the following command:

        dfx canister call daemon launch

5.  Verify the mock `launch` function returns "The daemon process is running" message":

        (""The daemon process is running"")

6.  Stop the IC processes running on your local computer by running the following command:

        dfx stop

-->