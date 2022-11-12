# Neuron のライフサイクル

Neuron は、Internet Computer のガバナンス Canister によって管理されるアセットの一種です。Neuron は、そのコントローラーがプロポーザルを提出し、投票することによって、ネットワークのガバナンスに参加することができます。

## Neuron の生成

Neuron を作るには2つのステップがあります：

-   任意の台帳のアカウントから、ある** ICP 数量**を Neuron のアドレスに転送します。台帳上のNeuron アドレスはガバナンス Canister に属しています。呼び出し側は、2つの部分から Neuron アドレスを計算します。

    -   ガバナンス Canister の Principal。

    -   **サブアカウント**：コントローラーの Principal のハッシュと整数の nonce です。サブアカウントの計算に異なる整数 nonce を選ぶことで、ひとつの Principal が複数の Neurons を制御することができます。

-   ガバナンス Canister の `claim_or_refresh_neuron_from_account` メソッドをコールします。呼び出しに成功すると、ペイロードに新しく作成された Neuron の数値識別子である **Neuron ID** が含まれます。ガバナンス Canister は Neuron ID とサブアカウント間のマッピングを保持します。ほとんどの管理操作は、Neuron ID またはサブアカウントアドレスのいずれかを使用して実行することができます。

## Neuron の属性

 Neuron は以下の属性を持ち、ライフサイクルと報酬の分配を制御します：

溶解遅延（Dissolve Delay）  
溶解遅延は、Neuron が溶解を開始してから「流動性」（liquid）になるまでの時間です。すでに溶解開始している場合、遅延時間は Neuron が **DISSOLVED** 状態に遷移するまでの残り時間を表します。新しく作成された Neuron の溶解遅延は 0 です。溶解遅延が 15,778,800秒（約6ヶ月）以上ある Neuron だけが、プロポーザルの投票に使用できます（正確な遅延要件は変更される可能性があります：2021-07-27現在）。一度設定すると、管理コマンドで溶解遅延を減らすことはできません。

溶解状態（Dissolve State）  
NOT_DISSOLVING  
Neuron は固定の溶解遅延を持ち、**エイジ（age）** を発生させます。これは新しく作成された Neuron のデフォルトの状態です。

DISSOLVING  
Neuron の溶解遅延は、時間の経過とともに減少しています。溶解遅延が 0 になると、Neuron は **DISSOLVED** 状態に遷移し、張り付けた ICP を払い出すことができるようになります。

DISSOLVED  
Neuron の溶解遅延は 0 になっています。Neuron ホルダーは、張り付けた ICP を払い出すか、溶解遅延を増加させるか、自由に行うことができ、これにより **NOT_DISSOLVING** 状態へ遷移します。

エイジ（Age）  
エイジ（Age）は、Neuron が最後に **NOT_DISSOLVING** 状態に遷移してから経過した時間です。Neuron が **DISSOLVING** である場合、そのエイジは 0 になります。

成熟度（Maturity）  
成熟度は、この Neuron で発生した未使用の報酬の総量です。Neuron のコントローラーは、2つの方法で成熟度を消費することができます：

1.  **産出（Spawn）** は溶解遅延が（DISSOLVED 状態で）7日あり、成熟度と同じステーク量で、新しい Neuron を産出します。

2.  **マージ（Merge）** は成熟度を再ステークし、議決権を増加させます。

投票力（Voting power）  
投票力は、張り付けられた **ICP 数**に比例します。エイジと溶解遅延から計算されたボーナスは、Neuron が **NOT_DISSOLVING** 状態の場合、投票力を増大させます。

## 報酬

プロポーザルが受け入れられると、そのプロポーザルに投票した全ての Neuron は成熟度の上昇という形で報酬を受け取ることができます。ガバナンス Canister は毎日報酬を分配します。報酬は投票力に比例します。

