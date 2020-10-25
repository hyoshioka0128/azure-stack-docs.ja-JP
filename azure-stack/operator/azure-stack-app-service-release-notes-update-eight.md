---
title: App Service on Azure Stack Hub update 8 のリリース ノート
description: App Service on Azure Stack Hub の Update 8 リリース ノート (新機能、修正点、既知の問題を含む)。
author: apwestgarth
manager: stefsch
ms.topic: article
ms.date: 05/05/2020
ms.author: anwestg
ms.reviewer: anwestg
ms.lastreviewed: 03/25/2019
ms.openlocfilehash: f19d64331e7ef64095bc91a04eb817e2f056d3a9
ms.sourcegitcommit: e9a1dfa871e525f1d6d2b355b4bbc9bae11720d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86489710"
---
# <a name="app-service-on-azure-stack-hub-update-8-release-notes"></a>App Service on Azure Stack Hub update 8 のリリース ノート

これらのリリース ノートでは、Azure App Service on Azure Stack Hub Update 8 の新機能、修正点、および既知の問題について説明します。 既知の問題は 2 つのセクションに分かれています。アップグレード プロセスに関連する問題とビルド (インストール後) に関する問題です。

> [!IMPORTANT]
> Azure App Service 1.8 をデプロイする前に、Azure Stack 統合システムに 1910 更新プログラムを適用するか、最新の Azure Stack Development Kit (ASDK) をデプロイします。

## <a name="build-reference"></a>ビルドのリファレンス

Azure App Service on Azure Stack Hub Update 8 のビルド番号は **86.0.2.13** です。

## <a name="prerequisites"></a>前提条件

デプロイを開始する前に、[App Service on Azure Stack Hub の前提条件](azure-stack-app-service-before-you-get-started.md)に関するページを参照してください。

Azure App Service on Azure Stack Hub の 1.8 へのアップグレードを開始する前に:

- Azure Stack Hub 管理者ポータルの Azure App Service の管理で確実にすべてのロールの準備ができているようにします。

- Azure Stack Hub 管理ポータルの Azure App Service 管理を使用して、App Service のシークレットをバックアップします

- App Service とマスター データベースをバックアップします。
  - AppService_Hosting
  - AppService_Metering
  - master

- テナント アプリのコンテンツ ファイル共有をバックアップします。

  > [!Important]
  > ファイル サーバーと SQL Server の保守と操作を担当するのは、クラウド オペレーターです。  リソース プロバイダーは、これらのリソースの管理は行いません。  クラウドオ ペレーターが、App Service データベースとテナント コンテンツ ファイル共有のバックアップを行います。

- Azure Stack Hub Marketplace から**カスタム スクリプト拡張機能**バージョン **1.9.3** を配信します

## <a name="new-features-and-fixes"></a>新機能と修正

Azure App Service on Azure Stack Hub Update 8 には、次の機能強化と修正が含まれています。

- **App Service のテナント ポータル、管理ポータル、関数ポータル、Kudu ツール**の更新。 Azure Stack Portal SDK バージョンと一貫性があります。

- **Azure 関数ランタイム**が **v1.0.12615** に更新されました。

- 信頼性を高めるためのコア サービスと、一般的な問題を簡単に診断できるようにするエラー メッセージの更新。

- **次のアプリ フレームワークとツールの更新**:

  - ASP.NET Core 3.1.0
  - ASP.NET Core 3.0.1
  - ASP.NET Core 2.2.8
  - ASP.NET Core Module v2 13.1.19331.0
  - Azul OpenJDK 8.38.0.13
  - Tomcat 7.0.94
  - Tomcat 8.5.42
  - Tomcat 9.0.21
  - PHP 7.1.32
  - PHP 7.2.22
  - PHP 7.3.9
  - Kudu を 85.11024.4154 に更新
  - MSDeploy 3.5.80916.15
  - NodeJS 10.16.3
  - NPM 6.9.0
  - Git for Windows 2.19.1.0

