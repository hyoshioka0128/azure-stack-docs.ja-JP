---
title: Azure Stack Hub 上の IoT Hub を削除する方法
description: Azure Stack Hub 上の IoT Hub をアンインストールする手順
author: yiyiguo
ms.author: yiygu
ms.service: azure-stack
ms.topic: how-to
ms.date: 1/6/2020
ms.openlocfilehash: 4095336f6a18441512e8c1052051d6e1608d904a
ms.sourcegitcommit: 0e3296fb27b9dabbc2569bf85656c4c7b1d58ba9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/30/2020
ms.locfileid: "93050337"
---
# <a name="how-to-remove-iot-hub-on-azure-stack-hub"></a>Azure Stack Hub 上の IoT Hub を削除する方法

[!INCLUDE [preview-banner](../includes/iot-hub-preview.md)]

この記事では、Azure Stack Hub 上の IoT Hub リソース プロバイダーを削除する手順を紹介しています。 この作業には約 37 分かかります。

> [!WARNING]
> IoT Hub のアンインストール後は、" **_すべてのデバイスとデータが削除されます_** "。 この操作を元に戻すことは **できません** 。

## <a name="uninstalling-iot-hub"></a>IoT Hub をアンインストールする

1) **[Marketplace management]\(Marketplace の管理\)** に移動します。 IoT Hub がインストール済みとして一覧に表示されます。 **[IoT Hub]** をクリックします。

    [![リソース プロバイダーの一覧](../operator/media/iot-hub-rp-remove/uninstall1.png)](../operator/media/iot-hub-rp-remove/uninstall1.png#lightbox)

2) **[IoT Hub]** の下の **[アンインストール]** をクリックし、リソース プロバイダーの名前として「 **microsoft.iothub** 」を入力し、その下にある **[アンインストール]** ボタンをクリックします。

    [![IoT Hub のアンインストールと確定](../operator/media/iot-hub-rp-remove/uninstall2.png)](../operator/media/iot-hub-rp-remove/uninstall2.png#lightbox)

3) アンインストールが完了するのを待ちます。 IoT Hub の再インストールを試みる場合は、10 分以上時間をおいてください。

>[!IMPORTANT]
>依存関係 ( Event Hub など) は、アンインストール **されません** 。 マーケットプレースからインストールしたいずれかの依存関係をアンインストール (削除) したい場合は、その作業を個別に行う必要があります。

## <a name="next-steps"></a>次のステップ

再インストールについては、[接続環境の Azure Stack に IoT Hub リソース プロバイダーをインストールする](iot-hub-rp-install.md)方法に関するページを参照してください。
