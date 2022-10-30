---
title: Canister はスマートコントラクトの次の進化形
card: /img/what-is-the-ic/canister.jpg
cardImageFit: cover
---

スマートコントラクトとは、ブロックチェーン上で実行されるコンピュータープログラムです。*Canister* または Canister スマートコントラクトは、コンピュータープログラムとデータのセットです。すべての Canister は IC のサブネット上でホストされています。

異なるサブネット上の Canister は、同時に実行することができます。さらに、同じサブネット上の複数の Canister を並行して実行することもでき、スループットをさらに向上させることができます。Canister はノンブロッキングで非同期メッセージを送信することにより、サブネット内やサブネット間で通信します。これらの特性により、基本的に無限のスケーラビリティを実現することができます。

IC の Canister には、以下の特徴があります。

* ブロックチェーンから直接ユーザーインターフェースを提供できます。
* 低料金で数ギガバイトのメモリを保持することができます。
* 低コストで相当量の演算を実行することができます。
* 演算相応額を自身で支払うことができます（[リバースガス・モデル](https://internetcomputer.org/features/reverse-gas/)を御覧ください）。

Canister は、WebAssembly にコンパイル可能な任意の言語で実装することができます。一般的な Canister 開発キット（CDK）は[こちら](https://internetcomputer.org/docs/current/developer-docs/build/cdks/)に記載されています。

<!--
---
title: Canisters are the next evolution of smart contracts
card: /img/what-is-the-ic/canister.jpg
cardImageFit: cover
---

A smart contract is a computer program executed on a blockchain. A canister is a smart contract that bundles a computer program and its data. Every canister is hosted on one subnet of the IC.
Canisters can be executed concurrently and communicate within and across subnets by sending asynchronous messages in a non-blocking manner, resulting in essentially unbounded scalability.
Canisters are more powerful than smart contracts on other blockchains as they can
* serve a user interface directly to any web browser,
* hold gigabytes of memory for a low fee,
* perform substantial amounts of computation at a low cost, and
* pay for their own computation (learn more about the [reverse gas model](https://internetcomputer.org/features/reverse-gas/)).

Canisters can be implemented in any language that compiles to WebAssembly. Popular canister development kits (CDKs) are described [here](https://internetcomputer.org/docs/current/developer-docs/build/cdks/).

-->
