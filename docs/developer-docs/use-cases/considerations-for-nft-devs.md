# NFT 開発者のための考慮事項

## NFT とは何か？

NFT または Non-fungible Token は、特定のデジタルアセットまたは物理的アセットに関連付けられたブロックチェーン上の記録です。ブロックチェーン上での一意なデジタル表現により、所有権の証明や売買が可能になります。 

## Internet Computer の NFT

Internet Computer（IC）は NFT に多くの可能性をもたらします。画像、サウンドクリップ、ビデオなどのデジタルアセットでは、アセット全体をオンチェーンでライブ配信し、オンチェーンゲームやメタバース体験に含めることができます。さらに、[HTTPS アウトコール](/https-outcalls) を介して IC 内部や外部のデータに基づいて変化する動的な NFT を想像することができます。

多くのアプリケーションにとって、その決定的な特徴は永続性と不変性（または事前に定義されたルールに従った変化）です。リバースガスモデルや Canister 型スマートコントラクトのアップグレード可能性など、IC の設計決定の中には、NFT 開発者が特に意識する必要があるものがあります。

IC 上の NFT 実装は通常、以下の3つの機能を備えています：

1. 所有権を追跡し、譲渡を可能にするレジストリ
2. 台帳または取引履歴
3. 現物（デジタルアセットの場合）

アーキテクチャによっては、これらの機能はすべてひとつの Canister に含まれることもあれば、複数の Canister にまたがって、個々の NFT ごとにひとつのアセット Canister まで含まれることもあります。これらの各 Canister は、 Cycle 不足になってはならず、また任意のコード変更から保護される必要があります。以下では、NFT 開発者とそのユーザがこれらの目標を達成するためのメカニズム、ツール、アイデアについて説明します。


## 基礎

### すべての Canister に十分にトップアップする

