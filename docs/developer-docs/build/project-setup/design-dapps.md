# Dapp のデザイン

アプリのアイデアを考えるとき、どのような構成にするかなど、さまざまな設計上の判断を迫られることになります。Internet Computer では、アプリの実装を考える上で、特に設定上で注意を払うべき判断がいくつかあります。

## シングルまたはマルチ Canister アーキテクチャー

Dapp を設計する際に考慮すべき最初の決定の1つは、単一の Canister にカプセル化するか、複数の Canister で構成するかです。

例えば、フロントエンドのないシンプルなサービスを書いている場合、単一の Canister を使用して、プロジェクト管理とメンテナンスを簡素化し、機能の追加に集中するのがよいでしょう。フロントエンドのアセットとバックエンドのビジネスロジックの両方を持つ Dapp の場合、プロジェクトは少なくともふたつの Canister で構成されることになります。ひとつの Canister はユーザーインターフェース・コンポーネントを管理し、もうひとつは Canister はアプリケーションが提供するバックエンド・サービスに対応します。

計画にあたっては、再利用可能な共通サービスを独自の Canister に配置し、より専門的な他の Canister からインポートして呼び出したり、他の開発者が使用できるようにすることも検討するとよいでしょう。[LinkedUp](https://github.com/dfinity/linkedup) サンプル Dapp は、プロフェッショナルサービスの Dapp をふたつの Canister に分割して、このアプローチを説明しています。LinkedUp の例では、ソーシャルコネクションを確立する機能は `connectd`  Canister で、プロフェッショナルプロファイルをセットアップする機能は `linkedup` Canister で定義されていて分離されています。例えば、プロファイルの属性や共有されたコネクションに基づいてイベントをスケジュールするために、3つ目の Canister でアプリケーションを拡張することを想像することは容易です。

## 型・ユーティリティから Actor を分離する

プロジェクトのアーキテクチャーを計画する際に、一般的なプラクティスとして、メインの Actor 用のコードをひとつのファイルに配置し、プログラムで使用する型や Actor を必要としないユーティリティ関数を定義するためのファイルを別途追加するというものがあります。

例えば、アプリのバックエンドロジックを以下のファイルで構成するように設定することができます：

-   `Main.mo` または `main.rs` は Actor がクエリやアップデートの呼び出しを送信するために必要な関数を表現します。

-   `Util.mo` または `util.rs` は Actor が使用できるようにインポートできるヘルパー関数と組み合わせて使用します。

-   `Types.mo` または `types.rs` は、Dapp 用のすべてのデータ型を定義します。

## クエリコールの利用

[クエリとアップデートメソッド](../../../concepts/canisters-code#query-update) で述べたように、クエリはアップデートコールより速く結果を返します。したがって、関数を明示的に `query` としてマークすることは、アプリケーションのパフォーマンスを向上させるための効果的な戦略です。計画や設計の段階では、クエリや更新を実行できる関数ではなく、クエリコールを使用するのが最良の方法であることを（最大限）考慮する必要があります。

これは一般的なルールであり、ほとんどの種類の Dapps に適用できます。ただし、クエリがコンセンサスを得られず、ブロックチェーンに表示されないというセキュリティとパフォーマンスのトレードオフも考慮する必要があります。アプリケーションによっては、そのトレードオフが適切な場合もあります。例えば、ブログプラットフォームを開発している場合、タグに一致する記事を取得するクエリは、おそらく大多数のノードが結果に同意していることを保証するコンセンサスを経る必要はないでしょう。しかし、金融データのような機密情報を取得するアプリの場合は、基本的なクエリよりも確実な結果を求めるかもしれません。

ベーシッククエリの代わりに、Internet Computer は**認証済みクエリ（certified queries）**もサポートしています。認証済みクエリを使用すると、エンドユーザーが信頼できる **認証済みレスポンス** を受け取ることができます。認証済みクエリの使用は高度な技術であり、チュートリアルやその他の開発者向けドキュメントでは扱っていませんが、[Interface specification](../../../references/ic-interface-spec.md) で、認証の仕組みと、クエリにレスポンスして認証済みデータを返すためのプログラム構成について学ぶことができます。

## データの保存と検索

Internet Computer では、直交型持続性と呼ばれる長期間のデータ保存に**ステーブルメモリ**を使用し、データの取得に**クエリコール**を使用することができます。1つ以上のキーを使って効率的にデータを取り出すには、通常、ハッシュテーブルのようなデータ構造を使用することで実現できます。また、より伝統的なデータベースを Canister 内に実装することも可能です。

<!--
# Designing Dapps

As you come up with ideas for dapps, you are going to make many design decisions about how to structure and organize your project. On the Internet Computer, there are a few design decisions that you should pay particular attention to as you plan the implementation for your app.

## Single or multiple canister architecture

One of the first decisions you might want to consider when designing your dapp is whether it should be encapsulated in a single canister or consist of multiple canisters.

For example, if you are writing a simple service with no frontend, you might want to use a single canister to simplify project management and maintenance and focus on adding features. If your dapp has both frontend assets and backend business logic, your project is likely to consist of at least two canisters, with one canister for managing user interface components and another canister for the backend services the application provides.

In planning, you might also consider placing some common reusable services in their own canister so that they can be imported and called from other more-specialized canisters or made available for other developers to use. The [LinkedUp](https://github.com/dfinity/linkedup) sample dapp illustrates this approach by splitting the professional service dapp into two canisters. In the LinkedUp example, the functions that establish social connections are defined in the `connectd` canister and separate from the functions used to set up professional profiles that are defined in the `linkedup` canister. It is easy to imagine extending the dapp with a third canister, for example to schedule events based on profile attributes or shared connections.

## Segregating actors from types and utilities

In planning the architecture for your project, one common practice is to place the code for the main actor in one file with separate additional files for defining the types you program uses and utility functions that don’t require an actor.

For example, you might set up the backend logic for your dapp to consist of the following files:

-   `Main.mo` or `main.rs` with the functions that require an actor to send query and update calls.

-   `Util.mo` or `util.rs` with helper functions that can be imported for the actor to use.

-   `Types.mo` or `types.rs` with all of the data type definitions for your dapp.

## Using query calls

As discussed in [Query and update methods](../../../concepts/canisters-code#query-update), queries return results faster than update calls. Therefore, explicitly marking a function as a `query` is an effective strategy for improving application performance. In the planning and design phase, you should consider how best to use query calls instead of functions that can perform queries or updates.

That is a good general rule to follow and can be applied broadly to most categories of dapps. However, you should also consider the security and performance trade-off that queries don’t go through consensus and do not appear on the blockchain. For some dapps, that trade-off might be appropriate. For example, if you are developing a blogging platform, queries that retrieve articles matching a tag probably don’t warrant going through consensus to ensure that a majority of nodes agree on the results. However, if your dapp is retrieving sensitive information—like financial data—you might want more assurance about your results than a basic query provides.

As an alternative to basic queries, the Internet Computer also supports **certified queries**. Certified queries enable you to receive **authenticated responses** that end users can trust. Using certified queries is an advanced technique that is not covered in the tutorials or other developer-focused documentation, but you can learn about how the authentication works and what you need to do to configure your program to return certified data in response to queries in the [Interface specification](../../../references/ic-interface-spec.md).

## Data storage and retrieval

The Internet Computer enables you to use **stable memory** to handle long-term data storage—often referred to as orthogonal persistence—and to use **query calls** to retrieve your data. Efficiently retrieving data using one or more keys can typically be achieved by using data structures like hash tables. It is also possible to implement a more traditional database inside a canister.

-->