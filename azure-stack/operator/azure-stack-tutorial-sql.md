---
title: 高可用性 SQL データベースの作成
titleSuffix: Azure Stack Hub
description: Azure Stack Hub で SQL Server リソース プロバイダーのホスト コンピューターと高可用性 SQL Always On データベースを作成する方法について学習します。
author: BryanLa
ms.topic: article
ms.date: 10/07/2019
ms.author: bryanla
ms.reviewer: xiaofmao
ms.lastreviewed: 10/23/2019
ms.openlocfilehash: b30126bcfbbe57cd36a54ce1f5fc487014fe7a03
ms.sourcegitcommit: 69cfff119ab425d0fbb71e38d1480d051fc91216
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/30/2020
ms.locfileid: "91572876"
---
# <a name="create-highly-available-sql-databases-with-azure-stack-hub"></a>Azure Stack Hub を使用して高可用性 SQL データベースを作成する

Azure Stack Hub オペレーターとして、SQL Server データベースをホストするようにサーバー VM を構成できます。 SQL ホスティング サーバーが作成されて、Azure Stack Hub によって管理されると、SQL サービスにサブスクライブしているユーザーは SQL データベースを簡単に作成できます。

この記事では、Azure Stack Hub クイックスタート テンプレートを使用して [SQL Server Always On 可用性グループ](/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server)を作成し、それを Azure Stack Hub SQL ホスティング サーバーとして追加し、高可用性 SQL データベースを作成する方法について示します。

ここでは、次の内容について学習します。

> [!div class="checklist"]
> * テンプレートから SQL Server AlwaysOn 可用性グループを作成する。
> * SQL Server AlwaysOn 可用性グループを Azure Stack Hub SQL ホスティング サーバーとして構成する。
> * 高可用性 SQL データベースを作成する。

入手できる Azure Stack Marketplace の項目を使用して、2 つの VM SQL Server AlwaysOn 可用性グループを作成および構成します。

開始する前に、[SQL Server リソース プロバイダー](azure-stack-sql-resource-provider-deploy.md)が正常にインストールされ、Azure Stack Marketplace で次の項目を入手できることを確認してください。

> [!IMPORTANT]
> Azure Stack Hub クイックスタート テンプレートを使用するには、以下のすべてが必要です。

- Windows Server 2016 Datacenter。
- Windows Server 2016 サーバー イメージ上の SQL Server 2016 SP1 または SP2 (Enterprise、Standard、または Developer) 
- [SQL Server IaaS Extension](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-agent-extension) バージョン 1.3.20180 以降 SQL IaaS 拡張機能を使用して、すべての Windows バージョンの Marketplace SQL Server 項目に必要なコンポーネントをインストールします。 SQL 仮想マシン (VM) 上で SQL 固有の設定を構成することが可能になります。 拡張機能がローカルのマーケットプレースにインストールされていない場合、SQL のプロビジョニングは失敗します。
- [Windows バージョン 1.9.1 以降のカスタム スクリプト拡張機能](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.CustomScriptExtension)。 カスタム スクリプト拡張機能は、デプロイ後の VM カスタマイズ タスクを自動的に起動するために使用できるツールです。
- [PowerShell Desired State Configuration (DSC)](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.DSC-arm) バージョン 2.76.0.0 以降。 DSC は、Windows PowerShell の管理プラットフォームであり、ソフトウェア サービス用の構成データのデプロイと管理を行うことができます。 また、プラットフォームでは、これらのサービスを実行する環境の管理も行われます。

Azure Stack Marketplace への項目の追加に関する詳細については、[Azure Stack Hub Marketplace の概要](azure-stack-marketplace.md)に関するページを参照してください。

## <a name="create-a-sql-server-alwayson-availability-group"></a>SQL Server AlwaysOn 可用性グループを作成する

このセクションの手順を使用し、[sql-2016-Always On Azure Stack Hub クイックスタート テンプレート](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/sql-2016-alwayson)を使用して SQL Server Always On 可用性グループをデプロイします。 このテンプレートでは、Always On 可用性グループに 2 つの SQL Server Enterprise または Developer インスタンスをデプロイします。 このテンプレートは次のリソースを作成します。

