---
title: ハードウェア ライフサイクル ホストの正常性の確認
description: ハードウェア ライフサイクル ホストの正常性の確認方法について説明します
author: PatAltimore
ms.topic: how-to
ms.date: 11/13/2020
ms.author: patricka
ms.reviewer: ''
ms.lastreviewed: ''
ms.openlocfilehash: 47d3d5198418042ad0bc24e35414bd309a492bc2
ms.sourcegitcommit: d719f148005e904fa426a001a687e80730c91fda
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/06/2021
ms.locfileid: "97909856"
---
# <a name="verifying-hardware-lifecycle-host-health"></a>ハードウェア ライフサイクル ホストの正常性の確認



ハードウェアの交換が完了し、サイトから移動する "前"に、iDRAC を使用してハードウェア ライフサイクル ホストの正常性を確認します。


1.  iDRAC Web インターフェイスでシステム正常性を確認し、FRU プロセスの前に Web インターフェイスに問題があった場合は、その問題が解決されていることを確認します。

    ![[概要] アクションが強調表示されている [システム] ページを示すスクリーンショット。](media/image-5.png)
    
2.  Hyper-V 仮想マシンが **Running** (実行中) の状態であることを確認します。

    ![[仮想マシン] セクションが強調表示されている [Hyper-V マネージャー] ページを示すスクリーンショット。](media/image-55.png) 

