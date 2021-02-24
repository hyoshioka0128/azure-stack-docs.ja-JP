---
title: Azure Stack Hub に MySQL ホスティング サーバーを追加する
description: MySQL アダプター リソース プロバイダーを使用したプロビジョニングのために MySQL ホスティング サーバーを追加する方法について説明します。
author: bryanla
ms.topic: article
ms.date: 12/07/2020
ms.author: bryanla
ms.reviewer: xiaofmao
ms.lastreviewed: 12/07/2020
ms.openlocfilehash: a5c965591a6eb7d11540bf63c298ffa2321e0014
ms.sourcegitcommit: 62eb5964a824adf7faee58c1636b17fedf4347e9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/07/2020
ms.locfileid: "96778429"
---
# <a name="add-mysql-hosting-servers-in-azure-stack-hub"></a>Azure Stack Hub に MySQL ホスティング サーバーを追加する

MySQL リソース プロバイダーが SQL インスタンスに接続できる限り、その MySQL ホスティング サーバー インスタンスを、[Azure Stack Hub](azure-stack-overview.md) 内の仮想マシン (VM) またはご利用の Azure Stack Hub 環境の外部にある VM でホストできます。

> [!NOTE]
> MySQL リソース プロバイダーは既定のプロバイダー サブスクリプションに作成する必要がありますが、MySQL ホスティング サーバーは課金対象のユーザー サブスクリプションに作成する必要があります。 リソース プロバイダー サーバーは、ユーザー データベースをホストするためには使用しないでください。

ホスティング サーバーには、MySQL バージョン 5.6、5.7、および 8.0 を使用できます。 MySQL RP では、caching_sha2_password 認証はサポートされていません。 mysql_native_password を使用するには、MySQL 8.0 サーバーを構成する必要があります。

## <a name="prepare-a-mysql-hosting-server"></a>MySQL ホスティング サーバーを準備する

### <a name="create-a-network-security-group-rule"></a>ネットワーク セキュリティ グループ規則を作成する

既定では、MySQL からホスト VM へのパブリック アクセスは構成されません。 Azure Stack Hub MySQL リソース プロバイダーが MySQL サーバーを接続して管理するには、受信ネットワーク セキュリティ グループ (NSG) 規則を作成する必要があります。

1. 管理者ポータル上で、MySQL サーバーのデプロイ時に作成されたリソース グループに移動して、ネットワーク セキュリティ グループ (**default-subnet-sg**) を選択します。

   ![Azure Stack Hub 管理者ポータル上でネットワーク セキュリティ グループを選択します](media/azure-stack-tutorial-mysqlrp/img6.png)

2. **[受信セキュリティ規則]** を選択してから、 **[追加]** を選択します。

    **[宛先ポート範囲]** に「**3306**」と入力し、必要に応じて **[名前]** フィールドと **[説明]** フィールドに説明を入力します。

   ![open](media/azure-stack-tutorial-mysqlrp/img7.png)

3. **[追加]** を選択して、受信セキュリティ規則のダイアログを閉じます。

### <a name="configure-external-access-to-the-mysql-hosting-server"></a>MySQL ホスティング サーバーへの外部アクセスを構成する

MySQL サーバーを Azure Stack Hub MySQL Server ホストとして追加する前に、外部アクセスを有効にする必要があります。 Azure Stack Hub マーケットプレースで入手できる Bitnami MySQL を例にとると、次の手順を行って外部アクセスを構成できます。

1. SSH クライアントを使用して (この例では [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) を使用)、パブリック IP にアクセスできるコンピューターから MySQL サーバーにログインします。

    パブリック IP を使用して、ユーザー名と、前に作成した特殊文字を含まないアプリケーション パスワードを入力して、VM にログインします。

   ![LinuxLogin](media/azure-stack-tutorial-mysqlrp/bitnami1.png)

2. SSH クライアント ウィンドウで、次のコマンドを使用して、bitnami サービスがアクティブで実行中であることを確認します。 プロンプトが表示されたら、もう一度 bitnami パスワードを入力します。

   `sudo service bitnami status`

   ![bitnami サービスを確認する](media/azure-stack-tutorial-mysqlrp/bitnami2.png)

