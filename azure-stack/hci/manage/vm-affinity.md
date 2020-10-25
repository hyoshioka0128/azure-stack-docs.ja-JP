---
title: Windows PowerShell を使用して VM アフィニティ ルールを設定する
description: Windows PowerShell を使用して VM アフィニティ ルールを設定する方法について説明します
author: v-dasis
ms.topic: how-to
ms.date: 10/14/2020
ms.author: v-dasis
ms.reviewer: JasonGerend
ms.openlocfilehash: e2cb3fa0efb11664924431ed0434547c832ceab4
ms.sourcegitcommit: 8122672409954815e472a5b251bb7319fab8f951
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/14/2020
ms.locfileid: "92060161"
---
# <a name="create-server-and-site-affinity-rules-for-vms"></a>VM のサーバーとサイトのアフィニティ ルールを作成する

> 適用対象: Azure Stack HCI バージョン 20H2

Windows Admin Center または Windows PowerShell のいずれかを使用すると、クラスター内の仮想マシン (VM) に対してアフィニティ ルールとアンチアフィニティ ルールを簡単に作成できます。

アフィニティとは、仮想マシン (VM) などの 2 つ以上のリソース グループまたはロール間に関係を確立し、それらを同じサーバー、クラスター、またはサイトにまとめて保持するためのルールです。 アンチアフィニティは、指定されたリソース グループを相互に分離するために使用されるという点で対照的です。たとえば、ディザスター リカバリー用に別々のサーバーまたは別々のサイトに配置された 2 つのドメイン コントローラーなどがあります。

アフィニティ ルールとアンチアフィニティ ルールは、Azure での Availability Zones の使用と同様の方法で使用されます。 Azure では、VM を別々のゾーンに配置すること、相互に分離すること、または同じゾーンに配置するように Availability Zones を構成することができます。  

アフィニティ ルールとアンチアフィニティ ルールを使用すると、クラスター化された VM を同じクラスター ノード内に配置するか、同じクラスター ノード内に一緒に配置されないようにすることができます。  この方法の場合、VM をノードから移動するには、手動で移動する必要があります。  また、VHDX が存在するクラスター共有ボリューム (CSV) など、独自のストレージと一緒に VM を保持することもできます。

アフィニティ ルールとアンチアフィニティ ルールを組み合わせることで、2 つのサイトにまたがってストレッチ クラスターを構成し、配置する必要があるサイトに VM を保持することもできます。

## <a name="using-windows-admin-center"></a>Windows Admin Center を使用する

Windows Admin Center を使用して、基本的なアフィニティおよびアンチアフィニティのルールを作成できます。

:::image type="content" source="media/vm-affinity/vm-affinity-rules.png" alt-text="仮想マシンの画面" lightbox="media/vm-affinity/vm-affinity-rules.png":::

1. Windows Admin Center のホームの **[すべての接続]** で、VM ルールを作成するサーバーまたはクラスターを選択します。
1. **[ツール]** の **[設定]** を選択します。
1. **[設定]** で **[アフィニティ ルール]** を選択し、 **[アフィニティ ルール]** の下で **[ルールの作成]** を選択します。
1. **[ルール名]** の下にルールの名前を入力します。
1. [ルールの種類] で、VM を同じサーバーに配置する場合は **[Together (same server)]\(まとめて (同じサーバー)\)** 、別のサーバーに配置する場合は **[Apart (different servers)]\(別々 (異なるサーバー)\)** を選択します。
1. **[適用対象]** で、このルールを適用する VM を選択します。 ルールにさらに VM を追加するには、 **[追加]** ボタンを使用します。
1. 完了したら、 **[ルールの作成]** をクリックします。
1. ルールを削除するには、それを選択して **[ルールの削除]** をクリックするだけです。

## <a name="using-windows-powershell"></a>Windows PowerShell を使用する

Windows PowerShell を使用すると、Windows Admin Center を使用した場合よりさらに複雑なルールを作成することができます。 通常、ルールは、クラスター内のホスト サーバーではなく、リモート コンピューターから管理します。 このリモート コンピューターは、管理コンピューターと呼ばれます。

