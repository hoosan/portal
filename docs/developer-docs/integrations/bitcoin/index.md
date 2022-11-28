# Bitcoin 統合

Internet Computer は、Bitcoin ネットワークと直接統合されています。これにより、Internet Computer 上の Canister は、Bitcoin の受信、保有、送信をすべて Bitcoin ネットワーク上のトランザクションと直接的に行うことができます。つまり、Canister は、Bitcoin ネットワーク上で Bitcoin を保有する通常のユーザーと全く同じように利用することが可能になります。これらはすべて、（1）Internet Computer がプロトコルレベルで Bitcoin と統合し、（2）Canister が新しい閾値 ECDSA プロトコルによって ECDSA 鍵を安全に保持（および使用）できることによって可能になりました。Internet Computer は、このような他のブロックチェーンとの直接統合を行う最初のブロックチェーンネットワークの一つであり、この目的のために新しい技術基盤を構築しています。

この統合により、斬新なユースケースを数多く実現することができます：

-   *Bitcoin のスマートコントラクト：* Canister は Bitcoin ネットワーク上の Bitcoin を直接保持できるため、エンジニアは Canister を使用して強力な Bitcoin のスマートコントラクトを実装することが可能になります。どのような Canister のスマートコントラクトでも、Bitcoin のスマートコントラクト機能を提供できるようになります。例えば、ユーザーが秘密鍵を管理する必要のない生体認証によるオンチェーン Bitcoin ウォレットや、ソーシャル Dapps を使用してユーザーがピアツーピアで Bitcoin 取引を行うことができる socialFi などです。
-   Internet Computer 上の分散型取引所での *Bitcoin の取引*。
-   SNS を活用した DAO が IC 上のサービスを分散化する際に、Bitcoin を使用して *分散型セール (Decentralization Sale)* でトークンを購入すること。
-   *Chain Key Bitcoin （ckBTC）*は Wrapped Bitcoin の発展型で、Bitcoin メインネットの Bitcoin 機能のリリースに伴い、IC 上で利用可能になります。ckBTC は IC 上で最も簡単に Bitcoin を扱うことができ、IC 上の Bitcoin に興味を持つ多くの人にとって正しい選択になるかもしれません。なお、ckBTC は今後数カ月間の Bitcoin の一般提供（ GA ）リリースと、IC 上での Bitcoin メインネットのローンチによってのみ利用可能となる予定です。

これらは Bitcoin との連携機能のほんの一例に過ぎません。あなたの想像力が及ぶ限り、無限の可能性が開かれています。このドキュメントでは、Bitcoin を使用した独自の Dapp を実装するための機能の使用方法について説明します。また、Bitcoin 統合 API は、UTXO と Bitcoin トランザクションのレベルで動作する低レベルの API であり、使用は簡単ではないことに注意してください。

来たるべき Bitcoin 統合機能の Bitcoin メインネットリリース（一般提供リリース）の一部として、*Chain Key Bitcoin* (ckBTC) Canister が利用できるようになります。ckBTC Canister は、IC 上のオンチェーン Bitcoin を提供します。これは、一見、Wrapped Bitcoin のように感じられますが、分散型アーキテクチャであり、ブリッジの代わりに閾値 ECDSA を使用し、はるかに強い基礎トラストモデルを持っています。私たちは、いくつかの明確な利点があるため、ネイティブ統合を代用する形で多くの人々がプロジェクトに ckBTC を使うようになることを想定しています：
-   低レベルの Bitcoin 統合 API を使用する代わりに、Internet Computer 上の他の台帳と同様に ckBTC 台帳にアクセスするだけでよいのです。ckBTC の台帳は今後予定されている IC トークンの規格に準拠したファンジブル・トークンです。
-   *高速かつ安価な送金：* ckBTC は Internet Computer の低い確定時間（数秒以内）で、Bitcoin ネットワーク上の Bitcoin 送金に比べ、わずかなコストで送金することが可能です。この方式では、Bitcoin ネットワークとの決済送金のみ Bitcoin ネットワーク上で行う必要がありますが、大半の送金は IC 上で直接、高速かつ低コストで行うことができます。

## もっと学びたい方へ
Bitcoin との統合の仕組みについて深く知りたい方は、[How it Works](bitcoin-how-it-works.md) ページで、私たちの新しい閾値 ECDSA 実装、API などの概要をご覧ください。

