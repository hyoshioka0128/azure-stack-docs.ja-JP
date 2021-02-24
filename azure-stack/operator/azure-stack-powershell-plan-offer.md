---
title: Azure Stack Hub で PowerShell を使用してサブスクリプション、プラン、およびオファーを管理する
description: Azure Stack Hub で PowerShell を使用してサブスクリプション、プラン、およびオファーを管理する方法。
author: PatAltimore
ms.topic: how-to
ms.date: 12/18/2020
ms.author: patricka
ms.lastreviewed: 12/18/2020
ms.reviewer: bganapa
ms.openlocfilehash: 3dcc3eae565f2e2a41078c2696a09f60ffa618b9
ms.sourcegitcommit: e13f27291bab236aac5d8b05401056961e9cc1e9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/19/2020
ms.locfileid: "97697867"
---
# <a name="use-powershell-to-manage-subscriptions-plans-and-offers-in-azure-stack-hub"></a>Azure Stack Hub で PowerShell を使用してサブスクリプション、プラン、およびオファーを管理する

オファー、プラン、およびサブスクリプションを使用することで、PowerShell でサービスを構成して提供できます。 PowerShell の使用。 Azure Stack Hub で PowerShell を使用してセットアップする手順については、「[Azure Stack Hub 用の PowerShell Az モジュールをインストールする](powershell-install-az-module.md)」を参照してください。 PowerShell を使用した Azure Stack Hub への接続に関する情報については、「[PowerShell を使用して Azure Stack Hub に接続する](azure-stack-powershell-configure-admin.md)」を参照してください。

開始する前に、Azure Stack Hub PowerShell モジュールが読み込まれていることを確認します。 PowerShell コンソールで、「`Import-Module AzureStack`」と入力します。

## <a name="create-a-plan"></a>プランの作成

プランを作成する際は、クォータが必要です。 既存のクォータを使用することも、新しいクォータを作成することもできます。 たとえば、ストレージ、コンピューティング、およびネットワークのクォータを作成するには、[New-AzsStorageQuota](/powershell/module/azs.storage.admin/new-azsstoragequota)、[New-AzsComputeQuota](/powershell/module/azs.compute.admin/new-azscomputequota)、および [New-AzsNetworkQuota](/powershell/module/azs.network.admin/new-azsnetworkquota) コマンドレットを使用できます。

```powershell
$serviceQuotas  = @()
$serviceQuotas += (New-AzsStorageQuota -Name "Example storage quota with defaults").Id
$serviceQuotas += (New-AzsComputeQuota -Name "Example compute quota with defaults").Id
$serviceQuotas += (New-AzsNetworkQuota -Name "Example network quota with defaults").Id
```

基本またはアドオン プランを作成または更新するには、[New-AzsPlan](/powershell/module/azs.subscriptions.admin/new-azsplan) を使用します。

```powershell
$testPlan = New-AzsPlan -Name "testplan" -ResourceGroupName "testrg" -QuotaIds $serviceQuotas -Description "Test plan"
```

## <a name="create-an-offer"></a>オファーの作成

オファーを作成するには、[New-AzsOffer](/powershell/module/azs.subscriptions.admin/new-azsoffer) を使用します。

```powershell
New-AzsOffer -Name "testoffer" -ResourceGroupName "testrg" -BasePlanIds @($testPlan.Id)
```

オファーを作成したら、プランをオファーに追加できます。 [Add-AzsPlanToOffer](/powershell/module/azs.subscriptions.admin/add-azsplantooffer) を使用します。 **-PlanLinkType** パラメーターによって、プランの種類が区別されます。

```powershell
Add-AzsPlanToOffer -PlanName "addonplan" -PlanLinkType Addon -OfferName "testoffer" -ResourceGroupName "testrg" -MaxAcquisitionCount 18
```

オファーの状態を変更する場合は、[Set-AzsOffer](/powershell/module/azs.subscriptions.admin/set-azsoffer) コマンドレットを使用します。

```powershell
$offer = Get-AzsAdminManagedOffer -Name "testoffer" -ResourceGroupName "testrg"
$offer.state = "Public"
$offer | Set-AzsOffer -Confirm:$false
```