管理コンピューターから Windows PowerShell コマンドを実行する場合は、管理しているクラスターの名前を指定した `-Name` パラメーターまたは `-Cluster` パラメーターを含めます。 必要に応じて、サーバー ノードに `-ComputerName` パラメーターを使用するときに完全修飾ドメイン名 (FQDN) も指定します。

### <a name="new-powershell-cmdlets"></a>新しい PowerShell コマンドレット

クラスターのアフィニティ ルールを作成するには、次の新しい PowerShell コマンドレットを使用します。

#### <a name="new-clusteraffinityrule"></a>New-ClusterAffinityRule

新しいルールを作成するには、 **`New-ClusterAffinityRule`** コマンドレットを使用します。  このコマンドでは、ルールの名前とルールの種類を指定します。次のように指定します。

**`-Name`** はルールの名前です

**`-RuleType`** 値は `SameFaultDomain` | `SameNode` | `DifferentFaultDomain` | `DifferentNode` です

例:

```powershell
New-ClusterAffinityRule -Name -RuleType SameFaultDomain -Cluster Cluster1
```

#### <a name="set-clusteraffinityrule"></a>Set-ClusterAffinityRule

**`Set-ClusterAffinityRule`** コマンドレットは、ルールを有効または無効にするために使用されます。次のように指定します。

**`-Name`** は、有効または無効にするルールの名前です

**`-Enabled`**  |  **`Disabled`** を使用してルールを有効または無効にします

例:

```powershell
Set-ClusterAffinityRule -Name -Enabled -Cluster Cluster1
```

#### <a name="get-clusteraffinityrule"></a>Get-ClusterAffinityRule

**`Get-ClusterAffinityRule`** コマンドレットは、指定されたルールとその種類を表示するために使用されます。  **`-Name`** が指定されていない場合は、すべてのルールが一覧表示されます。

例:

```powershell
Get-ClusterAffinityRule -Name -Cluster Cluster1
```

#### <a name="add-clustergrouptoaffinityrule"></a>Add-ClusterGroupToAffinityRule

**`Add-ClusterGroupToAffinityRule`** コマンドレットは、特定のアフィニティ ルールに VM ロールまたはグループの名前を追加するために使用されます。次のように指定します。

**`-Groups`** は、ルールに追加するグループまたはロールの名前です

**`-Name`** は、追加するルールの名前です

例:

```powershell
Add-ClusterGroupToAffinityRule -Groups -Name -Cluster Cluster1
```

#### <a name="add-clustersharedvolumetoaffinityrule"></a>Add-ClusterSharedVolumeToAffinityRule

**`Add-ClusterSharedVolumeToAffinityRule`** を使用すると、VHDX が存在するクラスター共有ボリュームと共に VM を維持できます。次のように指定します。

**`-ClusterSharedVolumes`** は、ルールに追加する CSV ディスクです

**`-Name`** は、追加するルールの名前です

例:

```powershell
Add-ClusterSharedVolumeToAffinityRule  -ClusterSharedVolumes -Name -Cluster Cluster1
```

#### <a name="remove-clusteraffinityrule"></a>Remove-ClusterAffinityRule

**`Remove-ClusterAffinityRule`** を使用すると、指定したルールを削除できます。 **`-Name`** はルールの名前です。

例:

```powershell
Remove-ClusterAffinityRule -Name -Cluster Cluster1
```

#### <a name="remove-clustergroupfromaffinityrule"></a>Remove-ClusterGroupFromAffinityRule

**`Remove-ClusterGroupFromAffinityRule`** を使用すると、特定のルールから VM グループまたはロールを削除できます。ただし、ルールは無効化または削除されません。次のように指定します。

**`-Name`** はルールの名前です

**`-Groups`** は、ルールから削除するグループまたはロールです

例:

```powershell
Remove-ClusterGroupFromAffinityRule -Name -Groups -Cluster Cluster1
```

