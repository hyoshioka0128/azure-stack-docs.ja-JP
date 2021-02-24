---
title: Azure Stack の顧客課金モデルの概要 | Microsoft Docs
description: Azure Stack ユーザーに対して、リソース使用量の請求がどのように行われるのかについて説明します。
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
ms.date: 12/23/2019
ms.author: sethm
ms.reviewer: alfredop
ms.lastreviewed: 12/23/2019
ms.openlocfilehash: 625ec975ee5f9f3f8a80beb505f2bd10929dcda0
ms.sourcegitcommit: aef251d6771400b21a314bbfbea4591ab263f8fb
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/22/2020
ms.locfileid: "97726154"
---
# <a name="billing-model-overview"></a>課金モデルの概要

MDC または Azure Stack Hub ラグドのユーザーは、各アプライアンスの保有期間に基づいて Microsoft から請求されます。 料金は期間単位であり、基本的なコンピューティング、ストレージ、およびネットワーク サービス、App Service、IoT Hub とその前提条件、Key Vault などの補助的なサービスを使用する権利が含まれています。 ゲスト オペレーティング システムとしての Windows Server の使用も含まれています。

アプライアンスの注文の出荷から 14 日後に課金が開始されます。 Azure Stack Hub Containerized の場合、最初の請求期間は 1 年です。 1 年後、注文を 1 年間延長することができ、もう 1 年使用量が課金されます。 注文が延長されず、デバイスが返却されていない場合は、月単位で課金が続行されます。 Azure Stack Hub ラグドの場合、課金は常に月単位で行われます。

アプライアンスが Microsoft に返却された場合は、アプライアンスが Microsoft の倉庫に届くとすぐに課金が停止されます。


