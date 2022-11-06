# ICP 台帳


Internet Computer プロトコル（ICP）は、ユーティリティ・トークン（ティッカー “ICP"）の管理を、**台帳 Canister **と呼ばれる特殊な Canister を使って行います。Internet Computer の特別なサブネット（NNS サブネット）上で他の Canister と一緒に動作する単一の台帳 Canister があります。台帳 Canister は、**アカウント**と**トランザクション**を保持するスマートコントラクトです。これらの取引は、アカウントに** ICP トークンをミント**するか、あるアカウントから別のアカウントに **ICP トークンを送金**するか、ICP トークンを Cycle に変換する等で存在を消す** ICPトークンをバーン**するかのいずれかです。台帳 Canister は、genesis 時のステート（初期のステート）から始まる追跡可能な全ての取引履歴を維持します。

このセクションの構成は以下の通りです：

- このページでは、ICP と台帳の基本を紹介します。台帳 Canister の完全な公開インターフェースについてのより詳しい説明は、 [references section](/docs/current/references/) の [The Ledger canister](/docs/current/references/ledger) をご覧ください。
- [ICP 台帳との連携](interact-with-ledger.md) には、コマンドライン、JavaScript アプリケーション、 Canister から ICP 台帳を操作するための実践的なチュートリアルが用意されています。
- [台帳のローカルセットアップ](./ledger-local-setup.md) は、開発用にローカルレプリカに台帳 Canister をデプロイする手順を説明します。
- [新しいトークンのデプロイ](./deploy-new-token.md) では、カスタム台帳 Canister をデプロイして独自のトークンを作成し、それを [Rosetta API](../rosetta/) を介して取引所で利用できるようにする方法を説明しています。

## 基礎

### アカウント

アカウントは、Internet Computer（IC）Principal を保持しているアカウント所有者に属し、その所有者によって管理されます。2人以上の IC Principal によってアカウントが所有されることはありません（ジョイントアカウントはないということ）。しかし、Principal は Canister と同様に外部ユーザーを参照することができるため、ジョイントアカウントは Canister として実装することができます。

アカウント所有者は、複数のアカウントを管理することができます。この場合、各アカウントはペア (account_owner, sub_account) に対応します。サブアカウントはオプションのビット文字列で、同じ所有者の異なるサブアカウントを区別するのに役立ちます。

台帳上のアカウントはアドレスで識別され、そのアドレスは Principal ID とサブアカウント識別子から導かれます。

この文脈では、Principal ID は、ビットコインやイーサリアムのユーザーの公開鍵のハッシュと大まかに等しいと考えることができます。ユーザーは対応する秘密鍵を使ってメッセージに署名し、その結果、台帳 Canister を認証して Principal のアカウントで操作することになります。Canister は、台帳 Canister にアカウントを持つこともでき、その場合のアドレスは Canister の Principal に由来します。

台帳 Canister は、Internet Computer 内部の管理操作を使用して初期化されます。初期化プロセスの一部として、アカウントと関連する ICP トークン残高のセットで Canister が作成されます。

:::note なぜ台帳は Principal ID だけでなく、Account ID を使用するのですか？

アカウント導入の主な理由は、Principal が複数のアカウントを管理できるようにするためです。しかし、Canister の場合、ウォレットソフトウェアを利用すれば、このような問題を回避することができます。
:::

### トランザクションタイプ

台帳 Canister の内部ステートを変更する操作には、次の3つがあります：

-   **ICP トークンのアカウントでのミント**

-   **ICP トークンのアカウント間の送金**

-   **ICP トークンのバーン**

すべての操作は、台帳 Canister にトランザクションとして記録されます。

台帳は、ハッシュ化されたブロックチェーン、すなわち Canister スマートコントラクト（台帳 Canister）の内部で稼働するブロックチェーンとして取引を管理し、それが NNS サブネットブロックチェーン上で稼働しています。

ステートの変化が記録されると、それぞれの新しいトランザクションがブロックに配置され、一意のインデックスが割り当てられます。チェーン全体は、最新の（ブロック）チェーンのリンクに署名することで定期的に認証されます。チェーンを認証するために使用される署名は、Internet Computer のルート公開鍵にアクセスできる任意の第三者によって検証することができます。特定の取引は、台帳を照会することで取得することができます。


### ICP ユーティリティ・トークンの基本プロパティ

ICP トークンはビットコインなどの分散型ネットワークを司るユーティリティ・トークンに似ていますが、重要な点で異なるところもあります。

ICP トークンは、以下の点でビットコインと類似しています：

- 各 ICP トークンは小数点 10^8（第八位）まで単位を持ちます。

- すべての取引は、genesis の初期状態から始まる台帳に保存されます。

