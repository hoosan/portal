# Bitcoin 統合の仕組み ー 技術的背景

Bitcoin を IC 内で統合するには、（1）IC と Bitcoin ネットワークのプロトコルレベルでの統合、（2）新しい閾値 ECDSA プロトコルの開発という、2つの高度な技術的課題を解決する必要がありました。

** IC と Bitcoin ネットワークのプロトコルレベルでの統合**

IC と Bitcoin ネットワークのプロトコルレベルでの統合により、IC は Bitcoin ブロックを Bitcoin ネットワークから直接取得し、含まれるトランザクションを処理することができます。これにより、IC 上で完全な Bitcoin UTXO セットをオンチェーンに維持することができます。Canister は、完全な Bitcoin UTXO セットに対してクエリーを実行することができます。これにより、Canister は自分自身のアドレスを含む任意の Bitcoin アドレスの保有する UTXO、すなわち残高について知ることができます。

**革新的の ECDSA プロトコル**

Canister 自身は、新しい閾値 ECDSA プロトコルを用いて ECDSA 鍵を持つことができるため、Bitcoin を受け取り、保持できます。また、Canister はBitcoin のトランザクションを作成し、Bitcoin API を介して Bitcoin ネットワークに送信することができます。Canister は閾値 ECDSA の機能を利用して、Bitcoin ネットワークに送信する取引で閾値 ECDSA の署名をリクエストして UTXO を消費することができます。閾値 ECDSA は、IC の Chain Key 暗号のツールボックスのプロトコルを拡張したものです。IC の閾値 ECDSA プロトコルの詳細は、[こちらの](../t-ecdsa/t-ecdsa-how-it-works.md) 閾値 ECDSA のドキュメントページで見ることができます。

プロトコルレベルの Bitcoin 統合と閾値 ECDSA プロトコルは、それぞれマネージメント Canister 上で API を公開しています。それらの API はエンジニアが IC 上で Bitcoin のスマートコントラクトを記述するために使用するシステムレベルの API です。 Bitcoin の UTXO とトランザクションの概念を中心に設計された低レベルの API であり、その使用は簡単なものではなく、Bitcoin の仕組みについて深い理解が必要とされます。

