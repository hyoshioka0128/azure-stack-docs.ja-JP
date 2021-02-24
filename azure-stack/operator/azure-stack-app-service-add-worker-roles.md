---
title: App Service on Azure Stack Hub 上で worker とインフラストラクチャを追加する
description: App Service on Azure Stack Hub をスケーリングするための詳細なガイダンス
author: bryanla
ms.topic: article
ms.date: 1/29/2020
ms.author: anwestg
ms.reviewer: anwestg
ms.lastreviewed: 1/29/2020
ms.openlocfilehash: 442831a05950342718c0ee0fc0ced2395627eca6
ms.sourcegitcommit: e88f0a1f2f4ed3bb8442bfb7b754d8b3a51319b4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/04/2021
ms.locfileid: "99534030"
---
# <a name="add-workers-and-infrastructure-in-azure-app-service-on-azure-stack-hub"></a>App Service on Azure Stack Hub で worker とインフラストラクチャを追加する

このドキュメントでは、App Service on Azure Stack Hub でインフラストラクチャおよび worker ロールをスケーリングする手順について説明します。 任意のサイズのアプリをサポートする追加の worker ロールを作成するのに必要なすべての手順について説明します。

> [!NOTE]
> Azure Stack Hub 環境の RAM が 96 GB 以下の場合、容量の追加が困難な可能性があります。

App Service on Azure Stack Hub は、既定で、無料の共有 worker 階層をサポートします。 他の worker 階層を追加するには、worker ロールを追加する必要があります。

既定の Azure Stack Hub インストール上の既定の Azure App Service でデプロイされる内容がわからない場合は、「[Azure Stack Hub 上の Azure App Service の概要](azure-stack-app-service-overview.md)」を参照してください。

App Service on Azure Stack Hub では、仮想マシン スケール セットを使用するすべてのロールをデプロイします。そのため、このワークロードのスケーリング機能を活用します。 したがって、worker 階層のすべてのスケーリングは、App Service 管理者を通して行われます。

## <a name="add-additional-workers-with-powershell"></a>PowerShell を使用した他の worker の追加



### <a name="az-modules"></a>[Az モジュール](#tab/az)

1. [PowerShell で Azure Stack Hub 管理環境をセットアップします。](azure-stack-powershell-configure-admin.md)

2. この例を使用して、スケール セットをスケールアウトします。

    ```powershell
    
    ##### Scale out the AppService Role instances ######
    
    # Set context to AzureStack admin.
    Connect-AzAccount -EnvironmentName AzureStackAdmin
                                                    
    ## Name of the Resource group where AppService is deployed.
    $AppServiceResourceGroupName = "AppService.local"
    
    ## Name of the ScaleSet : e.g. FrontEndsScaleSet, ManagementServersScaleSet, PublishersScaleSet , LargeWorkerTierScaleSet,      MediumWorkerTierScaleSet, SmallWorkerTierScaleSet, SharedWorkerTierScaleSet
    $ScaleSetName = "SharedWorkerTierScaleSet"
    
    ## TotalCapacity is sum of the instances needed at the end of operation. 
    ## e.g. if your VMSS has 1 instance(s) currently and you need 1 more the TotalCapacity should be set to 2
    $TotalCapacity = 2  
    
    # Get current scale set
    $vmss = Get-AzVmss -ResourceGroupName $AppServiceResourceGroupName -VMScaleSetName $ScaleSetName
    
    # Set and update the capacity
    $vmss.sku.capacity = $TotalCapacity
    Update-AzVmss -ResourceGroupName $AppServiceResourceGroupName -Name $ScaleSetName -VirtualMachineScaleSet $vmss 
    ```    

    > [!NOTE]
    > ロールの種類とインスタンスの数によっては、この手順が完了するまでに何時間もかかる場合があります。

3. App Service 管理で新しいロール インスタンスの状態を監視します。 個々のロール インスタンスの状態を確認するには、リスト内のロールの種類をクリックします。
### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm)

1. [PowerShell で Azure Stack Hub 管理環境をセットアップします。](azure-stack-powershell-configure-admin.md)

