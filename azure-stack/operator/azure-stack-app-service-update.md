---
title: Azure App Service on Azure Stack Hub を更新する
description: Azure App Service on Azure Stack Hub を更新する方法を説明します。
author: BryanLa
ms.topic: article
ms.date: 05/05/2019
ms.author: anwestg
ms.reviewer: anwestg
ms.lastreviewed: 01/13/2019
zone_pivot_groups: state-connected-disconnected
ms.openlocfilehash: 5dc8599a62c4b9e10d603bf2f02f18bf5024e203
ms.sourcegitcommit: edb60c948b445537e9411d6261c6c78359b71d0b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/07/2020
ms.locfileid: "91810662"
---
# <a name="update-azure-app-service-on-azure-stack-hub"></a>Azure App Service on Azure Stack Hub を更新する

[!INCLUDE [Azure Stack Hub update reminder](../includes/app-service-hub-update-banner.md)]

::: zone pivot="state-connected"
この記事では、インターネットに接続された Azure Stack Hub 環境にデプロイされている [Azure App Service リソース プロバイダー](azure-stack-app-service-overview.md)をアップグレードする方法について説明します。

> [!IMPORTANT]
> アップグレードを実行する前に、[Azure App Service on Azure Stack Hub のデプロイ](azure-stack-app-service-deploy.md)を完了しておく必要があります。 

## <a name="run-the-azure-app-service-resource-provider-installer"></a>Azure App Service リソース プロバイダーのインストーラーを実行する

このプロセス中に、アップグレードでは次のことを行います。

* Azure App Service の以前のデプロイを検出します。
* すべての更新プログラム パッケージとデプロイされるすべての OSS ライブラリの新しいバージョンを準備します。
* ストレージにアップロードします。
* すべての Azure App Service ロール (コントローラー、管理、フロントエンド、パブリッシャー、および worker ロール) をアップグレードする
* Azure App Service スケール セット定義を更新する
* Azure App Service リソース プロバイダー マニフェストを更新する

> [!IMPORTANT]
> Azure App Service インストーラーは、Azure Stack Hub 管理者の Azure Resource Manager エンドポイントにアクセスできるマシン上で実行する必要があります。

Azure App Service on Azure Stack Hub のデプロイをアップグレードするには、次の手順に従います。

