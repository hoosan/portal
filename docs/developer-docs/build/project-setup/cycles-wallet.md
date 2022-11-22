# Cycle ウォレット

[トークンと Cycle](../../../concepts/tokens-cycles) で説明したように、ICP トークンは **Cycle** に変換され、Canister の操作に利用されます。Cycle は、Dapp が消費する通信、計算、ストレージの運用コストを示しています。

ICP トークンとは異なり、Cycle は Canister にのみ関連付けられ、ユーザーや開発者の Principal には関連付けられません。 Canister だけが、操作の実行や使用するリソースの支払いに Cycle を必要とし消費するため、ユーザーと開発者は、**Cycle ウォレット**と呼ばれる特別なタイプの Canister を通じて Cycle の配布と所有権を管理します。Cycle ウォレットは、新しい Canister の作成などの操作に必要な Cycle を保持しているため、ユーザーの Principal の代わりに、Cycle ウォレットの Canister Principal を使用してこれらの操作が実行出来ます。

ローカル Canister 実行環境を使用するために、SDK はすべてのプロジェクトで自動的にデフォルトの Cycle ウォレットを作成します。また Cycle ウォレットを使用して実行される操作のほとんどはバックエンドで行われます。例えば、Cycle ウォレットはあなたの（Principal）代わりに、 Canister Principal の登録や、ローカル Canister 実行環境での Canister のデプロイを行います。

しかし、プロダクト環境では Principal を管理して、Cycle を明示的に登録することや、新しい Canister に転送し、カストディアンとして機能できる Principal を指定し、所有権を持たせる必要があります。これらのタスクの一部は、ウェブブラウザで実行されるデフォルトの Cycle ウォレット Dapp を使って実行できます。また、実行したい特定のアクションに応じて、ターミナルで `dfx wallet` コマンドを実行したり、デフォルトの Cycle ウォレット Canister のメソッドを直接呼び出すことで、これらの Cycle と Canister の管理タスクを実行することができます。

しかし、Cycle ウォレット Canister への呼び出しは、現在選択されているユーザー Identity に関連付けられた Cycle ウォレット Principal を使用して実行されることを覚えておく必要があります。現在選択されている Identity や、それに関連する Principal がウォレットのコントローラーまたはカストディアンとして追加されているかどうかによって、異なる結果が表示されたり、特定のメソッドへのアクセスが拒否されたりする場合があります。

現在使用している Identity を確認するには、以下のコマンドを実行します：

    dfx identity whoami

## コントローラーとカストディアンの役割

ユーザー Principal または Canister Principal は、**コントローラー**または**カストディアン**の役割に割り当てることができます。

