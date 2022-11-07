# 台帳のローカルセットアップ

ローカル開発環境、パブリックの Internet Computer ではなくローカルのレプリカで作業している場合、ICP 台帳にアクセスすることはできません。ICP 台帳と統合するアプリケーションをローカルでテストするには、ローカル台帳 Canister をデプロイする必要があります。ただし、このローカル台帳 Canister には、ライブの ICP 台帳の履歴と残高がありません。
以下の手順に従って、台帳 Canister のコピーをローカル・レプリカにデプロイしてください。

1.  ビルド済みの台帳 Canister モジュールと Candid インターフェースファイルを入手します。

    ``` sh
    export IC_VERSION=dd3a710b03bd3ae10368a91b255571d012d1ec2f
    curl -o ledger.wasm.gz https://download.dfinity.systems/ic/${IC_VERSION}/canisters/ledger-canister_notify-method.wasm.gz
    gunzip ledger.wasm.gz
    curl -o ledger.private.did https://raw.githubusercontent.com/dfinity/ic/${IC_VERSION}/rs/rosetta-api/ledger.did
    curl -o ledger.public.did https://raw.githubusercontent.com/dfinity/ic/${IC_VERSION}/rs/rosetta-api/ledger_canister/ledger.did
    ```

    :::note

    `IC_VERSION` 変数は、<http://github.com/dfinity/ic> リポジトリからのコミットハッシュです。

    :::

2.  DFX の最新版を使用していることを確認してください。DFX がインストールされていない場合は、[SDK のインストール](../../build/install-upgrade-remove) の手順に従ってインストールを行ってください。

3.  DFX プロジェクトがまだない場合は、以下の手順で新規に DFX プロジェクトを作成してください：[dfx-new](../../../references/cli-reference/dfx-new.md)

4.  最初のステップで取得したファイル（`ledger.wasm`, `ledger.private.did`, `ledger.public.did` ）を、プロジェクトのルートにコピーしてください。

5.  プロジェクトの `dfx.json` ファイルに、以下の Canister 定義を追加します：

    ``` json
    {
      "canisters": {
        "ledger": {
          "type": "custom",
          "wasm": "ledger.wasm",
          "candid": "ledger.private.did"
        }
      }
    }
    ```

6. レプリカが `System` サブネットで動作するように設定します。`dfx.json` を修正し、インクルードします：
     ```json
     {
       "defaults":{
         "replica": {
           "subnet_type":"system"
         }
       }
     }
     ```

6.  ローカルレプリカを開始します。

    ``` sh
    dfx start --background
    ```

7.  ミントする機能のある新しいアカウントとして Identity を作成します：

    ``` sh
    dfx identity new minter
    dfx identity use minter
    export MINT_ACC=$(dfx ledger account-id)
    ```

    ミントするアカウントから振り込むと、`Mint` というトランザクションが作成されます。ミントのアカウントに送金すると、`Burn` というトランザクションが作成されます。

8.  デフォルトの Identity に切り替え、台帳のアカウント識別子を記録します。

    ``` sh
    dfx identity use default
    export LEDGER_ACC=$(dfx ledger account-id)
    ```

9.  台帳 Canister をネットワークにデプロイします。

    ``` sh
    dfx deploy ledger --argument '(record {minting_account = "'${MINT_ACC}'"; initial_values = vec { record { "'${LEDGER_ACC}'"; record { e8s=100_000_000_000 } }; }; send_whitelist = vec {}})'
    ```

    プロダクト環境と同じように台帳をセットアップしたい場合は、アーカイブを有効にしてデプロイする必要があります。この設定では、台帳 Canister は古いブロックを保存するために新しい Canister を動的に作成します。ブロックを取得するためにインターフェイスを使用する場合は、このセットアップを使用することをお勧めします。

    開発用に使用する Identity の Principal を取得します。このPrincipal は、アーカイブのコントローラーとなります。

    ``` sh
    dfx identity use default
    export ARCHIVE_CONTROLLER=$(dfx identity get-principal)
    ```

    アーカイブオプションで台帳 Canister をデプロイします：

    ``` sh
    dfx deploy ledger --argument '(record {minting_account = "'${MINT_ACC}'"; initial_values = vec { record { "'${LEDGER_ACC}'"; record { e8s=100_000_000_000 } }; }; send_whitelist = vec {}; archive_options = opt record { trigger_threshold = 2000; num_blocks_to_archive = 1000; controller_id = principal "'${ARCHIVE_CONTROLLER}'" }})'
    ```

    `trigger_threshold` と `num_blocks_to_archive` オプションを低い値 (たとえば 10 と 5) に設定して、数ブロック後にアーカイブを起動するようにするとよいでしょう。

10. `dfx.json`ファイルのCanisterの定義を更新して、public Candid インターフェイスを使用するようにします：

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

11. `dfx.json` ファイルの Canister 定義を更新して、台帳に remote id を指定します。これにより、プロジェクトを Internet Computer にデプロイすることになった場合に、dfx が自分の台帳をデプロイするのを防ぐことができます：

    ```
    "ledger": {
      "type": "custom",
      "candid": "ledger.public.did",
      "wasm": "ledger.wasm",
      "remote": {
        "candid": "ledger.public.did",
        "id": {
          "ic": "ryjl3-tyaaa-aaaaa-aaaba-cai"
        }
      }
    }
    ```

