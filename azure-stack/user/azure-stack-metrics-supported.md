---
title: Azure Stack Hub 上の Azure Monitor でサポートされるメトリック
description: Azure Stack Hub 上の Azure Monitor でサポートされるメトリックについて説明します。
author: mattbriggs
ms.topic: article
ms.date: 2/1/2021
ms.author: mabrigg
ms.lastreviewed: 11/11/2019
ms.openlocfilehash: 19a113e4d088bb3b239ec87ef1116892a0e4879b
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99247593"
---
# <a name="supported-metrics-for-azure-monitor-on-azure-stack-hub"></a>Azure Stack Hub 上の Azure Monitor でサポートされるメトリック

Azure Stack Hub 上の Azure Monitor のメトリックは、グローバル Azure と同じ方法で取得されます。 ポータルでメジャーを作成したり、REST API からそれらを取得したり、PowerShell または CLI を使用してそれらを照会したりできます。

次の表には、Azure Stack Hub 上の Azure Monitor のメトリック パイプラインで利用できるメトリックの一覧を示しています。 これらのメトリックに対してクエリを実行してアクセスするには、**2018-01-01** api-version バージョンの API プロファイルを使用します。 API プロファイルと Azure Stack Hub の詳細については、「[Azure Stack Hub での API バージョンのプロファイルの管理](azure-stack-version-profiles.md)」を参照してください。

## <a name="microsoftcomputevirtualmachines"></a>Microsoft.Compute/virtualMachines

| メトリック | メトリックの表示名 | ユニット | 集計の種類 | 説明 | Dimensions |
|----------------|---------------------|---------|------------------|-----------------------------------------------------------------------------------------------|---------------|
| Percentage CPU | Percentage CPU | Percent | Average | VM で現在使用されている、割り当てられたコンピューティング ユニットの割合。 | ディメンションなし |

## <a name="microsoftstoragestorageaccounts"></a>Microsoft.Storage/storageAccounts

| メトリック | メトリックの表示名 | ユニット | 集計の種類 | 説明 | Dimensions |
|----------------------|------------------------|--------------|------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------|
| UsedCapacity | Used capacity (使用済み容量) | バイト | Average | アカウントの使用済み容量。 | ディメンションなし |
| トランザクション | トランザクション | Count | 合計 | ストレージ サービスまたは指定された API 操作に対して行われた要求の数。 この数には、成功した要求と失敗した要求、およびエラーが発生した要求が含まれます。 別の種類の応答の数には ResponseType ディメンションを使います。 | ResponseType、GeoType、ApiName |
| イングレス | イングレス | バイト | 合計 | イングレス データの量 (バイト単位)。 この値には、外部クライアントから Azure Storage へのイングレスおよび Azure 内でのイングレスが含まれます。 | GeoType、ApiName |
| エグレス | エグレス | バイト | 合計 | エグレス データの量 (バイト単位)。 この値には、外部クライアントから Azure Storage へのエグレスおよび Azure 内でのエグレスが含まれます。 したがって、この値には、課金対象のエグレスが反映されません。 | GeoType、ApiName |
| SuccessServerLatency | Success Server Latency (成功サーバー待機時間) | ミリ秒 | Average | 成功した要求の処理に Azure Storage が使った平均待機時間 (ミリ秒単位)。 この値には、AverageE2ELatency で指定されているネットワーク待機時間は含まれません。 | GeoType、ApiName |
| SuccessE2ELatency | Success E2E Latency (成功 E2E 待機時間) | ミリ秒 | Average | ストレージ サービスまたは指定された API 操作に対して行われた成功した要求の平均エンド ツー エンド待機時間 (ミリ秒単位)。 この値には、要求の読み取り、応答の送信、および応答の受信確認を受け取るために Azure Storage 内で必要な処理時間が含まれます。 | GeoType、ApiName |
| 可用性 | 可用性 | Percent | Average | ストレージ サービスまたは指定された API 操作の可用性の割合。 TotalBillableRequests の値を取得し、それを該当する要求の数 (予期しないエラーが発生した要求を含む) で割ることによって、可用性を計算します。 予期しないエラーすべてが、ストレージ サービスまたは指定された API 操作の可用性の低下をもたらします。 | GeoType、ApiName |