## ビルドしたい方へ
ローカルでの Bitcoin 統合を試したい方は、[Bitcoin Dapps をローカルで開発する](local-development.md) をご覧ください。
[Examples repo](https://github.com/dfinity/examples) には [Rust](https://github.com/dfinity/examples/tree/master/rust/basic_bitcoin) と [Motoko](https://github.com/dfinity/examples/tree/master/motoko/basic_bitcoin) のサンプルコードがあり、そこから [sample code walk-through](../../../samples/deploying-your-first-bitcoin-dapp.md) に従ってビルドを開始できます。

<!--
# Bitcoin Integration

The Internet Computer integrates directly with the Bitcoin network. This allows canisters on the Internet Computer to receive, hold, and send Bitcoin, all directly with transactions on the Bitcoin network. I.e., canisters can act exactly like regular users holding bitcoin on the Bitcoin network. All of this is made possible by (1) the Internet Computer integrating with Bitcoin at the protocol level and (2) canisters being able to securely hold (and use) ECDSA keys by means of a novel threshold ECDSA protocol. The Internet Computer is among the first blockchain networks performing such direct integration with other blockchains and has built a novel technology foundation for this purpose.

This integration allows for a plethora of novel use cases:

-   *Bitcoin smart contracts:* A canister can directly hold Bitcoin on the Bitcoin network, which allows engineers to implement powerful Bitcoin smart contracts using canisters. Any canister smart contract can now offer Bitcoin smart contract functionality. For example on-chain Bitcoin wallets with biometric authentication without the user being required to manage the private key, or Social-Fi, where users can do peer-to-peer Bitcoin transactions using social dApps.
-   *Trading Bitcoin* on decentralized exchanges on the Internet Computer.
-   Using Bitcoin to buy tokens in a *decentralization sale* when an SNS-powered DAO decentralizes a service on the IC.
-   *Chain Key Bitcoin (ckBTC)*, an advanced variant of wrapped Bitcoin, that will be available on the IC with the Bitcoin mainnet release of the Bitcoin feature. ckBTC will be the easiest way to handle Bitcoin on the IC and might be the right choice for many people interested in Bitcoin on the IC. Note that ckBTC will only be available with the general availability (GA) release of Bitcoin in the upcoming months together with Bitcoin mainnet launch on the IC.

These are only a few examples of how one can use the Bitcoin integration feature. Your imagination is the only limit to the endless range of possibilities being opened up by this feature. This documentation explains how to use the feature to implement your own dApps using Bitcoin. Please also note that the Bitcoin integration API is a low-level API that operates on the level of UTXOs and Bitcoin transactions and is non-trivial to use.

As part of the upcoming Bitcoin mainnet release (general availability release) of the Bitcoin integration feature, a *Chain Key Bitcoin* (ckBTC) Canister will be made available. The ckBTC canister will provide on-chain Bitcoin on the IC, which looks and feels like wrapped Bitcoin, but has a much stronger underlying trust model because of its decentralized architecture and using threshold ECDSA instead of bridges. We envision that many people will revert to using ckBTC instead of our native integration for their projects because of some distinct advantages:
-   *Easier to integrate:* Instead of using the low-level Bitcoin integration API, one can simply access the ckBTC ledger like any other ledger on the Internet Computer. The ckBTC ledger will adhere to the upcoming IC token standard for fungible tokens.
-   *Faster and cheaper transfers:* ckBTC can be transferred with the low finality time of the Internet Computer (within seconds) and for a fraction of the cost of a Bitcoin transfer on the Bitcoin network. Using this scheme, only the settlement transfers with the Bitcoin network need to be done on the Bitcoin network, the majority of transfers can be done with lightning speed and low cost directly on the IC.

## Learn more
If you want to take a deep dive into how Bitcoin integration works, see [How it Works](bitcoin-how-it-works.md) page to get an overview of our novel threshold ECDSA implementation, the API, and more. 

## Build more
If you're interested in experimenting with Bitcoin integration locally, see the [local development guide](local-development.md). 
In the [Examples repo](https://github.com/dfinity/examples) you can find sample code in [Rust](https://github.com/dfinity/examples/tree/master/rust/basic_bitcoin) and [Motoko](https://github.com/dfinity/examples/tree/master/motoko/basic_bitcoin) from which you can start to build following the [sample code walk-through](../../../samples/deploying-your-first-bitcoin-dapp.md)

-->