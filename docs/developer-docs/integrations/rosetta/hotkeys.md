# ホットキーの生成

このページでは、Neuron 管理のために使用するホットキーを生成する方法について説明します。
ホットキーを取得する方法として推奨されるのは、公開されている `ic` リポジトリと同じプロセスを使用してプログラム的に生成することです。
しかし、以下のように手動でホットキーを生成することも可能です。
このページは、期待されるデータフォーマットを示すいくつかの例とともに、順を追って説明します。

## 手動のプロセス

手動でホットキーを生成する場合、PEM ファイルを生成しそれに対応する秘密鍵と公開鍵を導出することになります。

### PEM の生成

鍵を生成するには、まずプライベートな PEM ファイルを生成することから始めます：

`openssl genpkey -algorithm ed25519 -outform PEM -out private.pem`

出力例です：
```
-----BEGIN PRIVATE KEY-----
MC4CAQAwBQYDK2VwBCIEIHAOt4HGrNdcIFhBy7N9p6iq3iRowd4NZjDZ8aaaDCcX
-----END PRIVATE KEY-----
```

そして、そこから公開用 PEM を作成します：

`openssl pkey -in private.pem -pubout > public.pem`

出力例です：

```
-----BEGIN PUBLIC KEY-----
MCowBQYDK2VwAyEA4JKtE2KNVUTo96cl202FgWv5ctwP7f1ds1O73PZ6+VE=
-----END PUBLIC KEY-----
```

## 16進数（hex）キー表現の生成

プライベート DER ファイルを作成します：

`openssl pkey -inform pem -outform der -in private.pem -out private.der`

公開用 DER ファイルを作成します：

`openssl pkey -inform pem -outform der -in private.pem -out public.der -pubout`


### 16進数（hex）表現

生成された DER ファイルは、人間が読めるように意図されていないバイナリ形式になっています。
しかし、両鍵とも 16進数（hex）表現を得ることはできます：


### プライベートキー

`xxd -p private.der`

出力例です：

```
302e020100300506032b657004220420700eb781c6acd75c205841cbb37d
a7a8aade2468c1de0d6630d9f1a69a0c2717
```

### パブリックキー

`xxd -p public.der`

```
302a300506032b6570032100e092ad13628d5544e8f7a725db4d85816bf9
72dc0fedfd5db353bbdcf67af951
```

公開鍵の最後の 32バイトだけを残しておく必要があります：

`xxd -s 12 -c 32 -p public.der`

出力例です：
```
e092ad13628d5544e8f7a725db4d85816bf972dc0fedfd5db353bbdcf67af951
```

これは、Rosetta の操作で、ホットキーを識別するために使用できる公開鍵です。


---------

## FAQ

- PEM ファイル生成時に *"Algorithm ed25519 not found”* と表示されるのはなぜですか？

MacOS に含まれる OpenSSL のバージョンは、デフォルトでは ed25519 をサポートしていません。
別のバージョンの OpenSSL を (たとえば *brew* を使って) インストールするか、Linux マシンからこのコマンドを実行する必要があるかもしれません。

<!--
# Hotkeys generation

This page will explain how to generate a hotkey for neuron management.
The recommended way to get hotkeys is to programmatically 
generate them using the same process used in the public `ic` repository.
However, it is possible to generate hotkeys manually as described
below.
This page is a step-by-step guide with some examples showing the data format to expect.

## Manual process

The manual hotkey generation process consists in generating a PEM file, then deriving the corresponding private and public keys.

### Generate PEM

To generate the keys, start with generating a private PEM file:

`openssl genpkey -algorithm ed25519 -outform PEM -out private.pem`

Example output:
```
-----BEGIN PRIVATE KEY-----
MC4CAQAwBQYDK2VwBCIEIHAOt4HGrNdcIFhBy7N9p6iq3iRowd4NZjDZ8aaaDCcX
-----END PRIVATE KEY-----
```

Then create the public PEM from it:

`openssl pkey -in private.pem -pubout > public.pem`

Example output:
```
-----BEGIN PUBLIC KEY-----
MCowBQYDK2VwAyEA4JKtE2KNVUTo96cl202FgWv5ctwP7f1ds1O73PZ6+VE=
-----END PUBLIC KEY-----
```


## Generating hex key representation

Create the private DER file:

`openssl pkey -inform pem -outform der -in private.pem -out private.der`

Create the public DER file:

`openssl pkey -inform pem -outform der -in private.pem -out public.der -pubout`


### Hex representation

The generated DER files are in a binary format not intended to be readable by humans.
But we can get a hex representation of both keys:


### Private key

`xxd -p private.der`

Example output:

```
302e020100300506032b657004220420700eb781c6acd75c205841cbb37d
a7a8aade2468c1de0d6630d9f1a69a0c2717
```

### Public key

`xxd -p public.der`

```
302a300506032b6570032100e092ad13628d5544e8f7a725db4d85816bf9
72dc0fedfd5db353bbdcf67af951
```

We need to keep only the last 32 bytes of the public key:

`xxd -s 12 -c 32 -p public.der`

Example output:

```
e092ad13628d5544e8f7a725db4d85816bf972dc0fedfd5db353bbdcf67af951
```

This is the public key you can use in Rosetta operations to identify your hotkey!


---------

## FAQ

- Why do I get *"Algorithm ed25519 not found"* while generating the PEM file?

The version of OpenSSL included in MacOS doesn't support ed25519 by default. 
You may have to install another version of OpenSSL (for example through *brew*), or run the command from a Linux machine.

-->