- トークンは完全にファンジブルです。

- アカウント識別子は 32バイトで、公開鍵のハッシュとほぼ等しく、オプションで追加のサブアカウント指定子もあります。

ICP トークンはビットコインと以下の点で異なります：

- ステークされた参加者ノードは、Proof of Work を使うのではなく、閾値 BLS 署名の一種を使い、チェーンの有効なステートに合意します。

- どんなトランザクションも 8バイトの memo を保管できます。この memo フィールドは Rosetta API によってトランザクションを区別するための nonce を格納するために使用されますが、このフィールドの他の用途も可能です。

<!--
# The ICP Ledger


The Internet Computer Protocol (ICP) implements management of its utility token (ticker "ICP") using a specialized canister, called the **ledger canister**. There is a single ledger canister which runs alongside other canisters on a special subnet of the Internet Computer - the NNS subnet. The ledger canister is a smart contract that holds **accounts** and **transactions**. These transactions either **mint ICP tokens** for accounts, **transfer ICP tokens** from one account to another, or **burn ICP tokens**, eliminating them from existence, e.g. while converting ICP tokens to cycles. The ledger canister maintains a traceable history of all transactions starting from its genesis state (initial state). 

This section is structured as follows:

- This page provides an introduction to the basics of ICP and the ledger. For a more detailed explanation of the complete public interface of the ledger canister head over to the [specification](/docs/current/references/ledger) in the [references section](/docs/current/references/).
- [Interact with the ICP ledger](interact-with-ledger.md) provides hands on tutorials to interact with the ICP ledger from the command line, JavaScript applications and from canisters.
- [Ledger Local Setup](./ledger-local-setup.md) walks you through the steps to deploy a ledger canister to your local replica for development.
- [Deploy New Token](./deploy-new-token.md) explains how you can create your own token by deploying a custom ledger canister and make it available to exchanges via the [Rosetta API](../rosetta/).

## The Basics

### Accounts

An account belongs to and is controlled by the account owner who must be an Internet Computer (IC) principal. No account can be owned by two or more IC principals (no "joint accounts"). However, since a principal can refer to an external user as well as to a canister, joint accounts can be implemented as canisters.

An account owner may control more than one account. In this case, each account corresponds to a pair (account_owner, sub_account). The sub-account is an optional bitstring which helps distinguish between the different sub-accounts of the same owner.

An account on the ledger is identified by its address, which is derived from the principal ID and sub-account identifier.

In this context, you can think of principal identifiers as a rough equivalent to the hash of a user’s public key for Bitcoin or Ethereum. You use the corresponding secret key to sign messages and therefore authenticate to the ledger canister and operate on the principal’s account. Canisters can also have accounts in the ledger canister, in which case the address is derived from the canister’s principal.

The ledger canister is initialized using administrative operations that are internal to the Internet Computer. As part of the initialization process, the canister is created with the set of accounts and associated ICP token balances.

:::note Why does the ledger use Account IDs and not just Principal IDs?

The main reason for introducing accounts was to allow a principal to control multiple accounts. While this could be abstracted away for a user by a the wallet software, this is not possible for canisters.
:::

### Transaction types

There are three operations that can change the internal state of the ledger canister:

-   **Minting ICP tokens** for accounts.

-   **Transferring ICP tokens** between accounts.

-   **Burning ICP tokens**.

All operations are recorded as transactions in the ledger canister.

The ledger maintains the transactions as a hashed blockchain, i.e., a blockchain running inside a cansiter smart contract (the ledger canister), which in turn is running on the NNS subnet blockchain.

As state changes are recorded, each new transaction is placed in a block and assigned a unique index. The entire chain is regularly authenticated by signing the latest chain link. The signature used to authenticate the chain can be verified by any third party who has access to the root public key of the Internet Computer. Specific transactions can be retrieved by querying the ledger.


### Basic properties for ICP utility tokens

The ICP token is similar to utility tokens governing decentralized networks such as Bitcoin, but also differs in important ways.

The ICP token is similar to Bitcoin in the following ways:

-   Each ICP token is divisible 10^8 times.

-   All transactions are stored in the ledger starting with the genesis initial state.

-   Tokens are entirely fungible.

-   Account identifiers are 32 bytes and are roughly the equivalent of the hash of a public key, optionally together with some additional sub-account specifier.

The ICP token differs from Bitcoin in the following ways:

-   Rather than using proof of work, staked participant nodes use a variant of threshold BLS signatures to agree on a valid state of the chain.

-   Any transaction can store an 8-byte memo — this memo field is used by the Rosetta API to store the nonce that distinguishes between transactions. However, other uses for the field are possible.

-->