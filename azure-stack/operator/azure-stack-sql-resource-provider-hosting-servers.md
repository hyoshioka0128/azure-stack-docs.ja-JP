---
title: SQL リソース プロバイダーへのホスティング サーバーの追加
titleSuffix: Azure Stack Hub
description: SQL リソース プロバイダー アダプターを使用したプロビジョニングのためにホスティング サーバーを追加する方法について説明します。
author: bryanla
ms.topic: article
ms.date: 10/02/2019
ms.author: bryanla
ms.reviewer: xiaofmao
ms.lastreviewed: 10/16/2019
ms.openlocfilehash: 0345c3290b717385d8080dc6be771660ea22a2e1
ms.sourcegitcommit: e9a1dfa871e525f1d6d2b355b4bbc9bae11720d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86487908"
---
# <a name="add-hosting-servers-for-the-sql-resource-provider"></a>SQL リソース プロバイダーへのホスティング サーバーの追加

SQL リソース プロバイダーがインスタンスに接続できる限り、[Azure Stack Hub](azure-stack-overview.md) 内の仮想マシン (VM) 上、または Azure Stack Hub 環境の外部にある VM 上で、SQL Server データベース ホスティング サーバーを作成できます。

> [!NOTE]
> SQL リソース プロバイダーは既定のプロバイダー サブスクリプションに作成する必要がありますが、SQL ホスティング サーバーは課金対象のユーザー サブスクリプションに作成する必要があります。 リソース プロバイダー サーバーは、ユーザー データベースをホストするためには使用しないでください。

## <a name="overview"></a>概要

SQL ホスティング サーバーを追加する前に、次の必須要件と一般要件を確認します。

### <a name="mandatory-requirements"></a>必須要件

* SQL Server インスタンス上で SQL 認証を有効にします。 SQL リソース プロバイダー VM は、ドメインに参加していないため、SQL 認証を使用してホスティング サーバーに接続することのみ可能です。
* Azure Stack Hub にインストールするときに、SQL インスタンスの IP アドレスをパブリックとして構成します。 リソース プロバイダーとユーザー (Web アプリなど) は、ユーザー ネットワーク経由で通信します。そのため、このネットワーク上の SQL インスタンスへの接続が必要です。

### <a name="general-requirements"></a>一般的な要件

* SQL インスタンスは、リソース プロバイダーとユーザー ワークロードでのみ使用されるようにする必要があります。 他のコンシューマーが使用中の SQL インスタンスは使用できません。 この制限は、App Services にも適用されます。
* リソース プロバイダー用に、適切な特権レベルが設定されたアカウントを構成します (下記参照)。
* SQL インスタンスとそのホストの管理は、ご自身で行う必要があります。 たとえば、リソース プロバイダーは、更新プログラムの適用、バックアップの処理、または資格情報のローテーションの処理は行いません。

### <a name="sql-server-vm-images"></a>SQL Server VM のイメージ

Marketplace 管理機能により、SQL IaaS VM イメージを使用できます。 これらのイメージは、Azure で使用できる SQL VM と同じです。

Marketplace 項目を使用して SQL VM をデプロイする前に、必ず **SQL IaaS 拡張機能**の最新バージョンをダウンロードしてください。 IaaS 拡張機能や対応するポータル拡張機能により、修正プログラムの自動適用やバックアップなどの追加機能が提供されます。 この拡張機能の詳細については、[SQL Server Agent 拡張機能を使用した Azure VM での管理タスクの自動化](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-agent-extension)に関する記事を参照してください。

> [!NOTE]
> SQL IaaS 拡張機能は、Marketplace にあるすべての SQL on Windows イメージに "_必須_" です。この拡張機能をダウンロードしなかった場合、VM のデプロイは失敗します。 これは、Linux ベースの SQL VM イメージでは使用されません。

