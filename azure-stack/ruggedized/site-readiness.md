---
title: Azure Stack Hub ラグドの Azure Stack Hub に対するサイトの準備 | Microsoft Docs
description: Azure Stack Hub ラグドのサイトの準備仕様について説明します。
services: azure-stack
documentationcenter: ''
author: PatAltimore
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/14/2020
ms.author: wamota
ms.reviewer: wamota
ms.lastreviewed: 10/14/2020
ms.openlocfilehash: 5a9eb7c73a30268eb4e53855e9ce277517019e4a
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97874304"
---
# <a name="azure-stack-hub-ruggedized-site-readiness"></a>Azure Stack Hub ラグドのサイトの準備

このトピックでは、Azure Stack Hub ラグドの環境と PDU 電力低下に関する要件について説明します。 

>[!NOTE]
>これらの値は、施設の計画のみを目的としており、概算の慎重な値です。 実際の要件は異なる場合があります。

## <a name="environmental-requirements"></a>環境要件

次の構成の Azure Stack Hub ラグド ソリューションの環境要件を次の表に示します。

- 14 TB スケール ユニット
- 200 ボルト AC 入力電圧
- 35°C の最高周囲温度

*表 1.高/低の構成環境要件*

| Object                         | Azure Stack Hub ラグドの要件               |
|--------------------------------|--------------------------------|
|操作温度           | Azure Stack Hub ラグドの動作温度 (ヒーター要件あり): -32°C (-25.6°F) から 43°C (109°F)。    |
|湿度と湿気           | ストレージ:33°C (91°F) の最大露点 5% から 95% RH。 大気は常に結露しないよう保ってください。 <br> 運転中:29°C (84°F) の最大露点 5% から 85% RH。
|物理的な接続性           | Azure Stack Hub ラグドは、次の方法で物理的に接続できます。 <br>4x10G SR SFP+ <br>4x1000BASE-SX <br>4x 1000BASE-T
|電源入力                     | 最大 4.981 Kw、平均 4.391 KW<br> 入力コネクタ C13/C14<br> 次の内容を入力します。100-240V 50/60Hz

## <a name="pdu-power-drop-requirements"></a>PDU の電源引き込み線の要件

Azure Stack Hub ラグドに必要な電源引き込み線を次の表に示します。

*表 2. 必要な電力低下の数*

| 構成  | 1 フェーズ  | 3 フェーズ Delta |3 フェーズ Wye |
|----------------|---------------|-------------------|----------------|
|高/低        | 2             | 2                 | 2              |

Azure Stack 統合システムを使用すると、さまざまな PDU コネクタの種類を使用して、最適な方法でデータセンターに統合できます。 コネクタの種類を次の表に示します。

*表 3. PDU とコネクタのオプション*

| 場所     | 1 フェーズ                                | 3 フェーズ Delta                                   | 3 フェーズ Wye                                        |
|--------------|---------------------------------------------|-----------------------------------------------------|-----------------------------------------------------------|
|北米 |- L630P<br>- L7-30P<br>- Russellstoll 3750DP |- Hubbell Pro CS8365L<br>- Russellstoll 9P54U2T/1100 |- Hubbell C530P6S<br>- ABL Sursum S52S0A<br>- Flying Leads |


