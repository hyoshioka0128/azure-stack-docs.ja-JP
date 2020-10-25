---
title: Azure Stack Hub のためのキャパシティ プランニングの概要 | Microsoft Docs
description: Azure Stack Hub デプロイのためのキャパシティ プランニングについて説明します。
services: azure-stack
documentationcenter: ''
author: justinha
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/10/2020
ms.author: justinha
ms.reviewer: prchint
ms.lastreviewed: 01/10/2020
ms.openlocfilehash: a608a64de28104513cb708ef350eb4db672f5f65
ms.sourcegitcommit: e4e2cc6a68f02c3e856f58ca5ee51b3313c7ff8f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/19/2020
ms.locfileid: "92183494"
---
# <a name="overview-of-azure-stack-hub-capacity-planning"></a>Azure Stack Hub のキャパシティ プランニングの概要

Azure Stack Hub ソリューションを評価する場合は、Azure Stack Hub クラウドの容量全体に直接影響するハードウェア構成の選択肢を考慮してください。 

たとえば、CPU、メモリ密度、ストレージ構成、全体的なソリューションの規模 (つまり、サーバー数) を選択する必要があります。 従来の仮想化ソリューションとは異なり、これらのコンポーネントを単純に計算して使用可能な容量を決定することはできません。 Azure Stack Hub は、ソリューション自体の中でインフラストラクチャまたは管理コンポーネントをホストするように構築されています。 また、ソリューションの容量の一部が、回復性をサポートするために予約されており、ソリューションのソフトウェアを更新する際にテナントのワークロードの中断を最小限に抑えます。 

> [!IMPORTANT]
> このキャパシティ プランニングの情報と [Azure Stack Hub Capacity Planner](https://aka.ms/azstackcapacityplanner) は、Azure Stack Hub の計画と構成を決定するための始点です。 この情報は、独自の調査や分析の代わりに利用することを目的としていません。 Microsoft は、ここで提供される情報に対して、明示または黙示を問わず、一切の表明または保証を行わないものとします。
 
Azure Stack Hub ソリューションは、コンピューティングおよびストレージのハイパーコンバージド クラスターとして構築されています。 この収束により、"*スケール ユニット*" と呼ばれる、クラスター内でのハードウェア容量の共有が可能になります。 Azure Stack Hub では、スケール ユニットにより、リソースの可用性とスケーラビリティが提供されます。 スケール ユニットは、"*ホスト*" と呼ばれる、Azure Stack Hub サーバーのセットから構成されます。 すべてのインフラストラクチャ ソフトウェアは一連の仮想マシン (VM) 内でホストされ、テナント VM と同じ物理サーバーを共有します。 その後、すべての Azure Stack Hub VM はスケール ユニットの Windows Server クラスタリング テクノロジと個々の Hyper-V インスタンスによって管理されます。 

スケール ユニットにより、Azure Stack Hub の取得と管理が簡素化されます。 また、スケール ユニットにより、Azure Stack Hub 全体でのすべてのサービス (テナントとインフラストラクチャ) の移動とスケーラビリティも実現できます。 

以下のトピックでは、各コンポーネントについて詳しく説明しています。

- [Azure Stack Hub のコンピューティング](../operator/azure-stack-capacity-planning-compute.md)
- [Azure Stack Hub のストレージ](../operator/azure-stack-capacity-planning-storage.md)
- [Azure Stack Hub キャパシティ プランニング ツール](azure-stack-capacity-planner.md)