SQL VM のデプロイには、[Azure Stack Hub Quickstart Gallery](https://github.com/Azure/AzureStack-QuickStart-Templates) 内のテンプレートなどのその他の選択肢もあります。

> [!NOTE]
> マルチノードの Azure Stack Hub にインストールされるホスティング サーバーは、既定のプロバイダー サブスクリプションではなくユーザー サブスクリプションから作成する必要があります。 ユーザー ポータル、または適切にログインされた PowerShell セッションから作成する必要があります。 すべてのホスティング サーバーが課金対象の VM であり、適切な SQL ライセンスが必要です。 サービス管理者は、そのサブスクリプションの所有者になることが "_できます_"。

### <a name="required-privileges"></a>必要な特権

SQL sysadmin より低い特権を持つ管理者ユーザーを作成できます。 そのようなユーザーには、次の操作のためのアクセス許可だけが必要です。

* データベース: 作成、変更、含有 (Always On のみ)、ドロップ、バックアップ
* 可用性グループ: データベースの変更、結合、追加/削除
* ログイン: 作成、選択、変更、ドロップ、取り消し
* 操作の選択: \[master\].\[sys\].\[availability_group_listeners\] (AlwaysOn)、sys.availability_replicas (AlwaysOn)、sys.databases、\[master\].\[sys\].\[dm_os_sys_memory\]、SERVERPROPERTY、\[master\].\[sys\].\[availability_groups\] (AlwaysOn)、sys.master_files

### <a name="additional-security-information"></a>追加のセキュリティに関する情報

次の情報は、追加のセキュリティに関するガイダンスを提供します。

* すべての Azure Stack Hub ストレージは BitLocker を使用して暗号化されているので、Azure Stack Hub 上のどの SQL インスタンスも暗号化された Blob ストレージを利用します。
* SQL リソース プロバイダーは、TLS 1.2 を完全にサポートします。 SQL RP を通じて管理される SQL Server はすべて、TLS 1.2 "_のみ_" に対して構成されており、RP は既定でこれに設定されることを確認します。 サポートされているすべてのバージョンの SQL Server は、TLS 1.2 をサポートしています。 詳細については、「[Microsoft SQL Server 用の TLS 1.2 のサポート](https://support.microsoft.com/help/3135244/tls-1-2-support-for-microsoft-sql-server)」を参照してください。
* SQL Server 構成マネージャーを使用して、SQL Server へのすべての通信が常に暗号化されるように **ForceEncryption** オプションを設定します。 詳細については、「[暗号化された接続を強制するサーバーを構成するには](/sql/database-engine/configure-windows/enable-encrypted-connections-to-the-database-engine?view=sql-server-2017#to-configure-the-server-to-force-encrypted-connections)」を参照してください。
* クライアント アプリが、暗号化された接続で通信することも確認します。
* RP は、SQL Server インスタンスで使用される証明書を信頼するように構成されています。

## <a name="provide-capacity-by-connecting-to-a-standalone-hosting-sql-server"></a>スタンドアロンのホスティング SQL Server に接続して容量を提供する

SQL Server 2014 または SQL Server 2016 の任意のエディションを使って、スタンドアロンの (非 HA) SQL Server を使用できます。 sysadmin 特権を持つアカウントの資格情報があることを確認してください。

既に設定されているスタンドアロンのホスティング サーバーを追加するには、次の手順を実行します。

1. Azure Stack Hub 管理ポータルにサービス管理者としてサインインします。

2. **[すべてのサービス]** 、 **[管理リソース]** 、 **[SQL Hosting Servers]\(SQL ホスティング サーバー\)** の順に選択します。

   ![Azure Stack Hub 管理者ポータルの SQL ホスティング サーバー](./media/azure-stack-sql-rp-deploy/sqlhostingservers.png)

   **[SQL Hosting Servers]\(SQL ホスティング サーバー\)** では、リソース プロバイダーのバックエンドとして機能する SQL Server のインスタンスに SQL リソース プロバイダーを接続できます。

   ![Azure Stack Hub 管理者ポータルの SQL アダプター ダッシュボード](./media/azure-stack-sql-rp-deploy/sql-rp-hosting-server.png)

3. **[追加]** をクリックし、 **[Add a SQL Hosting Server]\(SQL ホスティング サーバーの追加\)** ブレードで、SQL Server インスタンスの接続の詳細を指定します。

   ![Azure Stack Hub 管理者ポータルでの SQL ホスティング サーバーの追加](./media/azure-stack-sql-rp-deploy/sql-rp-new-hosting-server.png)

    必要に応じて、インスタンス名を指定します。また、インスタンスが既定のポート 1433 に割り当てられていない場合は、ポート番号を指定します。

   > [!NOTE]
   > ユーザーと管理者の Azure Resource Manager が SQL インスタンスにアクセスできる限り、SQL インスタンスをリソースプロバイダーの管理下に置くことができます。 SQL インスタンスは、リソース プロバイダーに排他的に割り当てる __必要があります__。

4. サーバーを追加するときは、追加するサーバーを既存の SKU に割り当てるか、または新しい SKU を作成する必要があります。 **[Add a Hosting Server]\(ホスティング サーバーの追加\)** の下で、 **[SKU]** を選択します。

   * 既存の SKU を使用するには、使用可能な SKU を選択してから、 **[作成]** を選択します。
   * SKU を作成するには、 **[+ Create new SKU]\(+ 新しい SKU の作成\)** を選択します。 必要な情報を **[Create SKU]\(SKU の作成\)** に入力し、 **[OK]** を選択します。

     ![Azure Stack Hub 管理者ポータルでの SKU の作成](./media/azure-stack-sql-rp-deploy/sqlrp-new-sku.png)

## <a name="provide-high-availability-using-sql-always-on-availability-groups"></a>SQL Always On 可用性グループを使用して高可用性を提供する

SQL Always On インスタンスの構成にはさらに手順が必要で、3 台の VM (または物理マシン) が必要です。この記事は、Always On 可用性グループについて十分に理解していることを前提にしています。 詳細については、次の記事を参照してください。

* [Azure Virtual Machines での SQL Server Always On 可用性グループの概要](/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-availability-group-overview)
* [Always On 可用性グループ (SQL Server)](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2017)

> [!NOTE]
> SQL アダプター リソース プロバイダーは、Always On 可用性グループに対して SQL 2016 SP1 Enterprise 以降のインスタンス _のみ_ をサポートします。 このアダプターの構成には、自動シード処理などの SQL の新機能が必要です。

### <a name="automatic-seeding"></a>自動シード処理

SQL Server のインスタンスごとに各可用性グループで[自動シード処理](/sql/database-engine/availability-groups/windows/automatically-initialize-always-on-availability-group)を有効にする必要があります。

すべてのインスタンスで自動シード処理を有効にするには、次の SQL コマンドを編集し、セカンダリ インスタンスごとにプライマリ レプリカで実行します。

  ```sql
  ALTER AVAILABILITY GROUP [<availability_group_name>]
      MODIFY REPLICA ON '<secondary_node>'
      WITH (SEEDING_MODE = AUTOMATIC)
  GO
  ```

可用性グループは角かっこで囲む必要があります。

セカンダリ ノードで、次の SQL コマンドを実行します。

  ```sql
  ALTER AVAILABILITY GROUP [<availability_group_name>] GRANT CREATE ANY DATABASE
  GO
  ```

### <a name="configure-contained-database-authentication"></a>包含データベース認証を構成する

包含データベースを可用性グループに追加する前に、包含データベース認証サーバー オプションが、可用性グループの可用性レプリカをホストするすべてのサーバー インスタンスで 1 に設定されていることを確認します。 詳細については、[包含データベース認証サーバー構成オプション](/sql/database-engine/configure-windows/contained-database-authentication-server-configuration-option?view=sql-server-2017)を参照してください。

これらのコマンドを使用して、包含データベース認証サーバー オプションをインスタンスごとに設定します。

  ```sql
  EXEC sp_configure 'contained database authentication', 1
  GO
  RECONFIGURE
  GO
  ```

### <a name="to-add-sql-always-on-hosting-servers"></a>SQL Always On ホスティング サーバーを追加するには

1. Azure Stack Hub 管理ポータルにサービス管理者としてサインインします。

2. **[参照]** 、 **[管理リソース]** 、 **[SQL Hosting Servers]\(SQL ホスティング サーバー\)** 、 **[+ 追加]** の順に選択します。

   **[SQL Hosting Servers]\(SQL ホスティング サーバー\)** では、リソース プロバイダーのバックエンドとして機能する SQL Server の実際のインスタンスに SQL Server リソース プロバイダーを接続できます。

3. SQL Server インスタンスの接続詳細をフォームに入力します。 Always On リスナーの FQDN アドレス (および必要に応じてポート番号とインスタンス名) を必ず使用してください。 sysadmin 特権で構成したアカウントの情報を提供します。

4. SQL Always On 可用性グループ ボックスのインスタンスのサポートを有効にするには、[Always On 可用性グループ] ボックスをオンにします。

   ![Azure Stack Hub 管理者ポータルでの Always On 可用性グループの有効化](./media/azure-stack-sql-rp-deploy/AlwaysOn.PNG)

5. SKU に SQL Always On インスタンスを追加します。

   > [!IMPORTANT]
   > 同じ SKU にスタンドアロン サーバーと Always On インスタンスを混在させることはできません。 最初のホスティング サーバーを追加した後に型を混在させようとすると、エラーになります。

## <a name="sku-notes"></a>SKU に関する注意
使用する SKU 名は、SKU 内のサーバーの機能 (容量やパフォーマンス) を示すものにしてください。 この名前は、該当する SKU にユーザーがデータベースをデプロイするときの助けになります。 たとえば、SKU 名を使用して、サービス内容を次の特性によって区別することもできます。
  
* 大容量
* ハイ パフォーマンス
* 高可用性

ベスト プラクティスとして、SKU 内のすべてのホスティング サーバーに同じリソースとパフォーマンス特性を持たせてください。

SKU は、特定のユーザーやグループに割り当てることはできません。

SKU はポータルに表示されるまで最大 1 時間かかることがあります。 SKU の作成が完了するまで、ユーザーはデータベースを作成できません。

SKU を編集するには、 **[すべてのサービス]**  >  **[SQL アダプター]**  >  **[SKU]** に移動します。 変更する SKU を選択し、必要な変更を加えた後、 **[保存]** をクリックして変更を保存します。 

不要になった SKU を削除するには、 **[すべてのサービス]**  >  **[SQL アダプター]**  >  **[SKU]** に移動します。 SKU 名を右クリックし、 **[削除]** を選択して削除します。

> [!IMPORTANT]
> 新しい SKU がユーザー ポータルで使用可能になるまでに最大で 1 時間かかることがあります。

## <a name="make-sql-databases-available-to-users"></a>ユーザーが SQL データベースを使用できるようにする

ユーザーが SQL データベースを使用できるようにするプランとオファーを作成します。 プランに **Microsoft.SqlAdapter** サービスを追加し、新しいクォータを作成します。

> [!IMPORTANT]
> ユーザー ポータルで新しいクォータが使用可能になるまで、または変更されたクォータが適用されるまでに最大で 2 時間かかることがあります。

## <a name="next-steps"></a>次のステップ

[データベースを追加する](azure-stack-sql-resource-provider-databases.md)
