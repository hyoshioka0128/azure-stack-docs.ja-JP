---
title: App Services のデプロイ ソースを構成する - Azure Stack Hub
description: Azure Stack Hub 上の App Services のデプロイ ソース (Git、GitHub、BitBucket、DropBox、および OneDrive) を構成する方法について説明します。
author: bryanla
ms.topic: article
ms.date: 03/11/2019
ms.author: anwestg
ms.reviewer: anwestg
ms.lastreviewed: 10/15/2019
ms.openlocfilehash: 2766c495087d32dba1162d2354b9b0a8ea673afb
ms.sourcegitcommit: 3e2460d773332622daff09a09398b95ae9fb4188
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/15/2020
ms.locfileid: "90573260"
---
# <a name="configure-deployment-sources-for-app-services-on-azure-stack-hub"></a>Azure Stack Hub 上の App Services のデプロイ ソースを構成する

App Service on Azure Stack Hub では、複数のソース管理プロバイダーからのオンデマンド デプロイがサポートされます。 この機能を使用して、アプリ開発者はソース管理リポジトリから直接デプロイできます。 App Service をリポジトリに接続するように構成するには、まず、クラウド オペレーターが App Service on Azure Stack Hub とソース管理プロバイダーの統合を構成する必要があります。  

ローカル Git の他に、次のソース管理プロバイダーがサポートされます。

* GitHub
* BitBucket
* OneDrive
* DropBox

## <a name="view-deployment-sources-in-app-service-administration"></a>App Service 管理でデプロイ ソースを表示する

1. サービス管理者として Azure Stack Hub 管理者ポータルにサインインします。
2. **[All Services (すべてのサービス)]** を参照し、 **[App Service]** を選択します。

    ![App Service リソース プロバイダー管理][1]

3. **[ソース管理の構成]** を選択します。 構成されているすべてのデプロイ ソースの一覧が表示されます。

    ![App Service リソース プロバイダー管理のソース管理の構成][2]

## <a name="configure-github"></a>GitHub を構成する

このタスクを完了するには、GitHub アカウントが必要です。 個人のアカウントではなく、組織のアカウントを使用できます。

1. GitHub にサインインし、 <https://www.github.com/settings/developers> に移動して、 **[Register a new application]\(新しいアプリケーションの登録\)** を選択します。

    ![GitHub - 新しいアプリケーションの登録][3]