Neuron の状態遷移と報酬については、[Internet Computer’s Network Nervous System, Neurons, and ICP Utility Tokens を理解する](https://medium.com/dfinity/understanding-the-internet-computers-network-nervous-system-neurons-and-icp-utility-tokens-730dab65cae8) を参照してください。

## 状態遷移

    stateDiagram-v2
        state "Non-dissolving Neuron" as non_dissolving
        state "Dissolving Reward Neuron" as dissolving
        state "Dissolved Reward Neuron" as dissolved
        [*] --> non_dissolving : transfer + claim_or_refresh_neuron_from_account
        non_dissolving --> non_dissolving : increase_dissolve_delay
        non_dissolving --> dissolving : start_dissolving
        dissolving --> non_dissolving : stop_dissolving
        dissolving --> dissolved : passage of time
        dissolved --> non_dissolving: increase_dissolve_delay
        dissolved --> [*] : disburse

<!--
# Neuron lifecycle

A neuron is a type of asset managed by the Internet Computer governance canister. Neurons allow their controllers to participate in the governance of the network by submitting and voting on proposals.

## Creating a neuron

Creating a neuron involves 2 steps:

-   Transfer some **amount** of ICPs from any ledger account to the neuron’s address. Neuron address on the ledger that belongs to the Governance canister. The caller computes the neuron address from two parts:

    -   the principal of the Governance canister

    -   the **subaccount** that is itself a hash of the controller’s principal with an integer nonce. One principal can control multiple neurons by picking different integer nonces for the subaccount computation.

-   Call the `claim_or_refresh_neuron_from_account` method of the governance canister. If the call is successful, the payload contains the **neuron ID**, a numeric identifier of the newly created neuron. The governance canister maintains a mapping between neuron IDs and subaccounts. Most management operations can be performed using either the neuron ID or the subaccount address.

## Neuron attributes

A neuron has the following attributes that control its lifecycle and rewards distribution:

Dissolve Delay  
is how long it will take a neuron to become "liquid" once it starts dissolving. If the neuron is already dissolving, the delay indicates the amount of time left before the neuron transitions to the **DISSOLVED** state. The dissolve delay of a newly created neuron is 0. Only neurons with a dissolve delay of at least 15778800 seconds (approx 6 months) can be used to vote on proposals (as of 2021-07-27, the exact delay requirements are subject to change). Once set, the dissolve delay cannot be reduced via management commands.

Dissolve State  
NOT_DISSOLVING  
Neuron has a fixed dissolve delay and accrues **age**. This is the default state of a newly created neuron.

DISSOLVING  
Neuron’s dissolve delay is decreasing with the passage of time. Once the dissolve delay becomes 0, the neuron transitions to the **DISSOLVED** state and the staked ICPs can be disbursed.

DISSOLVED  
Neuron’s dissolve delay is 0. The neuron holder is free to either disburse the staked ICPs or increase the dissolve delay, which will cause a transition to the **NOT_DISSOLVING** state.

Age  
is the amount of time that passed since the last time the neuron transitioned to the **NOT_DISSOLVING** state. If the neuron is **DISSOLVING**, its age is 0.

Maturity  
is the total amount of unspent rewards accrued by this neuron. Neuron’s controller can spend maturity in two ways:

1.  **Spawn** a new neuron with a dissolve delay of seven days and stake equal to the maturity.

2.  **Merge** maturity into the stake, increasing voting power.

Voting power  
is proportional to the **amount** of staked ICPs. Age and dissolve delay multiplicative bonuses increase the voting power if the neuron is in the **NOT_DISSOLVING** state.

## Rewards

Once a proposal is accepted, all the neurons that voted on that proposal receive rewards in form of a maturity increase. The governance canister distributes rewards daily. The rewards are proportional to the voting power.

For more information on neuron state transitions and rewards, see [Understanding the Internet Computer’s Network Nervous System, Neurons, and ICP Utility Tokens](https://medium.com/dfinity/understanding-the-internet-computers-network-nervous-system-neurons-and-icp-utility-tokens-730dab65cae8).

## State transitions

    stateDiagram-v2
        state "Non-dissolving Neuron" as non_dissolving
        state "Dissolving Reward Neuron" as dissolving
        state "Dissolved Reward Neuron" as dissolved
        [*] ==> non_dissolving : transfer + claim_or_refresh_neuron_from_account
        non_dissolving ==> non_dissolving : increase_dissolve_delay
        non_dissolving ==> dissolving : start_dissolving
        dissolving ==> non_dissolving : stop_dissolving
        dissolving ==> dissolved : passage of time
        dissolved ==> non_dissolving: increase_dissolve_delay
        dissolved ==> [*] : disburse

-->
<!--126, 127, 128, 129, 130, 131, 132 の==>はコメントアウトの関係からーー＞（敢えて全角にしています）にしています-->