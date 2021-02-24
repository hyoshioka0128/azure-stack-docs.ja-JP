---
title: Azure Stack Hub で複数サイト間 VPN トンネルを設定する
description: Azure Stack Hub で複数サイト間 VPN トンネルを設定する方法について説明します。
author: mattbriggs
ms.topic: how-to
ms.date: 12/2/2020
ms.author: mabrigg
ms.reviewer: sijuman
ms.lastreviewed: 12/2/2020
ms.openlocfilehash: c7462bebc919b85db64dae9c4a07fdc1ca276e33
ms.sourcegitcommit: 9ef2cdc748cf00cd3c8de90705ea0542e29ada97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/02/2020
ms.locfileid: "96525780"
---
# <a name="how-to-set-up-multiple-site-to-site-vpn-tunnels-in-azure-stack-hub"></a>Azure Stack Hub で複数サイト間 VPN トンネルを設定する方法

この記事では、Azure Stack Hub Resource Manager テンプレートを使用してソリューションをデプロイする方法について説明します。 このソリューションでは、仮想ネットワークに関連付けられた複数のリソース グループと、これらのシステムの接続方法を作成します。

テンプレートは [Azure Intelligent Edge Patterns](https://github.com/Azure-Samples/azure-intelligent-edge-patterns) GitHub リポジトリにあります。 このテンプレートは、**rras-gre-vnet-vnet** フォルダーにあります。 

## <a name="scenarios"></a>シナリオ

![5 つの VPN シナリオが図に示されています: 1 つのサブスクリプション内の 2 つのリソース グループ間、それぞれが独自のサブスクリプション内にある 2 つのグループ間、別個のスタック インスタンス内の 2 つのグループ間、1 つのグループとオンプレミスの複数ローカル リソースの間、および複数の VPN トンネル。](./media/azure-stack-network-howto-vpn-tunnel/scenarios.svg)

## <a name="create-multiple-vpn-tunnels"></a>複数の VPN トンネルを作成する

![この図は、それぞれが独自のサブスクリプションとスタック インスタンス内にあり、VPN によって接続されている 2 つのリソース グループを示しています。これら 2 つのグループのうちの 1 つは、VPN によってオンプレミスのローカル リソースに接続されています。](./media/azure-stack-network-howto-vpn-tunnel/azure-stack-network-vpn-tunnel1.svg)

-  3 層アプリケーション (Web、アプリ、および DB) をデプロイします。

-  最初の 2 つのテンプレートを別の Azure Stack Hub インスタンスにデプロイします。

-  **WebTier** は PPE1 にデプロイされ、**AppTier** は PPE2 にデプロイされます。

-  IKE トンネルを使用して **WebTier** と **AppTier** を接続します。

-  **AppTier** を、**DBTier** を呼び出すオンプレミス システムに接続します。

## <a name="steps-to-deploy-multiple-vpns"></a>複数の VPN をデプロイする手順

これは複数の手順があるプロセスです。 このソリューションでは、Azure Stack Hub ポータルを使用します。 ただし、PowerShell、Azure CLI、またはその他のコードとしてのインフラストラクチャ ツール チェーンを使用して出力をキャプチャし、入力として使用することができます。

![この図は、2 つのインフラストラクチャ間に VPN トンネルをデプロイする 5 つの手順を示しています。 最初の 2 つの手順では、テンプレートから 2 つのインフラストラクチャを作成します。 次の 2 つの手順ではテンプレートから 2 つの VPN トンネルを作成し、最後の手順ではトンネルを接続します。](./media/azure-stack-network-howto-vpn-tunnel/azure-stack-network-vpn-tunnel2.svg)

## <a name="walkthrough"></a>チュートリアル

### <a name="deploy-web-tier-to-azure-stack-hub-instances-ppe1"></a>Web 層を Azure Stack Hub インスタンス PPE1 にデプロイする

1. Azure Stack Hub ユーザー ポータルを開き、**[リソースの作成]** を選択します。

2. **[テンプレートのデプロイ]** を選択します。

    ![[ダッシュボード > 新規] ダイアログ ボックスに各種オプションが表示されています。 [テンプレートのデプロイ] ボタンが強調表示されています。](./media/azure-stack-network-howto-vpn-tunnel/image3.png)

3. **azure-intelligent-edge-patterns/rras-vnet-vpntunnel** リポジトリから azuredeploy.json の内容をコピーし、テンプレート ウィンドウに貼り付けます。 テンプレート内に含まれているリソースを確認し、**[保存]** を選択します。

    ![[ダッシュボード > 新規 > ソリューション テンプレートのデプロイ > テンプレートの編集] ダイアログ ボックスに、azuredeploy.json ファイルが貼り付けられたウィンドウがあります。](./media/azure-stack-network-howto-vpn-tunnel/image4.png)

4. **[リソース グループ]** の名前を入力し、パラメーターを確認します。

    > [!Note]  
    > WebTier のアドレス空間は **10.10.0.0/16** なので、リソース グループの場所は **PPE1** であることがわかります。

    ![[ダッシュボード > 新規 > ソリューション テンプレートのデプロイ > パラメーター] ダイアログ ボックスに、[リソース グループ] テキスト ボックスとラジオ ボタンがあります。 [新規作成] ボタンが選択され、テキストが "WebTier" になっています。](./media/azure-stack-network-howto-vpn-tunnel/image5.png)

### <a name="deploy-app-tier-to-the-second-azure-stack-hub-instances"></a>2 つ目の Azure Stack Hub インスタンスにアプリ層をデプロイする

次に示すように、**WebTier** と同じプロセスを使用できますが、パラメーターは異なります。

> [!NOTE]  
> AppTier のアドレス空間は **10.20.0.0/16** なので、リソース グループの場所は **WestUS2** であることがわかります。

![[ダッシュボード > ソリューション テンプレートのデプロイ > パラメーター] ダイアログ ボックスに、[リソース グループ] テキスト ボックスとラジオ ボタンがあります。 [新規作成] ボタンが選択され、テキストが "AppTier" になっています。 その他の強調表示された 8 つのテキスト ボックスには、テンプレート パラメーターが含まれています。](./media/azure-stack-network-howto-vpn-tunnel/image6.png)

### <a name="review-the-deployments-for-web-tier-and-app-tier-and-capture-outputs"></a>Web 層とアプリケーション層のデプロイとキャプチャの出力を確認する

1. デプロイが正常に完了したことを確認します。 **[出力]** を選択します。

    ![[ダッシュボード > Microsoft.Template - 概要] ダイアログ ボックスで [出力] オプションが強調表示されています。 "デプロイが完了しました" というメッセージが表示され、その下に WebTier グループのリソースの一覧がそれぞれの名前、種類、状態、詳細へのリンクと共に表示されています。](./media/azure-stack-network-howto-vpn-tunnel/image7.png)

1. 最初の 4 つの値をメモ帳アプリにコピーします。

    ![このダイアログ ボックスは [ダッシュボード > Microsoft.Template - 出力] です。 Web 層のデプロイからコピーする 4 つのテキスト ボックス [LOCALTUNNELENDPOINT]、[LOCALVNETADDRESSSPACE]、[LOCALVNETGATEWAY]、[LOCALTUNNELGATEWAY] が強調表示されています。](./media/azure-stack-network-howto-vpn-tunnel/image8.png)

1. **AppTier** のデプロイに対してこの手順を繰り返します。

    ![[ダッシュボード > Microsoft.Template - 概要] ダイアログ ボックスで [出力] オプションが強調表示されています。 "デプロイが完了しました" というメッセージが表示され、その下に AppTier グループのリソースの一覧がそれぞれの名前、種類、状態、詳細へのリンクと共に表示されています。](./media/azure-stack-network-howto-vpn-tunnel/image9.png)

    ![このダイアログ ボックスは [ダッシュボード > Microsoft.Template - 出力] です。 アプリ層のデプロイからコピーする 4 つのテキスト ボックス [LOCALTUNNELENDPOINT]、[LOCALVNETADDRESSSPACE]、[LOCALVNETGATEWAY]、[LOCALTUNNELGATEWAY] が強調表示されています。](./media/azure-stack-network-howto-vpn-tunnel/image10.png)

### <a name="create-tunnel-from-web-tier-to-app-tier"></a>Web 層からアプリ層へのトンネルを作成する

1. Azure Stack Hub ユーザー ポータルを開き、**[リソースの作成]** を選択します。

2. **[テンプレートのデプロイ]** を選択します。

3. **azuredeploy.tunnel.ike.json** の内容を貼り付けます。

4. **[パラメーターの編集]** を選択します。

![[ダッシュボード > 新規 > ソリューション テンプレートのデプロイ > パラメーター] ダイアログ ボックスに、[リソース グループ] テキスト ボックスとラジオ ボタンがあります。 [既存のものを使用] ボタンが選択され、テキストが "WebTier" になっています。 その他の強調表示された 4 つのテキスト ボックスには、テンプレート パラメーターが含まれています。](./media/azure-stack-network-howto-vpn-tunnel/image11.png)

### <a name="create-tunnel-from-app-tier-to-web-tier"></a>アプリ層から web 層へのトンネルを作成する

1. Azure Stack Hub ユーザー ポータルを開き、**[リソースの作成]** を選択します。

2. **[テンプレートのデプロイ]** を選択します。

3. **azuredeploy.tunnel.ike.json** の内容を貼り付けます。

4. **[パラメーターの編集]** を選択します。

![[ダッシュボード > 新規 > ソリューション テンプレートのデプロイ > パラメーター] ダイアログ ボックスに、[リソース グループ] テキスト ボックスとラジオ ボタンがあります。 [既存のものを使用] ボタンが選択され、テキストが "AppTier" になっています。 その他の強調表示された 4 つのテキスト ボックスには、テンプレート パラメーターが含まれています。](./media/azure-stack-network-howto-vpn-tunnel/image12.png)

### <a name="viewing-tunnel-deployment"></a>トンネルのデプロイの表示

カスタム スクリプト拡張機能からの出力を表示すると、トンネルが作成されていることを確認できます。また、その状態が表示されます。 一方には、もう一方の準備が整うまで待つ間、**接続中** と表示され、もう一方には、デプロイされると **接続済み** と表示されます。

![このダイアログ ボックスは、[ダッシュボード > リソース グループ > WebTier > WebTier-RRAS - 拡張機能] です。 CustomScriptExtension (強調表示) と InstallRRAS の 2 つの拡張機能が一覧表示されています。 どちらも [プロビジョニング成功] の状態を示しています。](./media/azure-stack-network-howto-vpn-tunnel/image13.png)

![このダイアログ ボックスは、[ダッシュボード > リソース グループ > WebTier > WebTier-RRAS - 拡張機能 > CustomScriptExtension] です。 [DETAILED STATUS]\(詳細な状態\) の値は "Provisioning Succeeded" (プロビジョニング成功) です。](./media/azure-stack-network-howto-vpn-tunnel/image14.png)

![AppTier および WebTier のカスタム スクリプト拡張機能からの出力が、横並びのメモ帳ウィンドウに表示されています。 AppTier のトンネルの状態は "Connecting" (接続中) と示されています。 WebTier では "Connected" (接続済み) と示されています。](./media/azure-stack-network-howto-vpn-tunnel/image15.png)

### <a name="troubleshooting-on-the-rras-vm"></a>RRAS VM でのトラブルシューティング

1. リモート デスクトップ (RDP) 規則を **[拒否]** から **[許可]** に変更します。

1. デプロイ時に設定した資格情報を使用して、RDP クライアントでシステムに接続します。

1. 管理者特権のプロンプトで PowerShell を開き、`get-VpnS2SInterface` を実行します。

    ![PowerShell ウィンドウに、Get-VpnS2SInterface コマンドの実行が表示されています。これは、サイト間インターフェイスの詳細を示します。 [ConnectionState] は "Connected" (接続済み) です。](./media/azure-stack-network-howto-vpn-tunnel/image16.png)

1. **RemoteAccess** コマンドレットを使用してシステムを管理します。

    ![アプリ層で実行されている cmd.exe ウィンドウがあり、ipconfig コマンドの出力が表示されています。 2 つのウィンドウがオーバーレイとして表示されています。Web 層に接続する RDP 接続ウィンドウと、Web 層に接続する [Windows セキュリティ] ウィンドウです。](./media/azure-stack-network-howto-vpn-tunnel/image17.png)

### <a name="install-rras-on-an-on-premises-vm-db-tier"></a>オンプレミスの VM DB 層に RRAS をインストールする

1. ターゲットは Windows 2016 イメージです。

1. リポジトリから `Add-Site2SiteIKE.ps1` スクリプトをコピーしてローカルで実行すると、スクリプトによって **WindowsFeature** と **RemoteAccess** がインストールされます。

    > [!NOTE]
    > 環境によっては、システムの再起動が必要になる場合があります。

    詳細については、オンプレミスのマシンのネットワーク構成を参照してください。

    ![ipconfig の出力を表示している cmd.exe ウィンドウと、[ネットワークと共有センター] ウィンドウがあります。](./media/azure-stack-network-howto-vpn-tunnel/image18.png)

1. AppTier テンプレートのデプロイから記録された **Output** パラメーターを追加してスクリプトを実行します。

    ![Add-Site2SiteIKE.psa スクリプトが PowerShell ウィンドウで実行され、出力が表示されています。](./media/azure-stack-network-howto-vpn-tunnel/image19.png)

1. これでトンネルが構成され、AppTier 接続の待機状態になります。

### <a name="configure-app-tier-to-db-tier"></a>アプリ層から DB 層を構成する

1. Azure Stack Hub ユーザー ポータルを開き、**[リソースの作成]** を選択します。

2. **[テンプレートのデプロイ]** を選択します。

3. **azuredeploy.tunnel.ike.json** の内容を貼り付けます。

4. **[パラメーターの編集]** を選択します。

    ![[ダッシュボード > 新規 > ソリューション テンプレートのデプロイ > パラメーター] ダイアログ ボックスに、[リソース グループ] テキスト ボックスとラジオ ボタンがあります。 [既存のものを使用] ボタンが選択され、テキストが "AppTier" になっています。 その他の強調表示された 3 つのテキスト ボックスには、テンプレート パラメーターが含まれています。](./media/azure-stack-network-howto-vpn-tunnel/image20.png)

5. AppTier が選択され、リモート内部ネットワークが 10.99.0.1 に設定されていることを確認します。

### <a name="confirm-tunnel-between-app-tier-and-db-tier"></a>アプリ層と DB 層の間のトンネルを確認する

1. VM にログインせずにトンネルを確認するには、カスタム スクリプト拡張機能を実行します。

2. RRAS VM (AppTier) にアクセスします。

3. **[拡張機能]** を選択し、**カスタム スクリプト拡張機能を実行** します。

4. **azure-intelligent-edge-patterns/rras-vnet-vpntunnel** リポジトリのスクリプト ディレクトリを参照します。 **Get-VPNS2SInterfaceStatus.ps1** を選択します。

    ![このダイアログ ボックスは、[ダッシュボード > リソース グループ > AppTier > AppTier-RRAS - 拡張機能 > CustomScriptExtension] です。 [DETAILED STATUS]\(詳細な状態\) の値は "Provisioning Succeeded" (プロビジョニング成功) です。 詳細は、メモ帳ウィンドウのオーバーレイにあります。](./media/azure-stack-network-howto-vpn-tunnel/image21.png)

5. RDP を有効にしてサインインし、`get-vpns2sinterface` で PowerShell を開いて実行すると、トンネルが接続されていることがわかります。

    **DBTier**

    ![DBTier で、PowerShell ウィンドウに Get-VpnS2SInterface コマンドの実行が表示されています。これは、サイト間インターフェイスの詳細を示します。 [ConnectionState] は "Connected" (接続済み) です。](./media/azure-stack-network-howto-vpn-tunnel/image22.png)

    **AppTier**

    ![AppTier で、PowerShell ウィンドウに Get-VpnS2SInterface コマンドの実行が表示されています。これは、サイト間インターフェイスの詳細を示します。 2 つの接続先の [ConnectionState] は "Connected" (接続済み) です。](./media/azure-stack-network-howto-vpn-tunnel/image23.png)

    > [!NOTE]  
    > 1 台目のマシンから 2 台目へ、および 2 台目から 1 台目の両方で RDP をテストできます。

    > [!NOTE]  
    > このソリューションをオンプレミスで実装するには、Azure Stack Hub リモート ネットワークへのルートをスイッチング インフラストラクチャに、または少なくとも特定の VM にデプロイする必要があります。

### <a name="deploying-a-gre-tunnel"></a>GRE トンネルのデプロイ

このチュートリアルのこのテンプレートでは、[IKE テンプレート](network-howto-vpn-tunnel-ipsec.md)を使用しました。 ただし、[GRE トンネル](network-howto-vpn-tunnel-gre.md)をデプロイすることもできます。 このトンネルにより、スループットがさらに高くなります。

プロセスはほぼ同じです。 ただし、トンネル テンプレートを既存のインフラストラクチャにデプロイする場合、最初の 3 つの入力に他のシステムからの出力を使用する必要があります。 接続対象のリソース グループではなく、デプロイ先のリソース グループの **LOCALTUNNELGATEWAY** を知る必要があります。

![[ダッシュボード > 新規 > ソリューション テンプレートのデプロイ > パラメーター] ダイアログ ボックスに、[リソース グループ] テキスト ボックスとラジオ ボタンがあります。 [既存のものを使用] ボタンが選択され、テキストが "AppTier" になっています。 その他の強調表示された 4 つのテキスト ボックスには、WebTier の出力からのデータが含まれています。](./media/azure-stack-network-howto-vpn-tunnel/image24.png)

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub ネットワークの違いと考慮事項](azure-stack-network-differences.md)  
[GRE を使用して VPN トンネルを作成する方法](network-howto-vpn-tunnel-gre.md)  
[IPSEC を使用して VPN トンネルを作成する方法](network-howto-vpn-tunnel-ipsec.md)