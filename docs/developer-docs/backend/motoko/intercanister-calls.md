# Canister 間コールを行う

 Internet Computer ブロックチェーンは、ある Canister の機能を別の Canister から呼び出せることが開発者にとって最も重要な機能のひとつです。この Canister 間の呼び出し機能（** Canister 間コール**とも呼ばれる）は、複数の Dapps で機能を再利用・共有することを可能にします。

例えば、仕事上の人脈づくりや、地域のイベント、募金活動などを行うためのアプリを作成したいとします。それぞれの Dapps には、友人や家族、同僚など、何らかの基準や共通の関心に基づいてソーシャルな関係を特定するためのソーシャルコンポーネントが含まれるかもしれません。

このようなソーシャルコンポーネントに対応するために、ユーザーとの関係を保存する Canister を1つ作成し、プロフェッショナルネットワーキング、コミュニティ主催、資金調達などのアプリケーションを作成して、 Canister に定義されているソーシャルコネクション用の関数をインポートして呼び出すとよいでしょう。その後、ソーシャルコネクション Canister を使用するアプリケーションを追加で作成したり、ソーシャルコネクション Canister が提供する機能を拡張して、他の開発者のより広いコミュニティで使用できます。

Motoko ベースの LinkedUp サンプル Dapp は、オープンプロフェッショナルネットワークの簡単な実装を提供し、プロジェクト内での Canister 間コールの使用方法を示してくれます。

LinkedUp サンプルアプリは、以下の Canister を使用して実装されています：

-   `linkedup`  Canister は、ユーザーの職歴や学歴などの基本プロファイル情報を作成し、保存します。

-   `connectd`  Canister は、ユーザーの接続を作成し、保存します。

-   `linkedup_assets` Canister は、ユーザーインターフェイスを定義する JavaScript、HTML、CSS ファイルを含むフロントエンドアセットを格納します。

## 始める前に

サンプル Dapp をビルドする前に、以下を確認してください：

