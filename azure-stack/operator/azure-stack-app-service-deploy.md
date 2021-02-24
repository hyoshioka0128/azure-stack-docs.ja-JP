---
title: Azure Stack Hub に App Service をデプロイする
description: Azure Stack Hub に App Service をデプロイする方法について説明します。
author: bryanla
ms.topic: article
ms.date: 10/28/2020
ms.author: bryanla
ms.reviewer: anwestg
ms.lastreviewed: 10/28/2020
zone_pivot_groups: state-connected-disconnected
ms.openlocfilehash: 78457d49b35c49cefc458070dc8c1f41391eb9eb
ms.sourcegitcommit: b50dd116d6d1f89d42bd35ad0f85bb25c5192921
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/26/2020
ms.locfileid: "94544488"
---
# <a name="deploy-app-service-in-azure-stack-hub"></a>Azure Stack Hub に App Service をデプロイする

[!INCLUDE [Azure Stack hub update reminder](../includes/app-service-hub-update-banner.md)]

> [!IMPORTANT]
> リソース プロバイダーのインストーラーを実行する前に、[前提条件](azure-stack-app-service-before-you-get-started.md)に関する手順を完了する必要があります

::: zone pivot="state-connected"
この記事では、Azure Stack Hub に App Service をデプロイする方法について説明します。これにより、ユーザーは Web、API、および Azure Functions アプリケーションを作成できるようになります。 以下を実行する必要があります。

- この記事で説明する手順を使用して、Azure Stack Hub デプロイに [App Service リソース プロバイダー](azure-stack-app-service-overview.md)を追加します。
- App Service リソース プロバイダーをインストールした後で、オファーやプランに含めることができます。 ユーザーはサブスクライブしてサービスを取得し、アプリケーションの作成を開始できます。

## <a name="run-the-app-service-resource-provider-installer"></a>App Service リソースプロバイダーのインストーラーを実行する

App Service リソース プロバイダーのインストールには少なくとも 1 時間かかります。 必要な時間の長さは、デプロイするロール インスタンスの数によって異なります。 デプロイ中に、インストーラーは次のタスクを実行します。

- [既定のプロバイダー サブスクリプション] で必要なリソース プロバイダーを登録する
- App Service ID アプリケーションへの共同作成者のアクセス権を付与する
- リソース グループと仮想ネットワークを作成する (必要な場合)
- App Service インストール アーティファクト、使用状況サービス、リソース ハイドレーション用のストレージ アカウントとコンテナーを作成する
- App Service アーティファクトをダウンロードし、App Service ストレージア カウントにアップロードする
- App Service をデプロイする
- 使用状況サービスを登録する
- App Service の DNS エントリを作成する
- App Service の管理者とテナント リソース プロバイダーを登録する
- ギャラリー項目 (Web、API、関数アプリ、App Service プラン、WordPress、DNN、Orchard、Django アプリケーション) を登録する

App Service リソースプロバイダーをデプロイするには、次の手順を実行します。

1. Azure Stack Hub 管理の Azure Resource Management エンドポイントにアクセスできるコンピューターから、appservice.exe を管理者として実行します。

2. **[Deploy App Service or upgrade to the latest version]\(App Service をデプロイするか、または最新バージョンにアップグレードする\)** を選択します。

    ![Azure App Service インストーラーのメイン画面を示すスクリーンショット。][1]

3. マイクロソフト ソフトウェア ライセンス条項を確認して同意し、 **[次へ]** を選択します。

4. サード パーティのライセンス条項を確認して同意し、 **[次へ]** を選択します。

5. App Service クラウド構成情報が正しいことを確認します。 ASDK のデプロイ中に既定の設定を使用した場合は、既定値をそのまま使用することもできます。 しかし、ASDK のデプロイ時にオプションをカスタマイズした場合、または Azure Stack Hub 統合システムにデプロイしている場合は、その差異を反映するように、このウィンドウで値を編集する必要があります。

   たとえば、ドメイン サフィックス mycloud.com を使用する場合は、Azure Stack Hub テナントの Azure Resource Manager エンドポイントを management.&lt;region&gt;.mycloud.com に変更する必要があります。 これらの設定を確認し、 **[次へ]** を選択して設定を保存します。

   ![App Service の ARM エンドポイントを指定するための画面を示すスクリーンショット。][2]

6. App Service インストーラーの次のページで、Azure Stack Hub に接続します。

    1. 使用する接続方法 ( **[資格情報]** または **[サービス プリンシパル]** ) を選択します。
 
        - **資格情報**
            - Azure Active Directory (Azure AD) を使用している場合は、Azure Stack Hub のデプロイ時に指定した、Azure AD の管理者アカウントとパスワードを入力します。 **[接続]** を選択します。
            - Active Directory フェデレーション サービス (AD FS) を使用している場合は、ご自分の管理者アカウントを指定します。 たとえば、「 cloudadmin@azurestack.local 」のように入力します。 パスワードを入力し、 **[接続]** を選択します。

        - **サービス プリンシパル**
            - 使用するサービス プリンシパルには、**既定のプロバイダー サブスクリプション** に対する **所有者** 権限が **必要です**
            - **サービス プリンシパル ID**、**証明書ファイル**、および **パスワード** を指定し、 **[接続]** を選択します。

    1. **[Azure Stack Hub Subscriptions]\(Azure Stack Hub サブスクリプション\)** で、 **[Default Provider Subscription]\(既定のプロバイダー サブスクリプション\)** を選択します。  Azure App Service on Azure Stack Hub は、 **[Default Provider Subscription]\(既定のプロバイダー サブスクリプション\)** にデプロイする **必要** があります。

    1. **[Azure Stack Hub Locations]\(Azure Stack Hub の場所\)** で、デプロイしているリージョンに対応する場所を選択します。 たとえば、ASDK にデプロイしている場合は、 **[ローカル]** を選びます。

    ![App Service インストーラー内の、Azure Stack Hub のサブスクリプション情報を指定する場所を示すスクリーンショット。][3]