1. [Azure App Service インストーラー](https://aka.ms/appsvcupdateQ2installer)をダウンロードします。

2. 管理者として appservice.exe を実行します。

    ![App Service インストーラーでデプロイまたはアップグレードのプロセスを開始する方法を示すスクリーンショット。][1]

3. **[Deploy Azure App Service or upgrade to the latest version]\(Azure App Service をデプロイするか、または最新バージョンにアップグレードする\)** をクリックします。

4. マイクロソフト ソフトウェア ライセンス条項を確認して同意し、 **[次へ]** をクリックします。

5. サード パーティのライセンス条項を確認して同意し、 **[次へ]** をクリックします。

6. Azure Stack Hub Azure Resource Manager エンドポイントと Active Directory テナントの情報が正しいことを確認します。 ASDK のデプロイ中に既定の設定を使用した場合は、ここで既定値をそのまま使用することもできます。 ただし、Azure Stack Hub のデプロイ時にオプションをカスタマイズした場合は、このウィンドウで値を編集する必要があります。 たとえば、ドメイン サフィックス *mycloud.com* を使用する場合は、Azure Stack Hub Azure Resource Manager エンドポイントを *management.region.mycloud.com* に変更する必要があります。 自分の情報を確認したら、 **[次へ]** をクリックします。

    ![App Service インストーラー内の、ARM エンドポイントを構成する場所を示すスクリーンショット。][2]

7. 次のページで、次の操作を行います。

    1. 使用する接続方法 ( **[資格情報]** または **[サービス プリンシパル]** ) を選択します。
        - **資格情報**
            - Azure Active Directory (Azure AD) を使用している場合は、Azure Stack Hub のデプロイ時に指定した、Azure AD の管理者アカウントとパスワードを入力します。 **[接続]** を選択します。
            - Active Directory フェデレーション サービス (AD FS) を使用している場合は、ご自分の管理者アカウントを指定します。 たとえば、「 cloudadmin@azurestack.local 」のように入力します。 パスワードを入力し、 **[接続]** を選択します。
        - **サービス プリンシパル**
            - 使用するサービス プリンシパルには、**既定のプロバイダー サブスクリプション**に対する**所有者**権限が**必要です**
            - **サービス プリンシパル ID**、**証明書ファイル**、および**パスワード**を指定し、 **[接続]** を選択します。

    1. **[Azure Stack Hub Subscriptions]\(Azure Stack Hub サブスクリプション\)** で、 **[Default Provider Subscription]\(既定のプロバイダー サブスクリプション\)** を選択します。    Azure App Service on Azure Stack Hub は、 **[Default Provider Subscription]\(既定のプロバイダー サブスクリプション\)** にデプロイする**必要**があります。

    1. **[Azure Stack Hub Locations]\(Azure Stack Hub の場所\)** で、デプロイしているリージョンに対応する場所を選択します。 たとえば、ASDK にデプロイしている場合は、 **[ローカル]** を選びます。

    1. 既存の Azure App Service デプロイが検出された場合は、リソース グループとストレージ アカウントにデータが入力されており、使用できません。

      ![App Service インストーラー内の、Azure Stack Hub のサブスクリプション情報を指定する場所を示すスクリーンショット。][3]

8. 概要ページで、次のことを行います。
   1. 選択した内容を確認します。 変更を加えるには、 **[前へ]** を使って前のページに戻ります。
   2. 構成が正しい場合は、チェック ボックスをオンにします。
   3. アップグレードを開始するには、 **[次へ]** をクリックします。

       ![App Service インストーラーで App Service のアップグレードの概要を示すスクリーンショット。][4]

9. アップグレードの進行状況ページ:
    1. アップグレードの進行状況を追跡します。 Azure App Service on Azure Stack Hub のアップグレードの期間は、デプロイされるロール インスタンスの数によって異なります。
    2. アップグレードが正常に完了したら、 **[終了]** をクリックします。

        ![App Service インストーラーでデプロイの進行状況を示すスクリーンショット。][5]
::: zone-end

::: zone pivot="state-disconnected"
この記事では、次のような Azure Stack Hub 環境にデプロイされている [Azure App Service リソース プロバイダー](azure-stack-app-service-overview.md)をアップグレードする方法について説明します。

* インターネットに接続されていない
* Active Directory フェデレーション サービス (AD FS) によって保護されている

> [!IMPORTANT]
> アップグレードを実行する前に、[接続されていない環境への Azure App Service on Azure Stack Hub のデプロイ](./azure-stack-app-service-deploy.md?pivots=state-disconnected&view=azs-2002)を完了しておく必要があります。 

## <a name="run-the-app-service-resource-provider-installer"></a>App Service リソースプロバイダーのインストーラーを実行する

Azure Stack Hub 環境内の App Service リソース プロバイダーをアップグレードするには、次のタスクを完了する必要があります。

1. [Azure App Service インストーラー](https://aka.ms/appsvcupdate8installer)をダウンロードします。
2. オフライン アップグレード パッケージを作成する。
3. App Service インストーラー (appservice.exe) を実行し、アップグレードを完了します。

このプロセス中に、アップグレードでは次のことを行います。

* App Service の以前のデプロイを検出する
* ストレージにアップロードする
* すべての App Service ロール (コントローラー、管理、フロントエンド、パブリッシャー、および worker ロール) をアップグレードする
* App Service スケール セット定義を更新する
* App Service リソース プロバイダー マニフェストを更新する

## <a name="create-an-offline-upgrade-package"></a>オフライン アップグレード パッケージを作成する

切断された環境で App Service をアップグレードするには、まずインターネットに接続されたマシン上でオフライン アップグレード パッケージを作成する必要があります。

1. 管理者として appservice.exe を実行する

    ![切断環境でアップグレードを開始する方法を示すスクリーンショット。][11]

2. **[詳細]**  >  **[Create offline package] (オフライン パッケージを作成する)** をクリックします。

    ![App Service インストーラーでオフライン パッケージを作成する方法を示すスクリーンショット。][12]

3. Azure App Service インストーラーによってオフライン アップグレード パッケージが作成され、そのパスが表示されます。  **[フォルダーを開く]** をクリックすると、エクスプローラーでフォルダーが開きます。

4. インストーラー (AppService.exe) とオフライン インストール パッケージを、Azure Stack Hub に接続されているマシンにコピーします。

## <a name="complete-the-upgrade-of-app-service-on-azure-stack-hub"></a>App Service on Azure Stack Hub のアップグレードを完了する

> [!IMPORTANT]
> Azure App Service インストーラーは、Azure Stack Hub Administrator Azure Resource Manager エンドポイントに到達できるマシン上で実行する必要があります。

1. 管理者として appservice.exe を実行します。

    ![アップグレードを開始する方法を示すスクリーンショット。][11]

2. **[詳細]**  >  **[Complete offline installation or upgrade] (オフライン インストールまたはアップグレードを完了する)** をクリックします。

    ![App Service インストーラーでオフラインのインストールまたはアップグレードを実行する方法を示すスクリーンショット。][12]

3. 前に作成したオフライン アップグレード パッケージの場所を参照し、 **[次へ]** をクリックします。

4. マイクロソフト ソフトウェア ライセンス条項を確認して同意し、 **[次へ]** をクリックします。

5. サード パーティのライセンス条項を確認して同意し、 **[次へ]** をクリックします。

6. Azure Stack Hub Azure Resource Manager エンドポイントと Active Directory テナントの情報が正しいことを確認します。 Azure Stack Development Kit の展開中に既定の設定を使用した場合は、ここで既定値を受け入れることができます。 ただし、Azure Stack Hub のデプロイ時にオプションをカスタマイズした場合は、このウィンドウで値を編集する必要があります。 たとえば、ドメイン サフィックス *mycloud.com* を使用する場合は、Azure Stack Hub Azure Resource Manager エンドポイントを *management.region.mycloud.com* に変更する必要があります。 自分の情報を確認したら、 **[次へ]** をクリックします。

    ![インストーラー内の、ARM エンドポイントを構成する場所を示すスクリーンショット。][13]

7. 次のページで、次の操作を行います。

   1. 使用する接続方法 ( **[資格情報]** または **[サービス プリンシパル]** ) を選択します。
        - **資格情報**
            - Azure Active Directory (Azure AD) を使用している場合は、Azure Stack Hub のデプロイ時に指定した、Azure AD の管理者アカウントとパスワードを入力します。 **[接続]** を選択します。
            - Active Directory フェデレーション サービス (AD FS) を使用している場合は、ご自分の管理者アカウントを指定します。 たとえば、「 cloudadmin@azurestack.local 」のように入力します。 パスワードを入力し、 **[接続]** を選択します。
        - **サービス プリンシパル**
            - 使用するサービス プリンシパルには、**既定のプロバイダー サブスクリプション**に対する**所有者**権限が**必要です**
            - **サービス プリンシパル ID**、**証明書ファイル**、および**パスワード**を指定し、 **[接続]** を選択します。

   1. **[Azure Stack Hub Subscriptions]\(Azure Stack Hub サブスクリプション\)** で、 **[Default Provider Subscription]\(既定のプロバイダー サブスクリプション\)** を選択します。  Azure App Service on Azure Stack Hub は、 **[Default Provider Subscription]\(既定のプロバイダー サブスクリプション\)** にデプロイする**必要**があります。

   1. **[Azure Stack Hub Locations]\(Azure Stack Hub の場所\)** で、デプロイしているリージョンに対応する場所を選択します。 たとえば、ASDK にデプロイしている場合は、 **[ローカル]** を選びます。
   
   1. 既存の App Service デプロイが検出された場合は、リソース グループとストレージ アカウントにデータが入力され、灰色表示されます。

      ![インストーラー内の、Azure Stack Hub サブスクリプションを構成する場所を示すスクリーンショット。][14]
8. 概要ページで、次のことを行います。
   1. 選択した内容を確認します。 変更を加えるには、 **[前へ]** を使って前のページに戻ります。
   2. 構成が正しい場合は、チェック ボックスをオンにします。
   3. アップグレードを開始するには、 **[次へ]** をクリックします。

       ![インストーラーで収集された情報の概要を示すスクリーンショット。][15]

9. アップグレードの進行状況ページ:
    1. アップグレードの進行状況を追跡します。 App Service on Azure Stack Hub のアップグレードの期間は、デプロイされるロール インスタンスの数によって異なります。
    2. アップグレードが正常に完了したら、 **[終了]** をクリックします。

        ![アップグレードが正常に完了したことを示すスクリーンショット。][16]
::: zone-end

## <a name="next-steps"></a>次のステップ

Azure App Service on Azure Stack Hub のための追加の管理者操作を準備します。

* [追加容量の計画を立てる](azure-stack-app-service-capacity-planning.md)
* [追加容量を追加する](azure-stack-app-service-add-worker-roles.md)

<!--Image references-->
[1]: ./media/azure-stack-app-service-update/app-service-exe.png
[2]: ./media/azure-stack-app-service-update/app-service-azure-resource-manager-endpoints.png
[3]: ./media/azure-stack-app-service-update/app-service-installation-detected.png
[4]: ./media/azure-stack-app-service-update/app-service-upgrade-summary.png
[5]: ./media/azure-stack-app-service-update/app-service-upgrade-complete.png

[11]: ./media/azure-stack-app-service-update-offline/app-service-exe.png
[12]: ./media/azure-stack-app-service-update-offline/app-service-exe-advanced.png
[13]: ./media/azure-stack-app-service-update-offline/app-service-azure-resource-manager-endpoints.png
[14]: ./media/azure-stack-app-service-update-offline/app-service-installation-detected.png
[15]: ./media/azure-stack-app-service-update-offline/app-service-upgrade-summary.png
[16]: ./media/azure-stack-app-service-update-offline/app-service-upgrade-complete.png
