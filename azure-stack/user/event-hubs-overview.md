---
title: Azure Stack Hub 上の Event Hubs の概要
description: Azure Stack Hub 上の Event Hubs について説明します。 ハイブリッド ソリューションの構築方法を確認します。 Azure Stack Hub 上の Azure Event Hubs と Event Hubs の機能を比較します。
author: BryanLa
ms.author: bryanla
ms.service: azure-stack
ms.topic: how-to
ms.date: 12/11/2020
ms.reviewer: jfggdl
ms.lastreviewed: 12/11/2020
ms.openlocfilehash: c7b54121f826fb514590f39c92cde0e62972ce97
ms.sourcegitcommit: 79e8df69b139bfa21eb83aceb824b97e7f418c03
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/12/2020
ms.locfileid: "97364151"
---
# <a name="overview-of-event-hubs-on-azure-stack-hub"></a>Azure Stack Hub 上の Event Hubs の概要

Azure Stack Hub 上の Event Hubs を使用すると、ハイブリッド クラウド シナリオを実現できます。 オンプレミスと Azure クラウドの両方の処理で、ストリーミングやイベントベースのソリューションがサポートされます。 ハイブリッド (接続型) シナリオであれ非接続型のシナリオであれ、大規模なイベント処理またはストリーム処理に対応したソリューションを実現できます。 シナリオは Event Hubs クラスターによってのみ制限され、その点は、必要に応じてプロビジョニングすることができます。 

## <a name="run-event-processing-tasks-and-build-event-driven-applications-on-site"></a>イベント処理タスクを実行し、サイトでイベントドリブン アプリケーションを構築する

Azure Stack Hub で Event Hubs を使用すると、次のようなビジネス シナリオを実装できます。

- Event Hubs がイベント ストリーミング エンジンである AI と機械学習のワークロード。
- Azure データセンターの外部にある独自のサイトでイベントドリブン アーキテクチャを実装する。
- オンプレミスにデプロイされた Web アプリケーションのクリックストリーム分析。
- デバイス テレメトリ分析。
- Apache Spark、Flink、Storm、Samza など、Apache Kafka を使用するオープンソース フレームワークを使ったストリーム処理。
- [コンピューティング ゲスト OS のメトリックとイベントを使用する](azure-stack-metrics-monitor.md)。

## <a name="build-hybrid-solutions"></a>ハイブリッド ソリューションを構築する

Azure Stack Hub にエッジ データをローカルに取り込んで処理するハイブリッド ソリューションを構築します。 さらなる処理、視覚化、および格納を行うために、集計されたデータを Azure に送信します。 必要に応じて、Azure でサーバーレス コンピューティングを活用します。

[![ハイブリッド ソリューションの図](media/event-hubs-overview/hybrid-architecture-ehoash.png)](media/event-hubs-overview/hybrid-architecture-ehoash.png#lightbox)

## <a name="features"></a>特徴 

(Azure Stack Hub 上および Azure 上の) Event Hubs エディションには、高度な機能パリティが実現されています。 ここでいうパリティとは、SDK、サンプル、PowerShell、CLI、ポータルからいずれも同様のエクスペリエンスが得られ、違いがほとんどないということです。 次の表は、各エディションで利用できる機能の相違を簡単にまとめたものです。  

| 機能 | Azure Stack Hub 上の Event Hubs | Azure Event Hubs |
|-|-|-|-|
| オペレーター管理者エクスペリエンス | ✔ | ✘ |
| Kafka サポート | ✔ | ✔ |
| 同じ SDK のセット | ✔ | ✔ |
| Azure Active Directory を使用した Event Hubs へのアクセス承認 | ✘ | ✔ |
| キャプチャ機能 | ✘ | ✔ |
| geo ディザスター リカバリー | ✘ | ✔ |
| Azure Monitor | ✔ | ✔ |
| 自動インフレ機能 | ✘ | ✔ |

Azure Resource Management の操作は、Azure Resource Manager テンプレート、[PowerShell](/powershell/module/Az.eventhub/)、[Azure CLI](/cli/azure/eventhubs/eventhub/) を使用して行うこともできます。 PowerShell および Azure CLI では現在、Operator Administration の操作はサポートされません。

## <a name="feature-documentation"></a>機能のドキュメント

[Azure Event Hubs のドキュメント](/azure/event-hubs/)は、両方の Event Hubs エディションを対象としています。Azure Stack Hub 上の Event Hubs、および Azure Event Hubs。 このドキュメントでは、Event Hubs とアクティビティの使用に関する次のようなトピックを扱っています。

- [Event Hubs の概念](/azure/event-hubs/event-hubs-features)に関する詳細情報
- [Event Hubs クラスターと名前空間を作成](event-hubs-quickstart-cluster-portal.md)する方法
- [イベント ハブ](/azure/event-hubs/event-hubs-create#create-an-event-hub)の作成方法
- [Kafka プロトコルを使用](/azure/event-hubs/event-hubs-quickstart-kafka-enabled-event-hubs)してストリーム配信する方法

### <a name="operator-documentation"></a>オペレーター ドキュメント 
 
Azure Stack Hub 上の Event Hubs のオペレーター エクスペリエンスについて詳しくは、[Event Hubs のオペレーター ドキュメント](../operator/event-hubs-rp-overview.md)を参照してください。 このドキュメントでは、次のようなアクティビティについての情報を取り上げています。

- Event Hubs をインストールする。
- Event Hubs をユーザーが利用できるようにする。
- サービスの正常性に関する情報を取得する。
- ログを収集する。


## <a name="next-steps"></a>次のステップ

ご利用のサブスクリプションで Event Hubs を利用できない場合は、管理者と協力して [Azure Stack Hub リソース プロバイダーに Event Hubs をインストール](../operator/event-hubs-rp-overview.md)してください。

Event Hubs をインストール済みで、使用する準備が整っている場合は、[Event Hubs のドキュメント](/azure/event-hubs/event-hubs-about)を参照して、サービスの詳細を確認してください。
