---
title: Integrated Dell Remote Access Controller の資格情報
description: Integrated Dell Remote Access Controller の資格情報を更新する方法について説明します
author: troettinger
ms.author: thoroet
ms.service: azure-stack
ms.topic: article
ms.date: 04/28/2020
ms.reviewer: justinha
ms.lastreviewed: 04/28/2020
ms.openlocfilehash: 2c6e7f0aa0e6e03af061083259234e49275a83de
ms.sourcegitcommit: 50b362d531c2d35a3a935811fee71252971bd5d8
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/09/2020
ms.locfileid: "96941355"
---
# <a name="update-credentials-for-the-integrated-dell-remote-access-controller"></a>Integrated Dell Remote Access Controller の資格情報を更新する

このセクションでは、Integrated Dell Remote Access Controller (iDRAC) の資格情報を変更する方法について説明します。 

## <a name="prerequisites"></a>前提条件

手順を実行する前に: 

- リモート デスクトップを使用して MGMT 仮想マシンに接続します。 
- アカウント用の新しい資格情報があることを確認します。 
 
## <a name="update-the-idrac-credentials"></a>iDRAC 資格情報の更新

環境内のすべての PowerEdge サーバー (HLH およびスケール ユニット ノード) に対する iDRAC 資格情報を更新するには:

1. Web ブラウザーで、 https://<iDRAC_IP> にログインします。 
1. **[iDRAC 設定]**  >  **[ユーザー]** に移動します。 
1. 変更するユーザーを選択し、 **[編集]** をクリックします。 
1. 次の図に示すように、 **[ユーザー の編集]** ウィンドウで、 **[パスワード]** と **[パスワードの確認入力]** に新しいパスワードを入力します。 

   ![ユーザー情報を示すスクリーンショット](../operator/media/idrac-credentials/enter-user.png)

1. **[保存]** をクリックし、 **[OK]** をクリックします。 

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub でシークレットをローテーションする](../../operator/azure-stack-rotate-secrets.md)