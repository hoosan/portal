# Rosetta

[Rosetta](https://www.rosetta-api.org/) は、取引所、ブロックエクスプローラー、ウォレットにおけるブロックチェーンベーストークンの統合を簡素化するために Coinbase が導入したオープンスタンダードです。
CeFi の取引所で取引できることを目指したトークンを IC 上にデプロイしたい場合や、ブロックエクスプローラーやウォレットに取り組んでいる場合、このドキュメントが役に立つはずです。

## Rosetta ノードのセットアップ

Rosetta API に対応したノードをセットアップして、Internet Computer と対話し、Internet Computer Protocol (ICP) トークンを交換することができます。
手順をシンプルにするため、Rosetta API との統合を作成するためにDocker イメージを使用します。
また、ソースコードを使ってバイナリをビルドして実行することもできます。
[Docker](https://docs.docker.com/get-docker/) がまだインストールされていない場合は、最新版をダウンロードしてインストールしてください。

Rosetta ノード（testnet に接続する）をセットアップするには：

1.  [Docker のインストール](https://docs.docker.com/get-docker/) と [Docker daemon](https://docs.docker.com/config/daemon/) を起動します。

    コンピュータを再起動すると、Docker デーモン（`dockerd`）が自動的に起動するはずです。手動で Docker デーモンを起動する場合は、ローカルのオペレーティングシステムによって手順が異なります。

2.  以下のコマンドを実行し、Docker Hub から最新の `dfinity/rosetta-api` イメージを取得します：

    ``` bash
    docker pull dfinity/rosetta-api
    ```

3.  以下のコマンドを実行して、統合ソフトを起動します：

    ``` bash
    docker run \
        --interactive \
        --tty \
        --publish 8080:8080 \
        --rm \
       dfinity/rosetta-api
    ```

    このコマンドは、ローカルホスト上でソフトウェアを起動し、次のような出力を表示します：

        Listening on 0.0.0.0:8080
        Starting Rosetta API server

    デフォルトでは、このソフトウェアはテストネットに接続します。
    Internet Computer ブロックチェーンメインネット上で動作している台帳 Canister には**接続しません**。

    テストネットワークとそれに対応する台帳 Canister の識別子が割り当てられている場合、追加の `canister` 引数を指定することで、そのネットワークに対してコマンドを実行することができます。
    たとえば、次のコマンドでは `canister` 引数を `2xh5f-viaaa-aaaab-aae3q-cai` に設定して、テストネットワーク上の台帳 Canister に接続する例を示しています。

    ``` bash
    docker run \
        --interactive \
        --tty \
        --publish 8080:8080 \
        --rm \
       dfinity/rosetta-api \
       --canister 2xh5f-viaaa-aaaab-aae3q-cai
    ```

    :::note

    最初にこのコマンドを実行したとき、ノードがチェーンの現在のリンクに追いつくまで、いくらか時間がかかるかもしれません。
    ノードが追いつくと、以下のような出力が表示されるはずです。

    :::

        You are all caught up to block height 109

    このステップの完了後、ノードはブロック作成に参加しない **passive** ノードとして実行を継続します。

4.  新しいターミナルウィンドウまたはタブを開いて `ps` コマンドを実行し、サービスの状態を確認します。

    サービスを停止する必要がある場合は、<kbd>Ctrl-C</kbd> を押してください。例えば、使用している Canister の識別子を変更するために、この操作を行いたい場合があります。

    ノードのセットアップ後に統合をテストするには、Principal がトランザクションを送信したり、口座残高を検索したりすることをシミュレートするプログラムを作成する必要があります。

### Rosetta ノードをプロダクトモードで稼動する

テストが終わったら、 `--interactive`、 `--tty`、`--rm` コマンドラインオプションを指定せずに、プロダクトモードで Docker イメージを実行する必要があります。
これらのコマンドラインオプションは、インタラクティブなターミナルセッションをアタッチし、コンテナを削除するために使用され、主にテスト目的のために用意されています。
プロダクト環境で実行する場合は、`--detach` オプションを使用して Docker イメージを起動し、コンテナをバックグラウンドで実行し、オプションとしてブロックを保存するための `--volume` コマンドを指定します。

Rosetta ノードインスタンスをメインネットに接続するためにフラグを追加します。`--mainnet` と `--not-whitelisted` を追加します。

Docker のコマンドラインオプションの詳細については、[Docker reference documentation](https://docs.docker.com/engine/reference/commandline/run/) を参照してください。

### 要求事項と制限事項

Docker イメージで提供される統合ソフトウェアには、Rosetta API の標準仕様にはない要件がひとつあります。

ICP トークンを含むトランザクションでは、署名なしトランザクションはネットワークが署名付きトランザクションを受け取る24時間以内に作成されなければなりません。
その理由は、各トランザクションの ‘created_at’ フィールドが、既存のトランザクション（基本的にはトランザクション作成時にローカルで利用可能な last_index）を参照しているからです。
あまりにも古いトランザクションを参照するトランザクション送信は、運用効率を維持するために拒否されます。

この要件以外では、Rosetta API 統合ソフトウェアは、すべての標準的な Rosetta エンドポイントに完全に準拠し、`rosetta-cli` のテストに合格しています。
ソフトウェアはあらゆる有効な Rosetta リクエストを受け入れることができます。
しかし、この統合ソフトウェアは、[ここにリストされているすべての署名スキーム](https://www.rosetta-api.org/docs/models/SignatureType.html#values)ではなく、Ed25519 を使用して署名するトランザクションを要求するだけです。そのため仕様でサポートされている潜在的なレスポンスだけが小さなサブセットとして返信されています。
例えば、このソフトウェアは Rosetta の UTXO 機能を実装していないため、ソフトウェアの応答に UTXO メッセージが含まれることはないでしょう。


## よくある質問

以下の質問は、Rosetta と Internet Computer の連携に関して、開発者コミュニティからよく報告される質問とエラー等をまとめたものです。

### Rosetta ノード

#### Rosetta ノードのインスタンスを実行するにはどうしたらいいですか？

これを実現する簡単な方法は、[`dfinity/rosetta-api`](https://hub.docker.com/r/dfinity/rosetta-api/tags?page=1&ordering=last_updated) Docker イメージを使用することです。
ノードが初期化され、全てのブロックが同期されると、ノード上で Rosetta API を呼び出してクエリを実行したり、トランザクションを送信したりすることができます。
ノードは `8080` ポートでリッスンします。

#### Rosetta ノードをメインネットに接続するにはどうしたらいいですか？

フラグの `--mainnet` と `--not-whitelisted` を使用します。

#### Rosetta ノードをメインネットに接続するにはどうしたらいいですか？

フラグの `--mainnet` と `--not-whitelisted` を使用します。

#### ノードがテストネットに追いついたかどうかは、どうすれば分かりますか？

Rosetta API サーバーの起動ログを検索してください。`You are all caught up to block XX` というログエントリーがあるはずです。
このメッセージは、すべてのブロックに追いついていることを確認するものです。

#### 同期されたブロックデータを永続化するには？

`data` ディレクトリを [ボリューム](https://docs.docker.com/storage/volumes/) としてマウントします。

```bash
docker volume create rosetta
```
```bash
docker run \
    --volume rosetta:/data \
    --interactive \
    --tty \
    --publish 8080:8080 \
    --rm \
   dfinity/rosetta-api
```

#### Rosetta ノードはバージョン管理されていますか？

はい、定期的に [DockerHub](https://hub.docker.com/r/dfinity/rosetta-api/tags) で新バージョンを公開しています。
プロダクト環境では `dfinity/rosetta-api:v1.7.0` のように、特定のバージョンを使用することをお勧めします。
`/network/options` エンドポイントを使用すると、実行中の Rosetta ノードのバージョンを照会することができます。

```console
$ curl -s -q -H 'Content-Type: application/json' -d '{"network_identifier": {"blockchain": "Internet Computer", "network": "00000000000000020101"}}' -X POST http://localhost:8080/network/options | jq '.version.node_version'

"1.7.0"
```

### ICP 固有のRosetta API の詳細

#### アカウントの生成と検証はどのように行われるのですか？

- ED25519 キーペアを生成します。
- 秘密鍵はトランザクションに署名するために使用されます。
- 公開鍵は、自己認証の Principal ID を生成するために使用されます。詳細は、[Section Principals in the Interface Specification](../../../references/ic-interface-spec.md#principals) を参照してください。
- Principal ID はハッシュ化されるとアカウントアドレスが生成されます。

#### 公開鍵を使って、そのアカウントアドレスを生成する方法は？

- 16進数（hex）エンコードされた 32バイトの公開鍵で [`/construction/derive`](https://www.rosetta-api.org/docs/ConstructionApi.html#constructionderive) エンドポイントを呼び出します。
- JavaScript SDK の `pub_key_to_address` 関数を呼び出します。

#### アカウントアドレスのチェックサムを確認する方法は？

- 16進数（hex）デコード後、最初の 4バイトはアドレスの残りの部分のビッグエンディアンの CRC32 チェックサムです。
- JavaScript SDK で[`address_from_hex`](https://github.com/dfinity/rosetta-client#working-with-account-addresses) を呼び出してください。チェックサムが一致しない場合はエラーを返します。
- アドレス検証ロジックの Java 実装は[こちら](https://gist.github.com/TerrorJack/d6c79b33e5b5d0f5d52f3a2c5cdacc60)です。

#### ED25519 の `signature_type` と `curve_type` とは何ですか？

- `signature_type` とは `"ed25519"` のことです。
- `curve_type` とは `"edwards25519"` のことです。

#### ロックに表示される取引にはどのような種類があり、どのような意味があるのでしょうか？

- エンドポイント[`/block`](https://www.rosetta-api.org/docs/BlockApi.html#block) から照会される各ブロックには、正確に1つのトランザクションが含まれます。
  `burn` などのいくつかの操作は、Rosetta API コールではサポートされていないことに注意してください。

- Transfer
  - 操作0: `"TRANSACTION"` と入力し、送金元のアカウントから送金数を引きます。
  - 操作1: `"TRANSACTION"` と入力し、同じ送金数を送金先のアカウントに追加します。
  - 操作2: `"FEE"` と入力し、送金元のアカウントから手数料を引きます。

  :::note

    前のポイントで提示した操作の順番に依存してはいけません。
    システムは `/construction/payloads` 呼び出しに渡された操作を並べ替えることができます。
    代わりに、トランザクションの種類と金額の符号をチェックする必要があります。

  :::

- Mint
  - 操作0：タイプ `"MINT”`、ミント額を送金先アカウントに追加します。
- Burn
  - 操作 0: `"BURN"` と入力し、ソースアカウントからバーンの金額を引きます。
- `"status"` はいつも `"COMPLETED"`.
  失敗した取引は台帳に表示されません。

#### どのような料金が必要ですか？料金のカスタマイズは可能ですか？

- [`/construction/metadata`](https://www.rosetta-api.org/docs/ConstructionApi.html#constructionmetadata) を呼び出すことで、`suggested_fee` を取得することができます。
- 現時点では、`suggested_fee` は定数であり、送金時に指定する料金はこれと等しくなければなりません。
- 手数料は Mint や Burn の操作には適用されません。

#### 送信されたトランザクションがチェーンにヒットしたかどうかを知るにはどうすればよいですか？

- Rosetta サーバーは `/construction/submit` を呼び出した後、しばらく待つ必要があります。トランザクションがチェーンにヒットした場合、サーバーはブロックハッシュを返します。
- 台帳からエラーが出た場合は、`/construction/submit` の結果でエラー情報を得ることができます。
- `/construction/submit` の呼び出しが正常に返されたとしても、それがチェーンにヒットするまでにはまだ時間がかかる可能性があります。
  最新のブロックをポーリングして、トランザクションハッシュを検索することができます。
  Rosetta ノードは [`/search/transactions`](https://www.rosetta-api.org/docs/SearchApi.html#searchtransactions) インターフェースのサブセットもサポートしており、ハッシュが与えられたトランザクションを検索することができます。

- 最悪の場合、５分でタイムアウトです。

- `mempool` API を使用しないでください。空のスタブ実装となっています。

#### Rosetta API の呼び出しで、どのようなエラーが発生する可能性がありますか？

- 呼び出しに成功すると、常に `200` 応答ステータスコードが表示されます。
- 失敗した呼び出しには、常に `500` 応答ステータスコードがあり、より詳細な情報を含む JSON ペイロードがあります。
  すべての Rosetta エラーコードとそのテキスト説明は、[`/network/options`](https://www.rosetta-api.org/docs/NetworkApi.html#networkoptions) のコール結果に記載されています。

#### Mint や Burn のトランザクションを送信するにはどうすればよいですか？

- Mint は高いレベルのアクセス権（特権）が必要な操作です。現在、Rosetta API コールによる Burn はサポートしていません。

#### 同じ署名付きトランザクションを複数回送信した場合はどうなりますか？

台帳は重複したトランザクションを拒否します。
最初のトランザクションだけがチェーンに登録されます。[`/construction/submit`](https://www.rosetta-api.org/docs/ConstructionApi.html#constructionsubmit) の呼び出しは、重複したトランザクションに対して失敗します。

#### Rosetta API を呼び出さずに署名する方法は？

JavaScript SDK には、オフライン署名ロジックの [実装](https://github.com/dfinity/rosetta-client/blob/master/lib/construction_combine.js) が含まれています。
この機能は内部実装の詳細と連動しているため不安定であり、可能であれば [`/construction/combine`](https://www.rosetta-api.org/docs/ConstructionApi.html#constructioncombine) を呼び出してトランザクションに署名することを強く推奨します。

#### イングレスの時間帯を設定するには？

`construct/payloads` の呼び出しでは、 `ingress_start` / `ingress_end` フィールドを一つまたは全て追加して、イングレスの時間帯を指定することができます。
これらは Unix のエポックからのナノ秒であり、次の 24時間以内である必要があります。
`ingress_start` / `ingress_end` フィールドを追加すると、トランザクションの生成と署名が可能になりますが、送信は後に延期されます。

#### 署名付きトランザクションをデシリアライズする方法は？

Rosetta ノードの [`/construction/parse`](https://www.rosetta-api.org/docs/ConstructionApi.html#constructionparse) エンドポイントを使用します。
この機能は、オフラインモードでも使用できます。

<!--
# Rosetta

[Rosetta](https://www.rosetta-api.org/) is an open standard introduced by Coinbase to simplify the integration of blockchain-based tokens in exchanges, block explorers, and wallets.
This documentation might help if you want to deploy a token on the IC that aims to be tradable on CeFi exchanges or if you are working on a block explorer or wallet.

## Set up a Rosetta node

You can set up a Rosetta API-compliant node to interact with the Internet Computer and exchange Internet Computer Protocol (ICP) tokens.
To keep the instructions simple, we use a Docker image to create the integration with the Rosetta API.
You can also build and run the binary using the source code.
If you don’t already have [Docker](https://docs.docker.com/get-docker/) on your computer, download and install the latest version.

To set up a Rosetta node (which connects to a testnet):

1.  [Install Docker](https://docs.docker.com/get-docker/) and [start the Docker daemon](https://docs.docker.com/config/daemon/).

    The Docker daemon (`dockerd`) should automatically start when you reboot your computer. If you start the Docker daemon manually, the instructions vary depending on the local operating system.

2.  Pull the latest `dfinity/rosetta-api` image from the Docker Hub by running the following command:

    ``` bash
    docker pull dfinity/rosetta-api
    ```

3.  Start the integration software by running the following command:

    ``` bash
    docker run \
        --interactive \
        --tty \
        --publish 8080:8080 \
        --rm \
       dfinity/rosetta-api
    ```

    This command starts the software on the local host and displays output similar to the following:

        Listening on 0.0.0.0:8080
        Starting Rosetta API server

    By default, the software connects to a testnet.
    It **does not** connect to the ledger canister running on the Internet Computer blockchain mainnet.

    If you have been assigned a test network and corresponding ledger canister identifier, you can run the command against that network by specifying an additional `canister` argument.
    For example, the following command illustrates connecting to the ledger canister on a test network by setting the `canister` argument to `2xh5f-viaaa-aaaab-aae3q-cai`.

    ``` bash
    docker run \
        --interactive \
        --tty \
        --publish 8080:8080 \
        --rm \
       dfinity/rosetta-api \
       --canister 2xh5f-viaaa-aaaab-aae3q-cai
    ```

    :::note

    The first time you run the command, it might take some time for the node to catch up to the current link of the chain.
    When the node is caught up, you should see output similar to the following:

    :::

        You are all caught up to block height 109

    After completing this step, the node continues to run as a **passive** node that does not participate in block making.

4.  Open a new terminal window or tab and run the `ps` command to verify the status of the service.

    If you need to stop the service, press <kbd>Ctrl-C</kbd>. You might want to do this to change the canister identifier you are using, for example.

    To test the integration after setting up the node, you will need to write a program to simulate a principal submitting a transaction or looking up an account balance.

### Run the Rosetta node in production

When you are finished testing, you should run the Docker image in production mode without the `--interactive`, `--tty`, and `--rm` command-line options.
These command-line options are used to attach an interactive terminal session and remove the container, and are primarily intended for testing purposes.
To run the software in a production environment, you can start the Docker image using the `--detach` option to run the container in the background and, optionally, specify the `--volume` command for storing blocks.

To connect the Rosetta node instance to the mainnet, add flags: `--mainnet` and `--not-whitelisted`.

For more information about Docker command-line options, see the [Docker reference documentation](https://docs.docker.com/engine/reference/commandline/run/).

### Requirements and limitations

The integration software provided in the Docker image has one requirement that is not part of the standard Rosetta API specification.

For transactions involving ICP tokens, the unsigned transaction must be created less than 24 hours before the network receives the signed transaction.
The reason is that each transaction's 'created_at' field refers to an existing transaction (essentially last_index available locally at the time of transaction creation).
Any submitted transaction that refers back to a too old transaction is rejected to maintain operational efficiency.

Other than this requirement, the Rosetta API integration software fully complies with all standard Rosetta endpoints and passes all of the `rosetta-cli` tests.
The software can accept any valid Rosetta request.
However, the integration software only prompts for transactions to be signed using Ed25519, rather than [all the signature schemes [listed here](https://www.rosetta-api.org/docs/models/SignatureType.html#values), and only replies with a small subset of the potential responses that the specification supports.
For example, the software doesn’t implement any of the UTXO features of Rosetta, so you won’t see any UTXO messages in any of the software responses.


## Frequently asked questions

The following questions come from the most commonly reported questions and blockers from the developer community regarding Rosetta integration with the Internet Computer.

### The Rosetta node

#### How do I run an instance of the Rosetta node?

An easy way to accomplish this is to use the [`dfinity/rosetta-api`](https://hub.docker.com/r/dfinity/rosetta-api/tags?page=1&ordering=last_updated) Docker image.
Once the node initializes and syncs all blocks, you can perform queries and submit transactions by invoking the Rosetta API on the node.
The node listens on the `8080` port.

#### How do I connect the Rosetta node to the mainnet?

Use flags `--mainnet` and `--not-whitelisted`

#### How do I connect the Rosetta node to the mainnet?

Use flags `--mainnet` and `--not-whitelisted`

#### How do I know if the node has caught up with the test net?

Search the `Starting Rosetta API server` startup log. There will be a log entry that says `You are all caught up to block XX`.
This message confirms that you are caught up with all blocks.

#### How to persist synced blocks data?

Mount the `/data` directory as a [volume](https://docs.docker.com/storage/volumes/).

```bash
docker volume create rosetta
```
```bash
docker run \
    --volume rosetta:/data \
    --interactive \
    --tty \
    --publish 8080:8080 \
    --rm \
   dfinity/rosetta-api
```

#### Is the Rosetta node versioned?

Yes, we regularly publish new versions on [DockerHub](https://hub.docker.com/r/dfinity/rosetta-api/tags).
We recommend using a specific version in production settings, e.g., `dfinity/rosetta-api:v1.7.0`
You can query the version of a running rosetta node using the `/network/options` endpoint.

```console
$ curl -s -q -H 'Content-Type: application/json' -d '{"network_identifier": {"blockchain": "Internet Computer", "network": "00000000000000020101"}}' -X POST http://localhost:8080/network/options | jq '.version.node_version'

"1.7.0"
```

### ICP-specific Rosetta API details

#### How are accounts generated and verified?

- Generate an ED25519 keypair.
- The secret key is used for signing transactions.
- The public key is used for generating a self-authenticating Principal ID. For more information, see [Section Principals in the Interface Specification](../../../references/ic-interface-spec.md#principals).
- The Principal ID is hashed to generate the account address.

#### How to use the public key to generate its account address?

- Call the [`/construction/derive`](https://www.rosetta-api.org/docs/ConstructionApi.html#constructionderive) endpoint with the hex-encoded 32-byte public key.
- Call the `pub_key_to_address` function in the JavaScript SDK.

#### How to verify the checksum of an account address?

- After hex decoding, the first 4 bytes is the big-endian CRC32 checksum of the rest of the address.
- Call [`address_from_hex`](https://github.com/dfinity/rosetta-client#working-with-account-addresses) in the JavaScript SDK. It returns and error if checksum doesn’t match.
- [Here](https://gist.github.com/TerrorJack/d6c79b33e5b5d0f5d52f3a2c5cdacc60) is a Java implementation of address validation logic.

#### What are `signature_type` and `curve_type` for ED25519?

- `signature_type` is `"ed25519"`
- `curve_type` is `"edwards25519"`

#### What kinds of transactions can appear in a block, and what do they mean?

- Each block as queried from the [`/block`](https://www.rosetta-api.org/docs/BlockApi.html#block) endpoint contains precisely one transaction.
  Note that some operations, such as `burn`, are not supported in Rosetta API calls.

- Transfer
  - Operation 0: type `"TRANSACTION"`, subtracts the transfer amount from the source account.
  - Operation 1: type `"TRANSACTION"`, adds the same transfer amount to the destination account.
  - Operation 2: type `"FEE"`, subtracts the fee from the source account.

  :::note

    Don’t rely on the order of operations presented in the previous point; the system can rearrange the operations passed the `/construction/payloads` call.
    You should check for transaction type and amount sign instead.

  :::

- Mint
  - Operation 0: type `"MINT"`, adds the minted amount to the destination account.
- Burn
  - Operation 0: type `"BURN"`, subtract the burned amount from the source account.
- `"status"` is always `"COMPLETED"`.
  Failed transactions do not appear on the ledger.

#### What fee is needed? Can I customize the fee?

- By calling [`/construction/metadata`](https://www.rosetta-api.org/docs/ConstructionApi.html#constructionmetadata), you can get `suggested_fee`.
- At the moment, `suggested_fee` is a constant, and the fee specified in a transfer must be equal to it.
- Fees do not apply to Mint or Burn operations.

#### How do I know if the submitted transaction hit the chain?

- The Rosetta server will wait for a short period of time after a `/construction/submit` call; if the transaction hits the chain, the server will return the block hash.
- In case of an error from the ledger, the error information will be available in the `/construction/submit` result.
- It’s still possible that a `/construction/submit` call has returned successfully, but there’s still some time before it hits the chain.
  You can poll the latest blocks and search for the transaction hash.
  The Rosetta node also supports a subset of the [`/search/transactions`](https://www.rosetta-api.org/docs/SearchApi.html#searchtransactions) interface that allows searching for a transaction given its hash.

- Five minutes is a worst-case timeout.

- Don’t use `mempool` APIs; our implementation is an empty stub.

#### What kinds of errors might I get from Rosetta API calls?

- Successful calls always have `200` response status code.
- Failed calls always have `500` response status code, with a JSON payload containing more information.
  You can find all possible Rosetta error codes and their text descriptions in the [`/network/options`](https://www.rosetta-api.org/docs/NetworkApi.html#networkoptions) call result.

#### How do I send Mint or Burn transactions?

- Mint is a privileged operation; we don’t support Burn through Rosetta API calls at the moment.

#### What happens if the same signed transaction is submitted multiple times?

The ledger rejects duplicate transactions.
Only the first transaction will make it to the chain; the [`/construction/submit`](https://www.rosetta-api.org/docs/ConstructionApi.html#constructionsubmit) call will fail for the duplicates.

#### How to sign a transaction without calling Rosetta API?

The JavaScript SDK contains an [implementation](https://github.com/dfinity/rosetta-client/blob/master/lib/construction_combine.js) of the offline signing logic.
This functionality is unstable because it is coupled with internal implementation details, so we strongly advise you to call [`/construction/combine`](https://www.rosetta-api.org/docs/ConstructionApi.html#constructioncombine) to sign transactions if possible.

#### How to configure the ingress time period?

In the `/construction/payloads` call, you can add one or all of the `ingress_start` / `ingress_end` fields to specify the ingress time period.
These are nanoseconds since the Unix epoch and must be within the next 24 hours.
Adding `ingress_start`/`ingress_end` fields enables generating & signing of a transaction but delays the submission to a later time.

#### How to deserialize a signed transaction?

Use the [`/construction/parse`](https://www.rosetta-api.org/docs/ConstructionApi.html#constructionparse) endpoint of a Rosetta Node.
This functionality is also available in offline mode.

-->