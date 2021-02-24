---
title: Azure Stack Hub のためのキャパシティ プランニングの概要 | Microsoft Docs
description: Azure Stack Hub デプロイのためのキャパシティ プランニングについて説明します。 Azure Stack Hub ラグドの高モデルと低モデルの仕様を参照してください。
services: azure-stack
documentationcenter: ''
author: PatAltimore
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/14/2020
ms.author: patricka
ms.reviewer: prchint
ms.lastreviewed: 10/14/2020
ms.openlocfilehash: 1231b69574466cadcebff56f23cbb85d4dbaa9fd
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97867980"
---
# <a name="overview-of-azure-stack-hub-capacity-planning"></a>Azure Stack Hub のキャパシティ プランニングの概要

Azure Stack Hub ソリューションを評価する場合は、Azure Stack Hub クラウドの容量全体に直接影響するハードウェア構成の選択肢を考慮してください。 

たとえば、CPU、メモリ密度、ストレージ構成、全体的なソリューションの規模 (つまり、サーバー数) を選択する必要があります。 従来の仮想化ソリューションとは異なり、これらのコンポーネントを単純に計算して使用可能な容量を決定することはできません。 Azure Stack Hub は、ソリューション自体の中でインフラストラクチャまたは管理コンポーネントをホストするように構築されています。 また、ソリューションの容量の一部が、回復性をサポートするために予約されており、ソリューションのソフトウェアを更新する際にテナントのワークロードの中断を最小限に抑えます。 

> [!IMPORTANT]
> このキャパシティ プランニングの情報と [Azure Stack Hub Capacity Planner](https://aka.ms/azstackcapacityplanner) は、Azure Stack Hub の計画と構成を決定するための始点です。 この情報は、独自の調査や分析の代わりに利用することを目的としていません。 Microsoft は、ここで提供される情報に対して、明示または黙示を問わず、一切の表明または保証を行わないものとします。
 
Azure Stack Hub ソリューションは、コンピューティングおよびストレージのハイパーコンバージド クラスターとして構築されています。 この収束により、"*スケール ユニット*" と呼ばれる、クラスター内でのハードウェア容量の共有が可能になります。 Azure Stack Hub では、スケール ユニットにより、リソースの可用性とスケーラビリティが提供されます。 スケール ユニットは、"*ホスト*" と呼ばれる、Azure Stack Hub サーバーのセットから構成されます。 すべてのインフラストラクチャ ソフトウェアは一連の仮想マシン (VM) 内でホストされ、テナント VM と同じ物理サーバーを共有します。 その後、すべての Azure Stack Hub VM はスケール ユニットの Windows Server クラスタリング テクノロジと個々の Hyper-V インスタンスによって管理されます。 

スケール ユニットにより、Azure Stack Hub の取得と管理が簡素化されます。 また、スケール ユニットにより、Azure Stack Hub 全体でのすべてのサービス (テナントとインフラストラクチャ) の移動とスケーラビリティも実現できます。 

以下のトピックでは、各コンポーネントについて詳しく説明しています。

- [Azure Stack Hub のコンピューティング](../operator/azure-stack-capacity-planning-compute.md?toc=/azure-stack/tdc/toc.json&bc=/azure-stack/breadcrumb/toc.json)
- [Azure Stack Hub のストレージ](../operator/azure-stack-capacity-planning-storage.md?toc=/azure-stack/tdc/toc.json&bc=/azure-stack/breadcrumb/toc.json)
- [Azure Stack Hub キャパシティ プランニング ツール](../operator/azure-stack-app-service-capacity-planning.md?toc=/azure-stack/tdc/toc.json&bc=/azure-stack/breadcrumb/toc.json)

## <a name="azure-stack-hub-ruggedized-high-and-low-models"></a>Azure Stack Hub ラグドの高モデルと低モデル

Azure Stack Hub ラグドの高モデルと低モデルの仕様を次の表に示します。

| コンポーネント               | 仕様 |
|-------------------------|---------------|
| CPU                     |[高]\:284 vCPU コア<br>[低]\:184 vCPU コア  |
| メモリ                  |[高]\:1,037 GB<br>[低]\:547 GB                |
| ストレージ                 |[高]\:34.2 TB<br>[低]\:15.4 TB                |
| Acceleration            |該当なし                                          |
| サイズと重量             |コンピューティング: 80.01 cm L x 60.45 cm W x 38.94 cm H (31.5 インチ L x 23.8 インチ W x 15.33 インチ H)<br>重量は 54 kg (120 ポンド) (2 台)<br>ネットワーク: 80.01 cm L x 60.45 cm W x 43.48 cm H (31.5 インチ L x 23.8 インチ W x 17.12 インチ H)<br>重量は 66 kg (145 ポンド) (1 台)              |
| オプションの構成 |プレヒーター<br>高または低の構成     |

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub のコンピューティング](../operator/azure-stack-capacity-planning-compute.md?toc=/azure-stack/tdc/toc.json&bc=/azure-stack/breadcrumb/toc.json)
[Azure Stack Hub のストレージ](../operator/azure-stack-capacity-planning-storage.md?toc=/azure-stack/tdc/toc.json&bc=/azure-stack/breadcrumb/toc.json)
[Azure Stack Hub Capacity Planner](../operator/azure-stack-app-service-capacity-planning.md?toc=/azure-stack/tdc/toc.json&bc=/azure-stack/breadcrumb/toc.json)
