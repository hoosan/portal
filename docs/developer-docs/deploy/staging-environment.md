# ステージング環境

多くのプロジェクトでは、通常のローカルおよびライブデプロイメントの他に、ステージング環境を利用できるようにすることで利益を得ることができます。このページでは、そのようなステージング環境をセットアップする方法を説明します。

## ステージング環境のメリット

_why_ から始めることに意味があります。なぜプロジェクトはステージング環境を望むのでしょうか？最も大きな理由のひとつは、テストです。
本番環境にデプロイする前に、機能をエンドツーエンドでテストできる別のデプロイメント環境を用意することは、非常に有効です。
開発者（と、おそらく一部のベータユーザー）は、実際の環境で自分自身でテストできるようになります。
ローカルでのデプロイは、ライブの Intenrnet Computer をできるだけ忠実に反映させますが、それでも同じではありません。
例えば、ローカルインスタンスは、単一のサブネットとしてのみ動作します。ふたつの異なるサブネットの Canister を互いに接続してテストしたい場合、ローカルでこれを行うことはできません。

さらに、ステージング環境を持つ理由としては、以下のようなことが挙げられます：
- 他のサービスとの統合テスト
- デプロイワークフローのテスト
- 全ユーザーを対象とした機能の本番稼働前のコスト見積もり。
- エンドツーエンドのテスト

## ステージング環境のセットアップ

このセクションでは、ステージング環境を設定する方法を説明します。ステージング環境があれば、 `dfx` コマンドを実行するときに、 `--network ic` を指定する代わりに `--network my-staging` を指定することができるようになります。
もちろん、 `my-staging` という名前は他の任意の名前に置き換えることができます（ただし、予約されている 2 つの名前は除きます：`ic` はライブの Internet Computer、 `local` は `dfx start` で実行されるデフォルトネットワークです）。

ネットワーク（この文脈では「環境」ともいう）には、想定されたものと明示的に設定されたもののふたつの方法があります。Dfx は想定されたネットワークとして、`ic` ネットワークだけを含んでいます。
`—network ic` を（ほとんど）どのコマンドにも追加すると、そのコマンドは Internet Computer 環境のコンテキストで実行されます。
他のすべてのネットワークは `dfx.json` で明示的に設定されます。任意の `dfx.json` （例えば `dfx new` で生成された新しいもの）を見てみると、 `"networks"` セクションに少なくとも `local` ネットワークが含まれているはずです。
この `local` ネットワークは、 `--network` で他のネットワークが指定されていない場合に、デフォルトで選択されるネットワークです。

### ネットワーク定義

`my-staging` という名前のステージングネットワークを `dfx.json` に追加するには、`dfx.json` の `"networks"` の下に以下のように追加してください：

``` json
"my-staging": {
    "providers": [
        "https://ic0.app"
    ],
    "type": "persistent"
}
```

この `"providers"` の値は、`dfx` にネットワークに接続する場所を指定します。これはハードコードされた `ic` ネットワークのものと同じです。
`persistent` という "type" は、このネットワーク上の Canister がそこに留まることを `dfx` に伝えます。そのため、Canister 識別子は `canister_ids.json` ファイルに保存されます。

### ウォレットの設定

デフォルトで使用する Cycle ウォレットは、ネットワークごとに別々に保存されています。このため、新しく作成された `my-staging` ネットワークには、まだウォレットが設定されていません。
ほとんどの人はメインの `ic` ネットワークと同じ Cycle ウォレットを使いたいででしょう。そのためには、正しい ID が設定されていることを確認します（`dfx identity use <identity name>`）。
次に、`dfx identity get-wallet --network ic` を使用して、`ic` ネットワークの現在設定されているウォレットを読み取ります。
最後に、`dfx identity set-wallet <wallet id> --network my-staging` で新しく定義したネットワークにウォレットを設定します。
あるいは、このふたつを組み合わせてワンライナーにします：

``` bash
dfx identity set-wallet "$(dfx identity get-wallet --network ic)" --network my-staging
```

ステージング環境用に別の Cycle ウォレットを使用したい場合は、[network quickstart](../quickstart/network-quickstart.md) のステップ 'Creating a Cycles Wallet' にある指示に従ってください

<!--
# Staging Environment

Many projects can benefit from having a staging environment available besides the usual local and live deployments. This page explains how to set up such a staging environment.

## Benefits of a staging environment

It makes sense to start with the _why_. Why would a project want a staing environment? One of the biggest reasons is testing.
Having a separate deployment environment available where features can be end-to-end tested before they get deployed to the production environment is very helpful.
It allows developers (and maybe some beta users) to try things for themselves in a real environment.
While local deployments mirror the live Intenrnet Computer as closely as possible, it is still not the same.
For example, the local instance only runs as a single subnet. If you want to test canisters of two different subnets connecting to each other, you cannot do this locally.

Some more reasons for having a staging environment are:
- Testing integration with other services.
- Testing deployment workflows.
- Estimating costs before setting a feature live for all users.
- End-to-end testing.

## Setting up a staging environment

This section shows how to configure a staging environment. With a working staging environment it is possible to run any `dfx` command that would otherwise take `--network ic` with `--network my-staging` instead.
Of course, the name `my-staging` can be replaced with any other name (except the two reserved ones: `ic`, the built in live Internet Computer and `local`, the implicit default network that runs with `dfx start`).

Networks (or also 'environments' in this context) are defined in two ways: assumed and explicitly configured. Dfx only contains one network as an assumed network: the `ic` network.
If you add `--network ic` to (almost) any command, it will run in the context of the live Internet Computer environment.
All other networks are explicitly configured in `dfx.json`. Looking at any random `dfx.json` (e.g. a fresh one generated with `dfx new`), the `"networks"` section should contain at least the `local` network.
The `local` network is the network that gets chosen by default if no other network is specified with `--network`.

### Network definition

To add a staging network named `my-staging` to `dfx.json`, add this under `"networks"` in your `dfx.json`:

``` json
"my-staging": {
    "providers": [
        "https://ic0.app"
    ],
    "type": "persistent"
}
```

This value for `"providers"` tells `dfx` where to connect to the network. It is identical to the one in the hard-coded `ic` network.
The type `persistent` tells `dfx` that the canisters on this network will stay there. Because of that, the canister identifiers will be saved in the `canister_ids.json` file.

### Configuring a wallet

Which cycles wallet to use by default is stored separately for every network. Because of this, the newly created `my-staging` network has no wallet configured yet.
Most people will just want to use the same cycles wallet as on the main `ic` network. To do so, make sure the correct identity is set (`dfx identity use <identity name>`).
Then, read the `ic` network's currently configured wallet using `dfx identity get-wallet --network ic`.
Finally, set the wallet for the newly defined network with `dfx identity set-wallet <wallet id> --network my-staging`.
Or, combining the two into a one-liner:

``` bash
dfx identity set-wallet "$(dfx identity get-wallet --network ic)" --network my-staging
```

If you prefer to use a separate cycles wallet for the staging environment, follow the instructions in the step 'Creating a Cycles Wallet' in the [network quickstart](../quickstart/network-quickstart.md).

-->