- ネットワーク セキュリティ グループ。
- 仮想ネットワーク。
- 4 つのストレージ アカウント (Active Directory (AD) 用、SQL 用、ファイル共有監視用、および VM 診断用にそれぞれ 1 つ)。
- 4 つのパブリック IP アドレス (AD 用に 1 つ、各 SQL VM 用に 2 つ、SQL AlwaysOn リスナーにバインドされたパブリック ロード バランサー用に 1 つ)。
- パブリック IP が SQL AlwaysOn リスナーにバインドされた SQL VM 用の 1 つの外部ロード バランサー。
- 単一のドメインを使用する新しいフォレスト用のドメイン コントローラーとして構成された 1 台の VM (Windows Server 2016)。
- SQL Server 2016 SP1 または SP2 Enterprise または Developer エディションで構成され、クラスター化されている 2 つの VM (Windows Server 2016)。 これらは Marketplace イメージである必要があります。
- クラスターのファイル共有監視として構成された 1 台の VM (Windows Server 2016)。
- SQL およびファイル共有監視の VM を含む 1 つの可用性セット。

1. 
   [!INCLUDE [azs-user-portal](../includes/azs-user-portal.md)]

2. **\+** **[リソースの作成]**  >  **[カスタム]** の順に選択し、 **[テンプレートのデプロイ]** を選択します。

   ![Azure Stack Hub 管理者ポータルでのカスタム テンプレートのデプロイ](media/azure-stack-tutorial-sqlrp/aoag-template-deployment-1.png)

