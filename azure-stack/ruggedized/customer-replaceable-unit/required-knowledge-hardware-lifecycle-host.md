---
title: ハードウェア ライフサイクル ホストを操作するために必要な知識
description: ハードウェア ライフサイクル ホストを操作するために必要な知識について説明します。
author: PatAltimore
ms.topic: how-to
ms.date: 02/05/2021
ms.author: patricka
ms.reviewer: ''
ms.lastreviewed: ''
ms.openlocfilehash: 3d3e650e9b6b1b6c37e2f265aa5c049246600c2f
ms.sourcegitcommit: 5ea0e915f24c8bcddbcaf8268e3c963aa8877c9d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/14/2021
ms.locfileid: "100487784"
---
# <a name="required-knowledge-for-working-with-the-hardware-lifecycle-host"></a>ハードウェア ライフサイクル ホストを操作するために必要な知識

FRU の手順を完了するには、次の概念とガイドに習熟し、アクセスできる必要があります。

## <a name="hardware-lifecycle-host"></a>ハードウェア ライフサイクル ホスト

ハードウェア ライフサイクル ホスト (HLH) は、Azure Stack Hub ラックの上部に搭載された物理的な管理サーバーです。 このホストにアクセスするには、次の 3 つの方法のいずれかを使用して接続します。

* Direct (クラッシュ カート)
* iDRAC (サービス ポート)
* iDRAC (IP アクセス)

データ センター内にいる場合は、VGA および USB の各ポートを使用して HLH に直接接続することができます。 たとえば、クラッシュ カートに接続するとします。

データ センター内にいる場合は、マイクロ USB ケーブルを使用して、自分のノート PC を iDRAC 9 サービス ポートに接続します。 詳細については、「直接 USB 接続経由で iDRAC インターフェイスにアクセスする」を参照してください。

顧客と協力して、その管理ネットワークおよび管理ワークステーションから iDRAC IP を介して HLH に接続してください。

> [!NOTE]
> HLH iDRAC に直接接続できるのは、あらかじめスイッチ ACL に追加されているネットワークからのみです。

## <a name="credentials"></a>資格情報

次の資格情報を取得するには、顧客と協力してください。

* HLH
* administrator
* iDRAC アカウント (省略可能)

完全な管理者権限を持つ Windows アカウント。

クラッシュ カートを使用してサーバーに直接接続していない場合、仮想 KVM にアクセスするには iDRAC アカウントの資格情報が必要です。


