---
title: Canister はスマートコントラクトの次の進化形
card: /img/what-is-the-ic/canister.jpg
cardImageFit: cover
---

スマートコントラクトとは、ブロックチェーン上で実行されるコンピュータープログラムです。*Canister* または Canister スマートコントラクトは、コンピュータープログラムとデータのセットです。すべての Canister は IC のサブネット上でホストされています。

異なるサブネット上の Canister は、同時に実行することができます。さらに、同じサブネット上の複数の Canister を並行して実行することもでき、スループットをさらに向上させることができます。Canister はノンブロッキングで非同期メッセージを送信することにより、サブネット内やサブネット間で通信します。これらの特性により、基本的に無限のスケーラビリティを実現することができます。

IC の Canister には、以下の特徴があります。

- ブロックチェーンから直接ユーザーインターフェースを提供できます。
- 低料金で数ギガバイトのメモリを保持することができ、低コストで相当量の演算を実行し、演算相応額を自身で支払うことができます（[リバースガス・モデル](https://internetcomputer.org/features/reverse-gas/)を御覧ください）。

エンジニアは、WebAssembly にコンパイルできる任意の言語で Canister を実装することができます。SDK は現在、[Rust](/docs/current/developer-docs/build/cdks/cdk-rs-dfinity/) と [Motoko](/docs/current/developer-docs/build/cdks/motoko-dfinity/motoko/) で利用することができます。

<!--
---
title: Canisters are the next evolution of smart contracts
card: /img/what-is-the-ic/canister.jpg
cardImageFit: cover
---

A smart contract is a computer program executed on a blockchain. A *canister*, or canister smart contract, is a bundle comprising a computer program and its data. Every canister is hosted on one subnet of the IC.

Canisters on different subnets can be executed concurrently. Furthermore, multiple canisters on the same subnet can also be executed in parallel, further increasing throughput. Canisters communicate within and across subnets by sending asynchronous messages in a non-blocking manner. These properties allow for essentially unbounded scalability.

Canisters on the IC have distinguishing properties. They can

- serve a user interface directly from the blockchain,
- hold gigabytes of memory for a low fee,
  perform substantial amounts of computation at a low cost, and
  pay for their own computation (learn more about the [reverse gas model](https://internetcomputer.org/features/reverse-gas/)).

Engineers can implement canisters in any language that compiles to WebAssembly. SDKs are currently available for [Rust](/docs/current/developer-docs/build/cdks/cdk-rs-dfinity/) and [Motoko](/docs/current/developer-docs/build/cdks/motoko-dfinity/motoko/).

-->
