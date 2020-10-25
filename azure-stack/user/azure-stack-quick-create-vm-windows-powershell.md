---
title: Azure Stack Hub で PowerShell を使用して Windows Server VM を作成する
description: Azure Stack Hub 内で PowerShell を使用して Windows Server VM を作成します。
author: mattbriggs
ms.topic: quickstart
ms.date: 08/24/2020
ms.author: mabrigg
ms.reviewer: kivenkat
ms.lastreviewed: 11/11/2019
ms.custom: conteperfq4
ms.openlocfilehash: 189f0b9472ed8f29b4cd3ee287d6c6630c850503
ms.sourcegitcommit: 3e2460d773332622daff09a09398b95ae9fb4188
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/15/2020
ms.locfileid: "90573872"
---
# <a name="quickstart-create-a-windows-server-vm-by-using-powershell-in-azure-stack-hub"></a>クイック スタート:Azure Stack Hub 内で PowerShell を使用して Windows Server VM を作成する

Azure Stack Hub PowerShell を使用して、Windows Server 2016 仮想マシン (VM) を作成できます。 この記事の手順に従って、VM を作成し、使用します。 この記事では、以下のことを実行する手順も示します。

* リモート クライアントを使用して VM に接続する。
* IIS Web サーバーをインストールし、既定のホーム ページを表示する。
* リソースをクリーンアップする。

> [!NOTE]
>  この記事で説明している手順は、Azure Stack Development Kit で、または VPN 経由で接続している場合は Windows ベースの外部クライアントで実行できます。

## <a name="prerequisites-for-windows-server-vm"></a>Windows Server VM の前提条件

* Azure Stack Hub オペレーターが Azure Stack Hub Marketplace に **Windows Server 2016** のイメージを追加済みであることを確認します。

* リソースを作成して管理するため、Azure Stack Hub には Azure PowerShell の特定のバージョンが必要です。 Azure Stack Hub 用に構成された PowerShell がない場合は、PowerShell を[インストール](../operator/azure-stack-powershell-install.md)する手順に従います。

* Azure Stack Hub PowerShell の設定を使用して、ご自身の Azure Stack Hub 環境に接続する必要があります。 手順については、「[PowerShell を使用してユーザーとして Azure Stack Hub に接続する](azure-stack-powershell-configure-user.md)」をご覧ください。

## <a name="create-a-resource-group"></a>リソース グループを作成する

リソース グループとは、Azure Stack Hub リソースのデプロイ先となって管理される論理コンテナーです。 開発キットまたは Azure Stack Hub 統合システムから、次のコード ブロックを実行してリソース グループを作成します。 

> [!NOTE]
> 値は、コード例のすべての変数に割り当てられます。 ただし、必要に応じて新しい値を割り当てることができます。

```powershell
# Create variables to store the location and resource group names.
$location = "local"
$ResourceGroupName = "myResourceGroup"

New-AzureRmResourceGroup `
  -Name $ResourceGroupName `
  -Location $location
```

## <a name="create-storage-resources"></a>ストレージ リソースの作成

ブート診断の出力を格納するためのストレージ アカウントを作成します。

```powershell
# Create variables to store the storage account name and the storage account SKU information
$StorageAccountName = "mystorageaccount"
$SkuName = "Standard_LRS"

# Create a new storage account
$StorageAccount = New-AzureRMStorageAccount `
  -Location $location `
  -ResourceGroupName $ResourceGroupName `
  -Type $SkuName `
  -Name $StorageAccountName

Set-AzureRmCurrentStorageAccount `
  -StorageAccountName $storageAccountName `
  -ResourceGroupName $resourceGroupName

```

## <a name="create-networking-resources"></a>ネットワーク リソースの作成

仮想ネットワーク、サブネット、パブリック IP アドレスを作成します。 これらのリソースは、VM にネットワーク接続を提供するために使用されます。

```powershell
# Create a subnet configuration
$subnetConfig = New-AzureRmVirtualNetworkSubnetConfig `
  -Name mySubnet `
  -AddressPrefix 192.168.1.0/24

# Create a virtual network
$vnet = New-AzureRmVirtualNetwork `
  -ResourceGroupName $ResourceGroupName `
  -Location $location `
  -Name MyVnet `
  -AddressPrefix 192.168.0.0/16 `
  -Subnet $subnetConfig

# Create a public IP address and specify a DNS name
$pip = New-AzureRmPublicIpAddress `
  -ResourceGroupName $ResourceGroupName `
  -Location $location `
  -AllocationMethod Static `
  -IdleTimeoutInMinutes 4 `
  -Name "mypublicdns$(Get-Random)"
```

### <a name="create-a-network-security-group-and-a-network-security-group-rule"></a>ネットワーク セキュリティ グループとネットワーク セキュリティ グループの規則を作成する

ネットワーク セキュリティ グループは、受信規則と送信規則を使用して VM をセキュリティで保護します。 ポート 3389 に、受信リモート デスクトップ接続を許可する受信規則を作成し、ポート 80 に、受信 Web トラフィックを許可する受信規則を作成しましょう。