#### <a name="remove-clustersharedvolumefromaffinityrule"></a>Remove-ClusterSharedVolumeFromAffinityRule

**`Remove-ClusterSharedVolumeFromAffinityRule`** コマンドレットは、特定のルールからクラスター共有ボリュームを削除するために使用されます。ただし、ルールは無効化または削除されません。次のように指定します。

**`-ClusterSharedVolumes`** は、ルールから削除する CSV ディスクです

**`-Name`** は、追加するルールの名前です

例:

```powershell
Remove-ClusterSharedVolumeFromAffinityRule -ClusterSharedVolumes -Name -Cluster Cluster1
```

### <a name="existing-powershell-cmdlets"></a>既存の PowerShell コマンドレット

新しいコマンドレットが登場したことで、一部の既存のコマンドレットに新しいスイッチが追加されました。

#### <a name="move-clustergroup"></a>Move-ClusterGroup

新しい `-IgnoreAffinityRule` スイッチを使用すると、ルールを無視し、クラスター化されたリソース グループを別のクラスター ノードに移動することができます。 このコマンドレットの詳細については、「[Move-ClusterGroup](/powershell/module/failoverclusters/move-clustergroup)」を参照してください。

例:

```powershell
Move-ClusterGroup -IgnoreAffinityRule -Cluster Cluster1
```

> [!NOTE]
> 移動ルールが有効である (サポートされている) 場合、影響を受けるすべてのグループとロールも移動されます。  VM の移動がルールに違反するとわかっていても、1 回限り、一時的に必要な場合は、`-IgnoreAffinityRule` スイッチを使用して移動を許可します。 この場合、VM に対する違反の警告が表示されます。 その後、必要に応じてルールを有効に戻すことができます。

#### <a name="start-clustergroup"></a>Start-ClusterGroup

新しい `-IgnoreAffinityRule` スイッチを使用すると、ルールを無視し、クラスター化されたリソース グループを現在の場所でオンラインにすることができます。 このコマンドレットの詳細については、「[Start-ClusterGroup](/powershell/module/failoverclusters/start-clustergroup)」を参照してください。

例:

```powershell
Start-ClusterGroup -IgnoreAffinityRule -Cluster Cluster1
```

## <a name="affinity-rule-examples"></a>アフィニティ ルールの例

アフィニティ ルールは、同じサーバー、クラスター、またはサイトにリソースを保持する "まとめる" ルールです。 アフィニティ ルールを設定するための一般的なシナリオをいくつか次に示します。

### <a name="scenario-1"></a>シナリオ 1

SQL サーバー VM と Web サーバー VM があるとします。 これらの 2 つの VM は常に同じサイトに存在する必要がありますが、必ずしもサイト内の同じクラスター ノード上に存在する必要はありません。  次に示すように、`SameFaultDomain` を使用すると、これを実行できます。

```powershell
New-ClusterAffinityRule -Name WebData -Ruletype SameFaultDomain -Cluster Cluster1

Add-ClusterGroupToAffinityRule -Groups SQL1,WEB1 –Name WebData -Cluster Cluster1

Set-ClusterAffinityRule -Name WebData -Enabled 1 -Cluster Cluster1
```

このルールとその構成方法を確認するには、 **`Get-ClusterAffinityRule`** コマンドレットを使用して出力を確認します。

```powershell
Get-ClusterAffinityRule -Name WebData -Cluster Cluster1

Name        RuleType          Groups        Enabled
----        ---------         ------        -------
WebData     SameFaultDomain   {SQL1, WEB1}     1
```

### <a name="scenario-2"></a>シナリオ 2

上記と同じシナリオを使用します。ただし、VM が同じクラスター ノード上に存在する必要があり、同じサイトに存在する必要はないことを指定する点が異なります。 `SameNode` を使用すると、次のように設定できます。

```powershell
New-ClusterAffinityRule -Name WebData1 -Ruletype SameNode -Cluster Cluster1

Add-ClusterGroupToAffinityRule -Groups SQL1,WEB1 –Name WebData1 -Cluster Cluster1

Set-ClusterAffinityRule -Name WebData1 -Enabled 1 -Cluster Cluster1
```