:::note
このドキュメントで説明するコントローラーの役割は、Internet Computer について話すときに通常意味するコントローラーの役割と同じではありません。通常、コントローラーは、 Canister を制御する Principal を意味します。ここでは、コントローラーは、たまたま同じ名前を持つウォレット内部のロールです。より詳細な区別については、[このフォーラムの投稿](https://forum.dfinity.org/t/why-is-my-cycles-wallet-canister-slowly-losing-cycles/13190/11)を参照してください。
:::

**コントローラー**は最も特権的なロールであり、コントローラーロールに割り当てられた Principal は、以下を含む特権的なタスクを実行することができます：

-   他の Principal をコントローラーとして追加・削除する。

-   他の Principal をカストディアンとして承認および承認解除する。

-   Cycle ウォレットのアドレス帳にエントリーを追加する。

-   Cycle ウォレットの残高と他のすべてのウォレット関連情報にアクセスできる。

-   他の Canister に Cycle を送信する。

-   他の Canister から Cycle の受け取りを許可する。

-   Cycle ウォレットの名前を変更する。

-    Canister と追加の Cycle ウォレットを作成する。

**カストディアン**の役割に割り当てられた Principal は、以下を含む Cycle ウォレット管理タスクのサブセットのみを実行することができます：

-   Cycle ウォレットの残高やその他のウォレット関連情報にアクセスできる。

-   他の Canister に Cycle を送信する。

-   他の Canister から Cycle の受け取りを許可する。

-    Canister を作成する。

 Principal にカストディアンとしての権限を与えても、 Principal に自動的に Cycle ウォレットへのアクセス権が与えられるわけではありません。カストディアンロールに割り当てられた Identity は、Cycle ウォレットの Principal も割り当てられなければなりません。例えば、`alice_custodian` という Identity をローカルプロジェクトの Cycle ウォレット（`rwlgt-iiaaa-aaaaa-cai`）のカストディアンとして承認する場合、そのユーザーも `dfx identity set-wallet rwlgt-iiaaa-aaaaa-cai` コマンドでそのウォレットの使用を割り当てられる必要があります。

## Cycle ウォレット作成

ローカルで開発を行っている場合、Cycle ウォレットは `dfx canister create` を使用して新しい Canister Principal を登録したり、または `dfx deploy` を使用して Canister を登録、ビルド、デプロイするときに作成されます。

Internet Computer にデプロイする場合、Cycle ウォレットが作成され、通常、ICP トークンを Cycle に変換し、Cycle を新しい Canister Principal に転送し、デフォルトの Cycle ウォレット WebAssembly モジュール（WASM）で Canister を更新することが可能です。[ICP トークンを Cycle に変換する](../../quickstart/network-quickstart.md#creating-a-cycles-wallet)では、この方法を説明しています。

ICP を Cycle に変換し、新しい Cycle ウォレットを作成するための Dapps があります。例えば、[NNS Dapp](./../tokenomics/token-holders/nns-app-quickstart#_deploy_a_canister_with_cycles) などです。

##  Cycle 残高の確認

ローカルの Canister 実行環境、または Inernet Computer 上の Cycle ウォレットでは、`dfx wallet balance` コマンドまたは `wallet_balance` メソッドを使用して現在の Cycle  残高を確認することができます。

### ローカル環境で Cycle 残高の確認する

ローカルで開発を行っている場合、`dfx wallet balance` コマンドを使用すると、プロジェクトごとに現在の Cycle 残高をチェックすることができます。

ローカルプロジェクトの Cycle 残高を確認するには：

1.  ターミナルを開き、プロジェクトのルートディレクトリに移動します。

2.  以下のコマンドを実行して、ローカルの Canister 実行環境を起動します：

        dfx start --background

3.  次のコマンドを実行して、現在選択されている Identity に関連付けられた Cycle ウォレットから、Cycle 残高を表示します：

        dfx wallet balance

    このコマンドは、次のような出力を表示します：

        78.000 TC (trillion cycles).

### Internet Computer で Cycle 残高を確認する

Internet Computer に Cycle ウォレットを導入している場合、`dfx wallet balance` コマンドを使用してネットワーク上の現在の Cycle 残高を確認することができます。

Internet Computer 上の Cycle 残高を確認するには：

1.  ターミナルを開き、設定ファイル `dfx.json` が格納されているディレクトリに移動します。

2.  以下のコマンドを実行して、Internet Computer への接続を確認します：

        dfx ping ic

3.  次のコマンドを実行して、現在選択されている Identity に関連する Cycle ウォレットから Cycle 残高を表示します：

        dfx wallet --network ic balance

    このコマンドは、次のような出力を表示します：

        67.992 TC (trillion cycles).

### Cycle wallet_balance メソッドを呼び出す

また、Cycle ウォレットの Canister にある `wallet_balance` メソッドを直接呼び出して、 Cycle の残高を確認することもできます。例えば、あなたの Principal が `h5aet-waaaa-aaaab-qaamq-cai` で Cycle ウォレットのコントローラーである場合、以下のコマンドを実行して現在の Cycle 残高をチェックすることができます：

    dfx canister --network ic call h5aet-waaaa-aaaab-qaamq-cai wallet_balance

このコマンドでは、以下のように金額フィールド（hash 3\_573\_748\_184）と残高 6,895,656,625,450 Cycle を持つレコードとして Candid 形式で残高が返されます：

    (record { 3_573_748_184 = 6_895_656_625_450 })

## その他 dfx がサポートする Wallet 機能

Cycle ウォレットは、dfx を通じてより多くの機能をサポートしています。サポートされている操作の完全なリストについては、[dfx wallet reference](../../../references/cli-reference/dfx-wallet.md) を参照してください。

## デフォルト Cycle ウォレットの追加メソッド

デフォルトの Cycle ウォレット Canister には、`dfx wallet` コマンドとして公開されていない追加のメソッドが含まれています。追加のメソッドは、新しい Canister の作成やイベントの管理など、より高度な Cycle 管理タスクをサポートします。

### 新しい Cycle ウォレットを作成する

`wallet_create_wallet` メソッドを使用して、初期 Cycle 残高と、オプションで特定の Principal をコントローラーとした新しい Cycle ウォレット Canister を作成します。もし、コントロールする Principal を指定しなければ、新しいウォレットを作成するために使用した Cycle ウォレットが新しいウォレットのコントローラーになります。

例えば、以下のようなコマンドを実行して、新しいウォレットを作成し、 Principal をコントローラーとして割り当てます：

    dfx canister --network  call f3yw6-7qaaa-aaaab-qaabq-cai wallet_create_wallet '(record { cycles = 5000000000000 : nat64; controller = principal "vpqee-nujda-46rtu-4noo7-qnxmb-zqs7g-5gvqf-4gy7t-vuprx-u2urx-gqe"})'

このコマンドは、新しいウォレットの Principal を返します：

    (record { 1_313_628_723 = principal "dcxxq-jqaaa-aaaab-qaavq-cai" })

### Canister Principal を新規登録する

Internet Computer に新しい Canister Principal を登録するには、 `wallet_create_canister` メソッドを使用します。このメソッドでは、新しい 「空」の  Canister プレースホルダーを作成し、初期 Cycle 残高と、オプションでそのコントローラーとして特定の Principal を設定できます。 Canister Principal を登録した後、別の手順で Canister 用のコードをインストールできます。

例えば、以下のようなコマンドを実行して、新しいウォレットを作成し、 Principal をコントローラーとして割り当てることができます：

    dfx canister --network  call f3yw6-7qaaa-aaaab-qaabq-cai wallet_create_canister '(record { cycles = 5000000000000 : nat64; controller = principal "vpqee-nujda-46rtu-4noo7-qnxmb-zqs7g-5gvqf-4gy7t-vuprx-u2urx-gqe"})'

このコマンドは、作成した新しい Canister の Principal を返します：

    (record { 1_313_628_723 = principal "dxqg5-iyaaa-aaaab-qaawa-cai" })

### Canister から Cycle を受け取る

Cycle を受信するためのエンドポイントとして `wallet_receive` メソッドを使用します。

### ウォレットから呼び出しをする

呼び出し元として Cycle ウォレット Principal を使用して呼び出しを転送するには、 `wallet_call` メソッドを使用します。

### アドレスを管理する

アドレス帳の項目を管理するには、次の方法を使用します：

-   `add_address`: (address: AddressEntry) → ();

-   `remove_address`: (address: principal) → ();

### イベントを管理する

イベントやチャートの情報を取得するには、以下のメソッドを使用します。

-   `get_events`: (opt record { from: opt nat32; to: opt nat32; }) → (vec Event) query;

-   `get_chart`: (opt record { count: opt nat32; precision: opt nat64; } ) → (vec record { nat64; nat64; }) query;

例えば、以下のようなコマンドを実行することで、 `get_events` メソッドを使用して `canister_create` などのイベントを返すことができます：

    dfx canister call <cycles-wallet-principal> get_events '(record {from = null; to = null})'

Cycle ウォレット（`gastn-uqaaa-aaaaafq-cai`）が Internet Computer のメインネットワークにデプロイされている場合、次のようなコマンドを実行するとイベントを返すことができます：

    dfx canister --network ic call gastn-uqaaa-aaaae-aaafq-cai get_events '(record {from = null; to = null})'

コマンドの出力は、以下のような Candid 形式です：

    (
      vec { record { 23_515 = 0; 1_191_829_844 = variant { 4_271_600_268 = record { 23_515 = principal "tsqwz-udeik-5migd-ehrev-pvoqv-szx2g-akh5s-fkyqc-zy6q7-snav6-uqe"; 1_224_700_491 = null; 1_269_754_742 = variant { 4_218_395_836 };} }; 2_781_795_542 = 1_621_456_688_636_513_683;}; record { 23_515 = 1; 1_191_829_844 = variant { 4_271_600_268 = record { 23_515 = principal "ejta3-neil3-qek6c-i7rdw-sxreh-lypfe-v6hjg-6so7x-5ugze-3iohr-2qe"; 1_224_700_491 = null; 1_269_754_742 = variant { 2_494_206_670 };} }; 2_781_795_542 = 1_621_461_468_638_569_551;}; record { 23_515 = 2; 1_191_829_844 = variant { 1_205_528_161 = record { 2_190_693_645 = 11_000_000_000_000; 2_631_180_839 = principal "gvvca-vyaaa-aaaae-aaaga-cai";} }; 2_781_795_542 = 1_621_462_573_993_647_258;}; record { 23_515 = 3; 1_191_829_844 = variant { 1_205_528_161 = record { 2_190_693_645 = 11_000_000_000_000; 2_631_180_839 = principal "gsueu-yaaaa-aaaae-aaagq-cai";} }; 2_781_795_542 = 1_621_462_579_193_578_440;}; record { 23_515 = 4; 1_191_829_844 = variant { 1_955_698_212 = record { 2_190_693_645 = 0; 2_374_371_241 = "install_code"; 2_631_180_839 = principal "aaaaa-aa";} }; 2_781_795_542 = 1_621_462_593_047_590_026;}; record { 23_515 = 5; 1_191_829_844 = variant { 1_955_698_212 = record { 2_190_693_645 = 0; 2_374_371_241 = "install_code"; 2_631_180_839 = principal "aaaaa-aa";} }; 2_781_795_542 = 1_621_462_605_779_157_885;}; record { 23_515 = 6; 1_191_829_844 = variant { 1_955_698_212 = record { 2_190_693_645 = 0; 2_374_371_241 = "authorize"; 2_631_180_839 = principal "gsueu-yaaaa-aaaae-aaagq-cai";} }; 2_781_795_542 = 1_621_462_609_036_146_536;}; record { 23_515 = 7; 1_191_829_844 = variant { 1_955_698_212 = record { 2_190_693_645 = 0; 2_374_371_241 = "greet"; 2_631_180_839 = principal "gvvca-vyaaa-aaaae-aaaga-cai";} }; 2_781_795_542 = 1_621_463_144_066_333_270;}; record { 23_515 = 8; 1_191_829_844 = variant { 4_271_600_268 = record { 23_515 = principal "ejta3-neil3-qek6c-i7rdw-sxreh-lypfe-v6hjg-6so7x-5ugze-3iohr-2qe"; 1_224_700_491 = null; 1_269_754_742 = variant { 2_494_206_670 };} }; 2_781_795_542 = 1_621_463_212_828_477_570;}; record { 23_515 = 9; 1_191_829_844 = variant { 1_955_698_212 = record { 2_190_693_645 = 0; 2_374_371_241 = "wallet_balance"; 2_631_180_839 = principal "gastn-uqaaa-aaaae-aaafq-cai";} }; 2_781_795_542 = 1_621_878_637_071_884_946;}; record { 23_515 = 10; 1_191_829_844 = variant { 4_271_600_268 = record { 23_515 = principal "b5quc-npdph-l6qp4-kur4u-oxljq-7uddl-vfdo6-x2uo5-6y4a6-4pt6v-7qe"; 1_224_700_491 = null; 1_269_754_742 = variant { 4_218_395_836 };} }; 2_781_795_542 = 1_621_879_473_916_547_313;}; record { 23_515 = 11; 1_191_829_844 = variant { 313_999_214 = record { 1_136_829_802 = principal "gastn-uqaaa-aaaae-aaafq-cai"; 3_573_748_184 = 10_000_000_000;} }; 2_781_795_542 = 1_621_977_470_023_492_664;}; record { 23_515 = 12; 1_191_829_844 = variant { 2_171_739_429 = record { 25_979 = principal "gastn-uqaaa-aaaae-aaafq-cai"; 3_573_748_184 = 10_000_000_000; 4_293_698_680 = 0;} }; 2_781_795_542 = 1_621_977_470_858_839_320;};},
    )

この例では、12個のイベントレコードがあります。Role フィールド（hash `1_269_754_742`）は、Principal がコントローラー（hash `4_218_395_836`）かカストディアン（hash `2_494_206_670`）かを指定するものです。この例のイベントは、10,000,000,000 Cycle の転送を行う金額フィールド（hash `3_573_748_184` で表される）も例示しています。

<!--
# Cycles Wallet

As discussed in [Tokens and cycles](../../../concepts/tokens-cycles), ICP tokens can be converted into **cycles** to power canister operations. Cycles reflect the operational cost of communication, computation, and storage that dapps consume.

Unlike ICP tokens, cycles are only associated with canisters and not user or developer principals. Because only canisters require and consume cycles—to perform operations and to pay for the resources they use—users and developers manage the distribution and ownership of cycles through a special type of canister called a **cycles wallet**. Because the cycles wallet holds the cycles required to perform operations such as creating new canisters, these operations are executed using the canister principal for the cycles wallet instead of your user principal.

For the purposes of using the local canister execution environment, the SDK automatically creates a default cycles wallet for you in every project and most of the operations performed using the cycles wallet happen behind the scenes. For example, the cycles wallet acts on your behalf to register canister principals and deploy canisters in the local canister execution environment.

In a production environment, however, you need to explicitly register and transfer cycles to new canisters, specify the principals that can act as custodians, and manage the principals with ownership rights. You can perform some of these tasks using the default cycles wallet dapp running in a web browser. Depending on the specific action you want to take, you can also perform these cycle and canister management tasks by running `dfx wallet` commands in a terminal or by calling methods in the default cycles wallet canister directly.

You should keep in mind, however, that calls to the cycles wallet canister are executed using the cycles wallet principal associated with the currently-selected user identity. Depending on your currently-selected identity and whether the principal associated with that identity has been added as a controller or a custodian for a wallet, you might see different results or be denied access to a specific method.

To check the identity you are currently using, run the following command:

    dfx identity whoami

## Controller and Custodian Roles

A user principal or canister principal can be assigned to a **controller** or **custodian** role.

:::note
The controller role this document talks about is NOT the same controller role usually meant when talking about the Internet Computer. Typically, the controller refers to a principal that controls a canister. Here, a controller is a wallet-internal role that happens to have the same name. For a more detailed differentiation, see [this forum post](https://forum.dfinity.org/t/why-is-my-cycles-wallet-canister-slowly-losing-cycles/13190/11).
:::

A **controller** is the most privileged role and a principal assigned to the controller role can perform privileged tasks including the following:

-   Add and remove other principals as controllers.

-   Authorize and de-authorize other principals as custodians.

-   Add entries to the cycles wallet address book.

-   Access the cycles wallet balance and all other wallet-related information.

-   Send cycles to other canisters.

-   Accept receipt of cycles from other canisters.

-   Rename the cycles wallet.

-   Create canisters and additional cycles wallets.

A principal assigned to the **custodian** role can only perform a subset of cycles wallet management tasks, including the following:

-   Access the cycles wallet balance and all other wallet-related information.

-   Send cycles to other canisters.

-   Accept receipt of cycles from other canisters.

-   Create canisters.

Authorizing a principal as a custodian does not automatically grant the principal access to a cycles wallet. The identity assigned to the custodian role must also be assigned a cycles wallet principal. For example, if you authorize the identity `alice_custodian` as a custodian of a cycles wallet (`rwlgt-iiaaa-aaaaa-aaaaa-cai`) in a local project, that user would also need to be assigned to use that wallet with the `dfx identity set-wallet rwlgt-iiaaa-aaaaa-aaaaa-cai` command.

## Create a Cycles Wallet

If you are doing local development, your cycles wallet is created when you register a new canister principal using `dfx canister create` or when you register, build, and deploy a canister with `dfx deploy`.

If you are deploying on the Internet Computer, you typically create your cycles wallet by converting ICP tokens to cycles, transferring the cycles to a new canister principal, and updating the canister with the default cycles wallet WebAssembly module (WASM). [Convert ICP to cycles](../../quickstart/network-quickstart.md#creating-a-cycles-wallet) shows how to do this.

There are dapps that can help you convert ICP to cycles and create a new cycles wallet, e.g., [NNS dapp](../../../tokenomics/token-holders/nns-app-quickstart#_deploy_a_canister_with_cycles).

## Check the Cycle Balance

In the local canister execution environment or with a cycles wallet on the Internet Computer, you can use the `dfx wallet balance` command or the `wallet_balance` method to check the current cycle balance.

### Check your cycles balance when developing locally

If you are doing local development, you can use the `dfx wallet balance` command to check the current cycles balance on a project-by-project basis.

To check the cycles balance in a local project:

1.  Open a terminal and navigate to the root directory of the project.

2.  Start the local canister execution environment by running the following command:

        dfx start --background

3.  Display the cycles balance from the cycles wallet associated with the currently-selected identity by running the following command:

        dfx wallet balance

    The command displays output similar to the following:

        78.000 TC (trillion cycles).

### Check the cycles balance on the Internet Computer

If you have deployed a cycles wallet on the Internet Computer, you can use the `dfx wallet balance` command to check the current cycles balance on the network.

To check the cycles balance on the Internet Computer:

1.  Open a terminal and navigate to a directory that contains a `dfx.json` configuration file.

2.  Check your connection to the Internet Computer by running the following command:

        dfx ping ic

3.  Display the cycle balance from the cycles wallet associated with the currently-selected identity by running the following command:

        dfx wallet --network ic balance

    The command displays output similar to the following:

        67.992 TC (trillion cycles).

### Call the cycles wallet\_balance method

You can also check the cycles balance by calling the `wallet_balance` method in the cycles wallet canister directly. For example, if your principal is a controller for the `h5aet-waaaa-aaaab-qaamq-cai` cycles wallet, you can check the current cycle balance by running the following command:

    dfx canister --network ic call h5aet-waaaa-aaaab-qaamq-cai wallet_balance

The command returns the balance using Candid format as a record with an amount field (represented by the hash 3\_573\_748\_184) and a balance of 6,895,656,625,450 cycles like this:

    (record { 3_573_748_184 = 6_895_656_625_450 })

## Other dfx-supported Wallet Functions

The cycles wallet supports a lot more functions through dfx. For a full list of supported operations, see the [dfx wallet reference](../../../references/cli-reference/dfx-wallet.md).

## Additional Methods in the Default Cycles Wallet

The default cycles wallet canister includes additional methods that are not exposed as `dfx wallet` commands. The additional methods support more advanced cycles management tasks such as creating new canisters and managing events.

### Create a new cycles wallet

Use the `wallet_create_wallet` method to create a new cycles wallet canister with an initial cycle balance and, optionally, with a specific principal as its controller. If you don’t specify a controlling principal, the cycles wallet you use to create the new wallet will be the new wallet’s controller.

For example, you can run a command similar to the following to create a new wallet and assign a principal as a controller:

    dfx canister --network  call f3yw6-7qaaa-aaaab-qaabq-cai wallet_create_wallet '(record { cycles = 5000000000000 : nat64; controller = principal "vpqee-nujda-46rtu-4noo7-qnxmb-zqs7g-5gvqf-4gy7t-vuprx-u2urx-gqe"})'

The command returns the principal for the new wallet:

    (record { 1_313_628_723 = principal "dcxxq-jqaaa-aaaab-qaavq-cai" })

### Register a new canister principal

Use the `wallet_create_canister` method to register a new canister principal on the Internet Computer. This method creates a new "empty" canister placeholder with an initial cycle balance and, optionally, with a specific principal as its controller. After you have registered the canister principal, you can install code for your canister as a separate step.

For example, you can run a command similar to the following to create a new wallet and assign a principal as a controller:

    dfx canister --network  call f3yw6-7qaaa-aaaab-qaabq-cai wallet_create_canister '(record { cycles = 5000000000000 : nat64; controller = principal "vpqee-nujda-46rtu-4noo7-qnxmb-zqs7g-5gvqf-4gy7t-vuprx-u2urx-gqe"})'

The command returns the principal for the new canister you created:

    (record { 1_313_628_723 = principal "dxqg5-iyaaa-aaaab-qaawa-cai" })

### Receive cycles from a canister

Use the `wallet_receive` method as an endpoint to receive cycles.

### Forward calls from a wallet

Use the `wallet_call` method to forward calls using the cycles wallet principal as caller.

### Manage addresses

Use the following methods to manage address book entries:

-   `add_address`: (address: AddressEntry) → ();

-   `remove_address`: (address: principal) → ();

### Manage events

Use the following methods to retrieve event and chart information.

-   `get_events`: (opt record { from: opt nat32; to: opt nat32; }) → (vec Event) query;

-   `get_chart`: (opt record { count: opt nat32; precision: opt nat64; } ) → (vec record { nat64; nat64; }) query;

For example, you can use the `get_events` method to return `canister_create` and other events by running a command similar to the following:

    dfx canister call <cycles-wallet-principal> get_events '(record {from = null; to = null})'

If the cycles wallet (`gastn-uqaaa-aaaae-aaafq-cai`) is deployed on the Internet Computer main network, you could run a command that looks like this to return events:

    dfx canister --network ic call gastn-uqaaa-aaaae-aaafq-cai get_events '(record {from = null; to = null})'

The output from the command is in Candid format similar to the following:

    (
      vec { record { 23_515 = 0; 1_191_829_844 = variant { 4_271_600_268 = record { 23_515 = principal "tsqwz-udeik-5migd-ehrev-pvoqv-szx2g-akh5s-fkyqc-zy6q7-snav6-uqe"; 1_224_700_491 = null; 1_269_754_742 = variant { 4_218_395_836 };} }; 2_781_795_542 = 1_621_456_688_636_513_683;}; record { 23_515 = 1; 1_191_829_844 = variant { 4_271_600_268 = record { 23_515 = principal "ejta3-neil3-qek6c-i7rdw-sxreh-lypfe-v6hjg-6so7x-5ugze-3iohr-2qe"; 1_224_700_491 = null; 1_269_754_742 = variant { 2_494_206_670 };} }; 2_781_795_542 = 1_621_461_468_638_569_551;}; record { 23_515 = 2; 1_191_829_844 = variant { 1_205_528_161 = record { 2_190_693_645 = 11_000_000_000_000; 2_631_180_839 = principal "gvvca-vyaaa-aaaae-aaaga-cai";} }; 2_781_795_542 = 1_621_462_573_993_647_258;}; record { 23_515 = 3; 1_191_829_844 = variant { 1_205_528_161 = record { 2_190_693_645 = 11_000_000_000_000; 2_631_180_839 = principal "gsueu-yaaaa-aaaae-aaagq-cai";} }; 2_781_795_542 = 1_621_462_579_193_578_440;}; record { 23_515 = 4; 1_191_829_844 = variant { 1_955_698_212 = record { 2_190_693_645 = 0; 2_374_371_241 = "install_code"; 2_631_180_839 = principal "aaaaa-aa";} }; 2_781_795_542 = 1_621_462_593_047_590_026;}; record { 23_515 = 5; 1_191_829_844 = variant { 1_955_698_212 = record { 2_190_693_645 = 0; 2_374_371_241 = "install_code"; 2_631_180_839 = principal "aaaaa-aa";} }; 2_781_795_542 = 1_621_462_605_779_157_885;}; record { 23_515 = 6; 1_191_829_844 = variant { 1_955_698_212 = record { 2_190_693_645 = 0; 2_374_371_241 = "authorize"; 2_631_180_839 = principal "gsueu-yaaaa-aaaae-aaagq-cai";} }; 2_781_795_542 = 1_621_462_609_036_146_536;}; record { 23_515 = 7; 1_191_829_844 = variant { 1_955_698_212 = record { 2_190_693_645 = 0; 2_374_371_241 = "greet"; 2_631_180_839 = principal "gvvca-vyaaa-aaaae-aaaga-cai";} }; 2_781_795_542 = 1_621_463_144_066_333_270;}; record { 23_515 = 8; 1_191_829_844 = variant { 4_271_600_268 = record { 23_515 = principal "ejta3-neil3-qek6c-i7rdw-sxreh-lypfe-v6hjg-6so7x-5ugze-3iohr-2qe"; 1_224_700_491 = null; 1_269_754_742 = variant { 2_494_206_670 };} }; 2_781_795_542 = 1_621_463_212_828_477_570;}; record { 23_515 = 9; 1_191_829_844 = variant { 1_955_698_212 = record { 2_190_693_645 = 0; 2_374_371_241 = "wallet_balance"; 2_631_180_839 = principal "gastn-uqaaa-aaaae-aaafq-cai";} }; 2_781_795_542 = 1_621_878_637_071_884_946;}; record { 23_515 = 10; 1_191_829_844 = variant { 4_271_600_268 = record { 23_515 = principal "b5quc-npdph-l6qp4-kur4u-oxljq-7uddl-vfdo6-x2uo5-6y4a6-4pt6v-7qe"; 1_224_700_491 = null; 1_269_754_742 = variant { 4_218_395_836 };} }; 2_781_795_542 = 1_621_879_473_916_547_313;}; record { 23_515 = 11; 1_191_829_844 = variant { 313_999_214 = record { 1_136_829_802 = principal "gastn-uqaaa-aaaae-aaafq-cai"; 3_573_748_184 = 10_000_000_000;} }; 2_781_795_542 = 1_621_977_470_023_492_664;}; record { 23_515 = 12; 1_191_829_844 = variant { 2_171_739_429 = record { 25_979 = principal "gastn-uqaaa-aaaae-aaafq-cai"; 3_573_748_184 = 10_000_000_000; 4_293_698_680 = 0;} }; 2_781_795_542 = 1_621_977_470_858_839_320;};},
    )

In this example, there are twelve event records. The Role field (represented by the hash `1_269_754_742`) specifies whether a principal is a controller (represented by the hash `4_218_395_836`) or a custodian (represented by the hash `2_494_206_670`). The events in this example also illustrate an amount field (represented by the hash `3_573_748_184`) with a transfer of 10,000,000,000 cycles.

-->