- **すべてのロールの基になっているオペレーティング システムの更新プログラム**:
  - [x64 ベース システム用 Windows Server 2016 に対する 2019-12 累積的な更新プログラム (KB4530689)](https://support.microsoft.com/help/4530689)

- **新しいデプロイのマネージド ディスクのサポート**

Azure App Service on Azure Stack Hub のすべての新しいデプロイでは、仮想マシンと仮想マシン スケール セットにマネージド ディスクが使用されます。 既存のすべてのデプロイでは、アンマネージド ディスクが引き続き使用されます。

- **フロントエンド ロード バランサーによって適用される TLS 1.2**

**TLS 1.2** はすべてのアプリに適用されるようになりました。

## <a name="known-issues-upgrade"></a>既知の問題 (アップグレード)

- SQL Server Always On クラスターがセカンダリ ノードにフェール オーバーされた場合、アップグレードが失敗します。

アップグレード中に、マスター接続文字列を使用してデータベースの存在を確認する呼び出しが行われます。これは、前のマスター ノードでのログインであったために失敗します。

次のいずれかの操作を実行し、インストーラー内で [再試行] を選択します。

- 現在のセカンダリ SQL ノードから `appservice_hostingAdmin` ログインをコピーします。

    **OR**

- SQL クラスターを前のアクティブ ノードにフェールオーバーします。

## <a name="post-deployment-steps"></a>デプロイ後の手順

> [!IMPORTANT]
> SQL Always On インスタンスを使用して App Service リソース プロバイダーを提供している場合は、データベースのフェールオーバーが発生したときにサービスが失われないように、[appservice_hosting と appservice_metering データベースを可用性グループに追加](/sql/database-engine/availability-groups/windows/availability-group-add-a-database)し、それらのデータベースを同期する "必要があります"。

## <a name="known-issues-post-installation"></a>既知の問題 (インストール後)

- App Service が既存の仮想ネットワークにデプロイされ、ファイル サーバーがプライベート ネットワークでしか使用できない場合、worker はファイル サーバーに到達することができません。Azure App Service on Azure Stack Hub のデプロイ ドキュメントで説明されているとおりです。

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

- Azure App Service on Azure Stack Hub 1.8 の新しいデプロイでは、データベースを包含データベースに変換する必要がある

    このリリースでの回帰が原因で、**新しく**デプロイしたときは、両方の App Service データベース (appservice_hosting と appservice_metering) を包含データベースに変換する必要があります。  これは、**アップグレードした**デプロイには影響**しません**。

    > [!IMPORTANT]
    > この手順の所要時間は 5 分から 10 分程度です。 この手順には、既存のデータベース ログイン セッションを中止する作業が伴います。 Azure App Service on Azure Stack Hub の移行とその後の検証に伴うダウンタイムを考慮してください。

    1. [AppService データベース (appservice_hosting and appservice_metering) を可用性グループ](/sql/database-engine/availability-groups/windows/availability-group-add-a-database)に追加します。

    1. 包含データベースを有効にします。

        ```sql

        sp_configure 'contained database authentication', 1;
        GO
        RECONFIGURE;
            GO
        ```

    1. データベースを部分的包含に変換します。 すべてのアクティブなセッションを強制終了する必要があるため、この手順ではダウンタイムが発生します。

        ```sql
        /******** [appservice_metering] Migration Start********/
            USE [master];

            -- kill all active sessions
            DECLARE @kill varchar(8000) = '';  
            SELECT @kill = @kill + 'kill ' + CONVERT(varchar(5), session_id) + ';'  
            FROM sys.dm_exec_sessions
            WHERE database_id  = db_id('appservice_metering')

            EXEC(@kill);

            USE [master]  
            GO  
            ALTER DATABASE [appservice_metering] SET CONTAINMENT = PARTIAL  
            GO  

        /********[appservice_metering] Migration End********/

        /********[appservice_hosting] Migration Start********/

            -- kill all active sessions
            USE [master];

            DECLARE @kill varchar(8000) = '';  
            SELECT @kill = @kill + 'kill ' + CONVERT(varchar(5), session_id) + ';'  
            FROM sys.dm_exec_sessions
            WHERE database_id  = db_id('appservice_hosting')

            EXEC(@kill);

            -- Convert database to contained
            USE [master]  
            GO  
            ALTER DATABASE [appservice_hosting] SET CONTAINMENT = PARTIAL  
            GO  

            /********[appservice_hosting] Migration End********/
        ```

    1. 包含データベース ユーザーにログインを移行します。

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

    **検証**

    1. SQL Server で包含が有効になっているかどうかを確認します。

        ```sql
        sp_configure  @configname='contained database authentication'
        ```

    1. 既存の包含の動作を確認します。

        ```sql
        SELECT containment FROM sys.databases WHERE NAME LIKE (SELECT DB_NAME())
        ```

- worker をスケールアウトできない

  新しい worker が、必要なデータベース接続文字列を取得できません。  この状況を解決するには、コントローラー インスタンスのいずれか (たとえば CN0-VM) に接続し、次の PowerShell スクリプトを実行します。

    ```powershell

    [System.Reflection.Assembly]::LoadWithPartialName("Microsoft.Web.Hosting")
    $siteManager = New-Object Microsoft.Web.Hosting.SiteManager

    $builder = New-Object System.Data.SqlClient.SqlConnectionStringBuilder -ArgumentList (Get-AppServiceConnectionString -Type Hosting)
    $conn = New-Object System.Data.SqlClient.SqlConnection -ArgumentList $builder.ToString()

    $siteManager.RoleServers | Where-Object {$_.IsWorker} | ForEach-Object {
        $worker = $_
        $dbUserName = "WebWorker_" + $worker.Name

        if (!$siteManager.ConnectionContexts[$dbUserName]) {
            $dbUserPassword = [Microsoft.Web.Hosting.Common.Security.PasswordHelper]::GenerateDatabasePassword()

            $conn.Open()
            $command = $conn.CreateCommand()
            $command.CommandText = "CREATE USER [$dbUserName] WITH PASSWORD = '$dbUserPassword'"
            $command.ExecuteNonQuery()
            $conn.Close()

            $conn.Open()
            $command = $conn.CreateCommand()
            $command.CommandText = "ALTER ROLE [WebWorkerRole] ADD MEMBER [$dbUserName]"
            $command.ExecuteNonQuery()
            $conn.Close()

            $builder.Password = $dbUserPassword
            $builder["User ID"] = $dbUserName

            $siteManager.ConnectionContexts.Add($dbUserName, $builder.ToString())
        }
    }

    $siteManager.CommitChanges()

    ```

## <a name="known-issues-for-cloud-admins-operating-azure-app-service-on-azure-stack-hub"></a>Azure App Service on Azure Stack Hub を運用するクラウド管理者に対する既知の問題

[Azure Stack Hub 1907 リリース ノート](./release-notes.md?view=azs-2002)内のドキュメントを参照してください。

## <a name="next-steps"></a>次のステップ

- Azure App Service の概要については、「[App Service on Azure Stack Hub と Azure Functions の概要](azure-stack-app-service-overview.md)」を参照してください。
- App Service on Azure Stack Hub のデプロイを準備する方法の詳細については、「[App Service on Azure Stack Hub のデプロイの前提条件](azure-stack-app-service-before-you-get-started.md)」をご覧ください。