次に、Bitcoin との直接統合の背後にある上記の技術の大まかな概要を説明します。詳細は、[Bitcoin page on the Internet Computer Wiki](https://wiki.internetcomputer.org/wiki/Bitcoin_integration) および [threshold ECDSA documentation page](../t-ecdsa/t-ecdsa-how-it-works.md) を参照してください。

## プロトコルレベルでの IC と Bitcoin ネットワークとの統合

Internet Computer プロトコルと Bitcoin プロトコルを統合し、2つのネットワーク間の直接的な技術統合が可能になりました。この統合は、任意の数の Internet Computer サブネットでアクティブにすることができます。まず最初は 1つの専用の Bitcoin-activated サブネットのみが存在することになり、任意のサブネット上の Canister からの Bitcoin API へのリクエストは、この単一の Bitcoin-activated サブネットにルーティングされることになります。この統合は、2つの重要な目的を果たすものです：
-    Bitcoin の UTXO セットを取得し、Internet Computer の複製されたステートに含めてチェーン上に保持し、Canister が発行した Bitcoin アカウントが行う UTXO セットと残高の問い合わせにレスポンスができるようにすること。
-   Canister の署名付き Bitcoin トランザクションを受け入れ、 Bitcoin ネットワークに送信すること。

![Bitcoin Integration](../_attachments/bitcoin_integration.png)

**コンポーネント**

 Bitcoin が有効なサブネットでは、マネージメント Canister の API を介して、マネージメント Canister の一部として、すなわちレプリカの一部として実装された *BTC Canister*（Bitcoin Canister）が、Canister にアクセスできるようにされています。BTC Canister は、オンチェーンの Bitcoin 関連のステート、すなわち UTXO セット、フォークの解決を可能にする最新の Bitcoin ブロック、および外に出ていくトランザクションを保持します。

ネットワーク層の * Bitcoin アダプター*は、通常の Bitcoin ノードと同じように、Bitcoin ネットワークのノードに接続します。

** Bitcoin UTXO セットの維持管理**

BTC Canister とアダプターは統合されており、IC のプロトコルスタックを介して相互に通信します。BTC Canister は、受け取った最新の Bitcoin ブロックの後継ブロックを Bitcoin アダプターに要求します。サブネットの各レプリカのアダプターは、要求されたブロックを Bitcoin ネットワークから取得し、ブロック作成レプリカのアダプターは、要求されたブロックをコンセンサスを経て Bitcoin Canister に提供します。 Bitcoin Canister は、プルーフ・オブ・ワークを検証し、ブロックのトランザクションを抽出し、トランザクションから UTXO を抽出し、複製されたステートに維持されている UTXO セットを更新するという形式でブロックを処理して、消費された UTXO とトランザクションによって作成された UTXO らを反映させます。UTXO セットと、まだ UTXO セットに吸収されていない最近のブロックのセットは、UTXO と残高に対するリクエストに応答するために使用されます。

フォークを安全に解決したり様々な種類の攻撃から保護するために、セキュリティの領域において非常に複雑な実装になっています。例えば、フォークを解決できるようにするために、Bitcoin Canister は UTXO セットにまだ吸収されていない Bitcoin ブロックを一定数維持する必要があります。しかし、与えられたアドレスの UTXO を計算するためには、UTXO セット内のものに加えて、吸収させずに維持しているブロック内にある UTXO を効率的に考慮しなければなりません。

**トランザクションを送信する**

Canister は、対応するマネージメント Canister  API を使用して、Bitcoin トランザクションを送信することができます。そうすることで、Bitcoin ネットワークに送信されるためのトランザクションをキューに入れます。すべてのサブネットラウンドで、アダプターは Bitcoin Canister から保留中のトランザクションを取得し、Bitcoin ネットワークに送信するために非同期にキューに入れます。サブネットの各レプリカが Bitcoin ネットワークの複数の接続されたノードを介してトランザクションを送信するので、Bitcoin ネットワーク内のトランザクションの効率的で迅速なディストリビューションに繋がります。

## 革新的な閾値 ECDSA プロトコル

閾値 ECDSA とは、閾値暗号を用いた ECDSA 署名プロトコルの実装のことです。閾値 ECDSA プロトコルでは、ECDSA 秘密鍵は複数の当事者間で秘密共有され、当事者のうち適格な定足数のみがそれぞれの秘密鍵シェアを使って署名を生成することができます。秘密鍵は復元された形では決して存在せず、秘密共有された形でのみ存在します。鍵の生成により、各当事者の秘密鍵シェアが生成されます。

Internet Computer の Chain Key 暗号ツールボックスの一部として実装された閾値 ECDSA プロトコルは、単一のマスター秘密鍵を使用し、そこから BIP-32 ライクな鍵派生を使用して Canister の鍵を派生させることができます。各 Canister は、Canister ID を使用して導き出された1つのルート鍵を持っており、BIP-32 の後方互換性のある拡張を使用して任意の数の追加の Canister 鍵を導き出すことができます。

Internet Computer 上の閾値 ECDSA が最初にデプロイされるのは、Canister の署名要求に答える1つの署名サブネットです。署名サブネットでは鍵を管理する Canister のみがこの鍵での署名を要求できます。すべての API コールは、ECDSA サブネットに到達するために Xnet トラフィックを通過する必要があり、それに応じて余分なレイテンシーが発生します。

Canister は、自分または他の Canister の公開鍵（Canister のさらなる派生公開鍵を含む）を問い合わせることができます。Canister は自身が管理する秘密鍵、すなわちルート秘密鍵や派生秘密鍵による署名を要求できます。公開鍵や派生鍵による署名の要求には、それぞれの API で派生パスを指定するか、BIP-32 を使用して派生させることができます。閾値 ECDSA の詳細は [こちら](../t-ecdsa/t-ecdsa-how-it-works.md) に記載されています。

## デプロイメント・アーキテクチャ

Bitcoin 機能は IC の単一サブネット上で起動され、Canister から Bitcoin API への API コールは Xnet 通信を経由するため、余分なレイテンシーが発生します。将来的には、Xnet のレイテンシーを回避し、時間単位でより多くのリクエストに対応するために、必要に応じてアプリケーションのサブネットでもこの機能が有効になる可能性があります。

閾値 ECDSA の要求には、1つのアクティブなサブネットが等しく答え、別のサブネットはディザスタリカバリのために秘密鍵シェア形式で秘密鍵をバックアップします。

## API

Bitcoin 統合により、以下のマネージメント Canister の API が利用可能になります（閾値 ECDSA APIについては、[そのドキュメントページ](../t-ecdsa/t-ecdsa-how-it-works.md)と[インターフェース仕様](../../../references/ic-interface-spec.md)で説明しています）。Bitcoin 関連の各メソッドは、Bitcoin `mainnet` と `testnet` のどちらを使用するかを指定する必要があります。

-   `bitcoin_get_utxos`:  `get_utxos_request` が与えられると（Bitcoin アドレスと Bitcoin ネットワーク（メインネットまたはテストネット）を指定する必要があります）、Bitcoin コンポーネントで有効になっている Bitcoin ブロックチェーンの現在のビューに基づいて、この関数は指定した Bitcoin ネットワーク内の指定したアドレスに関するすべての未使用トランザクションアウトプット（UTXOs）を返します。UTXO はブロックの高さの降順でソートされて返されます。<br/>
オプションのフィルターパラメータを使用して、返される UTXO のセットを制限することができます。パラメータに最小の承認数を指定するか、または多くの UTXO を持つアドレスに対してページ分割されている場合はページへの参照を指定することができます。前者のケースでは、指定された数以上の承認数を持つ UTXO のみが返されます。つまり、承認数がこの回数より少ないトランザクションは考慮されません。言い換えれば、承認数が c の場合、少なくとも c 回承認されているトランザクションによって生じたアウトプットのうち、消費されてないものが返されます。ただし、返されたアウトプットは承認回数が c 回未満のトランザクションにはすでに消費されている可能性があります。<br/>
オプションのフィルターなしの `get_utxos_request` は、ブロックチェーン全体を考慮したリクエストになり、これは `min_confirmations` を 0 に設定することと同じです。
-  `bitcoin_get_balance`: Bitcoin アドレスと Bitcoin ネットワーク (メインネットまたはテストネット) を指定する `get_balance_request` を受け取ると、この関数は、指定した Bitcoin ネットワークにおけるこのアドレスの現在の残高を Satoshi (10<sup>8</sup> Satoshi = 1 Bitcoin) で返します。`bitcoin_get_utxos` と同じアドレス形式がサポートされています。
-  `bitcoin_send_transaction`: `send_transaction_request` を与えると（Bitcoin トランザクション の Blob と Bitcoin ネットワーク（メインネットまたはテストネット）を指定する必要があります）、いくつかのチェックが行われ、成功すればトランザクションは Bitcoin ネットワークに転送されます。
-  `bitcoin_get_current_fee_percentiles`：Bitcoin ネットワークの取引手数料は、保留中の取引数に基づいて動的に変化します。Bitcoin 取引を作成する際に、Canister が適切な手数料を決定することが可能でなければなりません。<br/>
この関数は、過去10,000件の取引、つまり過去約4～10ブロックの取引について、millisatoshi/byte (10<sup>3</sup> millisatoshi = 1 satoshi) で測定した手数料のパーセンタイルを返します。これは通常、支払われるべき手数料の確かな指標となりますが、手数料のパーセンタイルの計算では Bitcoin mempool を考慮しないことに注意してください。

 Bitcoin 連携 API の詳細については、[インタフェイス仕様](../../../references/ic-interface-spec.md) を参照してください。

## 開発環境・プレプロダクション環境・プロダクション環境

閾値 ECDSA を含む Bitcoin の機能は、IC 上の開発ライフサイクルに必要なすべてのステージで利用可能です：
-   Canister をローカルに開発するための *SDK*。
-   *Bitcoin テストネット*での最終テストのためのプレプロダクション環境としての IC サポート。
-   *Bitcoin メインネット*を使用したリリースのためのプロダクション環境としての IC サポート。

### ローカル SDK

一般的な Canister の開発ワークフローでは、IC 上の Canister は、開発時に Motoko または Rust の Canister SDK を使ってローカル環境でコンパイル・実行されます。このように、SDK は開発ワークフローの最初のステージ、あるいは環境となります。この SDK は Bitcoin 統合 Canister API と閾値 ECDSA マネージメント Canister API の両方に対応できるようになっています。

それぞれ Bitcoin テストネットと Bitcoin メインネットと統合する機能の IC デプロイとは対照的に、SDK はリグレッションテスト（regtest）モードでローカルに動作する bitcoind ノードと統合されています。regtest モードで bitcoind を使用することは、Bitcoin の開発において好ましい方法です。開発者をできる限り手助けするために、私たちは SDK を regtest モードの bitcoind と統合し、IC に最高の Bitcoin 開発体験をもたらしました。このセットアップにより、スマートコントラクトの開発と自動テストの両方が、まずローカル環境で行われます。

ローカル SDK 環境を実行しているシングルレプリカの Bitcoin アダプターは、Bitcoin テストネットまたは メインネットの複数のノードではなく、ローカルの bitcoind ノードに接続します。dfx の関連フラグを確認するには、`dfx start —help` の出力を見てください。

### IC テストネット上の Bitcoin

スマートコントラクトが受け入れテストの準備ができると、IC 上にデプロイされます（パブリックな IC テストネットがないことを思い出してください）。Bitcoin テストネットに接続するために Bitcoin API セットを使用します。これはローカル開発で使用するのと同じ設定です。この設定では Dapp の受け入れテストを実行するために Bitcoin テストネットを使用します。つまり、実際に価値のあるものは危険にさらされません。

### IC メインネット上の Bitcoin

Bitcoinのスマートコントラクトの開発の最終段階は、Bitcoin API が設定された IC 上で Bitcoin メインネットを使用するデプロイメントです。これがスマートコントラクトの最終的な本番環境となります。これはまだ利用できませんが、GA リリースで有効化される予定です。

## 機能を利用する

Bitcoin を使って独自のアプリを作り始めるには、以下のチュートリアルをご覧ください：

[はじめてのビットコイン Dapp をデプロイする](../../../samples/deploying-your-first-bitcoin-dapp.md)

[Local Development](./local-development.md)

<!--
# How Bitcoin Integration Works — Technology Background

Bitcoin-enabling the IC has required us to solve two advanced engineering challenges: (1) A protocol-level integration of the IC with the Bitcoin network and (2) a novel threshold ECDSA protocol.

**Protocol-level integration of the IC with the Bitcoin network**

Through the protocol-level integration of the IC with the Bitcoin network, the IC can obtain Bitcoin blocks directly from the Bitcoin network and process the contained transactions. This allows for maintaining the full Bitcoin UTXO set on-chain on the IC. Canisters can run queries against the full Bitcoin UTXO set. This allows canisters to know about the held UTXOs, and thus balance, of any Bitcoin address, including their own addresses.

**Novel threshold ECDSA protocol**

Canisters themselves can have ECDSA keys using a novel threshold ECDSA protocol, and so can receive and hold Bitcoin. Canisters also can create Bitcoin transactions and submit them via the Bitcoin API to the Bitcoin network. They use the threshold ECDSA functionality to request threshold ECDSA signatures to spend UTXOs in a transaction to be submitted to the Bitcoin network. Threshold ECDSA is an extension of the IC's Chain Key cryptography toolbox of protocols. Details regarding the IC's threshold ECDSA protocol can be found on the threshold ECDSA documentation page [here](../t-ecdsa/t-ecdsa-how-it-works.md).

The protocol-level Bitcoin integration and threshold ECDSA protocol each expose an API on the management canister. Those APIs are the system-level APIs engineers use to write Bitcoin smart contracts on the IC. The APIs are low-level APIs designed around the concepts of Bitcoin UTXOs and transactions and are non-trivial to use and require an in-depth understanding of how Bitcoin works.

We next give a high-level overview of the abovementioned technology behind the direct Bitcoin integration. For details, we refer the reader to the [Bitcoin page on the Internet Computer Wiki](https://wiki.internetcomputer.org/wiki/Bitcoin_integration) as well as the [threshold ECDSA documentation page](../t-ecdsa/t-ecdsa-how-it-works.md).

## Protocol-level Integration of the IC with the Bitcoin Network

We integrated the Internet Computer Protocol with the Bitcoin protocol to obtain a direct technical integration between the two networks. This integration can be activated on any number of Internet Computer subnets. At the beginning, there will only be one dedicated Bitcoin-activated subnet and requests to the Bitcoin API from canisters on any subnet will be routed to this single Bitcoin-activated subnet. The integration serves two key purposes:
-   Obtaining the Bitcoin UTXO set and keeping it on chain in the replicated state of the Internet Computer to be able to answer queries for UTXO sets and balances of Bitcoin accounts issued by canisters.
-   Accepting signed Bitcoin transactions of canisters and submitting them to the Bitcoin network.

![Bitcoin Integration](../_attachments/bitcoin_integration.png)

**Components**

On a Bitcoin-activated subnet, a *BTC canister* (Bitcoin canister), implemented as part of the management canister, i.e, as part of the replica, is made accessible to canisters via an API of the management canister. The BTC canister holds the on-chain Bitcoin-related state: the UTXO set, the most recent Bitcoin blocks to allow for fork resolution, and outgoing transactions.

A *Bitcoin adapter* process at the networking layer connects to nodes of the Bitcoin network, much like a regular Bitcoin node does.

**Maintaining the Bitcoin UTXO set**

The BTC canister and adapter are integrated and communicate with each other via the IC's protocol stack: The BTC canister requests successor blocks to the latest Bitcoin blocks it has received from the Bitcoin adapter. The adapter on each replica of the subnet obtains the requested blocks from the Bitcoin network and the adapter of a block-making replica provides a requested block to the Bitcoin canister through consensus. The Bitcoin canister processes blocks in that it validates the proof of work, extracts the block's transactions, extracts the UTXOs from the transactions, and updates the UTXO set maintained in replicated state to reflect the UTXOs consumed and the UTXOs created by the transactions. The UTXO set and a set of recent blocks not yet absorbed into the UTXO set are used to respond to UTXO and balance requests.

Significant complexity of the implementation is in the area of securely resolving forks and protecting against various kinds of attacks. For example, in order to be able to resolve forks, the Bitcoin canister needs to maintain a certain number of Bitcoin blocks which are not yet absorbed into the maintained UTXO set, but UTXOs therein must be efficiently considered for calculating the UTXOs of a given address in addition to those in the UTXO set.

**Submitting transactions**

Canisters can submit Bitcoin transactions to the Bitcoin canister using the corresponding management canister API. Doing so queues the transactions for being submitted to the Bitcoin network. In every subnet round, Adapters obtain the pending transactions from the Bitcoin canister and queue them for being submitted asynchronously to the Bitcoin network. This leads to an efficient and quick distribution of transactions in the Bitcoin network as every replica of the subnet submits transactions via multiple connected nodes of the Bitcoin network.

## Novel Threshold ECDSA Protocol

Threshold ECDSA refers to the implementation of the ECDSA signature protocol using threshold cryptography. In a threshold ECDSA protocol, the private ECDSA key is secret shared between multiple parties and only an eligible quorum of the parties can generate a signature using their respective private key shares. The private key never exists in reconstructed form, but only in its secret-shared form. Key generation generates private key shares for the parties.

The threshold ECDSA protocol implemented as part of the Internet Computer's Chain Key cryptography toolbox uses a single master private key from which keys for canisters can be derived using BIP-32-like key derivation. Each canister has one such root key derived using the canister id and an arbitrary number of additional canister keys can be derived using a backward-compatible extension of BIP-32.

Threshold ECDSA on the Internet Computer is deployed on one signing subnet initially that will answer signing requests of canisters. The signing subnet enforces that only the canister that controls a key may request signatures with this key. All API calls need to go through Xnet traffic to reach the ECDSA subnet and thus incur extra latency accordingly.

Canisters can query their own or other canisters' public keys, including further derived public keys of canisters. Canisters can request signatures with private keys they control, i.e., their root private key and derived private keys. For requests of public keys or signatures with derived keys, a derivation path can be specified in the respective API or the derivation can be done using BIP-32. You can find more details on threshold ECDSA [here](../t-ecdsa/t-ecdsa-how-it-works.md).

## Deployment Architecture

The Bitcoin functionality will be activated on a single subnet of the IC and API calls from canisters to the Bitcoin API are routed via Xnet communication, thus extra latency is incurred. The feature may, if needed, in the future be additionally activated on (some) application subnets to avoid the additional Xnet latency and to be able to respond to more requests per time unit.

Threshold ECDSA requests will equally be answered by a single active subnet, another subnet will back up the private key in secret-shared form for disaster recovery.

## API

The Bitcoin integration makes the following management canister APIs available to canisters (the threshold ECDSA API is explained in [its documentation page](../t-ecdsa/t-ecdsa-how-it-works.md) and the [interface specification](../../../references/ic-interface-spec.md)). Each Bitcoin-related method needs to specify whether it uses Bitcoin `mainnet` or `testnet`.

-   `bitcoin_get_utxos`: Given a `get_utxos_request`, which must specify a Bitcoin address and a Bitcoin network (mainnet or testnet), the function returns all unspent transaction outputs (UTXOs) associated with the provided address in the specified Bitcoin network based on the current view of the Bitcoin blockchain available to the Bitcoin component. The UTXOs are returned sorted by block height in descending order.<br/>
The optional filter parameter can be used to restrict the set of returned UTXOs, either providing a minimum number of confirmations or a page reference when pagination is used for addresses with many UTXOs. In the first case, only UTXOs with at least the provided number of confirmations are returned, i.e. transactions with fewer than this number of confirmations are not considered. In other words, if the number of confirmations is c, an output is returned if it occurred in a transaction with at least c confirmations and there is no transaction that spends the same output with at least c confirmations.<br/>
A `get_utxos_request` without the optional filter results in a request that considers the full blockchain, which is equivalent to setting `min_confirmations` to 0.
-   `bitcoin_get_balance`: Given a `get_balance_request`, which must specify a Bitcoin address and a Bitcoin network (mainnet or testnet), the function returns the current balance of this address in Satoshi (10<sup>8</sup> Satoshi = 1 Bitcoin) in the specified Bitcoin network. The same address formats as for `bitcoin_get_utxos` are supported.
-   `bitcoin_send_transaction`: Given a `send_transaction_request`, which must specify a blob of a Bitcoin transaction and a Bitcoin network (mainnet or testnet), several checks are performed, and, if successful, the transaction is forwarded to the Bitcoin network.
-   `bitcoin_get_current_fee_percentiles`: The transaction fees in the Bitcoin network change dynamically based on the number of pending transactions. It must be possible for a canister to determine an adequate fee when creating a Bitcoin transaction.<br/>
This function returns the 100 fee percentiles, measured in millisatoshi/byte (10<sup>3</sup> millisatoshi = 1 satoshi), over the last 10,000 transactions, i.e., over the transactions in the last approximately 4-10 blocks. Please note that this usually gives a solid indication of the fees to be paid, but we do not consider the Bitcoin mempool in the computation of the fee percentiles.

We refer to the [Internet Computer Interface Specification](../../../references/ic-interface-spec.md) for the details of the Bitcoin integration API.

## Development, Pre-Production, and Production Environment

The Bitcoin functionality including threshold ECDSA is available in all stages required for the development life cycle on the IC:
-   The *SDK* for local development of canisters;
-   IC support as the pre-production environment for final testing on *Bitcoin testnet*;
-   IC support as the production environment for the release using *Bitcoin mainnet*.

### Local SDK

In the typical canister development workflow, canisters on the IC are compiled and run in the local environment using the Motoko or Rust canister SDK during their development. Thus, the SDK is the first stage, or environment, of the development workflow. The SDK has been enabled to support both the Bitcoin integration and threshold ECDSA management canister APIs.

In contrast to the IC deployments of the feature, which integrate with Bitcoin Testnet and Bitcoin Mainnet, respectively, the SDK integrates with a locally-running bitcoind node in regression testing (regtest) mode. Using bitcoind in regtest mode is the preferred way for Bitcoin development. To facilitate our developers as best as possible, we integrated the SDK with bitcoind in regtest mode to bring the best Bitcoin development experience to the IC. Both development and automated testing of smart contracts are first done in the local environment with this setup.

The Bitcoin adapter of the single replica running the local SDK environment connects to the local bitcoind node instead of multiple nodes of Bitcoin Testnet or Mainnet. To see the relevant flags on dfx, please look at the output of `dfx start --help`.

### Bitcoin Testnet on the IC

Once a smart contract is ready for acceptance testing, it is deployed on the IC (recall, there is no public IC testnet), still using the Bitcoin API set to connect to the Bitcoin testnet. This is the same setting as used in local development. This setup is used to perform the acceptance testing of the dApp, using the Bitcoin testnet, i.e., no real value is at stake.

### Bitcoin Mainnet on the IC

The final stage of development of a Bitcoin smart contract is its deployment on the IC with the Bitcoin API set to use Bitcoin Mainnet. This is the final production environment for the smart contract. This is not yet available, but will be activated with the GA release.

## Working with the Feature

To start building your own apps with Bitcoin see the following tutorials:

[Deploying Your First Bitcoin Dapp](../../../samples/deploying-your-first-bitcoin-dapp.md)

[Local Development](./local-development.md)

-->