ルールとその構成方法を確認するには、 **`Get-ClusterAffinityRule`** コマンドレットを使用して出力を確認します。

```powershell
Get-ClusterAffinityRule -Name WebData1 -Cluster Cluster1

Name    RuleType    Groups        Enabled
----    --------    ------        -------
DC      SameNode    {SQL1, WEB1}     1
```

## <a name="anti-affinity-rule-examples"></a>アンチアフィニティ ルールの例

アンチアフィニティ ルールは、リソースを分離し、それらを異なるサーバー、クラスター、またはサイトに配置する "分割" ルールです。

### <a name="scenario-1"></a>シナリオ 1
同じ Azure Stack HCI マルチサイト クラスター上でそれぞれ SQL Server を実行する 2 つの VM があります。  各 VM には、大量のメモリ、CPU、ストレージ リソースが使用されます。  同じノード上の 2 つが終了すると、メモリ、CPU、およびストレージ サイクルの競合が生じるため、いずれかまたは両方でパフォーマンスの問題が発生する可能性があります。  ルールの種類として `DifferentNode` を指定したアンチアフィニティ ルールを使用すると、これらの VM は常に異なるクラスター ノードにとどまります。  

この例のコマンドは次のようになります。

```powershell
New-ClusterAffinityRule -Name SQL -Ruletype DifferentNode -Cluster Cluster1

Add-ClusterGroupToAffinityRule -Groups SQL1,SQL2 –Name SQL -Cluster Cluster1

Set-ClusterAffinityRule -Name SQL -Enabled 1 -Cluster Cluster1
```

ルールとその構成方法を確認するには、 **`Get-ClusterAffinityRule`** コマンドレットを使用して出力を確認します。

```powershell
Get-ClusterAffinityRule -Name SQL -Cluster Cluster1

Name    RuleType        Groups        Enabled
----    -----------     -------       -------
SQL     DifferentNode   {SQL1, SQL2}     1
```

### <a name="scenario-2"></a>シナリオ 2

たとえば、2 つのサイト (障害ドメイン) を持つ Azure Stack HCI ストレッチ クラスターがあるとします。 別々のサイトに保持したいドメイン コントローラーが 2 つあります。 ルールの種類として `DifferentFaultDomain` を指定したアンチアフィニティ ルールを使用すると、これらのドメイン コントローラーは常に別のサイトにとどまります。  この例のコマンドは次のようになります。

```powershell
New-ClusterAffinityRule -Name DC -Ruletype DifferentFaultDomain -Cluster Cluster1

Add-ClusterGroupToAffinityRule -Groups DC1,DC2 –Name DC -Cluster Cluster1

Set-ClusterAffinityRule -Name DC -Enabled 1 -Cluster Cluster1
```

このルールとその構成方法を確認するには、 **`Get-ClusterAffinityRule`** コマンドレットを使用して出力を確認します。

```powershell
Get-ClusterAffinityRule -Name DC -Cluster Cluster1

Name    RuleType                Groups        Enabled
----    --------                -------       -------
DC      DifferentFaultDomain    {DC1, DC2}       1
```

## <a name="combined-rule-examples"></a>結合されたルールの例

アフィニティ ルールとアンチアフィニティ ルールを組み合わせると、マルチサイト クラスター全体でさまざまな VM の組み合わせを簡単に構成できます。  このシナリオでは、各サイトに 3 つの VM があります。SQL Server (SQL)、Web サーバー (WEB)、およびドメイン コントローラー (DC) です。  組み合わせごとに、アフィニティ ルールと `SameFaultDomain` を使用して、すべてを同じサイトに保持することができます。  また、次に示すように、各サイトのドメイン コントローラーにアンチアフィニティ ルールと `DifferentFaultDomain` を設定して、ドメイン コントローラー VM を別々のサイトに保持することもできます。

