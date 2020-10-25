---
title: Azure Stack HCI での Azure Kubernetes Service の要件
description: Azure Stack HCI で Azure Kubernetes Service を開始する前に
ms.topic: conceptual
author: abhilashaagarwala
ms.author: abha
ms.date: 09/22/2020
ms.openlocfilehash: 4c91dad12e4d6680ca0887da2e6ca314cdbd64ee
ms.sourcegitcommit: 3e225b30a54159b6b8dbeb2f843a2e5a721b746e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/29/2020
ms.locfileid: "91519388"
---
# <a name="system-requirements-for-azure-kubernetes-service-on-azure-stack-hci"></a>Azure Stack HCI での Azure Kubernetes Service のシステム要件

> 適用対象:Azure Stack HCI

この記事では Azure Stack HCI 上に Azure Kubernetes Service を設定し、それを使用して Kubernetes クラスターを作成するための要件について説明します。 Azure Stack HCI での Azure Kubernetes Service の概要については、[Azure Stack HCI での AKS の概要](overview.md)に関するページを参照してください。

## <a name="determine-hardware-requirements"></a>ハードウェア要件を判断する

Microsoft のパートナーから Azure Stack HCI ハードウェア/ソフトウェアの検証済みソリューションを購入することをお勧めします。 これらのソリューションは、互換性と信頼性を確保するために、Microsoft の参照アーキテクチャに照らして設計、組み立て、検証されているため、迅速に稼働させることができます。 お使いのシステム、コンポーネント、デバイス、ドライバーが、Windows Server カタログで認定されている Windows Server 2019 であることを確認します。 検証済みソリューションについては、[Azure Stack HCI ソリューション](https://azure.microsoft.com/overview/azure-stack/hci)の Web サイトを参照してください。

### <a name="general-requirements"></a>一般的な要件

Active Directory 環境で、Azure Stack HCI 上の Azure Kubernetes Service を最適に機能させるには、以下の要件が満たされるようにします。 

 - 時刻の同期が設定されていて、すべてのクラスター ノードとドメイン コントローラーにわたって相違が 2 分を超えていないことを確認します。 時刻の同期の設定については、[Windows タイム サービス](/windows-server/networking/windows-time-service/windows-time-service-top)に関するページを参照してください。 

 - Azure Stack HCI クラスターで Azure Kubernetes Service を追加、更新、管理するユーザー アカウントが、Active Directory で適切なアクセス許可を持っていることを確認します。 組織単位 (OU) を使用してサーバーとサービスのグループ ポリシーを管理しようとしている場合は、ユーザー アカウントに、OU 内のすべてのオブジェクトに対する一覧表示、読み取り、変更、削除のアクセス許可が必要です。 

 - Azure Stack HCI クラスター上の Azure Kubernetes Service を追加するサーバーとサービスに対しては、別個の OU を使用することが推奨されます。 これで、アクセスとアクセス許可をよりきめ細かに制御できるようになります。

 - Active Directory のコンテナーに対して GPO テンプレートを使用しようとしている場合は、そのポリシーから AKS HCI のデプロイが除外されていることを確認します。 サーバーのセキュリティ強化は、後続のプレビュー リリースで使用できるようになります。

### <a name="compute-requirements"></a>コンピューティングの要件

 - クラスター内に最大 4 台のサーバーがある Azure Stack HCI クラスター。 クラスター内の各サーバーには、少なくとも 24 個の CPU コアと少なくとも 512 GB の RAM を搭載することが推奨されます。

 - Azure Kubernetes Service は、技術的には単一ノードの Azure Stack HCI Server で実行可能ですが、そうすることは推奨されません。

 - Azure Stack HCI 上の Azure Kubernetes Service に対するその他のコンピューティング要件は、Azure Stack HCI の要件と一致しています。 Azure Stack HCI のサーバー要件の詳細については、[Azure Stack HCI の要件](../hci/deploy/before-you-start.md)に関するページを参照してください。  

 - このプレビュー リリースでは、EN-US リージョンと言語の選択を使用して、クラスター内の各サーバーに Azure Stack HCI オペレーティング システムをインストールする必要があります。現時点では、インストール後に変更しても十分ではありません。

### <a name="network-requirements"></a>ネットワークの要件 

Azure Stack HCI 上の Azure Kubernetes Service では、各サーバー ノード間に、信頼性が高く高帯域幅で、待ち時間が短いネットワーク接続が必要になります。 以下の項目について確認します。 

 - Windows Admin Center を使用している場合は、既存の外部仮想スイッチを構成済みであることを確認します。 Azure Stack HCI クラスターの場合、このスイッチはすべてのクラスター ノードにわたって同じである必要があります。 

 - すべてのネットワーク アダプターで IPv6 が無効になっていることを確認します。 

 - VM と VM ホストに TCP/IP アドレスを提供するため、ネットワークに使用可能な DHCP サーバーがある必要があります。 DHCP サーバーには、NTP と DNS のホスト情報も含まれている必要があります。 

 - また、Azure Stack HCI クラスターからアクセスできる専用の IPv4 アドレス スコープを持つ DHCP サーバーを用意することが推奨されます。 たとえば、既定のゲートウェイには 10.0.1.1、Kubernetes のサービスには 10.0.1.2 ～ 10.0.1.102 を予約し、Kubernetes クラスター VM には 10.0.1.103 ～ 10.0.1.254 を使用することができます。 

 - DHCP サーバーによって提供される IPv4 アドレスは、ルーティング可能で、VM の更新や再プロビジョニングの場合に IP 接続が失われないように 7 日間のリースの有効期限が設定されている必要があります。  

 - VLAN タグを付けることは推奨されません。 Azure Stack HCI クラスター ネットワーク スイッチで、アクセス ポートまたはタグなしのポートを使用します。 

 - セットアップ中に、ロード バランサーの仮想 IP プールのために専用の静的仮想 IP プールを使用することは推奨されません。 DHCP IP プールは仮想マシンに使用されますが、仮想 IP プールはロード バランサーに使用されるため、ルーティング可能である必要があります。 DHCP IP プールは、外部インターネットにルーティング可能である必要はありません。

 - すべてのノードが相互に通信できるようにするには、DNS 名前解決が必要です。 Kubernetes の外部名前解決には、IP アドレスが取得されたときに DHCP サーバーによって提供される DNS サーバーが使用されます。 Kubernetes の内部名前解決には、Kubernetes の既定の、DNS ベースのコア ソリューションが使用されます。 

 - このプレビュー リリースでは、Windows Admin Center、Azure Stack HCI クラスター ノード、および Azure Stack HCI クラスター ノード上の Azure Kubernetes Service を、プロキシ サーバーを使用してインターネットに接続することはサポートされていません。

### <a name="network-port-and-url-requirements"></a>ネットワーク ポートと URL の要件 

Azure Stack HCI 上に Azure Kubernetes クラスターを作成するときには、クラスター内の各サーバーで、以下のファイアウォール ポートが自動的に開かれます。 


| ファイアウォール ポート               | 説明         | 
| ---------------------------- | ------------ | 
| 45000           | wssdagent GPRC サーバー ポート           |
| 45001             | wssdagent GPRC 認証ポート  | 
| 55000           | wssdcloudagent GPRC サーバー ポート           |
| 55001             | wssdcloudagent GPRC 認証ポート  | 


Windows Admin Center マシンと Azure Stack HCI クラスター内のすべてのノードについて、ファイアウォールの URL の例外が必要です。 

| URL        | Port | サービス | メモ |
| ---------- | ---- | --- | ---- |
https://helm.sh/blog/get-helm-sh/  | 443 | ダウンロード エージェント、WAC | Helm バイナリのダウンロードに使用 
https://storage.googleapis.com/  | 443 | クラウドの初期化 | Kubernetes バイナリのダウンロード 
https://azurecliprod.blob.core.windows.net/ | 443 | クラウドの初期化 | バイナリとコンテナーのダウンロード 
https://aka.ms/installazurecliwindows | 443 | WAC | Azure CLI のダウンロード 
*.api.cdp.microsoft.com、*.dl.delivery.mp.microsoft.com、*.emdl.ws.microsoft.com | 80、443 | エージェントのダウンロード | メタデータのダウンロード 
*.dl.delivery.mp.microsoft.com、*.do.dsp.mp.microsoft.com. | 80、443 | エージェントのダウンロード | VHD イメージのダウンロード 
ecpacr.azurecr.io | 443 | Kubernetes | コンテナー イメージのダウンロード 

### <a name="storage-requirements"></a>ストレージの要件 

Azure Stack HCI 上の Azure Kubernetes Service では、以下のストレージの実装がサポートされています。 

|  名前                         | ストレージ型 | 必要な容量 |
| ---------------------------- | ------------ | ----------------- |
| Azure Stack HCI クラスター          | CSV          | 1 TB (テラバイト)              |
| 単一ノードの Azure Stack HCI | 直接接続ストレージ | 500 GB|

### <a name="review-maximum-supported-hardware-specifications"></a>サポートされているハードウェア最大仕様を確認する 

以下の仕様を超える Azure Stack HCI 上の Azure Kubernetes Service のデプロイはサポートされていません。 

| リソース                     | 最大値 |
| ---------------------------- | --------|
| クラスターあたりの物理サーバー数 | 4       |
| Kubernetes クラスター            | 4       |
| バーチャル マシンの合計数          | 200     |

### <a name="windows-admin-center"></a>Windows Admin Center 

Windows Admin Center は、Azure Stack HCI 上の Azure Kubernetes Service を作成および管理するためのユーザー インターフェイスです。 Azure Stack HCI 上の Azure Kubernetes Service を扱う Windows Admin Center を使用するには、下記の一覧のすべての条件を満たす必要があります。 

#### <a name="on-your-windows-admin-center-system"></a>Windows Admin Center システム上

Windows Admin Center ゲートウェイを実行するマシンは以下の条件を満たす必要があります。 

 - Windows 10 (現在、Windows Admin Center サーバーはサポートされていません)
 - 60 GB の空き領域
 - Azure に登録済み
 - Azure Stack HCI クラスターと同じドメイン内

## <a name="next-steps"></a>次の手順 

上記の前提条件すべてが満たされたら、以下を使用して Azure Stack HCI 上に Azure Kubernetes Service ホストを設定できます。
 - [Windows Admin Center](setup.md)
 - [PowerShell](setup-powershell.md)
