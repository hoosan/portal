# チュートリアル

[クイックスタート](../../quickstart/hello10mins.md)では、プロジェクトディレクトリの内容やサンプルコードを探索せずに、新しいプロジェクトを作成しデプロイするための基本作業の流れを簡略化して紹介しました。

次に、IC 上で動作するプログラムの作成を実際に体験してもらうために、いくつかの簡単なプログラムの書き方を探ります。

このチュートリアルの多くは、プログラミング言語 Motoko を使った Dapp の書き方を説明しています。他の言語によるアプリの例については、DFINITY [examples](https://github.com/dfinity/examples) リポジトリを参照してください。

以下のチュートリアルでは、IC 上で動作する Dapp を作成するための基本的な方法を紹介します：

-   [デフォルトプロジェクトを探索する](./explore-templates.md)では、新しいプロジェクトを作成する際にワークスペースに追加されるデフォルトのファイルとフォルダーを探索し、プロジェクト作成のワークフローを詳しく見ていきます。

-   [Actor を使ったクエリ](./define-an-actor.md) は、通常 "Hello, World!” Canister で定義される典型的な `print` 関数を、 `hello` 関数を持つ Actor（オブジェクト）の定義によって置き換える方法を紹介します。

-   [テキスト引数を渡す](./hello-location.md)では、ターミナルシェルでコマンドラインを使用して関数に引数を渡すさまざまな方法を紹介しています。

-   [自然数をインクリメントする](./counter-tutorial.md)は、カウンタの値をインクリメントして返す関数を持つ Actor を作成する Dapp の書き方を説明します。

-   [電卓の機能で整数を使う](./calculator.md) では、簡単な電卓アプリの書き方を紹介します。Motoko を使ってより実践的に作業し、プロジェクト環境をカスタマイズする方法について学びましょう。

-   [Import library modules](./phonebook.md) は、リスト内のキーと値のペアを扱うための、いくつかの基本的な Motoko ベースライブラリ関数をインポートして使用する方法を説明しています。

-   [Use multiple actors](./multiple-actors.md) は、1つのプロジェクトに複数の無関係な Actor を含める方法を説明し、同じプロジェクトに複数の Canister をコンパイルできることを説明しています。

-   [フロントエンドのカスタマイズ](../frontend/custom-frontend.md)では、シンプルな React フレームワークを使用して、デフォルトのサンプル Canister 用の新しいフロントエンドを作成し、表示されるインターフェイスをカスタマイズするためのいくつかの基本的な修正方法を説明します。CSS、HTML、JavaScript、React または他のフレームワークを使ってユーザー・インターフェースを構築する方法を既に知っている場合は、このチュートリアルを読み飛ばすことができます。

-   [Add a stylesheet](../frontend/my-contacts.md) は、React を使用してプロジェクトの新しいフロントエンドを作成するときに、スタイルシートを追加する方法を説明します。React にスタイルシートを追加する方法をすでに知っている場合は、このチュートリアルを読み飛ばすことができます。

-   [Make inter-canister calls](./intercanister-calls.md) は、ある Canister で定義された関数を、同じプロジェクト内の別の Canister から呼び出す簡単な方法を説明しています。

-   [スケーラブルなアプリケーションの作成](./scalability-cancan.md)では、複数の Canister を使用して拡張性のあるアプリケーションを作成することについて説明しています。

-   [Identity によるアクセスコントロールの追加](./access-control.md)では、複数のユーザーIdentity を作成し、切り替えて使用する方法を説明しています。
-   [Accept cycles from a wallet](./simple-cycles.md) は、デフォルトのウォレット Canister から送信された Cycle を受け入れる方法を示しています。

より高度なアプリを紹介するチュートリアルや、基本的な構成要素の詳しい使い方は、[examples](https://github.com/dfinity/examples) リポジトリや[*Motoko Programming Language Guide*](../cdks/motoko-dfinity/about-this-guide.md) で紹介されていますので、参考にしてください。

<!--
# Tutorials

The [Quick start](../../quickstart/hello10mins.md) provided a simplified introduction to the basic work flow for creating and deploying a new project without exploring the contents of the project directory or sample code.

Next, we’ll explore writing a few simple programs to give you hands-on experience creating programs that run on the IC.

Most of these tutorials illustrate how to write dapps using the Motoko programming language. For additional examples of dapps written in other languages, see the DFINITY [examples](https://github.com/dfinity/examples) repository.

The following tutorials introduce the basics for writing dapps that run on the IC:

-   [Explore the default project](./explore-templates.md) takes a closer look at the work flow for creating projects by exploring the default files and folders that are added to your workspace when you create a new project.

-   [Query using an actor](./define-an-actor.md) highlights how to replace the typical `print` function usually defined in a "Hello, World!" canister by defining an actor (object) with a `hello` function.

-   [Pass text arguments](./hello-location.md) introduces different ways you can pass arguments to a function using the command-line in a terminal shell.

-   [Increment a natural number](./counter-tutorial.md) guides you through the process of writing a dapp that creates an actor with functions to increment and return the value of a counter.

-   [Use integers in calculator functions](./calculator.md) shows you how to write a simple calculator dapp for more practice working with Motoko and to learn more about how you can customize your project environment.

-   [Import library modules](./phonebook.md) illustrates how to import and use a few basic Motoko base library functions for working with key-value pairs in a list.

-   [Use multiple actors](./multiple-actors.md) describes how to include multiple unrelated actors in a single project to illustrate how you can compile multiple canisters for the same project.

-   [Customize the frontend](../frontend/custom-frontend.md) illustrates using a simple React framework to create a new frontend for the default sample canister and guides you through some basic modifications to customize the interface displayed. If you already know how to use CSS, HTML, JavaScript, and React or other frameworks to build your user interface, you can skip this tutorial.

-   [Add a stylesheet](../frontend/my-contacts.md) illustrates how to add a stylesheet when you use React to create a new frontend for your project. If you already know how to add stylesheets to React, you can skip this tutorial.

-   [Make inter-canister calls](./intercanister-calls.md) illustrates how to make simple calls to functions defined in one canister from another canister in the same project.

-   [Create scalable apps](./scalability-cancan.md) describes using multiple canisters to create applications that scale.

-   [Add access control with identities](./access-control.md) describes how to create and switch between multiple user identities.

-   [Accept cycles from a wallet](./simple-cycles.md) illustrates how to accept cycles sent from the default wallet canister.

Additional tutorials covering more advanced dapps and more detailed examples of how to use the basic building blocks are available in the [examples](https://github.com/dfinity/examples) repository and [*Motoko Programming Language Guide*](../cdks/motoko-dfinity/about-this-guide.md).

-->