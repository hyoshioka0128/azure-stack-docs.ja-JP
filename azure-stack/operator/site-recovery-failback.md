---
title: Azure Site Recovery フェールバック ツールのユーザー ガイド
description: Azure Site Recovery フェールバック ツールを使用して仮想マシン (VM) を保護する方法について説明します。
author: sethmanheim
ms.author: sethm
ms.date: 9/18/2020
ms.topic: how-to
ms.reviewer: rtiberiu
ms.lastreviewed: 9/18/2020
ms.openlocfilehash: 2b57527f3a65e97f5b83ada115faa63ace563ea4
ms.sourcegitcommit: 0f2852c3302c6723e7afad637f55b80359182ae3
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/25/2020
ms.locfileid: "91366281"
---
# <a name="azure-site-recovery-failback-tool"></a>Azure Site Recovery フェールバック ツール

接続されている環境では、Azure Site Recovery を使用して、Azure Stack Hub で実行されている仮想マシン (VM) を保護することができます。 [こちらの記事](/azure/site-recovery/azure-stack-site-recovery)で、環境の設定方法と、これらのワークロードに対する事業継続とディザスター リカバリーの全体的な戦略に Site Recovery を役立てる方法について説明しています。

障害が発生した場合、Azure Stack Hub オペレーターは、"*フェールオーバー*" 手順を実行します。Azure Stack Hub が再稼働したら、"*フェールバック*" プロセスを実行します。 フェールオーバー プロセスについては[こちらの Site Recovery に関する記事](/azure/site-recovery/azure-stack-site-recovery)に記載されていますが、フェールバック プロセスには手動の手順がいくつか必要です。

- Azure で実行されている VM を停止します。
- VHD をダウンロードします。
- Azure Stack Hub に VHD をアップロードします。
- VM を再作成します。
- 最後に、Azure Stack Hub で実行されている VM を起動します。 

このプロセスは間違いやすく、時間がかかる場合があるため、このプロセスの高速化と自動化を支援するスクリプトが作成されています。

## <a name="failback-procedure"></a>フェールバックの手順

自動フェールバック プロセスには、次の 3 つの主要部分があります。

- **Copy-AzSiteRecoveryVmVHD**:
  - Azure VM をシャットダウンします。
  - ディスクのエクスポートを準備します。
  - AzCopy または StorageBlobCopy を使用してディスクをコピーします。
  - ディスクを Azure Stack Hub ストレージ アカウントにアップロードします。

- ディスクがコピーされた後に、**Prepare-AzSiteRecoveryVMFailBack** によって対処される 2 つのシナリオがあります。
  - 元の Azure Stack Hub が復旧しました。 元の VM は引き続き存在するので、その VHD を変更するだけで済みます。
  - 障害が発生した場合、元の VM が失われているのであれば、VM 全体を再構築する必要があります。

  この手順では、テンプレートと必要なパラメーター ファイルを作成して、両方のシナリオに対処します。

- パラメーター ファイルを使用した Azure Resource Manager テンプレートの実際のデプロイと、Azure Stack Hub での VM のデプロイ/作成。

### <a name="prerequisites"></a>前提条件

フェールバック手順を実行するには、次の前提条件が要件です。