3. **[カスタム デプロイ]** ブレード上で **[テンプレートの編集]**  >  **[クイック スタート テンプレート]** の順に選択し、利用可能なカスタム テンプレートのドロップダウン リストを使用して **sql-2016-alwayson** テンプレートを選択します。 **[OK]** 、 **[保存]** の順に選択します。

   [![Azure Stack Hub 管理者ポータル上でテンプレートを編集する](media/azure-stack-tutorial-sqlrp/aoag-template-deployment-2.png "クイックスタート テンプレートを選択する")](media/azure-stack-tutorial-sqlrp/aoag-template-deployment-2.png#lightbox)

4. **[カスタム デプロイ]** ブレードで、 **[パラメーターの編集]** を選択し、既定値を確認します。 必要に応じて値を変更し、必要なすべてのパラメーター情報を入力してから、 **[OK]** を選択します。

    少なくとも以下を実行します。
    - ADMINPASSWORD、SQLSERVERSERVICEACCOUNTPASSWORD、および SQLAUTHPASSWORD パラメーターに複雑なパスワードを指定します。
    - DNSSUFFIX パラメーターの場合、すべての小文字で逆引き参照の DNS サフィックスを入力します (ASDK インストールの場合は、**azurestack.external**)。
    
   [![Azure Stack Hub 管理者ポータル上でパラメーターを編集する](media/azure-stack-tutorial-sqlrp/aoag-template-deployment-3.png "カスタム デプロイ パラメーターを編集する")](media/azure-stack-tutorial-sqlrp/aoag-template-deployment-3.png#lightbox)

5. **[カスタム デプロイ]** ブレードで、使用するサブスクリプションを選択し、カスタム デプロイ用に新しいデプロイ グループを作成するか、既存のリソース グループを選択します。

    次に、リソース グループの場所 (ASDK のインストールの場合は **local**) を選択し、 **[作成]** をクリックします。 カスタムのデプロイ設定が検証され、デプロイが開始されます。

    [![Azure Stack Hub 管理者ポータル上でサブスクリプションを選択する](media/azure-stack-tutorial-sqlrp/aoag-template-deployment-4.png "カスタム デプロイを作成する")](media/azure-stack-tutorial-sqlrp/aoag-template-deployment-4.png#lightbox)

6. ユーザー ポータルで、 **[リソース グループ]** を選択してから、カスタム デプロイ用に作成したリソース グループの名前を選択します (この例では、**resource-group**)。 すべてのデプロイが正常に完了したことを確認するために、デプロイの状態を表示します。
    
    次に、リソース グループ項目を確認して、**SQLPIPsql\<resource group name\>** パブリック IP アドレス項目を選択します。 ロードバランサー パブリック IP のパブリック IP アドレスと完全な FQDN をメモします。 この SQL AlwaysOn 可用性グループを利用する SQL ホスティング サーバーを作成できるようにするには、これを Azure Stack Hub オペレーターに提供する必要があります。

   > [!NOTE]
   > テンプレートのデプロイには数時間かかる場合があります。

### <a name="enable-automatic-seeding"></a>自動シード処理を有効にする

テンプレートによる SQL AlwaysOn 可用性グループのデプロイと構成が正常に完了した後に、可用性グループの SQL Server の各インスタンスで[自動シード処理](/sql/database-engine/availability-groups/windows/automatically-initialize-always-on-availability-group)を有効にする必要があります。

自動シード処理を使用して可用性グループを作成すると、他の手動作業を必要とせずに、SQL Server によって、グループ内のすべてのデータベースに対するセカンダリ レプリカが自動的に作成されます。 この方法によって、AlwaysOn データベースの高可用性が保証されます。

これらの SQL コマンドを使用して、AlwaysOn 可用性グループの自動シード処理を構成します。 必要に応じて、`<PrimaryInstanceName>` をプライマリ インスタンス SQL Server 名に、`<SecondaryInstanceName>` をセカンダリ インスタンス SQL Server 名に、`<availability_group_name>` を AlwaysOn 可用性グループ名に置換してください。

プライマリ SQL インスタンスの場合:

  ```sql
  ALTER AVAILABILITY GROUP [<availability_group_name>]
      MODIFY REPLICA ON '<PrimaryInstanceName>'
      WITH (SEEDING_MODE = AUTOMATIC)
  GO
  
  ALTER AVAILABILITY GROUP [<availability_group_name>]
      MODIFY REPLICA ON '<SecondaryInstanceName>'
      WITH (SEEDING_MODE = AUTOMATIC)
  GO
  ```

![プライマリ SQL インスタンス スクリプト](./media/azure-stack-tutorial-sqlrp/sql1.png)

セカンダリ SQL インスタンスの場合:

  ```sql
  ALTER AVAILABILITY GROUP [<availability_group_name>] GRANT CREATE ANY DATABASE
  GO
  ```

![セカンダリ SQL インスタンス スクリプト](./media/azure-stack-tutorial-sqlrp/sql2.png)

### <a name="configure-contained-database-authentication"></a>包含データベース認証を構成する

包含データベースを可用性グループに追加する前に、包含データベース認証サーバー オプションが、可用性グループの可用性レプリカをホストするすべてのサーバー インスタンスで 1 に設定されていることを確認します。 詳細については、[包含データベースの認証](/sql/database-engine/configure-windows/contained-database-authentication-server-configuration-option)に関するページを参照してください。

これらのコマンドを使用して、可用性グループ内の SQL Server インスタンスごとに包含データベース認証サーバー オプションを設定します。

  ```sql
  EXEC sp_configure 'contained database authentication', 1
  GO
  RECONFIGURE
  GO
  ```

![包含データベースの認証を設定する](./media/azure-stack-tutorial-sqlrp/sql3.png)

## <a name="configure-an-azure-stack-hub-sql-hosting-server"></a>Azure Stack Hub SQL ホスティング サーバーを構成する

SQL Server AlwayOn 可用性グループが作成され、適切に構成されたら、Azure Stack Hub オペレーターは Azure Stack Hub SQL ホスティング サーバーとしてこれを構成する必要があります。 

以前に SQL AlwaysOn 可用性グループのリソース グループが作成されたときにメモした SQL ロード バランサーのパブリック IP またはパブリック IP の完全な FQDN を必ず使用します (**SQLPIPsql\<resource group name\>** )。 また、AlwaysOn 可用性グループの SQL インスタンスにアクセスするために使用される SQL Server 認証資格情報を知る必要があります。

> [!NOTE]
> この手順は、Azure Stack Hub オペレーターが Azure Stack Hub 管理者ポータルから実行する必要があります。

Azure Stack Hub オペレーターは、SQL AlwaysOn 可用性グループのロード バランサー リスナーのパブリック IP と SQL 認証ログイン情報を利用し、[SQL AlwaysOn 可用性グループを使用して SQL ホスティング サーバーを作成](azure-stack-sql-resource-provider-hosting-servers.md#provide-high-availability-using-sql-always-on-availability-groups)できます。 

また、SQL AlwaysOn データベースをユーザーが作成できるようにするプランとオファーが作成してあることも確認してください。 Operator は **Microsoft.SqlAdapter** サービスをプランに追加し、高可用性データベース専用の新しいクォータを作成する必要があります。 プランの作成の詳細については、「[サービス、プラン、オファー、サブスクリプションの概要](service-plan-offer-subscription-overview.md)」を参照してください。

> [!TIP]
> **Microsoft.SqlAdapter** サービスは、[SQL Server リソース プロバイダーがデプロイされる](azure-stack-sql-resource-provider-deploy.md)までプランに追加することができません。

## <a name="create-a-highly-available-sql-database"></a>高可用性 SQL データベースを作成する

SQL AlwaysOn 可用性グループが Azure Stack Hub オペレーターによって Azure Stack Hub SQL ホスティング サーバーとして作成、構成、追加された後、SQL Server データベース機能を含むサブスクリプションを持つテナント ユーザーは、AlwaysOn 機能をサポートする SQL データベースを作成できます。 このセクションの手順に従って、それらのデータベースを作成できます。

> [!NOTE]
> これらの手順は、SQL Server 機能 (Microsoft.SQLAdapter サービス) を提供するサブスクリプションを持つテナント ユーザーとして、Azure Stack Hub ユーザー ポータルから実行します。

1. 
   [!INCLUDE [azs-user-portal](../includes/azs-user-portal.md)]

2. **[\+** **リソースの作成]**  >  **[データ \+ ストレージ]** の順に選択し、 **[SQL Database]** を選択します。

    必要なデータベース プロパティ情報を指定します。 この情報には、名前、照合順序、最大サイズ、サブスクリプション、リソース グループ、およびデプロイに使用する場所が含まれます。

   ![Azure Stack Hub ユーザー ポータル上で SQL データベースを作成する](./media/azure-stack-tutorial-sqlrp/createdb1.png)

3. **[SKU]** を選択し、使用する適切な SQL ホスティング サーバー SKU を選択します。 この例では、Azure Stack Hub オペレーターは、SQL AlwaysOn 可用性グループの高可用性をサポートするために、**Enterprise HA** SKU を作成しました。

   ![Azure Stack Hub ユーザー ポータル上で SKU を選択する](./media/azure-stack-tutorial-sqlrp/createdb2.png)

4. **[ログイン]**  >  **[Create a new login]\(新しいログインの作成\)** の順に選択し、新しいデータベースに使用する SQL の認証資格情報を入力します。 完了したら **[OK]** 、 **[作成]** の順に選択し、データベースのデプロイ プロセスを開始します。

   ![Azure Stack Hub ユーザー ポータル上でログインを作成する](./media/azure-stack-tutorial-sqlrp/createdb3.png)

5. SQL データベースのデプロイが正常に完了したら、データベースのプロパティを確認し、新しい高可用性データベースへの接続に使用する接続文字列を見つけます。

   ![Azure Stack Hub ユーザー ポータル上で接続文字列を表示する](./media/azure-stack-tutorial-sqlrp/createdb4.png)

## <a name="next-steps"></a>次のステップ

[SQL リソース プロバイダーを更新する](azure-stack-sql-resource-provider-update.md)
