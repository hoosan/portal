# カスタムドメイン

Internet Computer ブロックチェーン上の Web3 Dapps を含むすべてのスマートコントラクトは、ルートキーによって保護されています。エンドツーエンドのセキュリティは、ブラウザに組み込まれたプロキシであるサービスワーカーによって提供され、 Internet Computer ブロックチェーンからダウンロードされたデータの整合性を検証します。

このガイドでは、特定の Canister に対してエンドツーエンドのセキュリティを備えたカスタムドメインを可能にするカスタムサービスワーカーを構築する方法を説明します。サービスワーカーは、インターネットに接続された任意のデバイスから静的アセットとして提供することができ、サービスワーカーがロードされた後、すべてのデータはクライアントと Internet Computer ブロックチェーン間で直接転送されます。

DNS を制御することでサイトのリダイレクトが可能になり、TLS 証明書の制御も可能になるため、標準的な Web 技術を使用するサイトのセキュリティは最終的に DNS に依存することになります。したがって、標準的な Web サイトでは、少なくとも DNS レジストラを信頼する必要があります。もしレジストラが静的ホスティングを提供するならば、カスタムサービスワーカーをデプロイすることで、信頼すべきエンティティを増やすことなく、Internet Computer のブロックチェーン上の標準的な Web3 Dapps にエンドツーエンドのセキュリティを提供することができます。

## カスタムサービスワーカーを作成する

1. Canister を IC にデプロイし、 Canister ID を記録してください。
2. [IC 公式レポ](https://github.com/dfinity/ic) をクローンし、`ic/typescript/service-worker` にある[service worker フォルダ](https://github.com/dfinity/ic/tree/master/typescript/service-worker)に移動します。
3. ファイル `service-worker/src/sw/http_request.ts` 内の `hostnameCanisterIdMap` にドメインと Canister のマッピングを追加し、 Canister  ID をマッピングしてください。
4. サービスワーカーを `service-worker/README.md` の説明に従ってビルドします。出力は以下のようになるはずです：
    - `index.html`
    - ミニファイされた `.js` ファイル
    - `.map` ファイル
5. サーバーまたは CDN からアセット（`index.html`、`.js`、`.map`ファイル）をホスティングし、カスタムドメイン名をこのサーバーに向けます。
6. テスト

:::caution
Internet Identity（II）を使用してユーザーを認証する Web サイト向け：
II によって提供される Principal は、ログイン要求が開始されたドメインに依存します。そのため、 Canister URL を通じてユーザーを認証している場合に、カスタムドメインに切り替えようとすると、ユーザーは同じ Principal を持てなくなります。これを防ぐには、[Alternative Origins](../../references/ii-spec.md#alternative-frontend-origins) を設定する必要があります。
:::

<!--
# Custom Domains

All smart contracts, including Web3 dapps, on the Internet Computer blockchain are secured by the root key. End-to-end security is provided by a service worker, a proxy embedded in the browser, which verifies the integrity of data downloaded from the Internet Computer blockchain.

This guide shows how to build a custom service worker which enables a custom domain with end-to-end security for a specific canister. The service worker can be served as static assets from any internet-connected device and after the service worker is loaded, all data is transferred directly between the client and the Internet Computer blockchain.

Ultimately the security of any site using standard web technology depends on DNS since control of DNS allows the site to be redirected and enables control of TLS certificates. Consequently, for a standard website trust must be placed at least in the DNS registrar. If the registrar provides static hosting, deployment of a custom service worker can provide end-to-end security for standard Web3 dapps on the Internet Computer blockchain without increasing the number of entities that must be trusted.

## Creating the custom Service Worker

1. Deploy your canister to the IC and note the canister id.
1. Clone the [official IC repo](https://github.com/dfinity/ic) and navigate to the [service worker folder](https://github.com/dfinity/ic/tree/master/typescript/service-worker) located under `ic/typescript/service-worker`.
1. Map your domain to the canister ID by adding your domain-to-canister mapping to `hostnameCanisterIdMap` in the file `service-worker/src/sw/http_request.ts`.
1. Build the service worker according to the instructions in `service-worker/README.md`. The output should be:
    - an `index.html`,
    - a minified `.js` file, and
    - a `.map` file.
1. Host the assets (`index.html`, `.js` and `.map` files) from a server or CDN and point your custom domain name at this server.
1. Test.

:::caution
For websites that use Internet Identity (II) to authenticate users: The principals provided by II depend on the domain from which the login request was started. So if you authenticate your users through the canister URL and want to switch over to a custom domain, users will not have the same principals anymore. You can prevent this by setting up [Alternative Origins](../../references/ii-spec.md#alternative-frontend-origins).
:::

-->