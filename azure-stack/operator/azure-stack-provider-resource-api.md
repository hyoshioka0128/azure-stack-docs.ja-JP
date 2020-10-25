---
title: Azure Stack Hub のプロバイダー リソース使用量 API
description: Azure Stack Hub の使用状況情報を取得するリソース使用量 API のリファレンス。
author: sethmanheim
ms.topic: article
ms.date: 07/27/2020
ms.author: sethm
ms.reviewer: alfredop
ms.lastreviewed: 01/25/2019
ms.openlocfilehash: 0e167bdfeb0685b8b4612e887b904d010ffd666b
ms.sourcegitcommit: b2337a9309c52aac9f5a1ffd89f1426d6c178ad5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/28/2020
ms.locfileid: "87250777"
---
# <a name="provider-resource-usage-api"></a>プロバイダー リソース使用量 API

*プロバイダー*という用語は、サービス管理者と委任されたすべてのプロバイダーに適用されます。 Azure Stack Hub オペレーターおよび委任されたプロバイダーは、プロバイダーの使用量 API を使用して、直接的に関係するテナントの使用状況を表示できます。 たとえば、次の図に示すように、P0 はプロバイダー API を呼び出して、P1 と P2 の直接の使用状況情報を取得でき、P1 は P3 と P4 の使用状況情報を呼び出すことができます。

![プロバイダー階層の概念モデル](media/azure-stack-provider-resource-api/image1.png)

## <a name="api-call-reference"></a>API 呼び出しリファレンス

### <a name="request"></a>Request

要求は、要求されたサブスクリプションと要求された期間の使用の詳細を取得します。 要求の本文はありません。

この使用状況 API はプロバイダー API であるため、呼び出し元に、プロバイダーのサブスクリプションの**所有者**、**共同作成者**、または**閲覧者**の役割が割り当てられている必要があります。

| Method | 要求 URI |
| --- | --- |
| GET |`https://{armendpoint}/subscriptions/{subId}/providers/Microsoft.Commerce.Admin/subscriberUsageAggregates?reportedStartTime={reportedStartTime}&reportedEndTime={reportedEndTime}&aggregationGranularity={granularity}&subscriberId={sub1.1}&api-version=2015-06-01-preview&continuationToken={token-value}` |

### <a name="arguments"></a>引数

| 引数 | 説明 |
| --- | --- |
| `armendpoint` |Azure Stack Hub 環境の Azure Resource Manager エンドポイント。 Azure Stack Hub の名前付け規則は、Azure Resource Manager エンドポイントの名前が `https://adminmanagement.{domain-name}` の形式であることです。 たとえば、Azure Stack Development Kit (ASDK) では、ドメイン名が *local.azurestack.external* の場合、Resource Manager エンドポイントは `https://adminmanagement.local.azurestack.external` になります。 |
| `subId` |呼び出しを行っているユーザーのサブスクリプション ID。 |
| `reportedStartTime` |クエリの開始時間。 `DateTime` の値は協定世界時 (UTC) で、13:00 など、毎時 0 分に設定する必要があります。 毎日の集計では、この値を UTC の午前 0 時に設定します。 形式はエスケープされた ISO 8601 (たとえば、`2015-06-16T18%3a53%3a11%2b00%3a00Z` など) です。URI に対応できるように、コロンは `%3a` に、プラスは `%2b` にエスケープされます。 |
| `reportedEndTime` |クエリの終了時間。 `reportedStartTime` に適用される制約は、この引数にも適用されます。 `reportedEndTime` の値は、将来、または現在の日付にすることはできません。 そうすると、結果は "処理が未完了" に設定されます。 |
| `aggregationGranularity` |**daily** と **hourly** の 2 つの個別の指定可能な値を持つ省略可能なパラメーター。 値が示すように、一方は日単位の粒度でデータを返し、もう一方は時間単位の解像度です。 **daily** オプションが既定値です。 |
| `subscriberId` |[サブスクリプション ID] が表示されます。 フィルター処理されたデータを取得するには、プロバイダーの直接のテナントのサブスクリプション ID が必要です。 サブスクリプション ID パラメーターが指定されていない場合、呼び出しは、すべてのプロバイダーの直接のテナントの使用状況データを返します。 |
| `api-version` |この要求を行うために使用するプロトコルのバージョン。 この値は `2015-06-01-preview` に設定されます。 |
| `continuationToken` |使用状況 API プロバイダーへの最後の呼び出しから取得されたトークン。 このトークンは、応答が 1,000 行より大きい場合に必要であり、 進行状況のブックマークとして機能します。 トークンが存在しない場合は、渡された細分性に基づいて、日または時間の開始点からのデータが取得されます。 |

### <a name="response"></a>Response

