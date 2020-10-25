---
title: 高可用性 MySQL データベースの作成
titleSuffix: Azure Stack Hub
description: Azure Stack Hub で MySQL Server リソース プロバイダーのホスト コンピューターと高可用性 MySQL データベースを作成する方法について説明します。
author: BryanLa
ms.topic: article
ms.date: 10/07/2019
ms.author: bryanla
ms.reviewer: xiaofmao
ms.lastreviewed: 10/23/2019
ms.openlocfilehash: 52dee37571715f6093609609b4f31a45739df1f7
ms.sourcegitcommit: 69cfff119ab425d0fbb71e38d1480d051fc91216
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/30/2020
ms.locfileid: "91572910"
---
# <a name="create-highly-available-mysql-databases"></a>高可用性 MySQL データベースの作成

Azure Stack Hub オペレーターとして、MySQL Server データベースをホストするようにサーバーの仮想マシン (VM) を構成できます。 MySQL クラスターが正常に作成され、Azure Stack Hub によって管理されると、MySQL サービスにサブスクライブしているユーザーは高可用性 MySQL データベースを簡単に作成できます。

この記事では、Azure Stack Marketplace の項目を使用して[レプリケーション クラスター付きの MySQL](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/bitnami.mysql-cluster) を作成する方法について説明します。 このソリューションは、複数の VM を使用して、マスター ノードから構成可能な数のレプリカにデータベースをレプリケートします。 作成したクラスターは、Azure Stack Hub MySQL ホスティング サーバーとして追加することができます。また、ユーザーは高可用性 MySQL データベースを作成できるようになります。

> [!IMPORTANT]
> **レプリケーション付きの MySQL** Azure Stack Marketplace の項目は、すべての Azure クラウド サブスクリプション環境では利用できない場合があります。 このチュートリアルの以降の部分に進む前に、ご自身のサブスクリプションでマーケットプレースの項目を入手できることを確認してください。

ここでは、次の内容について学習します。

> [!div class="checklist"]
> * マーケットプレースの項目から MySQL Server クラスターを作成する。
> * MySQL Server クラスターを Azure Stack Hub MySQL ホスティング サーバーとして構成する。
> * 高可用性 MySQL データベースを作成する。

利用可能な Azure Stack Marketplace の項目を使用して、3 つの VM MySQL Server クラスターを作成および構成します。

開始する前に、[MySQL Server リソース プロバイダー](azure-stack-mysql-resource-provider-deploy.md)が正常にインストールされ、Azure Stack Marketplace で次の項目を利用できることを確認してください。

> [!IMPORTANT]
> MySQL クラスターを作成するには、以下のすべてが必要です。

