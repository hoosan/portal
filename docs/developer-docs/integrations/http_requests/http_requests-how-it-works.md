# 技術概要 - Canister HTTPS アウトコール の仕組み

このページでは、Canister HTTP アウトコール（Canister HTTPS リクエスト）の仕組みと、API を利用する際の注意点について詳しく説明します。また、通常の（Web2.0）コンピューター・プログラムが HTTP コールを行う場合と比較して、いくつかの制限や違いがあること、この機能をうまく使うためにプログラマーが考慮すべき点についても言及したいと思います。この機能を使おうとするエンジニアは、このページを読んでいただき、この機能を素早く理解することをお勧めします。すぐにでもコーディングに取り組みたいというせっかちな読者は、コンセプトの部分を読み飛ばして、今すぐ[コーディングセクション](#coding-https-outcalls)にジャンプして、コーディングを始めることができます。

## テクノロジー

HTTPS アウトコール機能により、Canister は従来の Web2.0 HTTP サーバーに対して HTTP のアウトコールを行うことができます。リクエストのレスポンスは、サブネットのレプリカ間でステートが不一致する危険性がなく、安全に Canister の演算に利用することができます。

### HTTPS アウトコールが IC で処理される仕組み

Canister の HTTP アウトコール機能は、Internet Computer レプリカの一部として実装され、マネージメント Canister の API として公開されます。次に、Canister によるリクエストがどのように処理されるのか、簡略化して概要を説明します。HTTP リクエスト機能はサブネット単位で実現されており，各サブネットは自分の Canister の HTTP リクエストを他のサブネットから独立して処理し，HTTP リクエストが他のサブネットにルーティングされて実行されることはありません。
* Canister は、`http_request` メソッドを使用してマネージメント Canister API を呼び出すことにより、HTTP リクエストを発信します。
* このリクエストはサブネットで複製されたステートに一時的に保存されます。
* 定期的に（ラウンドごとに）、各レプリカのネットワーキング層にある*アダプター*は、複製されたステートから保留中の HTTP アウトコールを取得します（実際には、レプリカプロセスの内部にあるアダプター層（&lsquo;shim&rsquo;）がこれを行います。アダプター自体は、セキュリティ上の理由から、別の OS レベルのプロセスにサンドボックス化されています）。
* 各レプリカ上のアダプターは、HTTP リクエストをターゲット・サーバーに送信して実行します。
* サーバーからの HTTP レスポンスは、サブネットの各レプリカ上のアダプターが受信し、レプリカプロセスのコンポーネントに提供されます。アダプターは、ネットワークレスポンスサイズを `max_response_bytes` に制限しています。この値はリクエストの価格に影響するので、期待するレスポンスに対してできる限り低く設定することが重要です。価格は `max_response_bytes` のサイズとともに増加し、実際のレスポンスサイズは考慮されず、最大値のみが考慮されます。
* 各レプリカのレスポンスに対して、Canister の一部として実装されたオプションの変換関数が呼び出され，レスポンスが変換されます。これはレプリカプロセス内のコンポーネントによって行われます。
* 変換されたレスポンスは、各レプリカのコンセンサスに引き渡されます。
* ここで、少なくとも $2/3$（正確には $2f+1$、 $f$ はプロトコルが許容する欠陥レプリカ数です）のレプリカが、入力としてのリクエストに対して同じレスポンスを持つ場合、IC コンセンサスはレスポンスに同意します。この場合、コンセンサスはこのレスポンスをマネージメント Canister API に返します。コンセンサスが得られない場合やその他の問題がある場合は、エラーを返します。
* マネージメント Canister API は、呼び出しをした Canister にレスポンスまたはエラーを返します。

![HTTPS outcalls high-level architecture](../_attachments/HTTPS_outcalls_HL_architecture.jpg)

上の図は、Canister がどのように機能と相互作用し、サブネットレプリカが外部サーバーと通信するパターンを概観しています。

要約すると、HTTP リクエストを実行するために、各レプリカは、外部の Web サーバーから受け取った HTTP レスポンスの（オプションで変換された）インスタンスを Internet Computer のコンセンサス層を介してプッシュします。サブネットのレプリカは、レプリカが受け取ったすべてのサーバーレスポンスに基づいて、Canister に提供するレスポンスに合意できるようにします。オプションの変換により、サーバーから各レプリカで受信したレスポンスが部分的に異なる場合、それらの違いは排除され、同じ変換されたレスポンスがすべての（正直な）レプリカのコンセンサスに提供されることが保証されます。これにより、すべてのレプリカにおいて、全く同じレスポンス（または何も違わない）が Canister の実行に使用されることが保証され、それによってこの機能を使用するときに不一致が起こらず、サブネットの複製ステートマシンの性質が維持されることが保証されます。

この機能には、変換機能と IC コンセンサスが重要な役割を果たすことがおわかりいただけると思います。変換機能は、レプリカが受信したレスポンスの差異を確実に取り除き、異なるレプリカ上の変換されたレスポンスが全く同じになるようにすることで、コンセンサスが呼び出し側の Canister に合意されたレスポンスを提供することを可能にします。レプリカが受信したレスポンスをコンセンサスで実行することで、どのレプリカもスマートコントラクト Wasm の実行環境に対して同じレスポンスを提供することを保証しています。

### トラストモデルとプログラミングモデル

Canister HTTP アウトコールのトラストモデルは，呼び出された HTTP サーバーと IC のトラストモデルに基づいています：
* HTTP サービスは正直であると仮定します。そうでなければ，サブネットのどの呼び出し側レプリカに対しても，任意のレスポンスを提供することができません。これは、Web 2.0 サービスやオラクルからサービスを呼び出す場合と同様で、ブロックチェーンからサービスを呼び出す場合もこれと同じシュチュエーションです。
* IC のトラストモデルは、少なくとも $2/3$ のレプリカが正直であると仮定しています。この場合、正直なレプリカは（オプションの変換後に）同じレスポンスを取得し、そのレスポンスを呼び出し元の Canister に提供し返せるように、コンセンサスを得ることができます。
不誠実なサーバー（レプリカ）は、簡単にリクエストを失敗させたり、間違ったデータを提供したりすることになります。また、$1/3$ 以上の不誠実なサブネットレプリカでリクエスト自体を失敗させることができます。2/3$以上のレプリカが危険にさらされると、間違ったデータを提供してしまうことになります。

この機能のプログラミングモデルは、HTTP アウトコールを行う Canister が *HTTP クライアント*として動作し、様々なヘッダーを解釈し、それに応じて動作する、よくある一般的なケースようなものと似ています。API リクエストのような使用例では、これはかなり簡単で、標準的なケースでは多くの特別な考慮は必要ありません。IC プロトコルスタックは、概念的には、Canister と従来の HTTP サーバーの間の通信パイプとみなすことができ、HTTP レスポンスが合意を経て、すべての正直なレプリカが実行時にまったく同じレスポンスを受け取るようにする、つまり、呼び出し側の Canister が合意されたレスポンスを受け取るようにするものです。

## HTTPS アウトコールと オラクルの比較

*ブロックチェーンオラクル*、または単に*オラクル*は、他のユーザーと同様に、クエリを作成したり、イングレスメッセージを送信することによってブロックチェーンと対話する外部パーティです。これまでのところ、ブロックチェーンの世界では、スマートコントラクトを従来の Web2.0 サーバーと通信させるための主要な手段となっています。

基本的なアーキテクチャは、チェーン上にオラクルスマートコントラクトが配置されていることです。ユーザーはこのオラクルスマートコントラクトにリクエストを行い、オラクルスマートコントラクトはそのリクエストを一時的に保存します。オラクル、つまり Web 2.0 サーバーは、クエリやアップデートコールを使ってオラクルスマートコントラクトと通常のやり取りをすることで、保存されたリクエストを取得します。次に、オラクルは通常の HTTP コールによって Web 2.0 の世界でリクエストを実行し、ブロックチェーンとの通常の対話パラダイム（アップデートコール）を使用して、オラクルスマートコントラクトにレスポンスを返します。その後、オラクルスマートコントラクトは、標準的なオンチェーン相互作用を使用して、呼び出したスマートコントラクトにレスポンスを提供し返します。

オラクルアーキテクチャでは、単一のオラクルがオリジナルのリクエストを実行するかもしれないし、オラクルの分散ネットワークがそれぞれリクエストを発行し、合意したレスポンス（証拠を含む）をオラクルコントラクト、最終的には呼び出し側の Canister に提供するかもしれません。このように、オラクルの世界でアウトコールと同じ結果を効果的に達成することができますが、そのサービスの対価を求めるひとつ以上の外部パーティを必要とし、より複雑なアーキテクチャになります。したがって、HTTP アウトコールは多くの関連するユースケースでオラクルに取って代わり、より強力なトラストモデルで、より低い手数料とより低いリクエスト実行レイテンシーでそうすることができると仮定することができます。

オラクルサービスは、追加の機能またはサービスを提供することができます。ひとつの顕著な例は、与えられた情報項目（例えば、アセット価格）に対して複数のデータソースを使用し、その結果として正規化されたレスポンス（例えば、中央値価格）を提供することです。別の例としては、スマートコントラクトがオラクルとの対話なしに、直接アクセスできるように取得された価格履歴のセレクションをチェーン上で利用可能にしておくことです。これらの機能はいずれも、Canister の HTTP アウトコール機能で構築でき、オラクルは必要ありません。

全体として、Canister HTTP アウトコール機能は、オラクルやオラクルネットワークが行ってきたことを、さらなる障害点や手数料を獲得を目的にした付加的なパーティを必要とせずに、より強力なモデルで実現することができます。Canister HTTP アウトコール機能により、Web2.0 サーバーとのインタラクションは、よりシンプルなアーキテクチャにより、手数料の削減と低レイテンシーの恩恵を受けることができます。

### 開発者の便益

開発者にとって、Canister HTTP リクエスト機能は、 IC 以外にトラストする必要がなく、どの当事者（オラクル）を信頼するかを決定する必要がないという利点があります。トラストの前提を減らしより良い分散化を得るために、単一のオラクルを使用するか、複数のオラクルを使用するかを決定する必要がなく、HTTP アウトコールでこれらを得ることができるのです。HTTP アウトコールのコストは、確立されたオラクル・プロバイダーにそのサービスと関連するイングレス・コストを支払うよりもはるかに低いと思われます。したがって、HTTP コールを使用することには明らかな経済的利点があります。分かりにくい信頼性の判断から解放された開発者は、ビジネスロジックに集中し、必要な HTTP コールを行う（そして対応する変換関数を書く）だけでよく、より自然な形で開発することができます。

### エンドユーザーの便益

エンドユーザーは HTTP アウトコールから、様々な方法で利益を得ることができます。
* *より強力なトラストモデル：* エンドユーザーは、さらなる障害点になるかもしれない付加的なパーティーに依存しないことで、より強力なセキュリティを得ることができ、それによってより良い分散化の恩恵を受けることができます。オラクルサービスのようなサードパーティが関与しないので、サブネットのレプリカと呼び出される外部サービスだけを使った我々の HTTP アウトコールは分散化の最たるものです。
* *低料金：* オラクルのミドルパーティがカットされるため、ユーザーは通常より安いサービスを得ることができます。HTTP を直接呼び出す場合のレイテンシーは、オラクルコントラクトに（Xnet）リクエストを出し、ポーリングののちにサービスされる外部のオラクルサービスよりも低く、Canister と対話する際のユーザーエクスペリエンスが向上します。オラクルプロバイダが関連するすべてのサブネットにオラクルスマートコントラクトをセットアップしないといけなくなる（複数の） Xnet リクエストの場合に顕著で、これは IC のサブネットアーキテクチャの原則とやや矛盾することにもなります。

## 既知の制限事項

現在の MVP にはいくつかの制限があり、エンジニアがこの機能に何を期待できるのか、また、あるユースケースに対してこの機能を適用できるのかを知るために、明示することが必要です。これらの制限のいくつかは、将来の機能拡張で対処されるかもしれません。

### レスポンスは&ldquo;同様&rdquo;でなければならない

サブネットのレプリカが受け取るレスポンスは、各レスポンスが同じ変換関数を受けることができ、変換の結果がどのレプリカでも等しいという意味で、&ldquo;同様&rdquo;でなければなりません。したがって、この文脈で同様とは、我々が関心を持ついくつかの核となる情報がすべてのレスポンスで等しく、他の部分は異なっていても、レスポンスには関係ないことを意味します。HTTP API の世界でよくある設定は、レスポンスは構造的に等しいが、レスポンスに含まれるタイムスタンプや識別子などのために、あるフィールドが異なるというものです。

### POST リクエストはべき乗でなければならない

この機能を実装する方法は、すべての HTTP リクエストをサブネットのすべてのレプリカが行う必要があることを意味します。これは `GET` だけでなく `POST` リクエストにも当てはまり、後者では `GET` リクエストにはない課題が生じ、基本的な HTTP の原則に従って適切に実装されるならば、その課題はべき等です。べき等とは、リクエストが複数回行われた場合、同じ結果をもたらし、サーバーの状態を変化させないということです。べき等は `POST` リクエストには適用されません。つまり、さらなる予防措置がなければ、Canister から行われた `POST` は、サブネットのレプリカの数を $n$ として、$n$ 回、呼ばれたサーバーの更新につながるリクエストになる可能性があるということです。この動作は明らかに意図したものではありませんし、ほとんどのシナリオで受け入れられるものでもありません。

このようなシナリオで実際に使われる標準的な解決策は、リクエストに *べき等鍵* を使用することです。この鍵は、異なるレプリカが生成したすべてのリクエストと一緒に送られるヘッダーにある、一意のランダムな文字列です。サーバーはひとつ以外のすべてのリクエストを重複と見なし、それらはサーバーの状態を変更するために考慮されず、ひとつのみが考慮されます。この結果、`POST` の意図した動作が、サーバーによって一度だけ適用されることになります。しかし、サーバーによって処理されるこのひとつのリクエストは、実際には不正なレプリカによって行われたリクエストである可能性があり、また意図されていないリクエストであるかもしれないことに注意してください。また、不正なレプリカが、べき等鍵を変更することで、そのリクエストと実際の意図したリクエストがサーバーによって処理されてしまうかもしれません。

べき等鍵は一部のサーバーでサポートされていますが、すべてのサーバーでサポートされているわけではありません。べき等鍵の適用可能性はケースバイケースで評価される必要があります。べき等であるという課題は、将来的にこの機能を拡張し、定足数を減らすことで部分的に軽減することができます。つまり、Canister はクォーラムサイズを 1 と定義し、ひとつのレプリカに `POST` リクエストを実行させることができます。不正なレプリカの問題は明らかにまだ残っていますが、べき等性の問題はこの拡張機能で解決されます。

リクエストが適切に実行されたかどうかをチェックするために、`GET` を介して安全な *読み直し* 操作を行うことは、課題のいくつかを軽減するのに役立ちます。しかし、不正なレプリカが状態の全く異なる部分を更新することは、この方法では信頼性をもって検出できません。

### 不正なレプリカ

すでに述べたように、不正なレプリカはより深刻な問題です。そのようなレプリカは、正直なレプリカによって実装された Canister が気づいていない任意の `POST` リクエストを、いつでもサービスに対して行うことができます。つまり、外部サーバーを認証するために API キーやパスワードを保存している場合、不正なレプリカが保存されている平文の認証情報を使ってサーバーを認証してしまうという問題に直面するのです。

しかし、この問題はオラクルでも簡単に解決できないことに注意してください。不正なオラクルは不正なレプリカと全く同じことをすることになります。複数のオラクルがあっても、`POST` の問題は解決されません。これを解決するには、閾値 TLS のような高度な（暗号）機構を使用するか、サーバー側で何らかのサポートが必要です。例えば、サーバーは与えられた API キーに対して行われたすべてのステート変更のログを保持し、Canister はこのログを安全に読み返し、少なくとも不正なレプリカによる呼び出しを検出することができます。リクエスト元の IP アドレスを追跡することで、不正なレプリカを特定し、それに対して法的措置やその他の措置を取ることができるかもしれません。

Internet Computer と Web2.0 サーバー間の書き込み統合を安全にするための一例として、Chain Key 暗号を用いたサブネット署名で `POST` リクエストに署名し、サーバーにその署名をチェックさせるという方法があります。このようなアプローチはサーバーの適応を必要とするが、不正なレプリカが任意のリクエストを行うという、残りのセキュリティ問題を解決することができます。

これまで見てきたように、不正なレプリカが `POST` コールに与える影響を完全に取り除くには、HTTP2.0 サービスによるサポートが必要かもしれません。そのようなサポートがあれば、Web3.0 と Web2.0 の間のシームレスで安全な相互運用性を実現することができます。

### IPv6 のみサポート

現在、IC 自体が IPv6 ベースのシステムであるため、Canister の HTTP アウトコール機能は IPv6 のみの機能となっています。今後、コミュニティからの要望に応じて IPv4 対応を行う可能性があります。現在 IPv4 をサポートしていない主な理由は、IC の各レプリカが独自の IPv4 アドレスを取得できるほどの IPv4 アドレスがないため、バウンダリーノード上でトラフィックをルーティングするか、他のメカニズムを使用して IPv4 通信を可能にする必要があるためです。これらのアプローチはいずれも分散化を抑制し、Web2.0 サーバーと直接通信するレプリカと比較してトラストモデルを弱めることに繋がります。

### サーバーによるレート制限

一般消費者向けにサービスを提供しているほとんどの Web サーバーは、ある種の速度制限を実装しています。レート制限とは、ある時間間隔で IPv4 アドレスまたは IPv6 プレフィックスから行えるリクエストの数を制限することです。あるアドレスやプレフィックスの割り当てを使い切ると、この IP やプレフィックスからのリクエストは処理されず、代わりに相応のエラー応答が処理されることになります。

問題は、サブネット上のすべての Canister が、このサブネットのレプリカの IPv6 プレフィックスを共有し、したがって、レート制限を実装する各サーバーのクォータを共有することです。一般（未認証）ユーザー向けの HTTP サービスの典型的なクォータは、1秒あたり 10$ リクエストのオーダーです。どのレプリカも同じリクエストをするので、$n$ の増幅率があります。$n$ はサブネット内のレプリカの数です。このため、クォータはすぐに消費され、レート制限によりスロットリングやレプリカが（一時的に）ブロックされる可能性があります。

登録ユーザーを使い、API キーでリクエストを承認することで、同じサーバー上のユーザーを切り離し、それぞれに割り当てを与えることができ、公開 API での問題を解決することができます。ただし、API キーは各レプリカに保存され、レプリカが侵害された場合に公開される可能性があることに注意してください。これはスマートコントラクトのセキュリティモデルで考慮する必要があり、特に `POST` 操作に関連します。詳細については、[discussion on compromised replicas](#compromised-replicas) を参照してください。

登録ユーザーを利用することで、公開 API のレート制限や、すべての Canister がサービスの割り当てを共有することの問題のほとんど、あるいはすべてを解決できると考えているため、現在 IC プロトコルスタックにレート制限を実装していません。

## 今後の拡張性

すでにさらに上で述べたように、コミュニティの需要に基づいて、将来的に実装することを検討している複数の可能な拡張機能があります。現在の MVP は、機能セットを減らすことで、より早く市場に投入することができるため、それらを含んでいません。
* *IPv4 サポート：* IPv4 サポートは、Canister が IPv6 では利用できず、IPv4 のみで利用できるサーバーにリーチするのに役立つでしょう。
* *縮小クォーラム：* Canister は、縮小クォーラムがリクエストに使用されるべきと定義できます。例えば、すべての $n$ レプリカの代わりに、リクエストを行うサブネットの $1$ レプリカのみが使用されます。これは `POST` リクエストのべき等問題を解決するのに非常に有効です。また、信頼度は重要ではなく、サーバーのクォータによってリクエストの増幅を抑えたい場合にも有効です。

## HTTPS アウトコールのコーディング

次に、Canister スマートコントラクトで Canister HTTP リクエストを使用したいエンジニアのために、重要な情報を提供します。Canister HTTP リクエストを使用することは、一般的なケースにおいて、通常のエンタープライズ・アプリケーションで HTTP リクエストを行うよりもやや困難です。また、[The Internet Computer Interface Specification](../../../references/ic-interface-spec/#ic-method-http_request) の機能の API 定義も読者に紹介します。

### Canister HTTP コールのコーディングのレシピ

次の &ldquo;レシピ&rdquo; は、新しい Canister HTTP アウトコールタイプの実装に取り組むための最良の方法のブループリントになります。
* &ldquo;手動&rdquo; 例えば、`curl` ツールを使って、サブネットのレプリカによって行われることをエミュレートするために、短い時間枠（1-2秒）内に関心のある同じ HTTP リクエストを2回、ちょうどより小さい $n$ で行います。
* 2つのレスポンスの差分を取って、2つのリクエストで異なる項目をすべて見つけます。ここでは、ボディとヘッダーの両方を考慮することが重要です。あるいは、関心のあるコア情報と、それがどのようにレスポンスとして抽出されるかを特定します。
* 観測されたレスポンスの差分や意図したレスポンスに基づいて、各レプリカで等しくなるようにレスポンスを変換する*変換関数*を実装します。
* このタイプのリクエストに対するサーバーのレスポンスの最大サイズを決定して、 `max_response_bytes` パラメータにそれを入力します。これは API コールでよく機能し、Cycle に関してリクエスト側の Canister を過負荷しないようにすることが重要です。レスポンスサイズがかなり動的に変化する場合は、 `HEAD` リクエストタイプを使用して、これを行うことができます。
* リクエストを実装して、ローカル SDK 環境で試してみてください。ただし、ローカル環境にはレプリカがひとつしかなく、IC のサブネットには $n$、たとえば $n=13$ のレプリカがあるので、ローカル環境の動作が IC の動作に反映されないことに注意してください。

:::tip Pro tip

ヘッダーには、タイムスタンプなどの可変項目が含まれていることがあり、レスポンスがコンセンサスを通過するのを妨げる可能性があるため、レスポンスの可変部分を特定する際には、レスポンスヘッダーを考慮することを忘れないでください。

:::

### 変換関数

変換関数の目的は、レプリカ $i$ が受信した各レスポンス $r_i$ を変換することであり、異なるレプリカが受信した $r_i$ は異なる場合があります。この変換関数は、IC コンセンサスの一部としてレスポンスに合意できるように、誠実なレプリカのすべての $r'_i$ が等しくなることを意図して、レスポンス $r_i$ を変換されたレスポンス $r'_i$ に変換するものです。変換関数は Canister のプログラマーによって提供されなければならず、Canister によってクエリとして公開されます。Canister は任意の数の変換関数を定義することができます。マネージメント Canister に `http_request` を呼び出す際に、オプションで変換関数を入力として提供することができます。HTTP リクエストの目的に応じて、変換関数の書き方はさまざまです。
* 興味のある情報項目のみを抽出する。これは、例えば、 Canister に転送される日付と資産価格からなるペアのリスト、または単一の資産価格とすることができます。この方法は、Canister で HTTP レスポンス全体を必要とせず、特定の情報項目のみを必要とする場合に有効です。
* リクエストの可変部分をすべて個別に削除し、残りの部分をすべて保持する。その後、Canister は関連情報を抽出します。これは、Canister が HTTP リクエストの構造とヘッダをまだ必要とする場合に便利です。

私たちは、複数の利点があるため、可能な限り最初のアプローチを行うことをお勧めします。価格設定や他の多くの API への呼び出しは、ほとんど最初のアプローチで動作するはずです。
* 結果として得られるレスポンスはより小さいサイズになります。
* 変換関数はより速く計算されるかもしれません、つまり、関数（クエリ）はより少ない CPU Cycle で実行されます。IC のクエリは現在無料ですが、将来的には有料になる可能性があることに注意してください。
* 変換関数は、例えば、レスポンス・ボディに対する単純な、あるいはいくつかの単純な Json 操作によって、容易に実装することができます。

### エラーとデバッグ

この機能を使用する際に起こりうるエラーケースは数多くあります。最も重要なものを次に挙げます。
* *SysFatal - Url needs to specify https scheme:* この機能は現在、HTTPS 接続のみを許可しており、プレーン HTTP を使用するとエラーになります。
* *SysFatal - Timeout expired:* リクエストは、タイムアウト期間内に実行されない場合、タイムアウトします。これが起こる重要な例のひとつは、コンセンサスを得るために十分な数の等しいレスポンスがないときです。これは、例えば、変換関数が応答のすべての可変部分を考慮に入れて正確に書かれていないときに起きます。
* *SysTransient - Failed to connect: error trying to connect: tcp connect error: Connection refused (os error 111):* このエラーは、相手サーバーとの TCP 接続が確立できなかったことを示します。
* *CanisterReject - http_request request sent with 0 cycles, but ... cycles are required:* 少なくとも必要な量の Cycle が、サブネットで満たされるために要求と共に送信される必要があります。
* *CanisterReject - max_response_bytes expected to be in the range [0..2097152], got …:* このエラーは、サーバーから受信したネットワークレスポンスが大きすぎることを示します。このエラーは、サーバーから受け取ったネットワーク応答が大きすぎることを示します。これは、応答サイズが過小評価され、`max_response_bytes` の値が低く設定された場合に発生します。
* *SysFatal - Transformed http response exceeds limit: 2045952:* このエラーは、変換されたレスポンスサイズの制限に達したことを示します。これは現在、HTTPS アウトコール機能のハードレスポンスサイズ制限です。レスポンスサイズは、レスポンスボディとヘッダーに基づいて計算されることに注意してください。

この機能を初めて使う開発者は、最初のうちはある種の問題にぶつかる可能性が高く、それは以下のエラーのいずれかとして具体化されます。この機能を使い始める際に、最も顕著であると思われる問題を列挙します。
* ある特定のタイプの Canister HTTP リクエストがローカルの dfx 環境では動作しても、IC では通常のアプリケーションのサブネットで $n=13$ 個のレプリカが動作するのに対し、ローカル環境では $1$ 個のレプリカが動作するため、動作しない可能性があります。このような問題は、このような呼び出しを開発する際、特に開発者がその機能を扱うのに必要な経験をまだ積んでいない場合に予想されます。ここで予想される主な問題は、変換関数の不足または問題です。なお、dfx 環境と IC へのデプロイの違いは、dfx 環境の動作に起因するものであり、すぐに変わるものではありません。dfx 環境は、ローカルにひとつのレプリカを実行し、エンジニアリングプロセスにおけるすべての長所と短所を伴います。
* タイムアウトを受信する：HTTP サーバーから返されたリクエストが、機能で要求されるように &ldquo;同様&rdquo; でない場合、これは変換関数のエラーが原因である可能性が高いです。つまり、変換されたレスポンスは、合意を得るために十分に多くの正直なレプリカでまだ等しくないため、IC ブロックにレスポンスが追加されることはありません。最終的には、タイムアウトにより、この HTTP アウトコールに関連するすべてのアーティファクトが削除されます。この問題は、サービスに対して行われた複数のリクエストを差分し、変換関数が変換結果の変数部分を保持しないようにすることで、最も効果的にデバッグできます。
* リクエストがあまりにも多くの Cycle を消費する：Canister HTTPS アウトコールは Cycle が課金されます。しかし、かなり小さなレスポンスのリクエストが頻繁に非常に大きな Cycle を消費する場合、おそらく原因は `max_response_size` パラメータがリクエストに設定されていないことです。この場合、システムは最大応答サイズである $2$ MB を想定し課金します。このパラメータには常に、実際に期待される最大応答サイズにできるだけ近い値を設定し、少なくとも同程度の大きさで、それより小さい値でないことを確認してください。`max_response_size` パラメータはボディとヘッダの両方で構成され、Canister への最終的なレスポンスではなく、サーバーからのネットワークレスポンスを参照します。

### 価格

IC の他の機能と同様に、Canister HTTP アウトコール機能は、使用時に課金されます。現在の価格設定は、HTTP リクエストに対して $400$ M Cycle の基本料金に加え、リクエスト 1バイトと `max_response_bytes` バイトごとに $100$ K Cycle を課金するよう定義されています。リクエストごとの固定コストと、ヘッダーなどによる HTTP リクエストのオーバーヘッドのため、アプリケーションの観点から実行可能であれば、より小さなリクエストを多数回行うのと同じ情報を取得するために、より大きなレスポンスを持つより少ないリクエストを行うことがコストの観点から有利です。呼び出しで提供される Cycle は、リクエストのコストをカバーするのに十分でなければならず、過剰な Cycle は呼び出し元に返されます。

現在の価格設定はかなり保守的（高価）であると定義されており、価格は今後、価格モデルの更新が導入されることで変更される可能性があります。しかし、ファイナンス API への問い合わせに使用されるような小さなレスポンスを伴う HTTP アウトコールは、USD セントの端数しかかからず、ほとんどのブロックチェーンでオラクルがコールする際に課される手数料よりも大幅に安いことに注意してください。

## コミュニティへの貢献

ドキュメントにまだ記載されていない問題が発生した場合、またはエンジニアがより効果的にこの機能を使用できるようなドキュメントの改善案がある場合は、[forum topic](https://forum.dfinity.org/t/enable-canisters-to-make-http-s-requests/9670) でお知らせください。

<!--
# Technology Overview — How Canister HTTPS Outcalls Work

On this page we provide details on how canister HTTPS outcalls, or canister HTTPS requests, work and important aspects to consider when using the API. We also want to note that there are some limitations and differences  compared to regular (Web 2.0) computer programs making HTTP calls and considerations for programmers for successfully using this feature. Engineers who intend to use this feature are advised to read through this page to get up to speed quickly w.r.t. the feature. The impatient reader who wants to dive into coding right away can skip the conceptual part and jump right away to the [coding section](#coding-https-outcalls) to get started.

## Technology

The HTTPS outcalls feature allows canisters to make outgoing HTTP calls to conventional Web 2.0 HTTP servers. The response of the request can be safely used in computations of the canister, without the risk of state divergence between the replicas of the subnet.

### How an HTTPS Outcall is Processed by the IC

The canister HTTP outcalls feature is implemented as part of the Internet Computer replica and is exposed as an API of the management canister. We next outline, in a simplified form, how a request made by a canister is processed. The HTTP request functionality is realized at the level of subnets, i.e., each subnet handles canister HTTP requests of its canisters independently of other subnets and HTTP requests are never routed to other subnets for execution.
* A canister makes an outgoing HTTP request by calling the management canister API using the `http_request` method.
* The request is stored temporarily in the replicated state of the subnet.
* Periodically (each round) an *adapter* at the networking layer in each replica fetches the pending HTTP outcalls from replicated state. (In fact, a &lsquo;shim&rsquo; layer of the adapter that is inside the replica process does so as the adapter itself is sandboxed into a separate OS-level process for security reasons.)
* The adapter on each replica executes the HTTP request by sending it to the target server.
* The corresponding HTTP response from the server is received by the adapter on each replica of the subnet and provided to a component in the replica process. The adapter limits the network response size to `max_response_bytes` which defaults to $2$ MB and can be set to lower values. It is important to set this as low as reasonably possible for the expected response as it affects the price of the request: The price increases with the size `max_response_bytes` and the actual response size is not considered, only the maximum.
* An optional transformation function implemented as part of the canister is invoked on the respective response on each replica to transform the response. This is done by a component within the replica process.
* The transformed response is handed over to consensus on each replica.
* IC consensus agrees on a response if at least $2/3$ (to be more precise, $2f+1$, where $f$ is the number of faulty replicas tolerated by the protocol) of the replicas have the same response for the request as input. In this case, consensus provides this response back to the management canister API, or an error if no consensus can be reached or in case of other problems.
* The management canister API provides the response or error back to the calling canister.

![HTTPS outcalls high-level architecture](../_attachments/HTTPS_outcalls_HL_architecture.jpg)

The above figure shows a high-level view of how a canister interacts with the feature and the communication patterns of the subnet replicas with external servers.

To summarize, to perform an HTTP request, each replica pushes an (optionally-transformed) instance of the received HTTP response from the external Web server through the Internet Computer's consensus layer, so that the replicas of the subnet can agree on the response provided to the canister, based on all server responses received by the replicas. The optional transformation ensures that, if responses received on different replicas from the server are different in some parts, those differences are eliminated and the same transformed response is provided to consensus on every (honest) replica. This guarantees that on every replica the exact same response (or none at all) is used for canister execution, thereby ensuring that divergence does not happen when using this feature and the replicated state machine properties of the subnet are preserved.

As we can see, the transformation function and IC consensus play a crucial role for this feature: The transformation function ensures that differences in the responses received by the replicas are removed and transformed responses on different replicas will be exactly the same, thus enabling consensus to provide the agreed-upon response to the calling canister. By running the responses received by the replicas through consensus, we ensure that every replica provides the same response to the smart contract Wasm execution environment.

### Trust Model and Programming Model

The trust model for canister HTTP outcalls is based on the trust model of the called HTTP server and that of the IC:
* It is assumed that the HTTP service is honest, otherwise, it can provide any responses to any of the calling replicas of the subnet. This is the case also for calls to the service from a Web 2.0 service or an oracle, thus, the situation is the same as in our scenario of a blockchain making the call to the service.
* The trust model of the IC assumes that at least $2/3$ of the replicas are honest. In this case, the honest replicas will obtain the same response (after optional transformation) and consensus will be able to agree on the response so that the response can be provided back to the calling canister.
A dishonest server can easily make requests fail or provide wrong data. Also, $1/3$ or more of dishonest subnet replicas can make requests fail. For providing wrong data, more than $2/3$ of the replicas need to be compromised.

The programming model for this feature is such that the canister making an HTTP outcall acts as *HTTP client*, i.e., needs to, in the general case, interpret various headers and act accordingly. For use cases such as API requests this is rather straightforward and does not require many specific considerations in the standard case. The IC protocol stack can conceptually be simply seen as a communication pipe between the canister and the conventional HTTP server that makes sure that the HTTP response makes it through consensus and all honest replicas receive the exact same response in execution, i.e., the calling canister receives an agreed-upon response.

## HTTPS Outcalls and Oracles Compared

*Blockchain oracles*, or just *oracles*, are external parties that interact with the blockchain by making queries or sending ingress messages, just like any user. They have so far been the main means in the blockchain world of letting smart contracts communicate with traditional Web 2.0 servers.

The basic architecture is that an oracle smart contract is deployed on chain. Users make requests to this oracle contract, which stores the requests temporarily. An oracle, i.e., Web 2.0 server, obtains the stored request from the oracle smart contract through regular means of interacting with it using queries and update calls. Then the oracle executes the request in the Web 2.0 world by means of a regular HTTP call and provides back the response to the oracle smart contract, again using the normal interaction paradigm with the blockchain (update call). Then the oracle smart contract provides back the response to the calling smart contract using a standard on-chain interaction.

In the oracle architecture, a single oracle may fulfill the original request, or a decentralized network of oracles may each issue the request and then provide an agreed-upon response (including evidence) to the oracle contract and ultimately to the calling canister. As we can see, the oracle world effectively can achieve the same result as HTTP outcalls, but with a more complex architecture of requiring one or more external parties that want to be paid for their services. Thus, one can assume that HTTP outcalls can replace oracles for many relevant use cases, and do so in a stronger trust model and with lower fees and lower request execution latency.

Oracle services may provide additional functionality or services. One prominent example is using multiple data sources for a given information item, e.g., an asset price, and then providing a normalized response, e.g., the median price, as a result. Another example is keeping a selection of historic prices that have been retrieved available on chain for direct access by smart contracts without any oracle interaction. Both those functionalities can be built with the canister HTTP outcalls feature and do not require oracles.

Overall, the canister HTTP outcalls feature allows us to achieve what oracles or oracle networks do, but in a stronger model without requiring additional parties that are additional points of failure and want to earn fees for their services. With the canister HTTP outcalls feature, interactions with Web 2.0 servers benefit from reduced fees and lower latency due to the simpler architecture.

### Benefits for Developers

For developers, the canister HTTP requests feature has the benefit that they do not need to make a decision on which party (oracle) they want to trust in addition to the IC that they already need to trust anyway. They don't need to decide on whether they want to use a single oracle or multiple oracles to reduce trust assumptions and get better decentralization, but rather they get this out of the box with HTTP outcalls. The cost of the HTTP outcall is likely much lower than paying an established oracle provider for their services and the associated ingress cost, thus we have a clear economic advantage to use HTTP calls. Freed from making all those non-trivial trust decisions, a developer can focus on their business logic and simply make the HTTP call they need (and write a corresponding transform function), which is more natural.

### Benefits for End Users

End users can benefit from HTTP outcalls in various ways as well.
* *Stronger trust model:* They get stronger security by not relying on any additional parties which resemble further points of failure and thereby benefit from better decentralization. The decentralization we get using HTTP outcalls is the best we can achieve as it only involves the subnet replicas and the external service to be called, but no third parties such as oracle services.
* *Lower fees:* Users typically get a cheaper service as oracle middlemen are cut out in the calculation.
* *Lower latency:* The latency of direct HTTP calls is lower than making an (Xnet) request to an oracle contract that then gets polled and serviced by an external oracle service, which materializes in a better user experience when interacting with the canister. This is more pronounced for Xnet requests, which are required unless the oracle provider sets up an oracle smart contract on every relevant subnet, which would somewhat contradict the subnet architecture principle of the IC.

## Known Limitations

The current MVP has some limitations that we need to make explicit s.t. engineers know what they can expect from the feature and whether they can apply it for a given use case. Some of those limitations may be addressed with future extensions of the feature.

### Responses Must be &ldquo;Similar&rdquo;

Responses the replicas of the subnet receive must be &ldquo;similar&rdquo; in the sense that each response can be subjected to the same transformation function and the outcome of the transformation will be equal on every replica. Thus, &ldquo;similar&rdquo; in this context means that some core information we are interested in is equal in all responses and other parts may differ, but are not relevant for the response. A common setting in the world of HTTP APIs is that the responses are structurally equivalent, but contain certain fields that differ in the responses, for example, because of timestamps and identifiers contained in the responses.

### POST Requests Must be Idempotent

The way the feature is implemented implies that every HTTP request must be made by every replica of the subnet. This not only applies to `GET`, but also to `POST`, requests and for the latter creates a challenge that we do not have for `GET` requests: A `GET`, if implemented properly according to basic HTTP principles, is idempotent. Idempotency means that, if a request is made multiple times, it yields the same result and does not change the server's state. Idempotency does not apply to `POST` requests, meaning that, without further precautions, a `POST` made from a canister could result in the request leading to an update on the called server $n$ times, with $n$ the number of replicas in the subnet. This behaviour is clearly not intended, nor would it be acceptable in most scenarios.

One standard solution used in practice for such scenarios is the use of an *idempotency key* in the request. This key is a unique random string in a header sent along with all resulting requests by the different replicas. The server identifies all but one of the requests as duplicates and those are not considered for changing the state on the server, only one of them is. This results in exactly the intended behaviour of the `POST` being applied exactly once by the server. However, note that this single request being processed by the server can actually be the request made by a compromised replica and thus not be the intended request. Also, the compromised replica may change the idempotency key and thus lead to its request and the actual intended request getting processed by the server.

Idempotency keys are supported by some servers, but definitely not by all. The applicability of idempotency keys needs to be evaluated on a case-by-case basis. The challenge of idempotency can, in parts, be mitigated with a future extension of this feature allowing for a *reduced quorum*. That is, the canister could define the quorum size to be 1 and have only 1 replica execute the `POST` request. The issue with dishonest replicas clearly still persists, but the idempotency issue gets solved with this extension.

Making a secure *read back* operation via a `GET` to check whether the request has been properly executed can help mitigate some of the challenges. However, a compromised replica making an update on a completely different portion of the state can not reliably be detected using this approach.

### Compromised Replicas

As already hinted at above, a compromised replica is a more general problem: Such replica can at any point in time make arbitrary `POST` requests that the canister, as implemented by the honest replicas, is not even aware about, to the service. I.e., whenever we have an API key or password stored for authenticating to the external server, we run into the problem that a compromised replica can use the stored plaintext credentials to authenticate to the server.

Note, however, that this problem cannot be easily solved with oracles either: A compromised oracle can do exactly the same as a compromised replica. Multiple oracles do not solve the problem either for `POST`s. Solving this requires either advanced (cryptographic) mechanisms such as threshold TLS being used or some support from the side of the server. E.g., the server could keep a log of all state changes being done for a given API key and the canister can read back this log securely and at least detect calls made by compromised replicas. Tracking the IP address of the requestor may help in identifying the faulty replica and taking legal and other action against it.

One example of securing the write integration between the Internet Computer and Web 2.0 servers is to sign `POST` requests with a subnet signature using chain key cryptography and to have the server check the signature. Such approach requires adaptation of the server, but can resolve the remaining security problems of dishonest replicas making arbitrary requests.

As we have seen, completely removing the impact that dishonest replicas can have on `POST` calls may require support by the HTTP 2.0 service. With such support, seamless and secure interoperability between Web 3.0 and Web 2.0 can be achieved.

### IPv6-Only Support

Currently, as the IC itself is an IPv6-based system, the canister HTTP outcalls feature is therefore an IPv6-only feature. IPv4 support may be added in the future depending on demand from the community. The main reason for not supporting IPv4 now is that we do not have sufficiently many IPv4 addresses such that every replica of the IC could get its own IPv4 address and thus we would need to either route traffic over boundary nodes or use other mechanisms to allow for IPv4 communications. Any of those approaches leads to a reduced decentralization and thus a weaker trust model compared to replicas directly communicating with the Web 2.0 server.

### Rate Limiting by Servers

Most Web servers offering services to the general public implement some sort of rate limiting. Rate limiting means to constrain how many requests can be made from an IPv4 address or IPv6 prefix in a given time interval. Once the quota for an address of prefix is used up, requests from this IP or prefix would not be served, but an according error response would be served instead.

The problem is that all canisters on a subnet share the IPv6 prefixes of the replicas of this subnet and thus the quota at each server that implements rate limiting. Typical quotas of HTTP services for public (unauthenticated) users are in the order of $10$ requests per second. As every replica makes the same request, we have an amplification factor of $n$, where $n$ is the number of replicas in the subnet. Thus, the quotas can get consumed quickly and rate limits may lead to throttling or the replicas being (temporarily) blocked.

Using a registered user and authorizing the requests with an API key can decouple users on the same server and give each of them their own quota, thereby solving the issue with public APIs. Note, however, that the API key is stored in every replica and may be exposed in case a replica is compromised. This needs to be considered in the security model of the smart contract and is specifically relevant for `POST` operations. See the [discussion on compromised replicas](#compromised-replicas) for further information.

We do currently not implement any rate limiting in the IC protocol stack as we think the use of registered users may resolve many if not most or all of the issues with rate limiting of public APIs and all canisters sharing the service's quota.

## Future Extensions

As mentioned already further above, there are multiple possible extensions we are considering to implement in the future, based on demand in the community. The current MVP does not include them as it allows us to go to market faster because of the reduced feature set.
* *IPv4 support:* IPv4 support would help canisters reach servers that are not available on IPv6, but IPv4 only.
* *Reduced quorum:* A canister can define that a reduced quorum should be used for a request, e.g., only $1$ replica of the subnet making the request instead of all $n$ replicas. This trivially helps resolve the idempotency problem with `POST` requests. It may also be interesting for making reads where trust does not matter, but where we want to reduce the request amplification instead, e.g., because of quotas the server has in place.

## Coding HTTPS Outcalls

We next provide important information for engineers who want to use canister HTTP requests in their canister smart contracts. Using canister HTTP requests is somewhat harder in the general case than doing HTTP requests in a regular enterprise application as we need to consider aspects like responses going through consensus and idempotency of `POST` requests. We also refer the reader to the API definition of the feature in [The Internet Computer Interface Specification](../../../references/ic-interface-spec/#ic-method-http_request).

### Recipe for Coding a Canister HTTP Call

The following &ldquo;recipe&rdquo; gives you a blueprint of how to best tackle the implementation of a new canister HTTP outcall type.
* &ldquo;Manually,&rdquo; e.g., using the `curl` tool, make the same HTTP request of interest twice within a short time frame (1-2 seconds) to emulate what would be done by the replicas of a subnet, just with a smaller $n$.
* Diff the two responses to find all the items that differ in the two requests. Both the body and the headers are important to be considered here. Alternatively, identify the core information of interest and how it can be extracted as the response.
* Implement a *transformation function* that transforms responses such that they are equal on each replica based on the observed diff of the responses or the intended response.
* Determine the maximum response size of the server's response for this type of request to populate the `max_response_bytes` parameter with it. This often works well for API calls and is important to not overcharge the requesting canister in terms of cycles. The `HEAD` request type can be used to do this if responses change dynamically in response size considerably.
* Implement the request and try it out in the local SDK environment. However, note that the behaviour of the local environment does not reflect that of the IC as there is only one replica in the local environment and $n$, e.g., $n=13$, replicas on an IC subnet.

:::tip Pro tip

Do not forget to consider response headers when identifying the variable parts of your response because headers sometimes contain variable items such as timestamps which may prevent the responses from passing through consensus.

:::

### Transformation Function

The purpose of the transformation function is to transform each response $r_i$ received by a replica $i$, where the $r_i$ s received by different replicas may be different. The transformation function transforms a response $r_i$ to a transformed response $r'_i$ with the intention that all $r'_i$ s of honest replicas be equal in order to be able to agree on the response as part of IC consensus. The transformation function must be provided by the canister programmer and is exposed by the canister as a query. An arbitrary number of transformation functions can be defined by a canister. When making an `http_request` call to the management canister, a transformation function can be optionally provided as input. Depending on the purpose of the HTTP request being made, there are different approaches to writing the transformation function:
* Extract only the information item(s) of interest. This can, e.g., be a list of pairs each comprising a date and an asset price to be forwarded to the canister, or a single asset price. This approach makes sense if the full HTTP response is not required in the canister, but only specific information items are.
* Remove all variable parts of the request individually and retain all the remaining parts. The canister then extracts the relevant information. This may be useful when the canister still needs the structure of the HTTP request and the headers.

We recommend to go with the first approach whenever possible as it has multiple advantages. Calls to pricing and many other APIs should mostly work with the first approach.
* The resulting response is smaller in size.
* The transformation function may be faster to compute, that is, the function (query) is executing with fewer CPU cycles. Note that queries on the IC are for free currently, but are likely to be charged for in the future.
* The transformation function is often easier to implement, e.g., through a simple or few simple Json operations on the response body.

### Errors and Debugging

There are a number of error cases that can happen when using this feature. The most important ones are listed next.
* *SysFatal - Url needs to specify https scheme:* The feature currently only allows for HTTPS connections and using plain HTTP leads to an error.
* *SysFatal - Timeout expired:* Requests are timed out if not fulfilled within the timeout period. One important instance when this happens is when there are not sufficiently many equal responses to achieve consensus. This happens, for example, when the transformation function is not written accurately to account for all variable parts of responses.
* *SysTransient - Failed to connect: error trying to connect: tcp connect error: Connection refused (os error 111):* This error indicates that a TCP connection could not be established with the other server.
* *CanisterReject - http_request request sent with 0 cycles, but ... cycles are required:* At least the required amount of cycles need to be sent with the request in order for it to get fulfilled by the subnet.
* *CanisterReject - max_response_bytes expected to be in the range [0..2097152], got ...:* This error indicates that the network response received from the server was too large. This happens if the response size is underestimated and the `max_response_bytes` value set too low.
* *SysFatal - Transformed http response exceeds limit: 2045952:* This error indicates that the limit for the transformed response size was reached. This is currently a hard response size limit of the HTTPS outcalls functionality. Note that the response size is computed based on response body and headers.

Developers new to the feature are likely to run into certain problems in the beginning, materializing in one of the following errors. We list the issues we think are the most prominent ones when starting with this feature.
* If a specific type of canister HTTP request works in the local dfx environment, it may still not work on the IC because the local environment runs $1$ replica, whereas the IC runs $n=13$ replicas on the regular application subnets. Problems here are to be expected when developing such calls, particularly when a developer has not yet gained the necessary experience of working with the feature. The main issues to be expected here are with the lack of or problems with the transformation function. Note that this difference between the dfx environment and a deployment on the IC is not going to change any time soon as it results from the way the dfx environment works: It runs a single replica locally, with all the pros and cons during the engineering process.
* Receiving a timeout: If the requests returned by the HTTP server are not &ldquo;similar&rdquo; as required by the feature, this is most likely caused by an error in the transformation function, i.e., the transformed responses are still not equal on sufficiently many honest replicas in order to allow for consensus and thus no response is added to an IC block. Eventually, a timeout removes all artifacts related to this HTTP outcall. This issue is best debugged by diffing multiple requests made to the service and ensuring the transformation function does not retain any of the variable parts in the transformation result.
* Requests consume too many cycles: Canister HTTPS outcalls are charged cycles, but if requests with rather small responses frequently cost very large amounts of cycles, the likely cause is that the `max_response_size` parameter is not set in the request. In this case the system assumes and charges for the maximum response size which is $2$ MB. Always set this parameter to a value as close as possible to the actual maximum expected response size, and make sure it is at least as large and not smaller. The `max_response_size` parameter comprises both the body and the headers and refers to the network response from the server and not the final response to the canister.

### Pricing

Like most features of the IC, the canister HTTP outcalls feature is charged for when being used. The current pricing is defined to charge a base fee of $400$M cycles for an HTTP request in addition to $100$K cycles per request byte and per `max_response_bytes` byte. Because of the per-request fixed cost and the overhead of HTTP requests, e.g., due to headers, it is advantageous from a cost perspective to make fewer requests with larger responses to retrieve the same information as with a larger number of smaller requests, if this is feasible from an application perspective. The cycles provided with the call must be sufficient for covering the cost of the request, excessive cycles are returned to the caller.

The current pricing is defined to be rather conservative (expensive) and prices may change in the future with the introduction of an update of the pricing model. However, note that an HTTP outcall with a small response, like the ones used for querying financial APIs, only costs fractions of a USD cent, which is substantially cheaper than fees charged for a call by oracles on most blockchains.

## Community Contributions

If you are experiencing issues that are not yet described in the documentation or have other suggestions for improvement of the documentation that may help engineers use the feature more effectively, please let us know about it in the [forum topic](https://forum.dfinity.org/t/enable-canisters-to-make-http-s-requests/9670) for canister HTTP requests.

-->