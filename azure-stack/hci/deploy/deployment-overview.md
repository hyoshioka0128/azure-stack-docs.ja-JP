---
title: Azure Stack HCI のデプロイの概要
description: Azure Stack HCI のデプロイ プロセスの概要。
author: khdownie
ms.author: v-kedow
ms.topic: overview
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 12/7/2020
ms.openlocfilehash: 11c82a35d17a61fee474123a658f91ea90790221
ms.sourcegitcommit: 62eb5964a824adf7faee58c1636b17fedf4347e9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/07/2020
ms.locfileid: "96778021"
---
# <a name="what-is-the-deployment-process-for-azure-stack-hci"></a>Azure Stack HCI のデプロイ プロセスとは

> 適用対象:Azure Stack HCI バージョン 20H2

このトピックでは、Azure Stack HCI のデプロイ プロセスとその手順の概要を説明し、詳細情報へのリンクを示します。

## <a name="plan"></a>プラン

デプロイする前に、マルチサイト クラスタリング (存在する場合) のストレージ、ネットワーク、要件を慎重に計画してください。

### <a name="plan-storage"></a>ストレージを計画する

Azure Stack HCI では、可用性と拡張性が高いソフトウェアによるストレージを作成するため、ローカルに接続されたドライブを備える業界標準のサーバーが使用されます。 パフォーマンスと容量の要件を満たすには、慎重に[ドライブ を選択](../concepts/choose-drives.md)し、[ボリュームを計画](../concepts/plan-volumes.md)します。

### <a name="plan-networking"></a>ネットワークを計画する

デプロイのために、サーバー名、ドメイン名、RDMA プロトコルとバージョン、VLAN ID を記録しておきます。

### <a name="plan-stretched-clusters"></a>ストレッチ クラスターを計画する

Azure Stack HCI のデプロイを複数のサイトに拡張する場合は、各サイトで必要なサーバーの数と、クラスターの構成をアクティブ - パッシブまたはアクティブ - アクティブのどちらにするかを決定します。 詳細については、[ストレッチ クラスターの概要](../concepts/stretched-clusters.md)に関する記事を参照してください。

## <a name="deploy"></a>配置

OS をデプロイする前に、ハードウェアが Azure Stack HCI の[システム要件](../concepts/system-requirements.md)を満たしているかどうかを確認します。 その後、Azure Stack HCI クラスターを管理するために [Windows Admin Center をインストール](/windows-server/manage/windows-admin-center/deploy/install)します。

### <a name="1-deploy-azure-stack-hci"></a>1.Deploy Azure Stack HCI のデプロイ

[Azure Stack HCI をダウンロード](https://azure.microsoft.com/products/azure-stack/hci/hci-download/)し、クラスター化する各サーバーに Azure Stack HCI の[オペレーティング システム](operating-system.md)をデプロイします。 お好みの Microsoft ハードウェア パートナーから [Azure Stack HCI カタログ](https://aka.ms/azurestackhcicatalog) に載せられている Azure Stack HCI 統合システム ソリューション ハードウェアを購入した場合、Azure Stack HCI オペレーティング システムはプレインストールされているはずです。 その場合は、このステップを省略して #2 に進むことができます。

### <a name="2-create-the-cluster"></a>2.クラスターを作成する

[Windows Admin Center](create-cluster.md) または [PowerShell](create-cluster-powershell.md) を使用して、フェールオーバー クラスターを作成します。 ネイティブなディザスター リカバリーとビジネス継続性のため、2 つの地理的に異なるサイトにまたがる[ストレッチ クラスター](../concepts/stretched-clusters.md)をデプロイできます。

### <a name="3-set-up-a-cluster-witness"></a>3.クラスター監視のセットアップ

[監視リソースの設定](witness.md)は、すべてのクラスターに対して必須です。 2 ノード クラスターでは、どちらかのサーバーがオフラインになったときに他のノードも使用できなくなることがないように、ミラーリング監視サーバーが必要です。 3 つ以上のノードのクラスターでは、2 台のサーバーの障害またはオフラインに耐えられるように監視が必要です。 

### <a name="4-register-with-azure"></a>4.Azure に登録する

Azure Stack HCI には、Azure への接続が必要です。 クラスターを Azure に接続する方法については、[Azure への Azure Stack HCI の登録](register-with-azure.md)に関する記事を参照してください。 登録が済むと、クラスターはバックグラウンドで自動的に接続されます。

### <a name="5-validate-the-cluster"></a>5.クラスターを検証する

クラスターを作成して登録した後、クラスターの運用を始める前に、[クラスター検証テスト](validate.md)を実行して、ハードウェアまたは構成の問題を把握します。

### <a name="6-deploy-storage"></a>6.ストレージをデプロイする

単一サイトのクラスターでは[ボリュームを作成](../manage/create-volumes.md)し、[ストレッチ クラスターではボリュームを作成してレプリケーションを設定](../manage/create-stretched-volumes.md)します。

### <a name="7-deploy-workloads"></a>7.ワークロードのデプロイ

これで、Windows Admin Center を使用して、Azure Stack HCI にワークロードをデプロイできるようになりました。

Azure Stack HCI で Azure Kubernetes Service ワークロードのデプロイを開始する方法の詳細については、次の概要を参照してください。[Azure Kubernetes Service on Azure Stack HCI とは](https://docs.microsoft.com/azure-stack/aks-hci/overview)

## <a name="next-steps"></a>次のステップ

Azure Stack HCI オペレーティング システムのデプロイ方法をご確認ください。

> [!div class="nextstepaction"]
> [Azure Stack HCI オペレーティング システムのデプロイ](operating-system.md)
