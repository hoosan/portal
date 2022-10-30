---
title: 低レイテンシー・高スループットなコンセンサス
card: /img/what-is-the-ic/consensus.jpg
cardImageFit: cover
---

Internet Computer Protocol は、サブネットの複数のノード（最大で3分の1以下）が故障したり誤動作したりしても、どのサブネットの Canister ステートが辻褄が合うことを保証するものです。ICはプルーフ・オブ・ワークでもプルーフ・オブ・ステークでもなく、いわゆる *DAO-controlled network* であり、NNS DAO がサブネットノードのメンバーシップを管理します。
[コンセンサスプロトコル](/how-it-works/consensus/) は、以下のような望ましい特性を有しています。
* 低いレイテンシー - 少ないラウンド数の交換で合意に達します。通常、1～2秒以内にコンセンサスに到達します。
* 高いスループット - すべてのコンセンサス実行は、メガバイトオーダーのペイロードを処理することができます。
* 暗号学的なファイナリティ - IC コンセンサスは暗号学的に保証されたファイナリティに達します。すなわち、確定したステートの変更は元に戻すことができません。
- ビザンチンフォールトトレランス（BFT） - 任意の1/3まで（ただし、それ以下）の欠陥ノードを許容することができます。

<!--
---
title: Low-latency high-throughput consensus
card: /img/what-is-the-ic/consensus.jpg
cardImageFit: cover
---

The Internet Computer Protocol ensures that the canister state on any subnet is always consistent—even if multiple nodes of a subnet are faulty or misbehave. The IC is neither a proof-of-work, nor a proof-of-stake network, but a so-called DAO-controlled network, where the NNS DAO manages subnet node membership. The [consensus protocol](/how-it-works/consensus/) has the following properties:

* Low latency – A small number of rounds of exchange suffice to reach agreement. Consensus is normally reached within 1 to 2 seconds.
* High throughput – Every consensus execution can handle payloads in the order of megabytes.
* Cryptographic finality – IC consensus reaches cryptographically-guaranteed finality, that is, finalized state changes cannot be undone.
* Byzantine fault tolerance (BFT) – The IC can tolerate up to (but less than) one third of arbitrarily faulty nodes.