-   [ダウンロードとインストール](../../quickstart/local-quickstart#download-and-install)で説明されているように、SDK パッケージをダウンロードしインストールする。

-   `dfx`が提供するローカル Canister の実行環境を停止する。

## デモをダウンロードする

LinkedUp サンプル Dapp を使って、 Canister 間コールをテストするには：

1.  ターミナルシェルを開き、Internet Computer のサンプルプロジェクトに使用しているフォルダに移動します。

2.  `linkedup` レポジトリをクローンします。

        git clone https://github.com/dfinity/linkedup.git

3.  `linkedup` リポジトリのローカルワーキングディレクトリに移動します。

        cd linkedup

4.  以下のコマンドを実行して、node モジュールをインストールします。

        npm install

    必要に応じて、以下のコマンドを実行し、見つかった脆弱性を修正します。

        npm audit fix

5.  テキストエディタで `dfx.json` ファイルを開き、`dfx` の設定が、インストールした `dfx` 実行ファイルと同じバージョン番号になっていることを確認します。

## ローカル Canister の実行環境を起動する

開発用に `dfx` はローカル Canister の実行環境を提供します。これは `dfx.json` ファイルを必要とするので、linkedup のルートディレクトリにいることを確認する必要があります。

ローカル Canister の実行環境を起動するには：

1.  ローカルコンピューターで新しいターミナルウィンドウまたはタブを開きます。

2.  必要に応じて、プロジェクトのルートディレクトリに移動します。

3.  コンピューター上で以下のコマンドを実行し、ローカルの Canister 実行環境を起動します：

        dfx start --background

    ローカル Canister の実行環境が起動操作を完了したら、次のステップに進みます。

## Canister 識別子を登録する

ローカル Canister 実行環境が稼働したら、プロジェクトに固有の Canister 識別子（ID）を生成できます。

 Canister 識別子を登録するには：

1.  必要であれば、プロジェクトディレクトリに留まっていることを確認します。

2.  以下のコマンドを実行して、プロジェクトに固有の Canister 識別子を登録します：

        dfx canister create --all

    このコマンドは、`dfx.json` 設定ファイルに定義されている Canister の識別子を表示します。

        "connectd" canister created with canister id: "75hes-oqbaa-aaaaa-aaaaa-aaaaa-aaaaa-aaaaa-q"
        "linkedup" canister created with canister id: "cxeji-wacaa-aaaaa-aaaaa-aaaaa-aaaaa-aaaaa-q"
        "linkedup_assets" canister created with canister id: "7kncf-oidaa-aaaaa-aaaaa-aaaaa-aaaaa-aaaaa-q"

    ローカル Canister 実行環境を実行しているため、これらの識別子はローカル Canister 実行環境でのみ有効であることに留意してください。 Internet Computer のブロックチェーンメインネットに Canister を配置するには、`—network` コマンドラインオプションを使用して適切なターゲットを指定する必要があります。

## デモプロジェクトのビルドとデプロイ

LinkUp サンプル Dapp のビルドとデプロイは、以下の手順で行います：

1.  必要であれば、`pwd` コマンドを実行して、まだプロジェクトディレクトリにいることを確認します。

2.  以下のコマンドを実行して、LinkedUp Canister をビルドします：

        dfx build

3.  以下のコマンドを実行して、プロジェクトをローカル Canister 実行環境にデプロイします：

        dfx canister install --all

    次のようなメッセージとともに、`connectd`、`linkedup`、`linkedup_assets` の各 Canister の識別子が表示されるはずです：

        Installing code for canister connectd, with canister_id 75hes-oqbaa-aaaaa-aaaaa-aaaaa-aaaaa-aaaaa-q
        Installing code for canister linkedup, with canister_id cxeji-wacaa-aaaaa-aaaaa-aaaaa-aaaaa-aaaaa-q
        Installing code for canister linkedup_assets, with canister_id 7kncf-oidaa-aaaaa-aaaaa-aaaaa-aaaaa-aaaaa-q

4.  `dfx canister install` コマンドによって返された `linkedup_assets`  Canister 識別子をコピーします。

    このサンプルアプリでは、`linkedup_assets` Canister にのみ、アプリのユーザーインターフェースにアクセスするためのフロントエンドアセットが含まれています。したがって、ブラウザでアプリを開くには、`linkedup_assets` Canister の識別子を指定する必要があります。

5.  ウェブブラウザで `linkedup_assets` Canister を開きます。

    例えば、ローカル Canister 実行環境がデフォルトのローカルホストアドレスとポート番号にバインドされる場合、URL は次のようになります：

        http://127.0.0.1:8000/?canisterId=7kncf-oidaa-aaaaa-aaaaa-aaaaa-aaaaa-aaaaa-q

## プロフィールとコネクションを作成する

LinkedUp サンプル Dapp のデモを実行するには、次の手順を踏みます：

1.  ブラウザのタブまたはウィンドウを開きます。

2.  Web サーバのホスト名、ポート、および `canisterId` キーワードを入力し、表示する URL として `linkedup_assets` Canister の識別子を貼り付けます。

        127.0.0.1:8000/?canisterId=<ic-identifier-for-linkedup-assets>

    ブラウザに紹介ページが表示されます。

     Canister にアクセスするための本人確認のための公開鍵・秘密鍵のペアが自動的に生成されるので、サービスを利用する前にユーザー名とパスワードを入力したり、本人確認のためのアカウントを登録したりする必要はありません。

3.  **Login** をクリックします。

    ブラウザには、空のプロファイルページが表示されます。

    ![linkedup empty maya](../_attachments/linkedup-empty-maya.png)

4.  **Edit** をクリックし、プロフィール情報を入力し、アバター写真の画像アドレスをコピー＆ペーストして、**submit** をクリックしてください。

    ![linkedup edit maya](../_attachments/linkedup-edit-maya.png)

    **Submit** をクリックすると、プロフィールが作成され、職歴を閲覧することができます。

    例：

    ![linkedup profile maya](../_attachments/linkedup-profile-maya.png)

### 別のプロファイルを追加する

この時点では、他のプロフィールを検索したり、接続先として追加したりすることはできません。検索とコネクトの機能を試すには：

-   サンプルアプリに追加プロファイルを投入するスクリプトを実行します。

-   プライベートウィンドウを開いて、手動でプロフィールを作成します。

このチュートリアルでは、別のプロファイルを手動で作成します。

異なる Identity のユーザープロファイルを追加するには：

1.  ブラウザのウィンドウ右上にあるアイコンをクリックすると、ブラウザのメニューが表示されます。

    例えば、Google Chrome をお使いの場合、縦長の楕円をクリックすると、More メニューが表示されます。

2.  Google Chrome を使用している場合は **シークレット ウィンドウを開く**、Firefox を使用している場合は **新規
プライベートウィンドウ** をクリックして、Canister への最初のブラウザ接続で生成されたユーザ Identity を使用せずに Canister にナビゲートできるようにします。

3.  最初のブラウザセッションの URL をコピーしてプライベートブラウザウィンドウに貼り付け、**Login** をクリックします。

    ![linkedup incognito](../_attachments/linkedup-incognito.png)

    プライベートブラウザウィンドウにはプロフィールが表示されませんが、最初のブラウザタブには元のプロフィールが表示されていることに注意してください。

4.  **Edit** をクリックし、プロフィール情報を入力し、アバター写真の画像アドレスをコピー＆ペーストし、**Submit** をクリックします。

    ![linkedup edit dylan](../_attachments/linkedup-edit-dylan.png)

    **Submit** をクリックすると、2つ目のプロフィールが表示され、職歴を確認することができます。

    例：

    ![linkedup profile dylan](../_attachments/linkedup-profile-dylan.png)

5.  最初に作成したプロフィールの名前または姓を入力します。たとえば、Maya Garcia という人のプロフィールを作成した場合は、Maya と入力し、**Search** をクリックします。

    ![linkedup search from dylan for maya](../_attachments/linkedup-search-from-dylan-for-maya.png)

    検索条件に一致するプロフィールが表示されます。

    ![linkedup search result](../_attachments/linkedup-search-result.png)

6.  検索結果から連絡先を選択し、接続ボタンが表示されるまで待ち、**Connect** をクリックします。

    ![linkedup connect from dylan to maya](../_attachments/linkedup-connect-from-dylan-to-maya.png)

    接続要求が完了すると、第２のプロファイルは第１のプロファイルへの接続を表示します。たとえば、以下のようになります：

    ![linkedup connected to maya](../_attachments/linkedup-connected-to-maya.png)

7.  元のプロファイルが表示されているブラウザのタブに戻ります。

    元のプロファイルとプライベートブラウザウィンドウで作成したプロファイルの間に接続を作成したい場合は、Search、Select、Connect の手順を繰り返すことで行うことができます。

## 設定ファイルを調べる

サンプルアプリの基本的な機能を調べたので、設定ファイルやソースファイルがどのように使われるかを調べるためのコンテキストができました。

設定ファイルを調べるには：

1.  `linkedup` ディレクトリに移動し、プロジェクトの `dfx.json` ファイルを開いてください。

2.  ふたつのメイン Canister、`connectd` と `linkedup` が定義されており、それぞれ `main.mo` ソースファイルを持っていることに注意してください。

3.  `linkedup_assets`  Canister は、フロントエンドのエントリポイントである `main.js` と CSS と HTML ファイル形式のアセットを指定することに注意してください。

4.  なお、ローカル Canister の実行環境では、Dapp はデフォルトの IP アドレスとポート番号を使用してデプロイされるように設定されています。

## connectd のソースコードを調べる

ソーシャルコネクション Canister  `connectd` のソースコードは、以下のファイルのようになっています：

-   `digraph.mo` ファイルは、ユーザーの接続を記述するための頂点と辺の有向グラフを作成するための関数を提供します。

-   `main.mo`には、LinkedUp サンプルアプリが呼び出すことができ、ユーザープロファイルに関連する接続を定義するための Actor と主要な関数が含まれています。

-   `types.mo` ファイルは、`digraph` と `main` プログラムファイルで使用するために、頂点をユーザー Identity にマップするカスタムタイプを定義します。

## Linkedup ソースコードの調べる

職歴や学歴を記載したプロフィールのソースコードは、以下のファイルに整理されています：

-   `Main.mo`ファイルには、LinkedUp サンプルアプリの Actor と主要な関数が含まれています。

-   `types.mo` ファイルは、`linkedup` Canister の `main` プログラムファイルにインポートして使用する、ユーザー Indentity とプロファイルのフィールドを記述するカスタムタイプを定義します。

-   `utils.mo` ファイルはヘルパー関数を提供します。

### クエリとアップデートの操作

LinkedUp のサンプルアプリを使っていると、プロフィールを見たり検索したりすると、すぐに結果が返ってくるものがあることに気づくかもしれません。一方、プロフィールの作成や接続の追加などの操作では、少し時間がかかることがあります。

これらのパフォーマンスの違いは、`linkedup` Canister でクエリとアップデートコールを使用する場合の違いを示しています。

例えば、 `src/linkedup/main.mo` ファイルでは、 `create` と `update` 関数は Canister のステートを変更するアップデートコールであり、したがってコンセンサスを経る必要がありますが、プログラムでは `get` と `search` 関数のクエリコールを使ってプロファイルを表示または検索しています。

      // Profiles

      public shared(msg) func create(profile: NewProfile): async () {
        directory.createOne(msg.caller, profile);
      };

      public shared(msg) func update(profile: Profile): async () {
        if(Utils.hasAccess(msg.caller, profile)) {
          directory.updateOne(profile.id, profile);
        };
      };

      public query func get(userId: UserId): async Profile {
        Utils.getProfile(directory, userId)
      };

      public query func search(term: Text): async [Profile] {
        directory.findBy(term)
      };

### Canister 間の相互作用

このサンプルアプリでは、`linkedup`  Canister は `connectd` Canister で定義された関数を利用しています。このように分離することで、各 Canister のコードを単純化することができます。さらに重要なことは、ある Canister で定義された共通の関数を、他の Canister から呼び出すことでプロジェクトを拡張できることを説明することです。

ある Canister で定義されたパブリック関数を別の Canister で利用できるようにするには、次のようにします：

1.  呼び出し用 Canister に `import` ステートメントを追加します。

    この例では、パブリック関数は `connectd`  Canister で定義され、`linkedup`  Canister から呼び出されます。

    したがって、`src/linkedup/main.mo` には以下のコードが含まれます：

        // Make the Connectd app's public methods available locally
        import Connectd "canister:connectd";

2.  インポートされた Canister のパブリック・メソッドを呼び出すには、`canister.function` 構文を使用します。

    この例では、`linkedup` Canister はインポートされた `connectd` Canister の `connect` と `getConnections` 関数を呼び出します。

`linkedup` Canister と `connectd`  Canister の間の相互作用を可能にするコードは、`main.mo` ソースファイルから見ることができます。

例えば、`src/connectd/main.mo`では、以下のような関数が定義されています：

\+

    actor Connectd {
      flexible var graph: Digraph.Digraph = Digraph.Digraph();

      public func healthcheck(): async Bool { true };

      public func connect(userA: Vertex, userB: Vertex): async () {
        graph.addEdge(userA, userB);
      };

      public func getConnections(user: Vertex): async [Vertex] {
        graph.getAdjacent(user)
      };

    };

`Import` 文のおかげで、`connectd` 関数は `linkedup` Canister から利用できるようになり、`src/linkedup/main.mo` には以下のコードが含まれます：

      // Connections

      public shared(msg) func connect(userId: UserId): async () {
        // Call Connectd's public methods without an API
        await Connectd.connect(msg.caller, userId);
      };

      public func getConnections(userId: UserId): async [Profile] {
        let userIds = await Connectd.getConnections(userId);
        directory.findMany(userIds)
      };

      public shared(msg) func isConnected(userId: UserId): async Bool {
        let userIds = await Connectd.getConnections(msg.caller);
        Utils.includes(userId, userIds)
      };

      // User Auth

      public shared query(msg) func getOwnId(): async UserId { msg.caller }

    };

