---
title: Azure Stack の顧客課金モデルの概要 - MDC | Microsoft Docs
description: Modular Data Center (MDC) で Azure Stack ユーザーに対して、リソース使用量の請求がどのように行われるのかについて説明します。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: tzl
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/04/2020
ms.author: sethm
ms.reviewer: alfredop
ms.lastreviewed: 12/23/2019
ms.openlocfilehash: 15f894a668374be5380f322d368b76d88bb93cba
ms.sourcegitcommit: d719f148005e904fa426a001a687e80730c91fda
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/06/2021
ms.locfileid: "97910756"
---
# <a name="billing-model-overview---modular-data-center-mdc"></a>課金モデルの概要 - Modular Data Center (MDC)

MDC または Azure Stack Hub ラグドのユーザーは、各アプライアンスの保有期間に基づいて Microsoft から請求されます。 料金は期間単位であり、基本的なコンピューティング、ストレージ、およびネットワーク サービスを使用する権利が含まれています。 App Service、Event Hubs、およびその他の PaaS サービスの使用状況に基づいて課金され、Azure Stack Hub ラグド と MDC で実行される Windows Server PAYG VM も課金されます。 完全に切断され、使用状況データを報告できない場合は、PaaS サービスの容量ライセンスを取得し、Windows VM の BYOL (ライセンス持ち込み) を取得する必要があります。

アプライアンスの注文の出荷から 14 日後に課金が開始されます。 Azure Stack Hub Containerized の場合、最初の請求期間は 1 年です。 1 年後、注文を 1 年間延長することができ、もう 1 年使用量が課金されます。 注文が延長されず、デバイスが返却されていない場合は、月単位で課金が続行されます。 Azure Stack Hub ラグドの場合、課金は常に月単位で行われます。

アプライアンスが Microsoft に返却された場合は、アプライアンスが Microsoft の倉庫に届くとすぐに課金が停止されます。

## <a name="next-steps"></a>次のステップ

[Usage API リファレンス](analyze-usage-tzl.md)