3. MySQL ホスティング サーバーがバージョン 8.0 以上の場合、認証方法を **mysql_native_password** に変更する必要があります。 MySQL のバージョンが 8.0 より下の場合、この手順はスキップできます。

   Bitnami MySQL を例に取ると、構成ファイルは **/opt/bitnami/mysql/conf/my.cnf** にあります。 プロパティ **default_authentication_plugin** を値 **mysql_native_password** で設定します。
   ```
   [mysqld]
   default_authentication_plugin=mysql_native_password
   ```
   bitnami サービスを再起動し、bitnami が正しく動作していることを確認します。
   ```console
   sudo service bitnami restart
   sudo service bitnami status
   ```

4. Azure Stack Hub MySQL ホスティング サーバーが MySQL に接続するために使用するリモート アクセス ユーザー アカウントを作成します。

    *~/bitnami_credentials* で記録したルート パスワードを使用し、次のコマンドを実行して MySQL に root としてログインします。 新しい管理者ユーザーを作成し、お使いの環境に合わせて *\<username\>* と *\<password\>* を置き換えます。 この例では、作成されたユーザーの名前は **sqlsa** であり、強力なパスワードが使用されています。

   ```sql
   mysql -u root -p
   create user <username>@'%' identified by '<password>';
   grant all privileges on *.* to <username>@'%' with grant option;
   flush privileges;
   ```

   ![管理者ユーザーを作成する](media/azure-stack-tutorial-mysqlrp/bitnami3.png)

5. 作成された SQL ユーザー **sqlsa** のプラグインが **mysql_native_password** であることを確認し、SSH クライアントを終了します。
   
   ```sql
   SELECT user,host,plugin from mysql.user;
   ```
6. 新しい MySQL ユーザー情報をメモします。

   このユーザー名とパスワードは、Azure Stack Hub オペレーターがこの MySQL サーバーを使用して MySQL ホスティング サーバーを作成するときに使用されます。


## <a name="connect-to-a-mysql-hosting-server"></a>MySQL ホスティング サーバーに接続する

システム管理特権を持つアカウントの資格情報があることを確認してください。

> [!NOTE]
> MySQL 8.0 以降のバージョンでは、リモート アクセスは既定では有効になっていません。 ホスティング サーバーとして追加する前に、新しいユーザー アカウントを作成し、このユーザー アカウントにリモート アクセスの特権を付与する必要があります。

ホスティング サーバーを追加するには、次の手順に従います。

1. Azure Stack Hub 管理ポータルにサービス管理者としてサインインします。
2. **[すべてのサービス]** を選択します。
3. **[管理リソース]** カテゴリで **[MySQL ホスティング サーバー]**  >  **[+追加]** を選択します。 次の画面キャプチャに示される **[MySQL ホスティング サーバーの追加]** ダイアログが開きます。

   ![MySQL ホスティング サーバーを構成する](./media/azure-stack-mysql-rp-deploy/mysql-add-hosting-server-2.png)