12. 台帳 Canister が健全であることを確認します。以下のコマンドを実行します：

    ``` sh
    dfx canister call ledger account_balance '(record { account = '$(python3 -c 'print("vec{" + ";".join([str(b) for b in bytes.fromhex("'$LEDGER_ACC'")]) + "}")')' })'
    ```

    以下のような出力になるはずです：

        (record { e8s = 100_000_000_000 : nat64 })

ローカルの ICP 台帳 Canister が起動しました。これで、台帳 Canister と通信する必要のある他の Canister をデプロイすることができます。

<!--
# Ledger Local Setup

If you are working in a local development environment, i.e with a local replica instead of the public Internet Computer, you can't access the ICP ledger. In order to test your application that integrates with the ICP ledger locally, you need to deploy a local ledger canister. However, this local ledger canister won't have the history and balances of the live ICP ledger.
Follow the steps below to deploy your copy of the ledger canister to a local replica.

1.  Get a pre-built Ledger canister module and Candid interface files.

    ``` sh
    export IC_VERSION=dd3a710b03bd3ae10368a91b255571d012d1ec2f
    curl -o ledger.wasm.gz https://download.dfinity.systems/ic/${IC_VERSION}/canisters/ledger-canister_notify-method.wasm.gz
    gunzip ledger.wasm.gz
    curl -o ledger.private.did https://raw.githubusercontent.com/dfinity/ic/${IC_VERSION}/rs/rosetta-api/ledger.did
    curl -o ledger.public.did https://raw.githubusercontent.com/dfinity/ic/${IC_VERSION}/rs/rosetta-api/ledger_canister/ledger.did
    ```

    :::note

    The `IC_VERSION` variable is a commit hash from the <http://github.com/dfinity/ic> repository.

    :::

2.  Make sure you use a recent version of DFX. If you don’t have DFX installed, follow instructions on the [Installing the SDK](../../build/install-upgrade-remove) section to install it.

3.  If you don’t have a DFX project yet, follow these instructions to create a new DFX project: [dfx-new](../../../references/cli-reference/dfx-new.md)

4.  Copy the file you obtained at the first step (`ledger.wasm`, `ledger.private.did`, `ledger.public.did`) into the root of your project.

5.  Add the following canister definition to the `dfx.json` file in your project:

    ``` json
    {
      "canisters": {
        "ledger": {
          "type": "custom",
          "wasm": "ledger.wasm",
          "candid": "ledger.private.did"
        }
      }
    }
    ```
    
6. Configure your replica to run a `System` subnet. Modify `dfx.json` to include:
     ```json
     {
       "defaults":{
         "replica": {
           "subnet_type":"system"
         }
       }
     }
     ```

6.  Start a local replica.

    ``` sh
    dfx start --background
    ```

7.  Create a new identity that will work as a minting account:

    ``` sh
    dfx identity new minter
    dfx identity use minter
    export MINT_ACC=$(dfx ledger account-id)
    ```

    Transfers from the minting account will create `Mint` transactions. Transfers to the minting account will create `Burn` transactions.

8.  Switch back to your default identity and record its ledger account identifier.

    ``` sh
    dfx identity use default
    export LEDGER_ACC=$(dfx ledger account-id)
    ```

9.  Deploy the ledger canister to your network.

    ``` sh
    dfx deploy ledger --argument '(record {minting_account = "'${MINT_ACC}'"; initial_values = vec { record { "'${LEDGER_ACC}'"; record { e8s=100_000_000_000 } }; }; send_whitelist = vec {}})'
    ```

    If you want to setup the ledger in a way that matches the production deployment, you should deploy it with archiving enabled. In this setup, the ledger canister dynamically creates new canisters to store old blocks. We recommend using this setup if you are planning to exercise the interface for fetching blocks.

    Obtain the principal of the identity you use for development. This principal will be the controller of archive canisters.

    ``` sh
    dfx identity use default
    export ARCHIVE_CONTROLLER=$(dfx identity get-principal)
    ```

    Deploy the ledger canister with archiving options:

    ``` sh
    dfx deploy ledger --argument '(record {minting_account = "'${MINT_ACC}'"; initial_values = vec { record { "'${LEDGER_ACC}'"; record { e8s=100_000_000_000 } }; }; send_whitelist = vec {}; archive_options = opt record { trigger_threshold = 2000; num_blocks_to_archive = 1000; controller_id = principal "'${ARCHIVE_CONTROLLER}'" }})'
    ```

    You may want to set `trigger_threshold` and `num_blocks_to_archive` options to low values (e.g., 10 and 5) to trigger archivation after only a few blocks.

10. Update the canister definition in the `dfx.json` file to use the public Candid interface:

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

11. Update the canister definition in the `dfx.json` file to specify a remote id for the ledger. This will prevent dfx from deploying your own ledger in case you decide to deploy your project to the Internet Computer:

    ```
    "ledger": {
      "type": "custom",
      "candid": "ledger.public.did",
      "wasm": "ledger.wasm",
      "remote": {
        "candid": "ledger.public.did",
        "id": {
          "ic": "ryjl3-tyaaa-aaaaa-aaaba-cai"
        }
      }
    }
    ```

12. Check that the Ledger canister is healthy. Execute the following command:

    ``` sh
    dfx canister call ledger account_balance '(record { account = '$(python3 -c 'print("vec{" + ";".join([str(b) for b in bytes.fromhex("'$LEDGER_ACC'")]) + "}")')' })'
    ```

    The output should look like the following:

        (record { e8s = 100_000_000_000 : nat64 })

Your local ICP ledger canister is up and running now. You can now deploy other canisters that need to communicate with the ledger canister.

-->