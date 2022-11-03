#  Internet Identity の統合

これは、Internet Identity を使用してプロジェクトを統合し、テストする方法を示しています。Internet Identity の開発版 [build flavor](https://github.com/dfinity/internet-identity/blob/main/README.md#build-features-and-flavors) と [agent-js](https://github.com/dfinity/agent-js) ライブラリを使用します。

これはスタンドアローンのプロジェクトで、自分のプロジェクトにコピーすることができます。

## 使用方法

以下のコマンドでレプリカを起動し、開発用 Internet Identity Canister をインストールし、テストスイートを実行します：

```bash
# dfinity/internet-identity を確認後、`./demos/using-dev-build` で以下を実行します：
$ dfx start --background --clean
$ npm ci
$ dfx deploy --no-wallet --argument '(null)'
```

この時点で、レプリカ（現実的には Internet Computer のローカル版）が稼働し、3つの Canister が配備されたことになります：

- `internet_identity`: Internet Identity の開発版（[最新リリース](https://github.com/dfinity/internet-identity/releases/latest) からダウンロードできます。[`dfx.json`](https://github.com/dfinity/internet-identity/blob/main/demos/using-dev-build/dfx.json) を参照してください。）
- `webapp`: この小型のWeb アプリは、Identity（アンカー）の作成と認証のために `internet_identity` Canister を呼び出し、次に `whoami` Canister（下記参照）を呼び出して Identity が有効であることを表示します。この Web アプリのソースは [`index.html`](https://github.com/dfinity/internet-identity/blob/main/demos/using-dev-build/webapp/index.html) と [`index.js`](https://github.com/dfinity/internet-identity/blob/main/demos/using-dev-build/webapp/index.js) で見ることができます。
- `whoami`: 呼び出しが認証されているかどうかをチェックし、"呼び出し元の Principal” を返すシンプルな Canister です。実装は非常にシンプルです：
  ```motoko
  actor {
      public query ({caller}) func whoami() : async Principal {
          return caller;
      };
  };
  ```
IC では Principal は、リクエストを実行する人、またはコールをする人（いわゆるコーラー）の識別子です。すべての呼び出しには有効な Principal が必要です。また、匿名呼び出し用の特別な Principal があります。Internet Identity を使用する場合、[self-authenticating principals](../../../references/ic-interface-spec.md#principals) を使用しています。これは、ラップトップに隠れた秘密鍵があり（TouchID、Windows Hello など）、ブラウザが署名して IC への呼び出しを発行する人物であることを証明するために使用する非常に凝った方法です。

IC が実際に `whoami` Canister に通話（リクエスト）を通した場合、それはすべてがチェックアウトされたことを意味し、`whoami` Canister は IC がリクエストに追加する情報、すなわち、あなたの識別子（Principal）をレスポンスするのみです。

### Auth-Client ライブラリを使用した Internet Identity でのログイン

DFINITY では、Internet Identity でログインするための[使いやすいライブラリ（agent-js）](https://github.com/dfinity/agent-js) を提供しています。

ログインし、Identity を使って Canister コールに使用するために必要な手順です：
```js
// まず、AuthClient を作成する必要があります。
const authClient = await AuthClient.create();

// authClient.login(…) を呼び出して、Internet Identity でログインします。これでログインプロンプトで新しいタブが開きます。
// コードはログイン処理が完了するのを待つ必要があります。
// コールバック関数を直接使うか、プロミスでラップするかのどちらかです。
await new Promise((resolve, reject) => {
  authClient.login({
    onSuccess: resolve,
    onError: reject,
  });
});
```
ユーザーが Internet Identity で認証されると、Identity にアクセスできるようになります：
```js
// 認証クライアントから Identity を取得します。
const identity = authClient.getIdentity();
// 認証クライアントから取得した Identity を利用して、IC と対話するためのエージェントを作成することができます。
const agent = new HttpAgent({ identity });
// Web アプリのインターフェイス記述を使って、サービスメソッドの呼び出しに使う Actor を作成します。
const webapp = Actor.createActor(webapp_idl, {
  agent,
  canisterId: webapp_id,
});
// 現在のユーザーの principal（ユーザー Identity）を返す whoami を呼び出します。
const principal = await webapp.whoami();
```
完全な動作例については [`index.js`](https://github.com/dfinity/internet-identity/blob/main/demos/using-dev-build/webapp/index.js) を参照してください。
舞台裏で起こっていることの詳しい説明は、[client auth protocol specification](https://github.com/dfinity/internet-identity/blob/main/docs/internet-identity-spec.adoc#client-auth-protocol) にあります。

### Canister ID の取得

では、その Canister を使ってみましょう。詳細を気にしないなら [ヘルパー](#ヘルパー)まで読み飛ばしてください。

これらの Canister と対話するためには（例えばブラウザで Web アプリを表示するためには）、それぞれの Canister ID を把握して、 `https://localhost:8000/?canisterId=<canister ID>` という形式の URL を使う必要があります（ここで `8000` は `dfx` がレプリカへのプロキシコールに用いるポートです。 このポートは通常 `dfx.json` で指定されています）。Canister ID は `dfx コマンド` の出力で確認することができます。また、 `dfx` の "internal" (read: non-documented) ステートで確認することができます。

```
~/internet-identity/demos/using-dev-build$ cat .dfx/local/canister_ids.json
{
  "__Candid_UI": {
    "local": "r7inp-6aaaa-aaaaa-aaabq-cai"
  },
  "internet_identity": {
    "local": "rwlgt-iiaaa-aaaaa-aaaaa-cai"
  },
  "webapp": {
    "local": "rrkah-fqaaa-aaaaa-aaaaq-cai"
  },
  "whoami": {
    "local": "ryjl3-tyaaa-aaaaa-aaaba-cai"
}
```

異なる Canister ID が表示されるかもしれません(それはそれで全く問題ありません)。`webapp` の Canister ID が `rrkah-fqaaa-aaaaq-cai` であれば、ブラウザで [`http://localhost:8000/?canisterId=rrkah-fqaaa-aaaaa-aaaaq-cai`](http://localhost:8000/?canisterId=rrkah-fqaaa-aaaaa-aaaaq-cai) を指定すれば Web アプリケーションを見ることができるはずです。出来ましたか？

![](../_attachments/webapp.png)

_実際に Web アプリを使用する場合は、"Internet Identity URL” フィールドが `http://localhost:8000/?canisterId=<canister ID of internet_identity canister>` を指していることを確認してください_

## ヘルパー

Canister ID を計算し、`canisterId=...` クエリ・パラメータを使用することは、すべて少し面倒なことです。以下はお勧めのコマンドです：

- `npm run start`: アプリをビルドして `localhost:8080` で提供し、コード変更時にホットリロードします。
- `npm run proxy`: Internet Identity を `localhost:8086` に、Web アプリを `localhost:8087` に配信するプロキシを起動し、簡単にアクセスできるようにします。
- `npm run test`: プロキシを起動し、 `internet_identity` Canister に対してブラウザテストを実行します。

詳しくは、[`dfx.json`](https://github.com/dfinity/internet-identity/blob/main/demos/using-dev-build/dfx.json) ファイル、[Genesis talk on Internet Identity](https://youtu.be/oxEr8UzGeBo) および [SDK documentation](../../build/) をご確認ください。まだ飽きてないですか？[Internet Computer Specification](../../../references/ic-interface-spec.md) と [Internet Identity Specification](../../../references/ii-spec.md) をチェックしてみてください。


<!--
# Internet Identity Integration

This shows how to integrate and test a project with Internet Identity. This uses the development [build flavor](https://github.com/dfinity/internet-identity/blob/main/README.md#build-features-and-flavors) of Internet Identity and the [agent-js](https://github.com/dfinity/agent-js) library.

This is a standalone project that you can copy to your own project.

## Usage

The following commands will start a replica, install the development Internet Identity canister, and run the test suite:

```bash
# After checking out dfinity/internet-identity, run this in `./demos/using-dev-build`:
$ dfx start --background --clean
$ npm ci
$ dfx deploy --no-wallet --argument '(null)'
```

At this point, the replica (for all practical matters, a local version of the Internet Computer) is running and three canisters have been deployed:

- `internet_identity`: The development version of Internet Identity (downloaded from the [latest release](https://github.com/dfinity/internet-identity/releases/latest), see [`dfx.json`](https://github.com/dfinity/internet-identity/blob/main/demos/using-dev-build/dfx.json)  .
- `webapp`: A tiny webapp that calls out to the `internet_identity` canister for identity (anchor) creation and authentication, and that then calls the `whoami` canister (see below) to show that the identity is valid. You'll find the source of the webapp in [`index.html`](https://github.com/dfinity/internet-identity/blob/main/demos/using-dev-build/webapp/index.html) and [`index.js`](https://github.com/dfinity/internet-identity/blob/main/demos/using-dev-build/webapp/index.js).
- `whoami`: A simple canister that checks that calls are authenticated, and that returns the "principal of the caller". The implementation is terribly simple:
  ```motoko
  actor {
      public query ({caller}) func whoami() : async Principal {
          return caller;
      };
  };
  ```
  On the IC, a principal is the identifier of someone performing a request or "call" (hence "caller"). Every call must have a valid principal. There is also a special principal for anonymous calls. When using Internet Identity you are using [self-authenticating principals](../../../references/ic-interface-spec.md#principals), which is a very fancy way of saying that you have a private key on your laptop (hidden behind TouchID, Windows Hello, etc) that your browser uses to sign and prove that you are indeed the person issuing the calls to the IC.

If the IC actually lets the call (request) through to the `whoami` canister, it means that everything checked out, and the `whoami` canister just responds with the information the IC adds to requests, namely your identity (principal).

### Using the Auth-Client Library To Log In With Internet Identity

DFINITY provides an [easy-to-use library (agent-js)](https://github.com/dfinity/agent-js) to log in with Internet Identity. 

These are the steps required to log in and use the obtained identity for canister calls:
```js
// First we have to create and AuthClient.
const authClient = await AuthClient.create();

// Call authClient.login(...) to login with Internet Identity. This will open a new tab
// with the login prompt. The code has to wait for the login process to complete.
// We can either use the callback functions directly or wrap in a promise.
await new Promise((resolve, reject) => {
  authClient.login({
    onSuccess: resolve,
    onError: reject,
  });
});
```
Once the user has been authenticated with Internet Identity we have access to the identity:
```js
// Get the identity from the auth client:
const identity = authClient.getIdentity();
// Using the identity obtained from the auth client, we can create an agent to interact with the IC.
const agent = new HttpAgent({ identity });
// Using the interface description of our webapp, we create an Actor that we use to call the service methods.
const webapp = Actor.createActor(webapp_idl, {
  agent,
  canisterId: webapp_id,
});
// Call whoami which returns the principal (user id) of the current user.
const principal = await webapp.whoami();
```
See [`index.js`](https://github.com/dfinity/internet-identity/blob/main/demos/using-dev-build/webapp/index.js) for the full working example.
A detailed description of what happens behind the scenes is available in the [client auth protocol specification](https://github.com/dfinity/internet-identity/blob/main/docs/internet-identity-spec.adoc#client-auth-protocol).

### Getting the Canister IDs

Let's now use those canisters. Don't care about details? Skip to the [helpers](#helpers).

In order to talk to those canisters (for instance to view the webapp in your browser) you need to figure the ID of each canister and then use an URL of the form `https://localhost:8000/?canisterId=<canister ID>` (where `8000` is the port used by `dfx` to proxy calls to the replica; that port is usually specified in the `dfx.json`). You can find the canister IDs in the output of the `dfx command`, or by checking `dfx`'s "internal" (read: non-documented) state:

```
~/internet-identity/demos/using-dev-build$ cat .dfx/local/canister_ids.json
{
  "__Candid_UI": {
    "local": "r7inp-6aaaa-aaaaa-aaabq-cai"
  },
  "internet_identity": {
    "local": "rwlgt-iiaaa-aaaaa-aaaaa-cai"
  },
  "webapp": {
    "local": "rrkah-fqaaa-aaaaa-aaaaq-cai"
  },
  "whoami": {
    "local": "ryjl3-tyaaa-aaaaa-aaaba-cai"
}
```

You might get different canister IDs (and that's totally fine). If the `webapp` canister ID is `rrkah-fqaaa-aaaaa-aaaaq-cai`, you should be able to point your browser to [`http://localhost:8000/?canisterId=rrkah-fqaaa-aaaaa-aaaaq-cai`](http://localhost:8000/?canisterId=rrkah-fqaaa-aaaaa-aaaaq-cai) to see the webapp. Hurray!

![](../_attachments/webapp.png)

_If you actually use the webapp, make sure that the "Internet Identity URL" field points to `http://localhost:8000/?canisterId=<canister ID of the internet_identity canister>`._

## Helpers

Figuring the canister IDs, and using the `canisterId=...` query parameter is all a bit cumbersome. Here are some commands you might like:

- `npm run start`: Build the app and serve it on `localhost:8080` with hot reload on code changes, ideal for hacking on the webapp.
- `npm run proxy`: Start a proxy that serves Internet Identity on `localhost:8086` and the webapp on `localhost:8087` for easy access.
- `npm run test`: Start the proxy and run browser tests against the `internet_identity` canister.

For more information, check the [`dfx.json`](https://github.com/dfinity/internet-identity/blob/main/demos/using-dev-build/dfx.json) file, the [Genesis talk on Internet Identity](https://youtu.be/oxEr8UzGeBo) and the [SDK documentation](../../build/). Not bored yet? Check out the [Internet Computer Specification](../../../references/ic-interface-spec.md) and the [Internet Identity Specification](../../../references/ii-spec.md).

-->