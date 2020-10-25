---
title: Azure Stack Hub に使用状況の追跡のためのテナントを登録する
description: Azure Stack Hub でのテナントの登録方法とテナントの使用状況の追跡方法について学習します。
author: sethmanheim
ms.topic: article
ms.date: 09/01/2020
ms.author: sethm
ms.reviewer: alfredop
ms.lastreviewed: 10/14/2019
ms.openlocfilehash: 66a21943e19cef13aa7a3986b6a058f69cc85793
ms.sourcegitcommit: 0714ce748e20065b52f8283d5dbba7ab068978d1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/01/2020
ms.locfileid: "89285500"
---
# <a name="register-tenants-for-usage-tracking-in-azure-stack-hub"></a>Azure Stack Hub に使用状況の追跡のためのテナントを登録する

この記事では、登録操作について詳しく説明します。 これらの操作を使用して次のことができます。

- テナントの登録を管理する。
- テナントの使用状況の追跡を管理する。

## <a name="add-tenant-to-registration"></a>テナントを登録に追加する

この操作は、ご自分の登録に新しいテナントを追加するときに使用します。 テナントの使用状況は、Azure Active Directory (Azure AD) テナントに接続されている Azure サブスクリプションの下で報告されます。

この操作は、テナントに関連付けられているサブスクリプションを変更する場合にも使用できます。 以前のマッピングを上書きするには、PUT または **New-AzureRMResource** PowerShell コマンドレットを呼び出します。

テナントと関連付けることができる Azure サブスクリプションは 1 つだけです。 既存のテナントに第 2 のサブスクリプションを追加すると、最初のサブスクリプションは上書きされます。

### <a name="use-api-profiles"></a>API プロファイルの使用

次の登録コマンドレットでは、PowerShell を実行するときに API プロファイルを指定する必要があります。 API プロファイルは、一連の Azure リソース プロバイダーとその API バージョンを表します。 これにより、複数の Azure クラウドを対話操作するときに、適切なバージョンの API を使用できます。 たとえば、グローバルな Azure と Azure Stack Hub を操作するときに複数のクラウドを操作する場合、API プロファイルでは、そのリリース日と一致する名前が指定されます。 **2017-09-03** プロファイルを使用します。

Azure Stack Hub と API プロファイルの詳細については、「[Azure Stack Hub での API バージョンのプロファイルの管理](../user/azure-stack-version-profiles.md)」を参照してください。

### <a name="parameters"></a>パラメーター

| パラメーター                  | 説明 |
|---                         | --- |
| registrationSubscriptionID | 初期登録に使用された Azure サブスクリプション。 |
| customerSubscriptionID     | 登録される顧客に属する Azure サブスクリプション (Azure Stack Hub ではない)。 パートナー センターを使用してクラウド ソリューション プロバイダー (CSP) オファーで作成されている必要があります。 顧客が複数のテナントを持っている場合は、テナントが Azure Stack Hub にサインインするためのサブスクリプションを作成します。 顧客サブスクリプション ID は大文字と小文字の区別があります。 |
| resourceGroup              | 登録が格納されている Azure 内のリソース グループ。 |
| registrationName           | お使いの Azure Stack Hub の登録名。 Azure に格納されているオブジェクトです。 通常、名前は **azurestack-CloudID** の形式で、**CloudID** はお使いの Azure Stack Hub デプロイのクラウド ID です。 |

> [!NOTE]  
> テナントは、それが使用する各 Azure Stack Hub デプロイに登録されている必要があります。 テナントで複数の Azure Stack Hub を使用する場合は、各デプロイの初期登録をテナントのサブスクリプションで更新します。

### <a name="powershell"></a>PowerShell

テナントを追加するには、**New-AzureRmResource** コマンドレットを使用します。 [Azure に接続](/powershell/azure/get-started-azureps)し、管理者特権のプロンプトから次のコマンドを実行します。

```powershell  
New-AzureRmResource -ResourceId "subscriptions/{registrationSubscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.AzureStack/registrations/{registrationName}/customerSubscriptions/{customerSubscriptionId}" -ApiVersion 2017-06-01
```

### <a name="api-call"></a>API 呼び出し

**Operation**:PUT  
**要求 URI**: `subscriptions/{registrationSubscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.AzureStack/registrations/{registrationName}/customerSubscriptions/{customerSubscriptionId}?api-version=2017-06-01 HTTP/1.1`  
**応答**:201 Created  
**応答本文**:Empty  

