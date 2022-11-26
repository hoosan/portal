# Canister を管理する

このセクションのチュートリアルに沿って、または [examples](https://github.com/dfinity/examples) リポジトリからサンプルをクローンして、SDK を使った体験したことがある方は、プログラムを **Canister** としてビルドおよびデプロイする方法についてすでにご存知でしょう。このセクションでは、 Canister のライフサイクルと Canister の管理方法に関する追加情報を提供します。

## Canister 識別子を取得する

あなたの望む開発ワークフローに応じて、プログラムをコンパイルする前後に、 Canister の一意の識別子を取得することができます。例えば、コードを書く前にサブネット上の Canister の一意な識別子を確保したい場合、`dfx canister create` コマンドを実行することで可能になります。このコマンドは基本的に空の Canister プレースホルダーを作成し、そこに後でコードをインストールすることができます。作成された Canister は一意な識別子を取得します。

Canister に固有の識別子を取得するには：

1.  コンピュータで新しいターミナルウィンドウまたはタブを開きます。

2.  次のようなコマンドを実行して、作成する予定の Canister の新規プロジェクトを作成します：

        `dfx new YOUR-PROJECT-NAME`

    プロジェクトに使用する名前は、デフォルトで Canister の名前としても使用されることに注意してください。

3.  新しいプロジェクトディレクトリに移動します。

4.  設定ファイル `dfx.json` を開き、利用したい Canister 実行環境（例：IC ブロックチェーン）のホストとポートを設定します。

    ローカルにデプロイしている場合は、このステップを省略できます。

    また、コードをコンパイルする前に、必要と思われる追加 Canister の識別子を作成したい場合は、オプションで Canister の名前を変更したり、Canister の設定を設定ファイルに追加したりすることもできます。

5.  必要に応じて、以下のコマンドを実行して、ローカルの Canister 実行環境を起動します：

        dfx start --background

    ほとんどの場合、このステップはローカルで Canister を動作させる場合にのみ必要です。

    もし、IC ブロックチェーンなどのリモート実行環境上で実行する Canister を登録する場合、このパラメータで指定した環境上でタスクを実行するために、`—network` コマンドラインオプションを含めることになります。

6.  以下のコマンドを実行して、`dfx.json` で定義された Canister に一意の識別子を登録します：

        dfx canister create --all

    このコマンドは `.dfx/local` ディレクトリを作成し、そのディレクトリにプロジェクト用の `canister_ids.json` ファイルを追加します。

## ローカルの識別子を持つ Canister のビルド

プロジェクトのソースコードを記述した後、それを WebAssembly モジュールにコンパイルしてから、Canister としてデプロイする必要があります。

ローカルデバッグのためにのみプロジェクトをコンパイルする場合は、プロジェクトに対してローカルに定義された識別子を生成することができます。

ローカルに定義された識別子を生成するには：

1.  必要な設定ファイルとプログラムロジックを用意し、プロジェクトを作成します。

2.  必要に応じて、ローカルの Canister 実行環境を起動します。

    IC ブロックチェーンなどのリモート実行環境で実行する Canister をコンパイルする場合は、`--network` コマンドラインオプションを指定し、このパラメータで指定した環境上でタスクを実行するようにします。

3.  `dfx.json` で定義された Canister に対して、以下のコマンドを実行して、ハードコードされたローカル識別子を生成してください：

        dfx build --check

    なお、IC ブロックチェーン上にプロジェクトをデプロイする前に、ローカルで定義された識別子に代わるユニークな Canister 識別子を登録する必要があります。

## Canister のデプロイ

プログラムをコンパイルした後、コンパイルしたコードを、ローカルの Canister 実行環境または IC ブロックチェーン上で動作する Canister にインストールすることができます。

事前またはビルドプロセス中に作成された Canister 識別子が、デプロイ時にインストールされるコードの場所を決定します。

コードを初めてデプロイするには：

1.  新しいターミナルを開き、プロジェクトのディレクトリに移動します。

2.  必要に応じて、ローカル Canister 実行環境を起動します。

    ほとんどの場合、このステップは Canister をローカルで実行する場合のみ必要です。

    もし、IC ブロックチェーンなどのリモート実行環境で実行する Canister を登録する場合は、`--network` コマンドラインオプションを含め、このパラメータで指定した環境上でタスクを実行するようにします。

3.  デプロイしたいすべての Canister の Canister 識別子があることを確認します。

4.  次のコマンドを実行して、すべての Canister をデプロイします：

        dfx canister install --all

##  Canister ID の検索

すべての Canister には一意の識別子があります。 Canister と対話するために、これらの識別子を使用する必要があることがよくあります。たとえば、Dapp 用のフロントエンド Canister にアクセスしたい場合や、Candid Web インターフェイスを使用してサービスと対話したい場合は、適切な Canister 識別子を指定する必要があります。

識別子は Canister がデプロイされる環境に固有なので、情報を格納するためのファイルは異なるディレクトリにあります。例えば、ローカルにデプロイされた Canister の識別子は、プロジェクトの `.dfx/local/canister_ids.json` ファイルに格納されます。

特定の Canister の識別子を調べるには、`dfx canister id` コマンドを実行します。例えば、ローカルの Canister 実行環境にデプロイされた `lookup` Canister の Canister ID を調べるには、次のコマンドを実行します：

    dfx canister id lookup

`ic` エイリアスで指定された環境にデプロイされた同じ Canister の識別子を調べるには、次のコマンドを実行します：

    dfx canister --network=ic id lookup

## 既存の Canister にウォレットを追加する

ローカルで開発を行っている場合、新しいプロジェクトを作成すると、そのプロジェクト内の Canister で使用するためのデフォルトのウォレットが自動的に作成されます。もし、以前に作成した Canister を持つプロジェクトにウォレットを追加したい場合は、いくつかの手動ステップを踏むことで強制的に `dfx` にウォレットを生成させることができます。

既存の Canister で使用するためにウォレットを追加するには：

1.  ターミナルを開き、プロジェクトのディレクトリに移動します。

2.  必要に応じて、以下のコマンドを実行して、ローカルの Canister 実行環境を停止します：

        dfx stop

1.  `.dfx` ディレクトリを削除します。

2.  以下のコマンドを実行して、ローカル Canister 実行環境ネットワークを起動します：

        dfx start --clean

## Canister の再インストール

開発中に、デバッグや改良を行う際に、インストールしてプログラムをリプレイスしたい場合があります。

このような場合、登録した Canister の識別子を保持したいが、 Canister のコードやステートは保持したくないと考えるかもしれません。例えば、 Canister には保存したくないテストデータしかない場合や、プログラムを完全に変更することを決定したが、以前のプログラムのインストールに使用した Canister 識別子の下で再インストールしたい場合などです。

Canister を再インストールするには：

1.  新しいターミナルを開き、プロジェクトのディレクトリに移動します。

2.  必要に応じて、ローカルの Canister 実行環境を起動します。

    ほとんどの場合、このステップは Canister をローカルで実行する場合のみ必要です。

    IC ブロックチェーンなどのリモート実行環境で実行する Canister をコンパイルする場合は、`--network` コマンドラインオプションを指定し、このパラメータで指定した環境上でタスクを実行するようにします。

3.  再デプロイするすべての Canister の Canister ID があることを確認します。

4.  次のコマンドを実行して、すべての Canister を再デプロイします。

        dfx canister install --all --mode reinstall

Canister にコードやステートが関連付けられているかどうかに関係なく、`reinstall` モードを使用して、任意の Canister をリプレイスできることに注意してください。

##  Canister を所有するための Identity を設定する

ほとんどの場合、`dfx canister create` コマンドを最初に実行すると、`default` ユーザ ー Identity が自動的に作成されます。このデフォルトのユーザー Identity は、あなたのローカルユーザーアカウント用に生成された公開鍵と秘密鍵のペアで構成されています。通常、この `default` Identity は、あなたが作成するすべてのプロジェクトと、デプロイするすべての Canister のデフォルトの所有者でもあります。しかし、このデフォルトのユーザー Identity を使用しないように、積極的に自分の好きな Identity を作成し、使用することができます。

例として、次のシナリオでは `registered_owner` という Identity を作成し、それを使って `pubs` プロジェクトの登録、ビルド、デプロイ、コールを行う方法を説明します。

プロジェクトに Identity を設定するには：

1.  以下のコマンドを実行して、新しいプロジェクトを作成します：

        dfx new pubs

2.  以下のコマンドを実行して、プロジェクトディレクトリに移動します：

        cd pubs

3.  以下のコマンドを実行して、ローカル Canister の実行環境をバックグラウンドで起動します：

        dfx start --background

4.  以下のコマンドを実行して、新しい `registered_owner` Identity を作成します：

        dfx identity new registered_owner

5.  以下のコマンドを実行して、アクティブユーザーコンテキストが `registered_owner` Identity を使用するように設定します：

        dfx identity use registered_owner

6.  以下のコマンドを実行して、プロジェクト用の Canister を登録、ビルド、デプロイします：

        dfx canister create --all
        dfx build --all
        dfx canister install --all

    これらのコマンドは `registered_owner` という Identity で実行され、そのユーザがデプロイされた Canister の所有者になります。

7.  `greet` 関数を呼び出して、以下のコマンドを実行し、デプロイが成功したことを確認します：

        dfx canister call pubs greet '("Sam")'

##  Canister の動作ステートを管理する

Canister をデプロイすると、ユーザーや他の Canister からのリクエストの受信と処理を開始できるようになります。リクエストの送信とレスポンスの受信が可能な Canister は、**Running** 状態であると見なされます。

通常、Canister はデフォルトで実行状態になりますが、Canister を一時的または恒久的に停止したい場合があります。例えば、Canister をアップグレードする前に停止したい場合があります。 Canister を停止すると、進行中のメッセージが適切に処理され、完了するまで実行するかロールバックする必要がある場合に役立ちます。また、Canister を停止して、Canister 削除の前提条件として、そのメッセージ・ キューをきれいにクリアしたい場合もあります。

`dfx canister status` コマンドを実行することで、全ての Canister や指定した Canister の現在のステートを確認することができます。例えば、ローカルの Canister 実行環境上で動作している全ての Canister のステートを見るには、以下のコマンドを実行します：

    dfx canister status --all

現在 Canister が動作している場合、このコマンドは次のような出力を返します：

    Canister status_check's status is Running.
    Canister status_check_assets's status is Running.

`dfx canister stop` コマンドを実行することで、現在動作中の Canister を停止することができます。

    dfx canister stop --all

このコマンドは、次のような出力を表示します：

    Stopping code for canister status_check, with canister_id 75hes-oqbaa-aaaaa-aaaaa-aaaaa-aaaaa-aaaaa-q
    Stopping code for canister status_check_assets, with canister_id cxeji-wacaa-aaaaa-aaaaa-aaaaa-aaaaa-aaaaa-q

もし、`dfx canister status` コマンドを再実行すると、処理すべき保留中のメッセージがないことを示す `Stopped` というステータスや、処理すべきメッセージがあることを示す `Stopping` というステータスが表示されるはずです。

Canister を再起動するには、例えば Canister のアップグレードに成功した後、 `dfx canister start` コマンドを実行します。例えば、すべての Canister を再起動するには、次のコマンドを実行します：

    dfx canister start --all

このコマンドは、次のような出力を表示します：

    Starting code for canister status_check, with canister_id 75hes-oqbaa-aaaaa-aaaaa-aaaaa-aaaaa-aaaaa-q
    Starting code for canister status_check_assets, with canister_id cxeji-wacaa-aaaaa-aaaaa-aaaaa-aaaaa-aaaaa-q

## Canister のアップグレード

Canister の再インストールでは、Canister の識別子は保持されますが、ステートは保持されません。Canister のアップグレードでは、デプロイされた Canister のステートを保持し、コードを変更することができます。

たとえば、プロフェッショナルなプロフィールとソーシャルなつながりを管理する Dapp があるとします。このアプリに新しい機能を追加したい場合、以前に保存されたデータを失うことなく、Canister のコードを更新が可能である必要があります。Canister のアップグレードでは、プログラムのステートを失うことなく、プログラムの変更に伴って既存の Canister 識別子を更新することができます。

Motoko で記述された Canister をアップグレードする際にステートを保持するには、必ず `stable` キーワードを使用して、保持したい変数を特定します。Motoko で変数のステートを保持する方法については、[ステーブル（Stable）変数とアップグレード方法](../cdks/motoko-dfinity/upgrades.md) を参照してください。Rust で書かれた Canister をアップグレードする場合は、[Rust CDK asset storage](https://github.com/dfinity/cdk-rs/blob/master/examples/asset_storage/src/asset_storage_rs/lib.rs) の例にあるように `pre_upgrade` と `post_upgrade` 関数を使用して、Canister のアップグレード後にデータが適切に保存されるようにする必要があります。

Canister をアップグレードするには：

1.  新しいターミナルを開き、プロジェクトのディレクトリに移動します。

2.  必要に応じて、ローカル Canister の実行環境を起動します。

    ほとんどの場合、このステップは Canister をローカルで実行する場合のみ必要です。

    IC ブロックチェーンなどのリモート実行環境で実行する Canister をコンパイルする場合は、`--network` コマンドラインオプションを指定し、このパラメータで指定した環境上でタスクを実行するようにします。

3.  アップグレードするすべての Canister の Canister ID があることを確認します。

    変数宣言の中で `stable` キーワードを使って、ステートを保持する変数を特定する必要があることに注意してください。

    ステーブル変数宣言の詳細については、*Motoko Programming Language Guide* を参照してください。

4.  以下のコマンドを実行して、すべての Canister をアップグレードしてください：

        dfx canister install --all --mode upgrade

## Canister の削除

もし、特定のデプロイメント（ローカル、リモートのどちらか）の特定の Canister、または特定のプロジェクトのすべての Canister を永久に削除したい場合は、`dfx canister delete` コマンドを実行することで可能です。

 Canister を削除すると、 Canister の識別子、コード、およびステートが削除されます。ただし、Canister を削除する前に、まず Canister を停止して、保留中のメッセージ要求や返信をクリアする必要があります。

プロジェクトのすべてのキャニスタを削除するには：

1.  新しいターミナルを開き、プロジェクトのディレクトリに移動します。

2.  必要に応じて、ローカルの Canister 実行環境を起動します。

    ほとんどの場合、このステップは Canister をローカルで実行する場合のみ必要です。

    IC ブロックチェーンなどのリモート実行環境で実行する Canister をコンパイルする場合は、`--network` コマンドラインオプションを指定し、このパラメータで指定した環境上でタスクを実行するようにします。

3.  以下のコマンドを実行して、ローカル Canister 実行環境上で動作しているプロジェクト Canister のステートを確認します：

        dfx canister status --all

4.  以下のコマンドを実行して、すべてのプロジェクト Canister を停止します：

        dfx canister stop --all

5.  以下のコマンドを実行して、すべてのプロジェクト Canister を削除してください：

        dfx canister delete --all

<!--
# Managing Canisters

If you have experimented with using the SDK by following the tutorials in this section or by cloning examples from the [examples](https://github.com/dfinity/examples) repository, you are already familiar with how to build and deploy programs as **canisters**. This section provides additional information about the canister lifecycle and how to manage canisters.

## Obtaining a Canister Identifier

Depending on your preferred development workflow, you can obtain a unique identifier for your canister, before or after you have a program ready to compile. For example, if you want to reserve a unique identifier for your canister on a subnet before you have written any code, you can do so by running the `dfx canister create` command. This command essentially creates an empty canister placeholder into which you can later install your code. The resulting canister will obtain a unique identifier.

To obtain a unique identifier for a canister:

1.  Open a new terminal window or tab on your computer.

2.  Create a new project for the canister you plan to create by running a command similar to the following:

        `dfx new YOUR-PROJECT-NAME`

    Note that the name you use for the project is also used as the canister name by default.

3.  Change to your new project directory.

4.  Open the `dfx.json` configuration file and set the host and port for the canister execution environment you want to use (e.g. the IC blockchain).

    If you are using a local deployment, you can skip this step.

    You can also optionally change the names of your canisters or add canister settings to the configuration file if you want to create identifiers for any additional canisters you think you will need before compiling code.

5.  Start the local canister execution environment, if necessary, by running the following command:

        dfx start --background

    In most cases, this step is only necessary if you are running the canisters locally.

    If you were registering canisters to run on a remote execution environment, e.g. the IC blockchain, you would include the `--network` command-line option to perform tasks on the environment specified under this parameter.

6.  Register unique identifiers for the canisters defined in the `dfx.json` by running the following command:

        dfx canister create --all

    The command creates the `.dfx/local` directory and adds the `canister_ids.json` file to that directory for the project.

## Build a Canister with a Local Identifier

After you have written source code for your project, you need to compile it into a WebAssembly module before deploying it as a canister.

If you are only compiling your project for local debugging, you can generate a locally-defined identifier for your project.

To generate a locally-defined identifier:

1.  Create a project with the configuration settings and program logic to suit your needs.

2.  Start the local canister execution environment, if necessary.

    If you were compiling canisters to run on a remote execution environment, e.g. the IC blockchain, you would include the `--network` command-line option to perform tasks on the environment specified under this parameter.

3.  Generate hard-coded local identifiers for the canisters defined in the `dfx.json` by running the following command:

        dfx build --check

    Note that you must register unique canister identifiers to replace your locally-defined identifier before you can deploy the project on the IC blockchain.

## Deploy Canisters

After you have compiled a program, you can install the compiled code in a canister running either on a local canister execution environment or on the IC blockchain.

The canister identifier that was created in advance or during the build process determines where your code is installed during deployment.

To deploy the code for the first time:

1.  Open a new terminal and navigate to your project directory.

2.  Start the local canister execution environment, if necessary.

    In most cases, this step is only necessary if you are running the canisters locally.

    If you were registering canisters to run on a remote execution environment, e.g. the IC blockchain, you would include the `--network` command-line option to perform tasks on the environment specified under this parameter.

3.  Verify you have canister identifiers for all of the canisters you want to deploy.

4.  Deploy all of the canisters by running the following command:

        dfx canister install --all

## Look up a Canister ID

All canisters have unique identifiers. You often need to use these identifiers to interact with the canister. For example, if you want to access the frontend canister for a dapp or interact with a service using the Candid web interface, you must specify the appropriate canister identifier.

Because the identifiers are specific to the environment where the canisters are deployed, the files used to store the information are in different directories. For example, identifiers for a canister deployed locally are located in the project’s `.dfx/local/canister_ids.json` file.

You can look up the canister identifier for any specific canister by running the `dfx canister id` command. For example, to look up the the canister identifier for the `lookup` canister deployed on the local canister execution environment, you could run the following command:

    dfx canister id lookup

To look up the canister identifier for the same canister deployed on the environment specified by the `ic` alias, you would run the following command:

    dfx canister --network=ic id lookup

## Add a Wallet for Existing Canisters

When you are doing local development, creating a new project automatically creates a default wallet for you to use with the canisters in that project. If you want to add a wallet for projects with canisters that you have previously created, you can force `dfx` to generate one by taking a couple of manual steps.

To add a wallet for use with an existing canister:

1.  Open a terminal and navigate to your project directory.

2.  Stop the local canister execution environment, if necessary, by running the following command:

        dfx stop

3.  Delete the `.dfx` directory.

4.  Start the local canister execution environment network by running the following command:

        dfx start --clean

## Reinstall a Canister

During the development cycle, you might want to install, then replace your program as you debug and improve it.

In this scenario, you might want to keep the canister identifier you have registered but without preserving any of the canister code or state. For example, your canister might only have test data that you don’t want to keep or you might have decided to change the program altogether but want to reinstall under a canister identifier you used to install a previous program.

To reinstall a canister:

1.  Open a new terminal and navigate to your project directory.

2.  Start the local canister execution environment, if necessary.

    In most cases, this step is only necessary if you are running the canisters locally.

    If you were registering canisters to run on a remote execution environment, e.g. the IC blockchain, you would include the `--network` command-line option to perform tasks on the environment specified under this parameter.

3.  Verify you have canister identifiers for all of the canisters you want to re-deploy.

4.  Re-deploy all of the canisters by running the following command:

        dfx canister install --all --mode reinstall

Note that you can use the `reinstall` mode to replace any canister, regardless of whether the canister has code or state associated with it.

## Set an Identity to own a Canister

In most cases, a `default` user identity is created for you automatically the first time you run the `dfx canister create` command. This default identity consists of the public and private key pair generated for your local user account. Typically, this `default` identity is also the default owner of all of the projects you create and all of the canisters you deploy. You can, however, proactively create and use identities of your choice to circumvent the `default` user identity from being used.

As an example, the following scenario illustrates creating a `registered_owner` identity that is then used to register, build, deploy, and call the `pubs` project.

To set an identity for a project:

1.  Create a new project by running the following command:

        dfx new pubs

2.  Change to the project directory by running the following command:

        cd pubs

3.  Start the local canister execution environment in the background by running the following command:

        dfx start --background

4.  Create a new `registered_owner` identity by running the following command:

        dfx identity new registered_owner

5.  Set the active user context to use the `registered_owner` identity by running the following command:

        dfx identity use registered_owner

6.  Register, build, and deploy canisters for the project by running the following commands:

        dfx canister create --all
        dfx build --all
        dfx canister install --all

    These commands run using the `registered_owner` identity, making that user the owner of the canisters deployed.

7.  Call the `greet` function to verify a successful deployment by running the following command:

        dfx canister call pubs greet '("Sam")'

## Managing the Running State of a Canister

After you deploy a canister, it can begin receiving and processing requests from users and from other canisters. Canisters that are available to send requests and receive replies are considered in be in a **Running** state.

Although canisters are normally placed in the Running state by default, there are cases where you might want to temporarily or permanently stop a canister. For example, you might want to stop a canister before upgrading it. Stopping a canister helps to ensure proper handling of any messages that are in progress and need to either run to completion or be rolled back. You might also want to stop a canister to clear its message queue cleanly as a prerequisite to deleting the canister.

You can check the current status of all canisters or a specified canister by running the `dfx canister status` command. For example, to see the status for all canisters running on the local canister execution environment, you would run the following command:

    dfx canister status --all

This command returns output similar to the following if canisters are currently running:

    Canister status_check's status is Running.
    Canister status_check_assets's status is Running.

You can stop canisters that are currently running by running the `dfx canister stop` command.

    dfx canister stop --all

This command displays output similar to the following:

    Stopping code for canister status_check, with canister_id 75hes-oqbaa-aaaaa-aaaaa-aaaaa-aaaaa-aaaaa-q
    Stopping code for canister status_check_assets, with canister_id cxeji-wacaa-aaaaa-aaaaa-aaaaa-aaaaa-aaaaa-q

If you were to rerun the `dfx canister status` command, you might see a status of `Stopped` indicating that there were no pending messages that needed to processed or a status of `Stopping` indicating that there were messages in-flight that needed to be addressed.

To restart a canister-for example, after a successful canister upgrade—you can run the `dfx canister start` command. For example, to restart all of the canisters, you would run the following command:

    dfx canister start --all

This command displays output similar to the following:

    Starting code for canister status_check, with canister_id 75hes-oqbaa-aaaaa-aaaaa-aaaaa-aaaaa-aaaaa-q
    Starting code for canister status_check_assets, with canister_id cxeji-wacaa-aaaaa-aaaaa-aaaaa-aaaaa-aaaaa-q

## Upgrade a Canister

Unlike a canister reinstall that preserves the canister identifier but no state, a canister upgrade enables you to preserve the state of a deployed canister, and change the code.

For example, assume you have a dapp that manages professional profiles and social connections. If you want to add a new feature to the dapp, you need to be able to update the canister code without losing any of the previously-stored data. A canister upgrade enables you to update existing canister identifiers with program changes without losing the program state.

To preserve state when you are upgrading a canister written in Motoko, be sure to use the `stable` keyword to identify the variables you want to preserve. For more information about preserving variable state in Motoko, see [Stable variables and upgrade methods](../cdks/motoko-dfinity/upgrades.md). If you are upgrading a canister written in Rust, you should use `pre_upgrade` and `post_upgrade` functions as illustrated in the [Rust CDK asset storage](https://github.com/dfinity/cdk-rs/blob/master/examples/asset_storage/src/asset_storage_rs/lib.rs) example to ensure data is properly preserved after a canister upgrade.

To upgrade a canister:

1.  Open a new terminal and navigate to your project directory.

2.  Start the local canister execution environment, if necessary.

    In most cases, this step is only necessary if you are running the canisters locally.

    If you were registering canisters to run on a remote execution environment, e.g. the IC blockchain, you would include the `--network` command-line option to perform tasks on the environment specified under this parameter.

3.  Verify you have canister identifiers for all of the canisters you want to upgrade.

    Note that your program must identify the variables for which to maintain state by using the `stable` keyword in the variable declaration.

    For more information about declaring stable variables, see the *Motoko Programming Language Guide*.

4.  Upgrade all of the canisters by running the following command:

        dfx canister install --all --mode upgrade

## Delete a Canister

If you want to permanently delete a specific canister or all canisters for a specific project on a given deployment (either local, or remote), you can do so by running the `dfx canister delete` command.

Deleting a canister removes the canister identifier, code, and state. Before you can delete a canister, however, you must first stop the canister to clear any pending message requests or replies.

To delete all canisters for a project:

1.  Open a new terminal and navigate to your project directory.

2.  Start the local canister execution environment, if necessary.

    In most cases, this step is only necessary if you are running the canisters locally.

    If you were deleting canisters to run on a remote execution environment, e.g. the IC blockchain, you would include the `--network` command-line option to perform tasks on the environment specified under this parameter.

3.  Check the status of the project canisters running on the local canister execution environment by running the following command:

        dfx canister status --all

4.  Stop all of the project canisters by running the following command:

        dfx canister stop --all

5.  Delete all of the project canisters by running the following command:

        dfx canister delete --all

-->