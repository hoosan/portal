---
title: IC のセキュリティ・アーキテクチャを支える Chain Key 暗号
card: /img/what-is-the-ic/chainkey.jpg
cardImageFit: cover
---
サブネットの正しい運用（およびサブネット間の通信）は、[*Canin Key 暗号*](/how-it-works/chain-key-technology/)と総称される一連の革新的な暗号プロトコルに依拠しています。Chain Key 暗号は、サブネットの以下の認証を可能にします。

* ユーザーのリクエストに対するレスポンス
* サブネットのステート
* 完全な分散型サブネット間メッセージ

NNS は、あたかも分散型の認証局のようにサブネットの公開鍵を承認します。ユーザーは、NNS の48バイトの BLS 公開鍵だけで、どの Canister との対話も検証することができます。

従来のブロックチェーンは通常、新たに参加した当事者がチェーン上で行われたすべての取引を再検証する必要があります。これは IC のような高スループットのシステムでは実現不可能でした。
Chain Key 暗号を活用することで、以下のことが可能になります：

* サブネットのメンバー変更 - レプリカは直近の有効なチェックポイントから開始してサブネットに参加することができ、また任意の時点で離脱することができます。
* プロアクティブセキュリティ - サブネットの閾値キーは、サブネットの現在のノード間で定期的に再共有（re-sharing）されます。
* 永続的な公開鍵 - メンバーシップの変更および鍵の再共有は、サブネットの公開鍵に影響を与えません。
* ガーベッジコレクション - ストレージが無限に増加するのを防ぐため、定期的に各サブネットのブロックチェーンから以前のブロックが削除されます。

<!--
---
title: Chain-key cryptography underpins the IC's security architecture
card: /img/what-is-the-ic/chainkey.jpg
cardImageFit: cover
---

The correct operation of subnets (and inter-subnet communication) relies on a suite of novel cryptographic protocols, collectively referred to as [*chain-key cryptography*](/how-it-works/chain-key-technology/). Chain-key cryptography makes it possible for subnets to authenticate
* responses to user requests,
* the subnet state, and
* inter-subnet messages
in a completely decentralized way.

The NNS endorses the public keys of subnets, much like a decentralized certification authority. Users only need the 48-byte BLS public key of the NNS to validate the interaction with any canister. Traditional blockchains typically require newly joined parties to redo all transactions ever performed on the chain, which is not feasible in a high-throughput system like the IC. 

The IC uses chain-key cryptography to provide:
* Subnet membership changes – A replica can join a subnet, by starting from the most recent valid checkpoint, or leave at any point in time.
* Proactive security – Threshold keys of the subnet are periodically reshared between the current nodes of the subnet.
* Permanent public keys – Membership changes and key resharing do not affect the public key of any subnet.
* Garbage collection – Periodically, previous blocks are pruned from each subnet blockchain to prevent storage from growing infinitely.

-->