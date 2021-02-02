---
title: Windows PowerShell を使用して Azure Stack HCI クラスターを作成する
description: Windows PowerShell を使用して Azure Stack HCI 用のクラスターを作成する方法について説明します
author: v-dasis
ms.topic: how-to
ms.date: 01/22/2021
ms.author: v-dasis
ms.reviewer: JasonGerend
ms.openlocfilehash: 2099d7e9dcd2d01f949d54ad5bd59ce06ecaccbc
ms.sourcegitcommit: e772df8ac78c86d834a68d1a8be83b7f738019b7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/26/2021
ms.locfileid: "98772204"
---
# <a name="create-an-azure-stack-hci-cluster-using-windows-powershell"></a>Windows PowerShell を使用して Azure Stack HCI クラスターを作成する

> 適用対象:Azure Stack HCI バージョン v20H2

この記事では、Windows PowerShell を使用して、記憶域スペース ダイレクトを使用する Azure Stack HCI ハイパーコンバージド クラスターを作成する方法について説明します。 Windows Admin Center のクラスター作成ウィザードを使用してクラスターを作成する場合は、[Windows Admin Center でのクラスターの作成](create-cluster.md)に関する記事を参照してください。

2 種類のクラスターのどちらかを選択できます。

- すべてが単一のサイトに存在する、少なくとも 2 つのサーバー ノードを持つ標準クラスター。
- 2 つのサイトにまたがり、少なくとも 4 つのサーバー ノードを持つストレッチ クラスター。サイトごとに 2 つのノードがあります。

この記事では、Server1、Server2、Server3、Server4 という名前の 4 つのサーバー ノードで構成される Cluster1 という名前のクラスターの例を作成します。

ストレッチ クラスターのシナリオでは、ClusterS1 を名前として使用し、サイト Site1 と Site2 にストレッチされた同じ 4 つのサーバー ノードを使用します。

ストレッチ クラスターの詳細については、「[ストレッチ クラスターの概要](../concepts/stretched-clusters.md)」を参照してください。

