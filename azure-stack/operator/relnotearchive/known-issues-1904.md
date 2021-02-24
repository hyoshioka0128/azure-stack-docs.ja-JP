---
title: Azure Stack 1904 の既知の問題 | Microsoft Docs
description: Azure Stack 1904 の既知の問題について説明します。
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
ms.date: 05/31/2019
ms.author: sethm
ms.reviewer: hectorl
ms.lastreviewed: 05/31/2019
ROBOTS: NOINDEX
ms.openlocfilehash: f977c1671468257ac205c494f33ad46965f1f2eb
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99248809"
---
# <a name="azure-stack-1904-known-issues"></a>Azure Stack 1904 の既知の問題

この記事では、Azure Stack 1904 リリースの既知の問題について説明します。 新しい問題が特定されると、この一覧は更新されます。

> [!IMPORTANT]  
> 更新プログラムを適用する前に、このセクションを確認してください。

## <a name="update-process"></a>更新処理

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因:Azure Stack の更新プログラムをインストールしようとしたときに、更新の状況が失敗して、状態が **PreparationFailed** に変更される場合があります。 これは、更新リソース プロバイダー (URP) が、処理のためにストレージ コンテナーから内部インフラストラクチャ共有にファイルを正しく転送できないことが原因です。
- 修復: バージョン 1901 (1.1901.0.95) 以降、この問題は、 **[今すぐ更新]** ( **[再開]** ではない) をもう一度クリックすることで回避できるようになりました。 それにより、URP は前回の試行のファイルをクリーンアップして、もう一度ダウンロードを開始します。
- 発生頻度: 共通

## <a name="portal"></a>ポータル

### <a name="administrative-subscriptions"></a>管理サブスクリプション

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: バージョン 1804 で導入された 2 つの管理サブスクリプションは使用しないでください。 このサブスクリプションの種類は **Metering** サブスクリプションと **Consumption** サブスクリプションです。
- 修復: これら 2 つのサブスクリプション上でリソースが実行されている場合は、ユーザー サブスクリプションで再作成してください。
- 発生頻度: 共通

### <a name="subscription-resources"></a>サブスクリプション リソース

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因:ユーザー サブスクリプションを削除すると、リソースが孤立します。
- 修復: まず、ユーザー リソースまたはリソース グループ全体を削除してから、ユーザー サブスクリプションを削除します。
- 発生頻度: 共通

### <a name="subscription-permissions"></a>サブスクリプションのアクセス許可

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因:Azure Stack ポータルを使用して、サブスクリプションへのアクセス許可を表示することはできません。
- 修復: [PowerShell を使用してアクセス許可を確認](/powershell/module/azurerm.resources/get-azurermroleassignment)します。
- 発生頻度: 共通

### <a name="docker-extension"></a>Docker 拡張機能

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因:管理者ポータルとユーザー ポータルの両方で、**Docker** を検索すると、返される項目が正しくありません。 これは Azure Stack でご利用になれません。 作成しようとすると、エラーが表示されます。
- 修復: 軽減策はありません。
- 発生頻度: 共通

### <a name="marketplace-management"></a>Marketplace Management

- 適用先:これはリリース 1904 に関する新しい問題です。
- 原因:管理者ポータルにサインインしたときに、Marketplace Management 画面が表示されません。
- 修復: ブラウザーを更新します。
- 発生頻度: 間欠的

### <a name="marketplace-management"></a>Marketplace Management

- 適用先:この問題は、1904 に適用されます。
- 原因:管理者ポータル内の Marketplace management タブの **[Azure から追加]** ブレードで結果をフィルター処理したときに、間違ったフィルター処理結果が表示されることがあります。
- 修復: [名前] 列で結果の並べ替えを行うと、結果が修正されます。
- 発生頻度: 間欠的

### <a name="marketplace-management"></a>Marketplace Management

- 適用先:この問題は、1904 に適用されます。
- 原因:管理者ポータルの Marketplace Management で結果をフィルター処理すると、パブリッシャー ドロップダウンに、重複するパブリッシャー名が表示されます。 
- 修復: そのパブリッシャーで利用可能なすべてのマーケットプレース製品の正しい一覧を得るには、すべての重複を選択します。
- 発生頻度: 間欠的

### <a name="docker-extension"></a>Docker 拡張機能

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因:管理者ポータルとユーザー ポータルの両方で、**Docker** を検索すると、返される項目が正しくありません。 これは Azure Stack でご利用になれません。 作成しようとすると、エラーが表示されます。
- 修復: 軽減策はありません。
- 発生頻度: 共通

### <a name="upload-blob"></a>BLOB のアップロード

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: ユーザー ポータルで [OAuth(preview)]\(OAuth (プレビュー)\) オプションを使用して BLOB をアップロードしようとすると、タスクがエラー メッセージにより失敗します。
- 修復: SAS オプションを使用して BLOB をアップロードします。
- 発生頻度: 共通

