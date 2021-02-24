---
title: Azure Stack Hub の Event Hubs を削除する方法
description: Azure Stack Hub の Event Hubs リソース プロバイダーを削除する方法について説明します。
author: BryanLa
ms.author: bryanla
ms.service: azure-stack
ms.topic: how-to
ms.date: 12/09/2019
ms.reviewer: jfggdl
ms.lastreviewed: 12/09/2019
ms.openlocfilehash: b69d5b7a7b621d22745c89c18a3e7f000e54bcbd
ms.sourcegitcommit: f56a5b287c90b2081ae111385c8b7833931d4059
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/11/2020
ms.locfileid: "97343747"
---
# <a name="how-to-remove-event-hubs-on-azure-stack-hub"></a>Azure Stack Hub の Event Hubs を削除する方法

> [!WARNING]
> Event Hubs をアンインストールすると、リソース プロバイダーと、ユーザーが作成したすべての Event Hubs クラスター、名前空間、および Event Hubs リソースが削除 (消去) されます。 また、関連付けられているイベント データも削除されます。  
> Event Hubs のアンインストールを決定する際は、細心の注意を払ってください。 Event Hubs をアンインストールしても、Event Hubs のインストールに使用したパッケージは削除 **されません**。 これを行うには、「[Event Hubs パッケージを削除する](#delete-event-hubs-packages)」を参照してください。

## <a name="uninstall-event-hubs"></a>Event Hubs をアンインストールする

この一連の手順では、クラスター、名前空間、イベント ハブ、リソース プロバイダーなど、すべての Event Hubs リソースが削除されます。

Event Hubs と、ユーザーが作成したすべての関連リソースを削除するには、次の手順を実行します。

1. Azure Stack Hub 管理者ポータルにサインインします。
2. 左側にある **[Marketplace Management]** を選択します。
3. **[リソース プロバイダー]** を選択します。
4. リソース プロバイダーの一覧から **[Event Hubs]** を選択します。 表示される検索テキスト ボックスに「Event Hubs」と入力することで、一覧をフィルター処理することもできます。

   [![Event Hubs を削除する 1](media/event-hubs-rp-remove/1-uninstall.png)](media/event-hubs-rp-remove/1-uninstall.png#lightbox)

5. ページの上部に表示されるオプションから **[アンインストール]** を選択します。

   [![Event Hubs を削除する 2](media/event-hubs-rp-remove/2-uninstall.png)](media/event-hubs-rp-remove/2-uninstall.png#lightbox)

6. リソース プロバイダーの名前を入力し、 **[アンインストール]** を選択します。 この操作によって以下がアンインストールされることを確認するメッセージが表示されます。
   - Event Hubs リソース プロバイダー
   - ユーザーが作成したすべてのクラスター、名前空間、イベント ハブ、およびイベント データ。

   [![Event Hubs を削除する 3](media/event-hubs-rp-remove/3-uninstall.png)](media/event-hubs-rp-remove/3-uninstall.png#lightbox)

   [![Event Hubs の削除中 4](media/event-hubs-rp-remove/4-uninstall.png)](media/event-hubs-rp-remove/4-uninstall.png#lightbox)

   > [!IMPORTANT]
   > 再度 Event Hubs をインストールする場合は、Event Hubs が正常に削除されてから 10 分以上待つ必要があります。 これは、クリーンアップ アクティビティがまだ実行中である可能性があり、その場合は新しいインストールと競合することがあるためです。

## <a name="delete-event-hubs-packages"></a>Event Hubs パッケージを削除する

このオプションは、Event Hubs をアンインストールした後に、Event Hubs のインストールに使用したパッケージも削除する場合に使用します。 

## <a name="next-steps"></a>次のステップ

再インストールするには、[Event Hubs リソース プロバイダーのインストール](event-hubs-rp-install.md)に関するページに戻ってください。