## <a name="list-all-registered-tenants"></a>すべての登録済みテナントの一覧

登録に追加されているすべてのテナントの一覧を取得します。

 > [!NOTE]  
 > テナントが登録されていない場合は、応答を受信しません。

### <a name="parameters"></a>パラメーター

| パラメーター                  | 説明          |
|---                         | ---                  |
| registrationSubscriptionId | 初期登録に使用された Azure サブスクリプション。   |
| resourceGroup              | 登録が格納されている Azure 内のリソース グループ。    |
| registrationName           | お使いの Azure Stack Hub デプロイの登録名。 Azure に格納されているオブジェクトです。 通常、名前は **azurestack-CloudID** の形式で、**CloudID** はお使いの Azure Stack Hub デプロイのクラウド ID です。   |

### <a name="powershell"></a>PowerShell

登録されているすべてのテナントを一覧表示するには、**Get-AzureRmResource** コマンドレットを使用します。 [Azure Stack Hub に接続](azure-stack-powershell-configure-admin.md)してから、管理者特権のプロンプトから次のコマンドレットを実行します。

```powershell
Get-AzureRmResource -ResourceId "subscriptions/{registrationSubscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.AzureStack/registrations/{registrationName}/customerSubscriptions" -ApiVersion 2017-06-01
```

### <a name="api-call"></a>API 呼び出し

GET 操作を使用して、すべてのテナント マッピングの一覧を取得できます。

**Operation**:GET  
**要求 URI**: `subscriptions/{registrationSubscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.AzureStack/registrations/{registrationName}/customerSubscriptions?api-version=2017-06-01 HTTP/1.1`  
**応答**:200  
**応答本文**:

```json
{
    "value": [{
            "id": " subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.AzureStack/registrations/{registrationName}/customerSubscriptions/{ cspSubscriptionId 1}",
            "name": " cspSubscriptionId 1",
            "type": "Microsoft.AzureStack\customerSubscriptions",
            "properties": { "tenantId": "tId1" }
        },
        {
            "id": " subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.AzureStack/registrations/{registrationName}/customerSubscriptions/{ cspSubscriptionId 2}",
            "name": " cspSubscriptionId2 ",
            "type": "Microsoft.AzureStack\customerSubscriptions",
            "properties": { "tenantId": "tId2" }
        }
    ],
    "nextLink": "{originalRequestUrl}?$skipToken={opaqueString}"
}
```

## <a name="remove-a-tenant-mapping"></a>テナント マッピングの削除

登録に追加されているテナントを削除することができます。 そのテナントがまだ Azure Stack Hub のリソースを使用している場合、その使用は Azure Stack Hub の初期登録に使用されたサブスクリプションに課金されます。

### <a name="parameters"></a>パラメーター

| パラメーター                  | 説明          |
|---                         | ---                  |
| registrationSubscriptionId | 登録用のサブスクリプション ID です。   |
| resourceGroup              | 登録のリソース グループです。   |
| registrationName           | 登録の名前です。  |
| customerSubscriptionId     | 顧客サブスクリプション ID です。 顧客サブスクリプション ID は大文字と小文字の区別があります。  |

### <a name="powershell"></a>PowerShell

テナントを削除するには、**Remove-AzureRmResource** コマンドレットを使用します。 [Azure Stack Hub に接続](azure-stack-powershell-configure-admin.md)してから、管理者特権のプロンプトから次のコマンドレットを実行します。

```powershell
Remove-AzureRmResource -ResourceId "subscriptions/{registrationSubscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.AzureStack/registrations/{registrationName}/customerSubscriptions/{customerSubscriptionId}" -ApiVersion 2017-06-01
```

### <a name="api-call"></a>API 呼び出し

DELETE 操作を使用して、テナントのマッピングを削除することができます。

**Operation**:DELETE  
**要求 URI**: `subscriptions/{registrationSubscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.AzureStack/registrations/{registrationName}/customerSubscriptions/{customerSubscriptionId}?api-version=2017-06-01 HTTP/1.1`  
**応答**:204 No Content  
**応答本文**:Empty

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub からのリソースの使用量の取得方法](azure-stack-billing-and-chargeback.md)