### <a name="template"></a>テンプレート

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因:ユーザー ポータルで、テンプレート デプロイ UI によって "_" (アンダースコア文字) で始まるテンプレート名のパラメーターの設定が行われません。
- 修復: テンプレート名から "_" (アンダースコア文字) を削除します。
- 発生頻度: 共通

## <a name="networking"></a>ネットワーク

### <a name="load-balancer"></a>Load Balancer

#### <a name="add-backend-pool"></a>バックエンド プールを追加する

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因:ユーザー ポータルで **[バックエンド プール]** を **[Load Balancer]** に追加しようとすると、"**Failed to update Load Balancer**" (Load Balancer の更新に失敗しました) というエラー メッセージで操作が失敗します。
- 修復: PowerShell、CLI、または Azure Resource Manager テンプレートを使用して、バックエンド プールをロード バランサー リソースに関連付けます。
- 発生頻度: 共通

#### <a name="create-inbound-nat"></a>インバウンド NAT を作成する

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因:ユーザー ポータルで **[Load Balancer]** に対して **インバウンド NAT 規則** を作成しようとすると、"**Failed to update Load Balancer**" (Load Balancer の更新に失敗しました) というエラー メッセージで操作が失敗します。
- 修復: PowerShell、CLI、または Azure Resource Manager テンプレートを使用して、バックエンド プールをロード バランサー リソースに関連付けます。
- 発生頻度: 共通

#### <a name="create-load-balancer"></a>ロード バランサーの作成

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因:ユーザー ポータルで、 **[ロード バランサーの作成]** ウィンドウに **Standard** Load Balancer SKU を作成するためのオプションが表示されます。 このオプションは Azure Stack ではサポートされていません。
- 修復: 代わりに Basic ロード バランサー オプションを使用します。
- 発生頻度: 共通

#### <a name="public-ip-address"></a>パブリック IP アドレス

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因:ユーザー ポータルで、 **[パブリック IP アドレスの作成]** ウィンドウに **Standard** SKU を作成するためのオプションが表示されます。 **Standard** SKU は Azure Stack ではサポートされていません。
- 修復: パブリック IP アドレスではなく **Basic** SKU を使用します。
- 発生頻度: 共通

## <a name="compute"></a>Compute

### <a name="vm-boot-diagnostics"></a>VM ブート診断

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因:新しい Windows 仮想マシン (VM) を作成するときに、次のエラーが表示されることがあります。**仮想マシン 'vm-name' を起動できませんでした。エラー:Failed to update serial output settings for VM 'vm-name'** . (VM 'vm-name' のシリアル出力設定を更新できませんでした。)
このエラーは、VM でブート診断を有効にしても、ブート診断ストレージ アカウントを削除した場合に発生します。
- 修復: 以前使用したものと同じ名前のストレージ アカウントを再作成します。
- 発生頻度: 共通

### <a name="virtual-machine-scale-set"></a>仮想マシン スケール セット

#### <a name="centos"></a>CentOS

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因:仮想マシン スケール セットの作成エクスペリエンスでは、デプロイのオプションとして CentOS-based 7.2 が提供されます。 CentOS 7.2 は Azure Stack Marketplace では利用できず、イメージが見つからないという理由でデプロイ エラーが発生します。
- 修復: デプロイ用に別のオペレーティング システムを選択するか、またはデプロイする前にオペレーターが Marketplace からダウンロードしておいた別の CentOS イメージを指定する Azure Resource Manager テンプレートを使用します。
- 発生頻度: 共通

#### <a name="remove-scale-set"></a>スケール セットを削除する

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因:スケール セットを **[Virtual Machine Scale Sets]** ブレードから削除することはできません。
- 修復: 削除するスケール セットを選択し、 **[概要]** ウィンドウから **[削除]** ボタンをクリックします。
- 発生頻度: 共通

#### <a name="create-failures-during-patch-and-update-on-4-node-azure-stack-environments"></a>4 ノードの Azure Stack 環境でのパッチと更新プログラムの適用中に作成が失敗します

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因:4 ノードの Azure Stack 環境では、障害ドメインが 3 つの可用性セット内での VM の作成および仮想マシン スケール セット インスタンスの作成が更新プロセス中に **FabricVmPlacementErrorUnsupportedFaultDomainSize** エラーで失敗します。
- 修復: 障害ドメインが 2 つの可用性セット内には 1 つの VM を正常に作成できます。 ただし、4 ノードの Azure Stack では、依然として更新プロセス中にスケール セット インスタンスを作成することはできません。