## <a name="microsoftstoragestorageaccountsblobservices"></a>Microsoft.Storage/storageAccounts/blobServices

| メトリック | メトリックの表示名 | ユニット | 集計の種類 | 説明 | Dimensions |
|--------|---------------------|------|------------------|-------------|------------|
| BlobCapacity | Blob Capacity (BLOB 容量) | バイト | 合計 | ストレージ アカウントの Blob service によって使われているストレージの量 (バイト単位)。 | BlobType |
| BlobCount | Blob Count (BLOB 数) | Count | 合計 | ストレージ アカウントの Blob service 内の BLOB の数。 | BlobType |
| ContainerCount | Blob Container Count (BLOB コンテナー数) | Count | Average | ストレージ アカウントの Blob service 内のコンテナーの数。 | ディメンションなし |
| トランザクション | トランザクション | Count | 合計 | ストレージ サービスまたは指定された API 操作に対して行われた要求の数。 この数には、成功した要求と失敗した要求、およびエラーが発生した要求が含まれます。 別の種類の応答の数には ResponseType ディメンションを使います。 | ResponseType、GeoType、ApiName |
| イングレス | イングレス | バイト | 合計 | イングレス データの量 (バイト単位)。 この値には、外部クライアントから Azure Storage へのイングレスおよび Azure 内でのイングレスが含まれます。 | GeoType、ApiName |
| エグレス | エグレス | バイト | 合計 | エグレス データの量 (バイト単位)。 この値には、外部クライアントから Azure Storage へのエグレスおよび Azure 内でのエグレスが含まれます。 したがって、この値には、課金対象のエグレスが反映されません。 | GeoType、ApiName |
| SuccessServerLatency | Success Server Latency (成功サーバー待機時間) | ミリ秒 | Average | 成功した要求の処理に Azure Storage が使った平均待機時間 (ミリ秒単位)。 この値には、AverageE2ELatency で指定されているネットワーク待機時間は含まれません。 | GeoType、ApiName |
| SuccessE2ELatency | Success E2E Latency (成功 E2E 待機時間) | ミリ秒 | Average | ストレージ サービスまたは指定された API 操作に対して行われた成功した要求の平均エンド ツー エンド待機時間 (ミリ秒単位)。 この値には、要求の読み取り、応答の送信、および応答の受信確認を受け取るために Azure Storage 内で必要な処理時間が含まれます。 | GeoType、ApiName |
| 可用性 | 可用性 | Percent | Average | ストレージ サービスまたは指定された API 操作の可用性の割合。 TotalBillableRequests の値を取得し、それを該当する要求の数 (予期しないエラーが発生した要求を含む) で割ることによって、可用性を計算します。 予期しないエラーすべてが、ストレージ サービスまたは指定された API 操作の可用性の低下をもたらします。 | GeoType、ApiName |

## <a name="microsoftstoragestorageaccountstableservices"></a>Microsoft.Storage/storageAccounts/tableServices

| メトリック | メトリックの表示名 | ユニット | 集計の種類 | 説明 | Dimensions |
|----------------------|------------------------|--------------|------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------|
| TableCapacity | Table Capacity (テーブル容量) | バイト | Average | ストレージ アカウントの Table service によって使われているストレージの量 (バイト単位)。 | ディメンションなし |
| TableCount | Table Count (テーブル数) | Count | Average | ストレージ アカウントの Table service 内のテーブルの数。 | ディメンションなし |
| TableEntityCount | Table Entity Count (テーブル エンティティ数) | Count | Average | ストレージ アカウントの Table service 内のテーブル エンティティの数。 | ディメンションなし |
| トランザクション | トランザクション | Count | 合計 | ストレージ サービスまたは指定された API 操作に対して行われた要求の数。 この数には、成功した要求と失敗した要求、およびエラーが発生した要求が含まれます。 別の種類の応答の数には ResponseType ディメンションを使います。 | ResponseType、GeoType、ApiName |
| イングレス | イングレス | バイト | 合計 | イングレス データの量 (バイト単位)。 この値には、外部クライアントから Azure Storage へのイングレスおよび Azure 内でのイングレスが含まれます。 | GeoType、ApiName |
| エグレス | エグレス | バイト | 合計 | エグレス データの量 (バイト単位)。 この値には、外部クライアントから Azure Storage へのエグレスおよび Azure 内でのエグレスが含まれます。 したがって、この値には、課金対象のエグレスが反映されません。 | GeoType、ApiName |
| SuccessServerLatency | Success Server Latency (成功サーバー待機時間) | ミリ秒 | Average | 成功した要求の処理に Azure Storage が使った平均待機時間 (ミリ秒単位)。 この値には、AverageE2ELatency で指定されているネットワーク待機時間は含まれません。 | GeoType、ApiName |
| SuccessE2ELatency | Success E2E Latency (成功 E2E 待機時間) | ミリ秒 | Average | ストレージ サービスまたは指定された API 操作に対して行われた成功した要求の平均エンド ツー エンド待機時間 (ミリ秒単位)。 この値には、要求の読み取り、応答の送信、および応答の受信確認を受け取るために Azure Storage 内で必要な処理時間が含まれます。 | GeoType、ApiName |
| 可用性 | 可用性 | Percent | Average | ストレージ サービスまたは指定された API 操作の可用性の割合。 TotalBillableRequests の値を取得し、それを該当する要求の数 (予期しないエラーが発生した要求を含む) で割ることによって、可用性を計算します。 予期しないエラーすべてが、ストレージ サービスまたは指定された API 操作の可用性の低下をもたらします。 | GeoType、ApiName |

