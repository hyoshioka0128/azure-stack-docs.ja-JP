---
title: SQL リソース プロバイダーの削除
titleSuffix: Azure Stack Hub
description: Azure Stack Hub のデプロイから SQL リソース プロバイダーを削除する方法について説明します。
author: bryanla
ms.topic: article
ms.date: 10/02/2019
ms.author: bryanla
ms.reviewer: xiaofmao
ms.lastreviewed: 11/20/2019
ms.openlocfilehash: b232bf2efb690ddc0a351d762e6866a75ced4a81
ms.sourcegitcommit: 97ecba06aeabf2f30de240ac283b9bb2d49d62f0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/10/2020
ms.locfileid: "97011367"
---
# <a name="remove-the-sql-resource-provider"></a>SQL リソース プロバイダーの削除

SQL リソース プロバイダーを削除する前に、プロバイダーの依存関係をすべて削除する必要があります。 また、リソース プロバイダーのインストールに使用したデプロイ パッケージのコピーも必要になります。

> [!NOTE]
> リソース プロバイダーのインストーラーのダウンロード リンクは、[リソース プロバイダーを展開するための前提条件](./azure-stack-sql-resource-provider-deploy.md#prerequisites)に関するページにあります。

SQL リソース プロバイダーを削除すると、オペレーターが管理する関連プランとクォータが削除されます。 ただし、ホスティング サーバーからテナント データベースは削除されません。

## <a name="to-remove-the-sql-resource-provider"></a>SQL リソース プロバイダーを削除するには

1. 既存の SQL リソース プロバイダーの依存関係がすべて削除されていることを確認します。

   > [!NOTE]
   > 依存リソースがリソース プロバイダーを現在使用している場合でも、SQL リソース プロバイダーのアンインストールは続行されます。
  
2. SQL リソース プロバイダーのインストール パッケージのコピーを入手し、自己展開形式ファイルを実行してコンテンツを一時ディレクトリに展開します。

3. 新しい管理者特権の PowerShell コンソール ウィンドウを開き、SQL リソース プロバイダーのインストール ファイルを抽出したディレクトリに変更します。

4. 次のパラメーターを使用して、DeploySqlProvider.ps1 スクリプトを実行します。

    * **Uninstall**:リソース プロバイダーと関連付けられているすべてのリソースを削除します。
    * **PrivilegedEndpoint**:特権エンドポイントの IP アドレスまたは DNS 名。
    * **AzureEnvironment**:Azure Stack Hub のデプロイに使用する Azure 環境。 Azure AD のデプロイでのみ必須です。
    * **CloudAdminCredential**:特権エンドポイントへのアクセスに必要な、クラウド管理者の資格情報。
    * **AzCredential**:Azure Stack Hub サービス管理者アカウントの資格情報。 Azure Stack Hub のデプロイに使用したのと同じ資格情報を使用します。 AzCredential で使用するアカウントが多要素認証 (MFA) を必要とする場合、スクリプトは失敗します。

## <a name="next-steps"></a>次のステップ

[App Services を PaaS として提供する](azure-stack-app-service-overview.md)
