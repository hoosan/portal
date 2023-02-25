# Canister のデプロイとアップグレード

## デプロイのメカニズム

デプロイの際、懸念されるのは主に２点です。それは Canister とコードです。 Canister に関する懸念は、コードが最終的にどこで実行されるのか、そしてそれをどのように正しい場所に持っていくのか、ということに尽きます。もうひとつは、データの損失がなく、ダウンタイムが可能な限り短く、設定ミスがないように、コードをどのように管理するかということです。デプロイ時には考慮すべきことが多く、うまくいかないことも多いため、開発者を楽にするためのツールが存在します。そのようなツールのひとつが DFINITY Canister Software Development Kit（SDK）で、デプロイを管理する CLI ツールは `dfx` と呼ばれています（このページで図解されています）。

### Canister ライフサイクル

Internet Computer は、いわゆる Canister で任意の WebAssembly モジュールを実行します。最初のデプロイでは、実行するはずの個々の WebAssembly モジュールごとに新しい Canister を作成する必要があります。これらの Canister の作成時に、Canister ID が割り当てられます。Canister には中央レジストリは存在しないので、開発者（あるいはそのツール）はこれらの ID を追跡する必要があります。コードが Canister にデプロイされると、Canister ID を使用して他の Canister の関数を呼び出すことができるようになります。

 Canister は実行中に Cycle を消費します。計算の実行だけでなく、データの保存にも Cycle がかかります。 Canister が動作している間は、動作を維持するのに十分な Cycle を含んでいなければなりません。ここでも、開発者（または開発ツール）が ID や Cycle 残高を追跡する必要があります。Canister の Cycle が非常に少なくなると、フリーズし（更新の呼び出しに応答しなくなる）、完全になくなると、削除されます。

