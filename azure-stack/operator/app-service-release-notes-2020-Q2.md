---
title: App Service on Azure Stack Hub 2020 年第 2 四半期のリリース ノート
description: Azure App Service on Azure Stack Hub の 2020 年第2 四半期リリースの内容、既知の問題、および更新プログラムをダウンロードする場所について説明します。
author: apwestgarth
manager: stefsch
ms.topic: article
ms.date: 11/17/2020
ms.author: anwestg
ms.reviewer: anwestg
ms.lastreviewed: 04/30/2020
ms.openlocfilehash: 6534a4539fc4e0fd699b21e84490f1d25be1dfe1
ms.sourcegitcommit: 2562b86f47db20e2652d4636227afb9cfd0e03ae
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/18/2020
ms.locfileid: "94785857"
---
# <a name="app-service-on-azure-stack-hub-2020-q2-release-notes"></a>App Service on Azure Stack Hub 2020 年第 2 四半期のリリース ノート

これらのリリース ノートでは、Azure App Service on Azure Stack Hub 2020 Q2 における機能強化と修正点、および既知の問題について説明します。 既知の問題は、デプロイおよび更新プロセスに直接関係する問題と、ビルド (インストール後) に関する問題に分けられています。

[!INCLUDE [Azure Stack Hub update reminder](../includes/app-service-hub-update-banner.md)]

## <a name="build-reference"></a>ビルドのリファレンス

App Service on Azure Stack Hub 2020 Q2 のビルド番号は **87.0.2.10** です

## <a name="prerequisites"></a>前提条件

デプロイを開始する前に、[前提条件に関するドキュメント](azure-stack-app-service-before-you-get-started.md)をご覧ください。

Azure App Service on Azure Stack Hub を 2020 Q2 にアップグレードを開始する前に:

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

Azure App Service on Azure Stack Hub Update Q2 には、次の機能強化と修正が含まれています。

- **App Service のテナント ポータル、管理ポータル、Functions ポータル、Kudu ツール** の更新。 Azure Stack Portal SDK バージョンと一致しています。

- **Azure Functions ランタイム** が **v1.0.13021** に更新されました。

- 信頼性を高めるためのコア サービスと、一般的な問題を簡単に診断できるようにするエラー メッセージの更新。

- **次のアプリケーション フレームワークとツールの更新**:
  - ASP.NET Framework 4.7.2
  - ASP.NET Core 3.1.3
  - ASP.NET Core Module v2 13.1.19331.0
  - PHP 7.4.2
  - Kudu を 86.20224.4450 に更新
  - NodeJS 
    - 8.17.0
    - 10.19.0
    - 12.13.0
    - 12.15.0
  - NPM
    - 5.6.0
    - 6.1.0
    - 6.12.0
    - 6.13.4
  
