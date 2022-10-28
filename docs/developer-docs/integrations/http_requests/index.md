# Canister HTTPS アウトコール

これまでブロックチェーンは孤立した存在であり、スマートコントラクトが外部のサーバーや他のブロックチェーンと直接通信することはできませんでした。その理由は、ブロックチェーンは複製されたステートマシンであり、各ノードはラウンドごとに同じステートに対して同じ計算を行い、同じ遷移を行う必要があるからです。外部サービスの結果を入力として計算を行うことは、ナイーブに行うとノードのステート不一致につながりやすいため、動作させるためには技術的な配慮が必要です。

Internet Computer の _HTTP(S) リクエストの登録_または _HTTP(S) アウトコール_ の機能により、ブロックチェーン史上初めて、スマートコントラクトがブロックチェーンの外部の HTTP(S) サーバーを直接呼び出し、レスポンスをスマートコントラクトの処理で使用し、その入力を使って安全に複製ステートを更新することができるようになりました。これまでのところ、スマートコントラクトと外部サーバーとの通信手段は、いわゆる*オラクル*を介したものだけでした。このドキュメントの残りの部分では、基礎となるプロトコルを参照する *HTTP* と *HTTPS* を *HTTP* に統一して使用することに注意してください。近年、公共ネットワーク上の実質的にすべての HTTP トラフィックは、保護された HTTPS 上で実行されます。

Canister HTTP リクエストは、多くのユースケースを可能にし、現在使用されているオラクルモデルと比較して多くの利点を持っています。
* *より強力なトラストモデル：* Canister HTTP アウトコールは、スマートコントラクトが外部サーバーと通信するために外部仲介者（オラクル）が必要ないため、より強力なトラストモデルに基づいています。
* *低料金：* サービスに対して追加料金を請求する仲介者が存在しないため低料金です。
* *標準的なプログラミングパラダイムに近い：* スマートコントラクトが直接外部サーバーに HTTP リクエストを行うというパラダイムは、オラクルの使用と比較すると、エンジニアが慣れている通常のプログラミングパラダイムに非常に近いと言えます。そのため、ブロックチェーンのためのプログラミングするということをより抽象化することができます。

なぜブロックチェーンでは外部との連携が重要なのか？
* 現実の Dapp のユースケースのほとんどは、オフチェーンのエンティティとの何らかの形のデータ交換を必要とします。
* 世界のデータのほとんどは、現在伝統的な（Web 2.0）サービスに保持されており、多くの Dapps はこのデータに基づいて構築されているため、それにアクセスする必要があります。
* *ブロックチェーン・シンギュラリティ* に到達するためには、スマートコントラクトは Web 2.0 サービスと相互作用できる必要があります。ブロックチェーン・シンギュラリティへの過程で、ますます多くのデータが Web3.0 のブロックチェーンの世界に引き込まれ、Web2.0 サーバーを介さずに異なるスマートコントラクト間で相互作用が行われるようになるでしょう。

Canister HTTP アウトコールには多くのユースケースがあり、いくつかの顕著な例については以下のリストを参照してください。
* 最も重要なユースケースの1つは、外部 HTTP API からのデータの読み込みです。例えば、DEX で使用される価格データや分散型保険 Dapps で使用される天気データなどです。
* IoT Dapps はセンサーが相互作用する従来のサーバーからセンサーデータを取得する必要があります。将来的には、センサーと IC ブロックチェーンが直接やり取りすることも想定されます。
* チャットサービスでは、ユーザーにメッセージの着信に関するプッシュ通知を送信します。

HTTP アウトコールの大半は、Web2.0 データを読み込むための `GET` コールになると思われますが、Web2.0 サーバーにデータを書き込むための外部システムとのやり取りでは `POST` も明らかに重要な役割を担っています。

## もっと学ぶには？

HTTPS アウトコール機能がどのように動作するか、またCanister をコーディングする際にどのように使用するかについて深く知りたい場合は、[How it Works](http_requests-how-it-works.md) セクションを参照してください。

## ビルドをするには？

[Examples リポジトリ](https://github.com/dfinity/examples) には、[Rustのサンプルコード](https://github.com/dfinity/examples/tree/master/rust/exchange_rate) がありますので、これを起点に独自の Dapp を構築することができます。

<!--
# Canister HTTPS Outcalls

Up until now, blockchains have been isolated entities and smart contracts have not been able to directly communicate with external servers or other blockchains. The reason for this is that a blockchain is a replicated state machine where each replica needs to perform the same computations on the same state to make the same transitions in each round. Doing computations based on results from external services as input may easily lead to state divergence on the replicas if done in a naïve manner and thus requires some technical considerations to be workable.

The feature of _canister HTTP(S) requests_, or _HTTP(S) outcalls_, on the Internet Computer enables — for the first time in blockchain history — smart contracts to directly make calls to HTTP(S) servers external to the blockchain and use the response in the further processing of the smart contract such that the replicated state can safely be updated using those inputs. So far, the only means of communication of smart contracts with external servers has been through so-called *oracles*. Note that in the remainder of this documentation we may use *HTTP* representative for both *HTTP* and *HTTPS*, referring to the underlying protocol. Practically all HTTP traffic on public networks runs over secured HTTPS these days.

Canister HTTP requests allow for a plethora of use cases and have numerous advantages over the currently used oracle model.
* *Stronger trust model:* Canister HTTP outcalls are based on a stronger trust model because no external intermediaries (oracles) are required for smart contracts to communicate with external servers.
* *Lower fees:* No intermediaries are in place to charge additional fees for their services.
* *Closer to the standard programming paradigm*: The paradigm of a smart contract directly making HTTP requests to external servers is much closer to the &ldquo;normal&rdquo; programming paradigm engineers are used to when compared to using oracles. Thus, the fact that one programs for a blockchain can be further abstracted away.

Why is interfacing with the external world so important for a blockchain?
* Most real-world dApp use cases need some form of data exchange with off-chain entities.
* Most of the world's data is currently held in traditional (Web 2.0) services and many dApps build on this data and therefore need access to it.
* In order to be able to reach *blockchain singularity*, smart contracts need to be able to interact with Web 2.0 services. In our journey towards blockchain singularity, an ever increasing amount of data will be pulled into the blockchain world of Web 3.0 and interactions will increasingly take place between different smart contracts without involving Web 2.0 servers.

There are many use cases for canister HTTPS outcalls, see the following list for some prominent examples.
* One of the most important use cases is reading data from external HTTP APIs, e.g., pricing data used in DEXs or weather data used in decentralized insurance dApps.
* IoT dApps need to obtain sensor data from traditional servers with which the sensors interact. In the future, we may even envision direct interactions of sensors with the IC blockchain.
* Chat services sending push notifications about incoming messages to users.

We expect the majority of HTTP calls to be `GET` calls for reading Web 2.0 data, but `POST` clearly also plays an important role for the interaction with external systems in order to be able to write data to Web 2.0 servers.

## Learn more

If you want to take a deep dive into how the HTTPS outcalls feature works and how to use it when coding a canister, see the [How it Works](http_requests-how-it-works.md) section.

## Build more

In the [Examples repository](https://github.com/dfinity/examples) you can find [sample code in Rust](https://github.com/dfinity/examples/tree/master/rust/exchange_rate) which you can use as starting point for building your own dApp.

-->