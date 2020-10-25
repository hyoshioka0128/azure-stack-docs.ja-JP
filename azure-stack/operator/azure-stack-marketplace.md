---
title: Azure Stack Hub Marketplace の概要
description: Azure Stack Hub Marketplace と Marketplace アイテムの概要。
author: sethmanheim
ms.topic: article
ms.date: 07/27/2020
ms.author: sethm
ms.reviewer: ihcherie
ms.lastreviewed: 05/07/2019
ms.openlocfilehash: b9f01cb8237a3a6764b6ce450e279700a17eea01
ms.sourcegitcommit: b2337a9309c52aac9f5a1ffd89f1426d6c178ad5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/28/2020
ms.locfileid: "87250811"
---
# <a name="azure-stack-hub-marketplace-overview"></a>Azure Stack Hub Marketplace の概要

Azure Stack Hub Marketplace は、Azure Stack Hub 向けにカスタマイズされたサービス、アプリ、およびリソースのコレクションです。 リソースには、ネットワーク、仮想マシン (VM)、ストレージなどが含まれます。 Azure Stack Hub Marketplace を使用して、新しいリソースを作成したり、新しいアプリをデプロイしたり、使用するアイテムを参照して選択したりします。 ユーザーが Marketplace アイテムを使用するには、アイテムへのアクセスが許可されるオファーにサブスクライブする必要があります。

Azure Stack Hub オペレーターが、Azure Stack Hub Marketplace に追加 (公開) するアイテムを決定します。 データベース、アプリ サービスなどのアイテムを公開できます。 公開すると、アイテムはすべてのユーザーに表示されます。 作成したカスタム アイテムを公開することも、増え続けている [Azure Marketplace アイテムの一覧](azure-stack-marketplace-azure-items.md)からアイテムを公開することもできます。 Azure Stack Hub Marketplace にアイテムを公開すると、5 分以内にユーザーがそれを表示できます。

> [!CAUTION]  
> イメージや JSON ファイルを含むギャラリー アイテム アーティファクトはすべて、Azure Stack Hub Marketplace で使用可能にした後、認証なしでアクセスできます。 Marketplace アイテムを発行するときの考慮事項については、「[Marketplace アイテムを作成および発行する](azure-stack-create-and-publish-marketplace-item.md)」を参照してください。

Marketplace を開くには、管理者ポータルで **[+ リソースの作成]** を選択します。

![Azure Stack Hub 管理者ポータルでリソースを作成する](media/azure-stack-marketplace/marketplace1.png)

## <a name="marketplace-items"></a>Marketplace アイテム

Azure Stack Hub Marketplace のアイテムは、ユーザーがダウンロードおよび使用できるサービス、アプリ、またはリソースです。 すべてのユーザーに、プランやオファーなどの管理アイテムを含むすべての Azure Stack Hub Marketplace アイテムが表示されます。 これらの管理アイテムは、表示するためのサブスクリプションを必要としませんが、ユーザーは使用することができません。

各 Marketplace アイテムには以下のものが含まれます。

* リソース プロビジョニングのための Azure Resource Manager テンプレート。
* 文字列、アイコン、その他のマーケティング資料などのメタデータ。
* ポータルでアイテムを表示するための書式設定情報。

Azure Stack Hub Marketplace に発行されるすべてのアイテムでは、Azure ギャラリー パッケージ (.azpkg) 形式を使用します。 デプロイまたはランタイム リソース (コード、ソフトウェアの .zip ファイル、または VM イメージ) を Marketplace アイテムの一部としてではなく、Azure Stack Hub に個別に追加します。

バージョン 1803 以降の Azure Stack Hub では、Azure からの画像ダウンロード時やカスタム画像のアップロード時に、画像がスパース ファイルに変換されます。 このプロセスは画像を追加するときに時間がかかりますが、領域を節約し、画像の展開時間を短縮することができます。 変換は新しい画像にのみ適用されます。 既存の画像は変更されません。

## <a name="next-steps"></a>次のステップ

* [Azure から既存の Marketplace の項目をダウンロードして Azure Stack Hub に発行する](azure-stack-download-azure-marketplace-item.md)  
* [Azure Stack Hub Marketplace のカスタム アイテムを作成して発行する](azure-stack-create-and-publish-marketplace-item.md)
