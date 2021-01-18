---
title: Azure Stack HCI での Azure Kubernetes Service の要件
description: Azure Stack HCI で Azure Kubernetes Service を開始する前に
ms.topic: conceptual
author: abhilashaagarwala
ms.author: abha
ms.date: 12/02/2020
ms.openlocfilehash: 71c842cf44963988da7926003646b246bf80f802
ms.sourcegitcommit: 8776cbe4edca5b63537eb10bcd83be4b984c374a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/13/2021
ms.locfileid: "98175738"
---
# <a name="system-requirements-for-azure-kubernetes-service-on-azure-stack-hci"></a>Azure Stack HCI での Azure Kubernetes Service のシステム要件

> 適用対象:Azure Stack HCI、Windows Server 2019 Datacenter

この記事では、Azure Stack HCI または Windows Server 2019 Datacenter 上に Azure Kubernetes Service を設定し、それを使用して Kubernetes クラスターを作成するための要件について説明します。 Azure Stack HCI での Azure Kubernetes Service の概要については、[Azure Stack HCI での AKS の概要](overview.md)に関するページを参照してください。

## <a name="determine-hardware-requirements"></a>ハードウェア要件を判断する

Microsoft のパートナーから Azure Stack HCI ハードウェア/ソフトウェアの検証済みソリューションを購入することをお勧めします。 これらのソリューションは、互換性と信頼性を確保するために、Microsoft の参照アーキテクチャに照らして設計、組み立て、検証されているため、迅速に稼働させることができます。 お使いのシステム、コンポーネント、デバイス、ドライバーが、Windows Server カタログで認定されている Windows Server 2019 であることを確認します。 検証済みソリューションについては、[Azure Stack HCI ソリューション](https://azure.microsoft.com/overview/azure-stack/hci)の Web サイトを参照してください。

## <a name="general-requirements"></a>一般的な要件

Active Directory 環境で、Azure Stack HCI または Windows Server 2019 Datacenter 上の Azure Kubernetes Service を最適に機能させるには、以下の要件が満たされていることを確認します。 

 - 時刻の同期が設定されていて、すべてのクラスター ノードとドメイン コントローラーにわたって相違が 2 分を超えていないことを確認します。 時刻の同期の設定については、「[Windows タイム サービス](/windows-server/networking/windows-time-service/windows-time-service-top)」を参照してください。 

 - Azure Stack HCI または Windows Server 2019 Datacenter クラスターで Azure Kubernetes Service を追加、更新、管理するユーザー アカウントが、Active Directory で適切なアクセス許可を持っていることを確認します。 組織単位 (OU) を使用してサーバーとサービスのグループ ポリシーを管理しようとしている場合は、ユーザー アカウントに、OU 内のすべてのオブジェクトに対する一覧表示、読み取り、変更、削除のアクセス許可が必要です。 

 - Azure Stack HCI または Windows Server 2019 Datacenter クラスター上に Azure Kubernetes Service を追加するサーバーとサービスに対しては、別個の OU を使用することが推奨されます。 別個の OU を使用することで、アクセスとアクセス許可をよりきめ細かに制御できるようになります。

 - Active Directory のコンテナーに対して GPO テンプレートを使用しようとしている場合は、ポリシーから AKS HCI のデプロイが除外されていることを確認します。 サーバーのセキュリティ強化は、後続のプレビュー リリースで使用できるようになります。

## <a name="compute-requirements"></a>コンピューティングの要件

 - クラスター内に最大 4 台のサーバーがある Azure Stack HCI クラスターまたは Windows Server 2019 Datacenter フェールオーバー クラスター。 クラスター内の各サーバーには、少なくとも 24 個の CPU コアと少なくとも 256 GB の RAM を搭載することが推奨されます。

 - Azure Kubernetes Service は、技術的には単一ノードの Windows Server 2019 Datacenter で実行できますが、そうすることは推奨されません。 ただし、評価目的のために、単一ノードの Windows Server 2019 Datacenter 上で Azure Kubernetes Service を実行できます。

 - Azure Stack HCI 上の Azure Kubernetes Service に対するその他のコンピューティング要件は、Azure Stack HCI の要件と一致しています。 Azure Stack HCI のサーバー要件の詳細については、[Azure Stack HCI システム要件](../hci/concepts/system-requirements.md#server-requirements)に関するページを参照してください。

 - このプレビュー リリースでは、EN-US リージョンと言語の選択を使用して、クラスター内の各サーバーに Azure Stack HCI オペレーティング システムをインストールする必要があります。現時点では、インストール後に変更しても十分ではありません。

## <a name="network-requirements"></a>ネットワークの要件 

Azure Stack HCI クラスターと Windows Server 2019 Datacenter クラスターには、次の要件が適用されます。 

 - Windows Admin Center を使用している場合は、既存の外部仮想スイッチを構成済みであることを確認します。 Azure Stack HCI クラスターの場合、このスイッチとその名前は、すべてのクラスター ノードにわたって同じである必要があります。 

 - すべてのネットワーク アダプターで IPv6 が無効になっていることを確認します。 

 - デプロイを成功させるには、Azure Stack HCI クラスター ノードと Kubernetes クラスター VM に外部インターネット接続が必要です。
  
 - Azure Stack HCI ホストとテナント VM の間にネットワーク接続が存在することを確認します。

 - すべてのノードが相互に通信できるようにするには、DNS 名前解決が必要です。 Kubernetes の外部名前解決には、IP アドレスが取得されたときに DHCP サーバーによって指定される DNS サーバーを使用します。 Kubernetes の内部名前解決には、Kubernetes の既定の、DNS ベースのコア ソリューションが使用されます。 

 - このプレビュー リリースでは、デプロイ全体に対して 1 つの VLAN サポートのみが提供されています。 

 - このプレビュー リリースでは、PowerShell を使用して作成される Kubernetes クラスターに対するプロキシ サポートは限定的です。 
 
### <a name="ip-address-assignment"></a>IP アドレスの割り当て  
 
Azure Stack HCI での AKS のデプロイを成功させるため、DHCP サーバーで仮想 IP プールの範囲を構成することをお勧めします。 すべてのワークロード クラスターに対して、3 から 5 個の高可用性コントロール プレーン ノードを構成することもお勧めします。 

> [!NOTE]
> 静的 IP アドレスの割り当てのみを使用することはサポートされていません。 このプレビュー リリースの一環として、DHCP サーバーを構成する必要があります。

#### <a name="dhcp"></a>[DHCP]
DHCP を使用してクラスター全体に IP アドレスを割り当てるときは、次の要件に従ってください。  

 - VM と VM ホストに TCP/IP アドレスを提供するため、ネットワークに使用可能な DHCP サーバーがある必要があります。 DHCP サーバーには、ネットワーク タイム プロトコル (NTP) と DNS ホストの情報も含まれている必要があります。
 
 - Azure Stack HCI クラスターからアクセスできる専用の IPv4 アドレス スコープを持つ DHCP サーバー。
 
 - DHCP サーバーによって提供される IPv4 アドレスはルーティング可能であり、VM の更新や再プロビジョニングの場合に IP 接続が失われないようにするために 30 日間のリースの有効期限が設定されている必要があります。  

少なくとも、次の数の DHCP アドレスを予約する必要があります。

| クラスターの種類  | コントロール プレーン ノード | ワーカー ノード | 更新 | Load Balancer  |
| ------------- | ------------------ | ---------- | ----------| -------------|
| AKS ホスト |  1  |  0  |  2  |  0  |
| ワークロード クラスター  |  ノードあたり 1  | ノードあたり 1 |  5  |  1  |

ご覧のように、ご使用の環境におけるワークロード クラスター、コントロール プレーン、およびワーカー ノードの数に応じて、必要な IP アドレスの数は異なります。 DHCP IP プールでは、256 の IP アドレス (/24 サブネット) を予約することをお勧めします。
  
    
#### <a name="vip-pool-range"></a>VIP プールの範囲

仮想 IP (VIP) プールは Azure Stack HCI デプロイの AKS に強くお勧めします。 VIP プールとは、デプロイおよびアプリケーションのワークロードに常に到達可能であることを保証するために、長期間のデプロイに使用される予約済みの静的 IP アドレスの範囲です。 現時点では、IPv4 アドレスしかサポートされていないため、すべてのネットワーク アダプターで IPv6 が無効となっていることを確認する必要があります。 また、仮想 IP アドレスが DHCP IP 予約に含まれていないことを確認します。

少なくとも、クラスター (ワークロードと AKS ホスト) ごとに 1 つの IP アドレスを予約し、Kubernetes サービスごとに 1 つの IP アドレスを予約する必要があります。 ご使用の環境におけるワークロード クラスターと Kubernetes サービスの数に応じて、VIP プールの範囲内の必要な IP アドレスの数は異なります。 AKS HCI のデプロイには、16 個の静的 IP アドレスを予約することをお勧めします。 

AKS ホストを設定するとき、`Set-AksHciConfig` の `-vipPoolStartIp` パラメーターと `-vipPoolEndIp` パラメーターを使用して、VIP プールを作成します。

#### <a name="mac-pool-range"></a>MAC プールの範囲
各クラスターで複数のコントロール プレーン ノードを使用できるようにするには、範囲内で 16 個以上の MAC アドレスを設定することをお勧めします。 AKS ホストを設定するとき、`Set-AksHciConfig` の `-macPoolStart` および `-macPoolEnd` パラメーターを使用して、Kubernetes サービスに DHCP MAC プールの MAC アドレスを予約します。
  
### <a name="network-port-and-url-requirements"></a>ネットワーク ポートと URL の要件 

Azure Stack HCI 上に Azure Kubernetes クラスターを作成すると、クラスター内の各サーバーで、以下のファイアウォール ポートが自動的に開かれます。 


| ファイアウォール ポート               | 説明     | 
| ---------------------------- | ------------ | 
| 45000           | wssdagent GPRC サーバー ポート     |
| 45001             | wssdagent GPRC 認証ポート  | 
| 55000           | wssdcloudagent GPRC サーバー ポート      |
| 65000            | wssdcloudagent GPRC 認証ポート  | 


Windows Admin Center マシンと Azure Stack HCI クラスター内のすべてのノードについて、ファイアウォールの URL の例外が必要です。 

| URL        | Port | サービス | Notes |
| ---------- | ---- | --- | ---- |
https://helm.sh/blog/get-helm-sh/  | 443 | ダウンロード エージェント、WAC | Helm バイナリのダウンロードに使用 
https://storage.googleapis.com/  | 443 | クラウドの初期化 | Kubernetes バイナリのダウンロード 
https://azurecliprod.blob.core.windows.net/ | 443 | クラウドの初期化 | バイナリとコンテナーのダウンロード 
https://aka.ms/installazurecliwindows | 443 | WAC | Azure CLI のダウンロード 
https://:443 | 443 | TCP | Azure Arc エージェントをサポートするために使用されます  
*.blob.core.windows.net | 443 | TCP | ダウンロードに必要です
*.api.cdp.microsoft.com、*.dl.delivery.mp.microsoft.com、*.emdl.ws.microsoft.com | 80、443 | エージェントのダウンロード | メタデータのダウンロード 
*.dl.delivery.mp.microsoft.com、*.do.dsp.mp.microsoft.com. | 80、443 | エージェントのダウンロード | VHD イメージのダウンロード 
ecpacr.azurecr.io | 443 | Kubernetes | コンテナー イメージのダウンロード 
git://:9418 | 9418 | TCP | Azure Arc エージェントをサポートするために使用されます 

## <a name="storage-requirements"></a>ストレージの要件 

Azure Stack HCI 上の Azure Kubernetes Service では、以下のストレージの実装がサポートされています。 

|  名前                         | ストレージの種類 | 必要な容量 |
| ---------------------------- | ------------ | ----------------- |
| Azure Stack HCI クラスター          | CSV          | 1 TB (テラバイト)              |
| Windows Server 2019 Datacenter フェールオーバー クラスター          | CSV          | 1 TB (テラバイト)              |
| 単一ノードの Windows Server 2019 Datacenter | 直接接続ストレージ | 500 GB|

## <a name="review-maximum-supported-hardware-specifications"></a>サポートされているハードウェア最大仕様を確認する 

以下の仕様を超える Azure Stack HCI 上の Azure Kubernetes Service のデプロイはサポートされていません。 

| リソース                     | 最大値 |
| ---------------------------- | --------|
| クラスターあたりの物理サーバー数 | 4       |
| Kubernetes クラスター            | 4       |
| バーチャル マシンの合計数          | 200     |

## <a name="windows-admin-center-requirements"></a>Windows Admin Center の要件

Windows Admin Center は、Azure Stack HCI 上の Azure Kubernetes Service を作成および管理するためのユーザー インターフェイスです。 Azure Stack HCI 上の Azure Kubernetes Service を扱う Windows Admin Center を使用するには、下記の一覧のすべての条件を満たす必要があります。 

Windows Admin Center ゲートウェイを実行するマシンには、次の要件があります。 

 - Windows 10 または Windows Server マシン (現時点では、Azure Stack HCI または Windows Server 2019 Datacenter クラスターでの Windows Admin Center の実行はサポートされていません)
 - 60 GB の空き領域
 - Azure に登録済み
 - Azure Stack HCI または Windows Server 2019 Datacenter クラスターと同じドメイン内

## <a name="next-steps"></a>次のステップ 

上記の前提条件すべてが満たされたら、以下を使用して Azure Stack HCI 上に Azure Kubernetes Service ホストを設定できます。
 - [Windows Admin Center](setup.md)
 - [PowerShell](setup-powershell.md)