誰もが Canister の変更を許可されているわけではありません。 Canister にはすべて、設定の更新や実行中のコードのインストール/アップグレード、さらには Canister の削除を許可されたコントローラーのリストが存在します。開発者、自動化ツール、他の Canister、または何もない状態でも、コントローラーになることができます。 Canister の目標に応じて、さまざまなコントローラーを組み合わせることができます。以下のセクション[Demonstrating Trust](#demonstrating-trust) で、より詳しく説明しています。

 Canister のライフサイクルが終了したら、削除することができます。その前に、所有者の一人が Canister を削除する前に、残りの Cycle を引き出しておくことをお勧めします。そうしないと、これらの Cycle は失われます。これらの Cycle を手動で引き出すのはかなり面倒な作業なので、この作業を容易にするツールが存在します。 Canister を削除する以外の方法として、自動的に削除されるまで Canister を稼動させたままにすることができます。

### コードをデプロイする

 Canister は任意の WebAssembly（WASM）モジュールを実行できますが、 Internet Computer にはその機能を最大限に活用しやすくするためのいくつかの規約があります。例えば、`canister_init` という関数は、コードが初めてインストールされた後に最初に呼び出される関数です。これらの規約を守りやすくするために、 Canister 開発キット（CDK）が [異なる言語](../build/cdks/index.md)毎に存在します。

 Internet Computer の `install_code` 機能で利用できる 3つのモードのうちのひとつを Canister にコードをインストールするには：
- `install` モードはすべての Canister が持つモードです。インストールされていない Canister に対してのみ呼び出すことができ、 Canister に提供された Wasm モジュールを組み込みます。インストールが完了すると、前述の関数 `canister_init`（通常 CDK では `init` として公開されます）が存在すれば、それが呼び出されます。これにより、呼び出しが行われる前に、コードが必要な設定を行うことができます。
- `reinstall` モードは `install` とほぼ同じ動作をしますが、 Canister にすでに実行中のコードが含まれている場合、そのステートは破棄され、実行中のコードが削除されます。その後は `install` モードの手順が踏襲されます。
- `upgrade`モードは最もよく使われるモードです。このモードでは、 Canister のステートをすべて失うことなく、Canister のコードを変更することができます。このモードでは、まず Canister は `canister_pre_upgrade`（CDK では `pre_upgrade` として公開）関数でステーブルメモリに任意のステートを保存します。その後、新しいコードがインストールされ、init 関数を呼び出す代わりに `canister_post_upgrade` 関数（CDK では `post_ugrade` として公開）を実行して、ステーブルメモリからデータを読み込むことができるようになります。
3つの `install_code` モードは、すべてアトミックです。もし、説明した手順の中で何か問題が発生した場合、 Canister のステートは `install_code` 関数が呼ばれる前のステートに戻されます。また、これらの説明で十分に理解できない場合は、[IC Specification for `install_code`](../../references/ic-interface-spec.md#ic-install_code) を御覧ください。

:::caution
 Internet Computer のすべての関数呼び出しには、実行可能な命令数の制限があります。もし `pre_upgrade` 関数がこの制限を超えると、関数はキャンセルされ、アップグレードは失敗します。これによって、 Canister がアップグレードできなくなることがあります。この制限を回避する方法として、Wiki [contains some ideas](https://wiki.internetcomputer.org/wiki/Dealing_with_cycles_limit_exceeded_errors) があります。
:::

既存の Canister をアップグレードする場合、さらにいくつかの注意点があります：
- *未決定のコールバック*：Canister が他の Canister からのレスポンスを `await` している場合、レスポンスを送信してから受信するまでの間にアップグレードすることができます。コードが `upgrade` モードでインストールされている場合、コールバックはアップグレードが行われていないかのように実行されます。このようなことが起こしたくないのなら、まず Canister を停止するか、コードをアンインストールしてから、再度インストールする必要があります。
- *インターフェイスの互換性*：Canister やスクリプトが、アップグレードされた Canister に特定のインターフェイスがある場合、アップグレードによって既存のワークフローが破壊される可能性があります。DFX は、アップグレードによって特定の署名が壊れることを（可能な限り）ユーザーに警告しますが、見落とされる可能性のあるコーナーケースが存在します。

## 考慮すべきこと

  * [（ガスの）資金調達](#funding)
  * [Demonstrating Trust](#demonstrating-trust)

### 資金調達

 Internet Computer Canister は、時間の経過とともに徐々に消費される Cycle のバランスに応じて資金を調達しています。お客様は、割り当てられたメモリ量と、クエリーやアップデートにかかる CPU 時間に対して支払いを行うことになります。

ここでは、そのチェックリストを紹介します：

- [ ] Canister の資金源とはなにか？
  * デベロッパーによる前払い
  * コミュニティからの寄付金で調達
  * ICP や Cycle の継続的な収益で賄われる
- [ ] Canister のバランスはどのように把握するのか？
- [ ] 万が一、残高が足りなくなった場合の対策は？
- [ ] 残高がなくなり、最終的に Canister が消去されたらどうなるのか？

ユースケースによって、アプリケーション毎に異なる答えが最適解になるでしょう。追加されたすべてのコンテンツが永遠に利用できると仮定する他のブロックチェーンとは異なり、Internet Computer では、使用しているコンピューティングリソースのコストと持続可能性について考える必要があります。

NFT プロジェクトや DeFi Canister  は、取引手数料を使って自分たちの Cycle を購入することで、自己資金を確保することができるかもしれません。その他の使用例では、企業や DAO に残高を管理する責任を負ってもらうとよいでしょう。いずれにせよ、 Canister を長持ちさせるための計画について考えておくことは価値があります。

### 信頼の実践

ブロックチェーンの分野では、ユーザーがやり取りするスマートコントラクトをどのように信頼できるかが大きなトピックとなっています。あなたがデプロイしたいプロジェクトの種類によって、異なるレベルで適切な信頼モデルがあります。

ここでは、検討する必要がある事柄のチェックリストを示します：

- [ ] このプロジェクトには、どれくらいの信頼が必要か？
- [ ] Canister が本来の機能を発揮するためには、どのようにすればよいか？
  * [ Canister の信頼](../../concepts/trust-in-canisters.md)と[再現可能なビルド](../build/backend/reproducible-builds.md)には、このトピックに関連する情報が記載されています。
- [ ] コードが突然変更されないと、ユーザーは信頼できるか？

<!--
# Deploying & Upgrading Canisters

## Mechanics of a Deployment

During deployment, there are two main concerns: Canisters and code. The concern of canisters is all about where the code will end up running, and how to get it to the right place. The other concern is how code is managed to ensure there is no data loss, as little downtime as possible, and no misconfigurations. Since there are a lot of things to consider and many things that can go wrong during deployment, tools exist to make developers' lives easier. One such tool (which will be used as illustration on this page) is the DFINITY Canister Software Development Kit (SDK), whose CLI tool for managing deployments is called `dfx`.

### Canister lifecycle

The Internet Computer runs arbitrary WebAssembly modules in so-called canisters. For the first deployment, new canisters have to be created for every individual WebAssembly module that is supposed to run. On creation of those canisters, they will be assigned a canister ID. Since there is no central registry of canisters, the developers (or more likely their tools) have to keep track of those IDs. Once code is deployed to canisters, the canister IDs can be used to call functions on other canisters.

Canisters consume cycles while they're running. Performing computation costs cycles, but so does storing data. As long as a canister is supposed to be running it has to contain enough cycles to maintain operations. This again is a place where developers (or their tools) need to keep track of IDs and/or cycles balances. If a canister runs very low on cycles, it will be frozen (meaning it won't respond to update calls anymore), and if it runs out entirely, it will be deleted.

Not everyone is allowed to change a canister. Canisters all have a list of controllers that are allowed to update its settings, install/upgrade the running code or even delete the canister. Developers, automated tools, other canisters or nothing at all can be controllers. Depending on the goals for the canister, different combinations of controllers can make sense. The below section [Demonstrating Trust](#demonstrating-trust) explains more about that.

Once a canister is at the end of its lifecycle, it can be deleted. Before doing so, it is recommended that (one of) its owner(s) withdraws remaining cycles before they delete the canister. Otherwise those cycles are lost. Since withdrawing those cylces is a rather tedious process manually, tools exist to facilitate the process. The other option besides deleting the canister is to just leave it running until it gets deleted automatically.

### Deploying code

While canisters can run arbitrary WebAssembly (WASM) modules, the Internet Computer has a few conventions that make it easier to get the most out of its capabilities. For example, the function `canister_init` is the first function that gets called after the code is installed for the first time. To facilitate adhering to those conventions, Canister Development Kits (CDKs) exist for [many different languages](../build/cdks/index.md).

To install code in a canister, the `install_code` function of the Internet Computer is used in one of three modes:
- The `install` mode is the one every canister starts with: it is only callable for canisters without any installed code and populates the canister with the supplied WASM module. Once installation is complete, the aforementioned function `canister_init` (usually exposed as `init` in CDKs) is called if it exists. This allows the code to perform any required setup before any calls arrive.
- The `reinstall` mode works almost the same as `install`, but if the canister already contains running code, its state is discarded and the running code is deleted. After that, the procedure from mode `install` is followed.
- The `upgrade` mode is the one used most often. This mode allows canister code to be changed without losing all of its state. In this mode, the canister first has the chance to save any state to stable memory in the `canister_pre_upgrade` (exposed as `pre_upgrade` in CDKs) function. After that, the new code is installed and instead of calling the init function, the `canister_post_upgrade` function (exposed as `post_ugrade` in CDKs) is run so that data can be loaded from stable memory.
All three `install_code` modes are atomic. If _anything_ goes wrong in one of the described steps, the canister state is reverted to its state before the `install_code` function was called. And in case anything is not clear enough from these explanations, the [IC Specification for `install_code`](../../references/ic-interface-spec.md#ic-install_code) is the official source of truth.

:::caution
Every function call on the Internet Computer has a limit to how many instructions can be executed. If your `pre_upgrade` function exceeds this limit, it will be canceled and the upgrade fails. This can make a canister un-upgradeable. The Wiki [contains some ideas](https://wiki.internetcomputer.org/wiki/Dealing_with_cycles_limit_exceeded_errors) how one can work around the cycles limit.
:::

When upgrading existing canisters, there are a few more things that one should keep in mind:
- *Outstanding callbacks*: If a canister is `await`ing a response from another canister, it can be upgraded in-between sending and receiving a response. If the code is installed in `upgrade` mode, the callback will be executed as if no upgrade has been made. If this should not happen, the canister should either first be stopped, or the code has to be uninstalled and then installed again.
- *Interface compatibility*: If canisters or scripts expect the upgraded canister to have a certain interface, upgrades can break existing workflows. DFX will warn the user (if possible) that the upgrade will break certain signatures, but there are always corner cases that may be missed.

## Things to consider

  * [Funding](#funding)
  * [Demonstrating Trust](#demonstrating-trust)

### Funding

Internet Computer Canisters are funded using a balance of cycles that will gradually burn down over time. You will be paying for the amount of memory you have allocated, as well as the CPU time that queries and updates take up.

Here is a checklist of the things you will need to consider:

- [ ] What will the canister's source of funds be?
  * Paid for up-front by the developer
  * Funded by donations from the community
  * Funded by ongoing revenue in ICP or cycles
- [ ] How will I monitor the canister's balance?
- [ ] What is your plan in case the balance runs low?
- [ ] What will happen if the balance runs out and the canister is eventually erased?

Depending on your use case, different answers will make sense for your application. Unlike other blockchains, where you assume that all content added is available forever, the Internet Computer requires you to think about the costs and sustainability of the computing resources that you are using.

An NFT project or DeFi canister may be able to self-fund by using transaction fees to purchase their own cycles. For other use cases, you may want a company or DAO to be responsible for supervising the balance. In any case, it is worthwhile to think about how you plan to ensure your canister's longevity.

### Demonstrating Trust

A big topic in the blockchain space is how users can trust the smart contracts they interact with. Depending on the kind of project you want to deploy, different levels of trust can be appropriate.

Here is a checklist of the things you will need to consider:

- [ ] How much trust does this project require?
- [ ] How can I demonstrate that the canisters do what they are supposed to do?
  * The sections [Trust in Canisters](../../concepts/trust-in-canisters.md) and [Reproducible Builds](../build/backend/reproducible-builds.md) contain information related to this topic.
- [ ] How can users trust that the code will not suddenly change?

-->
