# Identity によるアクセスコントロールの追加

Dapps では、ユーザーごとに異なる操作を制御するために、ロールベースのパーミッションが必要になることがあります。

このチュートリアルでは、ユーザー Identity の作成と切り替えを説明するために、異なるロールに割り当てられたユーザーに対して異なる挨拶文を表示するシンプルな Dapp を作成します。

この例では、`owner`、`admin`、`authorized` の3つのロールが指定されています。

-   `admin` ロールが割り当てられているユーザーには、`You have a role with administrative privileges` という挨拶文が表示されます。

-   `Authorized`ロールが割り当てられているユーザーには、`Would you like to play a game?` という挨拶文が表示されます。

-   これらのロールが割り当てられていないユーザーには、`Nice to meet you!` という挨拶文が表示されます。

さらに、Canister を初期化したユーザー Identity にのみ `owner` ロールが割り当てられ、 `owner` と `admin` ロールのみが他のユーザーにロールを割り当てることができます。

大まかにいうと、各ユーザーは公開鍵と秘密鍵のペアを持ちます。ユーザーがアクセスする Canister ID に紐付いた公開鍵がセキュリティ Principal となり、これをメッセージ呼び出し元として使用して、Internet Computer ブロックチェーン上で動作する Canister への関数コールを認証することができます。次の図は、ユーザー Identity がメッセージ呼び出し元を認証する方法を簡略化して示しています。

![principal identities](../_attachments/principal-identities.svg)

## 始める前に

チュートリアルを始める前に、以下を確認してください：

