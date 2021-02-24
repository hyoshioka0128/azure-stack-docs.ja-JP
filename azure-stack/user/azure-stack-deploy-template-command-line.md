---
title: Azure Stack Hub でコマンド ラインを使ってテンプレートをデプロイする
description: Azure クロス プラットフォームのコマンド ライン インターフェイス (CLI) を使用して、Azure Stack Hub にテンプレートをデプロイする方法を学習します。
author: mattbriggs
ms.topic: article
ms.date: 2/1/2021
ms.author: mabrigg
ms.reviewer: unknown
ms.lastreviewed: 05/09/2019
ms.openlocfilehash: 38327d2e23fd74c1a5e0a197ccc2a32b65b3fa7a
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99247814"
---
# <a name="deploy-a-template-with-the-command-line-in-azure-stack-hub"></a>Azure Stack Hub でコマンド ラインを使ってテンプレートをデプロイする

Azure コマンド ライン インターフェイス (CLI) を使用して、Azure Resource Manager テンプレートを Azure Stack Hub にデプロイできます。 Azure Resource Manager テンプレートでは、単一の調整されたアクションで、アプリのためのリソースがデプロイされて設定されます。

## <a name="deploy-template"></a>テンプレートのデプロイ

1. [AzureStack-QuickStart-Templates リポジトリ](https://aka.ms/AzureStackGitHub)を参照して、**101-create-storage-account** テンプレートを見つけます。 テンプレート (`azuredeploy.json`) とパラメーター ファイル (`(azuredeploy.parameters.json`) を、ローカル ドライブ上の場所に保存します (`C:\templates\` など)
2. ファイルをダウンロードしたフォルダーに移動します。 
3. [インストールして Azure CLI で Azure Stack Hub に接続](azure-stack-version-profiles-azurecli2.md)します。
4. 次のコマンドで、リージョンと場所を更新します。 ASDK を使用している場合は、場所パラメーターに `local` を使用します。 テンプレートをデプロイするには、次の手順に従います。
    ```azurecli
    az group create --name testDeploy --location local
    az group deployment create --resource-group testDeploy --template-file ./azuredeploy.json --parameters ./azuredeploy.parameters.json
    ```

このコマンドを使うと、Azure Stack Hub インスタンス内のリソース グループ **testDeploy** にテンプレートがデプロイされます。

## <a name="validate-template-deployment"></a>テンプレートのデプロイを検証する

リソース グループとストレージ アカウントを確認するには、次の CLI コマンドを実行します。

```azurecli
az group list
az storage account list
```

## <a name="next-steps"></a>次のステップ

[PowerShell を使用したテンプレートのデプロイ](azure-stack-deploy-template-powershell.md)方法を確認します。
