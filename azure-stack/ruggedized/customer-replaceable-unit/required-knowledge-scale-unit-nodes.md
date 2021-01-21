---
title: 戦術的クラウド アプライアンスでスケール ユニット ノードを使用するために必要な知識
description: 戦術的クラウド アプライアンスでスケール ユニット ノードを使用するために必要な知識について説明します
author: PatAltimore
ms.topic: how-to
ms.date: 11/13/2020
ms.author: patricka
ms.reviewer: ''
ms.lastreviewed: ''
ms.openlocfilehash: 4df7cfa84ff9b4a08d6751aa19ffdb1430a80605
ms.sourcegitcommit: 9b0e1264ef006d2009bb549f21010c672c49b9de
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98256134"
---
# <a name="required-knowledge-for-working-with-scale-unit-nodes-in-a-tactical-cloud-appliance"></a>戦術的クラウド アプライアンスでスケール ユニット ノードを使用するために必要な知識

FRU の手順を完了するには、次の概念、ガイド、および Web サイトに習熟し、アクセスできる必要があります。

## <a name="privileged-access-workstation-and-the-privileged-endpoint"></a>特権アクセス ワークステーションと特権エンドポイント

特権アクセス ワークステーション (PAW) は、インターネットおよび外部の脅威ベクトルから保護された専用のワークステーションです。 これは、ハードウェア ライフサイクル ホスト (HLH) に配置することも、Azure Stack Hub スケール ユニット ノードへのルーティング可能なアクセスを使用して、外部の顧客ネットワークに配置することもできます。

PAW にアクセスするには、リモート デスクトップを使用してログインする必要があります。 顧客から資格情報と IP アドレスを取得します。

このコンピューターから、特権エンドポイント (PEP) にアクセスすることもできます。
特権アクセス ワークステーションと PEP の詳細については、「特権アクセス ワークステーションと特権エンドポイント アクセス」を参照してください。

## <a name="azure-stack-hub-administrator-portal"></a>Azure Stack Hub 管理者ポータル

顧客から管理者ポータルの資格情報と URL を取得します。
詳細については、「[Azure Stack Hub で](../../operator/azure-stack-manage-portals.md)
[管理者ポータルを使用する](../../operator/azure-stack-manage-portals.md)」を参照してください。

## <a name="dell-emc-poweredge-r640-installation-and-service-manual"></a>Dell EMC PowerEdge R640 設置およびサービス マニュアル

関連するハードウェアを物理的に交換する方法の詳細については、「[Dell EMC PowerEdge R640 設置およびサービス マニュアル](https://www.dell.com/support/manuals/us/en/04/poweredge-r640/per640_ism_pub/dell-emc-poweredge-r640-overview?guid=guid-f39be9ba-158c-45e3-b8b1-f07bb750d6d4)」の PowerEdge R640 システム コンポーネントの取り付けと取り外しの手順を参照してください。
「[Installing and removing](https://www.dell.com/support/manuals/us/en/04/poweredge-r640/per640_ism_pub/installing-and-removing-system-components?guid=guid-5a5943c4-fe26-4faa-a10c-2afa4c1993ff&lang=en-us)
[system components](https://www.dell.com/support/manuals/us/en/04/poweredge-r640/per640_ism_pub/installing-and-removing-system-components?guid=guid-5a5943c4-fe26-4faa-a10c-2afa4c1993ff&lang=en-us)」(システム コンポーネントの取り付けと取り外し) を参照してください。

## <a name="microsoft-azure-stack-hub-tactical-cloud-appliance-service-manual"></a>Microsoft Azure Stack Hub 戦略的クラウド アプライアンス サービス マニュアル

Microsoft Azure Stack Hub 戦略的クラウド アプライアンス サービス マニュアルには、Tracewell Tactical Pods からスケール ユニットのノード サーバーを取り外すための手順が記載されています。

## <a name="dell-emc-poweredge-idrac"></a>Dell EMC PowerEdge iDRAC

Dell EMC PowerEdge iDRAC Web インターフェイスを移動して使用する方法について理解しておく必要があります。 iDRAC の使用方法の詳細については、「[Integrated Dell Remote Access Controller 9 ユーザー ガイド](https://www.dell.com/support/manuals/us/en/04/poweredge-r840/idrac9_4.00.00.00_ug_new/overview-of-idrac?guid=guid-a03c2558-4f39-40c8-88b8-38835d0e9003)」を参照してください。\'