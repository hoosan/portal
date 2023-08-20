# スケーラブルなアプリの作成

CanCan サンプルアプリケーションは、動画共有サービスを簡略化したもので、独自のアプリケーションのモデルとして使用できる機能をいくつか紹介しています。例えば、CanCan のサンプルアプリケーションを利用することで、以下のようなことが学べます：

-   コンテンツをフラグメントに分割してアップロード・保存し、クエリを使用してフラグメントを取得・再集合して効率的なストリーミングを行うことで、スケーラブルなアプリケーションを構築する方法。

-   異なるバックエンド言語で記述された Canister を使用するアプリケーションの相互運用性を設定する方法。

-   異なるユーザーによってアップロードされたビデオを保存するための基本的な認証モデルを実装する方法。

-   デスクトップまたはモバイルアプリケーション向けに、より洗練されたユーザーインターフェース機能を実装したフロントエンドを構築する方法。

## アップロードされたコンテンツを複数の Canister に分割する

Canister はコンパイルされた WebAssembly モジュールであるため、一定の既知の制限があります。例えば、現在の WebAssembly モジュールでは、メモリに最大 4GB、オブジェクトの呼び出し回数に上限があります。

CanCan のような動画共有のサンプルアプリケーションでは、これらの制限があるため、複数の Canister が必要となり、データを小さなチャンクに分割して保存、取得する必要があります。

### 分散ハッシュテーブルを実装する

Internet Computer 向けにスケーラブルな動画共有サービスを構築する最初の試みでは、バックエンドサービスとして分散ハッシュ テーブル（DHT）を使用し、アップロードまたはストリーミングする動画のデータチャンクを事前に定義された Canister セットに分散する単純な get および put 関数を使用しました。プロジェクトの初期段階では、コンセプトの実証と、保存と検索のためにビデオデータを適切にトランスコードできることを確認するにはこのアプローチで十分でした。

しかし、分散ハッシュテーブルは、保存のためにデータを投入し、視聴のためにデータを取得できる特定の数の Canister に依存していたため、アプリケーションのスケーラビリティには限界がありました。さらに、分散ハッシュテーブルのバックエンドサービスの最初の実装には、ノードが利用できなかったりデータを失う原因となり得る一般的なネットワーク接続の問題に対応するためのコードが含まれていました。

### スケーラビリティを簡略化する

Internet Computer プロトコルは、サブネット内のノード間で複製されたステートに依存しているため、他のプラットフォームやプロトコル上で動作するアプリケーションでは一般的に利用できない、フォールトトレランスとフェールオーバーに関する特定の保証をネイティブに提供します。

Internet Computer は、Canister がリクエストを受信し応答を保証することを認識することで、オリジナルの分散ハッシュテーブルのバックエンドサービスは、よりシンプルで拡張性の高い BigMap というバックエンドサービスに置き換わりました。

BigMap は、Internet Computer 上で Key-Value ストレージを利用したスケーラブルなアプリケーションを構築するための、シンプルなプラグインライブラリです。BigMapライブラリをバックエンドサービスとして使用することで、CanCan サンプルアプリケーションは、動的にデータをチャンクし、シリアライズし、複数の Canister に配布することができます。

このライブラリは、アプリケーション固有のインメモリデータを抽象化するためのビルディングブロックを提供し、任意の数のキャニスタを使用して拡張することができます。各 Canister の容量はまだ限られていますが、アプリケーションは必要な Canister をインスタンス化し、各ユーザーのビデオコンテンツを完全に構成するフラグメントを `manifest` というインデックスファイルに記録しておきます。

`BigMap` サービスに必要なコードは、従来の分散ハッシュテーブルよりもはるかにシンプルです。なぜなら、プラットフォームとしての Internet Computer が、スケーラビリティ、レプリケーション、フェイルオーバー、フォールトトレランスを提供してくれるからです。

## 相互運用性を実証する

CanCan サンプルアプリケーションのリポジトリに含まれる BigMap サービスは、プログラミング言語 Rust で書かれています。しかし、CanCan サンプルアプリケーションは、異なる言語で書かれた Canister 間の相互運用性を示すものでもあります。

この場合、BigMap の機能は Rust プログラミング言語で実装され、その他のサービス（ビデオコンテンツのエンコードとデコード、認証のためのユーザープリシパルの管理など）は Motoko で実装されています。

