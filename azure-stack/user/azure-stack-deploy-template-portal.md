---
title: Azure Stack Hub でポータルを使用してテンプレートをデプロイする
description: Azure Stack Hub ポータルを使用してテンプレートをデプロイする方法について学習します。
author: mattbriggs
ms.topic: how-to
ms.date: 12/2/2020
ms.author: mabrigg
ms.reviewer: sijuman
ms.lastreviewed: 12/2/2020
ms.openlocfilehash: 7b5508f7c4e1bbd9b21f2160aecb6fd27c9a10e3
ms.sourcegitcommit: 9ef2cdc748cf00cd3c8de90705ea0542e29ada97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/02/2020
ms.locfileid: "96525440"
---
# <a name="deploy-a-template-using-the-portal-in-azure-stack-hub"></a>Azure Stack Hub でポータルを使用してテンプレートをデプロイする

Azure Stack Hub ユーザー ポータルを使用して、Azure Resource Manager テンプレートを Azure Stack Hub にデプロイできます。

## <a name="to-deploy-a-template"></a>テンプレートをデプロイするには

1. Azure Stack Hub ユーザー ポータルにサインインして、 **[+ リソースの作成]**  >  **[カスタム]**  >  **[テンプレートのデプロイ]** を選択します。

   ![Azure Stack Hub ポータルでリソースを作成する](media/azure-stack-deploy-template-portal/template-deploy1a.png)

2. **[Type to start filter]\(入力してフィルター処理を開始\)** を選択して GitHub クイックスタートテンプレートを選択するか、 **[エディターで独自のテンプレートを作成する]** を選択します。

   ![Azure Stack Hub ポータルでテンプレートをデプロイする](media/azure-stack-deploy-template-portal/template-deploy2a.png)

    [**AzureStack-QuickStart-Templates**](https://github.com/Azure/AzureStack-QuickStart-Templates) は、Microsoft ではなく Azure Stack Hub コミュニティのメンバーによって作成されています。 各テンプレートは、Microsoft ではなく、その所有者によるライセンス契約に基づいてお客様に使用許諾されます。 Microsoft は、これらのテンプレートに対して責任を負わず、セキュリティ、互換性、パフォーマンスなどについての調査を行っていません。 コミュニティ テンプレートは、Microsoft サポート プログラムまたはサービスのサポート対象ではなく、"*手を加えず*" に提供され、いかなる保証もありません。

3. **[エディターで独自のテンプレートを作成する]** を選択した場合は、JSON テンプレート コードをコード ウィンドウに貼り付けます。

   ![Azure Stack Hub ポータルでテンプレートを編集する](media/azure-stack-deploy-template-portal/template-deploy3a.png)

    - **[クイックスタート テンプレート]** を選択して、エディターでコミュニティ テンプレートを読み込みます。

    - ローカル コンピューターからエディターに Azure Resource Manager テンプレートを読み込むには、 **[ファイルの読み込み]** を選択します。

    - **[ダウンロード]** を選択して、Azure Resource Manager テンプレートをローカル マシンに保存します。

    テンプレートに変更を加えたら、 **[保存]** を選択します。

4. **[サブスクリプション]** を選択します。 使用するサブスクリプションを選択します。 **[リソース グループ]** を選択します。 既存のリソース グループを選択するか、新しいものを新規作成して、 **[OK]** を選択します。 次に、 **[Review + create]\(確認と作成\)** を選択します。

   ![Azure Stack Hub ポータルでパラメーターを編集する](media/azure-stack-deploy-template-portal/template-deploy4a.png)

5. **［作成］** を選択します

   ![Azure Stack Hub ポータルでサブスクリプションを選択する](media/azure-stack-deploy-template-portal/template-deploy5a.png)

6. ダッシュボードの新しいタイルにより、テンプレート デプロイの進行状況を追跡します。

   ![Azure Stack Hub ポータルでリソース グループを選択する](media/azure-stack-deploy-template-portal/template-deploy6a.png)

   Azure Resource Manager テンプレートを使用して、お使いのアプリケーションのすべてのリソースを、単一の連携した操作でデプロイしてプロビジョニングできます。 テンプレートを再デプロイして、リソース グループ内のリソースを変更することもできます。 Azure Stack Hub でのテンプレートの使用の詳細については、「[Azure Stack Hub で Azure Resource Manager テンプレートを使用する](azure-stack-arm-templates.md)」を参照してください。

## <a name="next-steps"></a>次のステップ

テンプレートのデプロイの詳細については、次の記事を参照してください。

- [PowerShell を使用したテンプレートのデプロイ](azure-stack-deploy-template-powershell.md)
