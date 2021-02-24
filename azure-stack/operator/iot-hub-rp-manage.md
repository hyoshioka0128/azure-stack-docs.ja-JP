---
title: Azure Stack Hub で IoT Hub を管理する方法
description: IoT Hub の管理エクスペリエンスとアラート
author: yiyiguo
ms.author: yiygu
ms.service: azure-stack
ms.topic: how-to
ms.date: 1/6/2020
ms.openlocfilehash: 3fe1459d28c399269ccdb1d4cffed024bd741000
ms.sourcegitcommit: 9b0e1264ef006d2009bb549f21010c672c49b9de
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98255743"
---
# <a name="how-to-manage-iot-hub-on-azure-stack-hub"></a>Azure Stack Hub で IoT Hub を管理する方法

IoT Hub の管理エクスペリエンスを使用すると、全体的な状態、アラート、容量を視覚化して管理することができます。

[!INCLUDE [preview-banner](../includes/iot-hub-preview.md)]

## <a name="iot-hub-management-dashboard"></a>IoT Hub の管理ダッシュボード

IoT Hub の管理ダッシュボードにアクセスするには:

1. Azure Stack Hub の管理者ポータルにサインインし、左側の **[ダッシュボード]** ビューを選択して、 **[IoT Hub]** リソース プロバイダーを選択します。

   [![オペレーター ダッシュボード](media\iot-hub-rp-manage\dashboard.png)](media\iot-hub-rp-manage-capacity\dashboard.png#lightbox)

2. IoT Hub ダッシュボードには概要ビューが用意されており、最新のアラート、スタンプに作成されたクォータ、サブスクリプションに含まれる IoT Hub クラスターの総数が表示されます。 

   [![IoT Hub ダッシュボード - 概要](media\iot-hub-rp-manage\dashboard-rp-iot-hub-overview.png)](media\iot-hub-rp-manage-capacity\dashboard-rp-iot-hub-overview.png#lightbox)

## <a name="alerts"></a>警告

IoT Hub リソース プロバイダーでは、次のアラートがサポートされています。

|カテゴリ|アラート:|Type|Description|
|-|-|-|-|
|パフォーマンス|IoT Hub CPU usage needs attention. (IoT Hub の CPU 使用率に注意が必要です。)|警告|過去 6 時間における IoT Hub リソース プロバイダーの CPU 使用率の平均が 75% を超えています。|
|パフォーマンス|IoT Hub memory usage needs attention. (IoT Hub のメモリ使用量に注意が必要です。)|警告|過去 6 時間における IoT Hub リソース プロバイダーの残りメモリ使用量が 1,024 MB を下回っています。|
|パフォーマンス|Low disk space for IoT Hub resource provider. (IoT Hub リソース プロバイダーのディスク領域が不足しています。)|警告|空きディスク領域が 10% を下回っています。|
|リソース|Creating or updating IoT Hub resource failed. (IoT Hub リソースの作成または更新に失敗しました。)|警告|IoT Hub リソース プロバイダーでの IoT Hub の作成または更新の失敗数が 15 分間で 1 回以上になっています。|
|サービス|IoT Hub resource provider log errors needs attention. (IoT Hub リソース プロバイダーのログ エラーに注意が必要です。)|警告|IoT Hub リソース プロバイダーのロール インスタンスあたりのログ エラー数が 15 分間で 3 個を超えています。|
|サービス|IoTHub-SU-InternalError|警告|IoTHub SU の内部エラーおよびタイムアウトの数が直近 30 分間で 5 回以上になっています。|
|サービス|IoT Hub data plane backup failure occurred. (IoT Hub データ プレーンのバックアップ エラーが発生しました。)|警告|直近 15 分間に、IoT Hub データ プレーンでバックアップ エラーが発生しました。 デバイス データは保護されません。|
|サービス|IoT Hub data plane consecutive backup failure has occurred. (IoT Hub データ プレーンの連続バックアップ エラーが発生しました。)|警告|過去 15 分間に、IoT Hub データ プレーンで連続バックアップ エラーが発生しました。 デバイス データは保護されません。|
|サービス|IoT Hub data plane no success backup failure occurred. (IoT Hub データ プレーンのバックアップ失敗のエラーが発生しました。)|警告|直近 3 時間に成功したバックアップが IoT Hub に存在しません。 デバイス データが保護されていない可能性があります。|
|サービス|IoT Hub data plane restore failed. (IoT Hub データ プレーンの復元に失敗しました。)|警告|IoT Hub デバイス情報の復元に失敗しました。 過去 15 分間、IotHubPerformanceMetrics がしきい値を超えています。|
|サービス|IoT Hub gateway failure occurred. (IoT Hub ゲートウェイ エラーが発生しました。)|警告|IoT Hub ゲートウェイ エラーが発生しました。 デバイス テレメトリ機能に影響が生じる可能性があります。|
|サービス|IoT Hub Container failure occurred. (IoT Hub コンテナー エラーが発生しました。)|警告|IoT Hub コンテナー エラーが発生しました。 デバイスの認証に失敗する可能性があります。 |
|サービス|IoT Hub device management container failure occurred. (IoT Hub デバイス管理コンテナーのエラーが発生しました。)|警告|IoT Hub デバイス管理コンテナーのエラーが発生しました。 デバイス ツインのダイレクト メソッド機能が低下している可能性があります。|

アラートおよびクォータを監視するには:

1. **[アラート]** を選択してアラートの履歴を表示します。 
 
   [![IoT Hub ダッシュボード - アラート](media\iot-hub-rp-manage\dashboard-rp-iot-hub-alerts.png)](media\iot-hub-rp-manage-capacity\dashboard-rp-iot-hub-alerts.png#lightbox)  

2. **[クォータ]** を選択して、有効なクォータの一覧を表示します。  

   > [!NOTE]
   > プレビュー期間中は、 **[作成]** 機能が無効になっているため、制限のない既定のクォータが提供されます。 **[作成]** は、GA で有効になる予定です。

   [![IoT Hub ダッシュボード - クォータ](media\iot-hub-rp-manage\dashboard-rp-iot-hub-quotas.png)](media\iot-hub-rp-manage-capacity\dashboard-rp-iot-hub-quotas.png#lightbox) 

## <a name="capacity-management"></a>容量管理

オペレーターは、任意のクラウド オペレーターと同じ方法で Azure Stack Hub インスタンスの管理と運用を行います。 ソフトウェアが正しくインストールされ、適切かつ安全に構成されていることと、また、運用の高可用性 (HA)、一貫性、効率性が確保されていることを確認します。 Azure Stack Hub インスタンスも IoT Hub リソース プロバイダーも、その運用には、容量管理の原則を適用する必要があります。

### <a name="azure-stack-hub-capacity-management"></a>Azure Stack Hub の容量管理

IoT Hub に必要な容量を調べるには、ワークロード (主にデバイスの数とメッセージのスループット) を見積もる必要があります。 計画を支援するために、参考までに 4 ノードの Azure Stack Hub 環境で次のテストを実施しました。

| シナリオ | VM | 仮想コア数 | ハブあたりのデバイス数 | [Hub のエディション](https://azure.microsoft.com/pricing/details/iot-hub) | ハブ | ハブあたりのユニット数 | 合計デバイス | 合計ハブ ユニット数 | 1 日あたりのメッセージ数 (単位: 100 万件) |
|----------|---------------|------------------|-----------------------|-------------------|-|-|-|-|-|
|Default|5|20|300,000|S2|4|200|1,200,000|800|4,800|
|12 VM|12|48|500,000|S2|4|200|2,000,000|800|4,800|
|18 VM|18|72|400,000|S3|4|10|1,600,000|40|12,000|

詳細については、「[Azure Stack Hub Capacity Planner](azure-stack-capacity-planner.md)」を参照してください。

### <a name="iot-hub-capacity-management"></a>IoT Hub の容量管理

Azure Stack Hub は、リソースに限りのあるオンプレミス データ センターにデプロイされるため、Azure Stack Hub で実行されるすべてのサービスでは、同じリソース プールを共有し、それらを奪い合います。 その容量は、オペレーターがビジネス ニーズに基づいて計画、管理する必要があります。 オペレーターは、IoT Hub リソース プロバイダーを使用すると、サービスの容量要件を管理することができます。

IoT Hub に含まれる VM の種類は 1 つだけです。 IoT Hub のデプロイの一環として、それらの一連の VM が Azure Stack Hub にプロビジョニングされます。 これらの VM では、一定数のデバイスとメッセージ スループットがサポートされます。 ほとんどの要件は、既定の設定で満たすことができるはずです。 ただし、デバイス数またはメッセージ スループットを増やす必要がある場合は、管理者ポータル、CLI、または PowerShell を使用して容量を増やすことができます。 

容量設定を監視したり変更したりするには:

1. 左側の **[Capacity]\(容量\)** ビューを選択します。 IoT Hub クラスターにプロビジョニングされている VM の数とそのリソース使用率など、容量の状態が表示されます。 表示されるノード数は、IoT Hub に現在割り当てられているノード数です。 

2. 容量を増やすには、IoT Hub のクラスター名を選択し、ノード数を変更して、 **[Update Scale]\(スケールの更新\)** を選択します。 使用可能なリソースで許される限りいくつでも VM を追加して容量を増やすことができます。

   > [!IMPORTANT]
   > プレビューでは、IoT Hub クラスターのスケールアウト (小規模から大規模) のみサポートされます。 スケールイン (大規模から小規模) は、一般提供 (GA) バージョンの IoT Hub でサポートされる予定です。

   [![IoT Hub ダッシュボード - 容量](media\iot-hub-rp-manage\dashboard-rp-iot-hub-capacity.png)](media\iot-hub-rp-manage-capacity\dashboard-rp-iot-hub-capacity.png#lightbox)


## <a name="next-steps"></a>次のステップ

詳細情報:

アラートなど、Azure Stack Hub の監視機能については、[正常性とアラートの監視](azure-stack-monitor-health.md)に関するページを参照してください。

Azure Stack Hub のログの収集については、[Azure Stack 診断ログの収集の概要](./diagnostic-log-collection.md)に関するページを参照してください。