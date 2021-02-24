---
title: App Service on Azure Stack Hub 2020 Q3 のリリース ノート
description: Azure App Service on Azure Stack Hub の 2020 Q3 リリースの内容、既知の問題、および更新プログラムをダウンロードする場所について説明します。
author: apwestgarth
manager: stefsch
ms.topic: article
ms.date: 11/17/2020
ms.author: anwestg
ms.reviewer: anwestg
ms.lastreviewed: 10/23/2020
ms.openlocfilehash: f9d575a91c55379326beaac2d76b3400172c8dd0
ms.sourcegitcommit: 2562b86f47db20e2652d4636227afb9cfd0e03ae
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/18/2020
ms.locfileid: "94785874"
---
# <a name="app-service-on-azure-stack-hub-2020-q3-release-notes"></a>App Service on Azure Stack Hub 2020 Q3 のリリース ノート

これらのリリース ノートでは、Azure App Service on Azure Stack Hub 2020 Q3 における機能強化と修正点、および既知の問題について説明します。 既知の問題は、デプロイおよび更新プロセスに直接関係する問題と、ビルド (インストール後) に関する問題に分けられています。

[!INCLUDE [Azure Stack Hub update reminder](../includes/app-service-hub-update-banner.md)]

## <a name="build-reference"></a>ビルドのリファレンス

App Service on Azure Stack Hub 2020 Q3 のビルド番号は **89.0.2.15** です

## <a name="prerequisites"></a>前提条件

デプロイを開始する前に、[前提条件に関するドキュメント](azure-stack-app-service-before-you-get-started.md)をご覧ください。

Azure App Service on Azure Stack Hub を 2020 Q3 にアップグレードを開始する前に:

- Azure Stack Hub 管理ポータルの Azure App Service の管理で確実にすべてのロールの準備ができているようにします。

- Azure Stack Hub 管理ポータルの Azure App Service 管理を使用して、App Service のシークレットをバックアップします

- App Service とマスター データベースをバックアップします。
  - AppService_Hosting
  - AppService_Metering
  - Master

- テナント アプリのコンテンツ ファイル共有をバックアップします。

  > [!Important]
  > ファイル サーバーと SQL Server の保守と操作を担当するのは、クラウド オペレーターです。  リソース プロバイダーは、これらのリソースの管理は行いません。  クラウドオ ペレーターが、App Service データベースとテナント コンテンツ ファイル共有のバックアップを行います。

- Marketplace から **カスタム スクリプト拡張機能** バージョン **1.9.3** を配信します

## <a name="updates"></a>更新プログラム

Azure App Service on Azure Stack Hub Update Q3 には、次の機能強化と修正が含まれています。

- **App Service のテナント ポータル、管理ポータル、Functions ポータル、Kudu ツール** の更新。 Azure Stack Portal SDK バージョンと一致しています。

- Web アプリと関数アプリの全画面作成エクスペリエンスの追加

- Web Apps と一貫性のある新しい Azure Functions ポータル エクスペリエンス

- **Azure Functions ランタイム** が **v1.0.13154** に更新されました。

- 信頼性を高めるためのコア サービスと、一般的な問題を簡単に診断できるようにするエラー メッセージの更新。

- **次のアプリケーション フレームワークとツールの更新**:
  - ASP.NET Core 2.1.22
  - ASP.NET Core 2.2.14
  - ASP.NET Core 3.1.8
  - ASP.NET Core Module v2 13.1.19331.0
  - Azul OpenJDK
    - 8.42.0.23
    - 8.44.0.11
    - 11.35.15
    - 11.37.17
  - Curl 7.55.1
  - Git for Windows 2.28.0.1
  - MSDeploy 3.5.90702.36
  - NodeJS
    - 14.10.1
  - NPM
    - 6.14.8
  - PHP 7.4.5
  - Tomcat
    - 8.5.47
    - 8.5.51
    - 9.0.273
    - 9.0.31
  - Kudu を 90.21005.4823 に更新

