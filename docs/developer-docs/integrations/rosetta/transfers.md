# スタンダードトークンを送金する

本ドキュメントは、Rosetta Construction API を使用した ICP 送金の方法について説明します。トランザクションの流れの概要については、[Construction API 概要](https://www.rosetta-api.org/docs/construction_api_introduction.html) を参照してください。

## 送金手順

送金額 `T` をアドレス `A` からアドレス `B` に転送するトランザクションは、3つのオペレーションを含むことになります：

-   `TRANSACTION` タイプのオペレーションがアドレス `A` に `-T` の値で適用されます。

-   `TRANSACTION` オペレーションタイプがアドレス `B` に金額 `T` で適用されます。

-   `FEE` タイプの操作が `/construction/metadata` エンドポイントによって提案された金額でアドレス `A` に適用されます（[ConstructionMetadataResponse](https://www.rosetta-api.org/docs/models/ConstructionMetadataResponse.html) タイプの `suggested_fee` フィールドを参照してください）。

トランザクション内の操作の順序は関係ありません。

1回の取引で複数回の送金はできません。そのような取引の結果は明示されません。

前提条件：

-   アドレス `A` は少なくとも `T` + `suggested_fee` の ICP を保持しています。

-   アドレス `A` は、サブアカウントがトランザクションの署名に使用した公開鍵に由来する Principal になっています。

-   `FEE` オペレーションで指定する金額は `suggested_fee` と絶対値で等しくなっています。

### オプションのメタデータフィールド

Rosetta ノードは [ConstructionPayloadRequest](https://www.rosetta-api.org/docs/models/ConstructionPayloadsRequest.html) で、以下のオプションのメタデータフィールドを認識します：

-   `memo` はトランザクションに関連付けられた任意の64ビット符号なし整数です。これを使用して、データをトランザクションに関連付けることができます。例えば、データベースの行のキーに memo を設定することができます。`memo` フィールドに有効な値は `0` から `264 - 1` (`18446744073709551615`) までの範囲です。

-   `ingress_start`、`ingress_end` および `created_at_time` は 64 ビットの符号なし整数で、UTC タイムゾーンの UNIX エポックからナノ秒経ったかを表現しています。これらのフィールドを使って、あらかじめトランザクションを構築して署名しておき、後で署名付きトランザクションを提出することができます。署名されたトランザクションは `created_at_time`（デフォルトでは `/construction/payloads` エンドポイントを呼び出した時点）から 24 時間以内に送信することが可能です。

### 例

以下は、アドレス `bdc4ee05d42cd0669786899f256c8fd7217fa71177bd1fa7b9534f568680a938` からアドレス `b64ec6f964d8597afa06d4209dbce2b2df9fe722e86aeda2351bd95500cf15f8` まで 1 ICP を譲渡した取引のサンプルです。

``` json
{
  "network_identifier": {
    "blockchain": "Internet Computer",
    "network": "00000000000000020101"
  },
  "operations": [
    {
      "operation_identifier": {
        "index": 0
      },
      "type": "TRANSACTION",
      "account": {
        "address": "bdc4ee05d42cd0669786899f256c8fd7217fa71177bd1fa7b9534f568680a938"
      },
      "amount": {
        "value": "-100000000",
        "currency": {
          "symbol": "ICP",
          "decimals": 8
        }
      }
    },
    {
      "operation_identifier": {
        "index": 1
      },
      "type": "TRANSACTION",
      "account": {
        "address": "b64ec6f964d8597afa06d4209dbce2b2df9fe722e86aeda2351bd95500cf15f8"
      },
      "amount": {
        "value": "100000000",
        "currency": {
          "symbol": "ICP",
          "decimals": 8
        }
      }
    },
    {
      "operation_identifier": {
        "index": 2
      },
      "type": "FEE",
      "account": {
        "address": "bdc4ee05d42cd0669786899f256c8fd7217fa71177bd1fa7b9534f568680a938"
      },
      "amount": {
        "value": "-10000",
        "currency": {
          "symbol": "ICP",
          "decimals": 8
        }
      }
    }
  ],
  "public_keys": [
    {
      "hex_bytes": "97d0b490ec4097b3653878274b1d9dd00bb1316ea3df0bfdf98327ef68fade63",
      "curve_type": "edwards25519"
    }
  ]
}
```

### レスポンス

``` json
{
  "transaction_identifier": {
    "hash": "97f4a8289f96ef46d8c8fa911f13cc402e4f69b36f4dd1ddc2579bb54dba5557"
  },
  "block_index": 1043
}
```

<!--
# Regular token transfers

This document details how to transfer ICP using the Rosetta Construction API. See [Construction API Overview](https://www.rosetta-api.org/docs/construction_api_introduction.html) for a high-level overview of the transaction flow.

## Transfer operations

A transaction that transfers amount `T` from address `A` to address `B` must contain three operations:

-   An operation of type `TRANSACTION` applied to address `A` with the amount of `-T`.

-   An operation of type `TRANSACTION` applied to address `B` with the amount of `T`.

-   An operation of type `FEE` applied to address `A` with the amount suggested by the `/construction/metadata` endpoint (see `suggested_fee` field of the [ConstructionMetadataResponse](https://www.rosetta-api.org/docs/models/ConstructionMetadataResponse.html) type).

The order of operations within a transaction is irrelevant.

Multiple transfers within a single transaction are not allowed. The outcome of such a transaction is unspecified.

Preconditions:

-   Address `A` holds at least `T` + `suggested_fee` ICP.

-   Address `A` is a subaccount of the principal derived from the public key that you use to sign the transaction.

-   The amount specified in the `FEE` operation is equal in absolute value to `suggested_fee`.

### Optional metadata fields

Rosetta node recognizes the following optional metadata fields in [ConstructionPayloadRequest](https://www.rosetta-api.org/docs/models/ConstructionPayloadsRequest.html):

-   `memo` is an arbitrary 64-bit unsigned integer associated with the transaction. You can use it to associate your data with the transaction. For example, you can set the memo to a row key in a database. Valid values for the `memo` field range from `0` to `264 - 1` (`18446744073709551615`).

-   `ingress_start`, `ingress_end`, and `created_at_time` are 64-bit unsigned integers representing the number of nanoseconds passed from UNIX epoch in UTC timezone. You can use these fields to construct and sign a transaction in advance and submit the signed transaction later. You can submit a signed transaction within 24 hours starting from `created_at_time` (by default equal to the time when you invoke the `/construction/payloads` endpoint).

### Example

Below is an example of a transaction that transfers 1 ICP from address `bdc4ee05d42cd0669786899f256c8fd7217fa71177bd1fa7b9534f568680a938` to address `b64ec6f964d8597afa06d4209dbce2b2df9fe722e86aeda2351bd95500cf15f8`.

``` json
{
  "network_identifier": {
    "blockchain": "Internet Computer",
    "network": "00000000000000020101"
  },
  "operations": [
    {
      "operation_identifier": {
        "index": 0
      },
      "type": "TRANSACTION",
      "account": {
        "address": "bdc4ee05d42cd0669786899f256c8fd7217fa71177bd1fa7b9534f568680a938"
      },
      "amount": {
        "value": "-100000000",
        "currency": {
          "symbol": "ICP",
          "decimals": 8
        }
      }
    },
    {
      "operation_identifier": {
        "index": 1
      },
      "type": "TRANSACTION",
      "account": {
        "address": "b64ec6f964d8597afa06d4209dbce2b2df9fe722e86aeda2351bd95500cf15f8"
      },
      "amount": {
        "value": "100000000",
        "currency": {
          "symbol": "ICP",
          "decimals": 8
        }
      }
    },
    {
      "operation_identifier": {
        "index": 2
      },
      "type": "FEE",
      "account": {
        "address": "bdc4ee05d42cd0669786899f256c8fd7217fa71177bd1fa7b9534f568680a938"
      },
      "amount": {
        "value": "-10000",
        "currency": {
          "symbol": "ICP",
          "decimals": 8
        }
      }
    }
  ],
  "public_keys": [
    {
      "hex_bytes": "97d0b490ec4097b3653878274b1d9dd00bb1316ea3df0bfdf98327ef68fade63",
      "curve_type": "edwards25519"
    }
  ]
}
```

### Response

``` json
{
  "transaction_identifier": {
    "hash": "97f4a8289f96ef46d8c8fa911f13cc402e4f69b36f4dd1ddc2579bb54dba5557"
  },
  "block_index": 1043
}
```

-->