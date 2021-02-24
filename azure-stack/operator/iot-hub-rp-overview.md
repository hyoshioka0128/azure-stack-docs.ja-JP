---
title: Azure Stack Hub 上の IoT Hub の概要
description: Azure Stack Hub の IoT Hub リソース プロバイダーについて説明します。Azure でホストされるバージョンの IoT Hub との違いについても取り上げます。
author: yiyiguo
ms.author: yiygu
ms.service: azure-stack
ms.topic: how-to
ms.date: 1/6/2020
ms.openlocfilehash: adf6635fbdcf62398218a8e6ad0ce061900e0059
ms.sourcegitcommit: 0e3296fb27b9dabbc2569bf85656c4c7b1d58ba9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/30/2020
ms.locfileid: "93050336"
---
# <a name="iot-hub-on-azure-stack-hub-overview"></a>Azure Stack Hub 上の IoT Hub の概要

[!INCLUDE [preview-banner](../includes/iot-hub-preview.md)]

Azure Stack Hub 上の IoT Hub を使用すると、ハイブリッド IoT ソリューションを作成できます。 IoT Hub は、IoT アプリケーションとそれが管理するデバイスの間の双方向通信に対する中央メッセージ ハブとして機能する管理サービスです。 Azure Stack Hub 上の IoT Hub を使って IoT ソリューションを構築し、IoT デバイスとオンプレミス ソリューション バックエンドとの間に、信頼性が高く、セキュリティで保護された通信を提供できます。 

## <a name="differences-between-azure-iot-hub-and-iot-hub-on-azure-stack-hub"></a>Azure IoT Hub と Azure Stack Hub 上の IoT Hub との違い

| 機能 | Azure IoT Hub | Azure Stack Hub 上の IoT Hub |
|-|-|-|
| スケール | https://docs.microsoft.com/azure/iot-hub/iot-hub-scaling | S2 と S3 の IoT Hub がサポートされます|
| サービスの更新 | 自動 | マニュアル |

## <a name="iot-hub-throttling"></a>IoT Hub のスロットリング

Azure Stack Hub 上の IoT Hub (S2 SKU および S3 SKU) におけるスロットリングとクォータは、Azure 上の IoT Hub と同じです。 詳細については、[Azure IoT Hub のクォータと調整](/azure/iot-hub/iot-hub-devguide-quotas-throttling#quotas-and-throttling)に関するセクションを参照してください。

## <a name="next-steps"></a>次のステップ

Azure Stack Hub 上の IoT Hub をインストールするための準備については、[前提条件](iot-hub-rp-prerequisites.md)に関するドキュメントを参照してください。

接続環境の Azure Stack Hub と非接続環境の Azure Stack Hub の違いについて詳しくは、[Azure Stack 接続モデル](azure-stack-connection-models.md)に関するページを参照してください。
