# 再現性の高い Canister のビルド

コンセンサスプロトコルのおかげで、 Internet Computer は常に Canister のコードを正しく実行することができます。しかし、これは Canister の *正しい* コードが実行されているという意味ではありません。誰かが開発した Canister を使用する場合、 Canister に重要な決定（例えば、ICP を Canister に送るなど）をさせる前に、その Canister が本当に意図したコードを実行しているか確認したいと思うかもしれません。これを確認するためには、ふたつの質問に答えることで検証できます。

1.   Canister のために実行されている WebAssembly（Wasm）コードはどれですか？

2.   Canister は通常、Motoko や Rust などの高級言語で書かれており、Wasm で直接書かれているわけではありません。よって、実行されている Wasm は本当にソースコードと称するものをコンパイルした結果なのか？

この後は、これらの質問にお答えします。最初の質問に対しては、 Internet Computer がどのように Canister コードに関する情報を提供するかを見ていきます。2番目の質問に答えられるように、 Canister の作者はソースから信頼できる、Wasm コードの[再現可能なビルド](https://reproducible-builds.org/docs/definition/)を保証しなければなりません。このようなビルドを使えば、誰でも Canister の作者と同じ手順を踏んで、まったく同じ Wasm コードを得ることができ、それを Internet Computer 上の Canister で実行されている Wasm コードと比較することができます。

再現性のあるビルドの話題に慣れているせっかちな読者は、そのまま[まとめ](#repro-build-summary)に飛べばよいでしょう。

##  Internet Computer が実行している Canister コードを調べる

 Internet Computer では、任意の Canister の Wasm コードにアクセスすることはできません。これは、開発者が一部のコードを非公開にしたい場合があるため、設計上の決定です。ただし、 Internet Computer では、Canister の Wasm コードの SHA-256 にアクセスすることができます。

このハッシュを取得するには、まず、コードを確認したい Internet Computer Canister の Principal を記録する必要があります。たとえば、Prinsipal が `rdmx6-jaaaa-aaa-aaadq-cai` である Internet Identity Canister のコードに関心があるとします。このサービスにアクセスする最も簡単な方法は、ターミナルから [`dfx`](https://internetcomputer.org/developers/) ツールを使用することです。ターミナルを開いて、以下を実行してください。

    $ dfx canister --network ic info rdmx6-jaaaa-aaaaa-aaadq-cai
    Controllers: r7inp-6aaaa-aaaaa-aaabq-cai
    Module hash: 0x2d95e90de5d7de11f25ac256690aff44c6685a1570b1becdf6e50192e983e103

古いバージョンの `dfx` を使用している場合は、有効な `dfx.json` ファイルを含むディレクトリからこのコマンドを実行する必要があります。もし、そのようなディレクトリがなければ、`dfx new` を使って作成することができます。

ここで、Internet Computer は、`rdmx6-jaaaa-aaa-aadq-cai` Canister （これはたまたま Internet Identity  Canister である）のWasm モジュールのハッシュが`0x2d95e90de5d7de11f25ac256690aff44c6685a1570b1becdf6e50192e983e103` だと教えてくれてます。

上記のチェックで、 Canister の Wasm モジュールの*現在の*ハッシュがわかりますが、 Internet Computer の Canister の*コントローラー*は、いつでもコードを変更できます（たとえば、 Canister をアップグレードする場合など）。しかし、コントローラーのリストが空だったり、コントローラーが[ブラックホール](https://github.com/ninegua/ic-blackhole) Canister だけだったりすると、誰もコードを変更する権限を持っていないので、その Canister はイミュータブルであることがわかります。

このハッシュを手に入れたら、次にそれが与えられたソースコードに対応しているかどうかをチェックすることができます。これは、そのコードのビルドプロセスが再現可能である場合にのみ機能します。

## 再現性のあるビルド

Canister の作成者として、ユーザーがあなたのビルドを再現できるようにするために、提供しなければならないものがいくつかあります：

-   Canister の Wasm モジュールを作成したときの同じソースコード。

-   ビルド環境を再構築する方法の説明。

-   ソースコードから Wasm を構築するプロセスを繰り返す方法に関する説明。ここで重要なのは、このプロセスが決定論的であり、まったく同じ Wasm になることを保証する必要があることです。また、Wasm がソースコードの忠実な翻訳であり、悪意のあるビルドツールの成果物ではないことをユーザーが確信できるよう、信頼できるものでなければなりません。特に、`.dfx`、`node_modules`、`target` ディレクトリにはビルド前のファイルが含まれている可能性があるため、注意が必要です。

次に、それぞれのポイントについて詳しく見ていきます。

### ソースコードを提供する

一般的に、`git` やその他のバージョン管理システムでコードをバージョン管理し、バージョン管理されたコードは GitHub などの公開リポジトリで利用できるでしょう。この場合、Internet Computer にデプロイするコードを作成する際に使用した特定のコミットを記録し、あなたの Canister を検証しようとしているユーザーに伝える必要があります。また、Canister のビルドに使用したソースコードを含むパッケージ（zip ファイルや tarball など）を提供することもできます。

### ビルド環境を再現する

コードをビルドする前に、使用しているビルド環境を詳細に文書化する必要があります。特に、Internet Computer SDK がサポートする言語については：

-   Canister をビルドするために使用している OS とそのバージョンをメモして下さい。

-   もし、`dfx` を使用している場合は、`dfx.json` で指定されている使用するバージョンをメモしてください。任意のバージョンの `dfx` をインストールするには、 `dfx toolchain install <version>` を使用するか、環境変数 `DFX_VERSION` に任意のバージョンを設定してインストールスクリプトを実行します。

-   もし、`dfx build` 以外の方法で Motoko コードをビルドしている場合は、使用している `moc` のバージョンをメモしてください。

-   Rust をビルドしている場合は、使用している `cargo` のバージョンをメモしてください。

-   フロントエンドの開発に Node.js や `webpack` を使用している場合、それらのバージョンをメモしてください。

-   ビルドプロセスが環境変数（タイムゾーンやロケールなど）に依存する場合はメモしておきます。

これら全てを説明書の中でユーザーに伝える必要があります。理想的には、Docker や Nix などのツールを使って、ビルド環境を再現するための実行可能なレシピを提供することで行います。Docker を使用すると、ソフトウェアのビルドに使用されたオペレーティングシステムを特定することができるので、Docker の使用をお勧めします。

### Docker を用いた環境をビルドする

[Docker コンテナ](https://docs.docker.com/)は、ビルド環境を提供するための一般的なソリューションです。OS X を使用している開発者には、[lima](https://github.com/lima-vm/lima) を使用して Docker をインストールすることをお勧めします。これは我々の経験上、Docker Desktop や Docker Machine よりも安定していることが証明されています（特に、Apple M1 マシンにおけるいくつかの QEMU バグを回避することができます）。Docker をセットアップした後、以下のような `Dockerfile` を使用して、ユーザに特定のバージョンの OS、`dfx`、Node.js、Rust ツールチェインを提供することが可能です。Docker コンテナの実行には、必ず `x86_64` を使用してください。一般的に、ビルドはアーキテクチャ間で再現性がありません。ホスト環境が `x86_64` でない場合のクロスプラットフォームな Docker コンテナのセットアップについては [docs](https://github.com/lima-vm/lima/blob/master/docs/multi-arch.md) を参照してください。

    FROM ubuntu:22.04

    ENV NVM_DIR=/root/.nvm
    ENV NVM_VERSION=v0.39.1
    ENV NODE_VERSION=18.1.0

    ENV RUSTUP_HOME=/opt/rustup
    ENV CARGO_HOME=/opt/cargo
    ENV RUST_VERSION=1.62.0
    ENV IC_CDK_OPTIMIZER_VERSION=0.3.4

    ENV DFX_VERSION=0.11.0

    # Install a basic environment needed for our build tools
    RUN apt -yq update && \
        apt -yqq install --no-install-recommends curl ca-certificates \
            build-essential pkg-config libssl-dev llvm-dev liblmdb-dev clang cmake rsync

    # Install Node.js using nvm
    ENV PATH="/root/.nvm/versions/node/v${NODE_VERSION}/bin:${PATH}"
    RUN curl --fail -sSf https://raw.githubusercontent.com/creationix/nvm/${NVM_VERSION}/install.sh | bash
    RUN . "${NVM_DIR}/nvm.sh" && nvm install ${NODE_VERSION}
    RUN . "${NVM_DIR}/nvm.sh" && nvm use v${NODE_VERSION}
    RUN . "${NVM_DIR}/nvm.sh" && nvm alias default v${NODE_VERSION}

    # Install Rust and Cargo
    ENV PATH=/opt/cargo/bin:${PATH}
    RUN curl --fail https://sh.rustup.rs -sSf \
            | sh -s -- -y --default-toolchain ${RUST_VERSION}-x86_64-unknown-linux-gnu --no-modify-path && \
        rustup default ${RUST_VERSION}-x86_64-unknown-linux-gnu && \
        rustup target add wasm32-unknown-unknown
    RUN cargo install --version ${IC_CDK_OPTIMIZER_VERSION} ic-cdk-optimizer

    # Install dfx
    RUN sh -ci "$(curl -fsSL https://internetcomputer.org/install.sh)"

    COPY . /canister
    WORKDIR /canister

Rust プロジェクトをビルドするスクリプトの例は、以下のようになります：

    #!/bin/bash
    #
    # additional setup, e.g., build frontend assets:
    # ...
    # Rust build:
    export RUSTFLAGS="--remap-path-prefix $(readlink -f $(dirname ${0}))=/build --remap-path-prefix ${CARGO_HOME}=/cargo"
    cargo build --locked --target wasm32-unknown-unknown --release
    ic-cdk-optimizer target/wasm32-unknown-unknown/release/example_backend.wasm -o example_backend.wasm

このようなビルドスクリプトは、以下のように `dfx.json` にカスタムビルドスクリプトとして設定することも可能です：

    "canisters": {
      "example_backend": {
        "candid": "src/example_backend/example_backend.did",
        "package": "example_backend",
        "type": "custom",
        "wasm": "./example_backend.wasm",
        "build": "./build_script.sh"
      }
    }

この `Dockerfile` には、いくつか注目すべき点があります：

-   公式の Docker イメージから起動します。さらに、インストールされているツールはすべて標準的なものであり、標準的なソースから提供されています。このことは、ビルド環境が改ざんされていないこと、つまり Docker を使ったビルドプロセスが信頼できることをユーザに確信させるものです。

-   特定のバージョンのビルドツールを確実にインストールするために、`apt`（コンテナ内で動作する Linux ディストリビューション、Ubuntu のパッケージマネージャ）経由ではなく、直接インストールするようになっています。このようなパッケージマネージャは、通常、ビルドツールを特定のバージョンに固定する方法を提供しません。

この `Dockerfile` を使用するには、Docker を [up and running](https://docs.docker.com) して、`Dockerfile` を Canister のプロジェクトディレクトリに配置し、実行して Docker コンテナを作成する必要があります：

    $ docker build -t mycanister .

これは `mycanister` という Docker コンテナイメージを作成し、その中に Node.js、 Rust、 `dfx` をインストールし、Canister のソースコードを `/canister` にコピーします（Canister プロジェクトディレクトリから `docker build` を起動することを思い出してください）。そして、コンテナ内でインタラクティブなシェルを実行することができます：

    $ docker run -it --rm mycanister

ここから、Canister をビルドするために必要なステップをテストすることができます。ステップが決定論的であると確信が持てたら、それらを `Dockerfile` に記述して（例えば、ビルドスクリプト `./build_script.sh` に対して `RUN ./build_script.sh` として）ユーザーが自動的にあなたの Canister のビルドを再現できるようにすることも可能です。[Internet Identity  Canister の Dockerfile](https://github.com/dfinity/internet-identity/blob/397d0087a29855564c47f0fd3323f60b5b67a8fa/Dockerfile) で例を見ることができます。次に、ビルドを決定論的にするために必要なことを調査します。

### ビルドプロセスの決定性を確保する

ビルドプロセスが決定論的であるためには：

1.  Canister の依存関係が常に同じ方法で解決されていることを確認する必要があります。現在、ほとんどのビルドツールは、依存関係を特定のバージョンに固定する方法をサポートしています。

    -   `npm` の場合、 `npm install` を実行すると、 `package.json` で指定した要件を満たす、プロジェクトの全ての（推移的な）依存パッケージの固定バージョンを含む `package-lock.json` ファイルが作成されます。しかし、 `npm install` は実行するたびに `package-lock.json` ファイルを上書きします。そのため、最終的な Canister を作成する準備ができたら、 `npm install` を一度だけ実行してください。その後、 `package-lock.json` をバージョン管理システムへコミットします。最後に、ビルドの再現性を確認する際には、 `npm install` の代わりに `npm ci` を使ってください。

    -   Rust のコードの場合、Cargo は自動的に `Cargo.lock` ファイルを生成し、固定バージョンの（推移的な）依存性を持たせます。`package-lock.json` と同様に、Canister の最終版を作成する準備ができたら、このファイルをバージョン管理システムへコミットしてください。さらに、Cargo はデフォルトで依存関係のロックされたバージョンを無視します。ロックされた依存関係を使用するようにするには、 `cargo` コマンドに `--locked` フラグを渡してください。

    -   Canister は Identity でお互いを参照するため、あらかじめ Canister ID を割り当てておく必要があります。

2.  あなた自身のビルドスクリプトには、非決定性を導入してはいけません。非決定性の明らかな原因は、ランダム性、タイムスタンプ、並行処理、コードの難読化などです。あまり明らかではありませんが、ロケール、ファイルの絶対パス、ディレクトリ内のファイルの順序、内容が変更される可能性のあるリモート URL なども含まれます。さらに、サードパーティのビルドプラグインに依存すると、これらによってもたらされる非決定性にもさらされることになります。

3.  同じ依存関係と決定論的なビルドスクリプトを考えると、ビルドツール自体（Motoko では `moc`、Rust では `cargo`、フロントエンド開発ではデフォルトで `webpack`）も決定論的である必要があります。良いニュースは、これらのツールはすべて決定論的であることを目指していることです。しかし、これらのツールは複雑なソフトウェアであり、決定性を確保することは自明ではありません。したがって、非決定論的なバグは起こり得ますし、実際に起こっています。Rust については、[Rust における現在の潜在的な非決定性問題のリスト](https://github.com/rust-lang/rust/labels/A-reproducibility)を参照してください。さらに、Linux と MacOS で Wasm にコンパイルされた Rust コードの違いが観察されたので、ビルドプラットフォームとそのバージョンを固定することをお勧めします。webpack については、バージョン5から使うべき[モジュールとチャンクの ID の決定論的命名](https://webpack.js.org/configuration/optimization/)が導入されました。Motoko コンパイラは、決定論的かつ再現可能であることを目指しています。再現性に問題がある場合は、[new issue](https://github.com/dfinity/motoko/issues/new/choose) を提出していただければ、可能な範囲で対処します。

### 再現性をテストする

もし、あなたのコードに再現性が不可欠であるなら、ビルドをテストして再現性に対する自信を深めるべきです。このようなテストは自明ではありません。私たちは、 Canister ビルドの非決定性が現れるまでに１ヶ月かかったという実例を見たことがあります！幸いなことに、Debian Reproducible Builds プロジェクトは [reprotest](https://salsa.debian.org/reproducible-builds/reprotest) というツールを作成し、再現性テストの自動化を手助けしてくれます。これは、パスや時間、ファイルの順序などの特性が異なるふたつの異なる環境でビルドを実行し、その結果を比較することでビルドをテストします。`reprotest` を使ってビルドをチェックするには、`Dockerfile` に以下の行を追加してください：

    RUN apt -yqq install --no-install-recommends reprotest disorderfs faketime rsync sudo wabt

`dfx build --network ic` を使用する場合、フロントエンドの依存関係をプレビルドする必要があります（例えば、`dfx build --network ic` の前に `npm ci` を実行するか、ビルドスクリプトで `dfx.json` にカスタムビルドタイプを設定し `npm ci` を実行してください）。またプロジェクトディレクトリには Internet Computer 上の Canister  ID を含む `canister_ids.json` ファイルを保存しておくとよいでしょう。｀canister_ids.json`ファイルの例は以下のようになります：

    {
      "example_backend": {
        "ic": "rrkah-fqaaa-aaaaa-aaaaq-cai"
      },
      "example_frontend": {
        "ic": "ryjl3-tyaaa-aaaaa-aaaba-cai"
      }
    }

Canister プロジェクトのルートディレクトリから、以下のように `dfx` のビルドの再現性をテストすることができます：

    $ docker build -t mycanister .
    ...
    $ docker run --rm --privileged -it mycanister
    /canister# mkdir artifacts
    /canister# reprotest -vv --store-dir=artifacts --variations '+all,-time' 'dfx build --network ic' '.dfx/ic/canisters/*/*.wasm'

最初のコマンドは、先に提供された `Dockerfile` を使って Docker コンテナをビルドします。2つ目のコマンドは、コンテナ内でインタラクティブなシェル（`-it` フラグ）を開きます。ここでは特権モードで実行します（`—privileged` フラグ）。これは、`reprotest` がカーネルモジュールを使用して、ビルド環境のバリエーションを増やしているためです。また、いくつかのバリエーションを除外することで、非特権モードで実行することもできます。[reprotest manual](https://manpages.debian.org/stretch/reprotest/reprotest.1.en.html)  を参照してください。`rm` フラグは、シェルを閉じた後にコンテナを破壊します。最後に、コンテナ内にビルド用のディレクトリを作成し、冗長モードで `reprotest` を起動します（`-vv` フラグ）。最初の引数として、実行したいビルドコマンドを与える必要があります。ここでは、`dfx build --network ic` とします。もし、別のビルドプロセスを使用している場合には、調整してください。これで、ふたつの異なる環境でビルドが実行されます。最後に、ふたつのビルドの最後に比較するパスを `reprotest` に指定する必要があります。ここでは、`.dfx/ic` ディレクトリにある、すべての Canister 用の Wasm コードを比較します。Rust コンパイラは動的なメモリ割り当てに `jemalloc` を使用しており、このライブラリは `reprotest` が時間変化の実装に使用している `faketime` と [compatible](https://github.com/wolfcw/libfaketime/issues/130) はないため、時間変化の実装を省略しています。しかし、手動でシステム時刻を変更しながら、 `reprotest` が生成する成果物を比較することをお勧めします。

比較の結果、差異が見つからなかった場合は、このような出力が表示されます：

    =======================
    Reproduction successful
    =======================
    No differences in ./.dfx/ic/canisters/*/*.wasm
    6b2a15a918219138836e88e9c95f9c5d2d7b6d465df83ae05d6fd2b0f14f8a97  ./.dfx/ic/canisters/example_backend/example_backend.wasm
    a047686c1d517e21d447bcd42c9394a12cdb240e06425b830c99d3a689b5ee20  ./.dfx/ic/canisters/example_frontend/assetstorage.wasm
    a047686c1d517e21d447bcd42c9394a12cdb240e06425b830c99d3a689b5ee20  ./.dfx/ic/canisters/example_frontend/example_frontend.wasm

おめでとうございます。これは、ビルドが環境の影響を受けていないことを示す良い指標です! `reprotest` では、依存関係が適切に固定されているかどうかをチェックすることはできませんので注意ください。さらに、コンテナ `reprotest` のビルドを複数のホスト OS で実行し、結果を比較することをお勧めします。比較の結果、ふたつのビルドで生成された Wasm コードの間に違いが見つかった場合、差分が出力されます。このとき、 `reprotest` の `--store-dir` フラグを使用して、出力と diff を分析できる場所に保存したいと思うことでしょう。もし、再現性の確保に苦労しているのであれば、 [DetTrace](https://github.com/dettrace/dettrace) の使用も検討してください。これは、任意のビルドを決定的にするためのコンテナ抽象化機能です。

最後に、ビルドの再現性を確保した後でも、長期的に考慮すべきことがあります。

### 長期的な視点での考察

 Canister コードを何年も使用し、再現性を維持することを期待する場合、再現性はより厳しいものになります。最大の課題は以下のとおりです：

1.  ビルドツールチェーンは今後も利用可能であるか

2.  依存関係が有効か

3.  ツールチェーンはまだ動作し、依存関係を正しくビルドするか

ディストリビューションやパッケージアーカイブは、あなたのツールチェーンとその依存関係を含む、古いバージョンのパッケージを削除することがあります。ウェブサイトはオフラインになり、URL が機能しなくなるかもしれません。したがって、あなたのツールチェーンと依存関係をすべてバックアップしておくことが賢明です。[Software Heritage](https://www.softwareheritage.org/) のような、大規模なプロジェクトに参加することを検討すべきです。後日、ある時点で、 Canister がまだビルドできるように、ビルドプロセスを調整しなければならないかもしれません（例えば、URL の変更など）。ビルドが変更されても、同じ結果が得られるのであれば、ユーザーは Canister が正しいコードを実行していると確信することができます。Software Heritage プロジェクトのような、信頼できるソースからの依存関係があれば、信頼性の議論はより簡単になります。

## まとめ

Canister の作成者への推奨事項をまとめます：

-   理想的には、コンテナコードの最終版を制作する際に、Docker または同様の技術を使用して、OS とビルドツールを有効にセットアップし、ユーザーのためにそれらのバージョンを修正することです。使用しているビルドツールが完全に再現可能なビルドを保証しない場合、Docker はパスや環境変数などの違いを最小化することでも支援することができます。

-   ビルドツールとベースとなる Docker イメージは、ユーザーが信頼できるところから提供されるべきです。

-   Rust と Motoko のコンパイラは決定論的であることを目指しており、そのため再現性のあるビルドをサポートしています。もし、非決定論に気づいたら、バグレポートをしてください。

-   NPM を使用する場合、すべての依存関係のバージョンを正確に指定してください（`package_lock.json` を git リポジトリにコミットしてください！）。ビルドを再現するために、`install` ではなく、`ci` コマンドを使って NPM を起動します。同様に、Rust パッケージの場合は、 `Cargo.lock` をリポジトリにコミットし、パッケージのビルド時に `cargo build --locked` を使用します。

-   Webpack のビルドは決定論的であるべきですが、obfuscators や類似のツールは再現性を損なう可能性があります。決定論的なチャンクとモジュール ID を使用するようにしてください。

-   ビルドツールは完璧ではないので、再現性のあるビルドを保証できない場合があります。もし、あなたの Canister にとって再現性が重要であれば（例えば、他のユーザーの資金を保管しているなど）、  テストしてください。Reprotest はこの目的のために有用なツールです。

-   理想的には、依存関係の数は最小限にしたいものです。完全な監査を行うためには、ユーザーは依存関係もすべて（再現可能な形で）再構築しなければならないかもしれないからです。

-   再現性の確保は、時間軸が長くなると難しくなります。これは主に、依存関係やビルドツールの信頼できるソースを利用し続ける必要があるためです。

最後に、ビルドが再現可能であれば、出来上がった Wasm コードのハッシュと、 Canister で実行中のコードのハッシュを比較し、以下のように取得することが可能です：

    $ dfx canister --network ic info <canister-id>

コントローラーが Canister コードをアップグレードした場合、このハッシュが変更される可能性がありますので注意してください。

<!--
# Reproducible Canister Builds

Thanks to its consensus protocol, the Internet Computer always runs canisters' code correctly. But this doesn’t mean that it’s running the *correct* code for a canister. If you are using a canister somebody else developed, you may want to verify that the canister is indeed running some intended code before giving it control to make important decision for you, e.g., sending your ICP to a canister. Verifying this requires answering two questions:

1.  Which WebAssembly (Wasm) code is being executed for a canister?

2.  The canisters are normally written in a higher-level language, such as Motoko or Rust, and not directly in Wasm. The second question is then: is the Wasm that’s running really the result of compiling the purported source code?

The rest of the document answers these questions. For the first question, we will see how the Internet Computer provides information about canister code. To be able to answer the second question, canister authors should ensure trusted and [reproducible builds](https://reproducible-builds.org/docs/definition/) of the Wasm code from the source. Such builds allow anyone to follow the same steps as the canister authors and yield the exact same Wasm code, which can then be compared to the Wasm code executing in the canister on the Internet Computer.

The impatient reader who is familiar with the topic of reproducible builds can skip straight to the [summary](#repro-build-summary).

## Finding out which canister code the Internet Computer is executing

Internet Computer does not allow you to access the Wasm code of an arbitrary canister. This is a design decision, as developers might want to keep some code private. However, the Internet Computer does allow you to access the SHA-256 of the Wasm code of a canister.

To obtain this hash, you must first note the principal of the Internet Computer canister whose code you want to check. For example, assume we’re interested in the code of the Internet Identity canister, whose principal is `rdmx6-jaaaa-aaaaa-aaadq-cai`. Then, the easiest way to access this service is using the [`dfx`](https://internetcomputer.org/developers/) tool from the terminal. Open your terminal, and run:

    $ dfx canister --network ic info rdmx6-jaaaa-aaaaa-aaadq-cai
    Controllers: r7inp-6aaaa-aaaaa-aaabq-cai
    Module hash: 0x2d95e90de5d7de11f25ac256690aff44c6685a1570b1becdf6e50192e983e103

If you are running an older version of `dfx`, you will need to run this command from a directory that contains a valid `dfx.json` file. If you don’t have such a directory, you can create it using `dfx new`.

Here, the Internet Computer tells us that the hash of the Wasm module of the `rdmx6-jaaaa-aaaaa-aaadq-cai` canister (which happens to be the Internet Identity canister) is `0x2d95e90de5d7de11f25ac256690aff44c6685a1570b1becdf6e50192e983e103`.

The check above provides you the *current* hash of the canister’s Wasm module, but the *controllers* of an Internet Computer canister may change the code at any time (e.g., to upgrade a canister). However, if the list of controllers is empty or the only controller is a [black-hole](https://github.com/ninegua/ic-blackhole) canister, you know that the canister is immutable since nobody has the power to change the code.

Armed with this hash, you can next check whether it corresponds to some given source code. This only works if the build process for the code is reproducible.

## Reproducible builds

As a canister author, there are a few things you have to provide to your users to allow them to reproduce your build:

-   The same source code that you used to create the Wasm module for the canister.

-   Instructions on how to recreate your build environment.

-   Instructions on how to repeat the process of building the Wasm from the source code. Crucially, the process must be deterministic, to ensure that it results in the exact same Wasm. It also has to be trusted, such that the user can be convinced that the Wasm is a faithful translation of the source code, and not an artifact of a malicious build tool. In particular, look for `.dfx`, `node_modules`, and `target` directories that could contain pre-built files.

We next look at each of these points in more detail.

### Providing source code

Typically, you will version your code in `git` or some other version control system, and your versioned code may be available in a public repository, e.g., GitHub. In this case, you should note the particular commit that you used when producing the code to be deployed to the Internet Computer, and communicate it to the users who are trying to verify your canister. Alternatively, you can also provide a package (e.g., a zip file or a tarball) containing the source code you used to build your canister.

### Reproducing build environments

Before building your code, you should document the build environment you are using in detail. In particular, for the languages supported by the Internet Computer SDK:

-   Note down the operating system and its version you are using to build your canister.

-   If you are using `dfx`, note the version used, as specified in `dfx.json`. You can install arbitrary versions of `dfx` using `dfx toolchain install <version>`, or by running the installation script with the `DFX_VERSION` environment variable set to the desired version.

-   If you are building Motoko code in a way other than `dfx build`, note the version of `moc` you are using.

-   If you are building Rust, note the version of `cargo` you are using.

-   If you are using Node.js and/or `webpack` for frontend development, note their versions.

-   If your build process depends on any environment variables (such as the time zone or locale), note them down.

You should communicate all of these to your user in the instructions. Ideally, do so by providing an executable recipe to recreate the build environment, using tools such as Docker or Nix. We recommend using Docker, as this allows you to also pinpoint the operating system used for building the software.

### Build environments using Docker

[Docker containers](https://docs.docker.com/) are a popular solution for providing build environments. For developers using OS X, we recommend installing Docker using [lima](https://github.com/lima-vm/lima), as it proved more stable than Docker Desktop or Docker Machine in our experience (in particular, it avoids some QEMU bugs on Apple M1 machines). After setting Docker up, you can use a `Dockerfile` such as the following to provide the user with a particular version of the operating system, as well as `dfx`, Node.js and the Rust toolchain. Make sure to stick with `x86_64` for running the Docker container, as builds are generally not reproducible across architectures. See [docs](https://github.com/lima-vm/lima/blob/master/docs/multi-arch.md) on setting up cross-platform Docker containers in case your host environment is not `x86_64`.

    FROM ubuntu:22.04

    ENV NVM_DIR=/root/.nvm
    ENV NVM_VERSION=v0.39.1
    ENV NODE_VERSION=18.1.0

    ENV RUSTUP_HOME=/opt/rustup
    ENV CARGO_HOME=/opt/cargo
    ENV RUST_VERSION=1.62.0
    ENV IC_CDK_OPTIMIZER_VERSION=0.3.4

    ENV DFX_VERSION=0.11.0

    # Install a basic environment needed for our build tools
    RUN apt -yq update && \
        apt -yqq install --no-install-recommends curl ca-certificates \
            build-essential pkg-config libssl-dev llvm-dev liblmdb-dev clang cmake rsync

    # Install Node.js using nvm
    ENV PATH="/root/.nvm/versions/node/v${NODE_VERSION}/bin:${PATH}"
    RUN curl --fail -sSf https://raw.githubusercontent.com/creationix/nvm/${NVM_VERSION}/install.sh | bash
    RUN . "${NVM_DIR}/nvm.sh" && nvm install ${NODE_VERSION}
    RUN . "${NVM_DIR}/nvm.sh" && nvm use v${NODE_VERSION}
    RUN . "${NVM_DIR}/nvm.sh" && nvm alias default v${NODE_VERSION}

    # Install Rust and Cargo
    ENV PATH=/opt/cargo/bin:${PATH}
    RUN curl --fail https://sh.rustup.rs -sSf \
            | sh -s -- -y --default-toolchain ${RUST_VERSION}-x86_64-unknown-linux-gnu --no-modify-path && \
        rustup default ${RUST_VERSION}-x86_64-unknown-linux-gnu && \
        rustup target add wasm32-unknown-unknown
    RUN cargo install --version ${IC_CDK_OPTIMIZER_VERSION} ic-cdk-optimizer

    # Install dfx
    RUN sh -ci "$(curl -fsSL https://internetcomputer.org/install.sh)"

    COPY . /canister
    WORKDIR /canister

An example script to build a Rust project looks as follows:

    #!/bin/bash
    #
    # additional setup, e.g., build frontend assets:
    # ...
    # Rust build:
    export RUSTFLAGS="--remap-path-prefix $(readlink -f $(dirname ${0}))=/build --remap-path-prefix ${CARGO_HOME}=/cargo"
    cargo build --locked --target wasm32-unknown-unknown --release
    ic-cdk-optimizer target/wasm32-unknown-unknown/release/example_backend.wasm -o example_backend.wasm

Such a build script can also be set as a custom build script in `dfx.json` as follows:

    "canisters": {
      "example_backend": {
        "candid": "src/example_backend/example_backend.did",
        "package": "example_backend",
        "type": "custom",
        "wasm": "./example_backend.wasm",
        "build": "./build_script.sh"
      }
    }

There are a couple of things worth noting about this `Dockerfile`:

-   It starts from an official Docker image. Furthermore, all the installed tools are standard, and come from standard sources. This provides the user with confidence that the build environment hasn’t been tampered with, and thus that the build process using Docker can be trusted.

-   To ensure that specific versions of the build tools are installed, it installs them directly, rather than through `apt` (the package manager of Ubuntu, the Linux distribution running inside of the container). Such package managers usually don’t provide a way of pinning the build tools to specific versions.

To use this `Dockerfile`, get Docker [up and running](https://docs.docker.com), place the `Dockerfile` in the project directory of your canister, and create the Docker container by running:

    $ docker build -t mycanister .

This creates a Docker container image called `mycanister`, with Node.js, Rust and `dfx` installed in it, and your canister source code copied to `/canister` (recall that you should invoke `docker build` from the canister project directory). You can then enter an interactive shell inside of your container by running:

    $ docker run -it --rm mycanister

From here, you can experiment with the steps needed to build your canister. Once you are confident that the steps are deterministic, you can also put them in the `Dockerfile` (e.g., as `RUN ./build_script.sh` for a build script `./build_script.sh`), to allow the user to automatically reproduce the build of your canister. You can see an example in the [Dockerfile of the Internet Identity canister](https://github.com/dfinity/internet-identity/blob/397d0087a29855564c47f0fd3323f60b5b67a8fa/Dockerfile). Next, we will investigate what is necessary to make the build deterministic.

### Ensuring the determinism of the build process

For the build process to be deterministic:

1.  You will need to ensure that any dependencies of your canister are always resolved in the same way. Most build tools now support a way of pinning dependencies to a particular version.

    -   For `npm`, running `npm install` will create a `package-lock.json` file with some fixed versions of all (transitive) dependencies of your project that satisfy the requirements specified in your `package.json`. However, `npm install` will overwrite the `package-lock.json` file every time it is invoked. Thus, once you are ready to create the final version of your canister, run `npm install` only once. After that, commit `package-lock.json` to your version control system. Finally, when checking the build for reproducibility, use `npm ci` instead of `npm install`.

    -   For Rust code, Cargo will automatically generate a `Cargo.lock` file with the fixed versions of your (transitive) dependencies. Like with `package-lock.json`, you should commit this file to your version control system once you are ready to produce the final version of your canister. Furthermore, Cargo by default ignores the locked versions of dependencies. Pass the `--locked` flag to the `cargo` command to ensure that the locked dependencies are used.

    -   You have to allocate canister IDs in advance, as canisters refer to each other by their IDs.

2.  Your own build scripts must not introduce non-determinism. Obvious sources of non-determinism include randomness, timestamps, concurrency, or code obfuscators. Less obvious sources include locales, absolute file paths, order of files in a directory, and remote URLs whose content can change. Furthermore, relying on third-party build plug-ins exposes you to any non-determinism introduced by these.

3.  Given the same dependencies and deterministic build scripts, the build tools themselves (`moc` for Motoko, `cargo` for Rust, `webpack` by default for frontend development) must also be deterministic. The good news is that all of these tools aim to be deterministic. However, they are complicated pieces of software, and ensuring determinism is non-trivial. Thus, non-determinism bugs can and do occur. For Rust, see the [list of current potential non-determinism issues in Rust](https://github.com/rust-lang/rust/labels/A-reproducibility). Furthermore, we have observed differences between Rust code compiled to Wasm under Linux and MacOS, and thus recommend pinning the build platform and its version. For webpack, [deterministic naming of module and chunk IDs](https://webpack.js.org/configuration/optimization/) that you should use have been introduced since version 5. The Motoko compiler aims to be deterministic and reproducible; if you find reproducibility issues, please submit a [new issue](https://github.com/dfinity/motoko/issues/new/choose), and we will try to address them to the extent possible.

### Testing reproducibility

If reproducibility is vital for your code, you should test your builds to increase your confidence in their reproducibility. Such testing is non-trivial: we have seen real-world examples where non-determinism in a canister build took a month to show up! Fortunately, the Debian Reproducible Builds project created a tool called [reprotest](https://salsa.debian.org/reproducible-builds/reprotest), which can help you automate reproducibility tests. It tests your build by running it in two different environments that differ in characteristics such as paths, time, file order, and others, and comparing the results. To check your build with `reprotest`, add the following line to your `Dockerfile`:

    RUN apt -yqq install --no-install-recommends reprotest disorderfs faketime rsync sudo wabt

When using `dfx build --network ic`, you need to prebuild your frontend dependencies (e.g., by running `npm ci` before `dfx build --network ic` or by setting the custom build type in `dfx.json` and running `npm ci` in your build script) and your project directory should contain a `canister_ids.json` file containing the IDs of your canisters on the Internet Computer. An example `canister_ids.json` file looks as follows:

    {
      "example_backend": {
        "ic": "rrkah-fqaaa-aaaaa-aaaaq-cai"
      },
      "example_frontend": {
        "ic": "ryjl3-tyaaa-aaaaa-aaaba-cai"
      }
    }

Now, from the root directory of your canister project, you can test the reproducibility of your `dfx` builds as follows:

    $ docker build -t mycanister .
    ...
    $ docker run --rm --privileged -it mycanister
    /canister# mkdir artifacts
    /canister# reprotest -vv --store-dir=artifacts --variations '+all,-time' 'dfx build --network ic' '.dfx/ic/canisters/＊/＊.wasm'

The first command builds the Docker container using the `Dockerfile` provided earlier. The second one opens an interactive shell (hence the `-it` flags) in the container. We run this in privileged mode (the `--privileged` flag), as `reprotest` uses kernel modules for some build environment variations. You can also run it in non-privileged mode by excluding some of the variations; see the [reprotest manual](https://manpages.debian.org/stretch/reprotest/reprotest.1.en.html). The `--rm` flag will destroy the container after you close its shell. Finally, once inside of the container, we create a directory for the build artifacts and launch `reprotest` in verbose mode (the `-vv` flags). You need to give it the build command you want to run as the first argument. Here, we assume that it’s `dfx build --network ic` - adjust it if you’re using a different build process. It will then run the build in two different environments. Finally, you need to tell `reprotest` which paths to compare at the end of the two builds. Here, we compare the Wasm code for all canisters, which is found in the `.dfx/ic` directory. We omit the time variation because the Rust compiler uses `jemalloc` for dynamic memory allocation and this library is not [compatible](https://github.com/wolfcw/libfaketime/issues/130) with `faketime` used by `reprotest` to implement the time variation. Nevertheless, we encourage you to compare the artifacts produced by `reprotest` while manually changing your system time.

If the comparison doesn’t find any differences, you will see an output similar to this one:

    =======================
    Reproduction successful
    =======================
    No differences in ./.dfx/ic/canisters/＊/＊.wasm
    6b2a15a918219138836e88e9c95f9c5d2d7b6d465df83ae05d6fd2b0f14f8a97  ./.dfx/ic/canisters/example_backend/example_backend.wasm
    a047686c1d517e21d447bcd42c9394a12cdb240e06425b830c99d3a689b5ee20  ./.dfx/ic/canisters/example_frontend/assetstorage.wasm
    a047686c1d517e21d447bcd42c9394a12cdb240e06425b830c99d3a689b5ee20  ./.dfx/ic/canisters/example_frontend/example_frontend.wasm

Congratulations - this is a good indicator that your build is not affected by your environment! Note that `reprotest` can’t check that your dependencies are pinned properly - use guidelines from the previous section for that. Moreover, we recommend you to run the container `reprotest` builds under several host operating systems and compare the results. If the comparison does find differences between the Wasm code produced in two builds, it will output a diff. You will then likely want to use the `--store-dir` flag of `reprotest` to store the outputs and the diff somewhere where you can analyze them. If you are struggling to achieve reproducibility, consider also using [DetTrace](https://github.com/dettrace/dettrace), which is a container abstraction that tries to make arbitrary builds deterministic.

Finally, even after you achieve reproducibility for your builds, there are still other things to consider for the long term.

### Long-term considerations

Reproducibility can be more demanding if you expect your canister code to stay around for years, and stay reproducible. The biggest challenges are to ensure that your:

1.  Build toolchain is still available in the future.

2.  Dependencies are avaiable.

3.  Toolchain still runs and still correctly builds your dependencies.

Distributions and package archives may drop old versions of packages, including both your toolchain and their dependencies. Web sites may go offline and URLs might stop working. Thus, it’s prudent to back up all of your toolchain and dependencies. You should consider getting involved in projects such as [Software Heritage](https://www.softwareheritage.org/), which do this on a large scale. At some later point in time, you might have to adjust your build process (e.g., by changing URLs) to ensure that your canister still builds. Even if the build changes, if it still yields the same result, your users can be confident that your canister is running the correct code. The trust argument is easier if your dependencies come from a trustworthy source, such as the Software Heritage project.

## Summary

Summarizing our recommendations for canister authors:

-   Ideally, when producing the final version of your container code, use Docker or a similar technology to conveniently set up the operating system and the build tools, and fix their versions for the user. If the build tools you are using don’t guarantee fully reproducible builds, Docker can also help by minimizing the differences in paths, environment variables etc.

-   The build tools and the base Docker image should be sourced from somewhere that the user can trust.

-   Rust and Motoko compilers aim to be deterministic, and thus to support reproducible builds. If you notice non-determinism, file bug reports.

-   When using NPM, ensure that you specify the exact versions of all your dependencies (commit `package_lock.json` to your git repo!). Invoke NPM using the `ci` command rather than `install` to reproduce the build. Similarly, for Rust packages commit `Cargo.lock` to your repository, and then use `cargo build --locked` when building the package.

-   Webpack builds should be deterministic, but obfuscators and similar tools may compromise reproducibility. Make sure you use deterministic chunk and module IDs.

-   Build tools aren’t perfect, and may fail to ensure reproducible builds. If reproducibility is critical for your canister (e.g., it holds other users' funds), test it. Reprotest is a useful tool for this purpose.

-   Ideally, you want to minimize the number of dependencies, as, in order to do a full audit, the user may have to (reproducibly) rebuild all of your dependencies too.

-   Achieving reproducibility is harder over longer time scales, primarily as you need to ensure that a trustworthy source of your dependencies and build tools stays available.

Finally, if your build is reproducible, you can compare the hash of the resulting Wasm code to the hash of the code that is running in a canister, which you retrieve as follows:

    $ dfx canister --network ic info <canister-id>

Beware that this hash might change if the controllers upgrade the canister code.
428, 437の"*"は「＊」に変更してあります
-->