# テキスト引数を渡す

このチュートリアルでは、デフォルトプログラムの簡単なバリエーションとして、1 つのテキスト引数を 1 つの Actor に渡し、コードをコンパイルして Canister スマートコントラクトを作成し、その引数を取得することを行います。 このドキュメントでは、Canister スマートコントラクトと Canister という用語は同義語とします。

このチュートリアルでは、Candid インターフェース記述言語（IDL）を使ってターミナルのコマンドラインで引数を渡す方法と、テキスト引数に複数の値を受け取れるようにプログラムを修正する方法を説明します。

## 始める前に

このチュートリアルを始める前に、以下のことを確認してください:

- [ダウンロードとインストール](../../quickstart/local-quickstart#download-and-install) に記載されている SDK パッケージをダウンロードしてインストールしていること

- ローカルコンピュータ上のすべての Canister 実行環境を止めていること

このチュートリアルの所要時間は 20 分程度です。

## 新しいプロジェクトを作成する

このチュートリアルで利用する新しいプロジェクトを作成する手順は以下になります。

1.  ローカルコンピュータ上でターミナルシェルを、まだ開いていなければ開きます。

2.  別のフォルダを利用しているなら、Internet Computer プロジェクトで使うフォルダに移動します。

3.  以下のコマンドを実行して、新しいプロジェクトを作成します。

        dfx new location_hello

4.  以下のコマンドを実行して、プロジェクトディレクトリに移動します。

        cd location_hello

## デフォルト設定を変更する

[デフォルトプロジェクトを探索する](explore-templates) チュートリアルでは、新しいプロジェクトを作成すると、デフォルトの `dfx.json` 設定ファイルがプロジェクトディレクトリに追加されることを確認しました。 ここではファイルのデフォルト設定を見て、使用したいプロジェクトの設定を正確に反映しているかどうかを常に確認する必要があります。 このチュートリアルでは、デフォルトの設定を変更して、使用しない設定を削除します。

設定ファイル `dfx.json` を変更するには以下のようにします。

1.  `dfx.json` 設定ファイルをテキストエディタで開きます。

2.  `location_hello` プロジェクトのデフォルト設定を確認します。

3.  不要な設定をすべて削除します。

    このチュートリアルでは、フロントエンドのアセットを作成しないので、ファイルから `location_hello_assets` の設定をすべて削除することができます。

4.  変更内容を保存し、ファイルを閉じて次に進みます。

## デフォルトプログラムを変更する

[デフォルトプロジェクトを探索する](explore-templates) チュートリアルでは、新しいプロジェクトを作成すると、デフォルト `src` ディレクトリが、テンプレート `main.mo` ファイルと共に作成されることがわかりました。

デフォルトのテンプレートソースコードを変更するには以下のようにします。

1.  `src/location_hello/main.mo` ソースコードファイルをテキストエディタで開きます。

2.  デフォルトのソースコードを修正して、`greet` 関数を `location` 関数に、`name` 引数を `city` 引数に置き換えます。

    例えば、[こちら](../_attachments/location_hello.mo) のようになります。

3.  変更内容を保存し、ファイルを閉じて次に進みます。

## ローカル Canister 実行環境を起動する

プロジェクトをビルドする前に、ローカルの Canister 実行環境か、 Internet Computer ブロックチェーンメインネットに接続する必要があります。

Canister 実行環境をローカルで起動するには、`dfx.json` ファイルが必要なので、プロジェクトのルートディレクトリにいることを確認してください。 このチュートリアルでは、2 つの独立したターミナルシェルを用意して、1 つのターミナルでネットワーク操作を開始・確認し、別のターミナルでプロジェクトを管理できるようにします。

ローカル Canister 実行環境を起動するには以下のようにします。

1.  新しいターミナルウィンドウまたはタブを開きます。

2.  必要であれば、プロジェクトのルートディレクトリに移動してください。

    - **2 つのターミナル** を開いている必要があります。

    - **プロジェクトディレクトリ** を **カレントワーキングディレクトリ** にする必要があります。

3.  以下のコマンドを実行して、ローカルコンピュータ上で Canister 実行環境を起動します。

        dfx start

    受信したネットワーク接続を許可するかどうかのプロンプトが表示されたら、**Allow** をクリックします。

4.  ネットワーク操作が表示されているターミナルを開いたまま、プロジェクトを作成した元のターミナルに注目します。

## Dapp の登録、ビルド、デプロイ

ローカル Canister 実行環境に接続すると、ローカルで Dapp の登録、ビルド、デプロイを行うことができます。

Dapp をローカルでデプロイするには以下のようにします。

1.  必要に応じて、プロジェクトのルートディレクトリにいることを確認します。

2.  以下のコマンドを実行して、アプリケーションの登録、ビルド、デプロイを行います。

        dfx deploy

    `dfx deploy` コマンドの出力には、実行する操作に関する情報が表示されます。

## テキスト引数を渡す

これで、ローカル Cansiter 実行環境に **Canister スマートコントラクト** としてプログラムがデプロイされ、`dfx canister call` コマンドを使ってプログラムをテストすることができます。

ローカルにデプロイしたプログラムをテストするには以下のようにします。

1.  以下のコマンドを実行して、プログラムの中で `location` メソッドを呼び出し、`text` 型の `city` 引数を渡すには、次のコマンドを実行します。

        dfx canister call location_hello location "San Francisco"

    この場合の引数には、`San` と `Francisco` の間にスペースが含まれているため、引数を引用符で囲む必要があります。 このコマンドは以下のような出力を表示します。

        ("Hello, San Francisco!")

    引数がスペースを含んでおらず、テキストを引用符で囲む必要がなければ、Candid インターフェース記述言語に次のようにデータ型を推測させることができます。

        dfx canister call location_hello location Paris

    Candid はデータを `Text` 型と推測し、プログラムの出力を以下のようなテキストとして返します。

        ("Hello, Paris!")

2.  プログラム内で `location` メソッドを呼び出し、`city` の引数を Candid インターフェース記述言語のテキスト引数の構文を使って明示的に渡します。

        dfx canister call location_hello location '("San Francisco and Paris")'

    このコマンドは、以下のような出力を表示します。

        ("Hello, San Francisco and Paris!")

    プログラムは 1 つのテキスト引数しか受け付けないため、複数の文字列を指定すると最初の引数のみが返されます。

    例えば、このコマンドを試してみてください。

        dfx canister call location_hello location '("San Francisco","Paris","Rome")'

    最初の引数である `("Hello, San Francisco!")` のみが返されます。

## プログラムのソースコードを修正する

このチュートリアルで学んだことを発展させるために、ソースコードを修正して異なる結果を返すようにしてみましょう。 例えば、`location` 関数を修正して、複数の都市名を返すようにしてみましょう。

ソースコードを変更して実験するには以下のようにします。

1.  テキストエディタで設定ファイル `dfx.json` を開き、デフォルトの `location_hello` の設定を `favorite_cities` に変更します。

    このステップでは、Canister 名と Canister スマートコントラクトのメインプログラムへのパスの両方を、`favorite_cities` を使用するために変更する必要があります。

2.  変更内容を保存して、`dfx.json` ファイルを閉じて、次に進みます。

3.  以下のコマンドを実行して、`location_hello` のソースファイルディレクトリを、`dfx.json` 設定ファイルで指定した名前に合わせてコピーします。

        cp -r src/location_hello src/favorite_cities

4.  テキストエディターで `src/favorite_cities/main.mo` ファイルを開きます。

5.  以下のコードサンプルをコピー＆ペーストして、`location` 関数を 2 つの新しい関数で置き換えます。

    例えば、[こちら](../_attachments/favorite_cities.mo)です。

    このコード例では、`Text` が角括弧（`[ ]`）で囲まれていることにお気づきかもしれません。`Text` それ自体は UTF-8 の文字の集まりを表しています。 角括弧で囲まれた型は、その型が **配列** であることを示しています。 したがって、この文脈では、`[Text]` は、UTF-8 文字の集まりの配列を示し、プログラムが複数のテキスト文字列を受け入れて返すことができるようになります。

    また、このコードサンプルでは、配列に対する `apply` 処理の基本的なフォーマットを使用しており、次のように抽象化することができます。

        public func apply<A, B>(fs : [A -> B], xs : [A]) : [B] {
            var ys : [B] = [];
            for (f in fs.vals()) {
              ys := append<B>(ys, map<A, B>(f, xs));
            };
            ys;
        };

    配列に対する操作を行う関数については、Motoko ベースライブラリの配列モジュールの説明や、_Motoko プログラミング言語レファレンス_ を参照してください。 配列の使用に焦点を当てた別の例については、[examples](https://github.com/dfinity/examples/) リポジトリにある [クイックスタート](https://github.com/dfinity/examples/tree/master/motoko/quicksort) を参照してください。

6.  以下のコマンドを実行して、Dapp を登録、ビルド、デプロイします。

        dfx deploy

7.  以下のコマンドを実行して、プログラムの中で `location` メソッドを呼び出し、`city` 引数を Candid インターフェース記述の構文を使って渡します。

        dfx canister call favorite_cities location '(vec {"San Francisco";"Paris";"Rome"})'

    このコマンドは、Candid インターフェース記述の構文にある `(vec { val1; val2; val3; })` を用いて、値のベクトルを返します。 インターフェイス記述言語 Candid の詳細については、[Candid](../candid/candid-intro.md) 言語ガイドをご覧ください。

    このコマンドは、以下のような出力を表示します。

        ("Hello, from ["San Francisco", "Paris", "Rome"]!")

8.  以下のコマンドを実行して、プログラムの中で `location_pretty` メソッドを呼び出し、インターフェイス記述の構文を使って `city` の引数を渡します。

        dfx canister call favorite_cities location_pretty '(vec {"San Francisco";"Paris";"Rome"})'

    このコマンドは、以下のような出力を表示します。

        ("Hello from San Francisco, Paris, Rome, bon voyage!")

9.  Candid UI でコードをテストします。

    コードをテストするには、[こちら](candid-ui.md) のインストラクションに従ってください。
    この例では、各関数はテキスト文字列の配列を受け取ります。そのため、まず配列の長さを選択し、各項目に値を設定してから **Call** をクリックします。

    ![Specifying an array](../_attachments/candid-favorite-cities-result.png)

## ローカル Canister 実行環境を停止する

プログラムの実験が終わったら、ローカル Canister 実行環境を停止して、バックグラウンドで実行し続けないようにします。

ローカル Canister 実行環境を停止するには以下のようにします。

1.  ネットワーク処理を表示するターミナルで Control-C を押して、ローカルプロセスを中断します。

2.  以下のコマンドを実行して、Canister 実行環境を停止します。

        dfx stop

<!--
# Pass text arguments

This tutorial provides a simple variation on the default program that lets you pass a single text argument to a single actor, compile the code to create a canister, then retrieve the argument. Throughout this document, the terms canister and canister are considered synonymous.

This tutorial illustrates how to pass arguments on the command-line in a terminal using the Candid interface description language (IDL) and how to modify the program to allow it to accept more than one value for the text argument.

## Before you begin

Before you start this tutorial, verify the following:

-   You have downloaded and installed the SDK package as described in [Download and install](../../quickstart/local-quickstart#download-and-install).

-   You have stopped any canister execution environments running on the local computer.

This tutorial takes approximately 20 minutes to complete.

## Create a new project

To create a new project for this tutorial:

1.  Open a terminal shell on your local computer, if you don’t already have one open.

2.  Change to the folder you are using for your Internet Computer projects, if you are using one.

3.  Create a new project by running the following command:

        dfx new location_hello

4.  Change to your project directory by running the following command:

        cd location_hello

## Modify the default configuration

In the [Explore the default project](explore-templates) tutorial, you saw that creating a new project adds a default `dfx.json` configuration file to your project directory. You should always review the default settings in the file to verify the information accurately reflects the project settings you want to use. For this tutorial, you’ll modify the default configuration to remove settings that aren’t used.

To modify settings in the `dfx.json` configuration file:

1.  Open the `dfx.json` configuration file in a text editor.

2.  Check the default settings for the `location_hello` project.

3.  Remove all of the unnecessary configuration settings.

    Because this tutorial does not involve creating any frontend assets, you can remove all of the `location_hello_assets` configuration settings from the file.

4.  Save your changes and close the file to continue.

## Modify the default program

In the [Explore the default project](explore-templates) tutorial, you saw that creating a new project creates a default `src` directory with a template `main.mo` file.

To modify the default template source code:

1.  Open the `src/location_hello/main.mo` source code file in a text editor.

2.  Modify the default source code to replace the `greet` function with a `location` function and the `name` argument with a `city` argument.

    For example like [this](../_attachments/location_hello.mo).

3.  Save your changes and close the file to continue.

## Start the local canister execution environment

Before you can build your project, you need to connect to a local canister execution environment or the Internet Computer blockchain mainnet.

Starting a canister execution environment locally requires a `dfx.json` file, so you should be sure you are in your project’s root directory. For this tutorial, you should have two separate terminal shells, so that you can start and see network operations in one terminal and manage your project in another.

To start the local canister execution environment:

1.  Open a new terminal window or tab on your local computer.

2.  Navigate to the root directory for your project, if necessary.

    -   You should now have **two terminals** open.

    -   You should have the **project directory** as your **current working directory**.

3.  Start the canister execution environment on your local computer by running the following command:

        dfx start

    If you are prompted to allow or deny incoming network connections, click **Allow**.

4.  Leave the terminal that displays network operations open and switch your focus to your original terminal where you created your project.

## Register, build, and deploy the dapp

After you connect to the local canister execution environment, you can register, build, and deploy your dapp locally.

To deploy the dapp locally:

1.  Check that you are still in the root directory for your project, if needed.

2.  Register, build, and deploy your application by running the following command:

        dfx deploy

    The `dfx deploy` command output displays information about the operations it performs.

## Pass a text argument

You now have a program deployed as a **canister** in your local canister execution environment and can test your program by using `dfx canister call` commands.

To test the program you have deployed locally:

1.  Call the `location` method in the program and pass your `city` argument of type `text` by running the following command:

        dfx canister call location_hello location "San Francisco"

    Because the argument in this case includes a space between `San` and `Francisco`, you need to enclose the argument in quotes. The command displays output similar to the following:

        ("Hello, San Francisco!")

    If the argument did not contain a space that required enclosing the text inside of quotation marks, you could allow the Candid interface description language to infer the data type like this:

        dfx canister call location_hello location Paris

    Candid infers the data type as `Text` and returns the output from your program as text like this:

        ("Hello, Paris!")

2.  Call the `location` method in the program and pass your `city` argument explicitly using the Candid interface description language syntax for Text arguments:

        dfx canister call location_hello location '("San Francisco and Paris")'

    The command displays output similar to the following:

        ("Hello, San Francisco and Paris!")

    Because your program only accepts a single text argument, specifying multiple strings returns only the first argument.

    For example, if you try this command:

        dfx canister call location_hello location '("San Francisco","Paris","Rome")'

    Only the first argument—`("Hello, San Francisco!")`—is returned.

## Revise the source code in your program

To extend what you have learned in this tutorial, you might want to try modifying the source code to return different results. For example, you might want to modify the `location` function to return multiple city names.

To experiment with modifying the source code for this tutorial:

1.  Open the `dfx.json` configuration file in a text editor and change the default `location_hello` settings to `favorite_cities`.

    For this step, you should modify both the canister name and the path to the main program for the canister to use `favorite_cities`.

2.  Save your changes and close the `dfx.json` file to continue.

3.  Copy the `location_hello` source file directory to match the name specified in the `dfx.json` configuration file by running the following command:

        cp -r src/location_hello src/favorite_cities

4.  Open the `src/favorite_cities/main.mo` file in a text editor.

5.  Copy and paste the following code sample to replace the `location` function with two new functions.

    For example like [this](../_attachments/favorite_cities.mo).

    You might notice that `Text` in this code example is enclosed by square (`[ ]`) brackets. By itself, `Text` represents a collection of UTF-8 characters. The square brackets around a type indicate that it is an **array** of that type. In this context, therefore, `[Text]` indicates an array of a collection of UTF-8 characters, enabling the program to accept and return multiple text strings.

    The code sample also uses the basic format of an `apply` operation for the array, which can be abstracted as:

        public func apply<A, B>(fs : [A -> B], xs : [A]) : [B] {
            var ys : [B] = [];
            for (f in fs.vals()) {
                ys := append<B>(ys, map<A, B>(f, xs));
            };
            ys;
        };

    For information about the functions that perform operations on arrays, see the description of the Array module in the Motoko base library or the *Motoko Programming Language Reference*. For another example focused on the use of arrays, see [Quicksort](https://github.com/dfinity/examples/tree/master/motoko/quicksort) in the [examples](https://github.com/dfinity/examples/) repository.

6.  Register, build, and deploy the dapp by running the following command:

        dfx deploy

7.  Call the `location` method in the program and pass your `city` argument using the Candid interface description syntax by running the following command:

        dfx canister call favorite_cities location '(vec {"San Francisco";"Paris";"Rome"})'

    The command uses the Candid interface description syntax `(vec { val1; val2; val3; })` to return a vector of values. For more information about the Candid interface description language, see the [Candid](../candid/candid-intro.md) language guide.

    This command displays output similar to the following:

        ("Hello, from ["San Francisco", "Paris", "Rome"]!")

8.  Call the `location_pretty` method in the program and pass your `city` argument using the interface description syntax by running the following command:

        dfx canister call favorite_cities location_pretty '(vec {"San Francisco";"Paris";"Rome"})'

    The command displays output similar to the following:

        ("Hello from San Francisco, Paris, Rome, bon voyage!")

9.  Test your code using the candid ui.

    To test your code, follow the instructions [here](candid-ui.md).
    In this example, each function accepts an array of text strings. Therefore, you first select the length of the array, then set values for each item before clicking **Call**.

    ![Specifying an array](../_attachments/candid-favorite-cities-result.png)

## Stop the local canister execution environment

After you finish experimenting with your program, you can stop the local canister execution environment so that it doesn’t continue running in the background.

To stop the local canister execution environment:

1.  In the terminal that displays network operations, press Control-C to interrupt the local process.

2.  Stop the canister execution environment by running the following command:

        dfx stop

-->
