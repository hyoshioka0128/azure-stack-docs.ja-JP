---
title: Azure Stack Hub 上の Event Hubs の概要
description: Azure Stack Hub の Event Hubs リソースプロバイダーについて説明します。
author: BryanLa
ms.author: bryanla
ms.service: azure-stack
ms.topic: how-to
ms.date: 12/09/2019
ms.reviewer: jfggdl
ms.lastreviewed: 12/09/2019
ms.openlocfilehash: 2c7574ef54176e4e4b91f5bee8af9e6953d13732
ms.sourcegitcommit: 4af79f4fa2598d57c81e994192c10f8c6be5a445
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/10/2020
ms.locfileid: "89742641"
---
# <a name="event-hubs-on-azure-stack-hub-overview"></a>Azure Stack Hub 上の Event Hubs の概要

[!INCLUDE [preview-banner](../includes/event-hubs-preview.md)]

Azure Stack Hub 上の Event Hubs を使用すると、ハイブリッド クラウド シナリオを実現できます。 オンプレミスと Azure クラウドの両方の処理で、ストリーミングやイベントベースのソリューションがサポートされます。 ハイブリッド (接続型) シナリオであれ非接続型のシナリオであれ、大規模なイベント処理またはストリーム処理に対応したソリューションを実現できます。 シナリオはクラスター サイズによってのみ制限され、その点は、必要に応じてプロビジョニングできます。 

## <a name="features"></a>特徴

次の機能比較については、[Azure Stack Hub のユーザー ドキュメント](../user/event-hubs-overview.md)を参照してください: Azure Stack 上の Event Hubs と Azure Event Hubs。

## <a name="feature-documentation"></a>機能のドキュメント

Event Hubs のユーザー エクスペリエンスについて詳しくは、[Azure Event Hubs のドキュメント](/azure/event-hubs/)を参照してください。 このドキュメントは、両方のエディションの Event Hubs を対象としており、次のようなトピックを取り上げています。

- [Event Hubs の概念](/azure/event-hubs/event-hubs-features)に関する詳細情報
- [Event Hubs クラスターと名前空間を作成](/azure/event-hubs/event-hubs-dedicated-cluster-create-portal)する方法
- [イベント ハブ](/azure/event-hubs/event-hubs-create#create-an-event-hub)の作成方法
- [Kafka プロトコルを使用](/azure/event-hubs/event-hubs-quickstart-kafka-enabled-event-hubs)してストリーム配信する方法


## <a name="next-steps"></a>次のステップ

インストール プロセスを開始する前に、[Azure Stack Hub で Event Hubs のキャパシティ プランニングを行う方法](event-hubs-rp-capacity-planning.md)に関するページを参照してください。 キャパシティ プランニングについての知識は、ユーザーに必要な容量を確実に提供するうえで役立ちます。
