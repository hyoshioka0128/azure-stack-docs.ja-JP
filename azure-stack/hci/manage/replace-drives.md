---
title: Azure Stack HCI で障害が発生したドライブを交換する
description: Azure Stack HCI で障害が発生したドライブを交換する方法について説明します。
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.date: 07/21/2020
ms.openlocfilehash: d3d03f1e5cc89186e2eb8198b52e96bffbd21768
ms.sourcegitcommit: 0e52f460295255b799bac92b40122a22bf994e27
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/21/2020
ms.locfileid: "86866485"
---
# <a name="replace-failed-drives-on-azure-stack-hci"></a>Azure Stack HCI で障害が発生したドライブを交換する

> 適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019

Azure Stack HCI は直接接続された SATA、SAS、NVMe、または永続メモリのドライブに対応しています。これらは、それぞれ 1 台のサーバーのみに物理的に接続されます。 ドライブに障害が発生した場合は、物理サーバー ハードウェアにアクセスして交換する必要があります。

## <a name="find-the-alert"></a>アラートを見つける
ドライブに障害が発生すると、**Windows Admin Center** ダッシュボードの左上の **[アラート]** 領域にアラートが表示されます。 また、左側にあるナビゲーションから **[ドライブ]** を選択するか、右下隅にあるタイルで **[VIEW DRIVES >]\(ドライブの表示 >\)** リンクをクリックして、ドライブを参照してその状態を自分で確認することもできます。 **[表示]** タブのグリッドでは、並べ替え、グループ化、およびキーワード検索がサポートされています。

1. ダッシュボードからアラートをクリックすると、ドライブの物理的な場所などの詳細が表示されます。
1. 詳細については、 **[ドライブ]** 詳細ページへの **[Go to drive]\(ドライブに移動\)** ショートカットをクリックします。
1. ハードウェアでサポートされている場合は、 **[Turn light on/off]\(ライトをオンにする/オフにする\)** をクリックして、ドライブのインジケーター ライトを制御できます。
   記憶域スペース ダイレクトは、障害が発生したドライブを自動的に廃止して退避させます。 これが発生すると、ドライブの状態は廃止になり、記憶域容量バーは空になります。
1. 障害が発生したドライブを取り外し、代わりのものを挿入します。

## <a name="wait-for-the-alert-to-clear"></a>アラートが消去されるのを待ちます。
**[ドライブ] > [インベントリ]** に新しいドライブが表示されます。 しばらくしてアラートが消去されると、ボリュームは OK 状態に戻り、記憶域は新しいドライブに再調整されます。ユーザー操作は不要です。

## <a name="next-steps"></a>次のステップ
- ドライブ レベルなど、さまざまなレベルで記憶域の正常性を追跡する方法の詳細については、「[正常性と動作状態](/windows-server/storage/storage-spaces/storage-spaces-states)」を参照してください。
- PMem を使用している場合は、「[永続メモリの理解と配置](/windows-server/storage/storage-spaces/deploy-pmem)」を参照してください。