まず、すべての Canister に数年間維持できる十分な Cycle があることを確認します。IC 上のストレージと計算は他のプラットフォームよりはるかに安価なので、これは一般的に大きな費用ではありません。他の人が簡単に Canister をトップアップ（補充）できるようにするには、NFT Canister のコントローラーとして[ブラックホール Canister ](https://github.com/ninegua/ic-blackhole)や他の不変のプロキシ Canister を追加することを検討すべきです。これにより、ユーザーは [Tip Jar サービス](https://k25co-pqaaa-aaaab-aaakq-cai.ic0.app/) を使って Canister にトップアップすることができるようになります。

### 凍結閾値（freezing threshold）を余裕をもって設定する

IC には、Canister を Cycle 切れから救うための便利なメカニズムがあります。Canister には設定可能な [`freezing_threshold`](../../references/ic-interface-spec.md#ic-create_canister) があります。`freezing_threshold` は、Canister のコントローラーで設定することができ、秒単位で指定できます。IC は、これを閾値として動的に Cycle を評価させます。この値は、`freezing_threshold` で指定された時間内は少なくとも Canister がアイドル状態のリソースを支払うことができるというものです。これを保証するために、Canister は Cycle バランスが閾値に達するとフリーズし、ハートビートを含むすべてのアップデートコールが直ちに拒否され、それ以後の Canister の Cycle バランスに影響を与えません。デフォルト値は約30日ですが、NFT の場合、開発者は `freezing_threshold` を最低でも90日、できれば180日に設定すべきです。これにより、NFT の開発者とそのユーザーは、Cycle を完全に使い果たす前に Canister をトップアップして対応するための十分な時間を確保することができます。


### Canister の監視が可能であることを確認する

IC 上では、Canister の Cycle バランスはコントローラーのみが確認することができます。NFT（コレクション）は作成者より長く存在する可能性があるため、サードパーティによる監視を計画する必要があります。[DIP721](https://github.com/Psychedelic/DIP721/blob/064b04fbaf0429bf9fefdc0663d53fae033be0f9/src/main.rs#L450) と [EXT](https://github.com/Toniq-Labs/extendable-token/blob/86eabb7336ea259876be9be830fb69b03046ea14/examples/erc721.mo#L254) 規格に含まれるように、簡単なクエリーメソッドを実装することでこれを行うことができます。

この点でも、ブラックホール Canister をコントローラーとして追加することは良い方法です。なぜなら、[`canister_status`](../../references/ic-interface-spec.md#c-canister_status) を取得するプロキシとして機能するからです。

また、[Canistergeek](https://canistergeek.app/) のような、より完全な監視ソリューションを使用することもできます。最近、Canistergeek の開発チームは [NFTgeek](https://t5t44-naaaa-aaaah-qcutq-cai.raw.ic0.app/) 製品に新しい機能を追加し、[人気のNFTコレクションの Cycle バランス](https://t5t44-naaaa-aaaah-qcutq-cai.raw.ic0.app/cycles)を観察できるようにしました。



### ベストプラクティスに従った効率的な実装

実装の仕方によっては、Canister が予想以上に高価になる可能性があります。ここでは、NFT Canister を導入する際に遭遇する可能性のある事例を紹介します。

* ハートビートを利用：何もしない素のハートビートでは、〜0.055T Cycle/day のコストがかかります。[より安価なスケジューリングを可能にする代替手段を実装する](https://forum.dfinity.org/t/heartbeat-improvements-timers-community-consideration/14201)という議論があります。
* Motoko 開発者へのアドバイス：
    * HashMap の代わりに `TrieMap` を使用すると、HashMap に関連する自動リサイズによるパフォーマンスの低下を回避することができます。
    * 動的にサイズを変更する必要がある場合は、`Array` ではなく `Buffer` を使用してください。
    * 大きなバイナリアセットを保存する場合は、`[Nat8]` の代わりに `Blob` を使用します。
    * Candid `vec nat8/blob` 値を送受信する場合は、`[Nat8]` ではなく `Blob` を使用することを検討してください。選択するのはあなたですが、`Blob` の方が4倍コンパクトで、ガベージコレクション（GC）への負担がずっと少なくなります。
    * Blob の手動メモリ管理が単純な場合（例えば、追加するだけで、削除はしない）、GC への負担をさらに減らすために、大きな `Blob` をステーブルメモリに格納することを検討してください。
    * `compacting-gc` 設定を使用することを検討してみてください。特に、追加のみ（Append-only）のシナリオでは、より大きなヒープにアクセスできるようにし、オブジェクトのコピーコストを削減することができます。
* Rust の開発者へのアドバイス：
    * アップグレードのためにステートをシリアライズ・デシリアライズする必要がある場合は、`Vec<u8>` や `String` 型を多用しないように注意してください。
    * [Roman の効果的な Rust Canister に関するブログポスト](https://mmapped.blog/posts/01-effective-rust-canisters.html)をお読みください。

また、[Good practices for canister development by Joachim Breitner](https://www.joachim-breitner.de/blog/788-How_to_audit_an_Internet_Computer_canister) という一般的な記事も必読です。

高い Cycle 消費率や命令制限に驚かないように、最近追加された[パフォーマンスカウンター API](https://forum.dfinity.org/t/introducing-performance-counter-on-the-internet-computer/14027) を使って、稼動前でも Canister をプロファイリングすることができます。さらに、IC 上のすべてのコストのリストは[ここ](../deploy/computation-and-storage-costs.md)で見ることができます。


### ステートのバックアップと復元を行う仕組みを実装する

IC 自体はまだ Canister のステートのバックアップと復元をサポートしていませんが、Canister 自体に実装することは可能です。定期的なバックアップは、Canister が割り当てられなくなったり、Canister のアップグレードに問題が生じたりする最悪のシナリオに対する保険となります。[このフォーラムの投稿](https://forum.dfinity.org/t/backup-restore-function-for-a-canister/12849/3)は、[Distrikt](https://distrikt.app) が使っているアプローチについて説明しています。


### 取引履歴の保存には、専用サービスの利用を検討する

IC 上には [CAP](https://cap.ooo/) のようなトランザクションの監査ログを保持する専用サービスがあり、NFT コレクションがサービスとして利用することが可能です。これにより、エクスプローラーやウォレットに履歴を簡単に統合することができます。
さらに、メインの NFT Canister が紛失した場合、所有ステートを再構築することができます。しかし、NFT の転送には Canister 間コールが必要なため、追加コストがかかるなどの欠点も考慮する必要があります。

## より高度なトピック

### ガバナンスについて考える

ほとんどの NFT の価値は、例えば[ブラックホール Canister](https://github.com/ninegua/ic-blackhole) を唯一のコントローラーとして設定することによる、永続性と不変性です。NFT の Canister が開発者をコントローラーとしている限り、ユーザーは開発者の信頼性（および運用上の安全性）に依存することになります。そのため、開発者は Canister をイミュータブルにするか、DAO で Canister を管理する必要があります。中間的な方法としては、Canister への変更を監査可能にする [Launchtrail](https://devpost.com/software/launch-trail) のようなメカニズムがあります。

Canister をブラックホール化することにも問題があります。Canister のコードにバグがあったり、非推奨になるかもしれない実験的なシステム API を使っていたりすると、後になって Canister が機能しなくなる可能性があります。

このトピックに関する詳細は、[Trust in Canisters](../../concepts/trust-in-canisters.md) の記事でご覧いただけます。

### 経済的な持続可能性を考える

理想的には、Canister がその存在を無期限に支払うために使用できる手数料を生成するメカニズムを実装することです。シンプルなアプローチは、送金手数料の一部を Canister のガスとして利用することですが、より精巧な仕組みとして、ステーキングや他の高度な仕組みを取り入れることもできます。私たちは一番良いとされるベストプラクティスをまだ発見していませんが、巧妙なメカニズムを実装しているプロジェクトをご存知でしたら、ぜひ教えてください。


## リンクとリソース

以下のリソースは、コミュニティーのプロジェクトです。ご自身で調べ、ご自身の責任でご利用ください。

### NFT インターフェースの仕様と実装

- [DIP721](https://github.com/Psychedelic/DIP721)：[ERC-721](https://eips.ethereum.org/EIPS/eip-721) に類似したインタフェース。
- [Extendable Token (EXT)](https://github.com/Toniq-Labs/extendable-token)：[ERC-1155](https://eips.ethereum.org/EIPS/eip-1155) にヒントを得た拡張可能なインターフェース。

### NFT マーケットプレイスおよびローンチパッド

- [Entrepot](https://entrepot.app/)
- [Jelly](https://jelly.xyz/)
- [Yumi](https://tppkg-ziaaa-aaaal-qatrq-cai.raw.ic0.app/)
- [NFT Anvil](https://nftanvil.com/)

<!--
# Considerations for NFT developers

## What is an NFT?

An NFT or Non-fungible Token is a record on a blockchain that is associated with a particular digital or physical asset. The unique digital representation on a blockchain allows the proving of ownership as well as their trading. 

## NFTs on the Internet Computer

The Internet Computer (IC) brings a lot of potential for NFTs. For digital assets like images, sound clips, or videos, the entire assets can live on-chain and can be included in on-chain games or metaverse experiences. Furthermore, we can imagine dynamic NFTs that change based on IC-internal and external data via [HTTPS outcalls](/https-outcalls).

For many applications, the defining characteristic is their permanence and immutability (or evolution according to predefined rules). Some of the design decisions of the IC, such as the reverse gas model and the upgradability of canister smart contracts, require the NFT developer to be particularly aware.

An NFT implementation on the IC typically has the following three functions:

1. A registry that tracks ownership and allows transfers
2. A ledger or transaction history
3. The actual asset (in the case of digital assets)

Depending on the architecture, all of these functions may be in one canister or spread across multiple canisters right up to an asset canister per individual NFT. Each of these canisters must not run out of cycles, and should be protected against arbitrary code changes. In the following, we discuss some of the mechanisms, tools, and ideas that support NFT developers and their users to achieve these goals.


## The Basics

### Top up all canisters very generously

Make sure that all canisters have enough cycles to sustain a few years to begin with. Storage and computation on the IC are magnitudes less expensive than on other platforms, so this is typically not a huge investment. To make it easy for others to top up the canisters you should consider adding the [black hole canister](https://github.com/ninegua/ic-blackhole) or some other immutable proxy canister as a controller to the NFT canisters. This allows users to use the [Tip Jar service](https://k25co-pqaaa-aaaab-aaakq-cai.ic0.app/) to top up the canisters.


### Set a generous freezing threshold

The IC has a useful mechanism to save your canister from running out of cycles. Canisters have a configurable [`freezing_threshold`](../../references/ic-interface-spec.md#ic-create_canister). The `freezing_threshold` can be set by the controller of a canister and is given in seconds. The IC dynamically evaluates this as a threshold value in cycles. The value is such that the canister will be able to pay for its idle resources for at least the time given in `freezing_threshold`. To guarantee that, the canister is frozen when the cycle balance reaches the threshold, and all update calls, including the heartbeat, are immediately rejected and won’t affect the canister’s cycle balance. The default value is approximately 30 days, but for NFTs, developers should set the `freezing_threshold` to at least 90 days, preferably 180 days. This makes sure that NFT developers and their users have enough time to react and top up the canisters before they finally run out of cycles.


### Make sure your canisters can be monitored

On the IC, the cycle balance of a canister is only visible to controllers. Since an NFT (collection) might outlive its creator, you should plan for monitoring by third parties. You can do this via implementing a simple query method as included in the [DIP721](https://github.com/Psychedelic/DIP721/blob/064b04fbaf0429bf9fefdc0663d53fae033be0f9/src/main.rs#L450) and [EXT](https://github.com/Toniq-Labs/extendable-token/blob/86eabb7336ea259876be9be830fb69b03046ea14/examples/erc721.mo#L254) standards.

Again, adding the black hole canister as a controller is a good practice in this regard, since it can act as a proxy to fetch the [`canister_status`](../../references/ic-interface-spec.md#c-canister_status). 

You can also use a more complete monitoring solution like [Canistergeek](https://canistergeek.app/). Recently, the team behind Canistergeek added a new feature to their [NFTgeek](https://t5t44-naaaa-aaaah-qcutq-cai.raw.ic0.app/) product that allows observing the [cycles balance of popular NFT collections](https://t5t44-naaaa-aaaah-qcutq-cai.raw.ic0.app/cycles). 



### Follow best practices for efficient implementations

There are a few foot guns that could make your canister more expensive than you’d expect. Here are a few examples that you might encounter when implementing NFT canisters.

* Use of the heartbeat: A plain heartbeat without doing anything will cost ~0.055 T cycles/day. There are discussions about [implementing alternatives that allow for cheaper scheduling](https://forum.dfinity.org/t/heartbeat-improvements-timers-community-consideration/14201).
* Some advice for Motoko developers: 
    * Use `TrieMap` instead of `HashMap` to avoid the performance cliff of automatic resizing associated with HashMaps.
    * Use `Buffer` instead of `Array` if you need to dynamically resize the structure
    * Use `Blob` instead of `[Nat8]` for storing large binary assets
    * Consider using `Blob` instead of `[Nat8]` when sending or receiving Candid `vec nat8/blob` values. The choice is yours but `Blobs` are 4x more compact and much less taxing on garbage collection (GC).
    * Consider storing large `Blob`s in stable memory, to reduce pressure on the GC even further, especially when the manual memory management of that Blob is simple (e.g. they are only added, never deleted).
    * Consider using the `compacting-gc` setting, especially in append-only scenarios, to allow access to larger heaps and reduce the cost of copying large, stationary objects.
* Some advice for Rust developers:
    * Be careful with extensive use of `Vec<u8>` and hence the `String` type if you need to (de-)serialize state for upgrades.
    * Read [Roman’s blog post on effective Rust canisters](https://mmapped.blog/posts/01-effective-rust-canisters.html)

Another must-read is the general article on [good practices for canister development by Joachim Breitner](https://www.joachim-breitner.de/blog/788-How_to_audit_an_Internet_Computer_canister). 

To make sure you won’t get surprised by a high cycle burn rate or hitting an instruction limit, you can use the recently added [performance counter API](https://forum.dfinity.org/t/introducing-performance-counter-on-the-internet-computer/14027) to profile your canisters even before going live. Furthermore, a list of all costs on the IC can be found [here](../deploy/computation-and-storage-costs.md). 


### Implement mechanisms to backup and restore state

The IC itself does not yet support backup and restoration of the canister state, but it can be implemented in the canister itself. Regular backups are insurance against the worst-case scenario that a canister gets deallocated or there are issues with upgrading a canister. [This forum post](https://forum.dfinity.org/t/backup-restore-function-for-a-canister/12849/3) describes the approach [Distrikt](https://distrikt.app) is using.


### Consider using a dedicated service for storing the transaction history 

There are dedicated services on the IC to keep an audit log of transactions such as [CAP](https://cap.ooo/), which can be used by an NFT collection as a service. This allows simple integration of the provenance history in explorers and wallets. 
Furthermore, the state of ownership could be reconstructed in case the main NFT canister gets lost. However, some drawbacks have to be considered, e.g. NFT transfers incur additional costs due to the necessary inter-canister calls. 

## Advanced Topics

### Think about governance

The value proposition of most NFTs is their permanence and immutability, e.g. by setting the [blackhole canister](https://github.com/ninegua/ic-blackhole) as the only controller. As long as NFT canisters have their developers as controllers, users depend on the trustworthiness (and operational security) of the developers. Developers should therefore make the canisters immutable or manage the canisters with a DAO. A middle ground are mechanisms like [Launchtrail](https://devpost.com/software/launch-trail) that make changes to a canister auditable.

Blackholing a canister has its issues as well. If there are bugs in the canister code or you’re using experimental system APIs that might get deprecated, later on, the canister might stop functioning. 

More information on this topic can be found in the [Trust in Canisters](../../concepts/trust-in-canisters.md) article.


### Think about economic sustainability

Ideally, your canisters implement mechanisms to generate fees that the canisters can use to pay for their existence indefinitely. A simple approach is to utilize (parts of) the transfer fee to fuel the canisters, but more elaborate schemes could involve staking or other advanced mechanisms. We’re unaware of any good best practices, but please share if you know of projects implementing clever mechanisms.


## Links and Resources

The following resources are community projects. Please do your own research and use them at your own risk.

### NFT interface specifications and implementations

- [DIP 721](https://github.com/Psychedelic/DIP721): An interface similar to [ERC-721](https://eips.ethereum.org/EIPS/eip-721).
- [Extendable Token (EXT)](https://github.com/Toniq-Labs/extendable-token): Extendable interface inspired by [ERC-1155](https://eips.ethereum.org/EIPS/eip-1155).

### NFT Marketplaces and Launchpads

- [Entrepot](https://entrepot.app/)
- [Jelly](https://jelly.xyz/)
- [Yumi](https://tppkg-ziaaa-aaaal-qatrq-cai.raw.ic0.app/)
- [NFT Anvil](https://nftanvil.com/)

-->


