---
title: ストレッチ クラスターの概要
description: ストレッチ クラスターについて説明します
author: v-dasis
ms.topic: how-to
ms.date: 07/21/2020
ms.author: v-dasis
ms.reviewer: JasonGerend
ms.openlocfilehash: ecf7974cec80888b149454c0a2ff90eb48c2e9e3
ms.sourcegitcommit: 0e52f460295255b799bac92b40122a22bf994e27
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/21/2020
ms.locfileid: "86868164"
---
# <a name="stretched-clusters-overview"></a>ストレッチ クラスターの概要

> 適用対象: Azure Stack HCI バージョン v20H2

ディザスター リカバリーのための Azure Stack HCI のストレッチ クラスター ソリューションでは、運用環境を迅速に復元するための自動フェールオーバーが提供され、手動で介入する必要はありません。 記憶域レプリカでは、ディザスター リカバリーのためにサイト間のボリュームのレプリケーションが提供され、すべてのサーバーの同期が維持されます。

記憶域レプリカでは、同期と非同期の両方のレプリケーションがサポートされます。

- 同期レプリケーションでは、クラッシュ整合ボリュームを使用して低待機時間のネットワークによりサイト間でデータがミラー化され、障害時のファイル システム レベルでのデータ損失をゼロにすることができます。
- 非同期レプリケーションは、大都市圏外のサイト間で待機時間の長いネットワーク リンクを通じてデータをミラーリングしますが、障害時に両方のサイトでデータが一致する保証はありません。

ストレッチ クラスターには、アクティブ - パッシブとアクティブ - アクティブの 2 種類があります。 優先されるサイトとレプリケーションの方向がある、アクティブ - パッシブのサイト レプリケーションを設定できます。 アクティブ - アクティブ レプリケーションでは、どちらのサイトからでも双方向のレプリケーションを実行できます。 この記事では、アクティブ - パッシブ構成のみについて説明します。

簡単に言うと、"*アクティブ*" サイトとは、リソースを保持し、クライアントが接続するための役割とワークロードを提供するもののことです。 "*パッシブ*" サイトとは、クライアントに対して役割やワークロードを提供しないものであり、ディザスター リカバリーのためにアクティブ サイトからのフェールオーバーを待機しています。

サイトは、2 つの異なる州、異なる都市、異なる階、または異なる部屋に配置できます。 2 つのサイトを使用するストレッチ クラスターでは、サイトで停止や障害が発生した場合に、ディザスター リカバリーとビジネス継続性が提供されます。

## <a name="active-passive-stretched-cluster"></a>アクティブ - パッシブ ストレッチ クラスター

次の図では、サイト 1 がアクティブ サイトであり、サイト 2 への 1 方向のレプリケーションが行われます。

:::image type="content" source="media/stretched-cluster/active-passive-stretched-cluster.png" alt-text="アクティブ - パッシブ ストレッチ クラスターのシナリオ"  lightbox="media/stretched-cluster/active-passive-stretched-cluster.png":::

## <a name="active-active-stretched-cluster"></a>アクティブ - アクティブ ストレッチ クラスター

次の図では、サイト 1 とサイト 2 の両方がアクティブ サイトであり、もう一方のサイトへの双方向レプリケーションが行われます。

:::image type="content" source="media/stretched-cluster/active-active-stretched-cluster.png" alt-text="アクティブ - アクティブ ストレッチ クラスターのシナリオ" lightbox="media/stretched-cluster/active-active-stretched-cluster.png":::

## <a name="next-steps"></a>次のステップ

- ストレッチ クラスターのハードウェアおよび他の要件について学習します。 [開始前](../deploy/before-you-start.md)に関する記事を参照してください。
- Windows Admin Center を使用してストレッチ クラスターをデプロイする方法について学習します。 [Windows Admin Center を使用したクラスターの作成](../deploy/create-cluster.md)に関する記事を参照してください。
- PowerShell を使用してストレッチ クラスターをデプロイする方法について学習します。 [PowerShell を使用したクラスターの作成](../deploy/create-cluster-powershell.md)に関する記事を参照してください。
- ストレッチ クラスター用のボリュームを作成し、レプリケーションを設定する方法を学習します。 [ストレッチ クラスター用のボリュームの作成とレプリケーションの設定](../manage/create-stretched-volumes.md)に関する記事を参照してください。
