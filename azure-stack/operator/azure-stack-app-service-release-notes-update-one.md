---
title: App Service on Azure Stack Hub Update 1 のリリース ノート
description: App Service on Azure Stack Hub Update 1 の機能強化、修正、既知の問題の詳細を説明します。
author: bryanla
manager: stefsch
ms.topic: article
ms.date: 03/25/2019
ms.author: anwestg
ms.reviewer: anwestg
ms.lastreviewed: 08/20/2019
ms.openlocfilehash: 433bfa9f6aee13131b80f3200c092bc0d72bd65e
ms.sourcegitcommit: 52c934f5eeb5fcd8e8f2ce3380f9f03443d1e445
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/07/2021
ms.locfileid: "97974389"
---
# <a name="app-service-on-azure-stack-hub-update-1-release-notes"></a>App Service on Azure Stack Hub Update 1 のリリース ノート

これらのリリース ノートでは、Azure App Service on Azure Stack Hub Update 1 における機能強化、修正点、および既知の問題について説明します。 既知の問題は、デプロイに直接関係する問題、更新プロセスの問題、ビルド (インストール後) の問題の 3 つのセクションに分けられています。

> [!IMPORTANT]
> Azure App Service をデプロイする前に、Azure Stack Hub 統合システムに 1802 更新プログラムを適用するか、最新の Azure Stack Development Kit (ASDK) をデプロイします。

## <a name="build-reference"></a>ビルドのリファレンス

App Service on Azure Stack Hub Update 1 のビルド番号は **69.0.13698.9** です。

## <a name="prerequisites"></a>前提条件

> [!IMPORTANT]
> Azure App Service で Kudu の SSO の処理方法が改善されたため、Azure App Service on Azure Stack Hub の新しいデプロイには、3 つのサブジェクトのワイルドカード証明書が必要になります。 新しいサブジェクトは、 **\*.sso.appservice.\<region\>.\<domainname\>.\<extension\>** です

デプロイを開始する前に、[App Service on Azure Stack Hub の前提条件](azure-stack-app-service-before-you-get-started.md)に関するページを参照してください。

## <a name="new-features-and-fixes"></a>新機能と修正

Azure App Service on Azure Stack Hub Update 1 には、次の機能強化と修正が含まれています。

- **Azure App Service の高可用性** - Azure Stack Hub 1802 更新プログラムにより、ワークロードを複数の障害ドメインにデプロイできるようになったため、App Service インフラストラクチャは、複数の障害ドメインへのデプロイによってフォールト トレランスを実現できます。 既定では、Azure App Service のすべての新しいデプロイにこの機能が含まれます。 ただし、Azure Stack Hub 1802 更新プログラムを適用する前に完了したデプロイについては、[App Service の障害ドメインに関するドキュメント](azure-stack-app-service-before-you-get-started.md)を参照してください。

