# ウォレットから Cycle を受け取る

ローカルで開発を行っている場合、プロジェクト内のデフォルトのウォレットを使用して Cycle を送信し、Cycle 残高を確認することができます。しかし、例えば機能を実行したり、ユーザーにサービスを提供したりするために Cycle を受信して使用する必要がある Canister はどうすればよいでしょうか？このチュートリアルでは、デフォルトのテンプレートプログラムに、Cycle を受信し、Cycle 残高を確認する機能を追加する方法を説明するための簡単な例を提供します。

この例は、次のように構成されています：

-   `wallet_balance` 関数は、Canister の現在の Cycle バランスを確認することができます。

-   `wallet_receive` 関数は、Canister に送信された Cycle をプログラムが受け取ることができるようにします。

-   `greet` 関数はテキスト引数を受け取り、ターミナルに挨拶文を表示します。

-   `owner` 関数は、メッセージの呼び出し元が使用している Principal を返します。

## 始める前に

チュートリアルを始める前に、以下を確認してください：

-   [ダウンロードとインストール](../../quickstart/local-quickstart#download-and-install)で説明されているように、SDK パッケージをダウンロードしインストールしていること。

-   IDE に Visual Studio Code を使用している場合は、[言語エディタプラグインのインストール](../../quickstart/local-quickstart.md#install-vscode)にあるように、Motoko 用プラグインをインストールしていること。

-   コンピューター上で動作しているローカル Canister の実行環境を停止していること。

## 新しいプロジェクトを作成する

アクセスコントロールとユーザー Identity の切り替えをテストするために、新しいプロジェクトディレクトリを作成するには：

1.  ローカルコンピューターでターミナルシェルを開きます（まだ開いていない場合）。

2.  Internet Computer のプロジェクトに使用しているフォルダがある場合はそちらに移動します。

3.  次のコマンドを実行して、新しいプロジェクトを作成します：

        dfx new cycles_hello

4.  以下のコマンドを実行して、プロジェクトディレクトリに移動してください：

        cd cycles_hello

## デフォルトのプログラムを修正する

このチュートリアルでは、Cycle を受け入れ、Cycle バランスをチェックするための新しい関数を含めるために、テンプレートのソースコードを変更することになります。

デフォルトのプログラムを修正する：

1.  `src/cycles_hello/main.mo` ファイルをテキストエディタで開き、既存のコンテンツを削除します。

2.  [このコード](../_attachments/cycles-main.mo)をコピーして、ファイルに貼り付けます。

    このプログラムのいくつかの重要な要素を見てみましょう：

    -   このプログラムは、Cycle を扱うための基本的な機能を提供する Motoko ベースライブラリ `ExperimentalCycles` をインポートしています。

    -   このプログラムでは、単一の Actor ではなく、「Actor クラス」を使用することで、複数の Actor インスタンスを持ち、すべてのインスタンスの `capacity` まで異なる Cycle の量を受け入れることができるようになっています。

    -   `capasity` は、Actor クラスの引数として渡されます。

    -   `msg.caller` は、呼び出しに関連する Principal を識別します。

3.  変更を保存して、`main.mo` ファイルを閉じてください。

## ローカル Canister 実行環境を起動する

`access_hello` プロジェクトをビルドする前に、開発環境のローカルで動作している Canister 実行環境に接続するか、アクセス可能なサブネットに接続する必要があります。

ローカル Canister の実行環境を起動するには：

1.  ローカルコンピューターで新しいターミナルウィンドウまたはタブを開きます。

2.  必要であれば、プロジェクトのルートディレクトリに移動します。

3.  以下のコマンドを実行して、PC のローカル Canister 実行環境を起動します：

        dfx start --clean --background

    ローカル Canister の実行環境が起動操作を完了したら、次のステップに進みます。

## Dapp の登録、ビルド、デプロイ

ローカル Canister 実行環境に接続した後、ローカルで Dapp の登録、ビルド、デプロイができます。

ローカルに Dapp をデプロイするには：

1.  必要に応じて、まだプロジェクトのルートディレクトリにいることを確認します。

2.  以下のコマンドを実行して、Dapp を登録、ビルド、デプロイしてください：

        dfx deploy --argument '(360000000000)'

    この例では、 Canister の `capacity` を 360,000,000,000  Cycle に設定します。そして、`dfx deploy` コマンドの出力は、このローカルプロジェクト用に作成されたウォレット Canister とウォレット Canister の識別子に関連する Identity など、実行した操作についての情報を表示します。

    例：

        Deploying all canisters.
        Creating canisters...
        Creating canister "cycles_hello"...
        Creating the canister using the wallet canister...
        Creating a wallet canister on the local network.
        The wallet canister on the "local" network for user "default" is "rwlgt-iiaaa-aaaaa-aaaaa-cai"
        "cycles_hello" canister created with canister id: "rrkah-fqaaa-aaaaa-aaaaq-cai"
        Creating canister "cycles_hello_assets"...
        Creating the canister using the wallet canister...
        "cycles_hello_assets" canister created with canister id: "ryjl3-tyaaa-aaaaa-aaaba-cai"
        Building canisters...
        Building frontend...
        Installing canisters...
        Installing code for canister cycles_hello, with canister_id rrkah-fqaaa-aaaaa-aaaaq-cai
        Installing code for canister cycles_hello_assets, with canister_id ryjl3-tyaaa-aaaaa-aaaba-cai
        Authorizing our identity (default) to the asset canister...
        Uploading assets to asset canister...
        Deployed canisters.

## Dapp をテストする

ローカル Canister 実行環境に Dapp をデプロイした後、`dfx canister call` コマンドを使ってウォレット機能を試したり、プログラムをテストしたりすることができます。

Dapp をテストするために：

1.  以下のコマンドを実行して、`default` ユーザ Identity の Principal を確認します：

        dfx canister call cycles_hello owner

    このコマンドは、現在の Identity に対して次のような出力を表示します：

        (principal "g3jww-sbmtm-gxsag-4mecu-72yc4-kef5v-euixq-og2kd-sav2v-p2sb3-pae")

    `dfx deploy` コマンドを実行するときに使用していた Identity を変更していない場合は、`dfx identity get-principal` コマンドを実行して同じ Principal を取得する必要があります。 Cycle の送信や他の **カストディアン** 識別子に Cycle を送信する許可を与えるなどの特定のタスクを実行するには、ウォレット Canister の所有者である必要があるため、これは重要なことです。

2.  以下のコマンドを実行して、初期のウォレット Cycle 残高を確認します：

        dfx canister call cycles_hello wallet_balance

     Canister に Cycle を送っていないので、コマンドは次のような残高を表示します：

        (0)

3.  以下のようなコマンドを実行して、 Canister の Principal を使用して、デフォルトウォレット Canister から `cycles_hello`  Canister にいくつかの Cycle を送信します。

        dfx canister call rwlgt-iiaaa-aaaaa-aaaaa-cai wallet_send '(record { canister = principal "rrkah-fqaaa-aaaaa-aaaaq-cai"; amount = (256000000000:nat64); } )'

4.  `wallet_balance` 関数を呼び出して、`cycles_hello`  Canister に転送した Cycle 数（許容容量以下の量を指定した場合）、または `dfx deploy` コマンドを実行したときに指定した `capacity` があるかどうかを確認します。

        dfx canister call cycles_hello wallet_balance

    このコマンドは、次のような出力を表示します：

        (256_000_000_000)

5.  以下のようなコマンドを実行して、`wallet_balance` 関数を呼び出し、デフォルトウォレットの Cycle 数を確認します：

        dfx canister call rwlgt-iiaaa-aaaaa-aaaaa-cai wallet_balance

    このコマンドは、指定したウォレット Canister の識別子の残高を、Candid フォーマットを使用したレコードとして返します。例えば、`amount` フィールド（ハッシュ `3_573_748_184` で表される）と残高 97,738,624,621,042  Cycleを持つレコードは、このように表示されるでしょう：

        (record { 3_573_748_184 = 97_738_624_621_042 })

    この簡単なチュートリアルでは、 Cycle は `cycles_hello` Canister からではなく、デフォルトのウォレット Canister の残高からしか消費されないようにしています。

6.  以下のようなコマンドを実行して、`greet` 関数を呼び出します：

        dfx canister call cycles_hello greet '("from DFINITY")'

7.  デフォルトのウォレットから差し引かれた Cycle 数を確認するには、`wallet_balance` 関数の呼び出しを再実行します：

        dfx canister call rwlgt-iiaaa-aaaaa-aaaaa-cai wallet_balance

    たとえば、次のような結果が得られます：

        (record { 3_573_748_184 = 97_638_622_179_500 })

## ローカル Canister の実行環境を停止する

プログラムの実験が終了したら、ローカル Canister 実行環境を停止して、バックグラウンドで実行し続けないようにできます。

ローカル Canister の実行環境を停止するには、以下の手順に従います：

1.  操作が表示されている端末で、Control-C キーを押して処理を中断します。

2.  以下のコマンドを実行して、ローカル Canister の実行環境を停止します：

        dfx stop

## もっと学ぶには？

Cycle を使った仕事についてもっと知りたい方は、以下の関連資料をご覧ください：

-   [トークンと Cycle](../../../concepts/tokens-cycles.md)

-   [dfx identity (command reference)](../../../references/cli-reference/dfx-identity.md)

<!-- -   [Set an identity to own a canister (how-to)](../working-with-canisters#set-owner) -->

-   [Cycle を管理する](../cdks/motoko-dfinity/cycles.md)

-   [ExperimentalCycles (base module)](../../../references/motoko-ref/ExperimentalCycles.md)

<!--
# Accept cycles from a wallet

When you are doing local development, you can use the default wallet in your project to send cycles and check your cycle balance. But what about the canisters that need to receive and use those cycles, e.g. to execute their functions and provide services to users? This tutorial provides a simple example to illustrate how you might add the functions to receive cycles and check your cycle balance to the default template program.

This example consists of the following:

-   The `wallet_balance` function enables you to check the current cycle balance for the canister.

-   The `wallet_receive` function enables the program to accept cycles that are sent to the canister.

-   The `greet` function accepts a text argument and displays a greeting in a terminal.

-   The `owner` function returns the principal used by the message caller.

## Before you begin

Before starting the tutorial, verify the following:

-   You have downloaded and installed the SDK package as described in [Download and install](../../quickstart/local-quickstart.md#download-and-install).

-   You have installed the Visual Studio Code plugin for Motoko as described in [Install the language editor plug-in](../../quickstart/local-quickstart.md#install-vscode) if you are using Visual Studio Code as your IDE.

-   You have stopped any local canister execution environment running on the local computer.

## Create a new project

To create a new project directory for testing access control and switching user identities:

1.  Open a terminal shell on your local computer, if you don’t already have one open.

2.  Change to the folder you are using for your Internet Computer projects, if you are using one.

3.  Create a new project by running the following command:

        dfx new cycles_hello

4.  Change to your project directory by running the following command:

        cd cycles_hello

## Modify the default program

For this tutorial, you are going to modify the template source code to include new functions for accepting cycles and checking the cycle balance.

To modify the default program:

1.  Open the `src/cycles_hello/main.mo` file in a text editor and delete the existing content.

2.  Copy and paste [this code](../_attachments/cycles-main.mo) into the file.

    Let’s take a look at a few key elements of this program:

    -   The program imports a Motoko base library—`ExperimentalCycles`—that provides basic functions for working with cycles.

    -   The program uses an `actor class` instead of a single actor so that it can have multiple actor instances to accept different cycle amounts up to a `capacity` for all instances.

    -   The `capacity` is passed as an argument to the actor class.

    -   The `msg.caller` identifies the principal associated with the call.

3.  Save your changes and close the `main.mo` file to continue.

## Start the local canister execution environment

Before you can build the `access_hello` project, you need to connect to the canister execution environment running locally in your development environment, or you need to connect to a subnet that you can access.

To start the local canister execution environment:

1.  Open a new terminal window or tab on your local computer.

2.  Navigate to the root directory for your project, if necessary.

3.  Start the local canister execution environment on your machine by running the following command:

        dfx start --clean --background

    After the local canister execution environment completes its startup operations, you can continue to the next step.

## Register, build, and deploy the dapp

After you connect to the local canister execution environment, you can register, build, and deploy your dapp locally.

To deploy the dapp locally:

1.  Check that you are still in the root directory for your project, if needed.

2.  Register, build, and deploy your dapp by running the following command:

        dfx deploy --argument '(360000000000)'

    This example sets the `capacity` for the canister to 360,000,000,000 cycles. The `dfx deploy` command output then displays information about the operations it performs, including the identity associated with the wallet canister created for this local project and the wallet canister identifier.

    For example:

        Deploying all canisters.
        Creating canisters...
        Creating canister "cycles_hello"...
        Creating the canister using the wallet canister...
        Creating a wallet canister on the local network.
        The wallet canister on the "local" network for user "default" is "rwlgt-iiaaa-aaaaa-aaaaa-cai"
        "cycles_hello" canister created with canister id: "rrkah-fqaaa-aaaaa-aaaaq-cai"
        Creating canister "cycles_hello_assets"...
        Creating the canister using the wallet canister...
        "cycles_hello_assets" canister created with canister id: "ryjl3-tyaaa-aaaaa-aaaba-cai"
        Building canisters...
        Building frontend...
        Installing canisters...
        Installing code for canister cycles_hello, with canister_id rrkah-fqaaa-aaaaa-aaaaq-cai
        Installing code for canister cycles_hello_assets, with canister_id ryjl3-tyaaa-aaaaa-aaaba-cai
        Authorizing our identity (default) to the asset canister...
        Uploading assets to asset canister...
        Deployed canisters.

## Test the dapp

After you have deployed the dapp on your local canister execution environment, you can experiment with the wallet functions and test your program by using `dfx canister call` commands.

To test the dapp:

1.  Check the principal for the `default` user identity by running the following command:

        dfx canister call cycles_hello owner

    The command displays output similar to the following for the current identity:

        (principal "g3jww-sbmtm-gxsag-4mecu-72yc4-kef5v-euixq-og2kd-sav2v-p2sb3-pae")

    If you haven’t made changes to the identity you were using to run the `dfx deploy` command, you should get the same principal by running the `dfx identity get-principal` command. This is important because you must be the owner of the wallet canister to perform certain tasks such as sending cycles or granting other **custodian** identities permission to send cycles.

2.  Check the initial wallet cycle balance by running the following command:

        dfx canister call cycles_hello wallet_balance

    You haven’t sent any cycles to the canister, so the command displays the following balance:

        (0)

3.  Send some cycles from your default wallet canister to the `cycles_hello` canister using the canister principal by running a command similar to the following:

        dfx canister call rwlgt-iiaaa-aaaaa-aaaaa-cai wallet_send '(record { canister = principal "rrkah-fqaaa-aaaaa-aaaaq-cai"; amount = (256000000000:nat64); } )'

4.  Call the `wallet_balance` function to see that the `cycles_hello` canister has the number of cycles you transferred, if you specified an amount under the allowed capacity, or the `capacity` you specified when you ran the `dfx deploy` command.

        dfx canister call cycles_hello wallet_balance

    The command displays output similar to the following:

        (256_000_000_000)

5.  Call the `wallet_balance` function to see the number of cycles in your default wallet by running a command similar to the following:

        dfx canister call rwlgt-iiaaa-aaaaa-aaaaa-cai wallet_balance

    The command returns the balance for the wallet canister identifier you specified as a record using Candid format. For example, the command might display a record with an `amount` field (represented by the hash `3_573_748_184`) and a balance of 97,738,624,621,042 cycles like this:

        (record { 3_573_748_184 = 97_738_624_621_042 })

    For this simple tutorial, cycles are only consumed from the balance in the default wallet canister, not from the `cycles_hello` canister.

6.  Call the `greet` function by running a command similar to the following:

        dfx canister call cycles_hello greet '("from DFINITY")'

7.  Rerun the call to the `wallet_balance` function to see the number of cycles deducted from your default wallet:

        dfx canister call rwlgt-iiaaa-aaaaa-aaaaa-cai wallet_balance

    For example, you might a result similar to this:

        (record { 3_573_748_184 = 97_638_622_179_500 })

## Stop the local canister execution environment

After you finish experimenting with the program, you can stop the local canister execution environment so that it doesn’t continue running in the background.

To stop the local canister execution environment:

1.  In the terminal that displays the operations, press Control-C to interrupt the process.

2.  Stop the local canister execution environment by running the following command:

        dfx stop

## Want to learn more?

If you are looking for more information about working with cycles, check out the following related resources:

-   [Tokens and cycles (overview)](../../../concepts/tokens-cycles.md)

-   [dfx identity (command reference)](../../../references/cli-reference/dfx-identity.md)

<!-- -   [Set an identity to own a canister (how-to)](../working-with-canisters#set-owner) -->
<!--
-   [Managing cycles (language reference)](../cdks/motoko-dfinity/cycles.md)

-   [ExperimentalCycles (base module)](../../../references/motoko-ref/ExperimentalCycles.md)

-->