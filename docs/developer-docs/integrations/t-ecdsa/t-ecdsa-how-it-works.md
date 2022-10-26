# 技術概要 - 仕組み

IC 上の閾値 ECDSA の概要を俯瞰して説明します。このセクションの情報の一部は、本機能を使用するために必要なものではありませんが、技術的な背景情報を得るために、技術志向の読者にとって興味深いものであるかもしれません。本 IC は、Groth と Shoup の[Eurocrypt 2022 paper](https://eprint.iacr.org/2021/1330) に記載された閾値 ECDSA プロトコルを実装しています。Groth と Shoup は、この分散署名プロトコルの包括的な [design and analysis](https://eprint.iacr.org/2022/506) も発表しています。

IC 上の閾値 ECDSA 実装は、抽象化すると複数のプロトコルを備えています：
- *鍵生成*：指定されたサブネットで実行され、このサブネットのレプリカ上で秘密鍵シェアとして分配される新規の閾値 ECDSA 秘密鍵を生成します。
- *鍵の re-sharing（再共有)*：このプロトコルはソースサブネットからターゲットサブネットに ECDSA （秘密）鍵を re-sharing します。その結果、ターゲットサブネットのレプリカ上で同じ閾値 ECDSA （秘密）鍵の別のランダムな秘密鍵シェアが使用されて秘密分散されることになります（ソースサブネットが使用する秘密鍵シェアとは異なる数のレプリカ上に秘密分散される可能性があります）。
- *事前署名の計算、署名*：これらのプロトコルは秘密分散された（秘密）鍵で署名を計算します。事前署名を計算するための*プロトコル*、すなわち実際の署名プロトコルで使用される（スカラー）4倍算は、事前署名を計算するために署名要求に対して非同期的に実行されます。この事前計算プロトコルは、閾値 ECDSA 署名を作成する手順の大部分を計算します。*署名プロトコル*は、Canister の署名要求によって起動されます。署名プロトコルは、閾値 ECDSA 署名を効率的に計算するために、事前に計算された（スカラー）4倍算を消費します。
- *公開鍵の取得*：Canister の公開鍵を取得することができます。その中には Canister が提供する導出パスに基づく BIP-32 ライクな鍵導出も含まれます。

秘密鍵は、生成時、サブネット間の秘密鍵シェアの re-sharing 時、署名の計算時など、使用するすべての期間で、秘密鍵シェアされた形でのみ存在し、再構成（復元）された形には決してならないことに注意することが重要です。

（秘密）鍵管理、つまり初期鍵生成と鍵の re-sharing を行うために、様々な NNS のプロポーザルが実装されてきました。これらのプロポーザルは、どのサブネットで ECDSA マスター（秘密）鍵を生成するか、どのサブネットに（秘密）鍵を re-sharing すればより利用しやすくなるか、どのサブネットで署名要求への応答を可能にするかを定義するために使用されています。

## ECDSA 鍵

IC で選択されたサブネットの鍵生成プロトコルで生成された、閾値 ECDSA の*マスター（秘密）鍵*と呼ばれる鍵が ECDSA 対応サブネットに保持されています。マスター（秘密）鍵は、Canister ECDSA 鍵の元となる（秘密）鍵です。つまり、ある楕円曲線に対するマスター（秘密）鍵が1つあれば、IC 上の各 Canister の ECDSA 鍵（* Canister ルート鍵*）を、Canister の Principal を入力として BIP-32 鍵導出機能の拡張を用いて導出することが可能です。鍵の導出は、署名および公開鍵検索 API の一部として、プロトコルによって内部で実行されます。マスター鍵から Canister のルート鍵の導出については、下図のレベル0鍵導出を参照してください。

 Canister のルート鍵から、BIP-32 鍵導出機構の後方互換性のある拡張を使用して、Canister の ECDSA 鍵を無制限に導出することができます。この拡張により、鍵導出機能の各レベルの入力として、32ビット整数だけでなく、任意の長さのバイト配列も使用することができるようになりました。下図のレベル1以上は、Canister のルート鍵に基づき、さらにCanister の鍵を派生させることを表しています。

Canister のルート鍵からさらに ECDSA 鍵を派生させることは、特定のユースケースを容易にするために、IC の関与なしに行うことも可能です。

![Threshold ECDSA Key derivation](../_attachments/key_derivation.png)

