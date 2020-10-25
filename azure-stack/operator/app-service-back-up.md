---
title: Azure Stack Hub 上の App Service をバックアップする
description: Azure Stack Hub 上の App Services をバックアップする方法について説明します。
author: bryanla
ms.topic: article
ms.date: 04/23/2019
ms.author: anwestg
ms.reviewer: anwestg
ms.lastreviewed: 03/21/2019
ms.openlocfilehash: 3b12c794550c3d39809369ae86117d9c684e1a53
ms.sourcegitcommit: e9a1dfa871e525f1d6d2b355b4bbc9bae11720d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86489795"
---
# <a name="back-up-app-service-on-azure-stack-hub"></a>Azure Stack Hub 上の App Service をバックアップする

このドキュメントでは、Azure Stack Hub 上の App Service をバックアップする方法の手順を説明します。

> [!IMPORTANT]
> Azure Stack Hub 上の App Service は、[Azure Stack Hub のインフラストラクチャ バックアップ](azure-stack-backup-infrastructure-backup.md)の一部としてはバックアップされません。 Azure Stack Hub のオペレーターとして、必要に応じて、App Service を確実に正常な復元ができるようにするための手順を実行する必要があります。

Azure Stack Hub 上の Azure App Service でディザスター リカバリーを計画する場合は、次の 4 つの考慮すべき主要なコンポーネントがあります。
1. リソース プロバイダーのインフラストラクチャ、サーバー ロール、worker 階層などの 4 つの主な要素を考慮する必要があります。 
2. App Service のシークレット。
3. App Service SQL Server ホスティングおよびメータリング データベース。
4. App Service のファイル共有に格納された App Service ユーザー ワークロード コンテンツ。

## <a name="back-up-app-service-secrets"></a>App Service のシークレットのバックアップ
App Service をバックアップから復旧する場合、初期デプロイで使用した App Service キーを入力する必要があります。 この情報は、App Service が正常にデプロイされ、安全な場所に格納されたら、すぐに保存する必要があります。 リソース プロバイダーのインフラストラクチャ構成は、App Service リカバリー PowerShell コマンドレットを使用して、復旧中にバックアップから再作成されます。

次の手順に従って、管理ポータルからアプリ サービスのシークレットをバックアップします。 

1. サービス管理者として Azure Stack Hub 管理者ポータルにサインインします。

2. **[App Service]**  ->  **[シークレット]** に移動します。 

3. **[シークレットのダウンロード]** を選択します。

   ![Azure Stack Hub 管理者ポータルでシークレットをダウンロードする](./media/app-service-back-up/download-secrets.png)

4. シークレットのダウンロードの準備が整ったら、 **[保存]** をクリックして安全な場所に App Service のシークレット (**SystemSecrets.JSON**) ファイルを保存します。 

   ![Azure Stack Hub 管理者ポータルでシークレットを保存する](./media/app-service-back-up/save-secrets.png)

> [!NOTE]
> これらの手順は、App Service のシークレットを更新するたびに繰り返します。

## <a name="back-up-the-app-service-databases"></a>App Service のデータベースのバックアップ
App Service を復元するには、**Appservice_hosting** データベースおよび **Appservice_metering** データベースのバックアップが必要です。 これらのデータベースが確実に定期的にバックアップされ安全な場所に保存されるようにするには、SQL Server メンテナンス プランまたは Azure Backup Server を使用することをお勧めします。 しかし、必ず定期的に SQL のバックアップが作成されるのであれば、いかなる方法でも使用できます。

SQL Server へのログイン中にこれらのデータベースを手動でバックアップするには、次の PowerShell のコマンドを使用します。

  ```powershell
  $s = "<SQL Server computer name>"
  $u = "<SQL Server login>" 
  $p = read-host "Provide the SQL admin password"
  sqlcmd -S $s -U $u -P $p -Q "BACKUP DATABASE appservice_hosting TO DISK = '<path>\hosting.bak'"
  sqlcmd -S $s -U $u -P $p -Q "BACKUP DATABASE appservice_metering TO DISK = '<path>\metering.bak'"
  ```

> [!NOTE]
> SQL AlwaysOn データベースをバックアップする必要がある場合、[これらの手順](/sql/database-engine/availability-groups/windows/configure-backup-on-availability-replicas-sql-server?view=sql-server-2017)に従ってください。 

すべてのデータベースを正常にバックアップできたら、.bak ファイルを App Service のシークレット情報と共に安全な場所にコピーします。

## <a name="back-up-the-app-service-file-share"></a>App Service のファイル共有のバックアップ
App Service では、ファイル共有にテナント アプリの情報が格納されます。 復元を行う必要がある場合に失われるデータを最小限にするには、このファイル共有を App Service データベースと共に定期的にバックアップする必要があります。

App Service のファイル共有のコンテンツをバックアップするには Azure Backup Server を使用するか、以前のすべての復旧情報を保存した場所にファイル共有のコンテンツを定期的にコピーする別の方法を使用します。

たとえば、次の手順を使用して、(PowerShell ISE ではなく) Windows PowerShell コンソール セッションから Robocopy を使用します。

```powershell
$source = "<file share location>"
$destination = "<remote backup storage share location>"
net use $destination /user:<account to use to connect to the remote share in the format of domain\username> *
robocopy $source $destination
net use $destination /delete
```

## <a name="next-steps"></a>次のステップ
[Azure Stack Hub 上の App Service の復元](app-service-recover.md)
