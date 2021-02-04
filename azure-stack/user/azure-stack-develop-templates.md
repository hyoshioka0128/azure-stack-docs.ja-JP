---
title: Azure Stack Hub 用のテンプレートの開発
description: Azure と Azure Stack Hub 間のアプリの移植性のために Azure Resource Manager テンプレートを開発する方法について説明します。
author: mattbriggs
ms.topic: article
ms.date: 2/1/2021
ms.author: mabrigg
ms.reviewer: unknown
ms.lastreviewed: 05/21/2019
ms.openlocfilehash: a99f3803f5b8ea8d23913efafd1f8d72005b49bd
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99247763"
---
# <a name="develop-templates-for-azure-stack-hub-with-azure-resource-manager"></a>Azure Resource Manager による Azure Stack Hub 用テンプレートの開発

アプリを開発するときは、Azure と Azure Stack Hub 間のテンプレートの移植性を確保することが重要です。 この記事では、[Azure Resource Manager テンプレート](https://download.microsoft.com/download/E/A/4/EA4017B5-F2ED-449A-897E-BD92E42479CE/Getting_Started_With_Azure_Resource_Manager_white_paper_EN_US.pdf)の開発に関する考慮事項について説明します。 これらのテンプレートを使用すると、Azure Stack Hub 環境にアクセスせずに、Azure でアプリのプロトタイプを作成し、デプロイをテストできます。

## <a name="resource-provider-availability"></a>リソース プロバイダーの可用性

デプロイする予定のテンプレートは、Azure Stack Hub で既に使用できるか、またはプレビューの段階にある Microsoft Azure サービスのみを使用する必要があります。

## <a name="public-namespaces"></a>パブリック名前空間

Azure Stack Hub はデータセンターでホストされるため、そのサービス エンドポイントの名前空間は、Azure パブリック クラウドとは異なります。 その結果、Azure Resource Manager テンプレートでハードコーディングされたパブリック エンドポイントは、Azure Stack Hub にデプロイしようとすると失敗します。 デプロイ中にリソース プロバイダーから値を取得するために、`reference` および `concatenate` 関数を使用してサービス エンドポイントを動的に構築できます。 たとえば、テンプレートに `blob.core.windows.net` をハードコーディングする代わりに、[primaryEndpoints.blob](https://github.com/Azure/AzureStack-QuickStart-Templates/blob/master/101-vm-windows-create/azuredeploy.json#L175) を取得して *osDisk.URI* エンドポイントを動的に設定します。

```json
"osDisk": {"name": "osdisk","vhd": {"uri":
"[concat(reference(concat('Microsoft.Storage/storageAccounts/', variables('storageAccountName')), '2015-06-15').primaryEndpoints.blob, variables('vmStorageAccountContainerName'),
 '/',variables('OSDiskName'),'.vhd')]"}}
```

## <a name="api-versioning"></a>API のバージョン管理

Azure のサービス バージョンが Azure と Azure Stack Hub で異なる場合があります。 各リソースには、提供される機能を定義する **apiVersion** 属性が必要です。 Azure Stack Hub での API のバージョンの互換性を確保するための各リソースプロバイダーで有効な API のバージョンは次のとおりです。

| リソース プロバイダー | apiVersion |
| --- | --- |
| Compute |**2015-06-15** |
| ネットワーク |**2015-06-15**、**2015-05-01-preview** |
| ストレージ |**2016-01-01**、**2015-06-15**、**2015-05-01-preview** |
| KeyVault | **2015-06-01** |
| App Service |**2015-08-01** |

## <a name="template-functions"></a>テンプレート関数

Azure Resource Manager の[関数](/azure/azure-resource-manager/resource-group-template-functions)は、動的テンプレートを構築するために必要な機能を提供します。 たとえば、次のようなタスクのために関数を使用できます。

* 文字列の連結やトリミング。
* その他のリソースからの値の参照。
* 複数のインスタンスをデプロイするためのリソースの反復処理。

次の関数は、Azure Stack Hub では使用できません。

* Skip
* Take

## <a name="resource-location"></a>リソースの場所

Azure Resource Manager テンプレートは、デプロイ時にリソースを配置するのに `location` 属性を使用します。 Azure では、場所は、米国西部や南アメリカなどのリージョンを指します。 Azure Stack Hub では、Azure Stack Hub はユーザーのデータセンター内にあるため、場所の意味が異なります。 Azure と Azure Stack Hub 間でテンプレートを確実に転送するには、個々のリソースをデプロイするときにリソース グループの場所を参照する必要があります。 そのためには、`[resourceGroup().Location]` を使用して、すべてのリソースがリソース グループの場所を継承するようにします。 次のコードは、ストレージ アカウントのデプロイ中にこの関数を使用する例を示しています。

```json
"resources": [
{
  "name": "[variables('storageAccountName')]",
  "type": "Microsoft.Storage/storageAccounts",
  "apiVersion": "[variables('apiVersionStorage')]",
  "location": "[resourceGroup().location]",
  "comments": "This storage account is used to store the VM disks",
  "properties": {
  "accountType": "Standard_GRS"
  }
}
]
```

## <a name="next-steps"></a>次のステップ

* [PowerShell を使用したテンプレートのデプロイ](azure-stack-deploy-template-powershell.md)
* [Azure CLI を使用したテンプレートのデプロイ](azure-stack-deploy-template-command-line.md)
* [Visual Studio を使用したテンプレートのデプロイ](azure-stack-deploy-template-visual-studio.md)