閾値 ECDSAのマスター（秘密）鍵は、閾値 ECDSA API では常に*鍵識別子*を通じて参照されます（運用開始を管理する NNS のプロポーザルでも同様です）。鍵識別子は楕円曲線名と識別子からなり、このふたつのタプル `(secp256k1, key_1)` は鍵識別子の例です。例えば、署名の計算時に秘密鍵シェアを選択するときや、対応する識別子を持つ鍵を保持する ECDSA 対応サブネットとの間で API コールと応答の XNet ルーティングの実装のときに、これらの鍵識別子は正しい鍵を参照するためにシステムによって使用されます。

## 開発

次に、現在提供されている Chromium（Beta）リリースと2022年後半に予定されている GA リリースのデプロイの概要を説明します。

### Chromium リリース

現在、Chromium リリースの一部として、楕円曲線 `secp256k1` のテスト（秘密）鍵だけが、複製係数13（ノード）でひとつのサブネットに配備されています。この（秘密）鍵は GA リリース後しばらくして NNS のプロポーザルにより削除される可能性があるため、価値のあるものには使用せず、開発およびテスト目的にのみ使用すべきです。具体的には、例えば、テスト（秘密）鍵が削除されたときに Bitcoin が失われるため本物の Bitcoin をテスト（秘密）鍵を使って保有にしなように強く助言します。Bitcoin のスマートコントラクトの開発を促進し、GA リリースの準備としてテストネット Bitcoin を保持することをテスト（秘密）鍵はむしろ目的としています。テスト（秘密）鍵は、API コールで参照するために `(secp256k1, test_key_1)` という ID を持っています。

### GA リリース（2022年後半予定）

来たるべき GA リリースでは、楕円曲線「secp256k1」の単一閾値 ECDSA （秘密）鍵を IC 上に配置する予定です。この鍵は、高い複製率 (&ge;34) を持つふたつの異なるサブネットで秘密分散された形で保持されます。（秘密）鍵は最初に NNS で生成され、そこで保管された後、re-sharing プロトコルを使用して新しい34ノードのサブネットに re-sharing されます。後者のサブネットは、この（秘密）鍵のアクティブな署名サブネットとして動作するようにアクティブ化されます。NNS は（秘密）鍵の可用性を高めるため、バックアップ用として秘密分散された形で鍵を保持しますが、署名要求には応じません。万が一、サブネットのいずれかが復旧不可能なほど破壊された場合、（秘密）鍵の複製を行うことで、災害時に別のサブネットに鍵を再共有することが可能となり、鍵の可用性を高めることができます。

### その他の側面

現在の Chromium デプロイメントでは、テスト（秘密）鍵、将来の GA リリースでは本番（秘密）鍵の両方とも、 閾値 ECDSA API へのリクエストは常に XNet を通じてリクエストされます。これは ECDSA 対応サブネットがユーザーの Canister をホストしていないためで、呼び出し側の Canister のサブネットから閾値 ECDSA 対応サブネットへの Xnet 通信とレスポンスを返すために数秒間の追加レイテンシーが発生します。

将来的には、さらなる楕円曲線とそれに対応するマスター（秘密）鍵のサポートが追加される可能性があります。楕円曲線 `secp256r1` は分散型認証局 (CA) のようなユースケースをサポートするために興味深いものであり、前述のようなユースケースを促進する最有力な追加候補のひとつです。

## API

