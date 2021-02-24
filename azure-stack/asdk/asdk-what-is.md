---
title: Azure Stack 開発キット (ASDK) とは | Microsoft Docs
description: Azure Stack 開発キットと、それを使用して Azure Stack Hub を評価する方法について説明します。
author: justinha
ms.topic: overview
ms.date: 11/27/2019
ms.author: justinha
ms.reviewer: misainat
ms.lastreviewed: 11/27/2019
ms.openlocfilehash: 38aaf52936d09c999765118e36bfcde909409cdd
ms.sourcegitcommit: e9a1dfa871e525f1d6d2b355b4bbc9bae11720d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86489914"
---
# <a name="what-is-the-azure-stack-development-kit-asdk"></a>Azure Stack 開発キット (ASDK) とは
[Microsoft Azure Stack Hub 統合システム](../operator/azure-stack-overview.md)には 4 ノードから 16 ノードまでのサイズがあり、ハードウェア パートナーと Microsoft によって共同でサポートされます。 Azure Stack Hub 統合システムの使用により、運用ワークロード向けの新しいシナリオが可能になります。 統合システム インフラストラクチャの管理およびサービスの提供を行う Azure Stack Hub オペレーターは、[オペレーター ドキュメント](../operator/index.yml)を参照してください。

ASDK は、Azure Stack Hub の単一ノード デプロイであり、**無料**でダウンロードして使用できます。 すべての ASDK コンポーネントは、単一のホスト コンピューター上で実行される仮想マシン (VM) にインストールされます。そのコンピューターは、[最低限のハードウェア要件](asdk-deploy-considerations.md#hardware)を満たす必要があります。 ASDK の目的は、"*非運用*" 環境で Azure と一貫性のある API とツールを使用して、Azure Stack Hub の評価と最新アプリの開発ができる環境を提供することです。 

> [!IMPORTANT]
> ASDK の運用環境での使用は、想定されておらず、サポートされていません。

すべての ASDK コンポーネントが単一のホスト コンピューターにデプロイされるため、使用可能な物理リソースは限られています。 ASDK のデプロイでは、Azure Stack Hub インフラストラクチャ VM とテナント VM の両方が同じサーバー コンピューター上に共存します。 この構成は、拡張性やパフォーマンスの評価のためのものではありません。

ASDK は、以下のユーザーに Azure の一貫したハイブリッド クラウド エクスペリエンスを提供するように設計されています。
- **管理者** (Azure Stack Hub オペレーター):ASDK は、利用可能な Azure Stack Hub サービスの評価と学習に役立つ優れたリソースです。
- **開発者**:ASDK を使用すると、オンプレミス (開発/テスト環境) でハイブリッド アプリや最新型アプリを開発することができます。 この柔軟性により、Azure Stack Hub の運用環境のデプロイの前またはそれと同時に、開発環境を再現することができるようになります。

ASDK の詳細については、次の短い動画をご覧ください。

> [!VIDEO https://www.youtube.com/embed/dbVWDrl00MM]


## <a name="asdk-and-multi-node-azure-stack-hub-differences"></a>ASDK とマルチノード Azure Stack Hub の相違点
単一ノードの ASDK デプロイは、マルチノードの Azure Stack Hub デプロイとはいくつかの重要な点で異なります。

|説明|ASDK|マルチノード Azure Stack Hub|
|-----|-----|-----|
|**スケール**|すべてのコンポーネントが、単一ノードのサーバー コンピューターにインストールされます。|サイズは 4 から 16 ノードの範囲で設定できます。|
|**回復力**|単一ノードの構成では、高可用性は提供されません|高可用性機能がサポートされています。|
|**ネットワーク**|ASDK ホストは、すべての ASDK ネットワーク トラフィックをルーティングします。 他のスイッチ要件はありません。|Top-Of-Rack (TOR)、ベースボード管理コントローラー (BMC)、境界 (データセンター ネットワーク) スイッチなど、より複雑な[ネットワーク ルーティング インフラストラクチャ](../operator/azure-stack-network.md#network-infrastructure)がマルチノード デプロイでは必要です。|
|**修正プログラムと更新プログラムの処理**|ASDK の新しいバージョンに移行するには、ASDK のホスト コンピューターで ASDK を再デプロイする必要があります。|インストール済みの Azure Stack Hub バージョンを更新するために、[修正プログラムと更新プログラム](../operator/azure-stack-updates.md)のプロセスが使用されます。|
|**サポート**|MSDN の Azure Stack フォーラム。 Microsoft サポートは、非運用環境では使用 "*できません*"。|[MSDN の Azure Stack フォーラム](https://social.msdn.microsoft.com/Forums/en-US/home?forum=AzureStack)と完全なサポート。|
| | |

## <a name="learn-about-available-services"></a>利用できるサービスの詳細
Azure Stack Hub オペレーターは、ユーザーにどのサービスを提供できるかを知っておく必要があります。 Azure Stack Hub は Azure サービスのサブセットをサポートしており、サポートされるサービスの一覧は常に進化し続けています。

### <a name="foundational-services"></a>基礎となるサービス
ASDK をデプロイすると、Azure Stack Hub には既定で以下の "基礎となるサービス" が含まれています。
- Compute
- ストレージ
- ネットワーク
- Key Vault

これらの基礎となるサービスによって、最小限の構成でユーザーにサービスとしてのインフラストラクチャ (IaaS) を提供できます。

### <a name="additional-services"></a>その他のサービス
現時点では、その他に以下の、サービスとしてのプラットフォーム (PaaS) サービスがサポートされています。
- App Service
- Azure Functions
- SQL および MySQL データベース

> [!NOTE]
> これらのサービスをユーザーが利用できるようにするには、追加の構成が必要です。ASDK をインストールした時点で、既定では使用できません。

## <a name="service-roadmap"></a>サービスのロードマップ
Azure Stack Hub には、引き続き Azure サービスのサポートが追加される予定です。 Azure Stack Hub の今後の予定については、[Azure Stack ロードマップ](https://azure.microsoft.com/updates/?query=azure%20stack%20hub)を参照してください。


## <a name="next-steps"></a>次のステップ
Azure Stack Hub の評価を開始するには、最初に[最新の ASDK](asdk-download.md) をダウンロードし、ASDK ホスト コンピューターを準備する必要があります。 その後、ASDK をインストールして管理者ポータルとユーザー ポータルにサインインし、Azure Stack Hub の使用を開始することができます。