7. [ここで示されている手順を使用して](azure-stack-app-service-before-you-get-started.md#virtual-network)構成した既存の仮想ネットワークにデプロイするか、または、App Service インストーラーで仮想ネットワークとサブネットを作成できます。 VNet を作成するには、次の手順に従います。

   a. **[Create VNet with default settings]\(既定の設定で VNet を作成する\)** をオンにし、既定値を受け入れて **[次へ]** を選択します。

   b. または、 **[Use existing VNet and Subnets]\(既存の VNet とサブネットを使用する\)** をオンにします。 次の操作を実行します。

     - 仮想ネットワークが含まれている **リソース グループ** を選択します。
     - デプロイする **仮想ネットワーク** の名前を選択します。
     - 必要なロール サブネットそれぞれに適切な **サブネット** 値を選択します。
     - **[次へ]** を選択します。

   ![App Service インストーラー内の、仮想ネットワークを構成する画面を示すスクリーンショット。][4]

8. ファイル共有の情報を入力してから、 **[次へ]** を選択します。 ファイル共有のアドレスには、ファイル サーバーの完全修飾ドメイン名 (FQDN)、または IP アドレスを使用する必要があります。 たとえば、\\\appservicefileserver.local.cloudapp.azurestack.external\websites、または \\\10.0.0.1\websites を使用します。  ドメインに参加しているファイル サーバーを使用している場合は、ドメインを含む完全なユーザー名を指定する必要があります。 たとえば、myfileserverdomain\FileShareOwner です。

   >[!NOTE]
   >インストーラーは、続行する前にファイル共有への接続性をテストしようとします。 しかし、既存の仮想ネットワークにデプロイする場合、この接続テストが失敗する可能性があります。 警告と、続行するためのプロンプトが表示されます。 ファイル共有情報が正しい場合は、デプロイを続行します。

   ![App Service インストーラーでファイル サーバーのパスと資格情報を構成する画面を示すスクリーンショット][5]

9. 次の App Service インストーラー ページで、次の手順に従います。

   a. **[Identity Application ID]\(ID アプリケーションの ID\)** ボックスで、[前提条件](azure-stack-app-service-before-you-get-started.md)の一部として作成した ID アプリケーションの GUID を入力します。

   b. **[Identity Application certificate file]\(ID アプリケーションの証明書ファイル\)** ボックスで、証明書ファイルの場所を入力 (または参照) します。

   c. **[Identity Application certificate password]\(ID アプリケーションの証明書パスワード\)** ボックスで、証明書のパスワードを入力します。 このパスワードは、証明書作成のスクリプトを使ったときに書き留めたものです。

   d. **[Azure Resource Manager root certificate file]\(Azure Resource Manager のルート証明書ファイル\)** ボックスで、証明書ファイルの場所を入力 (または参照) します。

   e. **[次へ]** を選択します。

   ![App Service インストーラーで AAD/ADFS ID アプリケーションと Azure Stack Resource Manager 証明書の詳細を指定する画面を示すスクリーンショット][6]

10. 3 つの証明書ファイルの各ボックスで、 **[参照]** を選択し、適切な証明書ファイルに移動します。 各証明書のパスワードを入力する必要があります。 これらの証明書は、「[App Service on Azure Stack Hub のデプロイの前提条件](azure-stack-app-service-before-you-get-started.md)」で作成したものです。 すべての情報を入力したら、 **[次へ]** を選択します。

    | ボックス | 証明書ファイル名の例 |
    | --- | --- |
    | **App Service の既定の SSL 証明書ファイル** | \_.appservice.local.AzureStack.external.pfx |
    | **App Service API の SSL 証明書ファイル** | api.appservice.local.AzureStack.external.pfx |
    | **App Service 公開元の SSL 証明書ファイル** | ftp.appservice.local.AzureStack.external.pfx |

    証明書の作成時に別のドメイン サフィックスを使った場合は、証明書ファイル名で *local.AzureStack.external* を使わないでください。 代わりに、ご自分のカスタム ドメイン情報を使用します。

    ![App Service インストーラーで必要な証明書の詳細を指定する画面を示すスクリーンショット][7]

11. App Service リソース プロバイダー データベースをホストするために使用するサーバー インスタンスについて、SQL Server の詳細を入力し、 **[次へ]** を選択します。 インストーラーにより、SQL 接続のプロパティが検証されます。<br><br>App Service インストーラーは、続行する前に SQL Server への接続性をテストしようとします。 既存の仮想ネットワークにデプロイする場合、この接続テストが失敗する可能性があります。 警告と、続行するためのプロンプトが表示されます。 SQL Server の情報が正しい場合は、デプロイを続行します。

    ![App Service インストーラーで SQL Server の接続の詳細を指定する画面を示すスクリーンショット][8]

12. ロール インスタンスと SKU のオプションを確認します。 運用環境デプロイの各ロールの既定値として、インスタンスの最小数および SKU の最小値が入力されています。  ASDK デプロイでは、インスタンスを下位の SKU にスケールダウンしてコアとメモリのコミットを減らすことができますが、パフォーマンスが低下します。 デプロイの計画に役立つ vCPU 要件とメモリ要件の概要が表示されています。 必要な項目を選択したら、 **[次へ]** を選択します。

    >[!NOTE]
    >運用環境デプロイの場合は、「[Azure Stack Hub での Azure App Service サーバー ロールのキャパシティ プランニング](azure-stack-app-service-capacity-planning.md)」のガイダンスに従ってください。

    | Role | インスタンスの最小値 | SKU の最小値 | Notes |
    | --- | --- | --- | --- |
    | コントローラー | 2 | Standard_A4_v2 - (4 コア、8192 MB) | App Service クラウドの正常性を管理および維持します。 |
    | 管理 | 1 | Standard_D3_v2 - (4 コア、14336 MB) | App Service Azure Resource Manager および API のエンドポイント、ポータル拡張機能 (管理、テナント、Functions ポータル)、データ サービスを管理します。 フェールオーバーをサポートするには、推奨されるインスタンスを 2 つに増やします。 |
    | Publisher | 1 | Standard_A2_v2 - (2 コア、4096 MB) | FTP および Web デプロイによってコンテンツを公開します。 |
    | FrontEnd | 1 | Standard_A4_v2 - (4 コア、8192 MB) | App Service アプリに要求をルーティングします。 |
    | 共有 Worker | 1 | Standard_A4_v2 - (4 コア、8192 MB) | Web または API アプリ、および Azure Functions アプリをホストします。 より多くのインスタンスの追加が必要になる場合があります。 オペレーターは、サービスを定義することや任意の SKU レベルを選ぶことができます。 レベルには、少なくとも 1 つの vCPU が必要です。 |

    ![App Service インストーラーでロール インスタンスの数とそれに対応するコンピューティング SKU を指定する画面を示すスクリーンショット][9]

    > [!NOTE]
    > **Windows Server 2016 Core は、Azure Stack Hub 上で Azure App Service と共に使用するためにサポートされているプラットフォーム イメージではありません。運用環境デプロイには評価版イメージを使用しないでください。**

13. **[プラットフォーム イメージの選択]** ボックスで、App Service クラウド用のコンピューティング リソース プロバイダーで選択可能なイメージの中から、ご自分のデプロイの Windows Server 2016 仮想マシン (VM) イメージを選びます。 **[次へ]** を選択します。

14. 次の App Service インストーラー ページで、次の手順に従います。

     a. Worker ロールの VM 管理者のユーザー名とパスワードを入力します。

     b. その他のロールの VM 管理者のユーザー名とパスワードを入力します。

     c. **[次へ]** を選択します。

    ![App Service インストーラーによって使用される Windows プラットフォーム イメージを選択する画面を示すスクリーンショット][10]

15. App Service インストーラーの概要ページで、次の手順に従います。

    a. 選択した内容を確認します。 変更を加えるには、 **[前へ]** を使って前のページに戻ります。

    b. 構成が正しい場合は、チェック ボックスをオンにします。

    c. デプロイを開始するには、 **[次へ]** を選択します。

    ![App Service インストーラーによるデプロイ用に指定されたオプションの概要を示すスクリーンショット][11]

16. 次の App Service インストーラー ページで、次の手順に従います。

    a. インストールの進行状況を追跡します。 既定の選択内容と Windows 2016 Datacenter の基本イメージの有効期間に基づいて App Service on Azure Stack Hub をデプロイするには、最長 240 分かかる場合があります。

    b. インストーラーが正常に完了したら、 **[終了]** を選択します。

    ![App Service インストーラーによって行われたデプロイの進行状況を示すスクリーンショット][12]

## <a name="post-deployment-steps"></a>デプロイ後の手順

> [!IMPORTANT]
> SQL Always On インスタンスを使用して App Service RP を提供している場合は、データベースのフェールオーバー時にサービスの停止を防ぐために、[appservice_hosting と appservice_metering データベースを可用性グループに追加](/sql/database-engine/availability-groups/windows/availability-group-add-a-database)し、それらのデータベースを同期する **必要があります**。

既存の仮想ネットワークにデプロイし、内部 IP アドレスを使用してファイル サーバーに接続する場合は、送信セキュリティ規則を追加する必要があります。 この規則により、worker サブネットとファイル サーバー間の SMB トラフィックが有効になります。 管理者ポータルでネットワーク セキュリティ グループである WorkersNsg に移動し、次のプロパティを持つ送信セキュリティ規則を追加します。

- ソース:Any
- 送信元ポート範囲: *
- 変換先:IP アドレス
- 宛先 IP アドレス範囲:ファイル サーバーの IP の範囲
- 送信先ポート範囲:445
- プロトコル:TCP
- アクション:Allow
- 優先順位:700
- 名前:Outbound_Allow_SMB445

## <a name="validate-the-app-service-on-azure-stack-hub-installation"></a>App Service on Azure Stack Hub インストールを検証する

1. Azure Stack Hub 管理者ポータルで、 **[管理 - App Service]** に移動します。

2. 概要の状態で、 **[状態]** に **[ロールはいずれも準備ができています]** と表示されていることを確認します。

    ![Azure Stack Hub 管理ポータルでの App Service の管理](media/azure-stack-app-service-deploy/image12.png)

## <a name="test-drive-app-service-on-azure-stack-hub"></a>App Service on Azure Stack Hub を試してみる

App Service リソースプロバイダーをデプロイして登録したら、テストしてユーザーが Web アプリと API アプリをデプロイできることを確認します。

>[!NOTE]
>プラン内に Microsoft.Web 名前空間があるオファーを作成する必要があります。 また、このオファーにサブスクライブするテナント サブスクリプションも必要です。 詳細については、[オファーの作成](azure-stack-create-offer.md)と[プランの作成](azure-stack-create-plan.md)に関するページをご覧ください。
>
>App Service on Azure Stack Hub を使うアプリを作るには、テナント サブスクリプションが *必要です*。 管理者ポータル内でサービス管理者が実行できるタスクは、App Service のリソース プロバイダー管理に関連するものだけです。 これには、容量の追加、デプロイ ソースの構成、worker 層と SKU の追加などが含まれます。
>
>Web アプリ、API アプリ、Azure Functions アプリを作るには、ユーザー ポータルを使う必要があり、テナント サブスクリプションがある必要があります。
>

テスト Web アプリを作成するには、次の手順に従います。

1. Azure Stack Hub ユーザー ポータルで、 **[+ リソースの作成]**  >  **[Web + モバイル]**  >  **[Web アプリ]** の順に選択します。

2. **[Web アプリ]** で、 **[Web アプリ]** に名前を入力します。

3. **[リソース グループ]** で、 **[新規]** を選択します。 **[リソース グループ]** の名前を入力します。

4. **[App Service プラン/場所]**  >  **[新規作成]** の順に選択します。

5. **[App Service プラン]** で、 **[App Service プラン]** の名前を入力します。

6. **[価格レベル]**  >  **[Free-Shared]\(無料 - 共有\)** または **[Shared-Shared]\(共有 - 共有\)**  >  **[選択]**  >  **[OK]**  >  **[作成]** の順に選択します。

7. ダッシュボードに新規 Web アプリのタイルが表示されます。 タイルを選択します。

8. **[Web アプリ]** で **[参照]** を選択して、このアプリの既定の Web サイトを表示します。

## <a name="deploy-a-wordpress-dnn-or-django-website-optional"></a>WordPress、DNN、Django の Web サイトを展開する (省略可能)

1. Azure Stack Hub ユーザー ポータルで、 **[+]** を選択して Azure Marketplace にアクセスし、Django Web サイトをデプロイして、デプロイが完了するまで待ちます。 Django Web プラットフォームでは、ファイル システム ベースのデータベースが使われます。 SQL や MySQL など、追加のリソース プロバイダーは必要ありません。

2. MySQL リソースプロバイダーも展開した場合は、Marketplace から WordPress Web サイトを展開できます。 データベース パラメーターを求められたら、お好きなユーザー名とサーバー名を使用して、ユーザー名を *User1\@Server1* として入力します。

3. SQL Server リソースプロバイダーも展開した場合は、Marketplace から DNN Web サイトを展開できます。 データベース パラメーターを求められたら、使用しているリソース プロバイダーに接続された SQL Server を実行しているコンピューター内のデータベースを選びます。
::: zone-end



<!----------------------------------------- DISCONNECTED PIVOT -------------------------------------------->
::: zone pivot="state-disconnected"
この記事では、次のような Azure Stack Hub 環境に [Azure App Service リソース プロバイダー](azure-stack-app-service-overview.md)をデプロイする方法を説明しました。
- インターネットに接続されていない。
- Active Directory フェデレーション サービス (AD FS) によって保護されている。

Azure App Service リソース プロバイダーをオフラインの Azure Stack Hub のデプロイに追加するには、次の最上位タスクをすべて実行する必要があります。

1. [前提となる手順](azure-stack-app-service-before-you-get-started.md)をすべて実行します (証明書の購入など。受け取るまでに数日かかる場合があります)。
2. インターネットに接続されているマシンに[インストール ファイルとヘルパー ファイルをダウンロードして展開](azure-stack-app-service-before-you-get-started.md)します。
3. オフライン インストール パッケージを作成します。
4. appservice.exe インストーラー ファイルを実行します。

## <a name="create-an-offline-installation-package"></a>オフライン インストール パッケージを作成する

オフライン環境で Azure App Service をデプロイするには、まず、インターネットに接続されているマシンでオフライン インストール パッケージを作成します。

1. インターネットに接続されているマシンで AppService.exe インストーラーを実行します。 

2. **[詳細]**  >  **[オフライン インストール パッケージを作成する]** の順に選択します。 この手順は、完了までに数分かかります。

    ![Azure App Service インストーラーでオフライン パッケージを作成する][13]

3. Azure App Service インストーラーにより、オフライン インストール パッケージが作成され、そのパッケージへのパスが表示されます。 **[フォルダーを開く]** を選択すると、エクスプローラーでフォルダーが開きます。

    ![Azure App Service インストーラーで正常に生成されたオフライン インストール パッケージ][14]

4. インストーラー (AppService.exe) とオフライン インストール パッケージを、Azure Stack Hub に接続されているマシンにコピーします。

## <a name="complete-the-offline-installation-of-azure-app-service-on-azure-stack-hub"></a>Azure App Service on Azure Stack Hub のオフライン インストールを完了させる

1. Azure Stack Hub 管理の Azure Resource Management エンドポイントに到達できるコンピューターから、appservice.exe を管理者として実行します。

1. **[詳細設定]**  >  **[Complete offline installation]\(オフライン インストールを完了する\)** の順に選択します。

    ![Azure App Service インストーラーでオフライン インストールを実行する][15]

1. 先ほど作成したオフライン インストール パッケージの場所を参照し、 **[次へ]** を選択します。

    ![Azure App Service インストーラーでオフライン インストール パッケージ パスを指定する][16]

1. マイクロソフト ソフトウェア ライセンス条項を確認して同意し、 **[次へ]** を選択します。

1. サード パーティのライセンス条項を確認して同意し、 **[次へ]** を選択します。


1. Azure App Service クラウド構成情報が正しいことを確認します。 ASDK のデプロイ中に既定の設定を使用した場合は、ここで既定値をそのまま使用することもできます。 ただし、Azure Stack Hub のデプロイ時にオプションをカスタマイズした場合、または統合システムにデプロイしている場合は、それらの変更を反映するように、このウィンドウで値を編集する必要があります。 たとえば、ドメイン サフィックス mycloud.com を使用する場合は、Azure Stack Hub テナントの Azure Resource Manager エンドポイントを `management.<region>.mycloud.com` に変更する必要があります。 自分の情報を確認したら、 **[次へ]** を選択します。

    ![Azure App Service インストーラーで Azure App Service クラウドを構成する][2]

1. App Service インストーラーの次のページで、Azure Stack Hub に接続します。

    1. 使用する接続方法 ( **[資格情報]** または **[サービス プリンシパル]** ) を選択します。
        - **資格情報**
            - Azure Active Directory (Azure AD) を使用している場合は、Azure Stack Hub のデプロイ時に指定した、Azure AD の管理者アカウントとパスワードを入力します。 **[接続]** を選択します。
            - Active Directory フェデレーション サービス (AD FS) を使用している場合は、ご自分の管理者アカウントを指定します。 たとえば、「 cloudadmin@azurestack.local 」のように入力します。 パスワードを入力し、 **[接続]** を選択します。
        - **サービス プリンシパル**
            - 使用するサービス プリンシパルには、**既定のプロバイダー サブスクリプション** に対する **所有者** 権限が **必要です**
            - **サービス プリンシパル ID**、**証明書ファイル**、および **パスワード** を指定し、 **[接続]** を選択します。

    1. **[Azure Stack Hub Subscriptions]\(Azure Stack Hub サブスクリプション\)** で、 **[Default Provider Subscription]\(既定のプロバイダー サブスクリプション\)** を選択します。  Azure App Service on Azure Stack Hub は、 **[Default Provider Subscription]\(既定のプロバイダー サブスクリプション\)** にデプロイする **必要** があります。

    1. **[Azure Stack Hub Locations]\(Azure Stack Hub の場所\)** で、デプロイしているリージョンに対応する場所を選択します。 たとえば、ASDK にデプロイしている場合は、 **[ローカル]** を選びます。

1. Azure App Service インストーラーでは、仮想ネットワークおよび関連付けられているサブネットを作成できます。 または、[こちらの手順](azure-stack-app-service-before-you-get-started.md#virtual-network)で構成された既存の仮想ネットワークにデプロイすることもできます。
   - Azure App Service インストーラーによる方法を使用する場合は、 **[既定の設定で VNet を作成する]** をオンにし、既定値を受け入れて、 **[次へ]** を選択します。
   - 既存のネットワークにデプロイする場合は、 **[Use existing VNet and Subnets]\(既存の VNet とサブネットを使用する\)** をオンにし、次の手順に従います。
       1. 仮想ネットワークが含まれている **リソース グループ** のオプションを選択します。
       2. デプロイする **仮想ネットワーク** の名前を選択します。
       3. 必要なロール サブネットそれぞれに適切な **サブネット** 値を選択します。
       4. **[次へ]** を選択します。

      ![Azure App Service インストーラーでの仮想ネットワークとサブネットの情報][4]

1. ファイル共有の情報を入力してから、 **[次へ]** を選択します。 ファイル共有のアドレスには、ファイル サーバーの完全修飾ドメイン名 (FQDN)、または IP アドレスを使用する必要があります。 たとえば、\\\appservicefileserver.local.cloudapp.azurestack.external\websites、または \\\10.0.0.1\websites を使用します。  ドメインに参加しているファイル サーバーを使用している場合は、ドメインを含む完全なユーザー名を指定する必要があります。 (例: `<myfileserverdomain>\<FileShareOwner>`)。

    > [!NOTE]
    > インストーラーは、続行する前にファイル共有への接続性をテストしようとします。 ただし、既存の仮想ネットワークにデプロイすることを選んだ場合、インストーラーはファイル共有に接続できない可能性があり、続行するかどうかを確認する警告が表示されます。 ファイル共有情報を確認し、正しい場合は続行します。

   ![Azure App Service インストーラーのファイル共有情報][5]

1. 次のページで、次の操作を行います。
    1. **[Identity Application ID]\(ID アプリケーションの ID\)** ボックスで、[前提条件](azure-stack-app-service-before-you-get-started.md)の一部として作成した ID アプリケーションの GUID を入力します。
    1. **[Identity Application certificate file]\(ID アプリケーションの証明書ファイル\)** ボックスで、証明書ファイルの場所を入力 (または参照) します。
    1. **[Identity Application certificate password]\(ID アプリケーションの証明書パスワード\)** ボックスで、証明書のパスワードを入力します。 このパスワードは、証明書作成のスクリプトを使ったときに書き留めたものです。
    1. **[Azure Resource Manager root certificate file]\(Azure Resource Manager のルート証明書ファイル\)** ボックスで、証明書ファイルの場所を入力 (または参照) します。
    1. **[次へ]** を選択します。

    ![Azure App Service インストーラーでアプリ ID と証明書情報を入力する][6]

1. 3 つの証明書ファイルの各ボックスで、 **[参照]** を選択し、適切な証明書ファイルに移動します。 各証明書のパスワードを入力する必要があります。 これらの証明書は、「[App Service on Azure Stack Hub のデプロイの前提条件](azure-stack-app-service-before-you-get-started.md)」で作成したものです。 すべての情報を入力したら、 **[次へ]** を選択します。

    | ボックス | 証明書ファイル名の例 |
    | --- | --- |
    | **App Service の既定の SSL 証明書ファイル** | \_.appservice.local.AzureStack.external.pfx |
    | **App Service API の SSL 証明書ファイル** | api.appservice.local.AzureStack.external.pfx |
    | **App Service 公開元の SSL 証明書ファイル** | ftp.appservice.local.AzureStack.external.pfx |

    証明書の作成時に別のドメイン サフィックスを使った場合は、証明書ファイル名で *local.AzureStack.external* を使わないでください。 代わりに、ご自分のカスタム ドメイン情報を使用します。

    ![Azure App Service インストーラーで SSL 証明書情報を入力する][7]

1. Azure App Service リソース プロバイダー データベースをホストするために使用するサーバー インスタンスについて、SQL Server の詳細を入力し、 **[次へ]** を選択します。 インストーラーにより、SQL 接続のプロパティが検証されます。 内部 IP または SQL Server 名の FQDN を入力する **必要** があります。

    > [!NOTE]
    > インストーラーは、続行する前に SQL Server を実行しているコンピューターへの接続性をテストしようとします。 ただし、既存の仮想ネットワークにデプロイすることを選んだ場合、インストーラーは SQL Server を実行しているコンピューターに接続できない可能性があり、続行するかどうかを確認する警告が表示されます。 SQL Server 情報を確認し、正しい場合は続行します。
    >
    > Azure App Service on Azure Stack Hub 1.3 以降では、インストーラーは、SQL Server を実行しているコンピューターがデータベースのコンテイメントを SQL Server レベルで有効にしているかどうかを確認します。 そうでない場合は、次の例外が表示されます。
    > ```sql
    >    Enable contained database authentication for SQL server by running below command on SQL server (Ctrl+C to copy)
    >    ***********************************************************
    >    sp_configure 'contained database authentication', 1;
    >    GO
    >    RECONFIGURE;
    >    GO
    >    ***********************************************************
    > ```
    > 詳細については、[Azure App Service on Azure Stack Hub 1.3 のリリース ノート](azure-stack-app-service-release-notes-update-three.md)を参照してください。

    ![Azure App Service インストーラーで SQL Server 情報を入力する][8]

1. ロール インスタンスと SKU のオプションを確認します。 運用環境デプロイの各ロールの既定値として、インスタンスの最小数および SKU の最小値が入力されています。  ASDK デプロイでは、インスタンスを下位の SKU にスケールダウンしてコアとメモリのコミットを減らすことができますが、パフォーマンスが低下します。  デプロイの計画に役立つ vCPU 要件とメモリ要件の概要が表示されています。 必要な項目を選択したら、 **[次へ]** を選択します。

     > [!NOTE]
     > 運用環境デプロイの場合は、「[Azure Stack Hub での Azure App Service サーバー ロールの容量計画](azure-stack-app-service-capacity-planning.md)」のガイダンスに従ってください。
     >
     >

    
    | Role | インスタンスの最小値 | SKU の最小値 | Notes |
    | --- | --- | --- | --- |
    | コントローラー | 2 | Standard_A4_v2 - (4 コア、8192 MB) | App Service クラウドの正常性を管理および維持します。 |
    | 管理 | 1 | Standard_D3_v2 - (4 コア、14336 MB) | App Service Azure Resource Manager および API のエンドポイント、ポータル拡張機能 (管理、テナント、Functions ポータル)、データ サービスを管理します。 フェールオーバーをサポートするには、推奨されるインスタンスを 2 つに増やします。 |
    | Publisher | 1 | Standard_A2_v2 - (2 コア、4096 MB) | FTP および Web デプロイによってコンテンツを公開します。 |
    | FrontEnd | 1 | Standard_A4_v2 - (4 コア、8192 MB) | App Service アプリに要求をルーティングします。 |
    | 共有 Worker | 1 | Standard_A4_v2 - (4 コア、8192 MB) | Web または API アプリ、および Azure Functions アプリをホストします。 より多くのインスタンスの追加が必要になる場合があります。 オペレーターは、サービスを定義することや任意の SKU レベルを選ぶことができます。 レベルには、少なくとも 1 つの vCPU が必要です。 |

    ![Azure App Service インストーラーでロール階層と SKU オプションを設定する][9]

1. **[プラットフォーム イメージの選択]** ボックスで、Azure App Service クラウド用のコンピューティング リソース プロバイダーで選択可能なイメージの中から、ご自分のデプロイの Windows Server 2016 仮想マシン (VM) イメージを選びます。 **[次へ]** を選択します。

    > [!NOTE]
    > Windows Server 2016 Core は、Azure App Service on Azure Stack Hub で使用するためにサポートされているプラットフォーム イメージでは *ありません*。  運用環境デプロイには評価版イメージを使用しないでください。 Azure App Service on Azure Stack Hub では、デプロイに使用されるイメージ上で Microsoft .NET 3.5.1 SP1 がアクティブにされている必要があります。 Marketplace にシンジケート化された Windows Server 2016 イメージでは、この機能は有効ではありません。 そのため、この機能をあらかじめ有効にして Windows Server 2016 イメージを作成し、使用する必要があります。
    >
    > カスタム イメージを作成して Marketplace に追加する方法の詳細については、「[Azure Stack Hub にカスタム VM イメージを追加する](azure-stack-add-vm-image.md)」を参照してください。 Marketplace にイメージを追加するときは、必ず以下を指定してください。
    >
    >- 発行元 = MicrosoftWindowsServer
    >- オファー = WindowsServer
    >- SKU = 2016-Datacenter
    >- バージョン = "最新" バージョンを指定

1. 次のページで、次の操作を行います。
     1. Worker ロールの VM 管理者のユーザー名とパスワードを入力します。
     2. その他のロールの VM 管理者のユーザー名とパスワードを入力します。
     3. **[次へ]** を選択します。

    ![Azure App Service インストーラーでロール VM 管理者を入力する][10]

1. 概要ページで、次のことを行います。
    1. 選択した内容を確認します。 変更を加えるには、 **[前へ]** を使って前のページに戻ります。
    2. 構成が正しい場合は、チェック ボックスをオンにします。
    3. デプロイを開始するには、 **[次へ]** を選択します。

    ![Azure App Service インストーラーで行われる選択の概要][11]

1. 次のページで、次の操作を行います。
    1. インストールの進行状況を追跡します。 既定の選択内容と Windows 2016 Datacenter の基本イメージの有効期間に基づいて App Service on Azure Stack Hub をデプロイするには、最長 240 分かかる場合があります。

    2. インストーラーの実行が完了したら、 **[終了]** を選択します。

    ![Azure App Service インストーラーでインストール プロセスを追跡する][12]

## <a name="post-deployment-steps"></a>デプロイ後の手順

> [!IMPORTANT]
> SQL Always On インスタンスで Azure App Service RP を提供した場合は、[appservice_hosting データベースと appservice_metering データベースを可用性グループに追加する](/sql/database-engine/availability-groups/windows/availability-group-add-a-database)**必要があります**。 また、データベースのフェールオーバーが発生した場合のサービスの損失を防ぐため、データベースを同期する必要もあります。

ファイル サーバーに接続するために既存の仮想ネットワークと内部 IP アドレスへデプロイする場合は、送信セキュリティ規則を追加して、worker サブネットとファイル サーバー間の SMB トラフィックを有効にする必要があります。 管理者ポータルでネットワーク セキュリティ グループである WorkersNsg に移動し、次のプロパティを持つ送信セキュリティ規則を追加します。

- ソース:Any
- 送信元ポート範囲: *
- 変換先:IP アドレス
- 宛先 IP アドレス範囲:ファイル サーバーの IP の範囲
- 送信先ポート範囲:445
- プロトコル:TCP
- アクション:Allow
- 優先順位:700
- 名前:Outbound_Allow_SMB445

## <a name="validate-the-azure-app-service-on-azure-stack-hub-installation"></a>Azure App Service on Azure Stack Hub のインストールを検証する

1. Azure Stack Hub 管理者ポータルで、 **[管理 - App Service]** に移動します。

1. 概要の状態で、 **[状態]** に **[ロールはいずれも準備ができています]** と表示されていることを確認します。

    ![Azure App Service 管理の概要](media/azure-stack-app-service-deploy/image12.png)

## <a name="test-drive-azure-app-service-on-azure-stack-hub"></a>Azure App Service on Azure Stack Hub のテスト

Azure App Service リソース プロバイダーをデプロイして登録したら、テストを行ってユーザーが Web アプリと API アプリをデプロイできることを確認します。

> [!NOTE]
> プラン内に Microsoft.Web 名前空間があるオファーを作る必要があります。 その後、このオファーをサブスクライブするテナント サブスクリプションが必要となります。 詳細については、[オファーの作成](azure-stack-create-offer.md)と[プランの作成](azure-stack-create-plan.md)に関するページをご覧ください。
>
> Azure App Service on Azure Stack Hub を使用するアプリを作成するには、テナント サブスクリプションが *必要です*。 管理者ポータル内でサービス管理者が実行できる機能は、Azure App Service のリソース プロバイダー管理に関連するものだけです。 これらの機能には、容量の追加、展開ソースの構成、Worker 層と SKU の追加などがあります。
>
> 3 番目のテクニカル プレビューの時点では、Web アプリ、API アプリ、Azure Functions アプリを作成するには、ユーザー ポータルを使用する必要があり、テナント サブスクリプションを用意する必要があります。

1. Azure Stack Hub ユーザー ポータルで、 **[+ リソースの作成]**  >  **[Web + モバイル]**  >  **[Web アプリ]** の順に選択します。

1. **[Web アプリ]** ブレードで、 **[Web アプリ]** ボックスに名前を入力します。

1. **[リソース グループ]** で、 **[新規]** を選択します。 **[リソース グループ]** ボックスに名前を入力します。

1. **[App Service プラン/場所]**  >  **[新規作成]** の順に選択します。

1. **[App Service プラン]** ブレードで、 **[App Service プラン]** ボックスに名前を入力します。

1. **[価格レベル]**  >  **[Free-Shared]\(無料 - 共有\)** または **[Shared-Shared]\(共有 - 共有\)**  >  **[選択]**  >  **[OK]**  >  **[作成]** の順に選択します。

1. 1 分未満に、ダッシュボードに新規 Web アプリのタイルが表示されます。 タイルを選択します。

1. **[Web アプリ]** ブレードで、 **[参照]** を選択して、このアプリの既定の Web サイトを表示します。

## <a name="deploy-a-wordpress-dnn-or-django-website-optional"></a>WordPress、DNN、Django の Web サイトを展開する (省略可能)

1. Azure Stack Hub ユーザー ポータルで、 **[+]** を選択して Azure Marketplace にアクセスし、Django Web サイトをデプロイして、正常に完了するまで待ちます。 Django Web プラットフォームでは、ファイル システム ベースのデータベースが使われます。 SQL や MySQL など、追加のリソース プロバイダーは必要ありません。

1. MySQL リソースプロバイダーも展開した場合は、Azure Marketplace から WordPress Web サイトを展開できます。 データベース パラメーターを求められたら、お好きなユーザー名とサーバー名を使用して、ユーザー名を *User1\@Server1* として入力します。

1. SQL Server リソースプロバイダーも展開した場合は、Azure Marketplace から DNN Web サイトを展開できます。 データベース パラメーターを求められたら、お使いのリソース プロバイダーに接続されている SQL Server を実行中のコンピューター内のデータベースを選びます。

::: zone-end

## <a name="next-steps"></a>次のステップ

App Service on Azure Stack Hub のための追加の管理者操作を準備します。

- [容量計画](azure-stack-app-service-capacity-planning.md)
- [デプロイ ソースを構成する](azure-stack-app-service-configure-deployment-sources.md)

<!-- Connected image references-->
[1]: ./media/azure-stack-app-service-deploy/app-service-installer.png
[2]: ./media/azure-stack-app-service-deploy/app-service-azure-stack-arm-endpoints.png
[3]: ./media/azure-stack-app-service-deploy/app-service-azure-stack-subscription-information.png
[4]: ./media/azure-stack-app-service-deploy/app-service-default-VNET-config.png
[5]: ./media/azure-stack-app-service-deploy/app-service-fileshare-configuration.png
[6]: ./media/azure-stack-app-service-deploy/app-service-identity-app.png
[7]: ./media/azure-stack-app-service-deploy/app-service-certificates.png
[8]: ./media/azure-stack-app-service-deploy/app-service-sql-configuration.png
[9]: ./media/azure-stack-app-service-deploy/app-service-cloud-quantities.png
[10]: ./media/azure-stack-app-service-deploy/app-service-role-credentials.png
[11]: ./media/azure-stack-app-service-deploy/app-service-azure-stack-deployment-summary.png
[12]: ./media/azure-stack-app-service-deploy/app-service-deployment-progress.png

<!-- Disconnected image references-->
[13]: ./media/azure-stack-app-service-deploy/app-service-exe-advanced-create-package.png
[14]: ./media/azure-stack-app-service-deploy/app-service-exe-advanced-create-package-complete.png
[15]: ./media/azure-stack-app-service-deploy/app-service-exe-advanced-complete-offline.png
[16]: ./media/azure-stack-app-service-deploy/app-service-exe-advanced-complete-offline-package-browse.png