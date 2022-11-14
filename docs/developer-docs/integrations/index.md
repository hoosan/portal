# 機能統合について

これまでのセクションでは、IC 上で Canister を作り始める方法を案内してきましたが、ここでは、様々な（時には最先端の）追加機能を Dapps に統合する方法を紹介します。

## Bitcoin 統合
Bitcoin ネットワークと直接統合することで、IC 上の Canister が Bitcoin を受け取り、保有し、送信することを、すべてBitcoin ネットワーク上のトランザクションで直接行うことができます。つまり、Canister は、Bitcoin ネットワーク上で Bitcoin を保有する一般ユーザーと全く同じように行動することができます。

* [Bitcoin 統合](./bitcoin/index.md)では長めに概要を説明しています。
* [Bitcoin 統合の仕組み ー 技術的背景](./bitcoin/bitcoin-how-it-works.md)ではより詳細な説明をしています。
* [Bitcoin Dapps をローカルで開発する](./bitcoin/local-development.md)ではローカル環境での開発の仕方を説明しています。

## 閾値 ECDSA
ブロックチェーン上の閾値 ECDSA 実装は、秘密鍵を安全に保管し、適格なエンティティの要求に応じて、それらにのみ署名を発行することができ、これはハードウェアセキュリティモジュール（HSM）のオンチェーン版と見なすことができます。特に（ECDSA ベースの）ブロックチェーンとの直接統合を促進する際に重要になります。

* [閾値 ECDSA](./t-ecdsa/index.md)では、この機能で実現できることの概要を説明しています。
* [技術概要 - 仕組み](./t-ecdsa/t-ecdsa-how-it-works.md)ではより詳細に説明しています。

## HTTPS アウトコール
IC 上の HTTP(S) アウトコールにより、 Canister はブロックチェーンの外部の HTTP(S) サーバーに直接コールし、処理をした上でそのレスポンスを Canister で使用し、この入力を使って（コンセンサス済みの）複製されたステートを安全に更新できるようになります。ブロックチェーン史上初の試みであり、オラクルの必要性を軽減するものです。

* [Canister HTTPS アウトコール](./http_requests/index.md)は、IC と外部との通信の仕組みの概要を説明します。
* [技術概要 - Canister HTTPS アウトコール の仕組み](./http_requests/http_requests-how-it-works.md)はより詳細に説明し、オラクルとの比較も行っています。

## Internet Identity
Internet Identity は、Web3 サービスや Dapps とのセッションを作成したり、従来のブロックチェーン取引に署名したりすることができます。
* [Internet Identity](./internet-identity/integrate-identity.md) では、アプリで Internet Identity を使用する方法の概要を説明します。

## ICP 台帳
Internet Computer プロトコル（ICP）は、台帳 Canister と呼ばれる特殊な Canister を使用して ICP の管理を実装しています。台帳 Canister はひとつで、NNS のサブネット上で他の Canister と一緒に稼働します。台帳 Canister は、アカウントとトランザクションを保持するスマートコントラクトです。 

* [ICP 台帳](./ledger/index.md)では、 ICP 台帳の基礎を説明しています。
* [ICP 台帳との連携](./ledger/interact-with-ledger.md)では、ICP 台帳を操作するためのコマンドとプロトコルフローを示します。
* [台帳のローカルセットアップ](./ledger/ledger-local-setup.md)では、ローカル環境での台帳の利用方法を説明しています。
* [新しいトークンのデプロイ](./ledger/deploy-new-token.md)では、新しいトークンをデプロイして、Rosetta に接続する方法を説明しています。

## Rosetta
Rosetta は、Coinbase が導入したオープンスタンダードで、取引所、ブロックエクスプローラー、ウォレットにおけるブロックチェーンベースのトークン統合を簡素化するためのものです。このドキュメントは、CeFi 取引所で取引可能なことを目的としたトークンを IC 上にデプロイしたい場合や、ブロックエクスプローラーやウォレットに取り組んでいる場合に役立つはずです。

