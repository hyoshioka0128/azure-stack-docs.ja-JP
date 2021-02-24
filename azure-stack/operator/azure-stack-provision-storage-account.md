---
title: Azure Stack Hub でストレージ アカウントを作成する
titleSuffix: Azure Stack Hub
description: Azure Stack Hub でストレージ アカウントを作成する方法について説明します。
author: PatAltimore
ms.topic: how-to
ms.date: 1/22/2020
ms.author: patricka
ms.lastreviewed: 01/18/2019
ms.openlocfilehash: 953ba2c8ee1c5d5950b4f42b0771d5c0976c2d89
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97869544"
---
# <a name="create-storage-accounts-in-azure-stack-hub"></a>Azure Stack Hub でストレージ アカウントを作成する

Azure Stack Hub のストレージ アカウントには、BLOB や Table service、およびストレージ データ オブジェクトの一意の名前空間が含まれます。 既定では、アカウントのデータはストレージ アカウント所有者だけが使用できます。

1. Azure Stack Hub POC コンピューターで、[管理者](../asdk/asdk-connect.md)として `https://adminportal.local.azurestack.external` にログインし、**[+ リソースの作成]** > **[データ + ストレージ]** > **[ストレージ アカウント]** の順にクリックします。

   ![Azure Stack Hub 管理者ポータルでストレージ アカウントを作成する](media/azure-stack-provision-storage-account/image01.png)

2. **[ストレージ アカウントの作成]** ブレードで、ストレージ アカウントの名前を入力します。 新しい **リソース グループ** を作成するか、既存のものを選択し、 **[作成]** をクリックしてストレージ アカウントを作成します。

   ![Azure Stack Hub 管理者ポータルでストレージ アカウントを確認する](media/azure-stack-provision-storage-account/image02.png)

3. 新しいストレージ アカウントを表示するには、 **[すべてのリソース]** をクリックしてストレージ アカウントを検索し、ストレージ アカウント名をクリックします。

    ![Azure Stack Hub 管理者ポータルでのストレージ アカウント名](media/azure-stack-provision-storage-account/image03.png)

### <a name="next-steps"></a>次のステップ

- [Azure Resource Manager テンプレートの使用](../user/azure-stack-arm-templates.md)
- [Azure ストレージ アカウントについて](/azure/storage/common/storage-create-storage-account)
- [Azure Stack Hub Azure 互換ストレージの検証ガイドのダウンロード](https://aka.ms/azurestacktp1doc)