次に、閾値 ECDSA の API の概要について説明します。正式な仕様については、[Internet Computer Interface Specification](../../../references/ic-interface-spec.md#ic-ecdsa_public_key) の該当箇所を参照してください。この API は、閾値 ECDSA 公開鍵を取得するための `ecdsa_public_key` と、秘密分散された閾値 ECDSA （秘密）鍵を持つサブネットから閾値 ECDSA 署名の計算を要求するための `create_ecdsa_signature` という二つのメソッドから構成されています。

各 API コールは、楕円曲線と鍵識別子の2つの識別子からなる閾値 ECDSA マスター（秘密）鍵を参照します。派生パスは、Canister のルート鍵より下位の鍵を派生階層で参照するために使用されます。マスター（秘密）鍵から Canister のルート鍵への鍵の導出は、API で内在的に行われます。

- `ecdsa_public_key`: このメソッドは、指定した Canister の ECDSA 公開鍵を SEC1 で暗号化したものを、指定した導出パスで返します。`canister_id` が指定されていない場合、デフォルトで呼び出し元の Canister ID が使用されます。`derivation_path` は可変長のバイト文字列のベクトルです。`key_id` は楕円曲線と名前の両方を指定する構造体です。特定の `key_id` を利用できるかどうかは実装に依存します。<br/>
楕円曲線 `secp256k1` の場合、公開鍵は BIP32 の一般化を用いて導かれる(ia.cr/2021/1330 fの付録Dを参照)。BIP-0032 互換の公開鍵を導出するためには、`derivation_path` の各バイト列 (blob) は、2<sup>31</sup>未満の符号なし整数を4バイトのビッグエンディアンで符号化したものでなければなりません。<br/>
SEC1 で圧縮された ECDSA の `public_key` と、その子鍵を決定論的に導出するための `chain_code` からなる拡張 `public_key` が返されます。
このコールでは、ECDSA 機能が有効であること、および `canister_id` が Canister ID の要件を満たしていることが必要です。そうでない場合は拒否されます。
- `sign_with_ecdsa`: このメソッドは、与えられた `message_hash` の新しい ECDSA 署名を返します。これは、派生した ECDSA 公開鍵に対して個別に検証することができます。この公開鍵は、呼び出し元の `canister_id` と、ここで使用したものと同じ `derivation_path` と `key_id` を指定して `ecdsa_public_key` を呼び出すことで取得できます。<br/>
署名は、SEC1 エンコーディングの2つの値 `r` と `s` を連結したものとして暗号化されます。楕円曲線 `secp256k1` の場合、これは 32バイトのビッグエンディアンに相当します。<br/> 
この呼び出しにより、ECDSA は `r` と `s` を暗号化する必要があります。
この呼び出しは、ECDSA 機能が有効で、呼び出し元が Canister であり、 `message_hash` が 32バイトの長さであることを必要とします。そうでない場合は拒否されます。

## 環境

IC 上での Canister 開発のライフサイクルを通じて開発者を支援するため、ローカルでの開発・テスト用の SDK と、 Canister のプレプロダクションのテストやプロダクション時の運用のための IC の両方でこの機能を利用することができます。

### SDK

Canister の開発は、通常、開発者のローカル環境で行われ、SDK の利用が促進されます。今回、SDK を拡張し閾値 ECDSA のマネージメント Canister API をローカル Canister の実行環境で利用できるようにしました。このため、開発・テスト用にローカルで閾値 ECDSA の API を使用した Canister を実行することができます。本機能は SDK で常に有効化されているため、API を利用するためにユーザーが追加で操作する必要はありません。

SDK 環境のレプリカが最初に起動されると、新しい ECDSA （秘密）鍵が生成されます。この鍵は不揮発性メモリに保存され、レプリカを再起動するたびに変更されることはありません。

技術的に関心のある読者のために、SDK はメインネットと全く同じ閾値 ECDSA の実装を使用していますが、単一のレプリカしか実行していないことに注意してください。したがって、このプロトコルは単一のレプリカという特殊なケースで動作していることになり、（秘密）鍵の生成や署名などのオーバーヘッドがわずかに発生するだけなので、SDK 環境の性能に目立った影響を与えることなく、SDK でデフォルトで有効のままにしておくことができます。また、ローカル SDK 環境での署名のスループットとレイテンシーは、IC 上のスループットとレイテンシーを代表するものではないことに注意してください。

### Internet Computer

IC のどのサブネット上の、どの Canister も、マネージメント Canister によって公開された閾値 ECDSA API を呼び出すことができます。呼び出しは、API コールで参照される（秘密）鍵を保持する ECDSA 対応サブネットに XNet 通信を介してルーティングされます（初期の Chromium リリースでは、テスト用の（秘密）鍵を保持するこのような署名サブネットはひとつしか存在しません）。このテスト（秘密）鍵は複製率が 13のサブネットでホストされており、将来的に削除される可能性があることに注意してください。したがって、価値のあるものには使用せず、開発およびテスト目的のみに使用する必要があります。主な意図する目的は、Bitcoin テストネットを使用し Bitcoin 対応 Dapps の開発とテストを容易にすることです。

2022年後半、この機能の一般公開（GA）リリースの一部として、`secp256k1` 楕円曲線上の本番 ECDSA （秘密）鍵が配置され、Bitcoin メインネットとの統合やその他の有用なユースケースで使用される予定です。

<!--
# Technology Overview — How It Works

We give a high-level outline of threshold ECDSA on the IC. Some of the information in this section is not required to use the feature, but may be of interest to the technically inclined reader for obtaining background information on the technology. The IC implements the threshold ECDSA protocol by Groth and Shoup as described in their [Eurocrypt 2022 paper](https://eprint.iacr.org/2021/1330). Groth and Shoup have also published a comprehensive [design and analysis](https://eprint.iacr.org/2022/506) of this distributed signing protocol.

At a high level, the threshold ECDSA implementation on the IC features multiple protocols:
-   *Key generation:* This protocol is executed on a specified subnet; it generates a new threshold ECDSA key such that the private key is secret shared over the replicas of this subnet.
-   *Key re-sharing:* This protocol re-shares an ECDSA key from a source subnet to a target subnet. It results in the same key being secret shared over the replicas of the target subnet using a different random secret sharing (potentially over a different number of replicas than the sharing in the source subnet uses).
-   *Computing pre signatures, signing:* These protocols compute signatures with the secret-shared private key. A *protocol for computing pre signatures*, i.e., quadruples that are used in the actual signing protocol, is run asynchronously to signing requests to precompute pre signatures. This precomputation protocol computes the vast majority of the steps of creating threshold ECDSA signatures. A *signing protocol* is triggered by a signing request of a canister. A signing protocol consumes one precomputed quadruple to efficiently compute a threshold ECDSA signature.
-   *Public key retrieval:* Allows for retrieving a public key of a canister, including potential BIP-32-like key derivation based on a canister-provided derivation path.

It is crucial to note that the private key never exists in reconstructed, but only in secret-shared, form during its whole lifetime, be it during its generation, the re-sharing of the key from one subnet to another, and computing signatures.

Various NNS proposals have been implemented to perform key management, i.e., initial key generation and key re-sharing. Those proposals are used to define on which subnet to generate an ECDSA master key, to which subnet to re-share the key to have it available for better availability, and which subnet to enable for answering signing requests.

## ECDSA Keys

ECDSA-enabled subnets hold what we call threshold ECDSA *master keys*, generated with the key generation protocol on selected subnets of the IC. A master ECDSA key is a key from which canister ECDSA keys can be derived. I.e., a single master key for a given elliptic curve suffices to allow for the derivation of an ECDSA key for each canister on the IC (*canister root key*) using an extension of the BIP-32 key derivation mechanism with the canister's principal as input. The key derivation is executed transparently by the protocol as part of the signing and public key retrieval APIs. See the level-0 key derivation in the below Figure for the derivation of canister root keys from a master key.

From a canister root key, an unlimited number of ECDSA keys can be derived for the canister using a backward-compatible extension of the BIP-32 key derivation mechanism. The extension allows not only 32-bit integers, but arbitrary-length byte arrays, to be used as input for each level of the key derivation function. See the levels 1 and greater in the below figure illustrating the derivation of further canister keys based on the canister root key.

The derivation of further ECDSA keys from a canister root key can be done without involvement of the IC as well to facilitate certain use cases.

![Threshold ECDSA Key derivation](../_attachments/key_derivation.png)

Threshold ECDSA master keys are always referred to through *key identifiers* in the threshold ECDSA API (as well as in the NNS proposals for managing the rollout). The key identifiers comprise an elliptic curve name and an identifier, e.g., an example key identifier is the 2-tuple `(secp256k1, key_1)`. Those key identifiers are used by the system to refer to the correct key, e.g., for selecting the key share when computing a signature or in the implementation of the XNet routing of API calls and responses to/from the ECDSA-enabled subnet holding the key with the corresponding identifier.

## Deployment

We next outline the deployment for the Chromium (Beta) release available now and the GA release coming later in 2022.

### Chromium Release

Currently, as part of the Chromium release, only a test key for curve `secp256k1` is deployed on one subnet with a replication factor of 13. This key may be deleted with an according NNS proposal some time after the GA release and therefore should not be used for anything that has value, but only for development and testing purposes. More concretely, it is, for example, strongly advised against holding real bitcoin with the test key as those Bitcoin would be lost when the key gets deleted. The test key is rather intended to facilitate development of Bitcoin smart contracts and hold Testnet bitcoin as preparation for the GA release. The test key has the id `(secp256k1, test_key_1)` for referring to it in API calls.

### General Availability Release (Coming Later in 2022)

A single threshold ECDSA production key for the `secp256k1` elliptic curve will be deployed on the IC for the upcoming GA release. The key will be maintained in secret-shared form on two different subnets with high replication factor (&ge;34). The key will be initially generated on the NNS and kept there and re-shared to a new 34-node subnet using the re-sharing protocol. The latter subnet will be activated to act as the active signing subnet for this key. The NNS will hold the key in secret-shared form for backup purposes for achieving better key availability, but will not respond to signing requests. In case of the unlikely event of one of the subnets getting destroyed beyond recoverability, the approach of key replication improves key availability by allowing for the key to be re-shared to a different subnet, should this be ever required in case of a disaster.

### Further Aspects

For both the current Chromium deployment with a test key and the future GA release with a production key, requests to the threshold ECDSA API are always XNet requests because the ECDSA-enabled subnets do not host user's canisters, thus some seconds of extra latency is incurred due to Xnet communications from the calling canister's subnet to the threshold-ECDSA-enabled subnet holding the respective key and communicating the response back.

Support for further elliptic curves and additional corresponding master keys may be added in the future. The curve `secp256r1` is interesting for supporting use cases such as decentralized certification authorities (CAs) and is the premier candidate group to be added to facilitate use cases like the mentioned one.

## API

We next give an overview of the API for threshold ECDSA. For the authoritative specification, the reader is referred to the corresponding part of the [Internet Computer Interface Specification](../../../references/ic-interface-spec.md#ic-ecdsa_public_key). The API comprises two methods, `ecdsa_public_key` for retrieving threshold ECDSA public keys, and `create_ecdsa_signature` for requesting threshold ECDSA signatures to be computed from the subnet holding the secret-shared private threshold ECDSA key.

Each API call refers to a threshold ECDSA master key by virtue of a 2-part identifier comprising a curve and a key id as outlined above. Derivation paths are used to refer to keys below a canister's root key in the key derivation hierarchy. The key derivation from the master key to the canister root key is implicit in the API.

-   `ecdsa_public_key`: This method returns a SEC1-encoded ECDSA public key for the given canister using the given derivation path. If the `canister_id` is unspecified, it will default to the canister id of the caller. The `derivation_path` is a vector of variable length byte strings. The `key_id` is a struct specifying both a curve and a name. The availability of a particular `key_id` depends on implementation.<br/>
For curve `secp256k1`, the public key is derived using a generalization of BIP32 (see ia.cr/2021/1330, Appendix D). To derive (non-hardened) BIP-0032-compatible public keys, each byte string (blob) in the `derivation_path` must be a 4-byte big-endian encoding of an unsigned integer less than 2<sup>31</sup>.<br/>
The return result is an extended public key consisting of an ECDSA `public_key`, encoded in SEC1 compressed form, and a `chain_code`, which can be used to deterministically derive child keys of the `public_key`.\
This call requires that the ECDSA feature is enabled, and the `canister_id` meets the requirement of a canister id. Otherwise it will be rejected.
-   `sign_with_ecdsa`: This method returns a new ECDSA signature of the given `message_hash` that can be separately verified against a derived ECDSA public key. This public key can be obtained by calling `ecdsa_public_key` with the caller's `canister_id`, and the same `derivation_path` and `key_id` used here.<br/>
The signatures are encoded as the concatenation of the SEC1 encodings of the two values `r` and `s`. For curve `secp256k1`, this corresponds to 32-byte big-endian encoding.<br/>
This call requires that the ECDSA feature is enabled, the caller is a canister, and `message_hash` is 32 bytes long. Otherwise it will be rejected.

## Environments

In order to facilitate developers throughout the canister development life cycle on the IC, the feature is available in both the SDK for local development and testing as well as on the IC for pre-production testing and production operation of canisters.

### SDK

The development of canisters is typically done in the developer's local environment, facilitated by use of the SDK. The SDK has been extended such that the management canister API for threshold ECDSA is available in the local canister execution environment. Thus, canisters using the threshold ECDSA API can be run locally for development and testing purposes. The feature is always enabled in the SDK, so no further user action is required in order to make use of the API.

When the replica of the SDK environment is first started up, a new ECDSA key is generated. This key is then stored in non-volatile memory so that it does not change with every restart of the replica.

For the technically interested readers we want to note that the SDK uses the exact same implementation of threshold ECDSA as mainnet, but only runs a single replica. Thus, the protocol is operating with the special case of a single replica, which means it degenerates to a special case and incurs only little overhead, e.g., for key generation and signing, and can thus remain enabled by default in the SDK without noticeably affecting performance of the SDK environment. Also note that the signing throughput and latency in the local SDK environment is not representative for the throughput and latency on the IC.

### Internet Computer

Any canister on any subnet of the IC can call the threshold ECDSA API exposed by the management canister. The calls are routed via XNet communication to the ECDSA-enabled subnet that holds the key referred to in the API call (only one such signing subnet holding a test key exists for testing purposes in the initial Chromium release). Note that this test key is hosted on a subnet with a replication factor of only 13 and may be deleted in the future, thus it should not be used for anything of value, but rather solely for development and testing purposes. The main intended purpose is to facilitate the development and testing of Bitcoin-enabled dApps using Bitcoin testnet.

Later in 2022, as part of the General Availability (GA) release of the feature, a production ECDSA key on the `secp256k1` elliptic curve will be deployed to be used for integration with Bitcoin mainnet and other use cases of interest.

-->