---
title: Azure Stack Hub 上の依存関係を持つ VM を削除する
description: Azure Stack Hub 上の依存関係を持つ VM (仮想マシン) を削除する方法
author: mattbriggs
ms.topic: how-to
ms.date: 12/16/2020
ms.author: mabrigg
ms.reviewer: kivenkat
ms.lastreviewed: 12/16/2020
ms.openlocfilehash: 63a4ed0c6f1d63b21e221713befa84b0d10c1a95
ms.sourcegitcommit: f30e5178e0b4be4e3886f4e9f699a2b51286e2a8
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/17/2020
ms.locfileid: "97620706"
---
# <a name="how-to-delete-a-vm-virtual-machine-with-dependencies-on-azure-stack-hub"></a>Azure Stack Hub 上の依存関係を持つ VM (仮想マシン) を削除する方法

この記事では、Azure Stack Hub 内の VM とその依存関係を削除する手順について説明します。

Azure Stack Hub から VM を削除すると、コンポーネントの依存関係 (データ ディスク、仮想ネットワーク インターフェイス、診断コンテナー) がリソース グループに残ります。 これらの項目は、OS ディスクと一緒に自動的に削除されません。

## <a name="delete-a-vm-overview"></a>VM の削除の概要

新しい VM を作成するときは、通常、新しいリソース グループを作成し、そのリソース グループ内にすべての依存関係を配置します。 VM とそのすべての依存関係を削除する場合は、リソース グループを削除できます。 Azure Resource Manager によって依存関係が処理され、それらは正常に削除されます。 リソース グループを削除しても VM を削除できない場合があります。 VM に、保持したい VM の依存関係ではないリソースが含まれる場合などです。

## <a name="delete-a-vm-with-dependencies"></a>依存関係がある VM を削除する

### <a name="with-the-portal"></a>[ポータルの場合](#tab/portal)

リソース グループを削除できない場合は、依存関係が同じリソース グループにないか、他のリソースが存在します。この場合、次の手順を実行します。

1. Azure Stack ユーザー ポータルを開きます。

2. **[Virtual Machines]** を選択します。 仮想マシンを検索し、マシンを選択して、仮想マシンのブレードを開きます。  
![依存関係がある VM を削除する](./media/delete-vm/azure-stack-hub-delete-vm-portal.png)  

3. VM と VM の依存関係を含むリソース グループをメモしておきます。

4. **[ネットワーク]** を選択し、ネットワーク インターフェイスをメモします。

5. **[ディスク]** を選択し、OS ディスクとデータ ディスクをメモしておきます。

6. **仮想マシン** のブレードに戻り、 **[削除]** を選択します。

7. 「`yes`」と入力し、削除を確定して、 **[削除]** を選択します。

7. **[リソース グループ]** を選択し、リソース グループを選択します。

8. 依存関係を手動で選択し、 **[削除]** を選択して削除します。
    1. 「`yes`」と入力し、削除を確定して、 **[削除]** を選択します。
    2. リソースが完全に削除されるまで待ちます。
    3. その後、次の依存関係を削除できます。

### <a name="az-modules"></a>[Az モジュール](#tab/ps-az)

リソース グループを削除できない場合は、依存関係が同じリソース グループにないか、他のリソースが存在します。この場合、次の手順を実行します。

Azure Stack Hub 環境に接続し、次の変数をお使いの VM 名とリソース グループに変更します。 PowerShell セッションに接続して Azure Stack Hub に接続する手順については、「[ユーザーとして PowerShell を使用して Azure Stack Hub に接続する](azure-stack-powershell-configure-user.md)」を参照してください。

```powershell
$machineName = 'VM_TO_DELETE'
$resGroupName = 'RESOURCE_GROUP'
$machine = Get-AzVM -Name $machineName -ResourceGroupName $resGroupName
```

VM 情報と依存関係の名前を取得します。 同じセッションで、次のコマンドレットを実行します。

```powershell
 $azResParams = @{
 'ResourceName' = $machineName
 'ResourceType' = 'Microsoft.Compute/virtualMachines'
     'ResourceGroupName' = $resGroupName
 }
 $vmRes = Get-AzResource @azResParams
 $vmId = $vmRes.Properties.VmId
```

ブート診断ストレージ コンテナーを削除します。 マシン名が 9 文字より短い場合は、`$diagContainer` 変数を作成するときに、インデックスを substring 内の文字列の長さに変更する必要があります。 

同じセッションで、次のコマンドレットを実行します。

```powershell
$container = [regex]::match($machine.DiagnosticsProfile.bootDiagnostics.storageUri, '^http[s]?://(.+?)\.').groups[1].value
$diagContainer = ('bootdiagnostics-{0}-{1}' -f $machine.Name.ToLower().Substring(0, 9), $vmId)
$containerRg = (Get-AzStorageAccount | where { $_.StorageAccountName -eq $container }).ResourceGroupName
$storeParams = @{
    'ResourceGroupName' = $containerRg
    'Name' = $container }
Get-AzStorageAccount @storeParams | Get-AzureStorageContainer | where { $_.Name-eq $diagContainer } | Remove-AzureStorageContainer -Force
```

仮想ネットワーク インターフェイスを削除します。

```powershell
$machine | Remove-AzNetworkInterface -Force
```

オペレーティング システム ディスクを削除します。

