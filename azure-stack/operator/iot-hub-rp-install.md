---
title: Azure Stack Hub に IoT Hub をインストールする方法
description: Azure Stack Hub に IoT Hub リソース プロバイダーをインストールする方法について説明します。
author: yiyiguo
ms.author: yiygu
ms.service: azure-stack
ms.topic: how-to
ms.date: 1/6/2020
zone_pivot_groups: state-connected-disconnected
ms.openlocfilehash: 47d06bc7363a9ce8d4de8971bf26c38bfdb16bcc
ms.sourcegitcommit: 9b0e1264ef006d2009bb549f21010c672c49b9de
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98256117"
---
# <a name="how-to-install-iot-hub-on-azure-stack-hub"></a>Azure Stack Hub に IoT Hub をインストールする方法

[!INCLUDE [preview-banner](../includes/iot-hub-preview.md)]

この記事では、IoT Hub リソース プロバイダーをダウンロードしてインストールし、サブスクリプションの顧客に提供できるようにする方法について説明します。 IoT Hub のインストール プロセスには約 2 時間かかります。

## <a name="download-iot-hub"></a>IoT Hub のダウンロード

<!-- ### Connected Scenario -->
::: zone pivot="state-connected"
Azure Stack Hub が Azure Marketplace にアクセスできる場合は、このセクションの手順に従って、IoT Hub とその依存関係をダウンロードしてインストールします。 

接続されたデプロイの場合に IoT Hub をダウンロードするには、次の手順を実行します。

