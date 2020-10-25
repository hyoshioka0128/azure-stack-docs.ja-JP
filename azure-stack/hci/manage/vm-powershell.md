---
title: Windows PowerShell を使用して VM を管理する - Azure Stack HCI
description: Windows PowerShell を使用して Azure Stack HCI 上の仮想マシンを管理する方法
author: v-dasis
ms.topic: how-to
ms.date: 07/21/2020
ms.author: v-dasis
ms.reviewer: JasonGerend
ms.openlocfilehash: 8d5f654ed5af1fe66c9a28bbfd6bebb09672fa88
ms.sourcegitcommit: 3e2460d773332622daff09a09398b95ae9fb4188
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/15/2020
ms.locfileid: "90573515"
---
# <a name="manage-vms-on-azure-stack-hci-using-windows-powershell"></a>Windows PowerShell を使用して Azure Stack HCI 上の VM を管理する

> 適用対象: Azure Stack HCI バージョン 20H2、Windows Server 2019

Windows PowerShell を使用して、Azure Stack HCI 上に仮想マシン (VM) を作成して管理できます。

通常、VM は、クラスター内のホスト サーバーではなく、リモート コンピューターから管理します。 このリモート コンピューターは、管理コンピューターと呼ばれます。

> [!NOTE]
> 管理コンピューターから PowerShell コマンドを実行する場合は、管理しているホスト サーバーの名前を指定した -ComputerName パラメーターを含めます。 NetBIOS 名、IP アドレス、および完全修飾ドメイン名を指定できます。

PowerShell を使用した VM の管理に関する完全なリファレンス ドキュメントについては、[Hyper-V リファレンス](/powershell/module/hyper-v/?view=win10-ps)を参照してください。

## <a name="create-a-vm"></a>VM の作成  

新しい VM を作成するには、`New-VM` コマンドレットを使用します。 詳細については、「[New-VM](/powershell/module/hyper-v/new-vm?view=win10-ps)」のリファレンス ドキュメントを参照してください。

既存の仮想ハード ディスクを使用して新しい VM を作成するときに指定できる設定は次のとおりです。

- **-Name** は、作成する仮想マシンに対して指定する名前です。

- **-MemoryStartupBytes** は、起動時に仮想マシンが使用できるメモリの量です。

- **-BootDevice** は、仮想マシンが起動時に起動するデバイスです。
 通常、これは仮想ハードディスク (VHD)、DVD ベースのブート用の .iso ファイル、またはネットワーク ブート用のネットワーク アダプター (NetworkAdapter) です。

- **-VHDPath** は、使用する仮想マシン ディスクへのパスです。

- **-Path** は、仮想マシンの構成ファイルを格納しているパスです。

- **-Generation** は、仮想マシンの世代です。 VHD の場合は第 1 世代、VHDX の場合は第 2 世代を使用します。

- **-Switch** は、他の仮想マシンやネットワークに接続するために仮想マシンで使用される仮想スイッチの名前です。 仮想スイッチの名前は、[Get-VMSwitch](/powershell/module/hyper-v/get-vmswitch?view=win10-ps) を使用して取得します。 次に例を示します。  

VM1 という VM を作成するための完全なコマンドは次のとおりです。

 ```powershell
 New-VM -ComputerName Server1 -Name VM1 -MemoryStartupBytes <Memory> -BootDevice <BootDevice> -VHDPath <VHDPath> -Path <Path> -Generation <Generation> -Switch <SwitchName>
 ```

次の例では、4 GB のメモリを備えた第 2 世代仮想マシンを作成します。 これは、現在のディレクトリの VMs\Win10.vhdx フォルダーから起動し、ExternalSwitch という名前の仮想スイッチを使用します。 仮想マシンの構成ファイルは、VMData フォルダーに格納されます。  

``` powershell
New-VM -ComputerName Server1 -Name VM1 -MemoryStartupBytes 4GB -BootDevice VHD -VHDPath .\VMs\Win10.vhdx -Path .\VMData -Generation 2 -Switch ExternalSwitch
```

仮想ハード ディスクを指定するには、次のパラメーターを使用します。

新しい仮想ハード ディスクを使用して仮想マシンを作成するには、次に示すように、上記の例の **-VHDPath** パラメーターを **-NewVHDPath** に置き換え、 **-NewVHDSizeBytes** パラメーターを追加します。  