```powershell
New-ClusterAffinityRule -Name Site1Trio -Ruletype SameFaultDomain -Cluster Cluster1

New-ClusterAffinityRule -Name Site2Trio -Ruletype SameFaultDomain -Cluster Cluster1

New-ClusterAffinityRule -Name TrioApart -Ruletype DifferentFaultDomain -Cluster Cluster1

Add-ClusterGroupToAffinityRule -Groups SQL1,WEB1,DC1 –Name Site1Trio -Cluster Cluster1

Add-ClusterGroupToAffinityRule -Groups SQL2,WEB2,DC2 –Name Site2Trio -Cluster Cluster1

Add-ClusterGroupToAffinityRule -Groups DC1,DC2 –Name TrioApart -Cluster Cluster1

Set-ClusterAffinityRule -Name Site1Trio -Enabled 1 -Cluster Cluster1

Set-ClusterAffinityRule -Name Site2Trio -Enabled 1 -Cluster Cluster1

Set-ClusterAffinityRule -Name TrioApart -Enabled 1 -Cluster Cluster1
```

ルールとその構成方法を確認するには、`-Name` スイッチを指定せずに **`Get-ClusterAffinityRule`** コマンドレットを使用すると、作成されたすべてのルールとその出力を確認できます。

```powershell
Get-ClusterAffinityRule -Cluster Cluster1

Name        RuleType               Groups            Enabled
----        --------               ------            -------
Site1Trio   SameFaultDomain        {SQL1, WEB1, DC1}    1
Site2Trio   SameFaultDomain        {SQL2, WEB2, DC2}    1
TrioApart   DifferentFaultDomain   {DC1, DC2}           1
```

## <a name="storage-affinity-rules"></a>ストレージ アフィニティ ルール

VM とその VHDX を同じクラスター ノード上のクラスター共有ボリューム (CSV) に保持することもできます。 これにより、CSV リダイレクトが発生しないようになり、VM の起動または停止が遅くなる可能性があります。  以前のアフィニティとアンチアフィニティを組み合わせたシナリオを考慮して、SQL VM とクラスター共有ボリュームを同じクラスター ノード上に保持することができます。  これを行うには、次のコマンドを使用します。

```powershell
New-ClusterAffinityRule -Name SQL1CSV1 -Ruletype SameNode -Cluster Cluster1

New-ClusterAffinityRule -Name SQL2CSV2 -Ruletype SameNode -Cluster Cluster1

Add-ClusterGroupToAffinityRule -Groups SQL1 –Name SQL1CSV1 -Cluster Cluster1

Add-ClusterGroupToAffinityRule -Groups SQL2 –Name SQL2CSV2 -Cluster Cluster1

Add-ClusterSharedVolumeToAffinityRule -ClusterSharedVolumes CSV1 -Name SQL1CSV1 -Cluster Cluster1

Add-ClusterSharedVolumeToAffinityRule -ClusterSharedVolumes CSV2 -Name SQL2CSV2 -Cluster Cluster1

Set-ClusterAffinityRule -Name SQL1CSV1 -Enabled 1 -Cluster Cluster1

Set-ClusterAffinityRule -Name SQL2CSV2 -Enabled 1 -Cluster Cluster1
```

これらのルールとその構成方法を確認するには、-Name スイッチを指定せずに **`Get-ClusterAffinityRule`** コマンドレットを使用して、出力を確認します。

```powershell
Get-ClusterAffinityRule -Cluster Cluster1

Name        RuleType               Groups            Enabled
----        --------               ------            -------
Site1Trio   SameFaultDomain        {SQL1, WEB1, DC1}    1
Site2Trio   SameFaultDomain        {SQL2, WEB2, DC2}    1
TrioApart   DifferentFaultDomain   {DC1, DC2}           1
SQL1CSV1    SameNode               {SQL1, <CSV1-GUID>}  1
SQL2CSV2    SameNode               {SQL2, <CSV2-GUID>}  1
```

## <a name="next-steps"></a>次のステップ

VM を管理する方法について確認します。 「[Windows Admin Center を使用して Azure Stack HCI 上の VM を管理する](../manage/vm.md)」を参照してください。