### <a name="ubuntu-ssh-access"></a>Ubuntu SSH アクセス

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因: SSH の承認を有効にして作成した Ubuntu 18.04 VM では、SSH キーを使用してサインインすることはできません。
- 修復: プロビジョニング後に Linux 拡張機能用の VM アクセスを使用して SSH キーを実装するか、パスワードベースの認証を使用します。
- 発生頻度: 共通

### <a name="compute-host-agent-alert"></a>コンピューティング ホスト エージェントのアラート

- 適用先:これはリリース 1904 に関する新しい問題です。
- 原因:**コンピューティング ホスト エージェント** の警告は、スケール ユニット内のノードの再起動後に表示されます。 再起動により、コンピューティング ホスト エージェント サービスの既定のスタートアップ設定が変更されます。 このアラートは次の例のようになります。

   ```shell
   NAME  
   Compute Host Agent is not responding to calls.
   SEVERITY  
   Warning
   STATE  
   Active
   CREATED TIME  
   5/16/2019, 10:08:23 AM
   UPDATED TIME  
   5/22/2019, 12:27:27 PM
   COMPONENT  
   M#####-NODE02
   DESCRIPTION  
   Could not communicate with the Compute Host Agent running on node: M#####-NODE02
   REMEDIATION  
   Please disable Compute Host Agent feature flag and collect logs for further diagnosis.
   ```

- 修復:
  - このアラートは無視してかまいません。 応答していないエージェントは、オペレーターとユーザーの操作やユーザーのアプリケーションに何の影響もありません。 アラートは、手動で閉じられた場合、24 時間後に再び表示されます。
  - この問題は、[1904 用の最新の Azure Stack 修正プログラム](https://support.microsoft.com/help/4505688)で修正されています。
- 発生頻度: 共通

### <a name="virtual-machine-scale-set-instance-view"></a>仮想マシン スケール セットのインスタンス ビュー

- 適用先:この問題は、1904 および 1905 リリースに適用されます。
- 原因:Azure Stack ポータル ( **[ダッシュボード]**  >  **[仮想マシン スケール セット]**  >  **[AnyScaleSet - インスタンス]**  >  **[AnyScaleSetInstance]** ) にあるスケール セットのインスタンス ビュー ブレードが読み込みに失敗します。
- 修復: 現在、修復方法はなく、修正に取り組んでいます。 それまでは、仮想マシン スケール セットのインスタンス ビューの取得には、CLI コマンドレット `az vmss get-instance-view` を使用してください。

### <a name="user-image-service"></a>ユーザー イメージ サービス
- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因:ユーザー イメージの作成に失敗すると、ユーザー イメージ サービスが正しくない状態になります。 ユーザー イメージの作成と削除の操作が失敗し始めます。 ユーザー イメージの削除は、次のエラーで失敗することがあります。"エラー: 内部ディスク管理エラーが発生しました。"
- 修復: 軽減策はありません。 Microsoft にサポート チケットを申請します。

## <a name="storage"></a>記憶域

- 適用先:この問題は、サポートされているすべてのリリースに適用されます。
- 原因:[ConvertTo-AzureRmVMManagedDisk](/powershell/module/azurerm.compute/convertto-azurermvmmanageddisk) は Azure Stack ではサポートされていないため、ID が **$null** のディスクが作成されます。 これにより、起動や停止など、VM に対する操作を実行できなくなります。 ディスクは UI に表示されず、API を使用しても表示されません。 その時点では VM を修復できないため、削除する必要があります。
- 修復: ディスクを正しく変換するには、[マネージド ディスクへの変換ガイド](../../user/azure-stack-managed-disk-considerations.md#convert-to-managed-disks)の手順に従ってください。

## <a name="app-service"></a>App Service

- テナントは、サブスクリプションで最初の Azure 関数を作成する前に、ストレージ リソースプロバイダーを登録する必要があります。
- 1903 のポータル フレームワークと互換性がないため、テナント ポータルの一部のユーザー エクスペリエンス (主に、デプロイ スロット、運用環境でのテスト、およびサイト拡張機能のユーザー エクスペリエンス) は壊れています。 この問題を回避するには、[Azure App Service PowerShell モジュール](/azure/app-service/deploy-staging-slots#automate-with-powershell)または [Azure CLI](/cli/azure/webapp/deployment/slot?view=azure-cli-latest&preserve-view=true) を使用します。 ポータル エクスペリエンスは、[Azure App Service on Azure Stack のデプロイを 1.6 (Update 6)](../azure-stack-app-service-release-notes-update-six.md) にアップグレードすることで復元されます。

## <a name="next-steps"></a>次のステップ

- [更新アクティビティのチェック リストを確認する](../release-notes-checklist.md)
- [セキュリティ更新プログラムの一覧を確認する](../release-notes-security-updates.md)