## ローカル Canister の実行環境を停止する

`linkedup` Dapp のテストが終わったら、ローカル Canister の実行環境を停止して、バックグラウンドで実行し続けないようにします。

ローカル Canister の実行環境を停止するには、以下のようにします：

1.  ネットワーク操作が表示されているターミナルで、Control-C キーを押して、ローカルネットワークの処理を中断します。

2.  以下のコマンドを実行して、ローカル Canister の実行環境を停止します：

        dfx stop

<!--
# Make inter-canister calls

One of the most important features of the Internet Computer blockchain for developers is the ability to call functions in one canister from another canister. This capability to make calls between canisters—also sometimes referred to as **inter-canister calls**—enables you to reuse and share functionality in multiple dapps.

For example, you might want to create a dapp for professional networking, organizing community events, or hosting fundraising activities. Each of these dapps might have a social component that enables users to identify social relationships based on some criteria or shared interest, such as friends and family or current and former colleagues.

To address this social component, you might want to create a single canister for storing user relationships then write your professional networking, community organizing, or fundraising application to import and call functions that are defined in the canister for social connections. You could then build additional applications to use the social connections canister or extend the features provided by the social connections canister to make it useful to an even broader community of other developers.

The Motoko-based LinkedUp sample dapp provides a simple implementation of an open professional network that demonstrates how to use inter-canister calls within a project.

