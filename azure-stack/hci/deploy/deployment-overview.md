---
title: Azure Stack HCI のデプロイの概要
description: Azure Stack HCI のデプロイ プロセスの概要。
author: khdownie
ms.author: v-kedow
ms.topic: overview
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 09/09/2020
ms.openlocfilehash: 34a93a65d45861c7c7ff1727347cc95465968151
ms.sourcegitcommit: 69cfff119ab425d0fbb71e38d1480d051fc91216
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/30/2020
ms.locfileid: "91572519"
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

### <a name="1-before-you-begin"></a>1.開始する前に

始める前に、[ハードウェアが基本要件を満たしているかどうかを確認し、Azure Stack HCI のデプロイに必要な情報を収集](before-you-start.md)します。 その後、Azure Stack HCI クラスターを管理するために [Windows Admin Center をインストール](/windows-server/manage/windows-admin-center/deploy/install)します。

### <a name="2-deploy-azure-stack-hci"></a>2.Deploy Azure Stack HCI のデプロイ

[Azure Stack HCI をダウンロード](https://azure.microsoft.com/products/azure-stack/hci/hci-download/)し、クラスター化する各サーバーに Azure Stack HCI の[オペレーティング システム](operating-system.md)をデプロイします。

### <a name="3-create-the-cluster"></a>3.クラスターを作成する

[Windows Admin Center](create-cluster.md) または [PowerShell](create-cluster-powershell.md) を使用して、フェールオーバー クラスターを作成します。 ネイティブなディザスター リカバリーとビジネス継続性のため、2 つの地理的に異なるサイトにまたがる[ストレッチ クラスター](../concepts/stretched-clusters.md)をデプロイできます。

### <a name="4-set-up-a-cluster-witness"></a>4.クラスター監視のセットアップ

[監視リソースの設定](witness.md)は、すべてのクラスターに対して必須です。 2 ノード クラスターでは、どちらかのサーバーがオフラインになったときに他のノードも使用できなくなることがないように、ミラーリング監視サーバーが必要です。 3 つ以上のノードのクラスターでは、2 台のサーバーの障害またはオフラインに耐えられるように監視が必要です。 

### <a name="5-register-with-azure"></a>5.Azure に登録する

Azure Stack HCI には、Azure への接続が必要です。 クラスターを Azure に接続する方法については、[Azure への Azure Stack HCI の登録](register-with-azure.md)に関する記事を参照してください。 登録が済むと、クラスターはバックグラウンドで自動的に接続されます。

### <a name="6-validate-the-cluster"></a>6.クラスターを検証する

クラスターを作成して登録した後、クラスターの運用を始める前に、[クラスター検証テスト](validate.md)を実行して、ハードウェアまたは構成の問題を把握します。

### <a name="7-deploy-storage"></a>7.ストレージをデプロイする

単一サイトのクラスターでは[ボリュームを作成](../manage/create-volumes.md)し、[ストレッチ クラスターではボリュームを作成してレプリケーションを設定](../manage/create-stretched-volumes.md)します。

### <a name="8-deploy-workloads"></a>8.ワークロードのデプロイ

これで、Windows Admin Center を使用して、[仮想マシンを作成](../manage/vm.md)し、Azure Stack HCI にワークロードをデプロイできるようになりました。

## <a name="next-steps"></a>次のステップ

Azure Stack HCI をデプロイする前に行う必要があることを学習します。

> [!div class="nextstepaction"]
> [Before you begin](before-you-start.md)
