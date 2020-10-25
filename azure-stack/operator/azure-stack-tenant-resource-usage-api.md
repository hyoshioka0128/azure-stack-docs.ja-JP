---
title: テナント リソース使用状況 API リファレンス
titleSuffix: Azure Stack
description: Azure Stack Hub の使用状況情報を取得する、リソース使用状況 API のリファレンス。
author: sethmanheim
ms.topic: article
ms.date: 08/25/2020
ms.author: sethm
ms.reviewer: alfredop
ms.lastreviewed: 01/14/2019
ms.openlocfilehash: 25bb94429624a1fde1ec8e1a963250837eebd0d9
ms.sourcegitcommit: d73637146daaba0ef0ab1729683bb52c65466099
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/25/2020
ms.locfileid: "88857552"
---
# <a name="tenant-resource-usage-api-reference"></a>テナント リソース使用状況 API リファレンス

テナントは、テナント API を使用して、テナントの独自のリソース使用状況データを表示できます。 これらの API は、Azure の使用状況の API と一致しています。

Windows PowerShell コマンドレット [Get-UsageAggregates](/powershell/module/azurerm.usageaggregates/get-usageaggregates) を使用して、Azure と同じように使用状況データを取得することができます。

## <a name="api-call"></a>API 呼び出し

### <a name="request"></a>Request

要求は、要求されたサブスクリプションと要求された期間の使用の詳細を取得します。 要求の本文はありません。

| **方法** | **要求 URI** |
| --- | --- |
| GET |https://{armendpoint}/subscriptions/{subId}/providers/Microsoft.Commerce/usageAggregates?reportedStartTime={reportedStartTime}&reportedEndTime={reportedEndTime}&aggregationGranularity={granularity}&api-version=2015-06-01-preview&continuationToken={token-value} |

### <a name="parameters"></a>パラメーター

| **パラメーター** | **説明** |
| --- | --- |
| Armendpoint |Azure Stack Hub 環境の Azure Resource Manager エンドポイント。 Azure Stack Hub の規則は、Azure Resource Manager エンドポイントの名前が `https://management.{domain-name}` の形式であることです。 たとえば、開発キットでは、ドメイン名が local.azurestack.external の場合、Resource Manager エンドポイントは `https://management.local.azurestack.external` です。 |
| subId |呼び出しを行っているユーザーのサブスクリプション ID。 この API は、1 つのサブスクリプションの使用状況をクエリするためにのみ使用できます。 プロバイダーは、プロバイダー リソース使用量 API を使用して、すべてのテナントの使用状況をクエリできます。 |
| reportedStartTime |クエリの開始時間。 *DateTime* の値は UTC で、13:00 など、毎時 0 分にする必要があります。 毎日の集計では、この値を UTC の午前 0 時に設定します。 形式はエスケープされた ISO 8601 です。たとえば、**2015-06-16T18%3a53%3a11%2b00%3a00Z** などで、URI に対応できるように、コロンは %3a にエスケープされ、プラスは %2b にエスケープされます。 |
| reportedEndTime |クエリの終了時間。 **reportedStartTime** に適用される制約は、このパラメーターにも適用されます。 **reportedEndTime** については、将来の値にすることはできません。 |
| aggregationGranularity |**daily** と **hourly** の 2 つの個別の指定可能な値を持つ省略可能なパラメーター。 値が示すように、一方は日単位の粒度でデータを返し、もう一方は時間単位の解像度です。 **daily** オプションが既定値です。 |
| api-version |この要求を行うために使用するプロトコルのバージョン。 **2015-06-01-preview** を使用する必要があります。 |
| continuationToken |使用状況 API プロバイダーへの最後の呼び出しから取得されたトークン。 このトークンは、応答が 1,000 行より大きい場合に必要であり、 進行状況のブックマークとして機能します。 これが存在しない場合、渡された単位に基づいて、日または時間の開始から、データが取得されます。 |

### <a name="response"></a>Response

```html
GET
/subscriptions/sub1/providers/Microsoft.Commerce/UsageAggregates?reportedStartTime=reportedStartTime=2014-05-01T00%3a00%3a00%2b00%3a00&reportedEndTime=2015-06-01T00%3a00%3a00%2b00%3a00&aggregationGranularity=Daily&api-version=1.0
```

```json
{
"value": [
{

"id":
"/subscriptions/sub1/providers/Microsoft.Commerce/UsageAggregate/sub1-meterID1",
"name": "sub1-meterID1",
"type": "Microsoft.Commerce/UsageAggregate",

"properties": {
"subscriptionId":"sub1",
"usageStartTime": "2015-03-03T00:00:00+00:00",
"usageEndTime": "2015-03-04T00:00:00+00:00",
"instanceData":"{\"Microsoft.Resources\":{\"resourceUri\":\"resourceUri1\",\"location\":\"Alaska\",\"tags\":null,\"additionalInfo\":null}}",
"quantity":2.4000000000,
"meterId":"meterID1"

}
},

...
```

### <a name="response-details"></a>応答の詳細

| **パラメーター** | **説明** |
| --- | --- |
| id |使用状況集計の一意の ID |
| name |使用状況集計の名前 |
| type |リソース定義 |
| subscriptionId |Azure ユーザーのサブスクリプション識別子。 |
| usageStartTime |この使用状況集計が属する使用状況バケットの UTC 開始時間 |
| usageEndTime |この使用状況集計が属する使用状況バケットの UTC 終了時間 |
| instanceData |インスタンスの詳細のキーと値のペア (新しい形式)。<br>  *resourceUri*:完全修飾リソース ID。リソース グループとインスタンス名が含まれます。 <br>  *location*:このサービスが実行されたリージョン。 <br>  *tags*:ユーザーが指定するリソース タグ。 <br>  *additionalInfo*:使用されたリソースに関する詳細。 たとえば、OS のバージョンやイメージの種類などです。 |
| 数量 |この期間に発生したリソース使用量 |
| meterId |使用されたリソースの一意の ID (**ResourceID** とも呼ばれます) |

## <a name="next-steps"></a>次のステップ

- [プロバイダー リソース使用量 API](azure-stack-provider-resource-api.md)
- [使用量に関するよくあるご質問 (FAQ)](azure-stack-usage-related-faq.md)