2. **アプリケーション名**を入力します。 たとえば、**App Service on Azure Stack Hub**  です。
3. **[Homepage URL (ホームページの URL)]** を入力します。 ホームページの URL は、Azure Stack Hub ポータルのアドレスである必要があります。 たとえば、「 `https://portal.<region>.<FQDN>` 」のように入力します。 Azure Stack Hub の完全修飾ドメイン名 (FQDN) の詳細については、「[Azure Stack Hub の DNS 名前空間](azure-stack-integrate-dns.md#azure-stack-hub-dns-namespace)」を参照してください。
4. **[アプリケーションの説明]** .を入力します。
5. **[Authorization callback URL (認証コールバック エンドポイント URL)]** を入力します。 既定の Azure Stack Hub デプロイでは、URL は `https://portal.<region>.<FQDN>/TokenAuthorize` という形式になります。 
6. **[アプリケーションの登録]** を選択します。 アプリの **[クライアント ID]** と **[クライアント シークレット]** を示すページが表示されます。

    ![GitHub - 完了したアプリケーション登録][5]

7. 新しいブラウザー タブまたはウィンドウで、Azure Stack Hub 管理者ポータルにサービス管理者としてサインインします。
8. **[リソースプロバイダー]** に移動し、 **[App Service Resource Provider Admin]\(App Service リソース プロバイダー管理\)** を選択します。
9. **[ソース管理の構成]** を選択します。
10. **クライアント ID** と**クライアント シークレット**をコピーして、GitHub 用の対応する入力ボックスに貼り付けます。
11. **[保存]** を選択します。

## <a name="configure-bitbucket"></a>BitBucket を構成する

このタスクを完了するには、BitBucket アカウントが必要です。 個人のアカウントではなく、組織のアカウントを使用できます。

1. BitBucket にサインインし、アカウントの下の **[Integrations (統合)]** に移動します。

    ![BitBucket ダッシュボード - 統合][7]

2. [アクセス管理] で **[OAuth]** を選択し、 **[Add consumer]\(コンシューマーの追加\)** を選択します。

    ![BitBucket への OAuth コンシューマーの追加][8]

3. コンシューマーの**名前**を入力します。 たとえば、**App Service on Azure Stack Hub**  です。
4. アプリの**説明**を入力します。
5. **[Callback URL (コールバック URL)]** を入力します。 既定の Azure Stack Hub デプロイでは、コールバック URL は `https://portal.<region>.<FQDN>/TokenAuthorize` という形式になります。 BitBucket の統合を成功させるには、URL の大文字と小文字をここに示されているとおりにする必要があります。
6. **[URL]** を入力します。 この URL は、Azure Stack Hub ポータルの URL にする必要があります。 たとえば、「 `https://portal.<region>.<FQDN>` 」のように入力します。
7. 必要な **[Permissions]\(権限\)** を次のように選択します。

    - **リポジトリ**: *読み取り*
    - **Webhooks**: *読み取りと書き込み*

8. **[保存]** を選択します。 これで、この新しいアプリが **[OAuth consumers]\(OAuth コンシューマー\)** の下に **[キー]** と **[シークレット]** と共に表示されます。

    ![BitBucket アプリケーション リスト][9]

9. 新しいブラウザー タブまたはウィンドウで、Azure Stack Hub 管理者ポータルにサービス管理者としてサインインします。
10. **[リソースプロバイダー]** に移動し、 **[App Service Resource Provider Admin]\(App Service リソース プロバイダー管理\)** を選択します。
11. **[ソース管理の構成]** を選択します。
12. **キー**をコピーして BitBucket 用の **[クライアント ID]** 入力ボックスに、**シークレット**をコピーして BitBucket 用の **[クライアント シークレット]** 入力ボックスに貼り付けます。
13. **[保存]** を選択します。

## <a name="configure-onedrive"></a>OneDrive を構成する

このタスクを完了するには、OneDrive アカウントにリンクされている Microsoft アカウントが必要です。 個人のアカウントではなく、組織のアカウントを使用できます。

> [!NOTE]
> 現時点では、OneDrive for Business アカウントはサポートされていません。

1. <https://apps.dev.microsoft.com/?referrer=https%3A%2F%2Fdev.onedrive.com%2Fapp-registration.htm> に移動し、Microsoft アカウントを使用してサインインします。
2. **[マイ アプリケーション]** で **[アプリの追加]** を選択します。

    ![OneDrive アプリケーション][10]

3. 新しく登録するアプリの **[名前]** を入力します。「**App Service on Azure Stack Hub**」と入力し、 **[アプリケーションの作成]** を選択します。
4. 次の画面に、新しいアプリのプロパティが表示されます。 **[アプリケーション ID]** を一時的な場所に保存します。

    ![OneDrive アプリケーションのプロパティ][11]

5. **[アプリケーション シークレット]** で **[新しいパスワードを生成]** を選択します。 **新しく生成されたパスワード**をメモします。 このパスワードがお客様のアプリケーション シークレットになります。 **[OK]** を選択した後で、もう一度取得することはできません。
6. **[プラットフォーム]** で **[プラットフォームの追加]** を選択し、 **[Web]** を選択します。
7. **[リダイレクト URI]** を入力します。 既定の Azure Stack Hub デプロイでは、リダイレクト URI は `https://portal.<region>.<FQDN>/TokenAuthorize` という形式になります。

    ![OneDrive アプリケーション - Web プラットフォームの追加][12]

8. **[Microsoft Graph のアクセス許可]**  -  **[委任されたアクセス許可]** を追加します。

    - **Files.ReadWrite.AppFolder**
    - **User.Read** ![OneDrive アプリケーション - Graph のアクセス許可][13]

9. **[保存]** を選択します。
10. 新しいブラウザー タブまたはウィンドウで、Azure Stack Hub 管理者ポータルにサービス管理者としてサインインします。
11. **[リソースプロバイダー]** に移動し、 **[App Service Resource Provider Admin]\(App Service リソース プロバイダー管理\)** を選択します。
12. **[ソース管理の構成]** を選択します。
13. **アプリケーション ID** をコピーして OneDrive 用の **[クライアント ID]** 入力ボックスに、**パスワード**をコピーして OneDrive 用の **[クライアント シークレット]** 入力ボックスに貼り付けます。
14. **[保存]** を選択します。

## <a name="configure-dropbox"></a>DropBox を構成する

> [!NOTE]
> このタスクを完了するには、DropBox アカウントが必要です。 個人のアカウントではなく、組織のアカウントを使用できます。

1. <https://www.dropbox.com/developers/apps> に移動し、DropBox アカウントの資格情報を使用してサインインします。
2. **[Create app]\(アプリの作成\)** を選択します。

    ![Dropbox アプリ][14]

3. **[Dropbox API]** を選択します。
4. アクセス レベルを **[App folder]\(アプリ フォルダー\)** に設定します。
5. アプリの**名前**を入力します。

    ![Dropbox アプリケーションの登録][15]

6. **[Create app]\(アプリの作成\)** を選択します。 **[アプリ キー]** と **[アプリケーション シークレット]** を含む、アプリの設定を一覧表示するページが表示されます。
7. **[App folder name]\(アプリ フォルダー名\)** が **[App Service on Azure Stack Hub]** に設定されていることを確認します。
8. **[OAuth 2 Redirect URI]\(OAuth 2 リダイレクト URI\)** を設定し、 **[Add]\(追加\)** を選択します。 既定の Azure Stack Hub デプロイでは、リダイレクト URI は `https://portal.<region>.<FQDN>/TokenAuthorize` という形式になります。

    ![Dropbox アプリケーションの構成][16]

9. 新しいブラウザー タブまたはウィンドウで、Azure Stack Hub 管理者ポータルにサービス管理者としてサインインします。
10. **[リソースプロバイダー]** に移動し、 **[App Service Resource Provider Admin]\(App Service リソース プロバイダー管理\)** を選択します。
11. **[ソース管理の構成]** を選択します。
12. **アプリケーション キー**をコピーして DropBox 用の **[クライアント ID]** 入力ボックスに、**アプリ シークレット**をコピーして DropBox 用の **[クライアント シークレット]** 入力ボックスに貼り付けます。
13. **[保存]** を選択します。

## <a name="next-steps"></a>次のステップ

[継続的配置](/azure/app-service/deploy-continuous-deployment)、[ローカルの Git デプロイ](/azure/app-service/deploy-local-git)、[クラウド フォルダーの同期](/azure/app-service/deploy-content-sync)などでデプロイ ソースを使用できるようになりました。

<!--Image references-->
[1]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin.png
[2]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-source-control-configuration.png
[3]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-github-developer-applications.png
[4]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-github-register-a-new-oauth-application-populated.png
[5]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-github-register-a-new-oauth-application-complete.png
[6]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-roles-management-server-repair-all.png
[7]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-bitbucket-dashboard.png
[8]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-bitbucket-access-management-add-oauth-consumer.png
[9]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-bitbucket-access-management-add-oauth-consumer-complete.png
[10]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-Onedrive-applications.png
[11]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-Onedrive-application-registration.png
[12]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-Onedrive-application-platform.png
[13]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-Onedrive-application-graph-permissions.png
[14]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-Dropbox-applications.png
[15]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-Dropbox-application-registration.png
[16]: ./media/azure-stack-app-service-configure-deployment-sources/App-service-provider-admin-Dropbox-application-configuration.png
