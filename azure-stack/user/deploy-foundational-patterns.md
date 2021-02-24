---
title: Azure Stack Hub 上に基本パターンをデプロイする
description: Azure Stack Hub で基本パターンをデプロイする方法について説明します。
author: mattbriggs
ms.topic: how-to
ms.date: 12/2/2020
ms.author: mabrigg
ms.reviewer: sijuman
ms.lastreviewed: 12/2/2020
ms.openlocfilehash: 5609d9195a909a20a54917555a7309fe0901d7fa
ms.sourcegitcommit: 9ef2cdc748cf00cd3c8de90705ea0542e29ada97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/02/2020
ms.locfileid: "96525474"
---
# <a name="deploy-foundational-patterns-overview"></a>基本パターンのデプロイの概要


これらの各パターンには、ガイダンス、Azure Resource Manager テンプレート、およびチュートリアルが含まれています。 これらのパターンをサードパーティ アプリと共に使用して、Azure Stack でまだサポートされていないオファリングを作成できます。 たとえば、オペレーターは、多くの場合、1 つの Azure Stack Hub インスタンスに仮想プライベート ネットワーク (VPN) を設定する際の複雑さに対応し、複数の環境にまたがる VPN を作成することはほとんどありません。 Azure Stack Hub の前にロード バランサーを作成してワークロードを管理しようとすると、オペレーターが問題に遭遇することがあります。 次のガイダンスを使用すると、運用環境に対応したワークロードを解放するためのデプロイ時間を短縮できます。

## <a name="networking"></a>ネットワーク

ネットワーク パターンを使用すると、Azure Stack Hub を使用して仮想ネットワーク ピアリングを作成する手順がわかります。 仮想ネットワーク ピアリングを使用すると、2 つの仮想ネットワークを接続し、1 つのネットワークとして見せることができます。 複数の仮想ネットワーク間のサイト間接続は、リモートおよびルーティング サービス (RRAS) を介して実現されます。 RRAS を使用すると、Windows 仮想マシン (VM) をルーターとして機能させることができます。 これらのスクリプトを使用すると、1 つの Azure Stack Hub リソース グループ内のリソース グループ間、サブスクリプション間、および 2 つの Azure Stack Hub インスタンス間に 2 つの仮想ネットワークをデプロイできます。 Azure Stack Hub およびグローバル Azure にスクリプトをデプロイできます。 

各記事では、次のような一般的な考慮事項について説明します。 
- スケール
- 帯域幅
- Security
- ビジネス継続性

|  仮想ネットワーク ピアリング  |  VPN  |  Load Balancer  |
| --- | --- | --- |
| ![VM との仮想ネットワーク ピアリング](media/deploy-foundational-patterns/icon-networking-61-virtual-networks.svg)<br>[VM との仮想ネットワーク ピアリング](azure-stack-network-howto-vnet-peering.md) | ![オンプレミスへの VPN を設定する](media/deploy-foundational-patterns/icon-networking-63-virtual-network-gateways.svg)<br>[オンプレミスへの VPN を設定する](azure-stack-network-howto-vnet-to-onprem.md) | ![F5 ロード バランサー](media/deploy-foundational-patterns/icon-networking-62-load-balancers.svg)<br>[F5 ロード バランサー](network-howto-f5.md) |
| ![FortiGate との仮想ネットワーク ピアリング](media/deploy-foundational-patterns/icon-networking-61-virtual-networks.svg)<br>[FortiGate との仮想ネットワーク ピアリング](azure-stack-network-howto-vnet-to-vnet.md) | ![仮想プライベート ネットワーク](media/deploy-foundational-patterns/icon-networking-63-virtual-network-gateways.svg)<br>[仮想ネットワークから仮想ネットワークへの接続](azure-stack-network-howto-vnet-to-vnet-stacks.md) |  |
|  | ![VPN トンネル (GRE) を作成する](media/deploy-foundational-patterns/icon-networking-63-virtual-network-gateways.svg)<br>[VPN トンネル (GRE) を作成する](network-howto-vpn-tunnel-gre.md) | |
|  | ![複数サイト間 VPN を設定する](media/deploy-foundational-patterns/icon-networking-63-virtual-network-gateways.svg)<br>[複数サイト間 VPN を設定する](network-howto-vpn-tunnel.md) | |
|  | ![VPN トンネル (IPSEC) を作成する](media/deploy-foundational-patterns/icon-networking-63-virtual-network-gateways.svg)<br>[VPN トンネル (IPSEC) を作成する](network-howto-vpn-tunnel-ipsec.md)| |


## <a name="storage"></a>ストレージ

Azure Stack Hub でストレージのオプションを増やすには、ストレージ パターンを使用します。 Azure Stack Hub では、ストレージは有限です。 既存のデータセンター内のリソースに接続します。 Azure Stack Hub に Windows VM を作成して外部 iSCSI ターゲットに接続する手順を確認してください。 マルチパス I/O (MPIO) などの主要な機能を有効にして、VM と外部ストレージ間のパフォーマンスと接続を最適化する方法を学習できます。

| iSCSI ストレージ | ストレージを拡張する |
| --- | --- |
| ![iSCSI ストレージに接続する](media/deploy-foundational-patterns/icon-storage-87-storage-accounts-classic.svg)<br>[iSCSI ストレージに接続する](azure-stack-network-howto-iscsi-storage.md) | ![データセンターを拡張する](media/deploy-foundational-patterns/icon-storage-88-recovery-services-vaults.svg)<br>[データセンターを拡張する](azure-stack-network-howto-extend-datacenter.md) |

## <a name="backup"></a>バックアップ

バックアップとディザスター リカバリー パターンを使用すると、サブスクリプション内のすべてのリソースを Azure または別の Azure Stack Hub インスタンスにコピーできます。 これらのパターンでは、Commvault のライブ同期を使用して、VM の内部に保存されている情報を別の環境にレプリケートすることを考慮しています。 ストレージ アカウントとバックアップ ストレージ アカウントを作成してデータを送信するスクリプトがあります。 モジュール Azure サブスクリプション レプリケーターを使用すると、リソース レプリケーションを調整できます。また、さまざまなリソースを処理するようにプロセッサをカスタマイズできます。 



|  バックアップ  |  コピー  |
| --- | --- |
| ![Commvault を使用して Azure Stack Hub で VM をバックアップする](media/deploy-foundational-patterns/icon-storage-100-import-export-jobs.svg)<br>[Commvault を使用して Azure Stack Hub で VM をバックアップする](azure-stack-network-howto-backup-commvault.md) | ![サブスクリプション リソースをコピーする](media/deploy-foundational-patterns/icon-storage-94-data-box.svg)<br>[サブスクリプション リソースをコピーする](azure-stack-network-howto-backup-replicator.md) |
|  ![Azure Stack Hub でストレージ アカウントをバックアップする](media/deploy-foundational-patterns/icon-storage-93-storage-sync-services.svg)<br>[Azure Stack Hub でストレージ アカウントをバックアップする](azure-stack-network-howto-backup-storage.md)  | |

## <a name="github-samples"></a>GitHub のサンプル

テンプレートは [Azure Intelligent Edge Patterns GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns) リポジトリにあります。

## <a name="next-steps"></a>次のステップ

[Azure のハイブリッド パターンとソリューションのドキュメント](/hybrid/app-solutions)
