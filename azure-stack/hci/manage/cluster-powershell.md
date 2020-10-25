---
title: PowerShell を使用して Azure Stack HCI クラスターを管理する
description: PowerShell を使用して Azure Stack HCI でクラスターを管理する方法について説明します
author: v-dasis
ms.topic: how-to
ms.date: 07/21/2020
ms.author: v-dasis
ms.reviewer: JasonGerend
ms.openlocfilehash: 88f6ace707d14466de481f60133875968b8edce0
ms.sourcegitcommit: a15a0f955bac922cebb7bf90a72384fd84ddfe56
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/22/2020
ms.locfileid: "86947437"
---
# <a name="manage-azure-stack-hci-clusters-using-powershell"></a>PowerShell を使用して Azure Stack HCI クラスターを管理する

> 適用対象: Azure Stack HCI バージョン 20H2、Windows Server 2019

Windows PowerShell を使用して、Azure Stack HCI クラスター上のリソースを管理し、機能を構成することができます。

クラスターは、クラスター内のホスト サーバーではなく、リモート コンピューターから管理します。 このリモート コンピューターは、管理コンピューターと呼ばれます。

> [!NOTE]
> 管理コンピューターから PowerShell コマンドを実行する場合は、管理しているクラスターの名前を指定した `-Name` パラメーターまたは `-Cluster` パラメーターを含めます。 さらに、サーバー ノードに対して `-ComputerName` パラメーターを使用する場合は、完全修飾ドメイン名 (FQDN) を指定する必要があります。

PowerShell を使用したクラスターの管理に関する完全なリファレンス ドキュメントについては、[FailoverCluster リファレンス](/powershell/module/failoverclusters/?view=win10-ps)を参照してください。

## <a name="using-windows-powershell"></a>Windows PowerShell を使用する

この記事のすべてのタスクを実行するには、Windows PowerShell を使用します。 利便性のために、このアプリをタスク バーにピン留めすることをお勧めします。

以下のコマンドレットが PowerShell セッションで使用できない場合は、PowerShell コマンド `Add-WindowsFeature RSAT-Clustering-PowerShell` を使用して、Windows PowerShell 機能の `Failover Cluster` モジュールを追加することが必要な場合があります。

> [!NOTE]
> Windows 10 October 2018 Update 以降では、RSAT は Windows 10 から直接 "オンデマンド機能" のセットとして含まれています。 **[設定] > [アプリ] > [アプリと機能] > [オプション機能] > [RSAT: フェールオーバー クラスタリング ツール]** に移動し、 **[インストール]** を選択します。 インストールの進行状況を確認するには、[戻る] ボタンをクリックして、[オプション機能の管理] ページに状態を表示します。 インストールされた機能は、Windows 10 のバージョンのアップグレードをまたがって保持されます。

## <a name="view-cluster-settings-and-resources"></a>クラスター設定およびリソースを表示する

Cluster1 という名前のクラスターに関する情報を取得します。

```powershell
Get-Cluster -Name Cluster1
```
Cluster1 の 1 つ以上のノード (サーバー) に関する情報を取得します。

```powershell
Get-ClusterNode -Cluster Cluster1
```

クラスター ノードにインストールされている Windows 機能を表示するには、`Get-WindowsFeature` コマンドレットを使用します。 次に例を示します。

```powershell
Get-WindowsFeature -ComputerName Server1
```

ネットワーク アダプターとそれらのプロパティ (名前、IPv4 アドレス、VLAN ID など) を表示するには、次を実行します。

```powershell
Get-NetAdapter -CimSession Server1 | Where Status -Eq "Up" | Sort InterfaceAlias | Format-Table Name, InterfaceDescription, Status, LinkSpeed, VLANID, MacAddress
```

Hyper-V 仮想スイッチと、物理ネットワーク アダプターのチーミング方法を表示するには、次を実行します。

```powershell
Get-VMSwitch -ComputerName Server1
```

ホスト仮想ネットワークアダプターを表示するには、次を実行します。

```powershell
Get-VMNetworkAdapter -ComputerName Server1
```

記憶域スペース ダイレクトが有効になっているかどうかを確認するには、次を実行します。

```powershell
Get-CimSession -ComputerName Server1 | Get-ClusterStorageSpacesDirect
```

## <a name="start-or-stop-a-cluster"></a>クラスターを開始または停止する

クラスターに対してサーバー ノードを追加または削除するには、`Start-Cluster` と `Stop-Cluster` のコマンドレットを使用します。 その他の例と使用方法に関する情報については、「[Start-Cluster](/powershell/module/failoverclusters/start-cluster?view=win10-ps)」と「[Stop-Cluster](/powershell/module/failoverclusters/stop-cluster?view=win10-ps)」のリファレンス ドキュメントを参照してください。

まだクラスター サービスが開始されていないクラスターのすべてのサーバー ノードでそのサービスを開始します。

```powershell
Start-Cluster -Name Cluster1
```

次の例では、Cluster1 という名前のクラスター内のすべてのノードでクラスター サービスを停止します。これにより、クラスターに構成されているすべてのサービスおよびアプリケーションが停止します。

```powershell
Stop-Cluster -Name Cluster1
```

## <a name="add-or-remove-a-server"></a>サーバーを追加または削除する