-   [ダウンロードとインストール](../../quickstart/local-quickstart#download-and-install) で説明されているように、SDK パッケージをダウンロードしインストールする。

-   デフォルトのユーザー Identity が作成されるようなコマンドを少なくとも 1 つ実行する。デフォルトのユーザー Identity は、すべてのプロジェクトのために `$HOME/.config/dfx/identity/` ディレクトリにグローバルに保存されます。

-   Visual Studio Code を IDE として使用している場合、[言語エディタプラグインのインストール](../../quickstart/local-quickstart#install-vscode) にあるように、Motoko 用の Visual Studio Code プラグインをインストールする。

-   コンピュータ上で動作しているローカル Canister 実行環境プロセスを停止する。

## 新しいプロジェクトを作成する

アクセスコントロールとユーザー Identity の切り替えをテストするために、新しいプロジェクトディレクトリを作成します：

1.  ローカルコンピューターでターミナルシェルを開きます（まだ開いていない場合）。

2.  Internet Computer ブロックチェーンのプロジェクト用に使用しているフォルダがある場合にはそちらに移動します。

3.  以下のコマンドを実行し、新しいプロジェクトを作成します：

        dfx new access_hello

4.  次のコマンドを実行して、プロジェクトディレクトリに移動します：

        cd access_hello

## デフォルトの Dapp を変更する

このチュートリアルでは、テンプレートのソースコード・ファイルを、ロールの割り当てと取得のための機能を持つ Dapp に置き換えます。

デフォルトの Dapp を変更するには、次のようにします：

1.  `src/access_hello/main.mo` ファイルをテキストエディタで開き、既存のコンテンツを削除します。

2.  [このコード](../_attachments/access-control-main.mo)をコピーして、ファイルに貼り付けます。

    この Dapp の主要な要素をいくつか見てみましょう：

    -   これまでのチュートリアルで見てきた `greet` 関数のバリエーションであることに気がつくかもしれません。

        しかし、このアプリでは、`greet` 関数がメッセージの呼び出し元を利用して、適用すべきパーミッションを決定し、呼び出し元に関連するパーミッションに基づいて、どの挨拶文を表示すべきかを決定します。

    -   1つは `Roles` 用、もう1つは `Permissions` 用のカスタムタイプです。

    -   `assign_role` 関数を使用すると、メッセージの呼び出し元が Identity に関連付けられた Principal にロールを割り当てることができます。

    -   `callerPrincipal` 関数を使用すると、Identity に関連付けられた Principal を返せるようになります。

    -   `my_role` 関数を使用すると、Identity に関連付けられたロールを返すことができます。

3.  変更を保存して、`main.mo` ファイルを閉じてください。

## ローカル Canister の実行環境を起動する

`access_hello` プロジェクトをビルドする前に、開発環境で動作しているローカル Canister 実行環境、または Internet Computer のブロックチェーンメインネットに接続する必要があります。

ローカル Canister 実行環境を起動するには：

1.  ローカルコンピューターで新しいターミナルウィンドウまたはタブを開きます。

2.  必要に応じて、プロジェクトのルートディレクトリに移動します。

3.  コンピューター上で以下のコマンドを実行し、ローカルの Canister 実行環境を起動します：

        dfx start --background

    ローカル Canister の実行環境が起動操作を完了したら、次のステップに進みます。

## Dapp の登録、ビルド、デプロイ

開発環境で動作しているローカル Canister 実行環境に接続したら、[`dfx deploy`](../../../references/cli-reference/dfx-deploy) コマンドを実行して、一度にアプリの登録、ビルド、およびデプロイを行うことができます。また、[`dfx canister create`](../../../references/cli-reference/dfx-canister#dfx_canister_create)、 [`dfx build`](../../../references/cli-reference/dfx-build) と [`dfx canister install`](../../../references/cli-reference/dfx-canister#dfx_canister_install) コマンドでこれらの手順を個別に行うことが可能です。

Dapp をローカルにデプロイするには：

1.  必要に応じて、まだプロジェクトのルートディレクトリにいることを確認します。

2.  以下のコマンドを実行して、`access_hello` バックエンド Dapp を登録、ビルド、デプロイします：

        dfx deploy access_hello

        Creating a wallet canister on the local network.
        The wallet canister on the "local" network for user "default" is "rwlgt-iiaaa-aaaaa-aaaaa-cai"
        Deploying: access_hello
        Creating canisters...
        Creating canister "access_hello"...
        "access_hello" canister created with canister id: "rrkah-fqaaa-aaaaa-aaaaq-cai"
        Building canisters...
        Installing canisters...
        Installing code for canister access_hello, with canister_id rrkah-fqaaa-aaaaa-aaaaq-cai
        Deployed canisters.

## 現在の Identity コンテキストを確認する

追加の Identity を作成する前に、`default` Identity に関連する Principal 識別子と `default` Identity の Cycle ウォレットを確認しましょう。Internet Computer ブロックチェーンでは、 Principal はユーザー、Canister、ノード、サブネットを識別するための内部的な代表値です。 Principal のテキスト表現は、 Principal データタイプで作業しているときに表示される外部識別子です。

現在の Identity と Principal を確認するには：

1.  以下のコマンドを実行し、現在アクティブな Identity を確認します：

        dfx identity whoami

    このコマンドは、次のような出力を表示します：

        default

2.  以下のコマンドを実行して、`default` ユーザー Identity の Principal を確認します：

        dfx identity get-principal

    このコマンドは、次のような出力を表示します：

        zen7w-sjxmx-jcslx-ey4hf-rfxdq-l4soz-7ie3o-hti3o-nyoma-nrkwa-cqe

3.  以下のコマンドを実行して、`default` ユーザー Identity に関連するロールを確認します：

        dfx canister --wallet=$(dfx identity get-wallet) call access_hello my_role

    このコマンドは、次のような出力を表示します：

        (opt variant { owner })

## 新しいユーザー Identity を作成する

Dapp のアクセスコントロールのテストを始めるために、いくつかの新しいユーザー Identity を作成し、それらのユーザーに異なるロールを割り当ててみましょう。

新しいユーザー Identity を作成するには、次のようにします：

1.  必要であれば、プロジェクトディレクトリに留まっていることを確認します。

2.  以下のコマンドを実行して、新しい管理者ユーザー Identity を作成します：

        dfx identity new ic_admin

    このコマンドは、次のような出力を表示します：

        Creating identity: "ic_admin".
        Created identity: "ic_admin".

3.  新しいユーザー Identity がどのロールにも割り当てられていないことを確認するために、`my_role` 関数を呼び出してください。

        dfx --identity ic_admin canister call access_hello my_role

    このコマンドは、次のような出力を表示します：

        Creating a wallet canister on the local network.
        The wallet canister on the "local" network for user "ic_admin" is "ryjl3-tyaaa-aaaaa-aaaba-cai"
        (null)

4.  次のコマンドを実行して、現在アクティブな Identity コンテキストを新しい `ic_admin` ユーザー Identity を使用するように切り替え、`ic_admin` ユーザーに関連付けられた Principal を表示します：

        dfx identity use ic_admin && dfx identity get-principal

    このコマンドは、次のような出力を表示します：

        Using identity: "ic_admin".
        c5wa6-3irl7-tuxuo-4vtyw-xsnhw-rv2a6-vcmdz-bzkca-vejmd-327zo-wae

5.  以下のコマンドを実行して、`access_hello`  Canister の呼び出しに使用された Principal を確認します：

        dfx canister call access_hello callerPrincipal

    このコマンドは、次のような出力を表示します：

        (principal "ryjl3-tyaaa-aaaaa-aaaba-cai")

    デフォルトでは、`access_hello`  Canister のメソッドを呼び出すために使用される Principal は、Cycle ウォレットの識別子になります。しかし、アクセスコントロールを説明するために、Cycle ウォレットではなく、ユーザーコンテキストに関連付けられた Principal を使用したいと思います。しかし、その前に `ic_admin` ユーザーにロールを割り当ててみましょう。そのためには、`owner` ロールを持つ `default` ユーザー Identity に切り替える必要があります。

## Identity にロールを割り当てる

IC_admin の Identity に  admin ロールを割り当てるには：

1.  次のコマンドを実行して、現在アクティブな Identity コンテキストを切り替えて、`default` ユーザ Identity を使用するようにします：

        dfx identity use default

2.  `IC_admin`  Principal に `admin` ロールを割り当てるには、Candid 構文で次のようなコマンドを実行します：

        dfx canister --wallet=$(dfx identity get-wallet) call access_hello assign_role '((principal "c5wa6-3irl7-tuxuo-4vtyw-xsnhw-rv2a6-vcmdz-bzkca-vejmd-327zo-wae"),opt variant{admin})'

`Principal` ハッシュは、 `ic_admin` Identity に対して `dfx identity get-principal` コマンドで返されたものに置き換える必要があります。

\+ オプションで、コマンドを再実行して `my_role` 関数を呼び出し、ロールの割り当てを確認することができます。

\+

    dfx --identity ic_admin canister call access_hello my_role

\+ このコマンドは、次のような出力を表示します：

\+

    (opt variant { admin })

1.  先ほど `admin` ロールを割り当てた `ic_admin` ユーザーの Identity を使用して、以下のコマンドを実行して `greet` 関数を呼び出してください：

        dfx --identity ic_admin canister call access_hello greet "Internet Computer Admin"

    このコマンドは、次のような出力を表示します：

        (
          "Hello, Internet Computer Admin. You have a role with administrative privileges.",
        )

## オーソライズされたユーザー Identity の追加

この時点では、`owner` ロールを持つ `default` ユーザー Identity と `admin` ロールを持つ `ic_admin` ユーザー Identity が存在しています。もうひとつのユーザー Identity を追加して、`authorized` ロールに割り当ててみましょう。ただし、この例では、ユーザーの Principal を保存するために環境変数を使用します。

新しいオーソライズされたユーザー Identity を追加するには、次のようにします：

1.  必要であれば、プロジェクトディレクトリに留まっていることを確認します。

2.  次のコマンドを実行して、新しい正規ユーザー Identity を作成します：

        dfx identity new alice_auth

    このコマンドは、次のような出力を表示します：

        Creating identity: "alice_auth".
        Created identity: "alice_auth".

3.  以下のコマンドを実行して、現在アクティブな Identity コンテキストを切り替えて、新しい `alice_auth` ユーザー Identity を使用するようにします：

        dfx identity use alice_auth

4.  以下のコマンドを実行して、`alice_auth` ユーザーの Principal を環境変数に格納します：

        ALICE_ID=$(dfx identity get-principal)

    以下のコマンドを実行することで、保存されている Principal を確認することができます：

        echo $ALICE_ID

    このコマンドは、次のような出力を表示します：

        b5quc-npdph-l6qp4-kur4u-oxljq-7uddl-vfdo6-x2uo5-6y4a6-4pt6v-7qe

5.  `ic_admin` の Identity を使用して、以下のコマンドを実行して `alice_auth` に `authorized` ロールを割り当ててください：

        dfx --identity ic_admin canister call access_hello assign_role "(principal \"$ALICE_ID\", opt variant{authorized})"

6.  ロールの割り当てを確認するために `my_role` 関数を呼び出します。

        dfx --identity alice_auth canister call access_hello my_role

    このコマンドは、次のような出力を表示します：

        (opt variant { authorized })

7.  先ほど `authorized` ロールを割り当てた `alice_auth` ユーザーの Identity を使用して、以下のコマンドを実行して `greet` 関数を呼び出してください：

        dfx canister call access_hello greet "Alice"

    このコマンドは、次のような出力を表示します：

        (
          "Welcome, Alice. You have an authorized account. Would you like to play a game?",
        )

## 未承認のユーザー Identity を追加する

ここまでで、特定のロールと権限を持つユーザーを作成する簡単な例を見ました。次のステップでは、ロールが割り当てられていない、あるいは特別なパーミッションが与えられていないユーザー Identity  を作成します。

未承認のユーザー Identity  を追加するには、以下の手順に従います：

1.  必要であれば、プロジェクトディレクトリに留まっていることを確認します。

2.  必要に応じて、以下のコマンドを実行して、現在アクティブな Identity  を確認します。

        dfx identity whoami

3.  以下のコマンドを実行して、新しいユーザーIdentity を作成します：

        dfx identity new bob_standard

    このコマンドは、次のような出力を表示します：

        Creating identity: "bob_standard".
        Created identity: "bob_standard".

4.  以下のコマンドを実行して、`bob_standard` ユーザの Principal を環境変数に格納します：

        BOB_ID=$(dfx --identity bob_standard identity get-principal)

5.  ロールを割り当てるために `bob_standard` Identity を使用しようとしてみてください。

        dfx --identity bob_standard canister call access_hello assign_role "(principal \"$BOB_ID\", opt variant{authorized})"

    のコマンドは `unauthorized` エラーを返します。

6.  以下のコマンドを実行して、`default` ユーザー Identity を使用して `bob_standard` に `owner` ロールを割り当ててみてください。

        dfx --identity default canister --wallet=$(dfx --identity default identity get-wallet) call access_hello assign_role "(principal \"$BOB_ID\", opt variant{owner})"

    このコマンドは、ユーザーに `owner` ロールを割り当てることができないため失敗します。

7.  次のコマンドを実行して、`bob_standard` ユーザー Identity を使用して `greet` 関数を呼び出します：

        dfx --identity bob_standard canister call access_hello greet "Bob"

    このコマンドは、次のような出力を表示します：

        ("Greetings, Bob. Nice to meet you!")

## 複数のコマンドでユーザー Identity を設定する

これまで、個々のコマンドでユーザー Identity を作成し、切り替える方法について見てきました。また、使用したいユーザー Identity を指定し、そのユーザー Identity のコンテキストで複数のコマンドを実行することもできます。

1つのユーザー Identity で複数のコマンドを実行するには：

1.  必要であれば、プロジェクトディレクトリに留まっていることを確認します。

2.  以下のコマンドを実行して、現在利用可能なユーザー Identity を一覧表示します：

        dfx identity list

    のコマンドは、現在アクティブなユーザー Identity を示すアスタリスクとともに、次のような出力を表示します。

        alice_auth
        bob_standard
        default *
        ic_admin

    この例では、明示的に別のIdentity を選択しない限り、`default` のユーザー Identity が使用されます。

3.  リストから新しいユーザー Identity  を選択し、次のようなコマンドを実行して、それをアクティブユーザーコンテキストにします：

        dfx identity use ic_admin

    \+ このコマンドは、次のような出力を表示します：

        Using identity: "ic_admin".

    `dfx identity list` コマンドを再実行すると、`ic_admin` ユーザー Identity  は現在アクティブなユーザコンテキストであることを示すためにアスタリスクを表示します。

    コマンドラインで `--identity` を指定しなくても、選択したユーザーIdentity を使用してコマンドを実行できるようになりました。

## ローカル Canister の実行環境を停止する

Dapp のテストや Identity の利用が終わったら、ローカル Canister 実行環境を停止して、バックグラウンドで実行し続けないようにします。

ローカル Canister の実行環境を停止するには、以下の手順で行います：

1.  ネットワーク操作が表示されているターミナルで、Control-C キーを押して、ローカルネットワークの処理を中断します。

2.  以下のコマンドを実行して、ローカル Canister の実行環境を停止します：

        dfx stop

## もっと学ぶには？

Indentity と認証についてより詳しい情報をお求めの方は、以下の関連資料をご覧ください：

-   [dfx identity (command reference)](../../../references/cli-reference/dfx-identity)

<!-- -   [Set an identity to own a canister (how-to)](../../working-with-canisters#set-owner) -->

<!--
# Add access control with identities

Dapps often require role-based permissions to control the operations different users can perform.

To illustrate how to create and switch between user identities, this tutorial creates a simple dapp that displays a different greeting for users who are assigned to different roles.

In this example, there are three named roles—`owner`, `admin`, and `authorized`.

-   Users who are assigned an `admin` role see a greeting that displays `You have a role with administrative privileges`.

-   Users who are assigned an `authorized` role see a greeting that displays `Would you like to play a game?`.

-   Users who are not assigned one of these roles see a greeting that displays `Nice to meet you!`.

In addition, only the user identity that initialized the canister is assigned the `owner` role and only the `owner` and `admin` roles can assign roles to other users.

At a high-level, each user has a public/private key pair. The public key combined with the canister identifier the user accesses forms a security principal that can then be used as a message caller to authenticate function calls made to the canister running on the Internet Computer blockchain. The following diagram provides a simplified view of how user identities authenticate message callers.

![principal identities](../_attachments/principal-identities.svg)

## Before you begin

Before starting the tutorial, verify the following:

-   You have downloaded and installed the SDK package as described in [Download and install](../../quickstart/local-quickstart#download-and-install).

-   You have run at least one command that resulted in your `default` user identity being created. Your default user identity is stored globally for all projects in the `$HOME/.config/dfx/identity/` directory.

-   You have installed the Visual Studio Code plugin for Motoko as described in [Install the language editor plug-in](../../quickstart/local-quickstart#install-vscode) if you are using Visual Studio Code as your IDE.

-   You have stopped any local canister execution environment processes running on your computer.

## Create a new project

To create a new project directory for testing access control and switching user identities:

1.  Open a terminal shell on your local computer, if you don’t already have one open.

2.  Change to the folder you are using for your Internet Computer blockchain projects, if you are using one.

3.  Create a new project by running the following command:

        dfx new access_hello

4.  Change to your project directory by running the following command:

        cd access_hello

## Modify the default dapp

For this tutorial, you are going to replace the template source code file with a dapp that has functions for assigning and retrieving roles.

To modify the default dapp:

1.  Open the `src/access_hello/main.mo` file in a text editor and delete the existing content.

2.  Copy and paste [this code](../_attachments/access-control-main.mo) into the file.

    Let's take a look at a few key elements of this dapp:

    -   You might notice that the `greet` function is a variation on the `greet` function you have seen in previous tutorials.

        In this dapp, however, the `greet` function uses a message caller to determine the permissions that should be applied and, based on the permissions associated with the caller, which greeting to display.

    -   The dapp defines two custom types—one for `Roles` and one for `Permissions`.

    -   The `assign_roles` function enables the message caller to assign a role to the principal associated with an identity.

    -   The `callerPrincipal` function enables you to return the principal associated with an identity.

    -   The `my_role` function enables you to return the role that is associated with an identity.

3.  Save your changes and close the `main.mo` file to continue.

## Start the local canister execution environment

Before you can build the `access_hello` project, you need to connect to the local canister execution environment running in your development environment or to the Internet Computer blockchain mainnet.

To start the local canister execution environment:

1.  Open a new terminal window or tab on your local computer.

2.  Navigate to the root directory for your project, if necessary.

3.  Start the local canister execution environment on your computer by running the following command:

        dfx start --background

    After the local canister execution environment completes its startup operations, you can continue to the next step.

## Register, build, and deploy the dapp

After you connect to the local canister execution environment running in your development environment, you can register, build, and deploy your dapp in a single step by running the [`dfx deploy`](../../../references/cli-reference/dfx-deploy) command. You can also perform each of these steps independently using separate [`dfx canister create`](../../../references/cli-reference/dfx-canister#dfx_canister_create), [`dfx build`](../../../references/cli-reference/dfx-build), and [`dfx canister install`](../../../references/cli-reference/dfx-canister#dfx_canister_install) commands.

To deploy the dapp locally:

1.  Check that you are still in the root directory for your project, if needed.

2.  Register, build, and deploy the `access_hello` backend dapp by running the following command:

        dfx deploy access_hello

        Creating a wallet canister on the local network.
        The wallet canister on the "local" network for user "default" is "rwlgt-iiaaa-aaaaa-aaaaa-cai"
        Deploying: access_hello
        Creating canisters...
        Creating canister "access_hello"...
        "access_hello" canister created with canister id: "rrkah-fqaaa-aaaaa-aaaaq-cai"
        Building canisters...
        Installing canisters...
        Installing code for canister access_hello, with canister_id rrkah-fqaaa-aaaaa-aaaaq-cai
        Deployed canisters.

## Check the current identity context

Before we create any additional identities, let’s review the principal identifiers associated with your `default` identity and the cycles wallet for your `default` identity. On the Internet Computer blockchain, a principal is the internal representative for a user, canister, node, or subnet. The textual representation for a principal is the external identifier you see displayed with working with the principal data type.

To review your current identity and principle:

1.  Verify the currently-active identity by running the following command:

        dfx identity whoami

    The command displays output similar to the following:

        default

2.  Check the principal for the `default` user identity by running the following command:

        dfx identity get-principal

    The command displays output similar to the following:

        zen7w-sjxmx-jcslx-ey4hf-rfxdq-l4soz-7ie3o-hti3o-nyoma-nrkwa-cqe

3.  Check the role associated with the `default` user identity by running the following command:

        dfx canister --wallet=$(dfx identity get-wallet) call access_hello my_role

    The command displays output similar to the following:

        (opt variant { owner })

## Create a new user identity

To begin testing the access controls in our dapp, let’s create some new user identities and assign those users to different roles.

To create a new user identity:

1.  Check that you are still in your project directory, if needed.

2.  Create a new administrative user identity by running the following command:

        dfx identity new ic_admin

    The command displays output similar to the following:

        Creating identity: "ic_admin".
        Created identity: "ic_admin".

3.  Call the `my_role` function to see that your new user identity has not been assigned to any role.

        dfx --identity ic_admin canister call access_hello my_role

    The command displays output similar to the following:

        Creating a wallet canister on the local network.
        The wallet canister on the "local" network for user "ic_admin" is "ryjl3-tyaaa-aaaaa-aaaba-cai"
        (null)

4.  Switch your currently-active identity context to use the new `ic_admin` user identity and display the principal associated with the `ic_admin` user by running the following command:

        dfx identity use ic_admin && dfx identity get-principal

    The command displays output similar to the following:

        Using identity: "ic_admin".
        c5wa6-3irl7-tuxuo-4vtyw-xsnhw-rv2a6-vcmdz-bzkca-vejmd-327zo-wae

5.  Check the principal used to call the `access_hello` canister by running the following command:

        dfx canister call access_hello callerPrincipal

    The command displays output similar to the following:

        (principal "ryjl3-tyaaa-aaaaa-aaaba-cai")

    By default, the cycles wallet identifier is the principal used to call the methods in the `access_hello` canister. To illustrate access control, however, we want to use the principal associated with the user context, not the cycles wallet. Before we get to that step, though, let’s assign a role to the `ic_admin` user. To do that, we need to switch to the `default` user identity that has the `owner` role.

## Assign a role to an identity

To assign the admin role to the IC\_admin identity:

1.  Switch your currently-active identity context to use the `default` user identity by running the following command:

        dfx identity use default

2.  Assign the `ic_admin` principal the `admin` role by running a command similar to the following using Candid syntax:

        dfx canister --wallet=$(dfx identity get-wallet) call access_hello assign_role '((principal "c5wa6-3irl7-tuxuo-4vtyw-xsnhw-rv2a6-vcmdz-bzkca-vejmd-327zo-wae"),opt variant{admin})'

Be sure to replace the `principal` hash with the one returned by the `dfx identity get-principal` command for the `ic_admin` identity.

\+ Optionally, you can rerun the command to call the `my_role` function to verify the role assignment.

\+

    dfx --identity ic_admin canister call access_hello my_role

\+ The command displays output similar to the following:

\+

    (opt variant { admin })

1.  Call the `greet` function using the `ic_admin` user identity that you just assigned the `admin` role by running the following command:

        dfx --identity ic_admin canister call access_hello greet "Internet Computer Admin"

    The command displays output similar to the following:

        (
          "Hello, Internet Computer Admin. You have a role with administrative privileges.",
        )

## Add an authorized user identity

At this point, you have a `default` user identity with the `owner` role and an `ic_admin` user identity with the `admin` role. Let’s add another user identity and assign it to the `authorized` role. For this example, however, we’ll use an environment variable to store the user’s principal.

To add a new authorized user identity:

1.  Check that you are still in your project directory, if needed.

2.  Create a new authorized user identity by running the following command:

        dfx identity new alice_auth

    The command displays output similar to the following:

        Creating identity: "alice_auth".
        Created identity: "alice_auth".

3.  Switch your currently-active identity context to use the new `alice_auth` user identity by running the following command:

        dfx identity use alice_auth

4.  Store the principal for the `alice_auth` user in an environment variable by running the following command:

        ALICE_ID=$(dfx identity get-principal)

    You can verify the principal stored by running the following command:

        echo $ALICE_ID

    The command displays output similar to the following:

        b5quc-npdph-l6qp4-kur4u-oxljq-7uddl-vfdo6-x2uo5-6y4a6-4pt6v-7qe

5.  Use the `ic_admin` identity to assign the `authorized` role to `alice_auth` by running the following command:

        dfx --identity ic_admin canister call access_hello assign_role "(principal \"$ALICE_ID\", opt variant{authorized})"

6.  Call the `my_role` function to verify the role assignment.

        dfx --identity alice_auth canister call access_hello my_role

    The command displays output similar to the following:

        (opt variant { authorized })

7.  Call the `greet` function using the `alice_auth` user identity that you just assigned the `authorized` role by running the following command:

        dfx canister call access_hello greet "Alice"

    The command displays output similar to the following:

        (
          "Welcome, Alice. You have an authorized account. Would you like to play a game?",
        )

## Add an unauthorized user identity

You have now seen a simple example of creating users with specific roles and permissions. The next step is to create a user identity that is not assigned to a role or given any special permissions.

To add an unauthorized user identity:

1.  Check that you are still in your project directory, if needed.

2.  Check your currently-active identity, if needed, by running the following command:

        dfx identity whoami

3.  Create a new user identity by running the following command:

        dfx identity new bob_standard

    The command displays output similar to the following:

        Creating identity: "bob_standard".
        Created identity: "bob_standard".

4.  Store the principal for the `bob_standard` user in an environment variable by running the following command:

        BOB_ID=$(dfx --identity bob_standard identity get-principal)

5.  Attempt to use the `bob_standard` identity to assign a role.

        dfx --identity bob_standard canister call access_hello assign_role "(principal \"$BOB_ID\", opt variant{authorized})"

    This command returns an `unauthorized` error.

6.  Attempt to use the `default` user identity to assign `bob_standard` the `owner` role by running the following command:

        dfx --identity default canister --wallet=$(dfx --identity default identity get-wallet) call access_hello assign_role "(principal \"$BOB_ID\", opt variant{owner})"

    This command fails because users cannot be assigned the `owner` role.

7.  Call the `greet` function using the `bob_standard` user identity by running the following command:

        dfx --identity bob_standard canister call access_hello greet "Bob"

    The command displays output similar to the following:

        ("Greetings, Bob. Nice to meet you!")

## Set the user identity for multiple commands

So far, you have seen how to create and switch between user identities for individual commands. You can also specify a user identity you want to use, then run multiple commands in the context of that user identity.

To run multiple commands under one user identity:

1.  Check that you are still in your project directory, if needed.

2.  List the user identities currently available by running the following command:

        dfx identity list

    The command displays output similar to the following with an asterisk indicating the currently-active user identity.

        alice_auth
        bob_standard
        default *
        ic_admin

    In this example, the `default` user identity is used unless you explicitly select a different identity.

3.  Select a new user identity from the list and make it the active user context by running a command similar to the following:

        dfx identity use ic_admin

    \+ The command displays output similar to the following:

        Using identity: "ic_admin".

    If you rerun the `dfx identity list` command, the `ic_admin` user identity displays an asterisk to indicate it is the currently active user context.

    You can now run commands using the selected user identity without specifying `--identity` on the command-line.

## Stop the local canister execution environment

After you finish experimenting with the dapp and using identities, you can stop the local canister execution environment so that it doesn’t continue running in the background.

To stop the local canister execution environment:

1.  In the terminal that displays network operations, press Control-C to interrupt the local network process.

2.  Stop the local canister execution environment by running the following command:

        dfx stop

## Want to learn more?

If you are looking for more information about identity and authentication, check out the following related resources:

-   [dfx identity (command reference)](../../../references/cli-reference/dfx-identity)

<!-- -   [Set an identity to own a canister (how-to)](../../working-with-canisters#set-owner) -->
<!--756行目のーー＞でコメントアウト終了
-->