- **既存の仮想ネットワークへのデプロイ** - App Service on Azure Stack Hub を既存の仮想ネットワーク内にデプロイできるようになりました。 既存の仮想ネットワークにデプロイすると、Azure App Service に必要な SQL Server とファイル サーバーにプライベート ポート経由で接続できます。 デプロイ時に既存の仮想ネットワークへのデプロイを選択できますが、デプロイ前に、[App Service が使用するサブネットを作成する必要があります](azure-stack-app-service-before-you-get-started.md#virtual-network)。

- **App Service のテナント ポータル、管理ポータル、Functions ポータル、Kudu ツール** の更新。 Azure Stack Hub Portal SDK バージョンと一貫性があります。

- **Azure Functions ランタイム** が **v1.0.11388** に更新されました。

- **次のアプリケーション フレームワークとツールの更新**:
    - **.NET Core 2.0** のサポートが追加されました。
    - **Node.js** の次のバージョンが追加されました。
        - 6.11.2
        - 6.11.5
        - 7.10.1
        - 8.0.0
        - 8.1.4
        - 8.4.0
        - 8.5.0
        - 8.7.0
        - 8.8.1
        - 8.9.0
    - **NPM** の次のバージョンが追加されました。
        - 3.10.10
        - 4.2.0
        - 5.0.0
        - 5.0.3
        - 5.3.0
        - 5.4.2
        - 5.5.1
    - 次の **PHP** 更新プログラムが追加されました。
        - 5.6.32
        - 7.0.26 (x86 および x64)
        - 7.1.12 (x86 および x64)
    - **Git for Windows** が v 2.14.1 に更新されました。
    - **Mercurial** が v4.5.0 に更新されました。

  - App Service ユーザー ポータルの [カスタム ドメイン] 機能内で **[HTTPS のみ]** 機能のサポートが追加されました。

  - Azure Functions のカスタム ストレージ ピッカーにストレージ接続の検証が追加されました。

### <a name="fixes"></a>修正

- オフライン展開パッケージの作成時に、App Service インストーラーからフォルダーを開くときに、アクセス拒否エラー メッセージが表示されることはなくなります。

- App Service ユーザー ポータルのカスタム ドメイン機能の使用時の問題が解決されました。

- セットアップ中に予約済みの管理者名を使用できなくなります。

- **ドメイン参加済み** ファイル サーバーで App Service デプロイが有効になりました。

- スクリプトでの Azure Stack Hub のルート証明書の取得が改善され、App Service インストーラーでルート証明書を検証する機能が追加されました。

- Microsoft.Web 名前空間のリソースを含むサブスクリプションが削除されたときに、Azure Resource Manager に正しくない状態が返される問題が修正されました。

## <a name="known-issues-with-the-deployment-process"></a>デプロイ プロセスに関する既知の問題

- 証明書の検証エラー。

    インストーラーの検証が過度に制限されているため、統合システムにデプロイするときに App Service インストーラーに証明書を指定しようとして問題が発生することがあります。 App Service インストーラーが再リリースされため、ユーザーは[更新されたインストーラーをダウンロードする](https://aka.ms/appsvconmasinstaller)必要があります。 更新されたインストーラーでも証明書の検証の問題が発生する場合は、サポートにお問い合わせください。

- 統合システムから Azure Stack Hub ルート証明書を取得するときの問題。

    ルート証明書がインストールされていないマシンでスクリプトを実行すると、Get-AzureStackRootCert.ps1 のエラーにより Azure Stack Hub ルート証明書の取得に失敗する問題がありました。 問題を解決するスクリプトも再リリースされました。 [ここから更新されたヘルパー スクリプトをダウンロード](https://aka.ms/appsvconmashelpers)してください。 更新されたスクリプトを使用してもルート証明書の取得時に問題が発生する場合は、サポートにお問い合わせください。

## <a name="known-issues-with-the-update-process"></a>更新プロセスに関する既知の問題

- Azure App Service on Azure Stack Hub Update 1 の更新に関する既知の問題はありません。

## <a name="known-issues-post-installation"></a>既知の問題 (インストール後)

- スロット スワップが機能しません。

このリリースでは、サイトのスロット スワップが破損しています。 機能を復元するには、次の手順を実行します。

1. App Services コントローラー インスタンスへのリモート デスクトップ接続を **許可** するように ControllersNSG Network Security Group を変更します。 AppService.local を、App Service をデプロイしたリソース グループの名前に置き換えます。

    ```powershell
      Add-AzureRmAccount -EnvironmentName AzureStackAdmin

      $nsg = Get-AzureRmNetworkSecurityGroup -Name "ControllersNsg" -ResourceGroupName "AppService.local"

      $RuleConfig_Inbound_Rdp_3389 =  $nsg | Get-AzureRmNetworkSecurityRuleConfig -Name "Inbound_Rdp_3389"

      Set-AzureRmNetworkSecurityRuleConfig -NetworkSecurityGroup $nsg `
        -Name $RuleConfig_Inbound_Rdp_3389.Name `
        -Description "Inbound_Rdp_3389" `
        -Access Allow `
        -Protocol $RuleConfig_Inbound_Rdp_3389.Protocol `
        -Direction $RuleConfig_Inbound_Rdp_3389.Direction `
        -Priority $RuleConfig_Inbound_Rdp_3389.Priority `
        -SourceAddressPrefix $RuleConfig_Inbound_Rdp_3389.SourceAddressPrefix `
        -SourcePortRange $RuleConfig_Inbound_Rdp_3389.SourcePortRange `
        -DestinationAddressPrefix $RuleConfig_Inbound_Rdp_3389.DestinationAddressPrefix `
        -DestinationPortRange $RuleConfig_Inbound_Rdp_3389.DestinationPortRange

      # Commit the changes back to NSG
      Set-AzureRmNetworkSecurityGroup -NetworkSecurityGroup $nsg
    ```

2. Azure Stack Hub 管理者ポータルの [Virtual Machines] で **CN0-VM** を参照し、 **[接続] をクリックして** コントローラー インスタンスとのリモート デスクトップ セッションを開きます。 App Service のデプロイ時に指定した資格情報を使用します。
3. **管理者権限で PowerShell** を起動し、次のスクリプトを実行します。

    ```powershell
        Import-Module appservice

        $sm = new-object Microsoft.Web.Hosting.SiteManager

        if($sm.HostingConfiguration.SlotsPollWorkerForChangeNotificationStatus=$true)
        {
          $sm.HostingConfiguration.SlotsPollWorkerForChangeNotificationStatus=$false
        #  'Slot swap mode reverted'
        }
        
        # Confirm new setting is false
        $sm.HostingConfiguration.SlotsPollWorkerForChangeNotificationStatus
        
        # Commit Changes
        $sm.CommitChanges()

        Get-AppServiceServer -ServerType ManagementServer | ForEach-Object Repair-AppServiceServer
        
    ```

4. リモート デスクトップ セッションを終了します。
5. App Services コントローラー インスタンスへのリモート デスクトップ接続を **拒否** するように ControllersNSG Network Security Group を戻します。 AppService.local を、App Service をデプロイしたリソース グループの名前に置き換えます。

    ```powershell

        Add-AzureRmAccount -EnvironmentName AzureStackAdmin

        $nsg = Get-AzureRmNetworkSecurityGroup -Name "ControllersNsg" -ResourceGroupName "AppService.local"

        $RuleConfig_Inbound_Rdp_3389 =  $nsg | Get-AzureRmNetworkSecurityRuleConfig -Name "Inbound_Rdp_3389"

        Set-AzureRmNetworkSecurityRuleConfig -NetworkSecurityGroup $nsg `
          -Name $RuleConfig_Inbound_Rdp_3389.Name `
          -Description "Inbound_Rdp_3389" `
          -Access Deny `
          -Protocol $RuleConfig_Inbound_Rdp_3389.Protocol `
          -Direction $RuleConfig_Inbound_Rdp_3389.Direction `
          -Priority $RuleConfig_Inbound_Rdp_3389.Priority `
          -SourceAddressPrefix $RuleConfig_Inbound_Rdp_3389.SourceAddressPrefix `
          -SourcePortRange $RuleConfig_Inbound_Rdp_3389.SourcePortRange `
          -DestinationAddressPrefix $RuleConfig_Inbound_Rdp_3389.DestinationAddressPrefix `
          -DestinationPortRange $RuleConfig_Inbound_Rdp_3389.DestinationPortRange

        # Commit the changes back to NSG
        Set-AzureRmNetworkSecurityGroup -NetworkSecurityGroup $nsg
    ```

6. App Service が既存の仮想ネットワークにデプロイされ、ファイル サーバーがプライベート ネットワークでしか使用できない場合、worker はファイル サーバーに到達することができません。

ファイル サーバーに接続するために既存の仮想ネットワークと内部 IP アドレスへデプロイする場合は、worker サブネットとファイル サーバー間の SMB トラフィックを有効にする送信セキュリティ規則を追加する必要があります。 管理者ポータルで WorkersNsg に移動し、次のプロパティを持つ送信セキュリティ規則を追加します。

- ソース:Any
- 送信元ポート範囲: *
- 変換先:IP アドレス
- 宛先 IP アドレス範囲:ファイル サーバーの IP の範囲
- 送信先ポート範囲:445
- プロトコル:TCP
- アクション:Allow
- 優先順位:700
- 名前:Outbound_Allow_SMB445

## <a name="known-issues-for-cloud-admins-operating-azure-app-service-on-azure-stack-hub"></a>Azure App Service on Azure Stack Hub を運用するクラウド管理者に対する既知の問題

[Azure Stack Hub 1802 リリース ノート](./release-notes.md?view=azs-1802&preserve-view=true)内のドキュメントを参照してください。

## <a name="next-steps"></a>次のステップ

- Azure App Service の概要については、「[Azure Stack Hub 上の App Service の概要](azure-stack-app-service-overview.md)」を参照してください。
- App Service on Azure Stack Hub のデプロイを準備する方法の詳細については、「[App Service on Azure Stack Hub のデプロイの前提条件](azure-stack-app-service-before-you-get-started.md)」をご覧ください。