``` powershell  
New-VM -ComputerName Server1 -Name VM1 -MemoryStartupBytes 4GB -BootDevice VHD -NewVHDPath .\VMs\Win10.vhdx -Path .\VMData -NewVHDSizeBytes 20GB -Generation 2 -Switch ExternalSwitch  
 ```  

新しい仮想ディスクを使用して、オペレーティング システム イメージで起動する仮想マシンを作成する場合は、[Windows 10 上の Hyper-V での仮想マシンの作成チュートリアル](/virtualization/hyper-v-on-windows/quick-start/create-virtual-machine)にある PowerShell の例を参照してください。  

## <a name="get-a-list-of-vms"></a>VM の一覧を取得する

次の例では、Server1 上のすべての VM の一覧を返します。

```powershell
Get-VM -ComputerName Server1
```

次の例では、`Where-Object` コマンドを使用してフィルターを追加することによって、サーバー上で実行中のすべての VM の一覧を返します。 詳細については、[Where-Object の使用](/previous-versions/windows/it-pro/windows-powershell-1.0/ee177028(v=technet.10))に関するドキュメントを参照してください。

```powershell
Get-VM -ComputerName Server1 | Where-Object -Property State -eq "Running"
```

次の例では、サーバー上のすべてのシャットダウンした VM の一覧を返します。

```powershell
Get-VM -ComputerName Server1 | Where-Object -Property State -eq "Off"
```

## <a name="start-and-stop-a-vm"></a>VM を起動および停止する

VM を起動または停止するには、`Start-VM` と `Stop-VM` のコマンドを使用します。 詳細については、「[Start-VM](/powershell/module/hyper-v/start-vm?view=win10-ps)」および「[Stop-VM](/powershell/module/hyper-v/stop-vm?view=win10-ps)」のリファレンス ドキュメントを参照してください。

次の例は、VM1 という名前の VM を起動する方法を示しています。

```powershell
Start-VM -Name VM1 -ComputerName Server1
```

次の例は、TestVM という名前の VM をシャットダウンする方法を示しています。

```powershell
Stop-VM -Name VM1 -ComputerName Server1
```

## <a name="move-a-vm"></a>VM の移動

`Move-VM` コマンドレットは、VM を別のサーバーに移動します。 詳細については、「[Move-VM](/powershell/module/hyper-v/move-vm?view=win10-ps)」のリファレンス ドキュメントを参照してください。

 次の例は、VM が Server1 の SMB 共有に格納されている場合に、VM を Server2 に移動する方法を示しています。

```powershell
Move-VM -ComputerName Server1 -Name VM1 -DestinationHost Server2
 ```

次の例は、Server1 から Server2 に VM を移動し、その VM に関連付けられているすべてのファイルをリモート コンピューターの D:\VM_name に移動する方法を示しています。

```powershell
Move-VM -ComputerName Server1 -Name VM1 -DestinationHost Server2 -IncludeStorage -DestinationStoragePath D:\VM_name
```

## <a name="import-or-export-a-vm"></a>VM をインポートまたはエクスポートする

`Import-VM` と `Export-VM` のコマンドレットで、VM のインポートとエクスポートを行います。 次にいくつかの例を示します。 詳細については、「[Import-VM](/powershell/module/hyper-v/import-vm?view=win10-ps)」と「[Export-VM](/powershell/module/hyper-v/export-vm?view=win10-ps)」のリファレンス ドキュメントを参照してください。

次の例は、VM をその構成ファイルからインポートする方法を示しています。 VM はインプレース登録されるため、そのファイルはコピーされません。

```powershell
Import-VM -ComputerName Server1 -Name VM1 -Path 'C:\<vm export path>\2B91FEB3-F1E0-4FFF-B8BE-29CED892A95A.vmcx'
```

次の例では、D ドライブのルートに VM をエクスポートします。

```powershell
Export-VM -ComputerName Server1 -Name VM1 -Path D:\
```

## <a name="rename-a-vm"></a>VM の名前を変更する

VM の名前を変更するには、`Rename-VM` コマンドレットを使用します。 詳細については、「[Rename-VM](/powershell/module/hyper-v/rename-vm?view=win10-ps)」のリファレンス ドキュメントを参照してください。

次の例では、VM1 の名前を VM2 に変更し、名前を変更した仮想マシンを表示します。

```powershell
Rename-VM -ComputerName Server1 -Name VM1 -NewName VM2
```

## <a name="create-a-vm-checkpoint"></a>VM チェックポイントを作成する

