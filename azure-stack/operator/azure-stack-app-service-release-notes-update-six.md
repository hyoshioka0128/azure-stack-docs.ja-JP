---
title: App Service on Azure Stack Hub update 6 のリリース ノート
description: App Service on Azure Stack Hub の Update 6 リリース ノート (新機能、修正点、既知の問題を含む)。
author: apwestgarth
manager: stefsch
ms.topic: article
ms.date: 06/24/2019
ms.author: anwestg
ms.reviewer: anwestg
ms.lastreviewed: 08/20/2019
ms.openlocfilehash: 689f5eaccb7dc30ce7888e9fffdbbd62ac9b5267
ms.sourcegitcommit: 52c934f5eeb5fcd8e8f2ce3380f9f03443d1e445
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/07/2021
ms.locfileid: "97974372"
---
# <a name="app-service-on-azure-stack-hub-update-6-release-notes"></a>App Service on Azure Stack Hub update 6 のリリース ノート

これらのリリース ノートでは、Azure App Service on Azure Stack Hub Update 6 の新機能、修正点、および既知の問題について説明します。 既知の問題は 2 つのセクションに分かれています。アップグレード プロセスに関連する問題とビルド (インストール後) に関する問題です。

> [!IMPORTANT]
> Azure App Service 1.6 をデプロイする前に、Azure Stack Hub 統合システムに 1904 更新プログラムを適用するか、最新の Azure Stack Development Kit (ASDK) をデプロイします。

## <a name="build-reference"></a>ビルドのリファレンス

Azure App Service on Azure Stack Hub Update 6 のビルド番号は **82.0.1.50** です。

## <a name="prerequisites"></a>前提条件

デプロイを開始する前に、[App Service on Azure Stack Hub の前提条件](azure-stack-app-service-before-you-get-started.md)に関するページを参照してください。

Azure App Service on Azure Stack Hub の 1.6 へのアップグレードを開始する前に:

- Azure Stack Hub 管理者ポータルの Azure App Service の管理で確実にすべてのロールの準備ができているようにします。

- Azure Stack Hub 管理ポータルの Azure App Service 管理を使用して、App Service のシークレットをバックアップします

- App Service とマスター データベースをバックアップします。
  - AppService_Hosting
  - AppService_Metering
  - master

- テナント アプリのコンテンツ ファイル共有をバックアップします。

  > [!Important]
  > ファイル サーバーと SQL Server の保守と操作を担当するのは、クラウド オペレーターです。  リソース プロバイダーは、これらのリソースの管理は行いません。  クラウドオ ペレーターが、App Service データベースとテナント コンテンツ ファイル共有のバックアップを行います。

- Azure Stack Hub Marketplace から **カスタム スクリプト拡張機能** のバージョン **1.9.1** を配信します。

## <a name="new-features-and-fixes"></a>新機能と修正

Azure App Service on Azure Stack Hub Update 6 には、次の機能強化と修正が含まれています。

- **App Service のテナント ポータル、管理ポータル、関数ポータル、Kudu ツール** の更新。 Azure Stack Hub Portal SDK バージョンと一貫性があります。

- **Azure Functions ランタイム** が **v1.0.12299** に更新されました。

- 信頼性を高めるためのコア サービスと、一般的な問題を簡単に診断できるようにするエラー メッセージの更新。

- **次のアプリ フレームワークとツールの更新**:

  - ASP.NET Core 2.2.4
  - NodeJS 10.15.2
  - Zulu OpenJDK 8.36.0.1
  - Tomcat 7.0.81
  - Tomcat 8.5.37
  - Tomcat 9.0.14
  - PHP 5.6.39
  - PHP 7.0.33
  - PHP 7.1.25
  - PHP 7.2.13
  - Kudu が 81.10329.3844 に更新されました