1. Azure Stack Hub 管理者ポータルにサインインします。 
2. 左側にある **[Marketplace Management]** を選択し、次に **[リソース プロバイダー]** を選択してから、 **[+ Add from Azure]\(+ Azure から追加\)** をクリックします。

    [![Marketplace での RP の検索](media/iot-hub-rp-install/marketplace-rp-add-from-azure.png)](media/iot-hub-rp-install/marketplace-rp-add-from-azure.png#lightbox)

3. 必要に応じて "IoT Hub" をフィルター処理し、**IoT Hub** パッケージを選択します。

    [![Marketplace の IoT Hub RP](../operator/media/iot-hub-rp-install/download1.png)](../operator/media/iot-hub-rp-install/download1.png#lightbox)

4. **IoT Hub** パッケージのページで、 **[ダウンロード]** を選択します。

    [![IoT Hub パッケージの詳細](../operator/media/iot-hub-rp-install/download2.png)](../operator/media/iot-hub-rp-install/download2.png#lightbox)

5. パッケージのダウンロードが完了するまで待ちます。 状態が **[ダウンロード]** と表示され、最大 10 分かかることがあります。

    [![IoT Hub パッケージのダウンロード状態](../operator/media/iot-hub-rp-install/download3.png)](../operator/media/iot-hub-rp-install/download3.png#lightbox)

6. パッケージがダウンロードされると、 **[Marketplace Management]** ページで状態が **[未インストール]** に変わります。

    [![ダウンロードされた IoT Hub パッケージが未インストール](../operator/media/iot-hub-rp-install/download4.png)](../operator/media/iot-hub-rp-install/download4.png#lightbox)
::: zone-end

<!-- ### Disconnected or partially connected scenario -->
::: zone pivot="state-disconnected"
接続されていないか、部分的に接続されているデプロイの場合に IoT Hub をダウンロードするには、パッケージをローカル コンピューターにダウンロードします。 ダウンロードが完了したら、Azure Stack Hub インスタンスにインポートします。

1. まだ行っていない場合は、[Marketplace 項目のダウンロードに関するページの「接続されていないか、部分的に接続されているシナリオ」](azure-stack-download-azure-marketplace-item.md?pivots=state-disconnected)の手順に従ってください。 ここでは、Marketplace シンジケーション ツールをダウンロードして実行します。これにより、IoT Hub パッケージをダウンロードできるようになります。
2. シンジケーション ツールの [Azure Marketplace の項目] ウィンドウが開いたら、[IoT Hub] を見つけて選択し、必要なパッケージをローカル コンピューターにダウンロードします。
3. ダウンロードが完了したら、Azure Stack Hub インスタンスにパッケージをインポートし、Marketplace に発行します。
::: zone-end

## <a name="install-iot-hub"></a>IoT Hub のインストール

**[Marketplace management]** ページで、次の手順に従って IoT Hub パッケージをインストールします。

1. **[IoT Hub]** の行、 **[インストールの開始]** の順にクリックします。

    [![IoT Hub RP のインストールの待機](../operator/media/iot-hub-rp-install/install1.png)](../operator/media/iot-hub-rp-install/install1.png#lightbox)

2. **[必須コンポーネントのインストール]** をクリックします。

    [![IoT Hub RP の必須コンポーネントのインストール](../operator/media/iot-hub-rp-install/install2.png)](../operator/media/iot-hub-rp-install/install2.png#lightbox)

3. インストールの状態は、通知ペインで追跡できます。 この手順には約 10 分かかります。

    [![IoT Hub RP の必須コンポーネントのインストールのインストール中](../operator/media/iot-hub-rp-install/install3.png)](../operator/media/iot-hub-rp-install/install3.png#lightbox)

4. 必須コンポーネントのインストールが完了するまで待ちます。 通常、この手順には環境に応じて 5 から 10 分かかります。

    [![IoT Hub RP シークレットの準備](../operator/media/iot-hub-rp-install/install4.png)](../operator/media/iot-hub-rp-install/install4.png#lightbox)

5. **[Prepare Secrets]\(シークレットの準備\)** の下にある **[証明書の追加]** をクリックします。 前提条件で作成した pfx 証明書を指定します。

    [![IoT Hub RP シークレットの準備 - 証明書のアップロード](../operator/media/iot-hub-rp-install/install5.png)](../operator/media/iot-hub-rp-install/install5.png#lightbox)

6. 作成した pfx とパスワード (スクリプトへの入力) を参照して指定します。

    [![IoT Hub RP シークレットの準備 - 証明書の選択](../operator/media/iot-hub-rp-install/install6.png)](../operator/media/iot-hub-rp-install/install6.png#lightbox)

    [![IoT Hub RP シークレットの準備 - パスワードの指定](../operator/media/iot-hub-rp-install/install61.png)](../operator/media/iot-hub-rp-install/install61.png#lightbox)

7. **[Install Resource Provider]\(リソース プロバイダーのインストール\)** の下にある **[インストール]** をクリックします。

    [![IoT Hub RP シークレットの準備 - 完了](../operator/media/iot-hub-rp-install/install7.png)](../operator/media/iot-hub-rp-install/install7.png#lightbox)

8. インストールが開始されると、Marketplace または通知ペインにデプロイの状態が表示されます。

    [![IoT Hub RP のインストールの進行状況](../operator/media/iot-hub-rp-install/install8.png)](../operator/media/iot-hub-rp-install/install8.png#lightbox)

9. インストールには、90 から 120 分かかることがあります。 インストールが完了するのを待ちます。

    [![IoT Hub RP のインストールの完了](../operator/media/iot-hub-rp-install/install91.png)](../operator/media/iot-hub-rp-install/install91.png#lightbox)

    [![Marketplace RP - インストールされた RP](../operator/media/iot-hub-rp-install/install92.png)](../operator/media/iot-hub-rp-install/install92.png#lightbox)

IoT Hub リソース プロバイダーが正常にインストールされました。 使用を開始するには、次の手順を実行します。

1. 必要に応じて、指示に従って[プラン、オファー、サブスクリプションを作成](./service-plan-offer-subscription-overview.md)します。

2. サブスクリプションが既に存在する場合は、関連付けられているオファーまたはプランを更新して、**Microsoft.Devices** サービスを含めます。 **[プラン]**  -> **更新するプランの選択** ->  **[Add Service and quota]\(サービスとクォータの追加\)** に移動します。

3. **Microsoft.Devices サービス** を追加し、 **[保存]** をクリックします。

    [![プランへの IoT Hub サービスの追加](../operator/media/iot-hub-rp-install/pd2.png)](../operator/media/iot-hub-rp-install/pd2.png#lightbox)

4. 以上で設定は完了です。 これで IoT ハブを作成できるようになりました。

## <a name="using-iot-hub"></a>IoT ハブの使用

IoT Hub の使用方法については、[Azure IoT Hub のドキュメント](/azure/iot-hub)を参照してください。

## <a name="next-steps"></a>次のステップ

Azure Stack Hub における IoT Hub の管理の詳細については、「[Azure Stack Hub で IoT Hub を管理する方法](iot-hub-rp-manage.md)」を参照してください。