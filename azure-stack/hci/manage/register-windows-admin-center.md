---
title: Windows Admin Center を Azure に登録する
description: Windows Admin Center ゲートウェイを Azure に登録する方法。
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.date: 01/25/2021
ms.openlocfilehash: c2067387d7b03252e7a2cc93aeddcb68dbd4bc83
ms.sourcegitcommit: 2ac64ac431411b673e655465939d3c95cc94c55d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/26/2021
ms.locfileid: "98811019"
---
# <a name="register-windows-admin-center-with-azure"></a>Windows Admin Center を Azure に登録する

> 適用対象: Azure Stack HCI v20H2、Windows Server 2019

Windows Admin Center で Azure サービスを使用するには、まず管理 PC に Windows Admin Center をインストールし、Windows Admin Center ゲートウェイの 1 回限りの登録を完了する必要があります。 これは、クラスターを Azure に登録するための前提条件であり、[クラスター登録](../deploy/register-with-azure.md)プロセスに使用する予定のものと同じ管理 PC で行う必要があります。

## <a name="complete-the-gateway-registration-process-using-windows-admin-center"></a>Windows Admin Center を使用してゲートウェイ登録プロセスを完了する

1. Windows Admin Center を起動し、右上にある **[設定]** 歯車アイコンをクリックします。これにより、[アカウント] ページに移動します。 次に、左側の **[ゲートウェイ]** メニューで **[Azure]** を選択し、 **[登録]** をクリックします。

   :::image type="content" source="media/register-wac/register-wac.png" alt-text="[設定] > [ゲートウェイ] > [Azure] を選択した後、[登録] をクリックする" lightbox="media/register-wac/register-wac.png":::

2. 一意のコードが表示されます。 コードの右にある **[コピー]** ボタンをクリックします。 **[コードの入力]** をクリックすると、別のブラウザー ウィンドウが開き、アプリまたはデバイスに表示されているコードを貼り付けることができます。

   :::image type="content" source="media/register-wac/enter-code.png" alt-text="一意のコードをコピーし、[コードの入力] をクリックして、ダイアログ ボックスに貼り付ける" lightbox="media/register-wac/enter-code.png":::

3. コードを貼り付けると、リモート デバイスまたはサービスで Windows Admin Center にサインインしようとしていることを示す通知が表示されます。 メール アドレスまたは電話番号を入力します。 デバイスが管理されている場合は、認証のために組織のサインイン ページが表示されます。 指示に従って、適切な資格情報を入力します。

   :::image type="content" source="media/register-wac/sign-in.png" alt-text="メール アドレスまたは電話番号を使用して Windows Admin Center にサインインする" lightbox="media/register-wac/sign-in.png":::

   次のメッセージが表示されます。"デバイスの Windows Admin Center アプリケーションにサインインしました。" ブラウザー ウィンドウを閉じて、元の登録ページに戻ります。

4. Azure Active Directory (テナント) ID を入力して、Azure Active Directory に接続します。 前記の手順に従った場合、ID フィールドには値があらかじめ設定されています。 組織から既存の ID を提供されていない場合は、 **[Azure Active Directory アプリケーション]** の設定を **[新規作成]** にままにしておきます。 ID を既に持っている場合は、 **[既存のものを使用]** をクリックすると、管理者によって提供された ID を入力するための空のフィールドが表示されます。ID を入力すると、その ID のアカウントが見つかることが Windows Admin Center によって確認されます。 既存の ID はあるものの、それが何かわからない場合は、[こちらの手順](/azure/active-directory/develop/howto-create-service-principal-portal#get-values-for-signing-in)に従って取得します。

   :::image type="content" source="media/register-wac/connect-to-aad.png" alt-text="既存の Azure Active Directory (テナント) ID を入力するか、新しいものを作成して、Azure Active Directory に接続する" lightbox="media/register-wac/connect-to-aad.png":::

5. **[接続]** ボタンをクリックして Azure に接続します。 Azure AD に接続されているという確認が表示されます。

6. Azure portal で **[アプリのアクセス許可]** に移動して、Azure でのアクセス許可を付与します。 **[同意する]** で、 **[管理者の同意を与えます]** を選択します。

7. ウィンドウを閉じ、Azure アカウントを使用して Windows Admin Center にサインインします。

## <a name="next-steps"></a>次のステップ

これで以下の準備が整いました。

- [Azure Stack HCI を Azure に接続する](../deploy/register-with-azure.md)
- [Windows Admin Center で Azure Stack HCI を使用する](../get-started.md)
- [Azure ハイブリッド サービスに接続する](/windows-server/manage/windows-admin-center/azure/)