サンプルアプリケーションの各部を Canister としてデプロイすることで、各部の相互通信がシームレスなユーザーエクスペリエンスを提供することができます。

CanCan のリポジトリでご覧いただける `BigMap` サービスは Rust で書かれていますが、実際には以下のことを実証するために、Rust と Motoko の両方のプログラミング言語で実装されています：

-   Motoko のコードと、 Canister としてデプロイされた Rust のコードの両方を Internet Computer 上で実行することができます。

-   CanCan サンプルアプリケーションの動作に影響を与えることなく、バックエンド言語の切り替えが可能です。

-   Candid 言語は、JavaScript、Rust、Motoko に依存しない BigMap API を記述するための共通言語を提供するため、どちらの言語実装もシームレスに動作します。

## 認証モデル

LinkedUp のサンプルアプリケーションと同様に、CanCan のサンプルアプリケーションでも、公開鍵と秘密鍵のペア、ブラウザベースのローカルストレージ、および `Principal` データ型を使用してユーザーを認証しています。

## フロントエンドの機能を実装する

CanCan のサンプルアプリケーションでは、React ライブラリと TypeScript を組み合わせて、フロントエンドのユーザーインターフェイスを実装しています。

## データモデル概要

このアプリケーションは、ユーザーに関する情報とビデオに関する情報を保存します。ほとんどのブラウザをサポートするために、動画はバイト配列に直列化され、動画データは [Uint8Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array) オブジェクトと呼ばれるバイトの 500kb セグメントに格納されています。動画が要求されると、マニフェストは動画の再生に必要なチャンクのリストをロードし、標準の `<video>` 要素で動画を表示する前にチャンクを連結します。

### ユーザープロファイル

ユーザプロファイルは `profiles/{username}` として保存され、以下のデータモデルで定義されます：

    export interface Profile {
      username: string;         // maya
      following: string[];      // [`alice`, `bob]`
      followers: string[];      // ['palice']
      uploadedVideos: string[]; // ['profiles/maya/videos/a5b54646-2ea3-4e0e-82d1-da3ab8148df2']
      likedVideos: string[];    // ['profiles/bob/videos/b74e4eb0-dea8-4a4a-a1ae-d4593dc86930']
      avatar?: string;          // ?ImageData (TODO)
    }

### ビデオ

アップロードされたビデオは `profiles/{username}/videos/{videoId}` と `public/videos`（プラットフォーム上のすべての既存ビデオの配列）に格納されている一意の識別子によって識別されます。

ビデオのメタデータは `profiles/{username}/videos/{videoId}/metadata` に格納されます。個々のビデオフラグメントは `profiles/{username}/videos/{videoId}/chunks/chunk.{0-10}` に格納されます。

ビデオは `profiles/{username}` に格納され、以下のデータモデルを使用して定義されます：

    export interface Video {
      src: string;       // 'profiles/maya/videos/f5a44646-2ea3-4e0f-83d2-da3ab8148df2'
      userId: string;    // 'maya'
      createdAt: string; // Date.now()
      caption: string;   // 'cool movie, punk'
      tags: string[];    // ['outside', 'grilling', 'beveragino']
      likes: string[];    // ['sam', 'kelly']
      viewCount: number; // 102
      name: string; // 'grilling'
    }

<!--
# Create scalable apps

The CanCan sample application is a simplified video-sharing service that demonstrates several features that you can use as models for your own applications. For example, here are a few things you can learn by exploring the CanCan sample application:

-   How to build a scalable application by splitting content into fragments for upload and storage then using queries to retrieve and reassemble the fragments for efficient streaming.

-   How to configure interoperability for an application that uses canisters written in different backend languages.

-   How to implement a basic authentication model for storing videos uploaded by different users.

-   How to build a frontend that implements more sophisticated user interface features for desktop or mobile apps.

## Splitting uploaded content into multiple canisters

Because canisters are compiled WebAssembly modules, they have certain known limitations. For example, currently WebAssembly modules have a maximum of 4GB for memory and an upper limit on the number of object calls allowed.

For a video-sharing sample applications like CanCan, these limitations mean that multiple canisters are required and that data must be broken into smaller chunks for storage and retrieval.

### Implementing a distributed hash table