## <a name="create-subscription-to-an-offer"></a>オファーに対するサブスクリプションを作成する

オファーを作成した後、ユーザーは、それを使用できるようにするために、そのオファーに対するサブスクリプションが必要になります。 ユーザーがオファーをサブスクライブできる方法は 2 つあります。

* クラウド オペレーターとして、ユーザー用のサブスクリプションを作成できます。 作成したサブスクリプションは、パブリック オファーとプライベート オファーの両方に対応します。
* ユーザーとして、パブリック オファーをサブスクライブできます。

クラウド オペレーターとしてユーザー用のサブスクリプションを作成するには、[New-AzsUserSubscription](/powershell/module/azs.subscriptions.admin/new-azsusersubscription) を使用します。

```powershell
New-AzsUserSubscription -Owner "user@contoso.com" -DisplayName "User subscription" -OfferId "/subscriptions/<Subscription ID>/resourceGroups/testrg/providers/Microsoft.Subscriptions.Admin/offers/testoffer"
```

ユーザーとしてパブリック オファーをサブスクライブするには、[New-AzsSubscription](/powershell/module/azs.subscriptions/new-azssubscription) を使用します。 *New-AzsSubscription* では、ユーザーの Azure Resource Manager 環境への接続が必要です。 「[PowerShell を使用して Azure Stack Hub に接続する](azure-stack-powershell-configure-admin.md)」の手順を使用しますが、ユーザーの Azure Resource Manager エンドポイントを使用します。 たとえば、「 `Add-AzEnvironment -Name "AzureStackUser" -ArmEndpoint "https://management.local.azurestack.external"` 」のように入力します。

```powershell
$testOffer = Get-AzsOffer | Where-Object Name -eq "testoffer"
New-AzsSubscription -OfferId "User subscription" -OfferId $testOffer.Id -DisplayName "My subscription"
```

## <a name="delete-quotas-plans-offers-and-subscriptions"></a>クォータ、プラン、オファーおよびサブスクリプションの削除

Azure Stack Hub のクォータ、プラン、オファー、およびサブスクリプションを削除するためのコンパニオン PowerShell コマンドレットがあります。 それぞれの例を次に示します。

オファーからサブスクリプションを削除するには、[Remove-AzsUserSubscription](/powershell/module/azs.subscriptions.admin/remove-azsusersubscription) を使用します。

```powershell
Remove-AzsUserSubscription -TargetSubscriptionId "c90173b1-de7a-4b1d-8600-b8325ca1eab1e"
```

オファーからプランを削除するには、[Remove-AzsPlanFromOffer](/powershell/module/azs.subscriptions.admin/remove-azsplanfromoffer) を使用します。

```powershell
Remove-AzsPlanFromOffer -PlanName "addonplan" -PlanLinkType Addon -OfferName "testoffer" -ResourceGroupName "testrg"
Remove-AzsPlanFromOffer -PlanName "testplan" -PlanLinkType Base -OfferName "testoffer" -ResourceGroupName "testrg"
```

プランを削除するには、[Remove-AzsPlan](/powershell/module/azs.subscriptions.admin/remove-azsplan) を使用します。

```powershell
Remove-AzsPlan -Name "testplan" -ResourceGroupName "testrg"
```

オファーを削除するには、[Remove-AzsOffer](/powershell/module/azs.subscriptions.admin/remove-azsoffer) を使用します。

```powershell
Remove-AzsOffer -Name "testoffer" -ResourceGroupName "testrg"
```

クォータを削除するには、[Remove-AzsStorageQuota](/powershell/module/azs.storage.admin/remove-azsstoragequota)、[Remove-AzsComputeQuota](/powershell/module/azs.compute.admin/remove-azscomputequota)、[Remove-AzsNetworkQuota](/powershell/module/azs.network.admin/remove-azsnetworkquota) を使用します。

```powershell
Remove-AzsStorageQuota -Name "Example storage quota with defaults"
Remove-AzsComputeQuota -Name "Example compute quota with defaults"
Remove-AzsNetworkQuota -Name "Example network quota with defaults"
```

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub での更新プログラム管理](./azure-stack-updates.md)
