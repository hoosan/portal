---
id: index
title: Deployment and Scaling Overview
description: When you are ready to deploy your project, this guide will help you have the most seamless possible launch or upgrade.
---

# デプロイとスケーリングの概況

プロジェクトをデプロイする準備ができたら、このガイドを参考にして、可能な限りシームレスにローンチまたはアップグレードしてください。

このページは、その概要について説明しています。具体的な手順やトラブルシューティングについては、次のページを参照してください：

* [Canister のデプロイとアップグレード](./deploying-and-upgrading.md)
* [Large web assembly modules](./larger-wasm.md)
* [カスタムドメイン](./custom-domain.md)
* [コンピューティングとストレージのコスト](./computation-and-storage-costs.md)

### 信頼の実践

Internet Computer では、Canister のソースコードを常に検査することはできず、いつでもコントローラーが Canister 上で実行されているコードを置き換えることができます。アップグレード可能なことは便利であり、継続的な開発を可能にしますが、安定し信頼性のあるコントラクトに依存したいと思っているユーザーにとってはリスクも生じます。このトピックの詳細については、[Trust in Canisters](../../concepts/trust-in-canisters.md) をお読みください。

## 安全にアップグレードする方法

これは、[Motoko upgrade compatibility](../build/cdks/motoko-dfinity/compatibility.md) で詳しく検討されています。

<!--

---
id: index
title: Deployment and Scaling Overview
description: When you are ready to deploy your project, this guide will help you have the most seamless possible launch or upgrade.
---

# Deployment and Scaling Overview

When you are ready to deploy your project, this guide will help you have the most seamless possible launch or upgrade.

This page is an overview of the topic. For specific steps and troubleshooting information, please refer to the following pages:

* [Deploying & Upgrading Canisters](./deploying-and-upgrading.md)
* [Large web assembly modules](./larger-wasm.md)
* [Custom Domains](./custom-domain.md)
* [Computation and Storage cost](./computation-and-storage-costs.md)

### Demonstrating Trust

In the Internet Computer, a canister's source code cannot always be inspected, and at any point, a controller can replace the code running on the canister. Upgradeability is convenient and allows for continuous development, but it also creates risk for users who need to depend on the contract to be stable and trustworthy. Read [Trust in Canisters](../../concepts/trust-in-canisters.md) for more information on this topic.

## How to upgrade safely

This is explored in depth in [Motoko upgrade compatibility](../build/cdks/motoko-dfinity/compatibility.md)

-->