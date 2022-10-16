# DFX を Windows にインストールする

Windows 上では `dfx` はネイティブにサポートされていません。しかし、Windows Subsystem for Linux (WSL) をインストールすることによって、以下に述べるように `dfx` を Windows システム上でも動作させることができます。

## WSL をインストールする

マイクロソフト社の説明書に従って、[Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install) をインストールします。Windows 10（Ver.2004 以降）または Windows 11 が動作していることを確認してください。

### サポートされている WSL のバージョン

理論的には、WSL1 と WSL2 はどちらも `dfx` を実行できるはずです。しかし、我々は WSL2 を推奨します。[WSL比較](https://docs.microsoft.com/en-us/windows/wsl/compare-versions) では、WSL1 とWSL2 の違いについて説明しています。

### WSL バージョンををチェックする

`wsl -list -verbose (wsl -l -v)` というコマンドを実行すると、Windows マシンにインストールされている Linux ディストリビューションを確認することができます。以下は出力例です。

```
  NAME      STATE           VERSION
* Ubuntu    Running         2
```

`wsl` コマンドの詳細については、[コマンドリファレンス for WSL](https://docs.microsoft.com/en-us/windows/wsl/basic-commands) を参照してください。


### WSL2 にアップグレードする

WSL1 をインストールしている場合は、[アップグレード手順](https://docs.microsoft.com/en-us/windows/wsl/install#upgrade-version-from-wsl-1-to-wsl-2) に従ってWSL2 にアップグレードしてください。基本的には以下の作業が必要です：
* [WSL2 Linux カーネルアップデートパッケージ](https://docs.microsoft.com/en-us/windows/wsl/install-manual#step-4---download-the-linux-kernel-update-package) をインストールします。
* 以下のコマンドを実行し、Linux ディストリビューションをバージョン2 に設定します。 
  `wsl --set-version <distribution name> 2`

## Linux を起動する

WSLをインストールした後、Linux ディストリビューションの名前で起動することができます。

例えば、`Ubuntu.exe` は、コマンドラインから `Ubuntu` ディストリビューションを起動するコマンドです。

## DFX をインストールする

WSL をインストールしたら、[SDKのインストール](../build/install-upgrade-remove) にあるように、WSL の Linux 端末内に `dfx` をインストールすることができます。

## トラブルシューティング

### Node.js が正しくインストールされない
WSL2 では、デフォルトで node.js `10.x.x` がインストールされています。しかし、最新の `dfx` には `16.0.0` 以降の node.js が必要です。詳しくは [Node.js](hello10mins#nodejs) を確認してください。

### `dfx start` の実行時にパーミッションが拒否される
`dfx` から作成するプロジェクトは、Windows のファイルシステムではなく、Linux のファイルシステム上にある必要があります。通常、WSL ターミナルで `cd ~` または `cd $HOME` を実行すると、ホームディレクトリに移動し、そこにプロジェクトを作成することができます。

### WSL でインターネットに接続できない
WSL でインターネットにアクセスできない場合、例えば、どのサーバーにもうまく ping を打てない場合、おそらく WSL 上のネームサーバーは、WSL 内部のプロキシネームサーバーに設定されています。その場合は、`/etc/resolv.conf` ファイルを確認することができます。もし、そうであれば、以下の手順で、有効なネームサーバーに設定してください。
* `/etc/wsl.conf` ファイルを作成し、以下の内容を追加します。これにより、WSL が再起動後に `/etc/resolv.conf` ファイルを再生成するのを防ぐことができます。
  ```
  [network]
  generateResolvConf = false
  ```
* `/etc/resolv.conf` ファイルのネームサーバーを、例えば google のネームサーバー `8.8.8.8` のように、有効なものに変更します。
* Windowsでは、この修正を有効にするために WSL を再起動します。 
   `wsl.exe --shutdown` 

<!--
# Installing DFX on Windows

There is no native support for `dfx` on Windows. However, by installing the Windows Subsystem for Linux (WSL), you can run `dfx` also on a Windows system as described below.

## Installing WSL

Follow Microsoft's instructions for installing the [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install). Make sure you are running Windows 10 (version 2004 or higher) or Windows 11.

### Supported WSL Versions

Theoretically, WSL 1 and WSL 2 should both allow you to run `dfx`. However, we recommend WSL 2. [WSL Comparison](https://docs.microsoft.com/en-us/windows/wsl/compare-versions) explains the differences between WSL1 and WSL 2.

### Check your WSL version

Run the command `wsl –list –verbose (wsl -l -v)` to check the Linux distributions installed on your Windows machine. Below is an example output.

```
  NAME      STATE           VERSION
* Ubuntu    Running         2
```

To learn more about the `wsl` command, check the [command reference for WSL](https://docs.microsoft.com/en-us/windows/wsl/basic-commands).


### Upgrade to WSL 2

If you have WSL 1 installed, follow the [upgrade instructions](https://docs.microsoft.com/en-us/windows/wsl/install#upgrade-version-from-wsl-1-to-wsl-2) to upgrade to WSL 2. Basically you need to: 
* Install the [WSL 2 Linux kernel update package](https://docs.microsoft.com/en-us/windows/wsl/install-manual#step-4---download-the-linux-kernel-update-package).
* Run the following command to set your Linux distributions to version 2.  
  `wsl --set-version <distribution name> 2`

## Running Linux

After you have WSL installed, you can launch the Linux distributions by name.

For example `Ubuntu.exe` is the command to start the `Ubuntu` distribution from the command line.

## Installing DFX

Once you have WSL installed, you can install `dfx` within your WSL Linux terminal as described in [Installing the SDK](../build/install-upgrade-remove).

## Troubleshooting

### Node.js is not properly installed
WSL 2 has node.js `10.x.x` installed by default. But the latest `dfx` requires node.js `16.0.0` or higher, please check [Node.js](hello10mins#nodejs) for more information.

### Permission Denied when running `dfx start`
Projects created from `dfx` need to be on the Linux filesystem instead of the Windows filesystem. Usually `cd ~` or `cd $HOME` in the WSL terminal will bring you to the home directory, and creating projects in there should work.

### No internet access on WSL
If you don't have internet access on WSL, for instance you cannot ping any server successfully, probably the nameserver on WSL is set to an internal WSL proxy nameserver. You can check the `/etc/resolv.conf` file to see if it's the case. If it's true, please follow the below steps to set to a valid nameserver:
* Create the `/etc/wsl.conf` file and add the below content to it, this will prevent WSL from regenerating the `/etc/resolv.conf` file after restarting.
  ```
  [network]
  generateResolvConf = false
  ```
* Modify the nameserver in the `/etc/resolv.conf` file to a valid one, for example the google nameserver `8.8.8.8`.
* On Windows, restart WSL to let this fix take effect.  
   `wsl.exe --shutdown`

-->