- [Azure Site Recovery フェールバック ツール](https://aka.ms/azshasr)をコピーします。

- PowerShell で FailbackTool.psm1 モジュールをインポートします。

- [こちらの記事](powershell-install-az-module.md)の手順に従って、Azure Stack Hub 用の Az モジュールをインストールします。

- (省略可能) [AzCopy バージョン 10 をダウンロードします](/azure/storage/common/storage-use-azcopy-v10)。

  - **AzCopy** を使用して BLOB をコピーする方が高速ですが、BLOB ファイルを一時的に格納する追加のローカル ディスク領域が必要になります。
  - **AzCopy** を使用しない場合、VHD コピーは **AzStorageBlobCopy** を使用して実行されます。 つまり、ローカル ストレージは不要ですが、プロセスにはより長い時間がかかります。

- Azure portal のリソースへのアクセス権と、Azure Stack Hub でこれらのリソースを作成するためのアクセス権。

## <a name="step-1-copy-blob-from-azure-to-azure-stack-hub"></a>手順 1:Azure から Azure Stack Hub に BLOB をコピーする

**Copy-AzSiteRecoveryVmVHD** PowerShell コマンドレットを呼び出して、Azure VM を停止し、Azure から VHD をダウンロードして Azure Stack Hub にアップロードします。 次に例を示します。

```powershell
$uris = Copy-AzSiteRecoveryVmVHD `
        -SourceVM $vmOnAzure `
        -TargetStorageAccountName "targetaccountName" `
        -TargetStorageEndpoint "redmond.ext-v.masd.stbtest.microsoft.com" `
        -TargetStorageAccountKey $accountKey `
        -AzCopyPath "C:\azcopy_v10\azcopy.exe" `
        -VhdLocalFolder "C:\tempfolder"
```

次の考慮事項に注意してください。

- この例では、`$uris` を使用して、手順 2 で使用される `SourceDiskVhdUris` 値を保持します。

- `-SourceVM` パラメーターは、`Get-AzVM` によって取得される VM オブジェクトです。
  - これは、Azure でフェールオーバーされた、Azure Stack Hub からの保護された VM です。
  - スクリプトによって VM がシャットダウンされるため、VM が実行されているかどうかは問題ではありません。 ただし、明示的にシャットダウンし、それに応じて VM 内のサービスを停止することをお勧めします。

- Azure Stack Hub 側のストレージ アカウントのアカウント キー (`TargetStorageAccountKey` を使用) または SAS トークン (`TargetStorageAccountSasToken` を使用) のいずれかを指定できます。 SAS トークンは、少なくとも次のアクセス許可を指定して、ストレージ アカウント レベルで作成する必要があります。

   :::image type="content" source="media/site-recovery-failback/sasperms.png" alt-text="SAS トークンのアクセス許可":::

- リージョンと FQDN を含むストレージ エンドポイント(`regionname.azurestack.microsoft.com` など)、または Azure Stack Hub の環境名 (`AzureStackTenant` など) を指定できます。 環境名が使用されている場合は、**Get-AzEnvironment** を使用してリストする必要があります。

- Azure から Azure Stack Hub に VHD をコピーするために、**AzCopy** または **AzStorageBlobCopy** を使用することを選択できます。 **AzCopy** の方が高速ですが、最初に VHD ファイルをローカル フォルダーにダウンロードする必要があります。
  - **AzCopy** を使用するには、パラメーター `-AzCopyPath` および `-VhdLocalFolder` (VHD がコピーされるパス) を指定します。
  - ローカルに十分な領域がない場合は、パラメーター `-AzCopyPath` および `-VhdLocalFolder` を省略して、**AzCopy** を使用せずに VHD を直接コピーすることができます。 既定では、このコマンドは **AzStorageBlobCopy** を使用して、Azure Stack Hub ストレージ アカウントに直接コピーします。

## <a name="step-2-generate-resource-manager-templates"></a>手順 2:Resource Manager テンプレートを生成する

ディスクがコピーされたら、**Prepare-AzSiteRecoveryVMFailBack** コマンドレットを使用して、Azure Stack Hub に VM をデプロイするために必要な `$templateFile` および `$parameterFile` を作成します。

```powershell
$templateFile, $parameterFile = Prepare-AzSiteRecoveryVMFailBack `
                                -SourceContextName "PublicAzure" `
                                -SourceVM $vmOnAzure `
                                -SourceDiskVhdUris $uris `
                                -TargetResourceLocation "redmond" `
                                -ArmTemplateDestinationPath "C:\ARMtemplates" `
                                -TargetVM $vmOnHub `
                                -TargetContextName "AzureStack"

```

次の考慮事項に注意してください。

- この例では、 (`$uris` を使用する) 手順 1 の戻り値として `-SourceDiskVhdUris` を使用します。

- このコマンドレットは、2 つのシナリオをサポートしています。
  - `-TargetVM` を指定して、VM が Azure Stack Hub 側でアクティブであると想定し、そのディスクを Azure からコピーした最新のものに置き換える必要があります。
  - スクリプトで、この VM をデプロイするための Resource Manager テンプレートが生成され、Azure Stack Hub から既存の VM が削除されます。
  
  > [!NOTE]
  > Azure Stack Hub の VM 自体を削除しても、他のオブジェクト (VNET、リソース グループ、NSG など) は削除されません。 VM リソース自体が削除されるだけです。その後、テンプレートは `-incremental` パラメーターを指定してデプロイされます。

  - `-TargetVM` パラメーターを指定しないことにより、スクリプトは VM が Azure Stack Hub 側に存在しなくなったと想定するため、スクリプトによって完全に新しい VM をデプロイする Resource Manager テンプレートが作成されます。

- 生成された Resource Manager テンプレート ファイルは `-ArmTemplateDestinationPath` に配置され、テンプレート ファイルまたはパラメーター ファイルの完全なパスが返されます。

- `-TargetVM` パラメーターが指定されている場合、コマンドレットにより VM が削除されるため、次の手順に進むことができます。

## <a name="step-3-deploy-the-resource-manager-template"></a>手順 3:Resource Manager テンプレートをデプロイする

この時点で、VHD が Azure Stack Hub にアップロードされ、Resource Manager テンプレートとそれぞれのパラメーター ファイルが作成されます。 あとは、Azure Stack Hub に VM をデプロイするだけです。

シナリオによっては、このテンプレートを編集して、一部の名前またはリソースを追加、削除、変更したい場合があります。 これは許可されています。必要に応じてテンプレートを編集および調整できるためです。

準備ができたら、Resource Manager テンプレートのリソースが想定どおりであることを確認した後、**New-AzResourceGroupDeployment** コマンドレットを呼び出してリソースをデプロイできます。 次に例を示します。

```powershell
New-AzResourceGroupDeployment `
  -Name "Failback" `
  -ResourceGroupName "failbackrg" `
  -TemplateFile $templateFile `
  -TemplateParameterFile $parameterFile `
  -Mode Incremental
```

次の考慮事項に注意してください。

- `-ResourceGroupName` パラメーターは既存のリソース グループである必要があります。
- `-TemplateFile` および `-TemplateParameterFile` パラメーターは、手順 2 の戻り値から取得されます。

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub VM の機能](../user/azure-stack-vm-considerations.md)
- [Azure Stack Hub に対してカスタム VM イメージを追加または削除する](azure-stack-add-vm-image.md)
- [Azure Stack Hub 内で PowerShell を使用して Windows VM を作成する](../user/azure-stack-quick-create-vm-windows-powershell.md)