- [レプリケーション付きの MySQL](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/bitnami.mysql-cluster): これは、MySQL クラスターのデプロイに使用される Bitnami ソリューション テンプレートです。
- [Debian 8 "Jessie"](https://azuremarketplace.microsoft.com/marketplace/apps/credativ.Debian): credativ によって提供されている Microsoft Azure のバックポート カーネル付き Debian 8 "Jessie"。 Debian GNU/Linux は最も人気のある Linux ディストリビューションの 1 つです。
- [Linux 2.0 用のカスタム スクリプト](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/microsoft.custom-script-linux?tab=Overview): カスタム スクリプト拡張機能は、VM のプロビジョニング後に VM のカスタマイズ タスクを実行するためのツールです。 この拡張機能が VM に追加されると、Azure ストレージからスクリプトをダウンロードして VM 上で実行できます。 カスタム スクリプト拡張タスクは、Azure PowerShell コマンドレットと Azure クロスプラットフォーム コマンドライン インターフェイス (xPlat CLI) を使用して自動化することもできます。
- Linux Extension 1.4.7 用 VM アクセス: VM アクセス拡張機能を使用すると、パスワード、SSH キー、または SSH 構成をリセットして、VM へのアクセスを再獲得することができます。 また、この拡張機能を使用して、パスワードまたは SSH キーを使用して新しいユーザーを追加したり、ユーザーを削除したりすることもできます。 この拡張機能は、Linux VM をターゲットとしています。

Azure Stack Marketplace に項目を追加する方法については、「[Azure Stack Marketplace の概要](azure-stack-marketplace.md)」を参照してください。

また、Linux VM のデプロイ後にログインするためには、[PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) などの SSH クライアントが必要になります。

## <a name="create-a-mysql-server-cluster"></a>MySQL Server クラスターを作成する。

このセクションの手順を使用して、[レプリケーション付き MySQL](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/bitnami.mysql-cluster) マーケットプレース項目を使用して MySQL Server クラスターをデプロイします。 このテンプレートで、高可用性 MySQL クラスターで構成された 3 つの MySQL Server インスタンスがデプロイされます。 既定で、次のリソースが作成されます。

- 仮想ネットワーク
- ネットワーク セキュリティ グループ
- ストレージ アカウント
- 可用性セット
- 3 つのネットワーク インターフェイス (既定の VM ごとに 1 つ)。
- パブリック IP アドレス (プライマリ MySQL クラスター VM の場合)
- MySQL クラスターをホストする 3 つの Linux VM

1. 
   [!INCLUDE [azs-admin-portal](../includes/azs-user-portal.md)]

2. サブスクリプションがまだ割り当てられていない場合は、ダッシュボードから **[サブスクリプションの取得]** を選択します。 ブレードで、サブスクリプションの名前を入力し、オファーを選択します。 誤って削除されないように、MySQL クラスターのデプロイは独自のサブスクリプションに保持することをお勧めします。

3. **[\+** **リソースの作成]**  >  **[計算]** の順に選択し、 **[MySQL with Replication]\(レプリケーション付き MySQL\)** を選択します。

   ![Azure Stack Hub でのカスタム テンプレートのデプロイ](media/azure-stack-tutorial-mysqlrp/img1.png)

4. **[基本]** ページで基本的なデプロイ情報を提供します。 既定値を確認し、必要に応じて変更して **[OK]** を選択します。

    少なくとも、以下の情報を指定します。

   - デプロイ名 (既定値は mymysql)。
   - アプリケーションのルート パスワード。 **特殊文字を含めずに** 12 文字の英数字のパスワードを入力します。
   - アプリケーション データベース名 (既定値は bitnami)。
   - 作成する MySQL データベースのレプリカ VM の数 (既定値は 2)。
   - 使用するサブスクリプションを選択します。
   - 使用するリソース グループを選択するか、新しく作成します。
   - 場所を選択します (ASDK の場合、既定値は local)。

     ![デプロイの基礎 — レプリケーション付きの MySQL の作成](media/azure-stack-tutorial-mysqlrp/img2.png)

5. **[Environment Configuration]\(環境の構成\)** ページに次の情報を入力し、 **[OK]** を選択します。

   - Secure Shell (SSH) 認証に使用するパスワードまたは SSH パブリック キー。 パスワードを使用する場合は、英字、数字を含める必要があります。また、特殊文字を含めることも**可能です**。
   - VM サイズ (既定値は Standard D1 v2 VM)。
   - データ ディスク サイズ (GB 単位)

     ![環境の構成 — レプリケーション付きの MySQL の作成](media/azure-stack-tutorial-mysqlrp/img3.png)

6. デプロイの **[概要]** を確認します。 必要に応じて、カスタマイズしたテンプレートとパラメーターをダウンロードして、 **[OK]** を選択します。

   ![概要 — レプリケーション付きの MySQL の作成](media/azure-stack-tutorial-mysqlrp/img4.png)

7. **[購入]** ページで **[作成]** を選択して、デプロイを開始します。

   ![[購入] ページ — レプリケーション付きの MySQL の作成](media/azure-stack-tutorial-mysqlrp/img5.png)

    > [!NOTE]
    > デプロイには約 1 時間かかります。 続行する前に、デプロイが完了し、MySQL クラスターの構成が完了していることを確認してください。

8. すべてのデプロイが正常に完了したら、リソース グループ項目を確認し、**mysqlip** パブリック IP アドレス項目を選択します。 クラスターのパブリック IP アドレスと完全な FQDN をメモします。

    この MySQL クラスターを利用する MySQL ホスティング サーバーを作成できるように、この IP アドレスを Azure Stack Hub オペレーターに提供する必要があります。

### <a name="create-a-network-security-group-rule"></a>ネットワーク セキュリティ グループ規則を作成する

既定では、MySQL からホスト VM へのパブリック アクセスは構成されません。 Azure Stack Hub MySQL リソース プロバイダーが MySQL クラスターを接続して管理するには、受信ネットワーク セキュリティ グループ (NSG) 規則を作成する必要があります。

1. 管理者ポータル上で、MySQL クラスターのデプロイ時に作成されたリソース グループに移動して、ネットワーク セキュリティ グループ (**default-subnet-sg**) を選択します。

   ![Azure Stack Hub 管理者ポータル上でネットワーク セキュリティ グループを選択します](media/azure-stack-tutorial-mysqlrp/img6.png)

2. **[受信セキュリティ規則]** を選択してから、 **[追加]** を選択します。

    **[宛先ポート範囲]** に「**3306**」と入力し、必要に応じて **[名前]** フィールドと **[説明]** フィールドに説明を入力します。

   ![open](media/azure-stack-tutorial-mysqlrp/img7.png)

3. **[追加]** を選択して、受信セキュリティ規則のダイアログを閉じます。

### <a name="configure-external-access-to-the-mysql-cluster"></a>MySQL クラスターへの外部アクセスを構成する

MySQL クラスターを Azure Stack Hub MySQL Server ホストとして追加する前に、外部アクセスを有効にする必要があります。

1. SSH クライアントを使用して (この例では [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) を使用)、パブリック IP にアクセスできるコンピューターからプライマリ MySQL マシンにログインします。 通常、プライマリ MySQL VM 名は末尾が **0** であり、パブリック IP が割り当てられています。

    パブリック IP を使用して、ユーザー名 **bitnami** と、前に作成した特殊文字を含まないアプリケーション パスワードを入力して、VM にログインします。

   ![LinuxLogin](media/azure-stack-tutorial-mysqlrp/bitnami1.png)

2. SSH クライアント ウィンドウで、次のコマンドを使用して、bitnami サービスがアクティブで実行中であることを確認します。 プロンプトが表示されたら、もう一度 bitnami パスワードを入力します。

   `sudo service bitnami status`

   ![bitnami サービスを確認する](media/azure-stack-tutorial-mysqlrp/bitnami2.png)

3. Azure Stack Hub MySQL ホスティング サーバーが MySQL に接続し、SSH クライアントを終了するために使用するリモート アクセス ユーザー アカウントを作成します。

    前に作成したルート パスワードを使用して、次のコマンドを実行して MySQL に root としてログインします。 新しい管理者ユーザーを作成し、お使いの環境に合わせて *\<username\>* と *\<password\>* を置き換えます。 この例では、作成されたユーザーの名前は **sqlsa** であり、強力なパスワードが使用されています。

   ```mysql
   mysql -u root -p
   create user <username>@'%' identified by '<password>';
   grant all privileges on *.* to <username>@'%' with grant option;
   flush privileges;
   ```

   ![管理者ユーザーを作成する](media/azure-stack-tutorial-mysqlrp/bitnami3.png)

4. 新しい MySQL ユーザー情報をメモします。

    Azure Stack Hub オペレーターには、このユーザー名とパスワードと共に、クラスターのパブリック IP アドレスまたはパブリック IP の完全な FQDN を渡し、この MySQL クラスターを使用して MySQL ホスティング サーバーを作成できるようにする必要があります。

## <a name="configure-an-azure-stack-hub-mysql-hosting-server"></a>Azure Stack Hub MySQL ホスティング サーバーを構成する

MySQL Server クラスターが作成されて適切に構成されたら、Azure Stack Hub オペレーターはそれを Azure Stack Hub MySQL ホスティング サーバーとして追加する必要があります。

以前に MySQL クラスターのリソース グループを作成したときにメモした、MySQL クラスターのプライマリ VM のパブリック IP、またはパブリック IP の完全な FQDN を必ず使用してください (**mysqlip**)。 また、オペレーターは、MySQL クラスター データベースにリモート アクセスするために作成した MySQL Server の認証資格情報を知っておく必要があります。

> [!NOTE]
> この手順は、Azure Stack Hub オペレーターが Azure Stack Hub 管理者ポータルから実行する必要があります。

Azure Stack Hub オペレーターは、MySQL クラスターのパブリック IP と MySQL の認証ログイン情報を利用して、[新しい MySQL クラスターを使用して MySQL ホスティング サーバーを作成](azure-stack-mysql-resource-provider-hosting-servers.md#connect-to-a-mysql-hosting-server)できるようになりました。

また、MySQL データベースの作成をユーザーが利用できるプランとオファーが作成済みであることも確認してください。 Operator は **Microsoft.MySqlAdapter** サービスをプランに追加し、高可用性データベース専用の新しいクォータを作成する必要があります。 プランの作成の詳細については、「[サービス、プラン、オファー、サブスクリプションの概要](service-plan-offer-subscription-overview.md)」を参照してください。

> [!TIP]
> **Microsoft.MySqlAdapter** サービスは、[MySQL Server リソース プロバイダーがデプロイされる](azure-stack-mysql-resource-provider-deploy.md)までプランに追加することができません。

## <a name="create-a-highly-available-mysql-database"></a>高可用性 MySQL データベースを作成する

MySQL クラスターが Azure Stack Hub オペレーターによって Azure Stack Hub MySQL ホスティング サーバーとして作成、構成、追加された後、MySQL Server データベース機能を含むサブスクリプションを持つテナント ユーザーは、このセクションの手順に従って、高可用性 MySQL データベースを作成できます。

> [!NOTE]
> これらの手順は、MySQL Server 機能 (Microsoft.MySQLAdapter サービス) を提供するサブスクリプションを持つテナント ユーザーとして、Azure Stack Hub ユーザー ポータルから実行します。

1. 
   [!INCLUDE [azs-user-portal](../includes/azs-user-portal.md)]

2. **[\+** **リソースの作成]**  >  **[データ \+ ストレージ]** の順に選択し、 **[MySQL Database]** を選択します。

    名前、照合順序、使用するサブスクリプション、デプロイに使用する場所などの必須のデータベースのプロパティを指定します。

    ![Azure Stack Hub ユーザー ポータル上で MySQL データベースを作成する](./media/azure-stack-tutorial-mysqlrp/createdb1.png)

3. **[SKU]** を選択し、使用する適切な MySQL ホスティング サーバー SKU を選択します。 この例では、Azure Stack Hub オペレーターは、MySQL クラスター データベースの高可用性をサポートするために **MySQL-HA** SKU を作成しました。

   ![Azure Stack Hub ユーザー ポータル上で SKU を選択する](./media/azure-stack-tutorial-mysqlrp/createdb2.png)

4. **[ログイン]**  >  **[Create a new login]\(新しいログインの作成\)** の順に選択し、新しいデータベースに使用する MySQL の認証資格情報を入力します。 完了したら **[OK]** 、 **[作成]** の順に選択し、データベースのデプロイ プロセスを開始します。

   ![Azure Stack Hub ユーザー ポータル上でログインを追加する](./media/azure-stack-tutorial-mysqlrp/createdb3.png)

5. MySQL データベースのデプロイが正常に完了したら、データベースのプロパティを確認し、新しい高可用性データベースへの接続に使用する接続文字列を見つけます。

   ![Azure Stack Hub ユーザー ポータル上で接続文字列を表示する](./media/azure-stack-tutorial-mysqlrp/createdb4.png)

## <a name="next-steps"></a>次のステップ

[MySQL リソースプロバイダーを更新する](azure-stack-mysql-resource-provider-update.md)
