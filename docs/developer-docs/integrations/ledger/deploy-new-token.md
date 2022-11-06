# 新しいトークンのデプロイ

このチュートリアルでは、独自のトークンを IC にデプロイし、Rosetta を接続する方法を順を追って説明します。

## 台帳のデプロイ

1.  台帳イメージ、プライベート台帳インターフェース、パブリック台帳インターフェースがあることを確認します。それらがない場合は、[台帳のローカルセットアップ](./ledger-local-setup) の手順に従います。

2.  DFX の最新バージョンを使用していることを確認します。DFX がインストールされていない場合は、[SDKのインストール](../../build/install-upgrade-remove) の手順に従ってインストールを行います。

3.  プロジェクト内の `dfx.json` ファイルに、以下の Canisters に定義を追加します：

    ``` json
    {
      "canisters": {
        "custom-ledger": {
          "type": "custom",
          "wasm": "ledger.wasm",
          "candid": "ledger.private.did"
        }
      }
    }
    ```

4.  IC に台帳 Canister をデプロイします：

    ``` bash
    # メインネットに台帳をデプロイする場合は変数を "ic "に変更します。
    export NETWORK=local

    # （環境）変数をトークンのミントとバーンが可能なアカウントに変更します。
    export MINT_ACC=$(dfx ledger account-id)

    # アーカイブ Canister を制御する Principal に（環境）変数を変更します。
    export ARCHIVE_CONTROLLER=$(dfx identity get-principal)

    export TOKEN_NAME="My Token"
    export TOKEN_SYMBOL=XMTK

    dfx deploy --network ${NETWORK} custom-ledger --argument '(record {
      name = "'${TOKEN_NAME}'";
      symbol = "'${TOKEN_SYMBOL}'";
      minting_account = "'${MINT_ACC}'";
      initial_values = vec {};
      send_whitelist = vec {};
      archive_options = opt record {
        trigger_threshold = 2000;
        num_blocks_to_archive = 1000;
        controller_id = principal "'${ARCHIVE_CONTROLLER}'";
        cycles_for_archive_creation = opt 10_000_000_000_000
      }
    })'
    ```

    それぞれの単語の説明です。

    - `NETWORK` は、台帳を配置するレプリカの URL または名前です (例: メインネットには ic を使用します)。

    - `TOKEN_NAME` は新しいトークンで可読性を高くした名前です。

    - `TOKEN_SYMBOL` は新しいトークンのシンボルです。

    - `MINT_ACC` はトークンのミントとバーンを担当する Principal のアカウントです (
 [ICP 台帳](./index.md)を参照してください）。

    - `ARCHIVE_CONTROLLER` はアーカイブ Canister の [controller Principal](../../build/project-setup/cycles-wallet#controller-and-custodian-roles) です。

    <!--<div class="important">-->

    メインネットにデプロイする場合：

    - 常に `archive_options` フィールドを設定してください。アーカイブを無効にすると、台帳の容量が Canister ひとつ分のメモリに制限されます。

    - 台帳 Canister に十分な Cycle があることを確認してください。Canister は必要に応じてアーカイブ Canister の新しいインスタンスを生成するために Cycle を必要とします。`create_canister` メッセージに付けられる正確な Cycle 数は、 `cycles_for_archive_creation` オプションで制御します。

    <!--</div>-->

5.  `dfx.json`ファイルの Canister の定義を更新して、public Candid インターフェイスを使用するようにします：

    ``` diff
     {
       "canisters": {
         "ledger": {
           "type": "custom",
           "wasm": "ledger.wasm",
    -       "candid": "ledger.private.did"
    +       "candid": "ledger.public.did"
         }
       }
     }
    ```

6.  台帳 Canister が健全であることを確認します。以下のコマンドを実行します：

    ``` sh
    dfx canister --network ${NETWORK} call ledger symbol
    ```

    以下のような出力になるはずです：

        (record { symbol = "XMTK" })

新しいトークンがデプロイされ、すぐに使用できるようになりました。

## Rosetta にコネクトする

Rosetta は、台帳 Canister に接続し、[Rosetta API](https://www.rosetta-api.org) を公開するアプリケーションです。その主な目的は、取引所との統合を容易にすることです。Rosetta の詳細については、[次のセクション](../rosetta/index.md)で説明します。

それでは、Rosetta を既存の台帳 Canister に接続してみましょう。

1.  台帳トークン・シンボルの取得します：

    ``` sh
    dfx canister call <ledger_canister_id> symbol
    ```

    以下のような出力になるはずです：

        (record { symbol = <token_symbol> })

2.  `rosetta-api` を実行します：

    ``` bash
    docker run \
        --interactive \
        --tty \
        --publish 8081:8080 \
        --rm \
        dfinity/rosetta-api:v1.3.0 \
        --canister-id <ledger_canister_id> \
        --ic-url <replica> \
        -t <token_symbol>
    ```

    出力には次の行が含まれるはずです：

        16:31:45.472550 INFO [main] ic_rosetta_api::rosetta_server - Starting Rosetta API server
        16:31:45.506905 INFO [main] ic_rosetta_api::ledger_client - You are all caught up to block <x>

    上記の `<x>` は、台帳ブロックチェーンの最後のブロックインデックスを表しています。

`rosetta-api` はあなたの台帳インスタンスに接続され、使用する準備ができました。Rosetta トークンの転送操作については、[Transfers tokens](../rosetta/transfers) の記事をお読みください。

<!--
# Deploy New Token

This tutorial will guide you step-by-step to deploy your own token to the IC and to connect Rosetta to it.

## Deploy your Ledger

1.  Ensure you have the ledger image, the private ledger interface, and the public ledger interface. If you do not have them, follow the steps in [Setup Ledger locally](./ledger-local-setup).

2.  Ensure you use a recent version of DFX. If you don’t have DFX installed, follow instructions on the [Installing the SDK](../../build/install-upgrade-remove) section to install it.

3.  Add the following canister definition to the `dfx.json` file in your project:

    ``` json
    {
      "canisters": {
        "custom-ledger": {
          "type": "custom",
          "wasm": "ledger.wasm",
          "candid": "ledger.private.did"
        }
      }
    }
    ```

4.  Deploy the ledger canister to the IC:

    ``` bash
    # Change the variable to "ic" to deploy the ledger on the mainnet.
    export NETWORK=local

    # Change the variable to the account that can mint and burn tokens.
    export MINT_ACC=$(dfx ledger account-id)

    # Change the variable to the principal that controls archive canisters.
    export ARCHIVE_CONTROLLER=$(dfx identity get-principal)

    export TOKEN_NAME="My Token"
    export TOKEN_SYMBOL=XMTK

    dfx deploy --network ${NETWORK} custom-ledger --argument '(record {
      name = "'${TOKEN_NAME}'";
      symbol = "'${TOKEN_SYMBOL}'";
      minting_account = "'${MINT_ACC}'";
      initial_values = vec {};
      send_whitelist = vec {};
      archive_options = opt record {
        trigger_threshold = 2000;
        num_blocks_to_archive = 1000;
        controller_id = principal "'${ARCHIVE_CONTROLLER}'";
        cycles_for_archive_creation = opt 10_000_000_000_000
      }
    })'
    ```

    where

    -   the `NETWORK` is the url or name of the replica where you want to deploy the ledger (e.g. use ic for the mainnet)

    -   the `TOKEN_NAME` is the human-readable name of your new token

    -   the `TOKEN_SYMBOL` is the symbol of your new token

    -   the `MINT_ACC` is the account of the Principal responsible for minting and burning tokens (see the [Ledger documentation](./index.md))

    -   the `ARCHIVE_CONTROLLER` is the [controller Principal](../../build/project-setup/cycles-wallet#controller-and-custodian-roles) of the archive canisters

    <div class="important">

    When you deploy to the mainnet:

    -   Always set the `archive_options` field. If the archiving is disabled, the capacity of your ledger is limited to the memory of a single canister.

    -   Make sure that the ledger canister has plenty of cycles. The canister will need cycles to spawn new instances of the archive canister on demand. The exact number of cycles attached to `create_canister` messages is controlled by the `cycles_for_archive_creation` option.

    </div>

5.  Update the canister definition in the `dfx.json` file to use the public Candid interface:

    ``` diff
     {
       "canisters": {
         "ledger": {
           "type": "custom",
           "wasm": "ledger.wasm",
    -       "candid": "ledger.private.did"
    +       "candid": "ledger.public.did"
         }
       }
     }
    ```

6.  Check that the Ledger canister is healthy. Execute the following command:

    ``` sh
    dfx canister --network ${NETWORK} call ledger symbol
    ```

    The output should look like the following:

        (record { symbol = "XMTK" })

Your new token is deployed and ready to be used.

## Connect Rosetta

Rosetta is an application that connects to a Ledger canister and exposes the [Rosetta API](https://www.rosetta-api.org). Its main purpose is to facilitate integration with exchanges. You can learn more about Rosetta in the [next section](../rosetta/index.md).

Let us now connect Rosetta to an existing Ledger canister.

1.  Get the Ledger token symbol

    ``` sh
    dfx canister call <ledger_canister_id> symbol
    ```

    The output should look like the following:

        (record { symbol = <token_symbol> })

2.  Run `rosetta-api`

    ``` bash
    docker run \
        --interactive \
        --tty \
        --publish 8081:8080 \
        --rm \
        dfinity/rosetta-api:v1.3.0 \
        --canister-id <ledger_canister_id> \
        --ic-url <replica> \
        -t <token_symbol>
    ```

    The output should contain the following lines:

        16:31:45.472550 INFO [main] ic_rosetta_api::rosetta_server - Starting Rosetta API server
        16:31:45.506905 INFO [main] ic_rosetta_api::ledger_client - You are all caught up to block <x>

    The `<x>` above stands for the last block index in the ledger blockchain.

`rosetta-api` is connected to your Ledger instance and ready to be used. Read [Transfers tokens](../rosetta/transfers) article to learn about Rosetta token transfer operations.

-->