VM のチェックポイントを作成するには、`Checkpoint-VM` コマンドレットを使用します。 詳細については、「[Checkpoint-VM](/powershell/module/hyper-v/checkpoint-vm?view=win10-ps)」のリファレンス ドキュメントを参照してください。

次の例では、Test という名前の VM に BeforeInstallingUpdates という名前のチェックポイントを作成します。

```powershell
Checkpoint-VM -ComputerName Server1 -Name VM1 -SnapshotName BeforeInstallingUpdates
```

## <a name="create-a-vhd-for-a-vm"></a>VM 用の VHD を作成する

VM の新しい VHD を作成するには、`New-VHD` コマンドレットを使用します。 その使用方法の詳細については、「[New-VHD](/powershell/module/hyper-v/new-vhd?view=win10-ps)」のリファレンス ドキュメントを参照してください。

次の例では、サイズが 10 GB である VHDX 形式の容量可変仮想ハード ディスクを作成します。 ファイル名拡張子によって形式が決まります。また、種類が指定されていないので、既定の種類の容量可変が使用されます。

```powershell
New-VHD -ComputerName Server1 -Name VM1 -Path c:\Base.vhdx -SizeBytes 10GB
```

## <a name="add-a-network-adapter-to-a-vm"></a>VM にネットワーク アダプターを追加する

VM に仮想ネットワーク アダプターを追加するには、`Add-VMNetworkAdapter` コマンドレットを使用します。 次にいくつかの例を示します。 その使用方法の詳細については、「[Add-VMNetworkAdapter](/powershell/module/hyper-v/add-vmnetworkadapter?view=win10-ps)」のリファレンス ドキュメントを参照してください。

次の例では、VM1 という名前の仮想マシンに Redmond NIC1 という名前の仮想ネットワーク アダプターを追加します。

```powershell
Add-VMNetworkAdapter -ComputerName Server1 -VMName VM1 -Name "Redmond NIC1"
```

次の例では、VM1 という名前の仮想マシンに仮想ネットワーク アダプターを追加し、Network という名前の仮想スイッチにそれを接続します。

```powershell
Add-VMNetworkAdapter -ComputerName Server1 -VMName VM1 -SwitchName Network
```

## <a name="create-a-virtual-switch-for-a-vm"></a>VM の仮想スイッチを作成する

VM ホスト上の新しい仮想スイッチの作成には、`New-VMSwitch` コマンドレットを使用します。 その使用方法の詳細については、「[New-VMSwitch](/powershell/module/hyper-v/new-vmswitch?view=win10-ps)」のリファレンス ドキュメントを参照してください。

次の例では、"QoS Switch" という新しいスイッチを作成します。これは、Wired Ethernet Connection 3 というネットワーク アダプターにバインドされ、重みベースの最小帯域幅をサポートします。

```powershell
New-VMSwitch "QoS Switch" -NetAdapterName "Wired Ethernet Connection 3" -MinimumBandwidthMode Weight
```

## <a name="set-memory-for-a-vm"></a>VM のメモリを設定する

VM のメモリを構成するには、`Set-VMMemory` コマンドレットを使用します。 その使用方法の詳細については、「[Set-VMMemory](/powershell/module/hyper-v/set-vmmemory?view=win10-ps)」のリファレンス ドキュメントを参照してください。

次の例では、VM1 という名前の VM の動的メモリを有効にし、その最小、起動、最大の各メモリ、そのメモリ優先度、およびそのバッファーを設定します。

```powershell
Set-VMMemory -ComputerName Server1 -Name VM1 -DynamicMemoryEnabled $true -MinimumBytes 64MB -StartupBytes 256MB -MaximumBytes 2GB -Priority 80 -Buffer 25
```

## <a name="set-virtual-processors-for-a-vm"></a>VM の仮想プロセッサを設定する

VM の仮想プロセッサを構成するには、`Set-VMProcessor` コマンドレットを使用します。 その使用方法の詳細については、「[Set-VMProcessor](/powershell/module/hyper-v/set-vmprocessor?view=win10-ps)」のリファレンス ドキュメントを参照してください。

次の例では、VM1 という名前の VM を、2 つの仮想プロセッサ、予約 10%、制限 75%、相対的な重み 200 で構成します。

```powershell
Set-VMProcessor -ComputerName Server1 -Name VM1 -Count 2 -Reserve 10 -Maximum 75 -RelativeWeight 200
```

## <a name="next-steps"></a>次のステップ  

Windows Admin Center を使用して、VM を作成および管理することもできます。 詳細については、「[Windows Admin Center](vm.md)」を参照してください。