Azure Stack HCI のテストには興味があるものの、ハードウェアに限りがある、または予備のハードウェアがない場合には、[Azure Stack HCI の評価ガイド](https://github.com/Azure/AzureStackHCI-EvalGuide/blob/main/README.md)をご覧ください。オンプレミスにある単一の物理システムまたは Azure で、入れ子になった仮想化を使用して Azure Stack HCI を体験することができます。

## <a name="before-you-begin"></a>開始する前に

開始する前に次の点を確認します。

- [Azure Stack HCI のシステム要件](../concepts/system-requirements.md)を確認します。
- Azure Stack HCI の[物理ネットワークの要件](../concepts/physical-network-requirements.md)と[ホスト ネットワークの要件](../concepts/host-network-requirements.md)に関するページを読みます。
- クラスター内の各サーバーに Azure Stack HCI OS をインストールします。 「[Azure Stack HCI オペレーティング システムのデプロイ](operating-system.md)」を参照してください。
- 各サーバーでローカルの Administrators グループのメンバーであるアカウントを用意します。
- Active Directory でオブジェクトを作成する権限を付与します。
- ストレッチ クラスターの場合は、あらかじめ 2 つのサイトを Active Directory に設定します。

## <a name="using-windows-powershell"></a>Windows PowerShell を使用する

PowerShell は、ホスト サーバー上の RDP セッションでローカルに実行することも、管理コンピューターからリモートで実行することもできます。 この記事では、リモート オプションについて説明します。

管理コンピューターから PowerShell を実行する場合は、管理しているサーバーまたはクラスターの名前を指定した `-Name` パラメーターまたは `-Cluster` パラメーターを含めます。 さらに、サーバー ノードに対して `-ComputerName` パラメーターを使用する場合は、完全修飾ドメイン名 (FQDN) を指定することが必要な場合があります。

また、Hyper-V およびフェールオーバー クラスタリング用の、リモート サーバー管理ツール (RSAT) コマンドレットと PowerShell モジュールも必要になります。 管理コンピューターの PowerShell セッションでこれらをまだ使用できない場合は、次のコマンドを使用して追加できます: `Add-WindowsFeature RSAT-Clustering-PowerShell`。

## <a name="step-1-provision-the-servers"></a>手順 1:サーバーをプロビジョニングする

最初に、各サーバーに接続し、それらをドメイン (管理コンピューターと同じドメイン) に参加させて、必要な役割と機能をインストールします。

### <a name="step-11-connect-to-the-servers"></a>手順 1.1: サーバーに接続する

サーバーに接続するには、まずネットワーク接続があり、同じドメインまたは完全に信頼されたドメインに参加していて、サーバーに対するローカル管理権限を持っている必要があります。

PowerShell を開き、接続先のサーバーの完全修飾ドメイン名または IP アドレスを使用します。 各サーバーで次のコマンドを実行すると、パスワードの入力を求められます。 

この例では、サーバーの名前は Server1、Server2、Server3、Server4 であるものと想定しています。

   ```powershell
   Enter-PSSession -ComputerName "Server1" -Credential "Server1\Administrator"
   ```

同じことを行うもう 1 つの例を次に示します。

   ```powershell
   $myServer1 = "Server1"
   $user = "$myServer1\Administrator"

   Enter-PSSession -ComputerName $myServer1 -Credential $user
   ```

> [!TIP]
> 管理 PC から PowerShell コマンドを実行すると、"*WinRM は要求を処理できません*" のようなエラーが表示されることがあります。 これを解決するには、PowerShell を使用して、管理コンピューターの信頼されたホストの一覧に各サーバーを追加します。 この一覧では、`Server*` などのワイルドカードがサポートされています。
>
> `Set-Item WSMAN:\Localhost\Client\TrustedHosts -Value Server1 -Force`
>  
> 信頼されたホストの一覧を表示するには、「`Get-Item WSMAN:\Localhost\Client\TrustedHosts`」と入力します。  
>
> 一覧を空にするには、「`Clear-Item WSMAN:\Localhost\Client\TrustedHost`」と入力します。  

### <a name="step-12-join-the-domain-and-add-domain-accounts"></a>手順 1.2: ドメインに参加してドメイン アカウントを追加する

ここまで、ローカル管理者アカウント `<ServerName>\Administrator` を使用して各サーバー ノードに接続しました。

続行するには、サーバーをドメインに参加させ、すべてのサーバーのローカルの Administrators グループにあるドメイン アカウントを使用する必要があります。

`Enter-PSSession` コマンドレットを使用して各サーバーに接続し、次のコマンドレットを実行します。サーバー名、ドメイン名、ドメイン資格情報は置き換えてください。

```powershell  
Add-Computer -NewName "Server1" -DomainName "contoso.com" -Credential "Contoso\User" -Restart -Force  
```

管理者アカウントが Domain Admins グループのメンバでない場合は、各サーバーのローカルの Administrators グループに管理者アカウントを追加します。または、さらによい方法としては、管理者に使用するグループを追加します。 これを行うには、次のコマンドを使用します。

```powershell
Add-LocalGroupMember -Group "Administrators" -Member "king@contoso.local"
```

### <a name="step-13-install-roles-and-features"></a>手順 1.3: 役割と機能をインストールする

次のステップでは、クラスターのすべてのサーバーに必要な Windows の役割と機能をインストールします。 インストールする役割は次のとおりです。

- BitLocker
- データ センター ブリッジング (RoCEv2 ネットワーク アダプター用)
- フェールオーバー クラスタリング
- ファイル サーバー
- FS-Data-Deduplication モジュール
- Hyper-V
- RSAT-AD-PowerShell モジュール
- ストレージ レプリカ (ストレッチ クラスターの場合)

各サーバーに対して次のコマンドを使用します。

```powershell
Install-WindowsFeature -ComputerName "Server1" -Name "BitLocker", "Data-Center-Bridging", "Failover-Clustering", "FS-FileServer", "Hyper-V", "Hyper-V-PowerShell", "RSAT-AD-Powershell", "RSAT-Clustering-PowerShell", "Storage-Replica" -IncludeAllSubFeature -IncludeManagementTools
```

クラスター内のすべてのサーバーでコマンドを同時に実行するには、次のスクリプトを使用して、最初に環境に合わせて変数の一覧を変更します。

```powershell
# Fill in these variables with your values
$ServerList = "Server1", "Server2", "Server3", "Server4"
$FeatureList = "BitLocker", "Data-Center-Bridging", "Failover-Clustering", "FS-FileServer", "Hyper-V", "Hyper-V-PowerShell", "RSAT-AD-Powershell", "RSAT-Clustering-PowerShell", "Storage-Replica"

# This part runs the Install-WindowsFeature cmdlet on all servers in $ServerList, passing the list of features in $FeatureList.
Invoke-Command ($ServerList) {
    Install-WindowsFeature -Name $Using:Featurelist -IncludeAllSubFeature -IncludeManagementTools
}
```
次に、すべてのサーバーを再起動します。

```powershell
$ServerList = "Server1", "Server2", "Server3", "Server4"
Restart-Computer -ComputerName $ServerList -WSManAuthentication Kerberos
```

## <a name="step-2-configure-networking"></a>手順 2:ネットワークを構成する

このステップでは、仮想スイッチやネットワーク アダプターなどのさまざまなネットワーク要素を環境内で構成します。 RDMA (iWARP と RoCE の両方) ネットワーク アダプターがサポートされています。

Azure Stack HCI の RDMA および Hyper-V ホスト ネットワークの詳細については、[ホスト ネットワークの要件](../concepts/host-network-requirements.md)に関する記事を参照してください。

### <a name="disable-unused-networks"></a>未使用のネットワークを無効にする

切断されているか、管理、ストレージ、またはワークロードのトラフィック (VM など) に使用されていないネットワークを、無効にする必要があります。 未使用のネットワークを識別する方法を次に示します。

```powershell
$ServerList = "Server1", "Server2", "Server3", "Server4"
Get-NetAdapter -CimSession $ServerList | Where-Object Status -eq Disconnected
```
これらを無効にする方法は次のとおりです。

```powershell
Get-NetAdapter -CimSession $ServerList | Where-Object Status -eq Disconnected | Disable-NetAdapter -Confirm:$False
```

### <a name="create-virtual-switches"></a>仮想スイッチを作成する

クラスター内の各サーバー ノードには、仮想スイッチが必要です。 次の例では、接続されているネットワーク アダプター (状態がアップ) を使用して、SR-IOV 機能を備えた仮想スイッチが作成されます。 また、SR-IOV は RDMA 対応の vmNIC (VM 用の vNIC) に必要なので、有効にすると役に立つ場合があります。

複数の NIC をチーミングするには、すべてのネットワーク アダプターが同一である必要があります。

```powershell
$ServerList = "Server1", "Server2", "Server3", "Server4"
$vSwitchName="vSwitch"
```

仮想スイッチを作成するには:

```powershell
Invoke-Command -ComputerName $ServerList -ScriptBlock {New-VMSwitch -Name $using:vSwitchName -EnableEmbeddedTeaming $TRUE -EnableIov $true -NetAdapterName (Get-NetAdapter | Where-Object Status -eq Up ).InterfaceAlias}
```

ここで、スイッチが正常に作成されたことを確認します。

```powershell
$ServerList = "Server1", "Server2", "Server3", "Server4"
Get-VMSwitch -CimSession $ServerList | Select-Object Name, IOVEnabled, IOVS*
Get-VMSwitchTeam -CimSession $ServerList
```

### <a name="assign-virtual-network-adapters"></a>仮想ネットワーク アダプターを割り当てる

次に、以下の例のように、管理用および残りのトラフィック用の仮想ネットワーク アダプター (vNIC) を割り当てます。 クラスター管理用のネットワーク アダプターを少なくとも 1 つ構成する必要があります。

```powershell
$ServerList = "Server1", "Server2", "Server3", "Server4"
$vSwitchName="vSwitch"
Rename-VMNetworkAdapter -ManagementOS -Name $vSwitchName -NewName Management -ComputerName $ServerList
Add-VMNetworkAdapter -ManagementOS -Name SMB01 -SwitchName $vSwitchName -CimSession $ServerList
Add-VMNetworkAdapter -ManagementOS -Name SMB02 -SwitchName $vSwitchName -Cimsession $ServerList
```

正常に追加され、割り当てられていることを確認します。

```powershell
$ServerList = "Server1", "Server2", "Server3", "Server4"
Get-VMNetworkAdapter -CimSession $ServerList -ManagementOS
```

### <a name="configure-ip-addresses-and-vlans"></a>IP アドレスと VLAN を構成する

1 つまたは 2 つのサブネットを構成できます。 スイッチの相互接続が過負荷にならないようにするには、2 つのサブネットをお勧めします。 たとえば、SMB ストレージ トラフィックでは、1 つの物理スイッチに専用のサブネットが常に使用されます。

> [!NOTE]
> IP アドレスの構成中、新しい IP アドレスを取得する間に仮想アダプターのいずれかに接続される可能性があるため、接続が数分間中断される場合があります。

#### <a name="obtain-network-interface-information"></a>ネットワーク インターフェイスの情報を取得する

ネットワーク インターフェイス カードの IP アドレスを設定するには、その前にまず、インターフェイス インデックス (`ifIndex`)、`Interface Alias`、`Address Family` など、必要な情報がいくつかあります。 これらは後で必要になるため、各サーバー ノードについて記録しておきます。

次のコマンドレットを実行します。

```powershell
$ServerList = "Server1", "Server2", "Server3", "Server4"
Get-NetIPInterface -CimSession $ServerList
```

#### <a name="configure-one-subnet"></a>1 つのサブネットを構成する

```powershell
$ServerList = "Server1", "Server2", "Server3", "Server4"
$StorNet="172.16.1."
$StorVLAN=1
$IP=1 #starting IP Address

#Configure IP Addresses
foreach ($Server in $ServerList){
    New-NetIPAddress -IPAddress ($StorNet+$IP.ToString()) -InterfaceAlias "vEthernet (SMB01)" -CimSession $Server -PrefixLength 24
    $IP++
    New-NetIPAddress -IPAddress ($StorNet+$IP.ToString()) -InterfaceAlias "vEthernet (SMB02)" -CimSession $Server -PrefixLength 24
    $IP++
}

#Configure VLANs
Set-VMNetworkAdapterVlan -VMNetworkAdapterName SMB01 -VlanId $StorVLAN -Access -ManagementOS -CimSession $ServerList
#Restart each host vNIC adapter so that the Vlan is active.
Restart-NetAdapter "vEthernet (SMB01)" -CimSession $ServerList
Restart-NetAdapter "vEthernet (SMB02)" -CimSession $ServerList
```

#### <a name="configure-two-subnets"></a>2 つのサブネットを構成する

```powershell
$ServerList = "Server1", "Server2", "Server3", "Server4"
$StorNet1="172.16.1."
$StorNet2="172.16.2."
$StorVLAN1=1
$StorVLAN2=2
$IP=1 #starting IP Address

#Configure IP Addresses
foreach ($Server in $ServerList){
    New-NetIPAddress -IPAddress ($StorNet1+$IP.ToString()) -InterfaceAlias "vEthernet (SMB01)" -CimSession $Server -PrefixLength 24
    New-NetIPAddress -IPAddress ($StorNet2+$IP.ToString()) -InterfaceAlias "vEthernet (SMB02)" -CimSession $Server -PrefixLength 24
    $IP++
}

#Configure VLANs
Set-VMNetworkAdapterVlan -VMNetworkAdapterName SMB01 -VlanId $StorVLAN1 -Access -ManagementOS -CimSession $ServerList
Set-VMNetworkAdapterVlan -VMNetworkAdapterName SMB02 -VlanId $StorVLAN2 -Access -ManagementOS -CimSession $ServerList
#Restart each host vNIC adapter so that the Vlan is active.
Restart-NetAdapter "vEthernet (SMB01)" -CimSession $Servers
Restart-NetAdapter "vEthernet (SMB02)" -CimSession $Servers
```

#### <a name="verify-vlan-ids-and-subnets"></a>VLAN ID とサブネットを検証する

```powershell
$ServerList = "Server1", "Server2", "Server3", "Server4"
#verify ip config
Get-NetIPAddress -CimSession $ServerList -InterfaceAlias vEthernet* -AddressFamily IPv4 | Sort-Object -Property PSComputername | ft pscomputername,interfacealias,ipaddress -AutoSize -GroupBy PSComputerName

#Verify that the VlanID is set
Get-VMNetworkAdapterVlan -ManagementOS -CimSession $ServerList | Sort-Object -Property Computername | Format-Table ComputerName,AccessVlanID,ParentAdapter -AutoSize -GroupBy ComputerName
```

## <a name="step-3-prep-for-cluster-setup"></a>手順 3:クラスターのセットアップを準備する

次に、サーバーでクラスタリングの準備ができていることを確認します。 最初にサニティ チェックとして、次のコマンドを実行して、サーバーがまだクラスターに属していないことを確認することを検討します。

`Get-ClusterNode` を使用してすべてのノードを表示します。

```powershell
Get-ClusterNode
```

`Get-ClusterResource` を使用してすべてのクラスター ノードを表示します。

```powershell
Get-ClusterResource
```

`Get-ClusterNetwork` を使用してすべてのクラスター ネットワークを表示します。

```powershell
Get-ClusterNetwork
```

### <a name="step-31-prepare-drives"></a>手順 3.1: ドライブを準備する

記憶域スペース ダイレクトを有効にする前に、ドライブが空であることを確認します。 次のスクリプトを実行して、古いパーティションまたは他のデータをすべて削除します。

```powershell
# Fill in these variables with your values
$ServerList = "Server1", "Server2", "Server3", "Server4"

Invoke-Command ($ServerList) {
    Update-StorageProviderCache
    Get-StoragePool | ? IsPrimordial -eq $false | Set-StoragePool -IsReadOnly:$false -ErrorAction SilentlyContinue
    Get-StoragePool | ? IsPrimordial -eq $false | Get-VirtualDisk | Remove-VirtualDisk -Confirm:$false -ErrorAction SilentlyContinue
    Get-StoragePool | ? IsPrimordial -eq $false | Remove-StoragePool -Confirm:$false -ErrorAction SilentlyContinue
    Get-PhysicalDisk | Reset-PhysicalDisk -ErrorAction SilentlyContinue
    Get-Disk | ? Number -ne $null | ? IsBoot -ne $true | ? IsSystem -ne $true | ? PartitionStyle -ne RAW | % {
        $_ | Set-Disk -isoffline:$false
        $_ | Set-Disk -isreadonly:$false
        $_ | Clear-Disk -RemoveData -RemoveOEM -Confirm:$false
        $_ | Set-Disk -isreadonly:$true
        $_ | Set-Disk -isoffline:$true
    }
    Get-Disk | Where Number -Ne $Null | Where IsBoot -Ne $True | Where IsSystem -Ne $True | Where PartitionStyle -Eq RAW | Group -NoElement -Property FriendlyName
} | Sort -Property PsComputerName, Count
```

### <a name="step-32-test-cluster-configuration"></a>手順 3.2: クラスターの構成をテストする

この手順では、サーバー ノードがクラスター作成用に正しく構成されていることを確認します。 `Test-Cluster` コマンドレットを使用してテストを実行し、構成がハイパーコンバージド クラスターとして機能するのに適していることを確認します。 次の例では、`-Include` パラメーターを使用して、特定のカテゴリのテストを指定しています。 これにより、正しいテストが検証に含まれるようになります。

```powershell
Test-Cluster –Node $ServerList –Include "Storage Spaces Direct", "Inventory", "Network", "System Configuration"
```

## <a name="step-4-create-the-cluster"></a>手順 4:クラスターを作成する

前の手順で検証したサーバー ノードを使用して、クラスターを作成できる状態になっています。

クラスターを作成するときに、`"There were issues while creating the clustered role that may prevent it from starting. For more information, view the report file below."` (クラスター化された役割の作成中に問題があったため、役割を開始できない可能性があります。詳細については、以下のレポート ファイルを参照してください。) という警告が表示されます。この警告は無視しても問題ありません。 これは、後で作成するクラスター監視に使用できるディスクがないことが原因です。 

> [!NOTE]
> サーバーで静的 IP アドレスが使用されている場合は、次のコマンドを変更し、パラメーター `–StaticAddress <X.X.X.X>;` を追加して IP アドレスを指定することにより、静的 IP アドレスを反映させます。

```powershell
$ClusterName="cluster1" New-Cluster -Name $ClusterName –Node $ServerList –nostorage
```

これでクラスターが作成されました。

クラスターが作成された後、ドメイン全体に DNS 経由でクラスター名がレプリケートされるまで時間がかかることがあります。特に、Active Directory にワークグループ サーバーが新たに追加されている場合は時間がかかります。 Windows Admin Center にクラスターが表示される場合がありますが、まだ接続することはできません。

すべてのクラスター リソースがオンラインであることを確認するための適切なチェック:

```powershell
Get-Cluster -Name $ClusterName | Get-ClusterResource
```

しばらくしてもクラスターの解決が成功しない場合は、クラスター名ではなく、いずれかのクラスター化サーバーの名前を使用することでほとんどの場合、接続できます。

## <a name="step-5-set-up-sites-stretched-cluster"></a>手順 5:サイトをセットアップする (ストレッチ クラスター)

このタスクは、2 つのサイト間にストレッチ クラスターを作成する場合にのみ適用されます。 

> [!NOTE]
> Active Directory のサイトとサービスを事前にセットアップしてある場合は、以下の説明に従って手動でサイトを作成する必要はありません。

### <a name="step-51-create-sites"></a>手順 5.1:サイトを作成する

次のコマンドレットで、*FaultDomain* は単にサイトの別の名前です。 この例では、ストレッチ クラスターの名前として "ClusterS1" を使用します。

```powershell
New-ClusterFaultDomain -CimSession "ClusterS1" -FaultDomainType Site -Name "Site1"
```

```powershell
New-ClusterFaultDomain -CimSession "ClusterS1" -FaultDomainType Site -Name "Site2"
```

`Get-ClusterFaultDomain` コマンドレットを使用して、両方のサイトがクラスター用に作成されていることを確認します。

```powershell
New-ClusterFaultDomain -CimSession "ClusterS1"
```

### <a name="step-52-assign-server-nodes"></a>手順 5.2:サーバー ノードを割り当てる

次に、4 つのサーバー ノードをそれぞれのサイトに割り当てます。 次の例では、Server1 と Server2 を Site1 に割り当て、Server3 と Server4 を Site2 に割り当てます。

```powershell
Set-ClusterFaultDomain -CimSession "ClusterS1" -Name "Server1", "Server2" -Parent "Site1"
```

```powershell
Set-ClusterFaultDomain -CimSession "ClusterS1" -Name "Server3", "Server4" -Parent "Site2"
```

`Get-ClusterFaultDomain` コマンドレットを使用して、ノードが正しいサイトにあることを確認します。

```powershell
Get-ClusterFaultDomain -CimSession "ClusterS1"
```

### <a name="step-53-set-a-preferred-site"></a>手順 5.3: 優先サイトを設定する

また、グローバルな "*優先*" サイトを定義することもできます。これは、指定したリソースとグループを優先サイトで実行する必要があることを意味します。  この設定は、次のコマンドを使用してサイト レベルで定義できます。  

```powershell
(Get-Cluster).PreferredSite = "Site1"
```

ストレッチ クラスターに対して優先サイトを指定すると、次のような利点があります。

- **コールド スタート** - コールド スタートの間に、仮想マシンが優先サイトに配置されます

- **クォーラム投票**
  - 動的クォーラムを使用すると、最初にパッシブ (レプリケートされた) サイトから重み付けが減らされて、他のすべての事柄が等しい場合に、優先サイトが維持されるようになります。 さらに、非対称ネットワーク接続障害などのイベントの後の再グループ化の間に、最初にパッシブ サイトからサーバー ノードが削除されます。

  - 2 つのサイトのクォーラム分割の間に、クラスター監視に接続できない場合は、優先サイトが自動的に選択されます。 そして、パッシブ サイトのサーバー ノードがクラスター メンバーシップから削除されます。 これにより、クラスターは同時に 50% の投票を失って存続することができます。

優先サイトは、クラスターの役割レベルまたはグループ レベルで構成することもできます。 この場合、仮想マシン グループごとに異なる優先サイトを構成することができます。 これにより、あるサイトをアクティブにして、特定の仮想マシンに対して優先させることができます。

## <a name="step-6-enable-storage-spaces-direct"></a>手順 6:記憶域スペース ダイレクトを有効にする

クラスターを作成した後、`Enable-ClusterStorageSpacesDirect` コマンドレットを使用すると、記憶域スペース ダイレクトが有効になり、次のことが自動的に行われます。

- **記憶域プールを作成する:** "Cluster1 Storage Pool" のような名前で、クラスターの記憶域プールが作成されます。

- **クラスター パフォーマンス履歴ディスクを作成する:** 記憶域プールにクラスター パフォーマンス履歴仮想ディスクが作成されます。

- **データ ボリュームとログ ボリュームを作成する:** データ ボリュームとログ ボリュームが記憶域プールに作成されます。

- **記憶域スペース ダイレクト キャッシュを構成する:** 記憶域スペース ダイレクトに使用できるメディア (ドライブ) の種類が複数ある場合、最も速いものがキャッシュ デバイスとして有効にされます (ほとんどの場合、読み取りと書き込み)。

- **階層を作成する:** 既定の階層として 2 つの階層が作成されます。 1 つは "Capacity" という名前で、もう 1 つは "Performance" という名前です。 コマンドレットにより、デバイスが分析されて、各階層がデバイスの種類と回復性を組み合わせて構成されます。

ストレッチ クラスターの場合、`Enable-ClusterStorageSpacesDirect` コマンドレットでは次のことも行われます。

- サイトがセットアップされているかどうかを確認する
- どのノードがどのサイトにあるかを判断する
- 各ノードで使用できるストレージを判断する
- 記憶域レプリカ機能が各ノードにインストールされているかどうかを確認する
- 各サイトに記憶域プールを作成し、そのサイトの名前で識別する
- 各記憶域プールにデータ ボリュームとログ ボリュームを作成する (サイトごとに 1 つ)

次のコマンドを実行すると、記憶域スペース ダイレクトが有効になります。 次に示すように、記憶域プールのフレンドリ名を指定することもできます。

```powershell
Enable-ClusterStorageSpacesDirect -PoolFriendlyName "$ClusterName Storage Pool" -CimSession $ClusterName
```

記憶域プールを表示するには、以下を使用します。

```powershell
Get-StoragePool -CimSession $session
```

これで、クラスターが作成されました。

## <a name="after-you-create-the-cluster"></a>クラスターを作成した後

これで終わりですが、まだ、いくつかの重要なタスクを行う必要があります。

- クラスター監視をセットアップします。 「[クラスター監視のセットアップ](../manage/witness.md)」を参照してください。
- ボリュームを作成します。 [ボリュームの作成](../manage/create-volumes.md)に関する記事を参照してください。
- ストレッチ クラスターの場合は、ボリュームを作成し、記憶域レプリカを使用してレプリケーションをセットアップします。 [ストレッチ クラスター用のボリュームの作成とレプリケーションの設定](../manage/create-stretched-volumes.md)に関する記事を参照してください。

## <a name="next-steps"></a>次のステップ

- クラスターを Azure に登録します。 「[Azure の登録を管理する](../manage/manage-azure-registration.md)」を参照してください。
- クラスターの最終検証を実行します。 「[Azure Stack HCI クラスターの検証](validate.md)」を参照してください