```powershell
# Create an inbound network security group rule for port 3389
$nsgRuleRDP = New-AzureRmNetworkSecurityRuleConfig `
  -Name myNetworkSecurityGroupRuleRDP `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 1000 `
  -SourceAddressPrefix * `
  -SourcePortRange * `
  -DestinationAddressPrefix * `
  -DestinationPortRange 3389 `
  -Access Allow

# Create an inbound network security group rule for port 80
$nsgRuleWeb = New-AzureRmNetworkSecurityRuleConfig `
  -Name myNetworkSecurityGroupRuleWWW `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 1001 `
  -SourceAddressPrefix * `
  -SourcePortRange * `
  -DestinationAddressPrefix * `
  -DestinationPortRange 80 `
  -Access Allow

# Create a network security group
$nsg = New-AzureRmNetworkSecurityGroup `
  -ResourceGroupName $ResourceGroupName `
  -Location $location `
  -Name myNetworkSecurityGroup `
  -SecurityRules $nsgRuleRDP,$nsgRuleWeb
```

### <a name="create-a-network-card-for-the-vm"></a>VM 用のネットワーク カードを作成する

ネットワーク カードは、VM をサブネット、ネットワーク セキュリティ グループ、パブリック IP アドレスに接続します。

```powershell
# Create a virtual network card and associate it with public IP address and NSG
$nic = New-AzureRmNetworkInterface `
  -Name myNic `
  -ResourceGroupName $ResourceGroupName `
  -Location $location `
  -SubnetId $vnet.Subnets[0].Id `
  -PublicIpAddressId $pip.Id `
  -NetworkSecurityGroupId $nsg.Id
```

## <a name="create-a-vm"></a>VM の作成

VM 構成を作成します。 この構成には、VM をデプロイするときに使用される設定が含まれています。 たとえば、ユーザー資格情報、サイズ、VM イメージなどです。

```powershell
# Define a credential object to store the username and password for the VM
$UserName='demouser'
$Password='Password@123'| ConvertTo-SecureString -Force -AsPlainText
$Credential=New-Object PSCredential($UserName,$Password)

# Create the VM configuration object
$VmName = "VirtualMachinelatest"
$VmSize = "Standard_A1"
$VirtualMachine = New-AzureRmVMConfig `
  -VMName $VmName `
  -VMSize $VmSize

$VirtualMachine = Set-AzureRmVMOperatingSystem `
  -VM $VirtualMachine `
  -Windows `
  -ComputerName "MainComputer" `
  -Credential $Credential -ProvisionVMAgent

$VirtualMachine = Set-AzureRmVMSourceImage `
  -VM $VirtualMachine `
  -PublisherName "MicrosoftWindowsServer" `
  -Offer "WindowsServer" `
  -Skus "2016-Datacenter" `
  -Version "latest"

# Sets the operating system disk properties on a VM.
$VirtualMachine = Set-AzureRmVMOSDisk `
  -VM $VirtualMachine `
  -CreateOption FromImage | `
  Set-AzureRmVMBootDiagnostics -ResourceGroupName $ResourceGroupName `
  -StorageAccountName $StorageAccountName -Enable |`
  Add-AzureRmVMNetworkInterface -Id $nic.Id


# Create the VM.
New-AzureRmVM `
  -ResourceGroupName $ResourceGroupName `
  -Location $location `
  -VM $VirtualMachine
```

## <a name="connect-to-the-vm"></a>VM に接続します

前の手順で作成した VM にリモート接続するには、仮想マシンのパブリック IP アドレスが必要です。 次のコマンドを実行して、VM のパブリック IP アドレスを取得します。

```powershell
Get-AzureRmPublicIpAddress `
  -ResourceGroupName $ResourceGroupName | Select IpAddress
```

次のコマンドを使用して、VM とのリモート デスクトップ セッションを作成します。 IP アドレスを VM の *publicIPAddress* に置き換えます。 求められたら、VM の作成時に使用されたユーザー名とパスワードを入力します。

```powershell
mstsc /v <publicIpAddress>
```

## <a name="install-iis-via-powershell"></a>PowerShell を使用した IIS のインストール

Azure VM にサインインしたら、1 行の PowerShell を使用して IIS をインストールし、Web トラフィックを許可するローカル ファイアウォール規則を有効にできます。 PowerShell プロンプトを開き、次のコマンドを実行します。

```powershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
```

## <a name="view-the-iis-welcome-page"></a>IIS のようこそページの表示

IIS をインストールし、VM 上のポート 80 を開いたら、任意のブラウザーを使用して IIS の既定のようこそページを表示することができます。 前のセクションで指定した *publicIpAddress* を使用して、既定のページにアクセスします。

![IIS の既定のサイト](./media/azure-stack-quick-create-vm-windows-powershell/default-iis-website.png)

## <a name="delete-the-vm"></a>VM の削除

不要になったときは次のコマンドを使用して、VM とその関連リソースを含むリソース グループを削除します。

```powershell
Remove-AzureRmResourceGroup `
  -Name $ResourceGroupName
```

## <a name="next-steps"></a>次のステップ

このクイック スタートでは、単純な Windows VM をデプロイしました。 Azure Stack Hub VM の詳細については、「[Azure Stack Hub VM の機能](azure-stack-vm-considerations.md)」をご覧ください。
