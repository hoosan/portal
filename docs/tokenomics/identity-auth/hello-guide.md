# Internet Identity のための Windows Hello ガイド

Internet Identity は、認証方法として Windows Hello をサポートしています。 このガイドでは、携帯電話またはセキュリティキーを使用して設定された既存の Identity Anchor に、Windows Hello 認証を設定する方法を説明します。 設定方法は、以下のオプション A と B でそれぞれ説明します。

## Windows マシンが Windows Hello に対応しているか確認する

Windowsの「設定」を開き、「アカウント」を選択します。

![settings accounts](../_attachments/settings-accounts.png)

次に、「サインインオプション」を選択します。

![settings sign in](../_attachments/settings-sign-in.png)

お使いのデバイスがサインインのために Windows Hello をサポートしていることを確認してください。

![settings hello](../_attachments/settings-hello.png)

お使いのデバイスが Windows Hello に対応していれば、次に進みます。 携帯電話を使用して Identity Anchor を設定した場合はオプション A、セキュリティキーを使用した場合はオプション B に進みます。

## オプション A: 携帯電話を認証手段として使用する Identity Anchor に Windows Hello を追加する

携帯電話で <https://identity.ic0.app> を開き、ログインします。

![anchor management](../_attachments/anchor-management.png)

"Remote Device" を選択してください。

![add remote device](../_attachments/add-remote-device.png)

Windows パソコンで <https://identity.ic0.app> にアクセスし、"Already have anchor but using new device?" をクリックしてください。

![add device start](../_attachments/add-device-start.png)

Identity Anchor を入力します。

![add device anchor](../_attachments/add-device-anchor.png)

Windows デバイスの名前を入力します。

![enter alias](../_attachments/enter-alias.png)

Windows Hello の認証を行い、Windows Hello ダイアログを完成させてください。

![add device hello](../_attachments/add-device-hello.png)

検証コードが表示されます。

![display verification code](../_attachments/display-verification-code.png)

携帯電話に検証コードを入力します。

![enter verification code](../_attachments/enter-verification-code.png)

携帯電話で Windows PC の検証を実施した後、Windows PC でWindows Hello を使用して認証できるようになります。

![login hello](../_attachments/login-hello.png)

## オプション B: セキュリティキーを認証方法として使用する Identity Anchor に Windows Hello を追加する

Windows パソコンで <https://identity.ic0.app> にアクセスし、セキュリティキーを使って認証すると、アンカーマネジメントのページが表示されます。 このページで "+ ADD NEW DEVICE" をクリックします。

![anchor management](../_attachments/anchor-management.png)

"Local Device" を選択します。

![add local device](../_attachments/add-local-device.png)

Windows Hello ダイアログを完成させてください。

![management add device dialog](../_attachments/management-add-device-dialog.png)

Windowsマシンの名前をつけます。

![management add device name](../_attachments/management-add-device-name.png)

ページをリフレッシュして更新すると、Windows Hello での認証ができるようになります。

![login hello](../_attachments/login-hello.png)

<!--
# Windows Hello Guide for Internet Identity

Internet Identity supports Windows Hello as an authentication method. This guide explains how to set up Windows Hello authentication for an existing Identity Anchor that was set up either on your phone or using a security key. The setup is explained below in Options A and B, respectively.

## Checking if your Windows machine supports Windows Hello

Open your Windows Settings, and select "Accounts"

![settings accounts](../_attachments/settings-accounts.png)

Then select "Sign-in options"

![settings sign in](../_attachments/settings-sign-in.png)

and check your device supports Windows Hello for signing in

![settings hello](../_attachments/settings-hello.png)

If your device supports Windows Hello we can continue. Follow Option A if you’ve set-up an Identity Anchor using your phone or Option B if you’ve used a security key.

## Option A: Adding Windows Hello to an Identity Anchor that uses your phone as authentication method

On your phone go to <https://identity.ic0.app> and log in

![anchor management](../_attachments/anchor-management.png)

Select "Remote Device"

![add remote device](../_attachments/add-remote-device.png)

Now switch to your Windows computer and go to <https://identity.ic0.app> and click on "Already have an Anchor but using a new device?"

![add device start](../_attachments/add-device-start.png)

Enter your Identity Anchor

![add device anchor](../_attachments/add-device-anchor.png)

Enter a name for the Windows device

![enter alias](../_attachments/enter-alias.png)

Complete the Windows Hello dialog by authenticating using Windows Hello

![add device hello](../_attachments/add-device-hello.png)

A verification code will be displayed

![display verification code](../_attachments/display-verification-code.png)

Switch back to your phone and enter the verification code

![enter verification code](../_attachments/enter-verification-code.png)

After verifying the Windows computer on your phone you should be able to authenticate on your Windows machine using Windows Hello

![login hello](../_attachments/login-hello.png)

## Option B: Adding Windows Hello to an Identity Anchor that uses your security key as authentication method

On your Windows computer go to <https://identity.ic0.app> and authenticate using your security key to reach the Anchor Management page. Once you’re there click on "+ ADD NEW DEVICE".

![anchor management](../_attachments/anchor-management.png)

Select "Local Device"

![add local device](../_attachments/add-local-device.png)

Complete the Windows Hello dialog

![management add device dialog](../_attachments/management-add-device-dialog.png)

and choose a name for your Windows machine

![management add device name](../_attachments/management-add-device-name.png)

If you refresh the page, you should now be able to authenticate with Windows Hello

![login hello](../_attachments/login-hello.png)

-->