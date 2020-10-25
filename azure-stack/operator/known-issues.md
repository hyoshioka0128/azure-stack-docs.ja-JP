---
title: Azure Stack Hub の既知の問題
description: Azure Stack Hub リリースの既知の問題について説明します。
author: sethmanheim
ms.topic: article
ms.date: 09/28/2020
ms.author: sethm
ms.reviewer: sranthar
ms.lastreviewed: 08/13/2020
ms.openlocfilehash: b52944255569197e9390db879f690f9e5d5a21d5
ms.sourcegitcommit: 703be61f2f1565bf478b8c184753869c29e5c33c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/29/2020
ms.locfileid: "91495781"
---
# <a name="azure-stack-hub-known-issues"></a>Azure Stack Hub の既知の問題

この記事では、Azure Stack Hub のリリースにおける既知の問題を一覧で示します。 新しい問題が特定されると、この一覧は更新されます。

別のバージョンの既知の問題にアクセスするには、左側の目次の上部にあるバージョン セレクターのドロップダウンを使用します。

::: moniker range=">=azs-1910"
> [!IMPORTANT]  
> 更新プログラムを適用する前に、このセクションを確認してください。
::: moniker-end
::: moniker range="<azs-1910"
> [!IMPORTANT]  
> お使いの Azure Stack Hub インスタンスが 2 つ前の更新プログラムより古い場合、コンプライアンスに対応していないとみなされます。 [サポートを受けるためには、少なくともサポートされる最小バージョンまで更新する](azure-stack-servicing-policy.md#keep-your-system-under-support)必要があります。 
::: moniker-end

<!---------------------------------------------------------->
<!------------------- SUPPORTED VERSIONS ------------------->
<!---------------------------------------------------------->

::: moniker range="azs-2005"
## <a name="update"></a>更新

Azure Stack Hub の更新に関する既知の問題については、[Azure Stack Hub の更新プログラムに関するトラブルシューティング](azure-stack-troubleshooting.md#troubleshoot-azure-stack-hub-updates)に関する記事を参照してください。

## <a name="portal"></a>ポータル

### <a name="administrative-subscriptions"></a>管理サブスクリプション

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: バージョン 1804 で導入された 2 つの管理サブスクリプションは使用しないでください。 このサブスクリプションの種類は **Metering** サブスクリプションと **Consumption** サブスクリプションです。
- 修復: これら 2 つのサブスクリプション上でリソースが実行されている場合は、ユーザー サブスクリプションで再作成してください。
- 発生頻度: 共通

## <a name="networking"></a>ネットワーク

### <a name="network-security-groups"></a>ネットワーク セキュリティ グループ

#### <a name="cannot-delete-an-nsg-if-nics-not-attached-to-running-vm"></a>実行中の VM に NIC が接続されていない場合、NSG を削除できない

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: NSG と、実行中の VM に接続されていない NIC の関連付けを解除すると、そのオブジェクトの更新 (PUT) 操作がネットワーク コントローラー レイヤーで失敗します。 NSG はネットワーク リソース プロバイダー レイヤーで更新されますが、ネットワーク コントローラー上では更新されないため、NSG はエラー状態に移行します。
- 修復: 削除する必要がある NSG に関連付けられている NIC を、実行中の VM に接続し、NSG の関連付けを解除するか、NSG に関連付けられている NIC をすべて削除します。
- 発生頻度: 共通

#### <a name="denyalloutbound-rule-cannot-be-created"></a>DenyAllOutbound ルールを作成できません

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: VM の作成中は、インターネットに対する明示的な **DenyAllOutbound** 規則を NSG に作成することはできません。VM のデプロイを完了するために必要な通信が妨げられるためです。
- 修復: VM の作成中はインターネットのアウトバウンド トラフィックを許可しておき、VM の作成後に、必要なトラフィックをブロックするよう NSG を変更してください。
- 発生頻度: 共通

### <a name="virtual-network-gateway"></a>Virtual Network ゲートウェイ

#### <a name="documentation"></a>ドキュメント

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: 仮想ネットワーク ゲートウェイの概要ページにあるドキュメントのリンクは、Azure Stack Hub ではなく Azure 固有のドキュメントにリンクされています。 Azure Stack Hub のドキュメントについては、次のリンクを使用してください。

  - [ゲートウェイ SKU](../user/azure-stack-vpn-gateway-about-vpn-gateways.md#gateway-skus)
  - [高可用性接続](../user/azure-stack-vpn-gateway-about-vpn-gateways.md#gateway-availability)
  - [Azure Stack Hub での BGP の構成](../user/azure-stack-vpn-gateway-settings.md#gateway-requirements)
  - [ExpressRoute 回線](azure-stack-connect-expressroute.md)
  - [カスタムの IPsec/IKE ポリシーの指定](../user/azure-stack-vpn-gateway-settings.md#ipsecike-parameters)
  
### <a name="load-balancer"></a>Load Balancer

#### <a name="load-balancer-directing-traffic-to-one-backend-vm-in-specific-scenarios"></a>Load Balancer が特定のシナリオでトラフィックを 1 つのバックエンド VM に送信する

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。 
- 原因: ロード バランサーで**セッション アフィニティ**を有効にすると、2 タプル ハッシュで、VM に割り当てられたプライベート IP ではなく、PA IP (物理アドレス IP) が使用されます。 ロード バランサーに送信されるトラフィックが VPN 経由で到着する、またはすべてのクライアント VM (ソース IP) が同じノード上に存在し、セッション アフィニティが有効になっているシナリオでは、すべてのトラフィックが 1 つのバックエンド VM に送信されます。
- 発生頻度: 共通

#### <a name="public-ip"></a>パブリック IP

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: ロード バランサーに関連付けられているパブリック IP の **IdleTimeoutInMinutes** 値は変更できません。 この操作により、パブリック IP は失敗状態になります。
- 修復: パブリック IP を正常な状態に戻すには、パブリック IP を参照するロード バランサー規則の **IdleTimeoutInMinutes** 値を元の値に戻します (既定値は 4 分です)。
- 発生頻度: 共通

## <a name="compute"></a>Compute

### <a name="issues-deploying-virtual-machine-scale-set-with-standard_ds2_v2-size-using-the-portal"></a>ポータルを使用した、Standard_DS2_v2 サイズでの仮想マシン スケール セットのデプロイに関する問題

- 適用先:この問題は 2005 リリースに適用されます。
- 原因:ポータルのバグにより、Standard_DS2_v2 サイズでのスケール セットの作成が失敗します。
- 修復: PowerShell または CLI を使用して、仮想マシン スケール セットのこの VM サイズをデプロイします。

### <a name="issues-using-vm-extensions-in-ubuntu-server-2004"></a>Ubuntu Server 20.04 での VM 拡張機能の使用に関する問題

- 適用先:この問題は **Ubuntu Server 20.04 LTS** に適用されます。
- 原因: 一部の Linux ディストリビューションは Python 3.8 に移行し、Python 用のレガシ `/usr/bin/python` エントリポイントは完全に削除されました。 Python 3.x に移行した Linux ディストリビューション ユーザーは、これらの拡張機能を VM にデプロイする前に、レガシ `/usr/bin/python` エントリポイントが確実に存在するようにする必要があります。 そうしないと、拡張機能のデプロイが失敗する可能性があります。
- 修復: 「[Python 3 対応 Linux Azure Virtual Machines システムでの VM 拡張機能の使用に関する問題](/azure/virtual-machines/extensions/issues-using-vm-extensions-python-3)」の解決手順に従います。ただし、Azure Stack Hub には **実行コマンド**機能がないため手順 2 はスキップします。

### <a name="nvv4-vm-size-on-portal"></a>ポータル上の NVv4 VM のサイズ

- 適用先:この問題は、2002 以降に適用されます。
- 原因: VM の作成エクスペリエンスを実行すると、次の VM サイズが表示されます:NV4as_v4。 AMD Mi25 ベースの Azure Stack Hub GPU プレビューに必要なハードウェアを所有しているお客様は、VM のデプロイを成功させることができます。 他のすべてのお客様は、この VM サイズでの VM のデプロイに失敗します。
- 修復: Azure Stack Hub GPU プレビューの準備のための仕様です。

### <a name="consumed-compute-quota"></a>コンピューティング クォータの消費

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: 新しい仮想マシンを作成すると、"**This subscription is at capacity for Total Regional vCPUs on this location.This subscription is using all 50 Total Regional vCPUs available.** " (このサブスクリプションはこの場所の [リージョンの vCPU の合計] の容量に達しています。このサブスクリプションは、[リージョンの vCPU の合計] の使用可能な 50 個をすべて使用しています。)" などのエラーを受け取ることがあります。 これは、使用可能な合計コア数のクォータに達したことを示します。
- 修復: クォータを追加したアドオン プランをオペレーターに依頼してください。 現在のプランのクォータの編集では解決せず、クォータの増加は反映されません。
- 発生頻度: 珍しい

### <a name="vm-overview-blade-does-not-show-correct-computer-name"></a>VM の概要ブレードに正しいコンピューター名が表示されない

- 適用先:この問題はすべてのリリースに適用されます。
- 原因: 概要ブレードで VM の詳細を表示すると、コンピューター名に **(使用できません)** と表示されます。 これは、特殊なディスクまたはディスク スナップショットから作成された VM の設計によるもので、Marketplace イメージでも発生します。
- 修復: **[設定]** の下にある **[プロパティ]** ブレードを確認します。

### <a name="virtual-machine-scale-set"></a>仮想マシン スケール セット

#### <a name="create-failures-during-patch-and-update-on-4-node-azure-stack-hub-environments"></a>4 ノードの Azure Stack Hub 環境では修正プログラムや更新プログラムの適用中に作成が失敗する

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: 4 ノードの Azure Stack Hub 環境では、障害ドメインが 3 つの可用性セット内での VM の作成および仮想マシン スケール セット インスタンスの作成が、更新プロセス中に **FabricVmPlacementErrorUnsupportedFaultDomainSize** エラーで失敗します。
- 修復: 障害ドメインが 2 つの可用性セット内には 1 つの VM を正常に作成できます。 ただし、4 ノードの Azure Stack Hub のデプロイでは、依然として更新プロセス中にスケール セット インスタンスを作成することはできません。

## <a name="storage"></a>ストレージ

### <a name="retention-period-reverts-to-0"></a>保持期間が 0 に戻る

- 適用先:この問題は、リリース 2002 および 2005 に適用されます。
- 原因: 保持期間の設定で 0 以外の期間を指定した場合は、2002 または 2005 の更新中に 0 (この設定の既定値) に戻ります。 0 日の設定は、更新が完了した直後に有効になります。これにより、削除された既存のストレージ アカウントと今後新たに削除されるストレージ アカウントはすべて、すぐに保持期間外となり、定期的なガベージ コレクション (1 時間ごとに実行) のマークが付けられます。
- 修復: 保持期間を正しい期間に手動で指定してください。 新しい保持期間が指定される前に既にガベージ コレクションされているストレージ アカウントは回復できません。  

## <a name="resource-providers"></a>リソース プロバイダー

### <a name="sqlmysql"></a>SQL/MySQL

- 適用先:この問題は、リリース 2002 に適用されます。
- 原因: スタンプに SQL リソースプロバイダー (RP) バージョン 1.1.33.0 以前が含まれている場合は、スタンプの更新時に SQL または MySQL のブレードが読み込まれません。
- 修復: RP をバージョン 1.1.47.0 に更新します。

### <a name="app-service"></a>App Service

- 適用先:この問題は、リリース 2002 に適用されます。
- 原因: スタンプに App Service リソース プロバイダー (RP) バージョン 1.7 以前が含まれている場合は、スタンプの更新時に App Service のブレードが読み込まれません。
- 修復: RP をバージョン 2002 Q2 に更新します。

<!-- ## Storage -->
<!-- ## SQL and MySQL-->
<!-- ## App Service -->
<!-- ## Usage -->
<!-- ### Identity -->
<!-- ### Marketplace -->
::: moniker-end

::: moniker range="azs-2002"
## <a name="update"></a>更新

2002 更新プログラムを適用した後、"Invalid Time Source"(無効なタイム ソース) のアラートが管理者ポータルに誤って表示される場合があります。 この誤検知アラートは無視してもかまいません。今後のリリースで修正される予定です。 

Azure Stack Hub の更新に関する既知の問題については、[Azure Stack Hub の更新プログラムに関するトラブルシューティング](azure-stack-troubleshooting.md#troubleshoot-azure-stack-hub-updates)に関する記事を参照してください。

## <a name="portal"></a>ポータル

### <a name="administrative-subscriptions"></a>管理サブスクリプション

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: バージョン 1804 で導入された 2 つの管理サブスクリプションは使用しないでください。 このサブスクリプションの種類は **Metering** サブスクリプションと **Consumption** サブスクリプションです。
- 修復: これら 2 つのサブスクリプション上でリソースが実行されている場合は、ユーザー サブスクリプションで再作成してください。
- 発生頻度: 共通

### <a name="subscription-permissions"></a>サブスクリプションのアクセス許可

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: Azure Stack Hub ポータルを使用して、サブスクリプションに対するアクセス許可を表示することはできません。
- 修復: [PowerShell を使用してアクセス許可を確認](/powershell/module/azurerm.resources/get-azurermroleassignment)します。
- 発生頻度: 共通

### <a name="storage-account-options"></a>ストレージ アカウントのオプション

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: ユーザー ポータルでは、ストレージ アカウントの名前が **[ストレージ アカウント - Blob、File、Table、Queue]** と表示されますが、**File** は Azure Stack Hub ではサポートされていません。
- 発生頻度: 共通

### <a name="create-managed-disk-snapshot"></a>マネージド ディスク スナップショットの作成

- 適用先:この問題は、リリース 2002 に適用されます。
- 原因: ユーザー ポータルで、マネージド ディスク スナップショットを作成するときに、 **[アカウントの種類]** ボックスは空になります。 アカウントの種類が空のときに **[作成]** ボタンを選択すると、スナップショットの作成は失敗します。
- 修復: **[アカウントの種類]** ドロップダウンリストからアカウントの種類を選択し、スナップショットを作成します。
- 発生頻度: 共通

### <a name="alert-for-network-interface-disconnected"></a>ネットワーク インターフェイスの切断についてのアラート

- 適用先:この問題は、1908 以降に適用されます。
- 原因: ケーブルがネットワーク アダプターから切り離されたときに、管理者ポータルにアラートが表示されません。 この問題は、Windows Server 2019 ではこの障害が既定で無効にされるために発生します。
- 発生頻度: 共通

## <a name="networking"></a>ネットワーク

### <a name="denyalloutbound-rule-cannot-be-created"></a>DenyAllOutbound ルールを作成できません

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: VM の作成中は、インターネットに対する明示的な **DenyAllOutbound** 規則を NSG に作成することはできません。VM のデプロイを完了するために必要な通信が妨げられるためです。
- 修復: VM の作成中はインターネットのアウトバウンド トラフィックを許可しておき、VM の作成後に、必要なトラフィックをブロックするよう NSG を変更してください。
- 発生頻度: 共通

### <a name="icmp-protocol-not-supported-for-nsg-rules"></a>ICMP プロトコルが NSG ルールでサポートされない

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。 
- 原因: 受信または送信のネットワーク セキュリティ規則を作成するとき、 **[プロトコル]** オプションに **[ICMP]** オプションが表示されます。 現在、これは Azure Stack Hub ではサポートされていません。 この問題は修正されており、次の Azure Stack Hub リリースでは表示されません。
- 発生頻度: 共通

### <a name="cannot-delete-an-nsg-if-nics-not-attached-to-running-vm"></a>実行中の VM に NIC が接続されていない場合、NSG を削除できない

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: NSG と、実行中の VM に接続されていない NIC の関連付けを解除すると、そのオブジェクトの更新 (PUT) 操作がネットワーク コントローラー レイヤーで失敗します。 NSG はネットワーク リソース プロバイダー レイヤーで更新されますが、ネットワーク コントローラー上では更新されないため、NSG はエラー状態に移行します。
- 修復: 削除する必要がある NSG に関連付けられている NIC を、実行中の VM に接続し、NSG の関連付けを解除するか、NSG に関連付けられている NIC をすべて削除します。
- 発生頻度: 共通

### <a name="load-balancer-directing-traffic-to-one-backend-vm-in-specific-scenarios"></a>Load Balancer が特定のシナリオでトラフィックを 1 つのバックエンド VM に送信する 

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。 
- 原因: ロード バランサーで**セッション アフィニティ**を有効にすると、2 タプル ハッシュで、VM に割り当てられたプライベート IP ではなく、PA IP (物理アドレス IP) が使用されます。 ロード バランサーに送信されるトラフィックが VPN 経由で到着する、またはすべてのクライアント VM (ソース IP) が同じノード上に存在し、セッション アフィニティが有効になっているシナリオでは、すべてのトラフィックが 1 つのバックエンド VM に送信されます。
- 発生頻度: 共通

### <a name="network-interface"></a>ネットワーク インターフェイス

#### <a name="addingremoving-network-interface"></a>ネットワーク インターフェイスの追加/削除

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: 新しいネットワーク インターフェイスを、**実行**状態にある VM に追加することはできません。
- 修復: 仮想マシンを停止してから、ネットワーク インターフェイスを追加または削除します。
- 発生頻度: 共通

#### <a name="primary-network-interface"></a>プライマリ ネットワーク インターフェイス

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: VM のプライマリ NIC を変更することはできません。 プライマリ NIC を削除またはデタッチすると、VM の起動時に問題が発生します。
- 発生頻度: 共通

### <a name="public-ip"></a>パブリック IP

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: ロード バランサーに関連付けられているパブリック IP の **IdleTimeoutInMinutes** 値は変更できません。 この操作により、パブリック IP は失敗状態になります。
- 修復: パブリック IP を正常な状態に戻すには、パブリック IP を参照するロード バランサー規則の **IdleTimeoutInMinutes** 値を元の値に戻します (既定値は 4 分です)。
- 発生頻度: 共通

### <a name="virtual-network-gateway"></a>Virtual Network ゲートウェイ

#### <a name="documentation"></a>ドキュメント

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: 仮想ネットワーク ゲートウェイの概要ページにあるドキュメントのリンクは、Azure Stack Hub ではなく Azure 固有のドキュメントにリンクされています。 Azure Stack Hub のドキュメントについては、次のリンクを使用してください。

  - [ゲートウェイ SKU](../user/azure-stack-vpn-gateway-about-vpn-gateways.md#gateway-skus)
  - [高可用性接続](../user/azure-stack-vpn-gateway-about-vpn-gateways.md#gateway-availability)
  - [Azure Stack Hub での BGP の構成](../user/azure-stack-vpn-gateway-settings.md#gateway-requirements)
  - [ExpressRoute 回線](azure-stack-connect-expressroute.md)
  - [カスタムの IPsec/IKE ポリシーの指定](../user/azure-stack-vpn-gateway-settings.md#ipsecike-parameters)

## <a name="compute"></a>Compute

### <a name="cannot-create-a-virtual-machine-scale-set-with-standard_ds2_v2-vm-size-on-portal"></a>ポータルで Standard_DS2_v2 VM サイズの仮想マシン スケール セットを作成できない

- 適用先:この問題は 2002 リリースに適用されます。
- 原因:ポータルにバグがあり、Standard_DS2_v2 VM サイズで仮想マシン スケール セットを作成できません。 作成すると、「"{"code":"DeploymentFailed","message":"少なくとも 1 つのリソースのデプロイ操作に失敗しました」というエラーが表示されます。 詳細については、デプロイ操作の一覧を表示してください。 詳しい使用方法については https://aka.ms/arm-debug を参照してください。","details":[{"code":"BadRequest","message":"{\r\n \" error\": {\r\n \" code\":\" NetworkProfileValidationError\" ,\r\n \" message\":\" 仮想マシン サイズ Standard_DS2_v2 は、VM スケール セット /subscriptions/x/resourceGroups/RGVMSS/providers/Microsoft.Compute/virtualMachineScaleSets/vmss の VM のインデックス 0 で有効にする高速ネットワークに許可される VM サイズの一覧に記載されていません。 許可されるサイズ: .\"\r\n }\r\n}"}]}"
- 修復: PowerShell またはリソース マネージャー テンプレートを使用して仮想マシン スケール セットを作成します。

### <a name="vm-overview-blade-does-not-show-correct-computer-name"></a>VM の概要ブレードに正しいコンピューター名が表示されない

- 適用先:この問題はすべてのリリースに適用されます。
- 原因: 概要ブレードで VM の詳細を表示すると、コンピューター名に **(使用できません)** と表示されます。 これは、特殊なディスクまたはディスク スナップショットから作成された VM の設計によるもので、Marketplace イメージでも発生します。
- 修復: **[設定]** の下にある **[プロパティ]** ブレードを確認します。

### <a name="nvv4-vm-size-on-portal"></a>ポータル上の NVv4 VM のサイズ

- 適用先:この問題は 2002 以降のリリースに適用されます。
- 原因: VM の作成エクスペリエンスを実行すると、次の VM サイズが表示されます:NV4as_v4。 AMD Mi25 ベースの Azure Stack Hub GPU プレビューに必要なハードウェアを所有しているお客様は、VM のデプロイを成功させることができます。 他のすべてのお客様は、この VM サイズでの VM のデプロイに失敗します。
- 修復: Azure Stack Hub GPU プレビューの準備のための仕様です。

### <a name="vm-boot-diagnostics"></a>VM ブート診断

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: 新しい仮想マシン (VM) を作成するときに、次のエラーが表示されることがあります。**仮想マシン 'vm-name' を起動できませんでした。エラー:Failed to update serial output settings for VM 'vm-name'** . (VM 'vm-name' のシリアル出力設定を更新できませんでした。) このエラーは、VM でブート診断を有効にしても、ブート診断ストレージ アカウントを削除した場合に発生します。
- 修復: 以前使用したものと同じ名前のストレージ アカウントを再作成します。
- 発生頻度: 共通

### <a name="vm-boot-diagnostics"></a>VM ブート診断

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: 停止解除された仮想マシンを起動しようとすると、次のエラーが表示されることがあります。**VM diagnostics Storage account 'diagnosticstorageaccount' not found. (VM 診断ストレージ アカウント 'diagnosticstorageaccount' が見つかりません。)Ensure storage account is not deleted**. (ストレージ アカウントが削除されていないことを確認してください。) ブート診断が有効になっている VM を起動しようとしたときに、参照されるブート診断ストレージ アカウントが削除されていると、このエラーが発生します。
- 修復: 以前使用したものと同じ名前のストレージ アカウントを再作成します。
- 発生頻度: 共通

### <a name="consumed-compute-quota"></a>コンピューティング クォータの消費

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: 新しい仮想マシンを作成すると、"**This subscription is at capacity for Total Regional vCPUs on this location.This subscription is using all 50 Total Regional vCPUs available.** " (このサブスクリプションはこの場所の [リージョンの vCPU の合計] の容量に達しています。このサブスクリプションは、[リージョンの vCPU の合計] の使用可能な 50 個をすべて使用しています。)" などのエラーを受け取ることがあります。 これは、使用可能な合計コア数のクォータに達したことを示します。
- 修復: クォータを追加したアドオン プランをオペレーターに依頼してください。 現在のプランのクォータの編集では解決せず、クォータの増加は反映されません。
- 発生頻度: 珍しい

### <a name="virtual-machine-scale-set"></a>仮想マシン スケール セット

#### <a name="create-failures-during-patch-and-update-on-4-node-azure-stack-hub-environments"></a>4 ノードの Azure Stack Hub 環境では修正プログラムや更新プログラムの適用中に作成が失敗する

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: 4 ノードの Azure Stack Hub 環境では、障害ドメインが 3 つの可用性セット内での VM の作成および仮想マシン スケール セット インスタンスの作成が、更新プロセス中に **FabricVmPlacementErrorUnsupportedFaultDomainSize** エラーで失敗します。
- 修復: 障害ドメインが 2 つの可用性セット内には 1 つの VM を正常に作成できます。 ただし、4 ノードの Azure Stack Hub のデプロイでは、依然として更新プロセス中にスケール セット インスタンスを作成することはできません。

### <a name="sql-vm"></a>SQL VM

#### <a name="storage-account-creating-failure-when-configuring-auto-backup"></a>自動バックアップを構成するときにストレージ アカウントの作成がエラーになる

- 適用先:この問題は 2002 に適用されます。
- 原因: 新しいストレージ アカウントを使用して SQL VM の自動バックアップを構成すると、エラー "**Deployment template validation failed. The template parameter for 'SqlAutobackupStorageAccountKind' is not found**" (デプロイ テンプレートの検証に失敗しました。'SqlAutobackupStorageAccountKind' のテンプレート パラメーターが見つかりません) が発生して失敗します。
- 修復: 最新の 2002 修正プログラムを適用します。

#### <a name="auto-backup-cannot-be-configured-with-tls-12-enabled"></a>TLS 1.2 が有効になっていると自動バックアップを構成できない

- 適用先:この問題は、2002 以降の新規インストール、または TLS 1.2 が有効になっている以前のリリースに適用されます。
- 原因: 既存のストレージ アカウントを使用して SQL VM の自動バックアップを構成すると、次のエラーで失敗します。**SQL Server IaaS Agent:基になる接続が閉じられました。送信時に、予期しないエラーが発生しました。**
- 発生頻度: 共通

## <a name="storage"></a>ストレージ

### <a name="retention-period-revert-to-0"></a>保持期間が 0 に戻る

- 適用先:この問題は、リリース 2002 および 2005 に適用されます。
- 原因: 保持期間の設定で 0 以外の期間を以前に指定した場合、2002 と 2005 の更新中に 0 (この設定の既定値) に戻されます。 0 日の設定は、更新が完了した直後に有効になります。これにより、削除された既存のストレージ アカウントと今後新たに削除されるストレージ アカウントはすべて、すぐに保持期間外となり、定期的なガベージ コレクション (1 時間ごとに実行) のマークが付けられます。 
- 修復: 保持期間を適切な期間に手動で指定してください。 ただし、新しい保持期間が指定される前に既にガベージ コレクションされているストレージ アカウントは回復できません。  

## <a name="resource-providers"></a>リソース プロバイダー

### <a name="sqlmysql"></a>SQL/MySQL

- 適用先:この問題は、リリース 2002 に適用されます。 
- 原因: スタンプに SQL リソース プロバイダー (RP) バージョン 1.1.33.0 以前が含まれている場合は、スタンプの更新時に SQL/MySQL のブレードが読み込まれません。
- 修復: RP をバージョン 1.1.47.0 に更新します。

### <a name="app-service"></a>App Service

- 適用先:この問題は、リリース 2002 に適用されます。
- 原因: スタンプに App Service リソース プロバイダー (RP) バージョン 1.7 以前が含まれている場合は、スタンプの更新時に App Service のブレードが読み込まれません。
- 修復: RP をバージョン [2020 Q2](azure-stack-app-service-update.md) に更新します。

<!-- ## Storage -->
<!-- ## SQL and MySQL-->
<!-- ## App Service -->
<!-- ## Usage -->
<!-- ### Identity -->
<!-- ### Marketplace -->
::: moniker-end

::: moniker range="azs-1910"
## <a name="update"></a>更新

Azure Stack Hub の更新に関する既知の問題については、[Azure Stack Hub の更新プログラムに関するトラブルシューティング](azure-stack-troubleshooting.md#troubleshoot-azure-stack-hub-updates)に関する記事を参照してください。

## <a name="portal"></a>ポータル

### <a name="administrative-subscriptions"></a>管理サブスクリプション

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: バージョン 1804 で導入された 2 つの管理サブスクリプションは使用しないでください。 このサブスクリプションの種類は **Metering** サブスクリプションと **Consumption** サブスクリプションです。
- 修復: これら 2 つのサブスクリプション上でリソースが実行されている場合は、ユーザー サブスクリプションで再作成してください。
- 発生頻度: 共通

### <a name="duplicate-subscription-button-in-lock-blade"></a>[ロック] ブレードの重複する [サブスクリプション] ボタン

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: 管理者ポータルのユーザー サブスクリプションの **[ロック]** ブレードには、**サブスクリプション**という 2 つのボタンがあります。
- 発生頻度: 共通

### <a name="subscription-permissions"></a>サブスクリプションのアクセス許可

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: Azure Stack Hub ポータルを使用して、サブスクリプションに対するアクセス許可を表示することはできません。
- 修復: [PowerShell を使用してアクセス許可を確認](/powershell/module/azurerm.resources/get-azurermroleassignment)します。
- 発生頻度: 共通

### <a name="storage-account-settings"></a>Storage アカウントの設定

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: ユーザー ポータルで、ストレージ アカウントの **[構成]** ブレードに、**安全な転送の種類**を変更するオプションが表示されます。 この機能は現在、Azure Stack Hub でサポートされていません。
- 発生頻度: 共通

### <a name="upload-blob-with-oauth-error"></a>OAuth を使用した BLOB のアップロードのエラー

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: ユーザー ポータルで **[OAuth(preview)]\(OAuth (プレビュー)\)** オプションを使用して BLOB をアップロードしようとすると、タスクがエラー メッセージにより失敗します。
- 修復: SAS オプションを使用して BLOB をアップロードします。
- 発生頻度: 共通

### <a name="upload-blob-option-unsupported"></a>サポートされていない BLOB のアップロード オプション

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: ユーザー ポータルのアップロード ブレードで BLOB をアップロードしようとすると、 **[AAD]** または **[キー認証]** を選択するオプションが表示されますが、Azure Stack Hub では **[AAD]** はサポートされていません。
- 発生頻度: 共通

### <a name="alert-for-network-interface-disconnected"></a>ネットワーク インターフェイスの切断についてのアラート

- 適用先:この問題は、1908 以降に適用されます。
- 原因: ケーブルがネットワーク アダプターから切り離されたときに、管理者ポータルにアラートが表示されません。 この問題は、Windows Server 2019 ではこの障害が既定で無効にされるために発生します。
- 発生頻度: 共通

### <a name="incorrect-tooltip-when-creating-vm"></a>VM 作成時のヒントが正しくない

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: ユーザー ポータルで、Premium SSD のディスクの種類でマネージド ディスクを選択すると、ドロップダウン リストに **[OS ディスク]** と表示されます。 そのオプションの横には、"**Azure の無料アカウントでは、一定の OS ディスク サイズを無料で利用できます**" というヒントが表示されますが、Azure Stack Hub はこの対象外です。 さらに、リストには **[Free account eligible] (無料アカウント利用可能)** が含まれていますが、これも Azure Stack Hub は対象外です。
- 発生頻度: 共通

### <a name="delete-a-storage-container"></a>ストレージ コンテナーを削除する

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: ユーザー ポータルでストレージ コンテナーを削除しようとしたときに、 **[Azure Policy と RBAC 役割の設定を上書きする]** を切り替えないと、操作が失敗します。
- 修復: **[Azure Policy と RBAC 役割の設定を上書きする]** チェック ボックスがオンになっていることを確認します。
- 発生頻度: 共通

### <a name="refresh-button-on-virtual-machines-fails"></a>仮想マシンの [更新] ボタンが失敗する

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: ユーザー ポータルで **[Virtual Machines]** に移動し、上部のボタンを使用して更新しようとすると、状態が正確に更新されません。
- 修復: [更新] ボタンがクリックされたかどうかに関係なく、状態は 5 分ごとに自動的に更新されます。 5 分待ってから状態を確認してください。
- 発生頻度: 共通

### <a name="storage-account-options"></a>ストレージ アカウントのオプション

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: ユーザー ポータルでは、ストレージ アカウントの名前が **[ストレージ アカウント - Blob、File、Table、Queue]** と表示されますが、**File** は Azure Stack Hub ではサポートされていません。
- 発生頻度: 共通

### <a name="storage-account-configuration"></a>Storage account configuration

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: ユーザー ポータルでストレージ アカウントを作成し、その **[構成]** を表示すると、AJAX エラーが発生するため、構成の変更を保存できません。
- 発生頻度: 共通

### <a name="capacity-monitoring-in-sql-resource-provider-keeps-loading"></a>SQL リソース プロバイダーの容量監視で読み込み処理が続く

- 適用先:この問題は、SQL リソース プロバイダー バージョン 1.1.33.0 以前がインストールされた、Azure Stack Hub 1910 更新プログラム以降に適用されます。
- 原因: SQL リソース プロバイダーの現在のバージョンは、1910 更新プログラムに含まれる最新のポータル変更の一部と互換性がありません。
- 修復: Azure Stack Hub が 1910 更新プログラム ([SQL RP バージョン 1.1.47.0](https://aka.ms/azurestacksqlrp11470)) にアップグレードされた後、リソース プロバイダーの更新プロセスに従って、SQL リソース プロバイダーの修正プログラム 1.1.47.0 を適用します。 MySQL リソース プロバイダーの場合、Azure Stack Hub を 1910 更新プログラム ([MySQL RP バージョン 1.1.47.0](https://aka.ms/azurestackmysqlrp11470)) にアップグレードした後、MySQL リソース プロバイダーの修正プログラム 1.1.47.0 を適用することもお勧めします。
- 発生頻度: 共通

### <a name="access-control-iam"></a>アクセス制御 (IAM)

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: IAM 拡張機能が古すぎます。 Azure Stack Hub に付属している Ibiza ポータルで導入された新しい動作は、ユーザーがグローバル サブスクリプション セレクター (ユーザー ポータルの **[ディレクトリ + サブスクリプション]** ) 内で選択されていないサブスクリプションの **[アクセス制御 (IAM)]** ブレードを開いている場合に、RBAC 拡張機能が失敗する原因となります。 このブレードには **[読み込み中]** がループで表示され、ユーザーはサブスクリプションに新しいロールを追加できません。 **[追加]** ブレードにも、 **[読み込み中]** がループで表示されます。
- 修復: **[ディレクトリ + サブスクリプション]** メニューでサブスクリプションがオンになっていることを確認します。 このメニューには、ポータルの上部から、または **[通知]** ボタンの近くからアクセスするか、 **[すべてのリソース]** ブレードのショートカット ( **[サブスクリプションが表示されていませんか?ディレクトリとサブスクリプションの設定を開きます]** と表示されている) を使用してアクセスします。 このメニューで、サブスクリプションを選択する必要があります。

### <a name="sql-resource-provider"></a>SQL リソース プロバイダー

- 適用先:この問題は、1908 以前を実行しているスタンプに適用されます。
- 原因: SQL リソース プロバイダー (RP) バージョン 1.1.47.0 のデプロイ時には、その SQL RP に関連付けられているもの以外の資産はポータルに表示されません。
- 修復: その RP を削除し、スタンプをアップグレードして、SQL RP を再デプロイします。

### <a name="activity-log-blade"></a>[アクティビティ ログ] ブレード

- 適用先:この問題は、1907 以降を実行しているスタンプに適用されます。 <!-- Note: Applies to 2002 as well -->
- 原因: アクティビティ ログにアクセスすると、ポータルにはエントリの最初のページのみが表示されます。 **[Load more results]\(さらに結果を読み込む\)** を選択しても追加エントリは読み込まれません。
- 修復: 最初のページよりも後にあるエントリを確認するには、フィルターの時間範囲を調整します。

## <a name="networking"></a>ネットワーク

### <a name="load-balancer"></a>Load Balancer

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: 可用性セットの VM をロード バランサーのバックエンド プールに追加すると、"**ロード バランサー バックエンド プールを保存できませんでした**" というエラー メッセージがポータルに表示されます。 これはポータルの表面的な問題です。機能は有効であり、内部的には VM はバックエンド プールに正常に追加されます。
- 発生頻度: 共通

### <a name="network-security-groups"></a>ネットワーク セキュリティ グループ

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。 
- 原因: 明示的な **DenyAllOutbound** 規則は、VM のデプロイを完了するために必要なインフラストラクチャへの内部通信がすべて妨げられるため、NSG に作成することはできません。
- 発生頻度: 共通

### <a name="service-endpoints"></a>サービス エンドポイント

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: ユーザー ポータルで、 **[仮想ネットワーク]** ブレードに、**サービス エンドポイント**を使用するオプションが表示されます。 この機能は現在、Azure Stack Hub ではサポートされていません。
- 発生頻度: 共通

### <a name="cannot-delete-an-nsg-if-nics-not-attached-to-running-vm"></a>実行中の VM に NIC が接続されていない場合、NSG を削除できない

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: NSG と、実行中の VM に接続されていない NIC の関連付けを解除すると、そのオブジェクトの更新 (PUT) 操作がネットワーク コントローラー レイヤーで失敗します。 NSG はネットワーク リソース プロバイダー レイヤーで更新されますが、ネットワーク コントローラー上では更新されないため、NSG はエラー状態に移行します。
- 修復: 削除する必要がある NSG に関連付けられている NIC を、実行中の VM に接続し、NSG の関連付けを解除するか、NSG に関連付けられている NIC をすべて削除します。
- 発生頻度: 共通

### <a name="network-interface"></a>ネットワーク インターフェイス

#### <a name="addingremoving-network-interface"></a>ネットワーク インターフェイスの追加/削除

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: 新しいネットワーク インターフェイスを、**実行**状態にある VM に追加することはできません。
- 修復: 仮想マシンを停止してから、ネットワーク インターフェイスを追加または削除します。
- 発生頻度: 共通

#### <a name="primary-network-interface"></a>プライマリ ネットワーク インターフェイス

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: VM のプライマリ NIC を変更することはできません。 プライマリ NIC を削除またはデタッチすると、VM の起動時に問題が発生します。
- 発生頻度: 共通

### <a name="virtual-network-gateway"></a>Virtual Network ゲートウェイ

#### <a name="next-hop-type"></a>次ホップの種類

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: ユーザー ポータルでルート テーブルを作成すると、次のホップの種類オプションの 1 つとして **[Virtual Network ゲートウェイ]** が表示されますが、これは Azure Stack Hub ではサポートされていません。
- 発生頻度: 共通

#### <a name="alerts"></a>警告

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: ユーザー ポータルで、 **[仮想ネットワーク ゲートウェイ]** ブレードに、使用する**アラート** オプションが表示されます。 この機能は現在、Azure Stack Hub ではサポートされていません。
- 発生頻度: 共通

#### <a name="active-active"></a>アクティブ/アクティブ

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: ユーザー ポータルで、作成中、および**仮想ネットワーク ゲートウェイ**のリソース メニューで、**アクティブ/アクティブ**構成を有効にするオプションが表示されます。 この機能は現在、Azure Stack Hub ではサポートされていません。
- 発生頻度: 共通

#### <a name="vpn-troubleshooter"></a>VPN トラブルシューティング ツール

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: ユーザー ポータルで、 **[接続]** ブレードに **VPN トラブルシューティング ツール**と呼ばれる機能が表示されます。 この機能は現在、Azure Stack Hub ではサポートされていません。
- 発生頻度: 共通

#### <a name="vpn-troubleshooter"></a>VPN トラブルシューティング ツール

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: ユーザー ポータルでは、VPN ゲートウェイ リソースに **[VPN のトラブルシューティング]** 機能と **[メトリック]** が表示されますが、これは Azure Stack Hub ではサポートされていません。
- 発生頻度: 共通

#### <a name="documentation"></a>ドキュメント

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: 仮想ネットワーク ゲートウェイの概要ページにあるドキュメントのリンクは、Azure Stack Hub ではなく Azure 固有のドキュメントにリンクされています。 Azure Stack Hub のドキュメントについては、次のリンクを使用してください。

  - [ゲートウェイ SKU](../user/azure-stack-vpn-gateway-about-vpn-gateways.md#gateway-skus)
  - [高可用性接続](../user/azure-stack-vpn-gateway-about-vpn-gateways.md#gateway-availability)
  - [Azure Stack Hub での BGP の構成](../user/azure-stack-vpn-gateway-settings.md#gateway-requirements)
  - [ExpressRoute 回線](azure-stack-connect-expressroute.md)
  - [カスタムの IPsec/IKE ポリシーの指定](../user/azure-stack-vpn-gateway-settings.md#ipsecike-parameters)

## <a name="compute"></a>Compute

### <a name="vm-boot-diagnostics"></a>VM ブート診断

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: 新しい Windows 仮想マシン (VM) を作成するときに、次のエラーが表示されることがあります。**仮想マシン 'vm-name' を起動できませんでした。エラー:Failed to update serial output settings for VM 'vm-name'** . (VM 'vm-name' のシリアル出力設定を更新できませんでした。) このエラーは、VM でブート診断を有効にしても、ブート診断ストレージ アカウントを削除した場合に発生します。
- 修復: 以前使用したものと同じ名前のストレージ アカウントを再作成します。
- 発生頻度: 共通

### <a name="consumed-compute-quota"></a>コンピューティング クォータの消費

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: 新しい仮想マシンを作成すると、"**This subscription is at capacity for Total Regional vCPUs on this location.This subscription is using all 50 Total Regional vCPUs available.** " (このサブスクリプションはこの場所の [リージョンの vCPU の合計] の容量に達しています。このサブスクリプションは、[リージョンの vCPU の合計] の使用可能な 50 個をすべて使用しています。)" などのエラーを受け取ることがあります。 これは、使用可能な合計コア数のクォータに達したことを示します。
- 修復: クォータを追加したアドオン プランをオペレーターに依頼してください。 現在のプランのクォータの編集では解決せず、クォータの増加は反映されません。
- 発生頻度: 珍しい

### <a name="privileged-endpoint"></a>特権エンドポイント

- 適用先:この問題は、1910 以前のリリースに適用されます。
- 原因: 英語以外のバージョンの Windows を実行しているコンピューターから特権エンドポイント (ERC VM) に接続できません。
- 修復: これは、1910 より後のリリースで修正されている既知の問題です。 この回避策として、**en-US** カルチャを使用して、**New-PSSession** および **Enter-PSSession** の PowerShell コマンドレットを実行できます。たとえば、次のスクリプトを使用してカルチャを設定します: https://resources.oreilly.com/examples/9780596528492/blob/master/Use-Culture.ps1 。
- 発生頻度: 珍しい

### <a name="virtual-machine-scale-set"></a>仮想マシン スケール セット

#### <a name="create-failures-during-patch-and-update-on-4-node-azure-stack-hub-environments"></a>4 ノードの Azure Stack Hub 環境では修正プログラムや更新プログラムの適用中に作成が失敗する

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: 4 ノードの Azure Stack Hub 環境では、障害ドメインが 3 つの可用性セット内での VM の作成および仮想マシン スケール セット インスタンスの作成が、更新プロセス中に **FabricVmPlacementErrorUnsupportedFaultDomainSize** エラーで失敗します。
- 修復: 障害ドメインが 2 つの可用性セット内には 1 つの VM を正常に作成できます。 ただし、4 ノードの Azure Stack Hub のデプロイでは、依然として更新プロセス中にスケール セット インスタンスを作成することはできません。

<!-- ## Storage -->
<!-- ## SQL and MySQL-->
<!-- ## App Service -->
<!-- ## Usage -->
<!-- ### Identity -->
<!-- ### Marketplace -->
::: moniker-end

::: moniker range=">=azs-1910"
## <a name="archive"></a>アーカイブ

アーカイブされた、以前のバージョンの既知の問題にアクセスするには、左側の目次の上部にあるバージョン セレクターのドロップダウンを使用して、表示するバージョンを選択します。

## <a name="next-steps"></a>次のステップ

- [更新アクティビティのチェック リストを確認する](release-notes-checklist.md)
- [セキュリティ更新プログラムの一覧を確認する](release-notes-security-updates.md)
::: moniker-end

<!------------------------------------------------------------>
<!------------------- UNSUPPORTED VERSIONS ------------------->
<!------------------------------------------------------------>
::: moniker range="azs-1908"
## <a name="1908-archived-known-issues"></a>1908 アーカイブされた既知の問題
::: moniker-end
::: moniker range="azs-1907"
## <a name="1907-archived-known-issues"></a>1907 アーカイブされた既知の問題
::: moniker-end
::: moniker range="azs-1906"
## <a name="1906-archived-known-issues"></a>1906 アーカイブされた既知の問題
::: moniker-end
::: moniker range="azs-1905"
## <a name="1905-archived-known-issues"></a>1905 アーカイブされた既知の問題
::: moniker-end
::: moniker range="azs-1904"
## <a name="1904-archived-known-issues"></a>1904 アーカイブされた既知の問題
::: moniker-end
::: moniker range="azs-1903"
## <a name="1903-archived-known-issues"></a>1903 アーカイブされた既知の問題
::: moniker-end
::: moniker range="azs-1902"
## <a name="1902-archived-known-issues"></a>1902 アーカイブされた既知の問題
::: moniker-end
::: moniker range="azs-1901"
## <a name="1901-archived-known-issues"></a>1901 アーカイブされた既知の問題
::: moniker-end
::: moniker range="azs-1811"
## <a name="1811-archived-known-issues"></a>1811 アーカイブされた既知の問題
::: moniker-end
::: moniker range="azs-1809"
## <a name="1809-archived-known-issues"></a>1809 アーカイブされた既知の問題
::: moniker-end
::: moniker range="azs-1808"
## <a name="1808-archived-known-issues"></a>1808 アーカイブされた既知の問題
::: moniker-end
::: moniker range="azs-1807"
## <a name="1807-archived-known-issues"></a>1807 アーカイブされた既知の問題
::: moniker-end
::: moniker range="azs-1805"
## <a name="1805-archived-known-issues"></a>1805 アーカイブされた既知の問題
::: moniker-end
::: moniker range="azs-1804"
## <a name="1804-archived-known-issues"></a>1804 アーカイブされた既知の問題
::: moniker-end
::: moniker range="azs-1803"
## <a name="1803-archived-known-issues"></a>1803 アーカイブされた既知の問題
::: moniker-end
::: moniker range="azs-1802"
## <a name="1802-archived-known-issues"></a>1802 アーカイブされた既知の問題
::: moniker-end

::: moniker range="<azs-1910"
[以前のバージョンの Azure Stack Hub の既知の問題は TechNet ギャラリー](https://aka.ms/azsarchivedrelnotes)でアクセスできます。 これらのアーカイブされたドキュメントは、参照のみを目的に提供されており、これらのバージョンのサポートを意味しているわけではありません。 Azure Stack のサポートについては、「[Azure Stack Hub サービス ポリシー](azure-stack-servicing-policy.md)」を参照してください。 さらにサポートが必要な場合は、Microsoft カスタマー サポート サービスにお問い合わせください。
::: moniker-end