クラスターに対してサーバー ノードを追加または削除するには、`Add-ClusterNode` と `Remove-ClusterNode` のコマンドレットを使用します。 その他の例と使用方法に関する情報については、「[Add-ClusterNode](/powershell/module/failoverclusters/add-clusternode?view=win10-ps)」と「[Remove-ClusterNode](/powershell/module/failoverclusters/remove-clusternode?view=win10-ps)」のリファレンス ドキュメントを参照してください。

この例では、Node4 という名前のサーバーを Cluster1 という名前のクラスターに追加します。 最初に、サーバーが実行されていて、クラスター ネットワークに接続されていることを確認してください。

```powershell
Add-ClusterNode -Cluster Cluster1 -Name Node4
```

次の例では、Node4 という名前のノードをクラスター Cluster1 から削除します。

```powershell
Remove-ClusterNode -Cluster Cluster1 -Name Node4
```

## <a name="setup-the-cluster-witness"></a>クラスター監視をセットアップする

`Set-ClusterQuorum` コマンドレットを使用して、クラスターのクォーラム監視オプションを設定します。 その他の例と使用方法に関する情報については、「[Set-ClusterQuorum](/powershell/module/failoverclusters/set-clusterquorum?view=win10-ps)」のリファレンス ドキュメントを参照してください。

次の例では、クラスター Cluster1 でクラウド監視を使用するようにクォーラム構成を変更します。

```powershell
Set-ClusterQuorum -Cluster Cluster1 -CloudWitness
```

この例では、ファイル共有監視用に \\fileserver\fsw のディスク リソースを使用して、クラスター Cluster1 でクォーラム構成をノードおよびファイル共有マジョリティに変更します。

```powershell
Set-ClusterQuorum -Cluster Cluster1 -NodeAndFileShareMajority \\fileserver\fsw
```

## <a name="enable-storage-spaces-direct"></a>記憶域スペース ダイレクトを有効にする

クラスターで記憶域スペース ダイレクトを有効にするには、`Enable-ClusterStorageSpacesDirect` コマンドレットを使用します。 その他の例と使用方法に関する情報については、「[Enable-ClusterStorageSpacesDirect](/powershell/module/failoverclusters/enable-clusterstoragespacesdirect?view=win10-ps)」のリファレンス ドキュメントを参照してください。

次の例では、Server1 で記憶域スペース ダイレクトを有効にします。

```powershell
Enable-ClusterStorageSpacesDirect -CimSession Cluster1
```

## <a name="configure-a-hyper-v-host"></a>Hyper-V ホストを構成する

VHD および VM のパス、ライブ マイグレーション、記憶域移行、認証、NUMA ノードにまたがるメモリ割り当てなど、さまざまな Hyper-V ホスト設定を構成するには、`Set-VMHost` コマンドレットを使用します。 その他の例と使用方法に関する情報については、「[Enable-ClusterStorageSpacesDirect](/powershell/module/hyper-v/set-vmhost?view=win10-ps)」のリファレンス ドキュメントを参照してください。

次の例では、ホスト サーバー Server1 上の仮想ハード ディスクおよび VM の新しい既定の場所を指定します。

```powershell
Set-VMHost -ComputerName Server1 -VirtualHardDiskPath "C:\Hyper-V\Virtual Hard Disks" -VirtualMachinePath "C:\Hyper-V\Configuration Files"
```

次の例では、ライブ マイグレーションと記憶域移行を同時に 10 個まで許可するように、ホスト サーバー Server1 を構成します。

```powershell
Set-VMHost -ComputerName Server1 -MaximumVirtualMachineMigrations 10 -MaximumStorageMigrations 10
```

次の例では、Kerberos を使用して受信ライブ マイグレーションを認証するように、ホスト サーバー Server1 を構成します。

```powershell
Set-VMHost -ComputerName Server1 -VirtualMachineMigrationAuthenticationType Kerberos
```

## <a name="remove-cluster-and-resources"></a>クラスターとリソースを削除する

クラスター上の 1 つまたはすべてのリソースを削除するには、`Remove-ClusterResource` コマンドレットを使用します。 その他の例と使用方法に関する情報については、「[Remove-ClusterResource](/powershell/module/failoverclusters/remove-clusterresource?view=win10-ps)」のリファレンス ドキュメントを参照してください。

> [!NOTE]
> クラスターを削除するには、Credential Security Service Provider (CredSSP) 認証を一時的に有効にする必要があります。 詳細については、「[Enable-WSManCredSSP](/powershell/module/microsoft.wsman.management/enable-wsmancredssp?view=powershell-7)」を参照してください。

次の例では、クラスター Cluster1 上のクラスター リソースを名前で削除します。

```powershell
Remove-ClusterResource -Cluster Cluster1 -Name "Cluster Disk 4"
```

次の例では、`Remove-Cluster` コマンドレットを使用して、クラスター Cluster1 を完全に削除します。

```powershell
Remove-Cluster -Cluster Cluster1
```

## <a name="next-steps"></a>次のステップ

- 変更を行った後で、クラスターを検証する必要があります。 詳細については、「[Azure Stack HCI クラスターの検証](../deploy/validate.md)」を参照してください。
- Windows Admin Center を使用してクラスターを管理する方法を確認します。 [Windows Admin Center を使用した Azure Stack HCI でのクラスターの管理](cluster.md)に関する記事を参照してください。
