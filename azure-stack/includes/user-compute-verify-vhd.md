---
author: mattbriggs
ms.author: mabrigg
ms.service: azure-stack
ms.topic: include
ms.date: 2/1/2021
ms.reviewer: thoroet
ms.lastreviewed: 08/04/2020
ms.openlocfilehash: a1659bb40e3917aa548a6df5b031562b6913802a
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99245786"
---
VHD をアップロードする前に、VHD が要件を満たしていることを確認する必要があります。 要件を満たしていない VHD は Azure Stack Hub に読み込むことができません。

1. Hyper-V で検出された PowerShell モジュールを使用します。 Hyper-V をアクティブ化すると、サポートする PowerShell モジュールがインストールされます。 そのモジュールがあることを確認するには、管理者特権のプロンプトで PowerShell を開き、次のコマンドレットを実行します。

    ```powershell  
    Get-Command -Module hyper-v
    ```

    Hyper-V コマンドを使用できない場合は、「[Hyper-V と Windows PowerShell の使用](/virtualization/hyper-v-on-windows/quick-start/try-hyper-v-powershell)」を参照してください。 

2. マシン上の VHD へのパスを取得します。 次のコマンドレットを実行します。

    ```powershell  
    get-vhd <path-to-your-VHD>
    ```

    このコマンドレットは VHD オブジェクトを返し、次のような属性を表示します。
    
    ```powershell  
    ComputerName            : YOURMACHINENAME
    Path                    : <path-to-your-VHD>
    VhdFormat               : VHD
    VhdType                 : Fixed
    FileSize                : 68719477248
    Size                    : 68719476736
    MinimumSize             : 32212254720
    LogicalSectorSize       : 512
    PhysicalSectorSize      : 512
    BlockSize               : 0
    ParentPath              :
    DiskIdentifier          : 3C084D21-652A-4C0E-B2D1-63A8E8E64C0C
    FragmentationPercentage : 0
    Alignment               : 1
    Attached                : False
    DiskNumber              :
    IsPMEMCompatible        : False
    AddressAbstractionType  : None
    Number                  :
    ```

3. VHD オブジェクトで、Azure Stack Hub の要件が満たされていることを確認します。
    - [VHD が固定型である。](#vhd-is-of-fixed-type)
    - [VHD の最小仮想サイズが 20 MB 以上である。](#vhd-has-minimum-virtual-size-of-at-least-20-mb)
    - [VHD がアラインされている。](#vhd-is-aligned)
    - [VHD BLOB の長さ = 仮想サイズ + VHD フッターの長さ (512)。](#vhd-blob-length) 
    
    また、Azure Stack Hub は、[第 1 世代 (1) の VM](#generation-one-vms) からのイメージのみサポートします。

4. VHD が Azure Stack Hub と互換性がない場合は、ソース イメージと Hyper-V に戻り、要件を満たす VHD を作成し、アップロードする必要があります。 アップロード プロセスでの破損の可能性を最小限に抑えるために、AzCopy を使用します。

### <a name="how-to-fix-your-vhd"></a>VHD を修正する方法

お使いの VHD と Azure Stack Hub と の互換性を確保するために、次の要件を満たす必要があります。

#### <a name="vhd-is-of-fixed-type"></a>VHD が固定型である
**識別**:`get-vhd` コマンドレットを使用して、VHD オブジェクトを取得します。  
**解決策**:VHDX ファイルを VHD に変換し、容量可変ディスクを容量固定ディスクに変換することはできますが、VM の世代を変更することはできません。
[Hyper-V マネージャーまたは PowerShell](/azure/virtual-machines/windows/prepare-for-upload-vhd-image#use-hyper-v-manager-to-convert-the-disk) を使用してディスクを変換します。

### <a name="vhd-has-minimum-virtual-size-of-at-least-20-mb"></a>VHD の最小仮想サイズが 20 MB 以上である
**識別**:`get-vhd` コマンドレットを使用して、VHD オブジェクトを取得します。  
**解決策**:[Hyper-V マネージャーまたは PowerShell](/azure/virtual-machines/windows/prepare-for-upload-vhd-image#use-hyper-v-manager-to-resize-the-disk) を使用してディスクのサイズを変更します。 

### <a name="vhd-is-aligned"></a>VHD がアラインされている
**識別**:`get-vhd` コマンドレットを使用して、VHD オブジェクトを取得します。  
**解決策**:仮想サイズは 1 MB の倍数である必要があります。 

ディスクの仮想サイズは、1 MiB にアラインする必要があります。 VHD に 1 MiB の端数がある場合は、1 MiB の倍数になるようにディスクのサイズを変更する必要があります。 MiB の端数があるディスクでは、アップロードした VHD からイメージを作成する際にエラーが発生します。 サイズを確認するには、PowerShell の Get-VHD コマンドレットを使用し、"Size" (Azure で 1 MiB の倍数である必要があります) と "FileSize" ("Size" に VHD フッターの 512 バイトを足した値に等しくなります) を表示します。

[Hyper-V マネージャーまたは PowerShell](/azure/virtual-machines/windows/prepare-for-upload-vhd-image#use-hyper-v-manager-to-resize-the-disk) を使用してディスクのサイズを変更します。 


### <a name="vhd-blob-length"></a>VHD BLOB の長さ
**識別**:`get-vhd` コマンドレットを使用して、`Size`  を表示します。  
**解決策**:VHD BLOB の長さ = 仮想サイズ + VHD フッターの長さ (512)。 BLOB 末尾の小さなフッターに、VHD のプロパティが記述されます。 `Size` は、Azure では 1 MiB の倍数である必要があり、`FileSize` は、`Size` に VHD フッターの 512 バイトを足した値に等しくなります。

[Hyper-V マネージャーまたは PowerShell](/azure/virtual-machines/windows/prepare-for-upload-vhd-image#use-hyper-v-manager-to-resize-the-disk) を使用してディスクのサイズを変更します。 

### <a name="generation-one-vms"></a>第 1 世代の VM
**識別**:お使いの仮想マシンが第 1 世代であるかどうかを確認するには、`Get-VM | Format-Table Name, Generation` コマンドレットを使用します。  
**解決策**:ハイパーバイザー (Hyper-V) で VM を再作成する必要があります。