```http
GET
/subscriptions/sub1/providers/Microsoft.Commerce.Admin/subscriberUsageAggregates?reportedStartTime=reportedStartTime=2014-05-01T00%3a00%3a00%2b00%3a00&reportedEndTime=2015-06-01T00%3a00%3a00%2b00%3a00&aggregationGranularity=Daily&subscriberId=sub1.1&api-version=1.0
```

```json
{
"value": [
{

"id":
"/subscriptions/sub1.1/providers/Microsoft.Commerce.Admin/UsageAggregate/sub1.1-

meterID1",
"name": "sub1.1-meterID1",
"type": "Microsoft.Commerce.Admin/UsageAggregate",

"properties": {
"subscriptionId":"sub1.1",
"usageStartTime": "2015-03-03T00:00:00+00:00",
"usageEndTime": "2015-03-04T00:00:00+00:00",
"instanceData":"{\"Microsoft.Resources\":{\"resourceUri\":\"resourceUri1\",\"location\":\"Alaska\",\"tags\":null,\"additionalInfo\":null}}",
"quantity":2.4000000000,
"meterId":"meterID1"

}
},

. . .
```

### <a name="response-details"></a>応答の詳細

| 引数 | 説明 |
| --- | --- |
|`id` |使用状況集計の一意の ID |
|`name` |使用状況集計の名前 |
|`type` |リソース定義 |
|`subscriptionId` |Azure Stack Hub ユーザーのサブスクリプション識別子。 |
|`usageStartTime`|この使用状況集計が属する使用状況バケットの UTC 開始時間|
|`usageEndTime`|この使用状況集計が属する使用状況バケットの UTC 終了時間 |
|`instanceData` |インスタンスの詳細のキーと値のペア (新しい形式)。<br> `resourceUri`:完全修飾リソース ID。リソース グループとインスタンス名が含まれます。 <br> `location`:このサービスが実行されたリージョン。 <br> `tags`:ユーザーによって指定されたリソース タグ。 <br> `additionalInfo`:OS のバージョンやイメージの種類など、使用されたリソースの詳細。 |
|`quantity`|この期間に発生したリソース使用量 |
|`meterId` |使用されたリソースの一意の ID (`ResourceID` とも呼ばれます)。 |

## <a name="retrieve-usage-information"></a>使用量情報を取得する

### <a name="powershell"></a>PowerShell

使用量データを生成するには、実行されていて、システムをアクティブに使っているリソース (アクティブな仮想マシン (VM) や、データを格納しているストレージ アカウントなど) が必要です。 Azure Stack Hub Marketplace で実行されているリソースがあるかどうかが不明な場合は、VM をデプロイし、VM 監視ブレードを調べて、リソースが実行されていることを確認します。 使用量データを表示するには、次の PowerShell コマンドレットを使います。

1. [PowerShell for Azure Stack Hub をインストールします](azure-stack-powershell-install.md)。
2. [Azure Stack Hub ユーザー](../user/azure-stack-powershell-configure-user.md)または [Azure Stack Hub オペレーター](azure-stack-powershell-configure-admin.md)の PowerShell 環境を構成します。
3. 使用状況データを取得するには、[Get-AzsSubscriberUsage](/powershell/module/azs.commerce.admin/get-azssubscriberusage) PowerShell コマンドレットを呼び出します。

   ```powershell
   Get-AzsSubscriberUsage -ReportedStartTime "2017-09-06T00:00:00Z" -ReportedEndTime "2017-09-07T00:00:00Z"
   ```

### <a name="rest-api"></a>REST API

**Microsoft.Commerce.Admin** サービスを呼び出すことで、削除されたサブスクリプションの利用状況情報を収集できます。

#### <a name="return-all-tenant-usage-for-deleted-for-active-users"></a>アクティブ ユーザーに対して削除されたすべてのテナントの使用状況を返す

| Method | 要求 URI |
| --- | --- |
| GET | `https://{armendpoint}/subscriptions/{subId}/providersMicrosoft.Commerce.Admin/subscriberUsageAggregates?reportedStartTime={start-time}&reportedEndTime={end-endtime}&aggregationGranularity=Hourly&api-version=2015-06-01-preview` |

#### <a name="return-usage-for-deleted-or-active-tenant"></a>削除済みまたはアクティブなテナントの使用状況を返す

| Method | 要求 URI |
| --- | --- |
| GET |`https://{armendpoint}/subscriptions/{subId}/providersMicrosoft.Commerce.Admin/subscriberUsageAggregates?reportedStartTime={start-time}&reportedEndTime={end-endtime}&aggregationGranularity=Hourly&subscriberId={subscriber-id}&api-version=2015-06-01-preview` |

## <a name="next-steps"></a>次のステップ

- [テナント リソース使用量 API リファレンス](azure-stack-tenant-resource-usage-api.md)
- [使用量に関するよくあるご質問 (FAQ)](azure-stack-usage-related-faq.md)
