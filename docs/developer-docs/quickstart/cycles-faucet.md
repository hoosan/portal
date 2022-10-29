# Cycle フォーセット
Internet Computer 上で最初のスマートコントラクトをデプロイする準備はできていますか？ 私たちの Cycle フォーセットを使って、わずか数分で20 T の無料 Cycle をセットアップすることができます。

    sh -ci "$(curl -fsSL https://smartcontracts.org/install.sh)"

または <https://smartcontracts.org> にある説明に従ってください。

:::warning
Cycle フォーセットは現在アップデート中です。このページの一部はクローズになりました。こちらの <https://faucet.dfinity.org> にアクセスしてください。手直しが終わり次第、このページも更新される予定です。
:::

## Cycles Wallet を登録する

### Step 1： 認証

まず、<https://faucet.dfinity.org> に移動する必要があります。アクティブな Twitter アカウントを接続する必要があります。

![Connecting to Twitter](_attachments/faucet_step_1.png)

以前に同じ Twitter アカウントで請求したことがない場合のみ、20 T Cycle を請求することができます。

![Eligible account](_attachments/faucet_step_2.png)

続けるには NEXT をクリックします。

### Step 2: SDK のセットアップ

適合者であることが確認されたら、ターミナルウィンドウを開きます。

すでにプロジェクトを作成している場合は、ターミナルでプロジェクトのルートに移動し、`dfx.json` ファイルを配置します。まだプロジェクトを作成していない場合は、ターミナルで以下のコマンドを実行してください：

    mkdir my_project && cd my_project
    echo '{}' > dfx.json

![Setup SDK](_attachments/faucet_step_4.png)

続けるには NEXT をクリックします。

### Step 3: Canister を生成し Cycle を請求する

画面に表示される`redeem` コマンドには、固有のクーポンコードが含まれており、このコマンドで Canister を作成し、20 T Cycle を読み込むことができます。

![Create Canister and Claim Cycles](_attachments/faucet_step_5.png)

`redeem` コマンドを正常に実行すると、作成した Canister ID が返されます。

`redeem` コマンドを実行した後、status コマンドを使用して作成した Canister とその残高を確認することができます。`redeem` コマンドによって返された Canister ID を使用します。

    dfx canister --network=ic status <canister id>

なお、Canister ID は次のステップで使用しますので、メモしておいてください。

続けるには NEXT をクリックします。

### Step 4: ウォレットをセットする

Canister は、ウォレット Canister としてあなたの Principal ID にリンクさせることができるようになりました。ウォレットへのリンク付けは `dfx identity` コマンドを呼び出すことで行われます。

![Set Wallet](_attachments/faucet_step_6.png)

続けるには NEXT をクリックします。

### Step 5: ウォレット Canister を検証する

最後のステップは、`dfx wallet` コマンドを使用してウォレットの残高をチェックし、ウォレットが正しく設定されていることを確認することです：

![Verify Wallet Canister](_attachments/faucet_step_7.png)

## セットアップ完了

これで、IC 上でウェブサイトをホストする、または Dapp チュートリアルをフォローする準備が整いました。

### このようなガイドが、この次よく見られています：

-   [Local Development](./local-quickstart)

-   [Network Deployment](./network-quickstart)


<!--
# Cycles Faucet
Ready to deploy your first smart contract on the Internet Computer
blockchain? You can use our Cycles Faucet to get set up with 20T free
cycles in just a few minutes.

    sh -ci "$(curl -fsSL https://internetcomputer.org/install.sh)"

or following the instructions on the [Installing the SDK](../build/install-upgrade-remove) section.

:::warning
The cycles faucet is currently getting reworked. Some parts of this page are outdated. Please go to <https://faucet.dfinity.org> for the current instructions. Once the rework is completed, this page will be updated.
:::

## Claim your Cycles Wallet

### Step 1: Authenticate

First, you will need to navigate to <https://faucet.dfinity.org>. You
will need to connect an active Twitter account to continue.

![Connecting to Twitter](_attachments/faucet_step_1.png)

If the Twitter account has not been used before, you are eligible to claim the 20T cycles.

![Eligible account](_attachments/faucet_step_2.png)

Click NEXT to continue.

### Step 2: Setup SDK

Once your eligibility has been confirmed, open up a terminal window.

If you already have created a project, go to the root of the project in the terminal, where the `dfx.json` file is located. If you haven't created a project yet, run these commands in the terminal:

    mkdir my_project && cd my_project
    echo '{}' > dfx.json

![Setup SDK](_attachments/faucet_step_4.png)

Click NEXT to continue.

### Step 3: Create Canister and Claim Cycles

The `redeem` command shown on the screen contains a unique coupon code, which with this command will be used to create a canister and load it with 20T cycles. 

![Create Canister and Claim Cycles](_attachments/faucet_step_5.png)

After a successfully running the `redeem` command, the created canister's ID is returned. 

After runninng the `redeem` command, the created canister and it's balance can be checked using the status command. Use the canister ID returned by the `redeem` command:

    dfx canister --network=ic status <canister id>

Please note the canister ID is used in the next step, so write down the canister ID. 

Click NEXT to continue.

### Step 4: Set Wallet

The canister can now be linked to your principal ID as your wallet canister. The wallet is linked by calling a `dfx identity` command:

![Set Wallet](_attachments/faucet_step_6.png)

Click NEXT to continue.

### Step 5: Verify Wallet Canister

The last step is to verify the wallet is setup correctly, by checking its balance using the `dfx wallet` command:

![Verify Wallet Canister](_attachments/faucet_step_7.png)

## Setup Completed

Now you are ready to host a website on the IC or follow one of our dapp tutorials.

### Next, people often look at these guides:

-   [Local Development](./local-quickstart)

-   [Network Deployment](./network-quickstart)

-->