- **すべてのロールの基になっているオペレーティング システムの更新プログラム**:
  - [x64 ベース システム用 Windows Server 2016 に対する 2019-04 累積的な更新プログラム (KB4493473)](https://support.microsoft.com/help/4493473/windows-10-update-kb4493473)

## <a name="post-deployment-steps"></a>デプロイ後の手順

> [!IMPORTANT]
> SQL Always On インスタンスを使用して App Service リソース プロバイダーを提供している場合は、データベースのフェールオーバーが発生したときにサービスが失われないように、[appservice_hosting と appservice_metering データベースを可用性グループに追加](/sql/database-engine/availability-groups/windows/availability-group-add-a-database)し、それらのデータベースを同期する必要があります。

## <a name="known-issues-post-installation"></a>既知の問題 (インストール後)

- App Service が既存の仮想ネットワークにデプロイされ、ファイル サーバーがプライベート ネットワークでしか使用できない場合、worker はファイル サーバーに到達することができません。Azure App Service on Azure Stack Hub のデプロイ ドキュメントで説明されているとおりです。

ファイル サーバーに接続するために既存の仮想ネットワークと内部 IP アドレスへデプロイする場合は、送信セキュリティ規則を追加して、worker サブネットとファイル サーバー間の SMB トラフィックを有効にする必要があります。 管理者ポータルで WorkersNsg に移動し、次のプロパティを持つ送信セキュリティ規則を追加します。

* ソース:Any
* 送信元ポート範囲: *
* 変換先:IP アドレス
* 宛先 IP アドレス範囲:ファイル サーバーの IP の範囲
* 送信先ポート範囲:445
* プロトコル:TCP
* アクション:Allow
* 優先順位:700
* 名前:Outbound_Allow_SMB445

## <a name="known-issues-for-cloud-admins-operating-azure-app-service-on-azure-stack-hub"></a>Azure App Service on Azure Stack Hub を運用するクラウド管理者に対する既知の問題

[Azure Stack Hub 1908 リリース ノート](./release-notes.md?view=azs-1908&preserve-view=true)内のドキュメントを参照してください。

## <a name="known-issues-for-tenants-deploying-applications-on-azure-app-service-on-azure-stack-hub"></a>Azure App Service on Azure Stack Hub にアプリケーションをデプロイするテナントに関する既知の問題

- デプロイ センターが淡色表示されます (使用できません)。

    テナントでは、デプロイ センターをまだ利用できません。これは、2018 年後半にパブリック クラウドにリリースされた機能です。 テナントではまだ、portal、CLI、PowerShell による標準のデプロイ方法 (FTP、Web 配置、Git など) を使用できます。

- デプロイ オプション (クラシック) UX およびデプロイ資格情報ポータルのオプションを使用できない。

    Azure Stack Hub のデプロイでデプロイ オプションとデプロイ資格情報のユーザー エクスペリエンスを実現するには、テナントから次の URL 形式を使用してポータルにアクセスする必要があります。`https://portal.&lt;*region*&gt;.&lt;*FQDN*&gt;/?websitesExtension_oldvsts=true`。ASDK の場合は `https://portal.local.azurestack.external/?websitesExtension_oldvsts=true` になり、アプリに移動します。

- ポータルで Azure Function の監視が継続して "読み込み中" と表示される。

    個々の関数を監視しようとしても、ユーザー ポータルに呼び出しのログ、成功数、エラー数が表示されません。 この機能を再度有効にするには、 **[関数アプリ]** 、 **[プラットフォーム機能]** 、 **[アプリケーション設定]** の順に移動します。  **AzureWebJobsDashboard** という名前の新しいアプリの設定を追加し、AzureWebJobsStorage と同じ値を設定します。 それから、関数の監視ビューに移動すると、監視情報が表示されます。

## <a name="next-steps"></a>次のステップ

- Azure App Service の概要については、「[App Service on Azure Stack Hub と Azure Functions の概要](azure-stack-app-service-overview.md)」を参照してください。
- App Service on Azure Stack のデプロイを準備する方法の詳細については、「[App Service on Azure Stack Hub のデプロイの前提条件](azure-stack-app-service-before-you-get-started.md)」を参照してください。
