---
title: 新しいハードウェア上の Azure Stack HCI に移行する
description: 新しいハードウェア上の Azure Stack HCI に移行する方法について説明します
author: v-dasis
ms.topic: how-to
ms.date: 12/10/2020
ms.author: v-dasis
ms.reviewer: JasonGerend
ms.openlocfilehash: 7a0c0ca7a99b3554b74cc80911acbee92793aa52
ms.sourcegitcommit: 9b0e1264ef006d2009bb549f21010c672c49b9de
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98254825"
---
# <a name="migrate-to-azure-stack-hci-on-new-hardware"></a>新しいハードウェア上の Azure Stack HCI に移行する

> 適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019、Windows Server 2016、Windows Server 2012 R2、Windows Server 2008 R2

このトピックでは、Windows PowerShell と Robocopy を使用して、Windows Server 2012 R2、Windows Server 2016、または Windows Server 2019 上の仮想マシン (VM) ファイルを、新しい Azure Stack HCI サーバー ハードウェアに移行する方法について説明します。 Robocopy は、サーバー間でファイルをコピーするための堅牢な方法です。 切断された場合は再開され、最後の既知の状態から処理が続けられます。 また、Robocopy では、サーバー メッセージ ブロック (SMB) を利用したマルチスレッド ファイルのコピーもサポートされています。 詳細については、「[Robocopy](/windows-server/administration/windows-commands/robocopy)」を参照してください。

> [!NOTE]
> Windows Server から Azure Stack HCI への Hyper-V ライブ マイグレーションと Hyper-V レプリカはサポートされていません。