```powershell
$osVhdUri = $machine.StorageProfile.OSDisk.Vhd.Uri
$osDiskConName = $osVhdUri.Split('/')[-2]
$osDiskStorageAcct = Get-AzStorageAccount | where { $_.StorageAccountName -eq $osVhdUri.Split('/')[2].Split('.')[0] }
$osDiskStorageAcct | Remove-AzureStorageBlob -Container $osDiskConName -Blob $osVhdUri.Split('/')[-1]
```

VM に接続されているデータ ディスクを削除します。

```powershell
if ($machine.DataDiskNames.Count -gt 0)
 {
    Write-Verbose -Message 'Deleting disks...'
        foreach ($uri in $machine.StorageProfile.DataDisks.Vhd.Uri )
        {
            $dataDiskStorageAcct = Get-AzStorageAccount -Name $uri.Split('/')[2].Split('.')[0]
             $dataDiskStorageAcct | Remove-AzureStorageBlob -Container $uri.Split('/')[-2] -Blob $uri.Split('/')[-1] -ea Ignore
        }
 }
```

最後に、VM を削除します。 コマンドレットの実行には一定の時間がかかります。 PowerShell で VM オブジェクトを確認することによって、VM に接続されているコンポーネントを監査できます。 オブジェクトを確認するには、VM オブジェクトを含む変数を参照するだけです。 「`$machine`.

VM を削除するには、同じセッションで、次のコマンドレットを実行します。

```powershell
$machine | Remove-AzVM -Force
```
### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/ps-azureRM)

リソース グループを削除できない場合は、依存関係が同じリソース グループにないか、他のリソースが存在します。この場合、次の手順を実行します。

Azure Stack Hub 環境に接続し、次の変数をお使いの VM 名とリソース グループに変更します。 PowerShell セッションに接続して Azure Stack Hub に接続する手順については、「[ユーザーとして PowerShell を使用して Azure Stack Hub に接続する](azure-stack-powershell-configure-user.md)」を参照してください。

```powershell
$machineName = 'VM_TO_DELETE'
$resGroupName = 'RESOURCE_GROUP'
$machine = Get-AzureRmVM -Name $machineName -ResourceGroupName $resGroupName
```

VM 情報と依存関係の名前を取得します。 同じセッションで、次のコマンドレットを実行します。

```powershell
 $azResParams = @{
 'ResourceName' = $machineName
 'ResourceType' = 'Microsoft.Compute/virtualMachines'
     'ResourceGroupName' = $resGroupName
 }
 $vmRes = Get-AzureRmResource @azResParams
 $vmId = $vmRes.Properties.VmId
```

ブート診断ストレージ コンテナーを削除します。 マシン名が 9 文字より短い場合は、`$diagContainer` 変数を作成するときに、インデックスを substring 内の文字列の長さに変更する必要があります。 

同じセッションで、次のコマンドレットを実行します。

```powershell
$container = [regex]::match($machine.DiagnosticsProfile.bootDiagnostics.storageUri, '^http[s]?://(.+?)\.').groups[1].value
$diagContainer = ('bootdiagnostics-{0}-{1}' -f $machine.Name.ToLower().Substring(0, 9), $vmId)
$containerRg = (Get-AzureRmStorageAccount | where { $_.StorageAccountName -eq $container }).ResourceGroupName
$storeParams = @{
    'ResourceGroupName' = $containerRg
    'Name' = $container }
Get-AzureRmStorageAccount @storeParams | Get-AzureStorageContainer | where { $_.Name-eq $diagContainer } | Remove-AzureStorageContainer -Force
```

仮想ネットワーク インターフェイスを削除します。

```powershell
$machine | Remove-AzureRmNetworkInterface -Force
```

オペレーティング システム ディスクを削除します。

```powershell
$osVhdUri = $machine.StorageProfile.OSDisk.Vhd.Uri
$osDiskConName = $osVhdUri.Split('/')[-2]
$osDiskStorageAcct = Get-AzureRmStorageAccount | where { $_.StorageAccountName -eq $osVhdUri.Split('/')[2].Split('.')[0] }
$osDiskStorageAcct | Remove-AzureStorageBlob -Container $osDiskConName -Blob $osVhdUri.Split('/')[-1]
```

VM に接続されているデータ ディスクを削除します。

```powershell
if ($machine.DataDiskNames.Count -gt 0)
 {
    Write-Verbose -Message 'Deleting disks...'
        foreach ($uri in $machine.StorageProfile.DataDisks.Vhd.Uri )
        {
            $dataDiskStorageAcct = Get-AzureRmStorageAccount -Name $uri.Split('/')[2].Split('.')[0]
             $dataDiskStorageAcct | Remove-AzureStorageBlob -Container $uri.Split('/')[-2] -Blob $uri.Split('/')[-1] -ea Ignore
        }
 }
```

最後に、VM を削除します。 コマンドレットの実行には一定の時間がかかります。 PowerShell で VM オブジェクトを確認することによって、VM に接続されているコンポーネントを監査できます。 オブジェクトを確認するには、VM オブジェクトを含む変数を参照するだけです。 「`$machine`.

VM を削除するには、同じセッションで、次のコマンドレットを実行します。

```powershell
$machine | Remove-AzureRmVM -Force
```
---
## <a name="next-steps"></a>次のステップ

[Azure Stack Hub VM の機能](azure-stack-vm-considerations.md)