# Large web assembly モジュール

現在、IC にインストールできるプログラムのサイズは 2MB に制限されています。
2MB を（わずかに）超える WebAssembly モジュールについては、アップロード前に gzip によるファイル圧縮を行い、IC がファイルを解凍し、含まれる WebAssembly モジュールをインストールすることにより、IC へのインストールが可能です。

## gzip 圧縮された WebAssembly モジュールのインストール

WebAssembly モジュールは `gzip` で圧縮され、`dfx install` でアップロードされます。既存の Canister にモジュールをアップロードする際には、`--mode reinstall` または `--mode upgrade` を追加する必要があるかもしれません。

``` bash
$ gzip my-canister.wasm
$ dfx canister install my-canister --wasm my-canister.wasm.gz
```

圧縮は現在、`dfx deploy` ではサポートされていません。

<!--
# Large web assembly modules

The size of programs that can be installed on the IC is currently limited to 2MB.
WebAssembly modules that are (slighly) larger than 2MB can still be installed on the IC by using gzip file compression before uploading; the IC will then decompress the file and install the contained WebAssembly module.

## Installing a gzip-compressed WebAssembly module

The WebAssembly module is compressed using `gzip` and then uploaded by `dfx install`, you may need to add `--mode reinstall` or `--mode upgrade` when uploading the module to an existing canister.

``` bash
$ gzip my-canister.wasm
$ dfx canister install my-canister --wasm my-canister.wasm.gz
```

Compression is currently not supported by `dfx deploy`.

-->