移行したい VM が Windows 2012 R2 以前の上にある場合は、「[古い VM の移行](#migrating-older-vms)」を参照してください。

同じハードウェアを使用して Azure Stack HCI に移行する方法については、「[同じハードウェア上の Azure Stack HCI に移行する](migrate-cluster-same-hardware.md)」を参照してください。

次の図は、例として Windows Server の移行元クラスターと Azure Stack HCI の移行先クラスターを示したものです。 スタンドアロン サーバー上の VM を移行することもできます。

:::image type="content" source="media/migrate/migrate-cluster.png" alt-text="クラスターを Azure Stack HCI に移行する" lightbox="media/migrate/migrate-cluster.png":::

予想されるダウンタイムに関しては、デュアル 40 GB RDMA East-West ネットワークを備えた単一の NIC をクラスター間に使用し、32 マルチスレッド用に構成された Robocopy を使用した場合、1 時間あたり 1.9 TB の転送速度を実現できます。

> [!NOTE]
> ストレッチ クラスター用 VM の移行については、この記事では説明しません。

## <a name="before-you-begin"></a>始める前に

移行を始める前に、要件と考慮事項がいくつかあります。

- すべての Windows PowerShell コマンドは、管理者として実行する必要があります。

- 移行元と移行先の両方のクラスターでの管理者アクセス許可があるドメイン資格情報と、両方のクラスターが含まれる移行元と移行先の組織単位 (OU) での完全な権限を、持っている必要があります。

- VM を移行するためのクラスター間での Kerberos 認証を容易にするため、両方のクラスターが Active Directory の同じフォレストとドメインに存在している必要があります。

- どちらのクラスターも、グループ ポリシー オブジェクト (GPO) の継承のブロックが設定された Active Directory OU 内に存在する必要があります。 これにより、ドメイン レベルの GPO とセキュリティ ポリシーによって、移行が影響を受けることがなくなります。

- クラスター間で整合性のある Kerberos 認証をサポートするため、両方のクラスターが同じタイム ソースに接続されている必要があります。

- 移行元クラスターの VM によって使用されている Hyper-V 仮想スイッチの名前を記録しておきます。 VM をインポートする前に、Azure Stack HCI 移行先クラスターの "仮想マシン ネットワーク" に同じ仮想スイッチ名を使用する必要があります。

- 移行元 VM の ISO イメージ ファイルをすべて削除します。 これは、Hyper-V マネージャーの **[VM Properties]\(VM のプロパティ\)** の **[Hardware section]\(ハードウェア セクション\)** を使用して行います。 すべての仮想 CD/DVD ドライブについて、 **[削除]** を選択します。

- 移行元クラスターのすべての VM をシャットダウンします。 これは、移行プロセス全体を通してバージョン管理と状態が維持されるようにするために必要です。

- 必要に応じて VM のインポートと更新を行うため、Azure Stack HCI で VM のバージョンがサポートされているかどうかを確認します。 それを行う方法については、「[VM のバージョンのサポートと更新](#vm-version-support-and-update)」セクションを参照してください。

- 移行元クラスター上のすべての VM をバックアップします。 すべてのアプリケーションとデータのクラッシュ整合バックアップ、およびすべてのデータベースのアプリケーション整合バックアップを完了します。 Azure へのバックアップについては、[Azure Backup の使用](../manage/use-azure-backup.md)に関するページを参照してください。

- 以前の状態にロールバックする必要がある場合に備えて、移行元クラスターの VM とドメイン コントローラーのチェックポイントを作成します。 これは、物理サーバーには適用されません。

- エンドツーエンドの転送パケット サイズが最も効率的になるように、移行元と移行先のクラスター ストレージ ネットワークで、最大ジャンボ フレーム サイズが同じであることを確認します (特に、RDMA ネットワーク アダプターと、それぞれのスイッチ ネットワーク ポート)。

- 移行元クラスターの Hyper-V 仮想スイッチの名前を記録しておきます。 移行先クラスターでそれを再利用します。

- Azure Stack HCI のハードウェアは、移行元のハードウェアと少なくとも同じ容量および同じ構成である必要があります。

- ファイル転送の速度を最高にするため、移行元と移行先のクラスター間のネットワーク ホップ数または物理的な距離を最小限にします。

## <a name="vm-version-support-and-update"></a>VM のバージョンのサポートと更新

次の表は、Windows Server OS のバージョンとその VM のバージョンの一覧です。

VM を実行できる OS のバージョンに関係なく、Azure Stack HCI への直接移行でサポートされる VM の最小バージョンはバージョン 5.0 です。 これは、Windows Server 2012 R2 での VM の既定のバージョンを表します。 したがって、たとえばバージョン 2.0、3.0、または 4.0 で実行されている VM は、移行前にバージョン 5.0 に更新する必要があります。

|OS バージョン|VM のバージョン|
|---|---|
|Windows Server 2008 SP1|2.0|
|Windows Server 2008 R2|3.0|
|Windows Server 2012|4.0|
|Windows Server 2012 R2|5.0|
|Windows Server 2016|8.0|
|Windows Server 2019|9.0|
|Azure Stack HCI|9.0|

Windows Server 2012 R2、Windows Server 2016、および Windows Server 2019 上の VM の場合、Robocopy の移行スクリプトを実行する前に、まず、移行元のハードウェアでサポートされている最新の VM バージョンに、すべての VM を更新します。 これにより、すべての VM が少なくともバージョン 5.0 になり、VM のインポートが成功するようになります。

Windows Server 2008 SP1、Windows Server 2008 R2-SP1、および Windows 2012 上の VM の場合、VM のバージョンはバージョン 5.0 より小さくなります。 また、これらの VM では、構成に .vcmx ファイルではなく .xml ファイルが使用されています。 そのため、Azure Stack HCI への VM の直接インポートはサポートされていません。 このような場合は、「[古い VM の移行](#migrating-older-vms)」で詳しく説明されている 2 つのオプションがあります。

### <a name="updating-the-vm-version"></a>VM バージョンの更新

以下のコマンドは、Windows Server 2012 R2 以降に適用されます。 1 台のサーバーについてすべての VM のバージョンを表示するには、次のコマンドを使用します。

```powershell
Get-VM * | Format-Table Name,Version
```

クラスター上のすべてのサーバーについて、すべての VM のバージョンを表示するには:

```powershell
Get-VM –ComputerName (Get-ClusterNode)
```

すべてのサーバーで、すべての VM をサポートされている最新バージョンに更新するには:

```powershell
Get-VM –ComputerName (Get-ClusterNode) | Update-VMVersion -Force
```

## <a name="rdma-recommendations"></a>RDMA に関する推奨事項

リモート ダイレクト メモリ アクセス (RDMA) を使用している場合、Robocopy によるクラスター間での VM のコピーにそれを利用できます。 以下に示すのは、RDMA の使用に関する推奨事項です。

- 移行元と移行先のクラスター間で最速のネットワーク パスを使用するため、両方のクラスターを同じ Top-of-Rack (ToR) スイッチに接続します。 ストレージ ネットワーク パスの場合、通常、これにより 10 GbE/25 GbE 以上の速度がサポートされ、RDMA が利用されます。

- RDMA アダプターまたは標準が移行元クラスターと移行先クラスターで異なる場合は (ROCE と iWARP)、Robocopy により、使用可能な最速のネットワークを通して TCP/IP 経由の SMB が代わりに利用されます。 これは、通常、East-West ネットワークでデュアル 10 Gbe/25 Gbe 以上の速度であり、クラスター間で VM の VHDX ファイルをコピーする最適な方法が提供されます。

- Robocopy でクラスター間 (East-West ネットワーク) に RDMA を確実に利用できるようにするには、移行元クラスターと移行先クラスターの間でルーティングできるように、RDMA ストレージ ネットワークを構成します。

## <a name="create-the-new-cluster"></a>新しいクラスターを作成する

Azure Stack HCI クラスターを作成する前に、クラスターに含まれるようになる各新規サーバーに、Azure Stack HCI OS をインストールする必要があります。 これを行う方法については、「[Azure Stack HCI オペレーティング システムのデプロイ](operating-system.md)」を参照してください。

新しいクラスターを作成するには、Windows Admin Center または Windows PowerShell を使用します。 これを行う方法の詳細については、「[Windows Admin Center を使用して Azure Stack HCI クラスターを作成する](create-cluster.md)」および「[Windows PowerShell を使用して Azure Stack HCI クラスターを作成する](create-cluster-powershell.md)」を参照してください。

> [!IMPORTANT]
> Hyper-V 仮想スイッチ (`VMSwitch`) の名前は、クラスターの間で同じである必要があります。 すべてのサーバーについて、移行先クラスターで作成された仮想スイッチの名前が、移行元クラスターで使用されているものと一致していることを確認します。 VM をインポートする前に、スイッチ名が同じであることを確認します。

> [!NOTE]
> 新しい VM を作成する前に、Azure Stack HCI クラスターを Azure に登録する必要があります。 詳細については、[Azure への登録](register-with-azure.md)に関するページを参照してください。

## <a name="run-the-migration-script"></a>移行スクリプトを実行する

PowerShell スクリプト `Robocopy_Remote_Server_.ps1` を実行すると、Robocopy を使用して、VM ファイルとその依存ディレクトリおよびメタデータが、移行元から移行先クラスターにコピーされます。 このスクリプトは、次の TechNet にある元のスクリプトから変更されています: 「[PowerShell と RoboCopy を使用してリモート サーバーにファイルをロボコピーする](https://gallery.technet.microsoft.com/scriptcenter/Robocoy-Files-to-Remote-bdfc5154)」。

このスクリプトを使用すると、特定のクラスター共有ボリューム (CSV) について、すべての VM の VHD、VHDX、VMCX ファイルが移行先クラスターにコピーされます。 一度に 1 つの CSV が移行されます。

RDMA と高速ネットワーク転送の利点を利用するため、移行スクリプトは各移行元サーバーでローカルに実行されます。 手順は次のとおりです。

1. 移行先クラスターの各ノードが、移行先 CSV の CSV 所有者に設定されていることを確認します。

1. コピー対象のすべての VM の VHD および VHDX ファイルの場所を確認するには、次のコマンドレットを使用します。 `C:\vmpaths.txt` ファイルを調べて、ステップ 4 で Robocopy を開始する最上位のソース ファイル パスを確認します。

    ```powershell  
    Get-ChildItem -Path "C:\Clusterstorage\Volume01\*.vhd*" -Recurse > c:\vmpaths.txt
    ```

    > [!NOTE]
    > VHD ファイルと VHDX ファイルが同じボリューム上の異なるパスに配置されている場合は、異なるパスごとに移行スクリプトを実行して、すべてをコピーする必要があります。

1. 移行元クラスターの VM のパスと移行先クラスターの VM のパスが一致するように、次の 3 つの変数を変更します。

    - `$Dest_Server = "Node01"`
    - `$source  = "C:\Clusterstorage\Volume01"`
    - `$dest = "\\$Dest_Server\C$\Clusterstorage\Volume01"`

1. 移行元の各 Windows Server サーバーで次のスクリプトを実行します。

```powershell
<#
#===========================================================================  
# Script: Robocopy_Remote_Server_.ps1
#===========================================================================  
.DESCRIPTION:
Change the following variables to match your source cluster VM path with the destination cluster VM path. Then run this script on each source Cluster Node CSV owner and make sure the destination cluster node is set to the CSV owner for the destination CSV.

        Change $Dest_Server = "Node01"
        Change $source  = "C:\Clusterstorage\Volume01"
        Change $dest = "\\$Dest_Server\C$\Clusterstorage\Volume01"
#>

$Space       = Write-host ""
$Dest_Server = "Node01"
$source      = "C:\Clusterstorage\Volume01"
$dest        = "\\$Dest_Server\C$\Clusterstorage\Volume01"
$Logfile     = "c:\temp\Robocopy1-$date.txt"
$date        = Get-Date -UFormat "%Y%m%d"
$cmdArgs     = @("$source","$dest",$what,$options)  
$what        = @("/COPYALL")
$options     = @("/E","/MT:32","/R:0","/W:1","/NFL","/NDL","/LOG:$logfile","/xf")
 
## Get Start Time
$startDTM = (Get-Date)
 
$Dest_Server     = "Node01"
$TARGETDIR   = \\$Dest_Server\C$\Clusterstorage\Volume01
$Space
Clear
## Provide Information
Write-host ".....Copying Virtual Machines FROM $Source to $TARGETDIR ....................." -fore Green -back black
Write-Host "........................................." -Fore Green

## Kick off the copy with options defined  
robocopy @cmdArgs
 
## Get End Time
$endDTM = (Get-Date)
 
## Echo Time elapsed
$Time = "Elapsed Time: = $(($endDTM-$startDTM).totalminutes) minutes"  
## Provide time it took
Write-host ""
Write-host " Copy Virtual Machines to $Dest_Server has been completed......" -fore Green -back black
Write-host " Copy Virtual Machines to $Dest_Server took $Time        ......" -fore Cyan
```

## <a name="import-the-vms"></a>VM をインポートする

VM のワークロードの回復性、パフォーマンス、スケーリングを向上させるためのベスト プラクティスは、クラスター ノードごとに少なくとも 1 つのクラスター共有ボリューム (CSV) を作成し、各 CSV 所有者の VM のバランスを均等にすることです。 既定では、このバランス処理は 5 分ごとに自動的に行われます。移行元クラスター ノードと移行先クラスター ノードの間で Robocopy を使用するときは、そのことを考慮して、移行元と移行先で CSV 所有者を一致させ、最適な転送パスと速度が提供されるようにする必要があります。

Azure Stack HCI クラスターで次の手順を実行して VM をインポートし、高可用性にしてから、それらを開始します。

1. 次のコマンドレットを実行して、すべての CSV 所有者ノードを表示します。

    ```powershell
    Get-ClusterSharedVolume
    ```

1. サーバー ノードごとに、`C:\Clusterstorage\Volume` に移動して、すべての VM のパスを設定します (例: `C:\Clusterstorage\volume01`)。

1. VM をインポートする前に、各 CSV 所有者ノードで次のコマンドレットを実行し、ボリュームごとにすべての VM VMCX ファイルへのパスを表示します。 環境に合わせてパスを変更します。

    ```powershell
    Get-ChildItem -Path "C:\Clusterstorage\Volume01\*.vmcx" -Recurse
    ```

    > [!NOTE]
    > Windows Server 2012 R2 以前の VM では、VCMX ファイルではなく XML ファイルが使用されています。 詳細については、「**古い VM の移行**」セクションを参照してください。

1. サーバー ノードごとに次のコマンドレットを実行することにより、各 CSV 所有者ノードに VM をインポートして、登録し、高可用性にします。 これにより、プロセッサとメモリの割り当てが最適になるように VM が均等に分散されます。

    ```powershell
    Get-ChildItem -Path "C:\Clusterstorage\Volume01\*.vmcx" -Recurse | Import-VM -Register | Get-VM | Add-ClusterVirtualMachineRole
    ```

1. 各ノードで各移行先 VM を起動します。

    ```powershell
    Start-VM -Name
    ```

1. ログオンし、すべての VM が実行されていること、およびすべてのアプリとデータが存在することを確認します。

    ```powershell
    Get-VM -ComputerName Server01 | Where-Object {$_.State -eq 'Running'}
    ```

1. すべての機能強化を利用するため、VM を Azure Stack HCI の最新バージョンに更新します。

    ```powershell
    Get-VM | Update-VMVersion -Force
    ```

1. スクリプトが完了した後、Robocopy のログ ファイルでエラーを調べて、すべての VM が正常にコピーされたことを確認します。

## <a name="migrating-older-vms"></a>古い VM の移行

Windows Server 2008 SP1、Windows Server 2008 R2-SP1、Windows Server 2012、または Windows Server 2012 R2 の VM がある場合は、このセクションが適用されます。 これらの VM を処理するには、次の 2 つのオプションがあります。

- これらの VM をまず Windows Server 2012 R2、Windows Server 2016、または Windows Server 2019 に移行し、VM のバージョンを更新してから、移行プロセスを開始します。

- Robocopy を使用し、すべての VM の VHD を Azure Stack HCI にコピーします。 その後、新しい VM を作成し、コピーした VHD を Azure Stack HCI の VM にアタッチします。 こうすることで、このような古い VM での VM のバージョンの制限が回避されます。

Windows Server 2012 R2 以前の Hyper-V ホストでは、VM の構成に XML ファイル形式が使用されています。これは、Windows Server 2016 以降の Hyper-V ホストで使用されている VCMX ファイル形式とは異なります。 これにより、これらの VM を Azure Stack HCI にコピーするには、異なる Robocopy コマンドが必要です。

### <a name="option-1-staged-migration"></a>オプション 1: 段階的な移行

これは、Windows Server 2008 SP1、Windows Server 2008 R2-SP、および Windows Server 2012 でホストされている VM に使用される 2 段階の移行です。 使用するプロセスは次のとおりです。

1. コピーするすべての VM の VHD ファイルと VHDX ファイルの場所を検出した後、`vmpaths.txt` ファイルを調べて、Robocopy を開始する最上位のソース ファイル パスを確認します。 次のコマンドレットを使用します。

    ```powershell
    Get-ChildItem -Path "C:\Clusterstorage\Volume01\*.vhd*" -Recurse > c:\vmpaths.txt
    ```

1. 最初に、次の例の Robocopy コマンドを使用し、ステップ 1 で確認した最上位のパスを使用して VM を Windows Server 2012 R2 にコピーします。

    `Robocopy \\2012R2-Clus01\c$\clusterstorage\volume01\Hyper-V\ \\20H2-Clus01\c$\clusterstorage\volume01\Hyper-V\ /E /MT:32 /R:0 /w:1 /NFL /NDL /copyall /log:c:\log.txt /xf`

1. Windows Server 2012 R2 クラスターで使用されている仮想スイッチ (`VMSwitch`) の名前が、Windows 2008 R2 または Windows Server 2008 R2-SP1 の移行元で使用されているスイッチの名前と同じであることを確認します。 クラスター内のすべてのサーバーで使用されているスイッチ名を表示するには、次のようにします。

     ```powershell
    Get-VMSwitch -CimSession $Servers | Select-Object Name
    ```

    必要に応じて、Windows Server 20212 R2 でスイッチの名前を変更します。 クラスター内のすべてのサーバーでスイッチの名前を変更するには、次のようにします。

    ```powershell
    Invoke-Command -ComputerName $Servers -ScriptBlock {rename-VMSwitch -Name $using:vSwitcholdName -NewName $using:vSwitchnewname}
    ```

1. VM をコピーして Windows Server 2012 R2 にインポートします。

     ```powershell
    Get-ChildItem -Path "c:\clusterstorage\volume01\Hyper-V\*.xml"-Recurse
    ```

    ```powershell
    Get-ChildItem -Path "c:\clusterstorage\volume01\image\*.xml" -Recurse    | Import-VM -Register | Get-VM | Add-ClusterVirtualMachineRole  
    ```

1. Windows Server 2012 R2 で、すべての VM のバージョンを 5.0 に更新します。

    ```powershell
    Get-VM | Update-VMVersion -Force
    ```

1. [移行スクリプトを実行](#run-the-migration-script)して、VM を Azure Stack HCI にコピーします。

1. 「[VM をインポートする](#import-the-vms)」の手順に従います。XML ファイルを処理して VM を Azure Stack HCI にインポートするには、ステップ 3 とステップ 4 を次のように置き換えます。

    ```powershell
    Get-ChildItem -Path "c:\clusterstorage\volume01\Hyper-V\*.xml"-Recurse
    ```

    ```powershell
    Get-ChildItem -Path "c:\clusterstorage\volume01\image\*.xml" -Recurse    | Import-VM -Register | Get-VM | Add-ClusterVirtualMachineRole  
    ```

1. 「[VM をインポートする](#import-the-vms)」の残りの手順を完了します。

### <a name="option-2-direct-vhd-copy"></a>オプション 2:VHD の直接コピー

この方法では、Robocopy を使用して、Windows 2008 SP1、Windows 2008 R2-SP1、Windows 2012 でホストされている VM の VHD を Azure Stack HCI にコピーします。 これにより、このような古い VM でのサポートされる最小 VM バージョンの制限が回避されます。 Windows Server 2008 SP1 および Windows Server 2008 R2-SP1 でホストされている VM には、このオプションを使用することをお勧めします。

Windows 2008 SP1 および Windows 2008 R2-SP1 でホストされている VM でサポートされているのは、第 1 世代の VHD を使用する第 1 世代の VM だけです。 そのため、コピーされた VHD を新しい VM にアタッチできるように、対応する第 1 世代の VM を Azure Stack HCI 上に作成する必要があります。 これらの VHD は第 2 世代の VHD にアップグレードできないことに注意してください。

> [!NOTE]
> Windows Server 2012 では、第 1 世代と第 2 世代両方の VM がサポートされています。

使用するプロセスは次のとおりです。

1. Robocopy の例を使用して、VM の VHD を Azure Stack HCI に直接コピーします。

    `Robocopy \\2012R2-Clus01\c$\clusterstorage\volume01\Hyper-V\ \\20H2-Clus01\c$\clusterstorage\volume01\Hyper-V\ /E /MT:32 /R:0 /w:1 /NFL /NDL /copyall /log:c:\log.txt /xf`

1. 第 1 世代の新しい VM を作成します。 これを行う方法の詳細については、[VM の管理](../manage/vm.md)に関するページを参照してください。

1. コピーした VHD ファイルを新しい VM にアタッチします。 詳細については、「[仮想ハード ディスク (VHD) の管理](/windows-server/storage/disk-management/manage-virtual-hard-disks)」を参照してください

参考までに、次の Windows Server ゲスト オペレーティング システムでは第 2 世代の VM がサポートされています。

- Windows Server 2019
- Windows Server 2016
- Windows Server 2012 R2
- Windows Server 2012
- Windows 10
- 64 ビット バージョンの Windows 8.1 (64 ビット)
- 64 ビット バージョンの Windows 8 (64 ビット)
- Linux ([サポートされる Linux と FreeBSD の VM](/windows-server/virtualization/hyper-v/Supported-Linux-and-FreeBSD-virtual-machines-for-Hyper-V-on-Windows) に関するページを参照)

## <a name="next-steps"></a>次のステップ

- 移行後にクラスターを検証します。 「[Azure Stack HCI クラスターの検証](validate.md)」を参照してください。

- 同じハードウェアを使用してインプレースで Azure Stack HCI に移行する方法については、「[同じハードウェア上の Azure Stack HCI に移行する](migrate-cluster-same-hardware.md)」を参照してください。