The LinkedUp sample dapp is implemented using the following canisters:

-   The `linkedup` canister creates and stores basic profile information for a user, including work experience and educational background.

-   The `connectd` canister creates and stores a user’s connections.

-   The `linkedup_assets` canister stores the frontend assets—including the JavaScript, HTML, and CSS files—that define the user interface

## Before you begin

Before building the sample dapp, verify the following:

-   You have downloaded and installed the SDK package as described in [Download and install](../../quickstart/local-quickstart#download-and-install).

-   You have stopped the local canister execution environment provided by `dfx`.

## Download the demo

To experiment with inter-canister calls using the LinkedUp sample dapp:

1.  Open a terminal shell and change to the folder you are using for your Internet Computer sample projects.

2.  Clone the `linkedup` repository.

        git clone https://github.com/dfinity/linkedup.git

3.  Change to the local working directory for the `linkedup` repository.

        cd linkedup

4.  Install node modules by running the following command:

        npm install

    If necessary, fix any vulnerabilities found by running the following command:

        npm audit fix

5.  Open the `dfx.json` file in a text editor and verify the `dfx` setting has same the version number as the `dfx` executable you have installed.

## Start the local canister execution environment

For development purposes `dfx` provides a local canister execution environment. This requires a `dfx.json` file, so you should be sure you are in the linkedup root directory.

To start the local canister execution environment:

1.  Open a new terminal window or tab on your local computer.

2.  Navigate to the root directory for your project, if necessary.

3.  Start the local canister execution environment by running the following command:

        dfx start --background

    After the local canister execution environment completes its startup operations, you can continue to the next step.

## Register canister identifiers

Once the local canister execution environment is up and running, you can generate unique canister identifiers for your project.

To register canister identifiers:

1.  Check that you are still in your project directory, if needed.

2.  Register unique canister identifiers for the project by running the following command:

        dfx canister create --all

    The command displays the canister identifiers for the canisters defined in the `dfx.json` configuration file.

        "connectd" canister created with canister id: "75hes-oqbaa-aaaaa-aaaaa-aaaaa-aaaaa-aaaaa-q"
        "linkedup" canister created with canister id: "cxeji-wacaa-aaaaa-aaaaa-aaaaa-aaaaa-aaaaa-q"
        "linkedup_assets" canister created with canister id: "7kncf-oidaa-aaaaa-aaaaa-aaaaa-aaaaa-aaaaa-q"

    Keep in mind that because you are running the local canister execution environment, these identifiers are only valid on the local canister execution environment. To deploy canisters on the Internet Computer blockchain mainnet, you must specify the appropriate target using the `--network` command-line option.

## Build and deploy the demo project

To build and deploy the LinkUp sample dapp, take the following steps:

1.  Check that you are still in your project directory by running the `pwd` command, if necessary.

2.  Build the LinkedUp canisters by running the following command:

        dfx build

3.  Deploy the project on the local canister execution environment by running the following command:

        dfx canister install --all

    You should see canister identifiers for the `connectd`, `linkedup` and `linkedup_assets` canisters with a message similar to the following:

        Installing code for canister connectd, with canister_id 75hes-oqbaa-aaaaa-aaaaa-aaaaa-aaaaa-aaaaa-q
        Installing code for canister linkedup, with canister_id cxeji-wacaa-aaaaa-aaaaa-aaaaa-aaaaa-aaaaa-q
        Installing code for canister linkedup_assets, with canister_id 7kncf-oidaa-aaaaa-aaaaa-aaaaa-aaaaa-aaaaa-q

4.  Copy the `linkedup_assets` canister identifier returned by the `dfx canister install` command.

    In this sample dapp, only the `linkedup_assets` canister includes the frontend assets used to access the dapp’s user interface. To open the dapp in a browser, therefore, you need to specify the `linkedup_assets` canister identifier.

5.  Open the `linkedup_assets` canister in your web browser.

    For example, if the local canister execution environment binds to the default localhost address and port number, the URL looks similar to this:

        http://127.0.0.1:8000/?canisterId=7kncf-oidaa-aaaaa-aaaaa-aaaaa-aaaaa-aaaaa-q

## Create a profile and connections

To run through a demonstration of the LinkedUp sample dapp, take the following steps:

1.  Open a browser tab or window.

2.  Type the web server host name, port, and the `canisterId` keyword, then paste the `linkedup_assets` canister identifier as the URL to display.

        127.0.0.1:8000/?canisterId=<ic-identifier-for-linkedup-assets>

    The browser displays an introductory page.

    A public-private key pair will be automatically generated to establish your identity for accessing the canister, so there’s no need to provide a user name and password or register an account to store your identity before using the service.

3.  Click **Login**.

    The browser displays an empty profile page.

    ![linkedup empty maya](../_attachments/linkedup-empty-maya.png)

4.  Click **Edit**, type profile information, copy and paste the image address for an avatar photo, then click **Submit**.

    ![linkedup edit maya](../_attachments/linkedup-edit-maya.png)

    After you click **Submit**, you have a profile with some work history that can be viewed.

    For example:

    ![linkedup profile maya](../_attachments/linkedup-profile-maya.png)

### Add another profile

At this point, there are no other profiles to search for or to add as connections. To try out the Search and Connect features, you can:

-   Run a script that populates the sample dapp with some additional profiles.

-   Create a profile manually by opening a private window.

For this tutorial, you will create another profile manually.

To add a user profile with different identity:

1.  At the top right of the browser window, click the appropriate icon to display the browser’s menu options.

    For example, if you are using Google Chrome, click the vertical ellipse to display the More menu.

2.  Click **New Incognito Window** if you are using Google Chrome or **New Private Window** if you are using Firefox to enable you to navigate to the canister without the user identity generated in your initial browser connection to the canister.

3.  Copy and paste the URL from your first browser session into the private browsing window, then click **Login**.

    ![linkedup incognito](../_attachments/linkedup-incognito.png)

    Notice that there’s no profile in the private browsing window but your original profile is still visible in your initial browser tab.

4.  Click **Edit**, type profile information, copy and paste the image address for an avatar photo, then click **Submit**.

    ![linkedup edit dylan](../_attachments/linkedup-edit-dylan.png)

    After clicking **Submit**, you have a second profile with some work history that can be viewed.

    For example:

    ![linkedup profile dylan](../_attachments/linkedup-profile-dylan.png)

5.  Type the first name or last name from the first profile you created—for example, if you created a profile for Maya Garcia, type Maya—then click **Search**.

    ![linkedup search from dylan for maya](../_attachments/linkedup-search-from-dylan-for-maya.png)

    The profile matching your search criteria is displayed.

    ![linkedup search result](../_attachments/linkedup-search-result.png)

6.  Select the contact from the search results, wait for the Connect button to be displayed, then click **Connect**.

    ![linkedup connect from dylan to maya](../_attachments/linkedup-connect-from-dylan-to-maya.png)

    When the connection request completes, the second profile displays the connection to the first profile. For example:

    ![linkedup connected to maya](../_attachments/linkedup-connected-to-maya.png)

7.  Return to the browser tab with your original profile.

    If you want to create a connection between the original profile and the profile you created in the private browsing window, you can do so by repeating the search, select, and connect steps.

## Explore the configuration file

Now that you have explored the basic features of the sample dapp, you have some context for exploring how the configuration settings and source files are used.

To explore the configuration file:

1.  Change to the `linkedup` directory, then open the project’s `dfx.json` file.

2.  Note that there are two main canisters defined—`connectd` and `linkedup`—each with a `main.mo` source file.

3.  Note that the `linkedup_assets` canister specifies a frontend entry point of `main.js` and assets in the form of CSS and HTML files.

4.  Note that the dapp is configured to use the default IP address and port number for deployment on the local canister execution environment.

## Explore the connectd source code

The source code for the social connections canister, `connectd`, is organized into the following files:

-   The `digraph.mo` file provides the functions to create a directed graph of vertices and edges to describe a user’s connections.

-   The `main.mo` contains the actor and key functions for defining the connections associated with a user profile that can be called by the LinkedUp sample dapp.

-   The `types.mo` file defines the custom type that maps a vertex to a user identity for use in the `digraph` and `main` program files.

## Explore the linkedup source code

The source code for the professional profile with work history and educational background is organized into the following files:

-   The `main.mo` file contains the actor and key functions for the LinkedUp sample dapp.

-   The `types.mo` file defines the custom types that describe the user identity and profile fields that are imported and used in the `main` program file for the `linkedup` canister.

-   The `utils.mo` file provides helper functions.

### Query and update operations

In working with the LinkedUp sample dapp, you might notice that some operations—such as viewing a profile or performing a search—returned results almost immediately. Other operations—such as creating a profile or adding a connection—took a little longer.

These differences in performance illustrate the difference between using query and update calls in the `linkedup` canister.

For example, in the `src/linkedup/main.mo` file, the `create` and `update` functions are update calls that change the state of the canister and therefore need to go through consensus, but the program uses query calls for the `get` and `search` functions to view or search for a profile:

      // Profiles

      public shared(msg) func create(profile: NewProfile): async () {
        directory.createOne(msg.caller, profile);
      };

      public shared(msg) func update(profile: Profile): async () {
        if(Utils.hasAccess(msg.caller, profile)) {
          directory.updateOne(profile.id, profile);
        };
      };

      public query func get(userId: UserId): async Profile {
        Utils.getProfile(directory, userId)
      };

      public query func search(term: Text): async [Profile] {
        directory.findBy(term)
      };

### Interaction between the canisters

In this sample dapp, the `linkedup` canister leverages functions defined in the `connectd` canister. This separation simplifies the code in each canister and, more importantly, illustrates how you can extend a project by calling common functions defined in one canister from one or more other canisters.

To make the public functions defined in one canister available in the another canister:

1.  Add an `import` statement in the calling canister.

    In this example, the public functions are defined in the `connectd` canister and are called by the `linkedup` canister.

    Therefore, the `src/linkedup/main.mo` includes the following code:

        // Make the Connectd app's public methods available locally
        import Connectd "canister:connectd";

2.  Use the `canister.function` syntax to call public methods in the imported canister.

    In this example, the `linkedup` canister calls the `connect` and `getConnections` function in the imported `connectd` canister.

You can see the code that enables interaction between the `linkedup` canister and the `connectd` canister in the `main.mo` source files.

For example, the `src/connectd/main.mo` defines the following functions:

\+

    actor Connectd {
      flexible var graph: Digraph.Digraph = Digraph.Digraph();

      public func healthcheck(): async Bool { true };

      public func connect(userA: Vertex, userB: Vertex): async () {
        graph.addEdge(userA, userB);
      };

      public func getConnections(user: Vertex): async [Vertex] {
        graph.getAdjacent(user)
      };

    };

Because of the `Import` statement, the `connectd` functions are available to the `linkedup` canister and the `src/linkedup/main.mo` includes the following code:

      // Connections

      public shared(msg) func connect(userId: UserId): async () {
        // Call Connectd's public methods without an API
        await Connectd.connect(msg.caller, userId);
      };

      public func getConnections(userId: UserId): async [Profile] {
        let userIds = await Connectd.getConnections(userId);
        directory.findMany(userIds)
      };

      public shared(msg) func isConnected(userId: UserId): async Bool {
        let userIds = await Connectd.getConnections(msg.caller);
        Utils.includes(userId, userIds)
      };

      // User Auth

      public shared query(msg) func getOwnId(): async UserId { msg.caller }

    };

## Stop the local canister execution environment

After you finish experimenting with the `linkedup` dapp, you can stop the local canister execution environment so that it doesn’t continue running in the background.

To stop the local canister execution environment:

1.  In the terminal that displays the output of the local canister execution environment, press Control-C to interrupt the local process.

2.  Stop the local canister execution environment by running the following command:

        dfx stop

-->