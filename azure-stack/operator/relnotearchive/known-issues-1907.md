---
title: Azure Stack 1907 の既知の問題
description: Azure Stack 1907 リリースの既知の問題について説明します。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/29/2019
ms.author: sethm
ms.reviewer: prchint
ms.lastreviewed: 10/29/2019
ROBOTS: NOINDEX
ms.openlocfilehash: 435f761633b3513be39cd5912cbcee697003e952
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99248784"
---
# <a name="azure-stack-1907-known-issues"></a>Azure Stack 1907 の既知の問題

この記事では、Azure Stack のリリースにおける既知の問題を一覧表示します。 新しい問題が特定されると、この一覧は更新されます。

別のバージョンの既知の問題にアクセスするには、左側の目次の上部にあるバージョン セレクターのドロップダウンを使用します。

> [!IMPORTANT]  
> 更新プログラムを適用する前に、このセクションを確認してください。

> [!IMPORTANT]  
> ご利用の Azure Stack インスタンスが 2 つ前の更新プログラムより遅れている場合、コンプライアンスに対応していないとみなされます。 [サポートを受けるためには、少なくともサポートされる最小バージョンまで更新する](../azure-stack-servicing-policy.md#keep-your-system-under-support)必要があります。

## <a name="1907-update-process"></a>1907 の更新処理

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因:1907 Azure Stack の更新プログラムをインストールしようとしたときに、更新の状態が失敗して、状態が **PreparationFailed** に変更される場合があります。 これは、更新リソース プロバイダー (URP) が、処理のためにストレージ コンテナーから内部インフラストラクチャ共有にファイルを正しく転送できないことが原因です。
- 修復: バージョン 1901 (1.1901.0.95) 以降、この問題は、 **[今すぐ更新]** ( **[再開]** ではない) をもう一度クリックすることで回避できるようになりました。 それにより、URP は前回の試行のファイルをクリーンアップして、ダウンロードを再度開始します。 問題が解決しない場合は、[更新プログラムのインポートとインストールのセクション](../azure-stack-apply-updates.md)に従って、更新プログラム パッケージを手動でアップロードすることをお勧めします。
- 発生頻度: 共通

## <a name="portal"></a>ポータル

### <a name="administrative-subscriptions"></a>管理サブスクリプション

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: バージョン 1804 で導入された 2 つの管理サブスクリプションは使用しないでください。 このサブスクリプションの種類は **Metering** サブスクリプションと **Consumption** サブスクリプションです。
- 修復: これら 2 つのサブスクリプション上でリソースが実行されている場合は、ユーザー サブスクリプションで再作成してください。
- 発生頻度: 共通

### <a name="subscriptions-properties-blade"></a>サブスクリプションの [プロパティ] ブレード

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: 管理者ポータルで、サブスクリプションの **[プロパティ]** ブレードが正しく読み込まれません
- 修復: これらのサブスクリプションのプロパティは、 **[サブスクリプションの概要]** ブレードの **[要点]** ウィンドウで表示できます。
- 発生頻度: 共通

### <a name="subscription-permissions"></a>サブスクリプションのアクセス許可

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因:Azure Stack ポータルを使用して、サブスクリプションへのアクセス許可を表示することはできません。
- 修復: [PowerShell を使用してアクセス許可を確認](/powershell/module/azurerm.resources/get-azurermroleassignment)します。
- 発生頻度: 共通

### <a name="storage-account-settings"></a>Storage アカウントの設定

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: ユーザー ポータルで、ストレージ アカウントの **[構成]** ブレードに、**安全な転送の種類** を変更するオプションが表示されます。 機能は現在 Azure Stack でサポートされていません。
- 発生頻度: 共通

### <a name="upload-blob"></a>BLOB のアップロード

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: ユーザー ポータルで **[OAuth(preview)]\(OAuth (プレビュー)\)** オプションを使用して BLOB をアップロードしようとすると、タスクがエラー メッセージにより失敗します。
- 修復: SAS オプションを使用して BLOB をアップロードします。
- 発生頻度: 共通

## <a name="networking"></a>ネットワーク

### <a name="service-endpoints"></a>サービス エンドポイント

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: ユーザー ポータルで、 **[仮想ネットワーク]** ブレードに、**サービス エンドポイント** を使用するオプションが表示されます。 この機能は現在 Azure Stack でサポートされていません。
- 発生頻度: 共通

### <a name="network-interface"></a>ネットワーク インターフェイス

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: 新しいネットワーク インターフェイスを、**実行** 状態にある VM に追加することはできません。
- 修復: 仮想マシンを停止してから、ネットワーク インターフェイスを追加/削除します。
- 発生頻度: 共通

### <a name="virtual-network-gateway"></a>Virtual Network ゲートウェイ

#### <a name="alerts"></a>警告

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: ユーザー ポータルで、 **[仮想ネットワーク ゲートウェイ]** ブレードに、使用する **アラート** オプションが表示されます。 この機能は現在 Azure Stack でサポートされていません。
- 発生頻度: 共通

#### <a name="active-active"></a>アクティブ/アクティブ

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: ユーザー ポータルで、作成中、および **仮想ネットワーク ゲートウェイ** のリソース メニューで、**アクティブ/アクティブ** 構成を有効にするオプションが表示されます。 この機能は現在 Azure Stack でサポートされていません。
- 発生頻度: 共通

#### <a name="vpn-troubleshooter"></a>VPN トラブルシューティング ツール

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: ユーザー ポータルで、 **[接続]** ブレードに **VPN トラブルシューティング ツール** と呼ばれる機能が表示されます。 この機能は現在 Azure Stack でサポートされていません。
- 発生頻度: 共通

### <a name="network-connection-type"></a>ネットワーク接続の種類

- 適用先:この問題は、すべての 1906 または 1907 環境に適用されます。 
- 原因: ユーザーポータルの **[AddConnection]** ブレードに、**VNet 間** を使用するオプションが表示されます。 この機能は現在 Azure Stack でサポートされていません。 
- 発生頻度: 共通 

#### <a name="documentation"></a>ドキュメント

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因:仮想ネットワーク ゲートウェイの概要ページにあるドキュメントのリンクは、Azure Stack ではなく Azure 固有のドキュメントにリンクされています。 Azure Stack のドキュメントについては、次のリンクを使用してください。

  - [ゲートウェイ SKU](../../user/azure-stack-vpn-gateway-about-vpn-gateways.md#gateway-skus)
  - [高可用性接続](../../user/azure-stack-vpn-gateway-about-vpn-gateways.md#gateway-availability)
  - [Azure Stack での BGP の構成](../../user/azure-stack-vpn-gateway-settings.md#gateway-requirements)
  - [ExpressRoute 回線](../azure-stack-connect-expressroute.md)
  - [カスタムの IPsec/IKE ポリシーの指定](../../user/azure-stack-vpn-gateway-settings.md#ipsecike-parameters)

## <a name="compute"></a>Compute

### <a name="vm-boot-diagnostics"></a>VM ブート診断

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: 新しい Windows 仮想マシン (VM) を作成するときに、次のエラーが表示されることがあります。**仮想マシン 'vm-name' を起動できませんでした。エラー:Failed to update serial output settings for VM 'vm-name'** . (VM 'vm-name' のシリアル出力設定を更新できませんでした。) このエラーは、VM でブート診断を有効にしても、ブート診断ストレージ アカウントを削除した場合に発生します。
- 修復: 以前使用したものと同じ名前のストレージ アカウントを再作成します。
- 発生頻度: 共通

### <a name="virtual-machine-scale-set"></a>仮想マシン スケール セット

#### <a name="create-failures-during-patch-and-update-on-4-node-azure-stack-environments"></a>4 ノードの Azure Stack 環境でのパッチと更新プログラムの適用中に作成が失敗します

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因:4 ノードの Azure Stack 環境では、障害ドメインが 3 つの可用性セット内での VM の作成および仮想マシン スケール セット インスタンスの作成が更新プロセス中に **FabricVmPlacementErrorUnsupportedFaultDomainSize** エラーで失敗します。
- 修復: 障害ドメインが 2 つの可用性セット内には 1 つの VM を正常に作成できます。 ただし、4 ノードの Azure Stack では、依然として更新プロセス中にスケール セット インスタンスを作成することはできません。

### <a name="ubuntu-ssh-access"></a>Ubuntu SSH アクセス

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: SSH の承認を有効にして作成した Ubuntu 18.04 VM では、SSH キーを使用してサインインすることはできません。
- 修復: プロビジョニング後に Linux 拡張機能用の VM アクセスを使用して SSH キーを実装するか、パスワードベースの認証を使用します。
- 発生頻度: 共通

### <a name="virtual-machine-scale-set-reset-password-does-not-work"></a>仮想マシン スケール セットのパスワードのリセットが機能しない

- 適用先:この問題は、1906 および 1907 リリースに適用されます。
- 原因:新しいパスワードのリセット ブレードはスケール セット UI には表示されますが、Azure Stack ではスケール セットでのパスワードのリセットがまだサポートされていません。
- 修復: [なし] :
- 発生頻度: 共通

### <a name="rainy-cloud-on-scale-set-diagnostics"></a>スケール セット診断の雨雲

- 適用先:この問題は、1906 および 1907 リリースに適用されます。
- 原因: 仮想マシン スケール セットの概要ページに、空のグラフが表示されます。 空のグラフをクリックすると、"雨雲" ブレードが開きます。 これは CPU 使用率などのスケール セット診断情報のグラフで、現在の Azure Stack ビルドでサポートされている機能ではありません。
- 修復: [なし] :
- 発生頻度: 共通

### <a name="virtual-machine-diagnostic-settings-blade"></a>仮想マシンの診断設定ブレード

- 適用先:この問題は、1906 および 1907 リリースに適用されます。    
- 原因: 仮想マシンの診断設定ブレードには、**Application Insight アカウント** を求める **[シンク]** タブがあります。 これは新しいブレードの結果で、Azure Stack ではまだサポートされていません。
- 修復: [なし] :
- 発生頻度: 共通
