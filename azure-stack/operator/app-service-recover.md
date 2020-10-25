---
title: Azure Stack 上の App Service の復旧
description: Azure Stack Hub 上の App Service のディザスター リカバリーについて説明します。
author: bryanla
ms.topic: article
ms.date: 03/21/2019
ms.author: anwestg
ms.reviewer: anwestg
ms.lastreviewed: 03/21/2019
ms.openlocfilehash: ba57de42f882ce62d433768f4fd2a265b862459c
ms.sourcegitcommit: e9a1dfa871e525f1d6d2b355b4bbc9bae11720d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86489761"
---
# <a name="app-service-recovery-on-azure-stack-hub"></a>Azure Stack 上の App Service の復旧

このトピックでは、App Service の障害復旧のために実行するアクションの手順を説明します。

Azure Stack Hub 上の App Service をバックアップから復旧するには、次のアクションを実行する必要があります。
1. App Service のデータベースを復元します。
2. ファイル サーバーの共有コンテンツを復元します。
3. App Service のロールとサービスを復元します。

Azure Stack Hub ストレージが Function App ストレージに使用されていた場合は、Function App を復元するための手順も実行する必要があります。

## <a name="restore-the-app-service-databases"></a>App Service のデータベースの復旧
App Service SQL Server のデータベースは、実稼働対応の SQL Server インスタンスで復旧する必要があります。 

App Service データベースをホストするように [SQL Server インスタンスを準備](azure-stack-app-service-before-you-get-started.md#prepare-the-sql-server-instance)してから、次の手順を使用して、データベースをバックアップから復旧します。

1. 復旧した App Service データベースをホストする SQL Server に管理者権限でサインインします。
2. 管理者権限で実行しているコマンド プロンプトから、App Service データベースを復元するための次のコマンドを使用します。
    ```dos
    sqlcmd -U <SQL admin login> -P <SQL admin password> -Q "RESTORE DATABASE appservice_hosting FROM DISK='<full path to backup>' WITH REPLACE"
    sqlcmd -U <SQL admin login> -P <SQL admin password> -Q "RESTORE DATABASE appservice_metering FROM DISK='<full path to backup>' WITH REPLACE"
    ```
3. 両方の App Service データベースが正常に復旧されたことを確認し、SQL Server Management Studio を終了します。

> [!NOTE]
> フェールオーバー クラスター インスタンスの障害から復旧するには、「[フェールオーバー クラスター インスタンス障害からの復旧](/sql/sql-server/failover-clusters/windows/recover-from-failover-cluster-instance-failure?view=sql-server-2017)」を参照してください。 

## <a name="restore-the-app-service-file-share-content"></a>App Service ファイル共有のコンテンツを復旧する
App Service ファイル共有をホストするように[ファイル サーバーを順](azure-stack-app-service-before-you-get-started.md#prepare-the-file-server)したら、テナント ファイル共有のコンテンツをバックアップから復旧する必要があります。 新しく作成された App Service ファイル共有場所にファイルをコピーするために、利用可能な任意の方法を使用できます。 ファイル サーバーで次の例を実行すると、リモート共有に接続してファイルを共有にコピーするために、PowerShell と Robocopy が使用されます。

```powershell
$source = "<remote backup storage share location>"
$destination = "<local file share location>"
net use $source /user:<account to use to connect to the remote share in the format of domain\username> *
robocopy /E $source $destination
net use $source /delete
```

ファイル共有のコンテンツのコピーに加えて、ファイル共有自体のアクセス許可をリセットする必要もあります。 アクセス許可をリセットするには、ファイル サーバー コンピューターで管理コマンド プロンプトを開き、**ReACL.cmd** ファイルを実行します。 **ReACL.cmd** ファイルは、**BCDR** ディレクトリー内の App Service インストール ファイルにあります。

## <a name="restore-app-service-roles-and-services"></a>App Service のロールとサービスの復旧
App Service データベースとファイル共有コンテンツを復旧したら、PowerShell を使用して App Service のロールとサービスを復旧する必要があります。 これらの手順により、App Service のシークレットとサービス構成を復旧します。  

1. App Service のインストール時に指定したパスワードを使用して、App Service コントローラー **CN0-VM** VM に **roleadmin** としてログインします。 
    > [!TIP]
    > VM のネットワーク セキュリティ グループが RDP 接続を許可するように変更する必要があります。 
2. **SystemSecrets.JSON** ファイルをコントローラー VM にコピーします。 次の手順で、このファイルへのパスを `$pathToExportedSecretFile` パラメーターとして渡す必要があります。
3. 管理者特権の PowerShell コンソール ウィンドウで次のコマンドを実行し、App Service のロールとサービスを復旧します。

    ```powershell
    # Stop App Service services on the primary controller VM
    net stop WebFarmService
    net stop ResourceMetering
    net stop HostingVssService # This service was deprecated in the App Service 1.5 release and is not required after the App Service 1.4 release.

    # Restore App Service secrets. Provide the path to the App Service secrets file copied from backup. For example, C:\temp\SystemSecrets.json.
    # Press ENTER when prompted to reconfigure App Service from backup 

    # If necessary, use -OverrideDatabaseServer <restored server> with Restore-AppServiceStamp when the restored database server has a different address than backed-up deployment.
    # If necessary, use -OverrideContentShare <restored file share path> with Restore-AppServiceStamp when the restored file share has a different path from backed-up deployment.
    Restore-AppServiceStamp -FilePath $pathToExportedSecretFile 

    # Restore App Service roles
    Restore-AppServiceRoles

    # Restart App Service services
    net start WebFarmService
    net start ResourceMetering
    net start HostingVssService  # This service was deprecated in the App Service 1.5 release and is not required after the App Service 1.4 release.

    # After App Service has successfully restarted, and at least one management server is in ready state, synchronize App Service objects to complete the restore
    # Enter Y when prompted to get all sites and again for all ServerFarm entities.
    Get-AppServiceSite | Sync-AppServiceObject
    Get-AppServiceServerFarm | Sync-AppServiceObject
    ```

> [!TIP]
> コマンドが完了したら、この PowerShell セッションを閉じることを強くお勧めします。

## <a name="restore-function-apps"></a>Function App を復旧する 
Azure Stack Hub 用の App Service では、テナント ユーザー アプリファイルの復元、または共有コンテンツ以外のデータの復元はサポートされていません。 他のすべてのデータは、App Service のバックアップと復旧操作以外でバックアップして復旧する必要があります。 Azure Stack Hub ストレージが Function App ストレージに使用されていた場合、失われたデータを復旧するには、次の手順を実行する必要があります。

1. Function App によって使用される新しいストレージ アカウントを作成します。 このストレージには、Azure Stack Hub ストレージ、Azure ストレージ、または互換性のある任意のストレージを指定できます。
2. ストレージの接続文字列を取得します。
3. 関数ポータルを開き、Function App を参照します。
4. **[プラットフォーム機能]** タブを参照し、 **[アプリケーションの設定]** をクリックします。
5. **AzureWebJobsDashboard** および **AzureWebJobsStorage** を新しい接続文字列に変更し、 **[保存]** をクリックします。
6. **[概要]** に切り替えます。
7. アプリを再起動します。 すべてのエラーを解消するまでに、複数回試行される可能性があります。

## <a name="next-steps"></a>次のステップ
[Azure Stack Hub 上の App Service の概要](azure-stack-app-service-overview.md)
