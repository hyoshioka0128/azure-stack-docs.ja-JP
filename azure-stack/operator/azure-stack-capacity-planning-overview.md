---
title: キャパシティ プランニングの概要
titleSuffix: Azure Stack Hub
description: Azure Stack Hub デプロイのためのキャパシティ プランニングについて説明します。
author: PatAltimore
ms.topic: conceptual
ms.date: 05/31/2019
ms.author: patricka
ms.reviewer: prchint
ms.lastreviewed: 05/31/2019
ms.openlocfilehash: 4b3a6b58b6738c4248eaeb5d76b48fa9d1fdb442
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97871354"
---
# <a name="capacity-planning-for-azure-stack-hub-overview"></a>Azure Stack Hub のためのキャパシティ プランニングの概要

Azure Stack Hub ソリューションを評価する場合は、Azure Stack Hub クラウドの容量全体に直接影響するハードウェア構成の選択肢を考慮してください。

CPU、メモリ密度、ストレージ構成、全体的なソリューションの規模 (つまり、サーバー数) を選択する必要があります。 ただし、一部の容量が既に使用されているため、使用可能な容量を決定することは、従来の仮想化ソリューションとは異なります。 Azure Stack Hub は、ソリューション自体の中でインフラストラクチャまたは管理コンポーネントをホストするように構築されています。 また、ソリューションの容量の一部が、回復性をサポートするために予約されています。 回復性とは、テナントのワークロードの中断を最小限に抑える形でソリューションのソフトウェアを更新することです。

> [!IMPORTANT]
> このキャパシティ プランニングの情報と [Azure Stack Hub Capacity Planner](https://aka.ms/azstackcapacityplanner) は、Azure Stack Hub の計画と構成を決定するための始点です。 この情報は、独自の調査や分析の代わりに利用することを目的としていません。 Microsoft は、ここで提供される情報に対して、明示または黙示を問わず、一切の表明または保証を行わないものとします。

## <a name="hyperconvergence-and-the-scale-unit"></a>ハイパーコンバージェンスとスケール ユニット
Azure Stack Hub ソリューションは、コンピューティングおよびストレージのハイパーコンバージド クラスターとして構築されています。 この収束により、"*スケール ユニット*" と呼ばれる、クラスター内でのハードウェア容量の共有が可能になります。 Azure Stack Hub では、スケール ユニットにより、リソースの可用性とスケーラビリティが提供されます。 スケール ユニットは、"*ホスト*" と呼ばれる、Azure Stack Hub サーバーのセットから構成されます。 すべてのインフラストラクチャ ソフトウェアは一連の仮想マシン (VM) 内でホストされ、テナント VM と同じ物理サーバーを共有します。 その後、すべての Azure Stack Hub VM はスケール ユニットの Windows Server クラスタリング テクノロジと個々の Hyper-V インスタンスによって管理されます。

スケール ユニットにより、Azure Stack Hub の取得と管理が簡素化されます。 また、スケール ユニットにより、Azure Stack Hub 全体でのすべてのサービス (テナントとインフラストラクチャ) の移動とスケーラビリティも実現できます。

以下のトピックでは、各コンポーネントについて詳しく説明しています。

- [Azure Stack Hub のコンピューティング](azure-stack-capacity-planning-compute.md)
- [Azure Stack Hub のストレージ](azure-stack-capacity-planning-storage.md)
- [Azure Stack Hub キャパシティ プランニング ツール](azure-stack-capacity-planner.md)
