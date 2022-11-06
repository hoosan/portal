# ICP 台帳との連携



## コマンドラインから ICP 台帳を操作する

DFX は、ICP 台帳 Canister と関連する機能を操作するための便利なコマンドを提供します。ドキュメントは[こちら](https://internetcomputer.org/docs/current/references/cli-reference/dfx-ledger/) を参照してください。また、以下のコマンドをコンソールに入力するだけでも利用できます：

``` bash
dfx ledger --help
```

サブコマンドの `--help` フラグもチェックしておくとよいでしょう。

現在、dfx は ICP の台帳機能の一部、すなわち `balance` と `transfer` のみを公開しています。
どちらのコマンドも、台帳の Canister ID を指定するフラグを提供します (`--ledger-canister-id`) 。これにより、ローカルの台帳の デプロイや、同じインターフェイスを提供する他のトークンとのやりとりを簡略化することができます。

#### 残高

特定のアカウントの ICP 残高を取得します：
``` bash
dfx ledger --network ic balance <account-id>
```
`<account-id>` は hex string（16進数文字列）でエンコードされています。
多くの場合、特定の本人のメインアカウントの残高を確認したいものです。この場合、以下のコマンドを使用することができます。

``` bash
dfx ledger --network ic balance $(dfx ledger account-id --of-principal <principal-id>)
```

#### 送金

送金機能は、自分のアカウントから他のアカウントに ICP を転送するために使用することができます。

``` bash
dfx ledger --network ic transfer --amount <amount> --memo <memo> <receiver-account-id>
```


<!-- ## Interact with ICP using Candid UI -->

## Web アプリケーションから ICP 台帳と連動する

JavaScript アプリケーションから ICP 台帳を簡単に操作するために、[nns-js ライブラリ](https://github.com/dfinity/nns-js)を使用することができます。

## Canister からの ICP を操作する

[ICP 送金サンプル](/docs/current/samples/token-transfer)は、Canister から ICP 台帳を操作するための良い出発点となります。この例では、Motoko と Rust で `balance` と `transfer` を使用する方法を紹介しています。

### ICPの受け取り

ICP で Canister が支払いを受けるようにするには、Canister が支払いについて知っていることを確認する必要があります。送金は送信者と台帳 Canister にしか関与しないからです。

これを実現するために、現在、主に2つのパターンがあります。さらに、台帳とトークナイゼーションに関する[Chartered Working Group](https://forum.dfinity.org/t/announcing-technical-working-groups/11781) があり、台帳スタンダード・トークン・インターフェースとペイメントフローの定義に重点を置いています。

#### 送信者による直接通知

このパターンでは、送信者は受信者に支払いについて通知します。しかし、受信者は台帳の[`query_blocks` インターフェース](/docs/current/references/ledger#_getting_ledger_blocks)を使って支払いを確認する必要があります。
次の図は、このパターンを単純化したものです。

```plantuml
    participant Sender
    participant "ICP Ledger"
    participant Receiver

    Sender -> "ICP Ledger": transfer()
    "ICP Ledger" --> Sender: blockNumber
    Sender -> Receiver: notify(blockNumber)
    Receiver -> "ICP Ledger": query_blocks(blockNumber)
    "ICP Ledger" --> Receiver: block
    Receiver -> Receiver: verify payment
```


#### ICP 台帳による通知(現在は無効)

このパターンでは、台帳自身が受信者に通知します。これにより、受信者はその通知を即座に信用することができます。ただし、受信者への呼び出しが一方通行で実装されていないため、現状、このフローは無効です。

```plantuml
    participant Sender
    participant "ICP Ledger"
    participant Receiver

    Sender -> "ICP Ledger": transfer()
    "ICP Ledger" --> Sender: blockNumber
    Sender -> "ICP Ledger": notify(blockNumber, receiver)
    "ICP Ledger" --> "Receiver": transaction_notification(details)
```

<!--
# Interact with the ICP ledger



## Interact with ICP ledger from the command line

Dfx provides a convenience command to interact with the ICP ledger canister and related functionality. You can find the documentation [here](https://internetcomputer.org/docs/current/references/cli-reference/dfx-ledger/) or just enter the following command into your console:

``` bash
dfx ledger --help
```

It's worth checking out the `--help` flag of the subcommands as well.

Currently, dfx exposes only a subset of the ICP ledger functionality, namely `balance` and `transfer`.
Both commands provide a flag to specify a ledger canister id (`--ledger-canister-id`). This simplifies interacting with a local ledger deployment or other tokens that provide the same interface.

#### Balance

Get the ICP balance of a specific account:
``` bash
dfx ledger --network ic balance <account-id>
```
The `<account-id>` is encoded as a hex string.
In many cases you want to check the main account balance of a specific principal. You can use the following command for this:

``` bash
dfx ledger --network ic balance $(dfx ledger account-id --of-principal <principal-id>)
```

#### Transfer

The transfer function can be used to transfer ICP from your account to another. 

``` bash
dfx ledger --network ic transfer --amount <amount> --memo <memo> <receiver-account-id>
```


<!-- ## Interact with ICP using Candid UI -->

<!--
## Interact with ICP ledger from your web application

In order to simplify working with ICP ledger from JavaScript applications, you can use the [nns-js library](https://github.com/dfinity/nns-js).

## Interact with ICP from a canister

The [ICP transfer example](/docs/current/samples/token-transfer) provides a good starting point for interacting with ICP  ledger from a canister. The example showcases the usage of `balance` and `transfer` in Motoko and Rust.

### Receiving ICP

If you want a canister to receive payment in ICP you need to make sure that the canister knows about the payment, because a transfer only involves the sender and the ledger canister.

There are currently two main patterns to achieve this. Furthermore, there is a [chartered working group](https://forum.dfinity.org/t/announcing-technical-working-groups/11781) on Ledger & Tokenization which is focused on defining a standard ledger/token interface as well payment flows.

#### Direct notification by sender

In this pattern the sender notifies the receiver about the payment. However, the receiver needs to verify the payment by using the [`query_blocks` interface](/docs/current/references/ledger#_getting_ledger_blocks) of the ledger.
The following diagram shows a simplified illustration of this pattern:

```plantuml
    participant Sender
    participant "ICP Ledger"
    participant Receiver
    
    Sender -> "ICP Ledger": transfer()
    "ICP Ledger" ==> Sender: blockNumber
    Sender -> Receiver: notify(blockNumber)
    Receiver -> "ICP Ledger": query_blocks(blockNumber)
    "ICP Ledger" ==> Receiver: block
    Receiver -> Receiver: verify payment
```


#### Notification by ICP ledger (Currently disabled)

In this pattern the ledger iteself notifies the receiver. Thereby, the receiver can trust the notification immediately. However, this flow is currently disabled because the call to the receiver is not yet implemented as a one-way call. 

```plantuml
    participant Sender
    participant "ICP Ledger"
    participant Receiver
    
    Sender -> "ICP Ledger": transfer()
    "ICP Ledger" ==> Sender: blockNumber
    Sender -> "ICP Ledger": notify(blockNumber, receiver)
    "ICP Ledger" ==> "Receiver": transaction_notification(details)
```

-->
<!--158, 161, 176, 178行目の==>はコメントアウト対策のためにーー＞（あえて全角にしています）から変更しました-->