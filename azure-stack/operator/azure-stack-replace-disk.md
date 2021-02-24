---
title: 物理ディスクを交換する
titleSuffix: Azure Stack Hub
description: Azure Stack Hub の物理ディスクを交換する方法について説明します。
author: PatAltimore
ms.topic: how-to
ms.date: 03/04/2020
ms.author: patricka
ms.reviewer: thoroet
ms.lastreviewed: 12/02/2019
ms.openlocfilehash: 17c8a963ec49ebefff08f56a3411e71ae9d68011
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97869493"
---
# <a name="replace-a-physical-disk-in-azure-stack-hub"></a>Azure Stack Hub の物理ディスクを交換する

この記事では、Azure Stack Hub で物理ディスクを交換する一般的なプロセスについて説明します。 物理ディスクが正常に機能しなくなった場合、早急に交換する必要があります。

> [!Note]  
> 物理データ ドライブを交換するために、スケール ユニット ノードをメンテナンス モード (ドレイン) にする必要は **ありません**。 物理ドライブを交換した後で、Azure Stack Hub 管理者ポータルを使用してスケール ユニット ノードを修復する必要もありません。 修復が必要な場合、詳細については、記事「[Azure Stack Hub スケール ユニット ノードのハードウェア コンポーネントを交換する](azure-stack-replace-component.md)」を参照してください。

この手順は、ホット スワップ可能なディスクを含むデプロイに使用できます。

実際のディスク交換手順は、ご利用の OEM (Original Equipment Manufacturer) ハードウェア ベンダーによって異なります。 お使いのシステムに特化した詳しい手順については、ベンダーの現場交換可能ユニット (FRU) ドキュメントをご覧ください。

## <a name="review-disk-alert-information"></a>ディスクのアラート情報の確認
ディスクが正常に機能しなくなると、物理ディスクへの接続が切断されたことを通知するアラートが表示されます。

![物理ディスクへの接続が切断されたことを示す Azure Stack Hub 管理のアラート](media/azure-stack-replace-disk/DiskAlert.png)

アラートを開くと、スケール ユニット ノードおよび交換が必要なディスクの厳密な物理スロットの場所が、アラートの説明に示されます。 Azure Stack Hub は、LED インジケーター機能を使用して、障害が発生したディスクを特定するのにとても役立ちます。

## <a name="replace-the-physical-disk"></a>物理ディスクを交換する

実際のディスク交換については、OEM ハードウェア ベンダーの FRU 手順に従ってください。

> [!note]
> 一度に 1 つのスケール ユニット ノードのディスクを交換します。 仮想ディスク修復ジョブが完了するのを待ってから、次のスケール ユニット ノードに移ります。

統合システムでサポートされていないディスクの使用を回避するために、ベンダーでサポートされていないディスクはシステムでブロックされます。 サポートされていないディスクを使用しようとすると、サポートされていないモデルまたはファームウェアであるためディスクが検疫された、というアラートが新たに表示されます。

ディスクを交換すると、Azure Stack Hub が自動的に新しいディスクを検出し、仮想ディスクの修復プロセスを開始します。

## <a name="check-the-status-of-virtual-disk-repair-using-azure-stack-hub-powershell"></a>Azure Stack Hub PowerShell を使用した仮想ディスクの修復状態の確認

ディスクの交換後、Azure Stack Hub PowerShell を使用して、仮想ディスクの正常性状態と修復ジョブの進行状況を監視できます。

1. Azure Stack Hub PowerShell がインストールされていることを確認してください。 詳細については、「[PowerShell for Azure Stack Hub をインストールする](powershell-install-az-module.md)」を参照してください。
2. オペレーターとして、PowerShell を使用して Azure Stack Hub に接続します。 詳細については、[オペレーターとして PowerShell を使用する Azure Stack Hub への接続](azure-stack-powershell-configure-admin.md)に関する記事を参照してください。
3. 仮想ディスクの正常性と修復状態を検証するには、次のコマンドレットを実行します。

    ```powershell  
    $scaleunit=Get-AzsScaleUnit
    $StorageSubSystem=Get-AzsStorageSubSystem -ScaleUnit $scaleunit.Name
    Get-AzsVolume -StorageSubSystem $StorageSubSystem.Name -ScaleUnit $scaleunit.name | Select-Object VolumeLabel, OperationalStatus, RepairStatus
    ```

    ![Powershell での Azure Stack Hub のボリュームの正常性](media/azure-stack-replace-disk/get-azure-stack-volumes-health.png)

4. Azure Stack Hub システムの状態を検証します。 手順については、「[Azure Stack Hub システムの状態を検証する](azure-stack-diagnostic-test.md)」を参照してください。
5. 必要に応じて、次のコマンドを実行して、交換した物理ディスクの状態を検証できます。

    ```powershell  
    $scaleunit=Get-AzsScaleUnit
    $StorageSubSystem=Get-AzsStorageSubSystem -ScaleUnit $scaleunit.Name

    Get-AzsDrive -StorageSubSystem $StorageSubSystem.Name -ScaleUnit $scaleunit.name | Sort-Object StorageNode,MediaType,PhysicalLocation | Format-Table Storagenode, Healthstatus, PhysicalLocation, Model, MediaType,  CapacityGB, CanPool, CannotPoolReason
    ```

    ![Powershell を使用する Azure Stack Hub の物理ディスクの交換](media/azure-stack-replace-disk/check-replaced-physical-disks-azure-stack.png)

## <a name="check-the-status-of-virtual-disk-repair-using-the-privileged-endpoint"></a>特権エンドポイントを使用した仮想ディスクの修復状態の確認

ディスクの交換後、特権エンドポイントを使用して、仮想ディスクの正常性状態と修復ジョブの進行を監視できます。 特権エンドポイントにネットワーク接続されている任意のコンピューターから、次の手順に従います。

1. Windows PowerShell セッションを開き、特権エンドポイントに接続します。

    ```powershell
        $cred = Get-Credential
        Enter-PSSession -ComputerName <IP_address_of_ERCS>`
          -ConfigurationName PrivilegedEndpoint -Credential $cred
    ```
  
2. 仮想ディスクの正常性を表示するには、次のコマンドを実行します。

    ```powershell
        Get-VirtualDisk -CimSession s-cluster
    ```

   ![Get-VirtualDisk コマンドの Powershell 出力](media/azure-stack-replace-disk/GetVirtualDiskOutput.png)

3. 現在の記憶域ジョブの状態を表示するには、次のコマンドを実行します。

    ```powershell
        Get-VirtualDisk -CimSession s-cluster | Get-StorageJob
    ```

    ![Get-StorageJob コマンドの Powershell 出力](media/azure-stack-replace-disk/GetStorageJobOutput.png)

4. Azure Stack Hub システムの状態を検証します。 手順については、「[Azure Stack Hub システムの状態を検証する](azure-stack-diagnostic-test.md)」を参照してください。

## <a name="troubleshoot-virtual-disk-repair-using-the-privileged-endpoint"></a>特権エンドポイントを使用した仮想ディスクの修復のトラブルシューティング

仮想ディスクの修復ジョブが進行しないようであれば、次のコマンドを実行してジョブを最初からやり直してください。

```powershell
Get-VirtualDisk -CimSession s-cluster | Repair-VirtualDisk
```