- **すべてのロールの基になっているオペレーティング システムの更新プログラム**:
  - [x64 ベース システム用 Windows Server 2016 に対する 2020-04 累積的な更新プログラム (KB4550929)](https://support.microsoft.com/help/4550929)
  - [x64 ベース システム用 Windows Server 2016 に対する 2020-04 サービス スタック更新プログラム (KB4550994)](https://support.microsoft.com/help/4550994)

- **デプロイとアップグレードの一部として、Windows Server の累積的な更新プログラムがコントローラー ロールに適用されるようになりました**

- **新しいデプロイの既定の仮想マシンとスケール セットの SKU を更新しました**: パブリック クラウド サービスとの一貫性を維持するために、Azure App Service on Azure Stack Hub の新しいデプロイでは、リソース プロバイダーの運用に使用される基になるマシンとスケール セットに対して次の SKU を使用します
  
  | Role | SKU の最小値 |
  | --- | --- |
  | コントローラー | Standard_A4_v2 - (4 コア、8192 MB) |
  | 管理 | Standard_D3_v2 - (4 コア、14336 MB) |
  | Publisher | Standard_A2_v2 - (2 コア、4096 MB) |
  | FrontEnd | Standard_A4_v2 - (4 コア、8192 MB) |
  | 共有 Worker | Standard_A4_v2 - (4 コア、8192 MB) |
  | 小規模専用 worker | Standard_A1_v2 - (1 コア、2048 MB) |
  | 中規模専用 worker | Standard_A2_v2 - (2 コア、4096 MB) |
  | 大規模専用 worker | Standard_A4_v2 - (4 コア、8192 MB) |

ASDK デプロイでは、インスタンスを下位の SKU にスケールダウンしてコアとメモリのコミットを減らすことができますが、パフォーマンスが低下します。

## <a name="issues-fixed-in-this-release"></a>このリリースで修正された問題

- SQL Server Always On クラスターがセカンダリ ノードにフェール オーバーされた場合も、アップグレードが完了するようになりました
- Azure App Service on Azure Stack Hub の新しいデプロイでは、データベースを包含データベースに手動で変換する必要はなくなりました
- worker またはインフラストラクチャのロール インスタンスの追加は、手動での介入なしに正常に完了します
- カスタム worker 層の追加は、手動での介入なしに正常に完了します
- カスタム worker 層の削除は、ポータル エラーなしで完了するようになりました
- ローカル ディスクの空き領域が不足している場合、worker は準備完了とマークされなくなりました
- Azure Resource Manager 証明書を取得する際のタイムアウトが増加しました
- サーバー ログから取得され管理ポータルに表示されるメッセージの数は、Azure Resource Manager の最大要求サイズ以下になるように制限されます
- 使用状況サービスの起動の問題の原因となっていたタイムアウトの問題
- Orchard CMS サイト作成時のデータベースのデプロイに関する問題が解決されました
- デプロイとアップグレードの一部として、コントローラーが Windows の累積的な更新プログラムで更新されるようになりました
- カスタム ドメインの検証が失敗しても、App Service によって操作がロックされなくなりました

## <a name="pre-update-steps"></a>更新前の手順

[更新の既知の問題](#known-issues-update)を確認し、規定されている措置を講じます。

## <a name="post-deployment-steps"></a>デプロイ後の手順

> [!IMPORTANT]
> SQL Always On インスタンスを使用して App Service リソース プロバイダーを提供している場合は、データベースのフェールオーバーが発生したときにサービスが失われないように、[appservice_hosting と appservice_metering データベースを可用性グループに追加](/sql/database-engine/availability-groups/windows/availability-group-add-a-database)し、それらのデータベースを同期する必要があります。

## <a name="known-issues-update"></a>既知の問題 (更新プログラム)

- お客様が appservice_hosting および appservice_metering データベースを包含データベースに変換した場合、ログインが包含ユーザーに正常に移行されていないと、アップグレードが失敗することがあります

デプロイ後に Appservice_hosting および appservice_metering データベースを包含データベースに変換し、データベース ログインを包含ユーザーに正常に移行できていない場合、アップグレード エラーが発生する可能性があります。  

お客様は、Azure App Service on Azure Stack Hub インストールを 2020 Q2 にアップグレードする前に、appservice_hosting と appservice_metering をホストしている SQL Server に対して次のスクリプトを実行する必要があります。  **このスクリプトは非破壊的であり、ダウンタイムが発生することはありません。**

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

- テナントは、テナント ポータルの App Service プラン ビューで新規を使用して App Service プランを作成できない

新しいアプリケーションを作成する場合、テナントはアプリの作成ワークフローで App Service プランを作成できます。また、App Service プランは、現在のアプリの App Service プランを変更する際や、App Service プランの Marketplace 項目を使用して作成することもできます

- カスタム ドメインは、切断された環境ではサポートされない

App Service は、パブリック DNS エンドポイントに対してドメインの所有権の検証を実行します。このため、カスタム ドメインは、切断されたシナリオではサポートされません。

- 場合によっては、ワーカーが正常性チェックに合格しないことがあります (ディスク領域不足)

多くのサイトがワーカーに割り当てられている場合や、サイトで大量の要求が処理されている場合、ワーカーによって C:\DWAS\LogFiles に多数のランタイム ログ ファイルが生成されます。  これは、これらのログ ファイルのクリーンアップ ロジックのバグが原因です。  

この問題を軽減するには、個々のワーカーを削除し、フォルダーの内容を消去します。

この問題は、[App Service on Azure Stack Hub 2020 Q3](app-service-release-notes-2020-Q3.md) で修正されました。できるだけ早く 2020 Q3 リリースにアップグレードすることをお勧めします。

> [!IMPORTANT]
> Azure App Service on Azure Stack Hub 2020 Q3 に更新するには、Azure Stack Hub 2008 へアップグレードする **必要があります**

## <a name="next-steps"></a>次のステップ

- Azure App Service の概要については、「[Azure Stack 上の App Service の概要](azure-stack-app-service-overview.md)」をご覧ください。
- App Service on Azure Stack のデプロイの準備をする方法の詳細については、「[App Service on Azure Stack を開始する前に](azure-stack-app-service-before-you-get-started.md)」をご覧ください。