- **すべてのロールの基になっているオペレーティング システムの更新プログラム**:
  - [x64 ベース システム用 Windows Server 2016 に対する 2020-10 累積的な更新プログラム (KB4580346)](https://support.microsoft.com/help/4580346)
  - [x64 ベース システム用 Windows Server 2016 に対する 2020-09 サービス スタック更新プログラム (KB4576750)](https://support.microsoft.com/help/4576750)
  - Defender 定義 1.325.755.0

- **デプロイとアップグレードの一部として、Windows Server の累積的な更新プログラムがコントローラー ロールに適用されるようになりました**

## <a name="issues-fixed-in-this-release"></a>このリリースで修正された問題

- テナントにおいて、テナント ポータルの App Service プラン ビューで新規を使用して App Service プランを作成できるようになりました

- テナントのテナント ポータルでアプリケーションの証明書を管理できます

- Functions の監視で、TLS 1.2 を適用するストレージ エンドポイントからデータを取得できるようになりました

- デプロイとアップグレードの信頼性を向上させるため、インストール時の Management Servers 待機ステップがクラウド デプロイ ステップの外に移動されました

- クリーンアップ ロジックでエラーが発生した後、ワーカー ランタイムのログ ファイル フォルダーのサイズがクォータ制限に違反しているために、ワーカーが正常性チェックの実行を完了できない問題。  クリーンアップ ロジックは、この更新プログラムで修正されています。

## <a name="pre-update-steps"></a>更新前の手順

[更新の既知の問題](#known-issues-update)を確認し、規定されている措置を講じます。

## <a name="post-deployment-steps"></a>デプロイ後の手順

> [!IMPORTANT]
> SQL Always On インスタンスを使用して App Service リソース プロバイダーを提供している場合は、データベースのフェールオーバーが発生したときにサービスが失われないように、[appservice_hosting と appservice_metering データベースを可用性グループに追加](/sql/database-engine/availability-groups/windows/availability-group-add-a-database)し、それらのデータベースを同期する必要があります。

## <a name="known-issues-update"></a>既知の問題 (更新プログラム)

- お客様が appservice_hosting および appservice_metering データベースを包含データベースに変換した場合、ログインが包含ユーザーに正常に移行されていないと、アップグレードが失敗することがあります

デプロイ後に Appservice_hosting および appservice_metering データベースを包含データベースに変換し、データベース ログインを包含ユーザーに正常に移行できていない場合、アップグレード エラーが発生する可能性があります。  

お客様は、Azure App Service on Azure Stack Hub インストールを 2020 Q3 にアップグレードする前に、appservice_hosting と appservice_metering をホストしている SQL Server に対して次のスクリプトを実行する必要があります。  **このスクリプトは非破壊的であり、ダウンタイムが発生することはありません。**

このスクリプトは、次の条件下で実行する必要があります

- システム管理者特権を持つユーザー (SQL SA アカウントなど) が実行する
- SQL Always On を使用する場合は、次の形式の App Service ログインをすべて含む SQL インスタンスからスクリプトを実行する。

    - appservice_hosting_FileServer
    - appservice_hosting_HostingAdmin
    - appservice_hosting_LoadBalancer
    - appservice_hosting_Operations
    - appservice_hosting_Publisher
    - appservice_hosting_SecurePublisher
    - appservice_hosting_WebWorkerManager
    - appservice_metering_Common
    - appservice_metering_Operations
    - WebWorker_<instance ip address> の形式のすべての WebWorker ログイン

```sql
        USE appservice_hosting
        IF EXISTS(SELECT * FROM sys.databases WHERE Name=DB_NAME() AND containment = 1)
        BEGIN
        DECLARE @username sysname ;  
        DECLARE user_cursor CURSOR  
        FOR
            SELECT dp.name
            FROM sys.database_principals AS dp  
            JOIN sys.server_principals AS sp
                ON dp.sid = sp.sid  
                WHERE dp.authentication_type = 1 AND dp.name NOT IN ('dbo','sys','guest','INFORMATION_SCHEMA');
            OPEN user_cursor  
            FETCH NEXT FROM user_cursor INTO @username  
                WHILE @@FETCH_STATUS = 0  
                BEGIN  
                    EXECUTE sp_migrate_user_to_contained
                    @username = @username,  
                    @rename = N'copy_login_name',  
                    @disablelogin = N'do_not_disable_login';  
                FETCH NEXT FROM user_cursor INTO @username  
            END  
            CLOSE user_cursor ;  
            DEALLOCATE user_cursor ;
            END
        GO

        USE appservice_metering
        IF EXISTS(SELECT * FROM sys.databases WHERE Name=DB_NAME() AND containment = 1)
        BEGIN
        DECLARE @username sysname ;  
        DECLARE user_cursor CURSOR  
        FOR
            SELECT dp.name
            FROM sys.database_principals AS dp  
            JOIN sys.server_principals AS sp
                ON dp.sid = sp.sid  
                WHERE dp.authentication_type = 1 AND dp.name NOT IN ('dbo','sys','guest','INFORMATION_SCHEMA');
            OPEN user_cursor  
            FETCH NEXT FROM user_cursor INTO @username  
                WHILE @@FETCH_STATUS = 0  
                BEGIN  
                    EXECUTE sp_migrate_user_to_contained
                    @username = @username,  
                    @rename = N'copy_login_name',  
                    @disablelogin = N'do_not_disable_login';  
                FETCH NEXT FROM user_cursor INTO @username  
            END  
            CLOSE user_cursor ;  
            DEALLOCATE user_cursor ;
            END
        GO
```

## <a name="known-issues-post-installation"></a>既知の問題 (インストール後)

- App Service が既存の仮想ネットワークにデプロイされ、ファイル サーバーがプライベート ネットワークでしか使用できない場合、worker はファイル サーバーに到達することができません。Azure App Service on Azure Stack のデプロイ ドキュメントで説明されているとおりです。

  ファイル サーバーに接続するために既存の仮想ネットワークと内部 IP アドレスへデプロイする場合は、送信セキュリティ規則を追加して、worker サブネットとファイル サーバー間の SMB トラフィックを有効にする必要があります。 管理者ポータルで WorkersNsg に移動し、次のプロパティを持つ送信セキュリティ規則を追加します。
  - ソース:Any
  - 送信元ポート範囲: *
  - 変換先:IP アドレス
  - 宛先 IP アドレス範囲:ファイル サーバーの IP の範囲
  - 送信先ポート範囲:445
  - プロトコル:TCP
  - アクション:Allow
  - 優先順位:700
  - 名前:Outbound_Allow_SMB445

### <a name="known-issues-for-cloud-admins-operating-azure-app-service-on-azure-stack"></a>Azure App Service on Azure Stack を運用するクラウド管理者に関する既知の問題

- カスタム ドメインは、切断された環境ではサポートされない

App Service は、パブリック DNS エンドポイントに対してドメインの所有権の検証を実行します。このため、カスタム ドメインは、切断されたシナリオではサポートされません。

## <a name="next-steps"></a>次のステップ

- Azure App Service の概要については、「[Azure Stack 上の App Service の概要](azure-stack-app-service-overview.md)」をご覧ください。
- App Service on Azure Stack のデプロイの準備をする方法の詳細については、「[App Service on Azure Stack を開始する前に](azure-stack-app-service-before-you-get-started.md)」をご覧ください。