The initial attempt to build a scalable video-sharing service for the Internet Computer used a distributed hash table (DHT) as a backend service with simple get and put functions that distributed data—chunks of the video to be uploaded or streamed—into a predefined set of canisters. In the early phases of the project, this approach was sufficient for a proof-of-concept and verifying that the video data could be properly transcoded for storage and retrieval.

However, the scalability of the application was limited because the distributed hash table relied on a specific number of canisters that it could populate with data for storage and retrieve data from for viewing. In addition, the original implementation of the distributed hash table backend service included code to accommodate common network connectivity issues that could cause nodes to be unavailable or lose data.

### Simplifying scalability

Because the Internet Computer protocol relies on replicated state across nodes in a subnet, it provides certain guarantees about fault tolerance and failover natively that are not generally available to applications running on other platforms or protocols.

With the realization that the Internet Computer could ensure that canisters were available to receive and respond to requests,the original distributed hash table backend service was replaced with a simpler but more scalable backend service called BigMap.

BigMap provides a simple, plug-in library for building scalable applications using key-value storage on the Internet Computer. By using the BigMap library as a backend service, the CanCan sample application can dynamically chunk, serialize, and distribute data to multiple canisters.

The library offers building blocks for application-specific, in-memory data abstractions that scale using any number of canisters. Each canister still has limited capacity, but the application instantiates the canisters it needs and keeps track of the fragments that make up the full video content for each user’s videos in an index file called the `manifest`.

The code required for the `BigMap` service is much simpler than a traditional distributed hash table because the Internet Computer as a platform provides scalability, replication, failover, and fault tolerance.

## Demonstrating interoperability

The BigMap service included in the CanCan sample application repository is written in the Rust programming language. However, the CanCan sample application also demonstrates interoperability between canisters written in different languages.

In this case, the `BigMap` functionality is implemented using the Rust programming language and other services—such as encoding and decoding of video content and the management of user principals for authentication—are implemented using Motoko.

By deploying different parts of the sample application as canisters, the interaction between them provides a seamless user experience.

Although the `BigMap` service you see in the CanCan repository is written in Rust, the service was actually implemented in both Rust and Motoko programming languages to demonstrate the following:

-   You can run both Motoko code and Rust code deployed as canisters on the Internet Computer.

-   You can switch between backend languages without affecting the operation of the CanCan sample application.

-   Both language implementation work seamlessly because the Candid language provides a common language for describing the BigMap API, independent of JavaScript, Rust, or Motoko.

## Authentication model

Much like the LinkedUp sample application, the CanCan sample application uses the public-private key pair, browser-based local storage, and the `Principal` data type to authenticate users.

## Implementing frontend features

The CanCan sample application uses the React library in combination with TypeScript to implement frontend user interface.

## Data model overview

The application stores information about users and information about videos. To support most browsers, the videos are serialized into byte arrays with video data stored in 500kb segments of bytes that are referred to as [Uint8Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array) objects. When a video is requested, the manifest loads the list of chunks required to play the video and concatenates the chunks before displaying the video in a standard `<video>` element.

### User profiles

User profiles are stored as `profiles/{username}` and are defined using the following data model:

    export interface Profile {
      username: string;         // maya
      following: string[];      // [`alice`, `bob]`
      followers: string[];      // ['palice']
      uploadedVideos: string[]; // ['profiles/maya/videos/a5b54646-2ea3-4e0e-82d1-da3ab8148df2']
      likedVideos: string[];    // ['profiles/bob/videos/b74e4eb0-dea8-4a4a-a1ae-d4593dc86930']
      avatar?: string;          // ?ImageData (TODO)
    }

### Videos

Uploaded videos are identified by a unique identifier stored at `profiles/{username}/videos/{videoId}` and in `public/videos` (an array of all existing videos on the platform).

Metadata for videos is stored at `profiles/{username}/videos/{videoId}/metadata` Individual video fragments are stored at `profiles/{username}/videos/{videoId}/chunks/chunk.{0-10}`.

Videos are stored as `profiles/{username}` and are defined using the following data model:

    export interface Video {
      src: string;       // 'profiles/maya/videos/f5a44646-2ea3-4e0f-83d2-da3ab8148df2'
      userId: string;    // 'maya'
      createdAt: string; // Date.now()
      caption: string;   // 'cool movie, punk'
      tags: string[];    // ['outside', 'grilling', 'beveragino']
      likes: string[];    // ['sam', 'kelly']
      viewCount: number; // 102
      name: string; // 'grilling'
    }

-->