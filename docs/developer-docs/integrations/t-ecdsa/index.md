# 閾値 ECDSA

Internet Computer は、革新的な閾値 ECDSA プロトコルを実装しています。このプロトコルでは、ECDSA 秘密鍵は秘密分散されて複数のノード、つまり IC 上の閾値 ECDSA 対応サブネットのレプリカによってに保持され、秘密鍵を再構成・復元することなく、分散された秘密鍵を用いて署名されます。このためサブネットの各レプリカは、それ自体では何も情報を提供しない秘密鍵シェアを保持しています。少なくとも3分の1のレプリカがそれぞれの秘密鍵シェアを提供して初めて閾値署名をすることが出来ます。

Internet Computer の任意のサブネット上の各 Canister は、固有の ECDSA 鍵ペアを制御し、対応する秘密鍵による署名の計算を（ECDSA サブネットに）要求することができます。署名は、その資格のある Canister、すなわち ECDSA 鍵の正当な所持者にのみ発行されます。各 Canister は、固有の ECDSA 鍵に対してのみしか署名を取得することができません。Canister は自分では ECDSA 秘密鍵や秘密鍵シェアを持たず、IC の特定の閾値 ECDSA 対応サブネットに委任していることに注意してください。閾値暗号は、ブロックチェーンのトラストモデルにおいて、従来の暗号だけでは実現不可能な機能を実現するのに役立ちます。

秘密鍵を安全に保管し、適格なエンティティの要求に応じて、それらに対してのみ署名を発行するハードウェアセキュリティモジュール（HSM）のオンチェーン版としてブロックチェーン上の閾値 ECDSA 実装は見なすことができます。

閾値 ECDSA が利用できることで、例えば、以下のような多数の重要なユースケースが可能になります：
-   Bitcoin をネイティブに Canister に保持すること。
-   Ethereum との統合、例えば Ethereum の ERC-20 トークンを IC に取り込んだり、Ethereum のトランザクションに署名したりすること。
-   トランザクションへの署名に  ECDSA を署名方式として使用する他のブロックチェーンとの統合すること。
-   分散型認証局（CA）を実現し、閾値 ECDSA を用いて証明書を発行すること（そのためには現在実装されている楕円曲線 `secp256k1` とは異なる楕円曲線、`secp256r1` が必要となります）。

これらは、私たちの閾値 ECDSA プロトコルの使用のほんの一例に過ぎません。クリエイティブな技術者や起業家は、さらに多くのエキサイティングなユースケースを思いつくことでしょう。

## さらに学ぶには
もしあなたが閾値 ECDSA についてもっと知りたいなら、[how it works](./t-ecdsa-how-it-works.md) ページをチェックしてください。さらに深く掘り下げるなら、Groth と Shoup の [Eurocrypt 2022 paper](https://eprint.iacr.org/2021/1330) を参照してください。

## ビルドしたい方は
`threshold-ecdsa` のサンプルコードは [examples repository](https://github.com/dfinity/examples) の [`motoko`](https://github.com/dfinity/examples/tree/master/motoko/threshold-ecdsa) または [`rust`](https://github.com/dfinity/examples/tree/master/rust/threshold-ecdsa) サブディレクトリで提供されています。対応するコード・ウォークスルーは [samples documentation](../../../samples/t-ecdsa-sample.md) で見ることができます。

<!--
# Threshold ECDSA

The Internet Computer implements a novel threshold ECDSA protocol. In this protocol, the private ECDSA key is held in a secret-shared manner by multiple parties, namely the replicas of a threshold-ECDSA-enabled subnet on the IC, and signatures are computed using those secret shares without the private key ever being reconstructed. Each replica of such subnet holds a key share that provides no information about the key on its own. At least one third of the replicas are required to generate a threshold signature using their respective key shares.

Each canister on any subnet of the Internet Computer has control over a unique ECDSA key pair and can request signatures with the corresponding private key to be computed. A signature is only issued to the eligible canister, i.e., the legitimate holder of the ECDSA key. Each canister can obtain signatures only for its ECDSA keys. Note that canisters do not hold any private ECDSA keys or key shares themselves, but delegate this to specific threshold-ECDSA-enabled subnets of the IC. Threshold cryptography can help enable functionality in the trust model of a blockchain that would be impossible to achieve with conventional cryptography alone.

A threshold ECDSA implementation on a blockchain can be viewed as the on-chain pendant to a hardware security module (HSM) that stores private keys securely and issues signatures on request of the eligible entities, and only to those.

The availability of threshold ECDSA allows for a multitude of important use cases, as for example:
-   Canisters natively holding Bitcoin;
-   Integration with Ethereum, e.g., getting the ERC-20 tokens of Ethereum into the IC or signing Ethereum transactions;
-   Integrations with other blockchains that use ECDSA as signature scheme for signing transactions.
-   Realizing a decentralized certification authority (CA), where certificates are issued using threshold ECDSA (this requires a different elliptic curve to the currently implemented curve `secp256k1`, namely `secp256r1`).

Those are only a few examples for use cases of our threshold ECDSA protocol. Creative engineers and entrepreneurs will likely come up with a large number of further exciting use cases.

## Learn More
If you want to learn more about threshold ECDSA, check the [how it works](./t-ecdsa-how-it-works.md) page to see more. If you want to take an even deeper dive see Groth and Shoup's [Eurocrypt 2022 paper](https://eprint.iacr.org/2021/1330).

## Build More
Sample code for `threshold-ecdsa` is provided in the [examples repository](https://github.com/dfinity/examples), under either [`motoko`](https://github.com/dfinity/examples/tree/master/motoko/threshold-ecdsa) or [`rust`](https://github.com/dfinity/examples/tree/master/rust/threshold-ecdsa) sub-directories. You can find the corresponding code-walkthrough in the [samples documentation](../../../samples/t-ecdsa-sample.md)

-->