4. MySQL サーバー インスタンスの接続詳細を指定します。

   * **[MySQL Hosting Server Name]\(MySQL ホスティング サーバー名\)** では、完全修飾ドメイン名 (FQDN) または有効な IPv4 アドレスを指定します。 短い VM 名は使用しないでください。
   * Azure Stack Hub Marketplace で利用可能な Bitnami MySQL イメージ用の既定の管理者 **ユーザー名** は *root* です。
   * root の **パスワード** がわからない場合は、[Bitnami のドキュメント](https://docs.bitnami.com/azure/faq/#how-to-find-application-credentials)で取得方法を確認してください。
   * 既定の MySQL インスタンスが指定されていないため、 **[ホスティング サーバーのサイズ (GB)]** を指定する必要があります。 データベース サーバーの容量に近いサイズを入力します。
   * **[サブスクリプション]** の既定の設定のままにします。
   * **[リソース グループ]** で、新しいリソース グループを作成するか、既存のグループを使用します。

   > [!IMPORTANT]
   > デプロイ中に MySQL リソース プロバイダーのインストーラーによって作成された **リソース グループ** `system.<region>.sqladapter` は選択しないでください。 ホスティング サーバー用に別のリソース グループを用意する必要があります。    

   > [!NOTE]
   > テナントと管理者の Azure Resource Manager から MySQL インスタンスにアクセスできる場合は、そのインスタンスをリソース プロバイダーの管理下に置くことができます。 ただし、MySQL インスタンスは、リソース プロバイダーに排他的に割り当てる **必要があります**。

5. **[SKU]** を選択して **[Create SKU]\(SKU の作成\)** ダイアログを開きます。

   ![MySQL SKU を作成する](./media/azure-stack-mysql-rp-deploy/mysql-new-sku.png)

   ユーザーが自分のデータベースを適切な SKU にデプロイできるように、SKU の **[名前]** には SKU のプロパティが反映される必要があります。

6. **[OK]** を選択して SKU を作成します。
   > [!NOTE]
   > SKU はポータルに表示されるまで最大 1 時間かかることがあります。 SKU がデプロイされて実行されるまで、データベースを作成することはできません。

7. **[Add a MySQL Hosting Server]\(MySQL ホスティング サーバーの追加\)** の下で、 **[作成]** を選択します。

サーバーを追加する際に、サービス内容を区別するために新規または既存の SKU に割り当てます。 たとえば、増加したデータベースと自動バックアップを提供する MySQL エンタープライズ インスタンスを使用できます。 この高パフォーマンス サーバーは、組織内のさまざまな部門用に予約できます。

## <a name="security-considerations-for-mysql"></a>MySQL のセキュリティに関する考慮事項

次の情報は、RP および MySQL ホスティング サーバーに適用されます。

* すべてのホスティング サーバーが TLS 1.1 を使用した通信用に構成されていることを確認します。 [暗号化された接続を使用するための MySQL の構成](https://dev.mysql.com/doc/refman/5.7/en/using-encrypted-connections.html)に関するページを参照してください。
* [Transparent Data Encryption](https://dev.mysql.com/doc/mysql-secure-deployment-guide/5.7/en/secure-deployment-data-encryption.html) を採用します。
* MySQL RP では、caching_sha2_password 認証はサポートされていません。

## <a name="increase-backend-database-capacity"></a>バックエンド データベース容量を増やす

バックエンド データベース容量を増加するには、Azure Stack Hub ポータルで追加で MySQL サーバーをデプロイします。 これらのサーバーを新規または既存の SKU に追加します。 サーバーを既存の SKU に追加する場合は、サーバーの特性が SKU 内の他のサーバーと同じであることを確認してください。

## <a name="sku-notes"></a>SKU に関する注意
使用する SKU 名は、SKU 内のサーバーの機能 (容量やパフォーマンス) を示すものにしてください。 この名前は、該当する SKU にユーザーがデータベースをデプロイするときの助けになります。 たとえば、SKU 名を使用して、サービス内容を次の特性によって区別することもできます。
  
* 大容量
* ハイ パフォーマンス
* 高可用性

ベスト プラクティスとして、SKU 内のすべてのホスティング サーバーに同じリソースとパフォーマンス特性を持たせてください。

SKU は、特定のユーザーやグループに割り当てることはできません。

SKU を編集するには、 **[すべてのサービス]**  >  **[MySQL アダプター]**  >  **[SKU]** に移動します。 変更する SKU を選択し、必要な変更を加えた後、 **[保存]** をクリックして変更を保存します。 

不要になった SKU を削除するには、 **[すべてのサービス]**  >  **[MySQL アダプター]**  >  **[SKU]** に移動します。 SKU 名を右クリックし、 **[削除]** を選択して削除します。

> [!IMPORTANT]
> 新しい SKU がユーザー ポータルで使用可能になるまでに最大で 1 時間かかることがあります。

## <a name="make-mysql-database-servers-available-to-your-users"></a>ユーザーが MySQL データベース サーバーを使用できるようにする

ユーザーが MySQL データベース サーバーを使用できるようにプランとオファーを作成します。 プランに Microsoft.MySqlAdapter サービスを追加し、新しいクォータを作成します。 MySQL では、データベースのサイズの制限が許可されません。

> [!IMPORTANT]
> ユーザー ポータルで新しいクォータが使用可能になるまで、または変更されたクォータが適用されるまでに最大で 2 時間かかることがあります。

## <a name="next-steps"></a>次のステップ

[MySQL データベースの作成](azure-stack-mysql-resource-provider-databases.md)
