# Internet Identity の使い方

Internet Identity とは何か、ということから学びたい方は [Internet Identity とは？](what-is-ic-identity)をご覧ください。

Internet Identity アンカーの作成やデバイスの管理を行いたい方は、[Internet Identity Page](https://identity.ic0.app) をご覧ください。

現在サポートされているすべての認証方法は、*WebAuthn* 規格に準拠しています。ただし、以下のような制限があります:

-   OS Xでは、Safari を使った認証は、ブラウザのプロファイルとリンクしています。別のブラウザで Dapp を認証したい場合や、複数の Safari ブラウザのプロファイルを使用している場合は、認証方法と新しいブラウザの組み合わせを、新しいデバイスとして追加する必要があります。詳しくは [`デバイスの追加`](#デバイスの追加) をご覧ください。 なお、OS Xとは異なり、iOSでは、認証はブラウザ間で横断的に機能します。

-   OS Xおよび iOS では、Safari のブラウザ履歴を消去すると、ユーザーが登録した WebAuthn キーが Secure Enclave から削除され、これらのキーを使った認証ができなくなります。

    <div class="warning">

    Identity を必要とする Dapps からロックアウトされないように、リカバリー手段を設定することを強くお勧めします。リカバリー手段の設定方法はこのページの下の方でご説明します。

    </div>

-   Firefox は現在、セキュリティキー以外のデバイス認証方法が OS X ではサポートされていません。

-   Windows Hello 認証は、Chrome、Edge、Firefox でサポートされています。

## Identity Anchor の作成

Identity Anchor を作成し、そこに 1 つ以上のデバイスを追加していれば、ユーザーの認証方法として Internet Identity を使用している Internet Computer の Dapps に安全にアクセスすることができます。認証のために Internet Identity に提供した Identity Anchor に基づいて、Internet Identity はアクセスする各 Dapp に対して異なる偽名を作成します。新しい Identity Anchor を作成することで、必要な数の偽名を作成することができます。

Dapp にアクセスすると、Internet Identity に誘導され、Identity Anchor を入力して認証するよう求められます。Identity Anchor を持っていない場合は、以下の手順で Identity Anchor を作成する必要があります:

1.  "**Create a new Internet Identity Anchor**" をクリックします.

2.  Identity Anchor の作成に使用する認証方法の名前を入力します。例：iPhone、YubiKey など。

3.  認証方法としてデバイスを使用して Identity Anchor を作成します。

    専用のセキュリティキーを使用するか、使用しているデバイスの認証方法を使用するかのどちらの方法で Identity Anchor を作成するかを選択します（画面上に選択肢が出た場合）。

    例えば、デバイスのロック解除に生体認証が有効になっている場合、認証方法としてそれらを使用するオプションが表示されることがあります。また、お使いのデバイスによっては、コンピュータのロックを解除するパスワードや、携帯電話のロックを解除する PIN 番号を使用することもできます。

    <div class="note">

    ベストプラクティスとして、Identity Anchor ごとに少なくとも1つの専用のセキュリティキーを使用してください。 その後、電話やコンピュータなどの他の認証方法や、自分がよく使う第 2 のセキュリティキーを追加することができます。 第 1 のセキュリティキーは、使用するデバイスを使用できない場合に備えて、安全な場所に保管してください。 専用のセキュリティキーを使用すると、Internet Computer 上で動作している任意の Dapp に対して、任意のブラウザを使用して、それを認識する任意のデバイスで認証を行うことができます。  
    セキュリティキーをお持ちでない場合は、シードフレーズからキーを生成し、そのキーをリカバリーメカニズムとして追加することもできます（下記の最後のステップを参照してください）。

    </div>

4.  デバイスを認証してください。

    画面が表示されたら、選択した方法で認証してください。

5.  "**Confirm**" をクリックしてください。

    この手順を実行するまでは、Identity Anchor は作成されません。

    ここで、お使いの機器によっては、「端末認証」または「セキュリティキー」のいずれかを選択する画面が表示されます。初めて登録する場合は、「端末認証」を選択してください。

6.  Identity Anchor を記録してください。

    デバイスの追加が完了すると、Identity Anchor が表示されます。

    Identity Anchor は、固有の番号で表されます。これは秘匿すべき番号ではありませんので、紛失しないように複数の場所に保管してください。 ブラウザは Identity Anchor を記憶していますが、別のコンピュータで認証を行う場合や、ブラウザのプロファイルを変更する場合、またはブラウザの状態をクリアする場合に必要になります。

    <div class="warning">

    Identity Anchor を忘れ、すべてのデバイスからログアウトした場合、シードフレーズを使用してアカウントのリカバリーを設定していない限り、Internet Identity での認証ができなくなります。 Identity Anchor を紛失しないようにしてください。

    </div>

7.  "**Continue**" をクリックしてください。

8.  Identity Anchor へのリカバリ設定を行います。

    複数のデバイスの追加やセキュリティキーの使用に加えて、"**Add a recovery mechanism to an Identity Anchor**" をクリックすると、画面上でアカウントのリカバリーを設定することができます。

    次の画面では、以下のいずれかのオプションを選択できます:

    -   **シードフレーズ**

        このオプションを選択すると、Identity Anchor を復元するために使用できる、暗号的に安全なシードフレーズを生成することができます。 このフレーズは安全な場所に保管し、自分だけが知っているようにしてください。 シードフレーズを知っている人は、この Identity Anchor を完全に制御することができます。 **シードフレーズの最初の文字列が Identity Anchor** であることに注意してください。 復旧作業を開始するには、この番号が必要です。

        <div class="note">

        "**Copy**" ボタンをクリックしてから "**Continue**" ボタンをクリックしないと、シードフレーズが登録されません。

        </div>

    -   **セキュリティキー**

        認可されたデバイスへのアクセスができなくなった場合、Identity Anchor をリカバリするために専用のセキュリティキーを使用します。このキーは、指定された Identity Anchor を使用して Internet Identity を認証する際に頻繁に使用するキーとは異なるものでなければなりません。 このキーは安全な場所に保管し、自分だけが利用できるようにしてください。 上記のとおり、このセキュリティキーを持っている人は、自分の Identity Anchor を完全に制御することができます。 リカバリーを開始するには、Identity Anchor を知っている必要があります。

    -   **リカバリーを後で設定する**

        アカウントのリカバリーメカニズムの設定を省略し、Internet Identity のトップページから後で設定することを選択できます。

        <div class="warning">

        アカウントへのアクセスを失わないように、リカバリーメカニズムを設定することを強くお勧めします。

        </div>

9.  "**Continue**" をクリックしてください。

    次の画面では、Identity Anchor と登録されている認証方法が表示されます。 認証方法の追加や削除、アカウントのリカバリー方法の追加設定を行うことができます。

## デバイスの追加

デバイスを追加するためのワークフローは、Identity Anchor に既に追加したデバイスに依存します。例えば、もしあなたが Identity Anchor を作成するためにコンピュータを最初に認証し、その後携帯電話を新たな認証方法として追加したい場合、認証済みのコンピュータ上で携帯電話を認証する必要があります。 すでに認証されているデバイスを使用し、追加したいデバイスを常に認証できるようにする必要があります。

<div class="note">

Windows Hello 認証をサポートしている Windows デバイス上でデバイスの追加を開始すると、ブラウザは最初に新しい認証方法として Windows Hello を追加するように求めます。Windows Hello 認証をサポートしている Windows デバイス上で新たなデバイスの追加を開始すると、ブラウザは最初に新しい認証方法として Windows Hello を追加するように求めます。すでに Windows Hello でデバイスを登録していて、代わりにセキュリティキーなどを追加したい場合は、Windows Hello の操作画面をキャンセルする必要があります。その後、セキュリティキーなどの別の認証方法をブラウザで選択することができます。

</div>

セキュリティキーなどの新しいデバイスを追加する場合や、すでに認証方法となっているコンピュータや携帯電話を使って新しいブラウザのプロファイルを追加する場合は、Internet Identity Management から直接、簡単に行うことができます。

その他のワークフローはより複雑になります。例えば、認証済みのコンピューターを使って携帯電話のアンロック方法を認証方法として追加するには、以下のような手順となります:

1.  PC　から : Internet Identity のページを開き、ログインします。

2.  PC　から : **add new device** をクリックします。

3.  PC　から : **remote device** を選択します。

    - これにより、この Identity Anchor に対してデバイス登録モードが有効になります。以降の手順は15分以内に完了させる必要があります。

4.  携帯電話から : **Already have an anchor but using a new device?** をクリックします。

5.  携帯電話から : Identity Anchor を入力します。

6.  携帯電話から : デバイスのエイリアスを入力します。

    - このステップを完了すると認証コードが表示されます。

7. 携帯電話から : デバイスの認証コードを入力します。

7.  携帯電話と Identity をリンクさせます。

<div class="warning">

この方法で追加されたデバイスは、あなたの Identity を**完全にコントロールすることができます**。*個人的に所有している*デバイスのみを追加してください。

</div>

<div class="warning">

デバイスを紛失して Dapps へのアクセスができなくなるのを防止するために、できるだけ多くのデバイスを追加しておくべきです。繰り返しになりますが、過ってデバイスを紛失した場合に備え、リカバリー方法を設定することが最善の方法です。また、追加した複数の認証方法のうちの１つの方法で Identity Anchor へのアクセスが可能になるため、追加した認証方法はすべて保管し、紛失しないようにしてください。

</div>

<div class="warning">

デバイスを紛失した場合は、攻撃者が認証方法を追加した可能性を考え、すぐにそのデバイスを認証方法から削除し、すべての認証方法が自分の管理下にあることを確認してください。また、デバイスを紛失してからそのデバイスを認証方法から外すまでの間、Identity Anchor が危険な状態であるということを認識してください。

</div>

## 紛失した Identity の復旧方法

Identity Anchor を作成する際には、シードフレーズをコピーするか、リカバリー方法として専用のセキュリティキーを追加するように画面上で促されます。

これらの作業はいつ行っても良いですが、Identity Anchor を紛失した場合や、認証済みのデバイスにアクセスできなくなった場合には、Identity Anchor を復元するためのシードフレーズかセキュリティキーが必要になりますのでご注意ください。これらがないと、関連する Identity を必要とするすべての Dapps からロックアウトされてしまいます。

Identity Anchor にリカバリーフレーズやセキュリティキーを設定していれば、以下のステップによってアクセスを復旧することができます。

**1. Internet Identity のランディングページから、 **Lost access and want to recover?** をクリックします。**

![welcome to internet identity](../_attachments/welcome-to-internet-identity.png)

**2. Identity Anchor を入力します。**

![recover identity anchor](../_attachments/recover-identity-anchor.png)

**3. シードフレーズを入力します。**

![your seed phrase](../_attachments/your-seed-phrase.png)

<!--
# How to use Internet Identity

If you would like to learn what Internet Identity is, see [What is Internet Identity?](./what-is-ic-identity.md)

If you would like to create an Internet Identity anchor, or manage your devices, go to the [Internet Identity Page](https://identity.ic0.app).

All currently supported authentication methods follow the *WebAuthn* standard. The following restrictions apply, however:

-   On OS X, authentication using Safari is coupled to your browser profile. If you want to authenticate to a dapp in a different browser, or if you use multiple Safari browser profiles, you have to add the combination of your authentication method and the new browser as a new device. See: [`Add a device`](#_add_a_device). Note that on iOS, in contrast to OS X, authentication works across browsers.

-   On OS X and iOS, clearing Safari’s browser history leads to the user’s registered WebAuthn keys being deleted from the secure enclave, and authentication with these keys is no longer possible.

    <div class="warning">

    We highly recommend to set up recovery mechanisms so you won’t be locked out of any dapps that require the associated identity. How a recovery mechanism can be set up is described below.

    </div>

-   Firefox does not currently accept OS X with any device authentication method other than a security key.

-   Windows Hello authentication is supported in Chrome, Edge, and Firefox.

## Create an Identity Anchor

You can securely access dapps that run on the Internet Computer and use Internet Identity for authentication, provided you have created an Identity Anchor and added one or more devices to it. Based on the Identity Anchor you provide to Internet Identity for authentication, it will create a different pseudonym for each dapp that you access for you. You can create as many sets of pseudonyms as you want by creating new Identity Anchors.

When you access a dapp, you are directed to Internet Identity and asked to enter an Identity Anchor to authenticate. If you do not have an Identity Anchor, you need to first create one:

1.  Click **Create a new Internet Identity Anchor**.

2.  Enter a name for the authentication method you would like to use to create an Identity Anchor. For example: iPhone, or YubiKey.

3.  Create the Identity Anchor using your device as an authentication method.

    Choose to create the Identity Anchor using either a dedicated security key, or with an authentication method of the device you are using, if that option is available.

    For example, if your device has biometrics enabled to unlock it, you might see the option to use those as your authentication method. You can also use the password that unlocks your computer or a pin that unlocks your phone, depending on the device you’re using.

    <div class="note">

    As a best practice, use at least one dedicated security key per Identity Anchor. You can then add other authentication methods, such as your phone, your computer, or a second security key you actively use. Store the first key in a safe place for the event that you are unable to to use your preferred device. When you use a dedicated security key, you can authenticate to any dapp running on the Internet Computer using any browser, with any device that recognizes it.  
    If you do not have a security key, you can alternatively also generate a key from a seed phrase and add that key as recovery mechanism (see last step below).

    </div>

4.  Authenticate the device.

    Authenticate using the method you selected when prompted.

5.  Click **Confirm**.

    Your Identity Anchor is not created until you perform this step.

    At this point, depending on the device you are using, you might be asked to either use your device authentication method, or to use your security key. If you are registering for the first time, choose to use the device authentication.

6.  Record your Identity Anchor.

    When your device has been added, you’ll receive an Identity Anchor.

    Your Identity Anchor is represented by a unique number. It is not a secret and you should store it in multiple places so you don’t lose it. Your browser will remember your Identity Anchor, but you will need it when you authenticate on a different computer, change your browser profile, or if you clear your browser state.

    <div class="warning">

    If you forget your Identity Anchor and are logged out of all devices, you will no longer be able to authenticate with Internet Identity, unless you have set up account recovery using a seed phrase in the next step. So don’t lose your Identity Anchor!

    </div>

7.  Click **Continue**.

8.  Add a recovery mechanism to an Identity Anchor

    In addition to adding multiple devices and using security keys, you can set up account recovery at the prompt by clicking **Add a recovery mechanism to an Identity Anchor**.

    On the next screen, you can select one of the following options:

    -   **Seed Phrase**

        Select this option to generate a cryptographically-secure seed phrase that you can use to recover an Identity Anchor. Make sure you store this phrase somewhere safe and it is known only to you, as anyone who knows the seed phrase will be able to take full control of this Identity Anchor. **Note that the first string in your seed phrase is the Identity Anchor**. You will need this number to begin the recovery process.

        <div class="note">

        You must click the **copy** button and then **continue** or the seed phrase will not be registered.

        </div>

    -   **Security Key**

        Use a dedicated security key to recover an Identity Anchor in the event that you lose access to your authorized devices. This key must be different from the ones you actively use to authenticate to Internet Identity using the given Identity Anchor. Keep this key somewhere safe and ensure it is available only to you. As above, anyone in possession of this security key will be able to take full control of your Identity Anchor. You will need to know the Identity Anchor to begin recovery.

    -   **Set recovery later**

        You can skip adding an account recovery mechanism and choose to set it up later from the Internet Identity landing page.

        <div class="warning">

        However, we highly recommend setting up a recovery mechanism so you don’t lose access to this account.

        </div>

9.  Click **Continue**

    On the next screen, you will see your Identity Anchor and your registered authentication methods. From here, you can add and remove authentication methods, and set up additional account recovery methods.

## Add a device

The workflow for adding a device can vary depending on what devices you’ve already added to an Identity Anchor. For example, if you first authorized your computer to create the Identity Anchor, and you’d like to add your phone as a second authentication method, you must be able to authenticate your phone on the authorized computer. You must always be able to authorize the device you want to add by using a device that is already authorized.

<div class="note">

If you start the add device flow on a Windows device that supports Windows Hello authentication, the browser first asks you to add Windows Hello as the new authentication method. If you have registered the device with Windows Hello already and would like to add e.g. a security key instead, you need to cancel the Windows Hello prompt. Then the browser lets you choose a different authentication method, such as a security key.

</div>

If you are adding a new device, such as a new security key, or a new browser profile using a computer or phone that has already been added as an authentication method, you can do this easily and directly from within Internet Identity Management.

Other workflows can be more complex. For example, to add your phone’s unlock methods as an additional authentication method using your authenticated computer, proceed as follows:

1.  On your computer: Open the Internet Identity web page and log in.

2.  On your computer: Click **add new device**

3.  On your computer: choose **remote device**

    -   This enables the device registration mode for this identity anchor. The subsequent steps need to be completed within 15 minutes.

4.  On your phone: click **Already have an anchor but using a new device?**

5.  On your phone: enter your identity anchor

6.  On your phone: enter a device alias

    -   After completing this step a verification code will be shown

7.  On your computer: enter the device verification code

<div class="warning">

Any device added this way will have **full control over your identity**. Only add devices that you *personally own*.

</div>

<div class="warning">

You should add as many devices as possible to prevent you from losing access to dapps in case you lose a device. Again, the best way to deal with accidental loss is to set up a recovery method. Also, make sure to keep all added authentication methods safe and do not lose them, as a single authentication method gives access to the Identity Anchor.

</div>

<div class="warning">

If you lose a device, remove it from the authentication methods immediately and make sure that all added authentication methods are in your control, as an attacker may have added more methods in the meanwhile. Also, consider the Identity Anchor compromised starting from the time the device was lost until it was removed from the authentication methods.

</div>

## How to recover a lost identity

When you create an Identity Anchor, you will be prompted to copy a seed phrase or to add a dedicated security key as recovery mechanism.

You can choose to do this at any time, but note that if you lose an Identity Anchor or if you no longer have access to authorized devices, you will need the seed phrase or the recovery security key to recover the Identity Anchor. Without one of these, you will be locked out of any dapps that require the associated identity.

If you have set up a recovery phrase or recovery security key for an Identity Anchor, you can regain access by following these steps:

**1. Click **Lost access and want to recover?** from the Internet Identity landing page.**

![welcome to internet identity](../_attachments/welcome-to-internet-identity.png)

**2. Input your identity anchor**

![recover identity anchor](../_attachments/recover-identity-anchor.png)

**3. Input your seed phrase**

![your seed phrase](../_attachments/your-seed-phrase.png)

-->