2. この例を使用して、スケール セットをスケールアウトします。

    ```powershell
    
    ##### Scale out the AppService Role instances ######
    
    # Set context to AzureStackAdmin admin.
    Login-AzureRMAccount -EnvironmentName AzureStackAdmin
                                                    
    ## Name of the Resource group where AppService is deployed.
    $AppServiceResourceGroupName = "AppService.local"
    
    ## Name of the ScaleSet : e.g. FrontEndsScaleSet, ManagementServersScaleSet, PublishersScaleSet , LargeWorkerTierScaleSet,      MediumWorkerTierScaleSet, SmallWorkerTierScaleSet, SharedWorkerTierScaleSet
    $ScaleSetName = "SharedWorkerTierScaleSet"
    
    ## TotalCapacity is sum of the instances needed at the end of operation. 
    ## e.g. if your VMSS has 1 instance(s) currently and you need 1 more the TotalCapacity should be set to 2
    $TotalCapacity = 2  
    
    # Get current scale set
    $vmss = Get-AzureRMVmss -ResourceGroupName $AppServiceResourceGroupName -VMScaleSetName $ScaleSetName
    
    # Set and update the capacity
    $vmss.sku.capacity = $TotalCapacity
    Update-AzureRMVmss -ResourceGroupName $AppServiceResourceGroupName -Name $ScaleSetName -VirtualMachineScaleSet $vmss 
    ```   

    > [!NOTE]
    > ロールの種類とインスタンスの数によっては、この手順が完了するまでに何時間もかかる場合があります。

3. App Service 管理で新しいロール インスタンスの状態を監視します。 個々のロール インスタンスの状態を確認するには、リスト内のロールの種類をクリックします。
---

## <a name="add-additional-workers-using-the-administrator-portal"></a>管理者ポータルを使用した他の worker の追加

1. サービス管理者として Azure Stack Hub 管理者ポータルにサインインします。

2. **[App Services]** を参照します。

    ![Azure Stack Hub 管理者ポータルの App Service](media/azure-stack-app-service-add-worker-roles/image01.png)

3. **[ロール]** をクリックします。 デプロイされているすべての App Service ロールの内訳が表示されます。

4. 拡大縮小する種類の行を右クリックし、**[ScaleSet]** をクリックします。

    ![Azure Stack Hub 管理者ポータルの ScaleSet App Service ロール](media/azure-stack-app-service-add-worker-roles/image02.png)

5. **[拡大縮小]** をクリックし、拡大縮小するインスタンス数を選択し、**[保存]** をクリックします。

    ![Azure Stack Hub 管理者ポータルの App Service ロールでスケーリングするためにインスタンスを設定する](media/azure-stack-app-service-add-worker-roles/image03.png)

6. App Service on Azure Stack Hub によって、VM が追加および構成され、必要なすべてのソフトウェアがインストールされ、このプロセスの完了時には準備完了とマークされます。 このプロセスには約 80 分かかる可能性があります。

7. **[ロール]** ブレードで worker を表示することで、新しいロールの準備状況を監視できます。

## <a name="result"></a>結果

完全にデプロイされ、準備ができると、worker をユーザーが利用できるようになり、それらにワークロードを展開できます。 既定で使用できる複数の価格レベルの例を次のスクリーンショットに示します。 特定の worker 階層に使用できる worker がない場合、対応する価格レベルを選択するオプションは使用できません。

![Azure Stack Hub 管理者ポータルの新しい App Service プランの価格レベル](media/azure-stack-app-service-add-worker-roles/image04.png)

>[!NOTE]
> 管理、フロントエンド、またはパブリッシャー ロールをスケール アウトするには、適切なロールの種類を選択して同じ手順を実行してください。 コントローラーはスケール セットとしてデプロイされていないため、すべての運用デプロイでインストール時に 2 つをデプロイする必要があります。

### <a name="next-steps"></a>次の手順

[デプロイ ソースを構成する](azure-stack-app-service-configure-deployment-sources.md)
