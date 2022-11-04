# 代替フロントエンドオリジン

アプリケーションがドメイン名を変更する段階になり、Internet Identity で認証を行っている場合、ユーザーがこれまでと同じ Principal をシームレスに使用できるようにしたいと思うことでしょう。この機能をサポートするために、このガイドを使用して代替フロントエンドオリジン用にアプリケーションを設定することができます。

![End Result](../_attachments/alternative-origins.png)

以下のような場合に、このガイドが必要になるはずです：

- `<canister-id>.ic0.app` からカスタムドメインへ移動する
- ユーザーに `/` ではなく `/login` でログインするように要求する
- `raw.ic0.app` を使用するユーザーをサポートする
- 組織内の複数のアプリが同じ Principal を使用するように設定する
- その他

## 制約

現在、最大 **10** の代替オリジンをリストアップすることができます。

II は、これらの代替を構成するオリジンが、Certified Assets を使用する Canister で ホストされている場合にのみ、この仕様に従います。

詳細については、[Internet Identity Spec](https://github.com/dfinity/internet-identity/blob/main/docs/ii-spec.md#alternative-frontend-origins) を参照してください。

## 代替オリジンを設定する

この例では、**A** と **B** という2つのドメインを用意します。**A** が正規のオリジンで、**B** が代替ドメインとなります。このモデルを説明するために、https://internetcomputer.org と https://hwvjt-wqaaa-aaaam-qadra-cai.ic0.app の両方でホストされているこの Web サイトを考えてみましょう。

この例では、**A** が Canister ID、つまり、https://hwvjt-wqaaa-aaaam-qadra-cai.ic0.app になります。

**B** は代替オリジン、すなわち、https://internetcomputer.org です。

### オリジンをリストする

オリジン **A** に対して、Internet Identity に **B** が有効なオリジンであることを伝えるファイルを提供する必要があります。この例では、設定ファイルを `src/assets` に配置することにします。アセット Canister が現在 `dist` フォルダからアセットをデプロイするように設定されている場合、Canister の `sources` を更新して両方を含めるようにしてください。

```json
"source": [
    "dist",
    "src/assets"
]
```

`src/assets` の中に `.well-known` フォルダを作り、 `ii-alternative-origins` という名前のファイルを追加します。

:::note

ファイル名は `ii-alternative-origins` とし、拡張子はつけないこと。中のコンテンツは JSON でフォーマットされていますが、ファイルの末尾に `.json` を付けないようにしてください。

:::

このファイルの中で、**B** の代替オリジンをリストアップしてください。このようになります：

```json
{
  "alternativeOrigins": ["https://internetcomputer.org"]
}
```

:::note

オリジンから末尾のスラッシュやクエリパラメータを削除していることを確認してください。

:::

あなたのプロジェクトは次のようなっているはずです：

```
├── dfx.json
├── src
│   ├── assets
│   │   ├── .well-known
│   │   │   └── ii-alternative-origins
```

### アセット Canister を設定する

`.well-known`のドット構文は通常、ファイルシステムによって “隠しファイル" として扱われるため、ドキュメントをアップロードするためにはアセット Canister を設定することが必要です。アセット Canister を設定するには、`.ic-assets.json` というファイルを新規に作成します。`.ic-assets.json` は Canister の `sources` にリストされているディレクトリの中に置く必要があります。新しいファイル一覧はこのようになります。

```
├── dfx.json
├── package.json
├── src
│   ├── assets
│   │   ├── .ic-assets.json
│   │   ├── .well-known
│   │   │   └── ii-alternative-origins
```

次に、`.well-known` ディレクトリが含まれるように下記のように設定します。

```json
[
  {
    "match": ".well-known",
    "ignore": false
  },
  {
    "match": ".well-known/ii-alternative-origins",
    "headers": {
      "Access-Control-Allow-Origin": "*",
      "Content-Type": "application/json"
    },
    "ignore": false
  }
]
```

これには、`.well-known` ディレクトリを無視しない一般的なルールと、`ii-alternative-origins` をアクセス制御とコンテントタイプヘッダー付きで配信するルールが含まれます。

あとは、Canister を配備するだけです。これ以降、オリジン **B** から認証を試みると、**A** を使用しているときと同じ Principal が返されます。


<!--
# Alternative Frontend Origins

If your application has reached the stage where you want to change domain names, and you have been authenticating with Internet Identity, you will want to make sure that your users can seamlessly keep the same Principals they have already been using. To support this functionality, you can configure your application for Alternative Frontend origins using this guide.

![End Result](../_attachments/alternative-origins.png)

You may need this guide if you are doing any of the following:

- Moving from `<canister-id>.ic0.app` to a custom domain
- Asking users to login at `/login` instead of `/`
- Supporting users using `raw.ic0.app`
- Configuring multiple apps in your organization to use the same principals
- And more

## Constraints

Currently, a maximum of **10** alternative origins can be listed.

II will only follow this specification when the origin configuring these alternatives is hosted on a canister using Certified Assets.

For more information, see the [Internet Identity Spec](https://github.com/dfinity/internet-identity/blob/main/docs/ii-spec.md#alternative-frontend-origins).

## Configuring Alternative Origins

For this example, we will have two domains, **A** and **B**. **A** will be the canonical origin, and **B** will be the alternative domain. To help illustrate this model, consider this website, which is hosted both at https://internetcomputer.org and https://hwvjt-wqaaa-aaaam-qadra-cai.ic0.app.

In this example, **A** would be the canister ID, or https://hwvjt-wqaaa-aaaam-qadra-cai.ic0.app.

**B** would be the alternative origin, or https://internetcomputer.org.

### Listing origins

For origin **A**, you will need to provide a file that tells Internet Identity that **B** is a valid origin. We'll be placing the config files in `src/assets` for this example. If your asset canister is currently configured to deploy assets from a `dist` folder, make sure to update the `sources` for your canister to include both:

```json
"source": [
    "dist",
    "src/assets"
]
```

Inside of `src/assets`, create a `.well-known` folder, and add a file named `ii-alternative-origins.`

:::note

The file needs to exactly be named `ii-alternative-origins`, with no file extension. The content inside will be formatted as JSON, but the file should not end with `.json`.

:::

Inside of the file, list your alternative origin for **B**. It will look something like this:

```json
{
  "alternativeOrigins": ["https://internetcomputer.org"]
}
```

:::note

Make sure that you remove any trailing slash or query parameters from the origin

:::

Now, your project should look something like this:

```
├── dfx.json
├── src
│   ├── assets
│   │   ├── .well-known
│   │   │   └── ii-alternative-origins
```

### Configuring Your Asset Canister

Because the dot syntax in `.well-known` ordinarily will be treated as "hidden" by the file system, the asset canister will need to be configured to upload your document. To configure the asset canister, create a new file, `.ic-assets.json`. `.ic-assets.json` needs to be placed inside a directory listed in `sources` for your canister, so we can use `src/assets` again. Your new list of files should look like this:

```
├── dfx.json
├── package.json
├── src
│   ├── assets
│   │   ├── .ic-assets.json
│   │   ├── .well-known
│   │   │   └── ii-alternative-origins
```

Then, configure the `.well-known` directory to be included, with

```json
[
  {
    "match": ".well-known",
    "ignore": false
  },
  {
    "match": ".well-known/ii-alternative-origins",
    "headers": {
      "Access-Control-Allow-Origin": "*",
      "Content-Type": "application/json"
    },
    "ignore": false
  }
]
```

This includes a general rule to not ignore the `.well-known` directory, and rules to deliver the `ii-alternative-origins` with access control and content-type headers.

Then, all you need to do is deploy your canister. When you attempt to authenticate from origin **B** from then on, you will get back the same principal you get while using **A**.

-->