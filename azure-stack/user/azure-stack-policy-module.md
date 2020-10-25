---
title: Azure Stack Hub ポリシー モジュールを使用する
description: Azure Stack Hub サブスクリプションと同様の動作をするように、Azure サブスクリプションを制限する方法を説明します。
author: sethmanheim
ms.topic: article
ms.date: 06/09/2020
ms.author: sethm
ms.lastreviewed: 03/26/2019
ms.openlocfilehash: f39bbf689cd3b847b29c2d5b046721029078a5dd
ms.sourcegitcommit: d91e47a51a02042f700c6a420f526f511a6db9a0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/10/2020
ms.locfileid: "84666483"
---
# <a name="manage-azure-policy-using-the-azure-stack-hub-policy-module"></a>Azure Stack Hub ポリシー モジュールを使用して Azure ポリシーを管理する

Azure Stack Hub ポリシー モジュールを使用すると、Azure Stack Hub と同じバージョン管理およびサービス可用性を利用して Azure サブスクリプションを構成できます。 このモジュールでは、[**New-AzureRmPolicyDefinition**](/powershell/module/azurerm.resources/new-azurermpolicydefinition) PowerShell コマンドレットを使用して、サブスクリプションで使用できるリソースの種類とサービスを制限する Azure ポリシーを作成します。 次に、[**New-AzureRmPolicyAssignment**](/powershell/module/azurerm.resources/new-azurermpolicyassignment) コマンドレットを使用して、適切な範囲内のポリシー割り当てを作成します。 ポリシーの構成が完了すると、Azure サブスクリプションを使用して Azure Stack Hub を対象とするアプリを開発できます。

## <a name="install-the-module"></a>モジュールのインストール

1. 「[PowerShell for Azure Stack Hub のインストール](../operator/azure-stack-powershell-install.md)」の手順 1 の説明に従って、必要なバージョンの AzureRM PowerShell モジュールをインストールします。
2. [GitHub から Azure Stack Hub ツールをダウンロードします](../operator/azure-stack-powershell-download.md)。
3. [Azure Stack Hub で使用するために PowerShell を構成します](azure-stack-powershell-configure-user.md)。
4. **AzureStack.Policy.psm1** モジュールをインポートします。

   ```powershell
   Import-Module .\Policy\AzureStack.Policy.psm1
   ```

## <a name="apply-policy-to-azure-subscription"></a>Azure サブスクリプションにポリシーを適用する

以下のコマンドを使用して、Azure サブスクリプションに既定の Azure Stack Hub ポリシーを適用できます。 これらのコマンドを実行する前に、`Azure subscription name` を自分の Azure サブスクリプションの名前に置き換えてください。

```powershell
Add-AzureRmAccount
$s = Select-AzureRmSubscription -SubscriptionName "Azure subscription name"
$policy = New-AzureRmPolicyDefinition -Name AzureStackPolicyDefinition -Policy (Get-AzsPolicy)
$subscriptionID = $s.Subscription.SubscriptionId
New-AzureRmPolicyAssignment -Name AzureStack -PolicyDefinition $policy -Scope /subscriptions/$subscriptionID
```

## <a name="apply-policy-to-a-resource-group"></a>リソース グループにポリシーを適用する

さらに詳細なポリシーを適用する必要がある場合があります。 たとえば、同じサブスクリプションで他のリソースが実行されている場合があります。 ポリシーの適用範囲を特定のリソース グループに設定できます。そうすることで、Azure リソースを使用して Azure Stack Hub 用のアプリをテストできます。 次のコマンドを実行する前に、`Azure subscription name` を自分の Azure サブスクリプションの名前に置き換えてください。

```powershell
Add-AzureRmAccount
$rgName = 'myRG01'
$s = Select-AzureRmSubscription -SubscriptionName "Azure subscription name"
$policy = New-AzureRmPolicyDefinition -Name AzureStackPolicyDefinition -Policy (Get-AzsPolicy)
$subscriptionID = $s.Subscription.SubscriptionId
New-AzureRmPolicyAssignment -Name AzureStack -PolicyDefinition $policy -Scope /subscriptions/$subscriptionID/resourceGroups/$rgName
```

## <a name="policy-in-action"></a>実行中のポリシー

Azure ポリシーをデプロイした後、このポリシーで禁止されているリソースをデプロイしようとすると、エラーが表示されます。

![ポリシーの制約によるリソースのデプロイの失敗の結果](./media/azure-stack-policy-module/image1.png)

## <a name="next-steps"></a>次のステップ

* [PowerShell を使用したテンプレートのデプロイ](azure-stack-deploy-template-powershell.md)
* [Azure CLI を使用したテンプレートのデプロイ](azure-stack-deploy-template-command-line.md)
* [Visual Studio を使用したテンプレートのデプロイ](azure-stack-deploy-template-visual-studio.md)