## <a name="microsoftstoragestorageaccountsqueueservices"></a>Microsoft.Storage/storageAccounts/queueServices

| メトリック | メトリックの表示名 | ユニット | 集計の種類 | 説明 | Dimensions |
|----------------------|------------------------|--------------|------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------|
| QueueCapacity | Queue Capacity (キュー容量) | バイト | Average | ストレージ アカウントの Queue サービスによって使われているストレージの量 (バイト単位)。 | ディメンションなし |
| QueueCount | Queue Count (キュー数) | Count | Average | ストレージ アカウントの Queue サービス内のキューの数。 | ディメンションなし |
| QueueMessageCount | Queue Message Count (キュー メッセージ数) | Count | Average | ストレージ アカウントの Queue サービス内のキュー メッセージの概数。 | ディメンションなし |
| トランザクション | トランザクション | Count | 合計 | ストレージ サービスまたは指定された API 操作に対して行われた要求の数。 この数には、成功した要求と失敗した要求、およびエラーが発生した要求が含まれます。 別の種類の応答の数には ResponseType ディメンションを使います。 | ResponseType、GeoType、ApiName |
| イングレス | イングレス | バイト | 合計 | イングレス データの量 (バイト単位)。 この値には、外部クライアントから Azure Storage へのイングレスおよび Azure 内でのイングレスが含まれます。 | GeoType、ApiName |
| エグレス | エグレス | バイト | 合計 | エグレス データの量 (バイト単位)。 この値には、外部クライアントから Azure Storage へのエグレスおよび Azure 内でのエグレスが含まれます。 したがって、この値には、課金対象のエグレスが反映されません。 | GeoType、ApiName |
| SuccessServerLatency | Success Server Latency (成功サーバー待機時間) | ミリ秒 | Average | 成功した要求の処理に Azure Storage が使った平均待機時間 (ミリ秒単位)。 この値には、AverageE2ELatency で指定されているネットワーク待機時間は含まれません。 | GeoType、ApiName |
| SuccessE2ELatency | Success E2E Latency (成功 E2E 待機時間) | ミリ秒 | Average | ストレージ サービスまたは指定された API 操作に対して行われた成功した要求の平均エンド ツー エンド待機時間 (ミリ秒単位)。 この値には、要求の読み取り、応答の送信、および応答の受信確認を受け取るために Azure Storage 内で必要な処理時間が含まれます。 | GeoType、ApiName |
| 可用性 | 可用性 | Percent | Average | ストレージ サービスまたは指定された API 操作の可用性の割合。 TotalBillableRequests の値を取得し、それを該当する要求の数 (予期しないエラーが発生した要求を含む) で割ることによって、可用性を計算します。 予期しないエラーすべてが、ストレージ サービスまたは指定された API 操作の可用性の低下をもたらします。 | GeoType、ApiName |

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub 上の Azure Monitor](azure-stack-metrics-azure-data.md) に関する詳細情報。
