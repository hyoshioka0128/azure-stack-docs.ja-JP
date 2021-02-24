---
title: Azure Stack Hub 上の IoT Hub の概要
description: Azure Stack Hub の IoT Hub リソース プロバイダーについて説明します。
author: yiyiguo
ms.author: yiygu
ms.service: azure-stack
ms.topic: how-to
ms.date: 12/12/2019
ms.openlocfilehash: 098168947a28214a3955a1961c80a2c06825126b
ms.sourcegitcommit: 296c95cad20ed62bdad0d27f1f5246bfc1c81d5e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/30/2020
ms.locfileid: "93064720"
---
# <a name="overview-of-iot-hub-on-azure-stack-hub"></a>Azure Stack Hub 上の IoT Hub の概要

[!INCLUDE [preview-banner](../includes/iot-hub-preview.md)]

Azure Stack Hub 上の IoT Hub を使用すると、ハイブリッド IoT ソリューションを作成できます。 IoT Hub は、IoT アプリケーションとそれが管理するデバイスの間の双方向通信に対する中央メッセージ ハブとして機能する管理サービスです。 Azure Stack Hub 上の IoT Hub を使って IoT ソリューションを構築し、IoT デバイスとオンプレミス ソリューション バックエンドとの間に、信頼性が高く、セキュリティで保護された通信を提供できます。

## <a name="features"></a>特徴

| 機能 | Azure 上の IoT Hub | Azure Stack Hub 上の IoT Hub プレビュー (S2 または S3) |
|-|-|-|
|device-to-cloud テレメトリ| ✔ | ✔ |
|クラウドからデバイスへのメッセージ| ✔ | ✔ |
|デバイスごとの ID| ✔ | ✔ |
|メッセージ ルーティング <sub>1</sub><sub>、4</sub>| ✔ | ✔ |
|HTTP、AMQP、MQTT プロトコル| ✔ | ✔ |
|マルチテナント| ✔ | ✔ |
|監視と診断| ✔ | ✔ |
|クラウドからデバイスへのメッセージ| ✔ | ✔ |
|デバイス管理、デバイス ツイン、モジュール ツイン| ✔ | ✔ |
|ツインの通知、デバイス ライフ サイクル イベント| ✔ | ✔ |
|Edge 多層デプロイ| ✔ | 近日公開 |
|管理者ポータル <sub>2</sub>| ✘ | ✔ |
|シークレットのローテーション <sub>2</sub>| ✘ | ✔ |
|容量管理 <sub>2</sub>| ✘ | ✔ |
|バックアップと復元 <sub>3</sub>| ✘ | ✘ |
|DeviceConnected、DeviceDisconnected、ASC <sub>4</sub>| ✔ | ✘ |
|デバイス モジュールの構成| ✔ | 近日公開 |
|デバイス ストリーミング、IoT プラグ アンド プレイ、ジョブ、ファイル アップロード <sub>5</sub>| ✔ | ✘ |
|Event Grid を使用したデバイスの接続状態の監視 <sub>4</sub>| ✔ | ✘ |
|フェールオーバー <sub>6</sub>| ✔ | ✘ |

<sub>1</sub> 組み込みのエンドポイント、Event Hubs、Storage に限られます。 Service Bus は Azure Stack Hub では利用できません。  
<sub>2</sub> ASH 上の IoT Hub を管理するオペレーター向け。  
<sub>3</sub> バックアップはプレビュー リリースで使用できます。 復元は GA でサポートされる予定です。  
<sub>4</sub> Azure Stack Hub では使用できない他のサービスに依存します。  
<sub>5</sub> 将来的には、Azure Stack Hub でも利用できるようになる予定です。  
<sub>6</sub> Azure Stack Hub では使用できません。  

## <a name="api-available-for-iot-hub-on-azure-stack-hub"></a>Azure Stack Hub 上の IoT Hub で使用できる API

|API|Azure Stack Hub 上の IoT Hub|
|-|-|
|Apply Configuration On Device| ✔ |
| Configuration Create | ✔ |
| Configuration Delete | ✔ |
| Configuration Read | ✔ |
|Configuration Read Many| ✔ |
|Configuration Service Apply|  ✔ |
|構成の更新|  ✔ |
|Device Direct Invoke Method|  ✔ |
|GetDeviceAndModuleInScope|  ✔ |
|GetDevicesAndModulesInScope| ✔ |
|Unregister Device| ✔ |
|Get Devices| ✔ |
|Update Module Twin| ✔ |
|D2C Get Twin| ✔ |
|デバイスをインポートします。| ✔ |
|Get Twin| ✔ |
|Unregister Module| ✔ |
|デバイスの更新| ✔ |
|Update Module| ✔ |
|Query Devices| ✔ |
|デバイスをエクスポートします。| ✔ |
|Back up and Restore – ADM| ✔ |
|Replace Twin| ✔ |
|Back up and Restore – DSS| ✔ |
|D2C Notify DesiredProperties| ✔ |
|D2C Patch ReportedProperties| ✔ |
|Get Module Twin| ✔ |
|Module D2C Get Twin| ✔ |
|モジュールの取得| ✔ |
|Module D2C Notify DesiredProperties| ✔ |
|Module D2C Patch ReportedProperties| ✔ |
|Module Direct Invoke Method| ✔ |
|Update Twin| ✔ |
|Bulk Device Operations| ✔ |
|Device to Cloud Telemetry| ✔ |
|Register Device| ✔ |
|Register Module| ✔ |
|Replace Module Twin| ✔ |
|GenericAuthentication| ✔ |
|Get Device| ✔ |
|Partition Move/Role Change| ✔ |

## <a name="differences-between-iot-hub-on-azure-cloud-and-iot-hub-on-azure-stack"></a>Azure クラウド上の IoT Hub と Azure Stack 上の IoT Hub との違い

| 側面 | クラウド上の IoT Hub | Stack 上の IoT Hub |
|-|-|-|
| メッセージの利用 | https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-messages-read-builtin |既定では、メッセージは Event Hubs と互換性のある、サービスに接続された組み込みエンドポイント (messages/events) にルーティングされます。 Azure クラウドでは、IoT Hub の接続文字列または Event Hub の接続文字列を指定することで、エンド ポイントからメッセージにアクセスできます。 一方、Azure Stack Hub でサポートされるのは、イベント ハブの接続文字列のみとなります。 |

## <a name="next-steps"></a>次の手順

ご利用のサブスクリプションで IoT Hub を利用できない場合は、管理者と協力して [Azure Stack Hub リソース プロバイダーに IoT Hub をインストール](../operator/iot-hub-rp-overview.md)してください。

IoT Hub の使用方法については、[Azure IoT Hub のドキュメント](/azure/iot-hub/)を参照してください。