* [Rosetta](./rosetta/index.md) では、Rosetta ノードの設定方法とよくある質問に対する回答をしています。
* [スタンダードトークンを送金する](./rosetta/transfers.md)では、Rosetta Construction API を使用した ICP の転送方法について説明しています。
* [Neuron のライフサイクル](./rosetta/neuron-lifecycle.md)では、Neuron の概要を説明します（Neuronとは、コントローラーがプロポーザルの提出や投票によってネットワークのガバナンスに参加することを可能とする IC のアセットです）。
* [ステーキングと Neuron 管理](./rosetta/staking-support.md)では、Rosetta API を拡張し、IC 上で資金のステーキングやガバナンス Neuron の管理を可能にすることについて説明しています。
* [Rosetta ステーキングのチュートリアル](./rosetta/staking-tutorial.md)では、Neuron を作成するプロセスを説明しています。
* [ホットキー](./rosetta/hotkeys.md)では、Neuron 管理用ホットキーの生成方法について説明しています。

<!--
# Functionality Integrations

While previous sections guide you to start building canisters on the IC, here you can see how to integrate various (sometimes advanced) extra functionality to your dapp.

## Bitcoin Integration
Integrate directly with the Bitcoin network allowing canisters on the IC to receive, hold, and send Bitcoin, all directly with transactions on the Bitcoin network. I.e., canisters can act exactly like regular users holding bitcoin on the Bitcoin network.

* [Bitcoin Integration](./bitcoin/index.md) gives a longer overview
* [How it works](./bitcoin/bitcoin-how-it-works.md) dives further into the details
* [local development](./bitcoin/local-development.md) contains a tutorial showing how to experiment locally

## Threshold ECDSA
A threshold ECDSA implementation on a blockchain can be viewed as the on-chain pendant to a hardware security module (HSM) that stores private keys securely and issues signatures on request of the eligible entities, and only to those. It is particularly important to facilitate direct integration with (ECDSA-based) blockchains.

* [Threshold ECDSA](./t-ecdsa/index.md) gives an overview of what can be achieved with this feature
* See [How it works](./t-ecdsa/t-ecdsa-how-it-works.md) to dive further into the details

## HTTPS Outcalls
HTTP(S) outcalls on the IC enable canisters to directly make calls to HTTP(S) servers external to the blockchain and use the response in the further processing of the canister such that the replicated state can safely be updated using those inputs. A first in blockchain history, and alleviates the need for oracles.
* [HTTPS outcalls](./http_requests/index.md) gives an overview of how the IC can communicate with the world outside
* [How it works](./http_requests/http_requests-how-it-works.md) to dive further into the details and gives a comparison against oracles

## Internet Identity
Internet Identity allows users to create sessions with Web3 services and dapps, and sign traditional blockchain transactions.
* [Internet Identity](./internet-identity/integrate-identity.md) gives an overview of how to use internet identity in your app

## ICP Ledger
The Internet Computer Protocol (ICP) implements management of ICP using a specialized canister, called the ledger canister. There is a single ledger canister which runs alongside other canisters on the NNS subnet. The ledger canister is a smart contract that holds accounts and transactions. 

* [Ledger overview](./ledger/index.md) to get a view of the ICP ledger basics
* [Interact](./ledger/interact-with-ledger.md) shows the commands and protocol flows to interact with the ICP ledger
* [Local Setup](./ledger/ledger-local-setup.md) shows how to experiment with the ledger in your local environment
* [Deploy New Token](./ledger/deploy-new-token.md) describes how to deploy a new token and connect to Rosetta

## Rosetta
Rosetta is an open standard introduced by Coinbase to simplify the integration of blockchain-based tokens in exchanges, block explorers, and wallets. This documentation might help if you want to deploy a token on the IC that aims to be tradable on CeFi exchanges or if you are working on a block explorer or wallet.
* The [Rosetta page](./rosetta/index.md) describes how to set up a Rosetta node and answers some FAQs
* [transfers](./rosetta/transfers.md) details how to transfer ICP using the Rosetta Construction API
* [neuron lifecycle](./rosetta/neuron-lifecycle.md) gives an overview of neurons (IC assets allowing controllers to participate in the governance of the network by submitting and voting on proposals)
* [staking support](./rosetta/staking-support.md) specifies extensions of the Rosetta API enabling staking funds and managing governance neurons on the IC
* [staking tutorial](./rosetta/staking-tutorial.md) walks through the process of creating a neuron
* [hotkeys](./rosetta/hotkeys.md) explains how to generate a hotkey for neuron management

-->