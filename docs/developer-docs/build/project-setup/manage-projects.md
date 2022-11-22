# プロジェクトを管理する

各プロジェクトの `dfx.json` 設定ファイルを変更することで、個々のプロジェクトに対する重要な設定のいくつかを変更することができます。これらの設定は `dfx config` コマンドを使用してプログラムで変更することもできますし、 `dfx.json` ファイルを直接手動で編集することもできます。

## ソースディレクトリを変更する方法

`dfx build` コマンドを使用してプロジェクトのソースコードをコンパイルする前に、Dapp のソースコードを保存するデフォルトの場所を確認しておくとよいでしょう。新しいプロジェクトを作成するときに使う名前は、デフォルトでひとつの Canister（`canister_name`）とひとつのアセット Canister（`canister_name_assets`）とし、Dapp のソースコードは `src/canister_name` ディレクトリにあることが慣習です。同様に、フロントエンドのソースコードは `src/canister_name_assets/src` ディレクトリに、フロントエンドのアウトプットは `dist/canister_name_assets` ディレクトリに置かれることがデフォルトとなります。

アプリの複雑さやアーキテクチャーによっては、アプリのソースコード、フロントエンドのソースコード、フロントエンドのアウトプットのデフォルトの場所を変更したいと思うかもしれません。

例えば、シンプルなアプリの場合、ディレクトリをひとつ削除し、ソースコードを `src` ディレクトリに配置することができます：

      "main": "src/main.mo",

より複雑な Dapps の場合は、多階層のディレクトリ構造を採用するとよいでしょう：

    "canisters": {
      "profiles": {
        "main": "src/profiles/utils/main.mo"
      },
      "events": {
        "main": "src/events/calendar/main.mo"
      },
      "media": {
        "main": "src/events/reports/main.mo"
      }
    }

ソースコードのディレクトリのデフォルトの設定を変更する場合は、`dfx.json` 設定ファイルの設定がファイルシステム上のディレクトリの位置と一致していることを確認してください。

## メイン Dapp のファイル名を変更する方法

`dfx build` コマンドを使用してプロジェクトのソースコードをコンパイルする前に、Dapp のソースコードに使用されている場所とファイル名を確認する必要があります。

例えば、`factorial` というアプリの Canister をビルドする場合、そのアプリのソースコードが `src/math/factorial.mo` にあるのなら、設定ファイルの `canisters` セクションの `main` 設定で正しいパスを指定しているかどうか確認する必要があります。

例：

    "main": "src/math/factorial.mo",

Dapp ファイル名の設定を変更しても、`dfx build` コマンドがコンパイルするソースコードを探す場所にしか影響しないことを覚えておいてください。設定ファイルで変更を加えても、ファイルシステム上のファイルやディレクトリの名前は変更されません。メインの Dapp ファイルへのパスやファイル名を変更した場合は、プロジェクトディレクトリ内の名前と場所を変更するようにしてください。

## Dapp フロントエンドを提供する場所を変更する方法

Dapp フロントエンドを提供するためのデフォルトのホスト名とポート番号は、設定ファイル `dfx.json` にあるローカルネットワークの設定を修正することで変更可能です。

例えば、`bind` の設定を変更することで、ローカルネットワークの IP アドレスを変更することができます：

    "networks": {
      "local": {
        "bind": "192.168.47.1:8000",
        "type": "ephemeral"
      }
    }

<!--
# Managing Projects

You can modify some key settings for individual projects by modifying each project’s `dfx.json` configuration file. You can use the `dfx config` command to change these settings programmatically or manually edit the `dfx.json` file directly.

## How to change the source directory

Before you compile source code for your project using the `dfx build` command, you might want to check the default location for storing the source code for your dapp. By default, the name you use to create a new project is the name used for one canister (`canister_name`) and one assets canister (`canister_name_assets`), and dapp source code is expected to be in the `src/canister_name` directory. Similarly, the default location for frontend source code is in the `src/canister_name_assets/src` directory and frontend output is located in the `dist/canister_name_assets` directory.

Depending on your dapp’s complexity and architecture, however, you might want to modify the default location for the dapp source code, the frontend source code, or frontend output.

For example, for a simple dapp, you might want to eliminate one directory level and place the source code in the `src` directory:

      "main": "src/main.mo",

For more complex dapps, you might want to use a multi-tiered directory structure:

    "canisters": {
      "profiles": {
        "main": "src/profiles/utils/main.mo"
      },
      "events": {
        "main": "src/events/calendar/main.mo"
      },
      "media": {
        "main": "src/events/reports/main.mo"
      }
    }

If you modify the default settings for a source code directory, be sure that the settings in the `dfx.json` configuration file match the directory location on the file system.

## How to change the main dapp filename

Before you compile source code for your project using the `dfx build` command, you should verify the location and file name used for your dapp’s source code.

For example, if you want to build a canister for the `factorial` dapp and the source code for the dapp is located in `src/math/factorial.mo`, you should be sure that you have the correct path specified for the `main` setting in the `canisters` section of the configuration file.

For example:

    "main": "src/math/factorial.mo",

Keep in mind that changing the configuration setting for the dapp file name only affects where the `dfx build` command looks for the source code to compile. Making changes in the configuration file does not rename any files or directories on the file system. If you change the path to the main dapp file or the name of the file itself, be sure to change the name and location within your project directory.

## How to change the location for serving the dapp frontend

You can change the default host name and port number for serving the dapp frontend by modifying the local network settings in the `dfx.json` configuration file.

For example, you can change the IP address for the local network by modifying the `bind` setting:

    "networks": {
      "local": {
        "bind": "192.168.47.1:8000",
        "type": "ephemeral"
      }
    }

-->