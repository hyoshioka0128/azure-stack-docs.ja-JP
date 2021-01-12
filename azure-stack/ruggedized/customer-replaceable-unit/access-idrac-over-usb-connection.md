---
title: USB 接続経由で iDRAC にアクセスする
description: USB 接続経由で iDRAC にアクセスする方法について説明します
author: PatAltimore
ms.topic: how-to
ms.date: 11/13/2020
ms.author: patricka
ms.reviewer: ''
ms.lastreviewed: ''
ms.openlocfilehash: 5c270de88d78bb8cb7ba1f7b9216c4a160a4aae8
ms.sourcegitcommit: d719f148005e904fa426a001a687e80730c91fda
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/06/2021
ms.locfileid: "97910060"
---
# <a name="accessing-the-idrac-interface-over-a-direct-usb-connection"></a>直接 USB 接続経由で iDRAC インターフェイスにアクセスする

iDRAC ダイレクト機能では、iDRAC USB ポートにノート PC を直接接続できます。 この機能を利用すると、高度なサーバー管理とサービス提供のために、Web インターフェイス、RACADM、WSMan などの iDRAC インターフェイスと直接やりとりできます。



USB ポート経由で iDRAC インターフェイスにアクセスするには、ノート PC から次の操作を実行します。

**手順**

1.  ノート PC で無線ネットワークがあればそれをオフにし、それ以外で有線のネットワークにつながっているものがあれば、切断します。

2.  ノート PC からサーバーの前面にある iDRAC ダイレクト ポートにマイクロ USB ケーブルを接続します。
    図の項目 4 を参照してください。

    ![電源ボタン、USB、およびマイクロ USB ポートを示す図。](media/image-67.png)

3.  ノート PC が IP アドレス 169.254.0.4 を取得するまで待ちます。

    IP アドレスの取得には数秒かかることがあります。 iDRAC では IP アドレス 169.254.0.3 が取得されます。

4.  iDRAC Web インターフェイスに接続します。

    iDRAC Web インターフェイスにアクセスするには、ブラウザーを開き、169.254.0.3 に移動します。

5.  必要なアクティビティを完了します。

    

    > [!NOTE]
    > 上の図の項目 3 である USB ポートが iDRAC で使用されているとき、LED が点滅し、アクティブであることを示します。 点滅の頻度は 1 秒あたり 4 回です。
    
6.  マイクロ USB ケーブルを取り外します。

    必要な操作が完了したら、システムから USB ケーブルを取り外します。 LED がオフになります。
    
