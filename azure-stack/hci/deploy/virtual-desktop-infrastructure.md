---
title: Azure Stack HCI に仮想デスクトップ インフラストラクチャ (VDI) をデプロイする
description: このトピックでは、Azure Stack HCI オペレーティング システム上で 仮想デスクトップ インフラストラクチャ (VDI) を計画、構成、およびデプロイする方法に関するガイダンスを提供します。
author: JohnCobb1
ms.author: v-johcob
ms.topic: how-to
ms.date: 01/15/2021
ms.openlocfilehash: dfe22eb768b5bf661760c31c913d8c93caf590a4
ms.sourcegitcommit: 0983c1f90734b7ea5e23ae614eeaed38f9cb3c9a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/19/2021
ms.locfileid: "98571595"
---
# <a name="deploy-virtual-desktop-infrastructure-vdi-on-azure-stack-hci"></a>Azure Stack HCI に仮想デスクトップ インフラストラクチャ (VDI) をデプロイする

>適用対象:Azure Stack HCI バージョン 20H2

このトピックでは、Azure Stack HCI オペレーティング システム上で 仮想デスクトップ インフラストラクチャ (VDI) を計画、構成、およびデプロイする方法に関するガイダンスを提供します。 Azure Stack HCI への投資を活用して、組織内のユーザーのために、一元化され簡略化された高可用性かつ安全な管理を実現します。 このガイダンスを使用すると、ユーザー所有デバイスの持ち込み (BYOD) のようなシナリオがユーザーのために可能になると同時に、セキュリティを犠牲にすることなく、ビジネスに不可欠なアプリケーションに対して一貫した信頼性の高いエクスペリエンスを提供できます。

## <a name="overview"></a>概要
VDI は、サーバー ハードウェアを使用して、仮想マシン (VM) 上でデスクトップ オペレーティング システムとソフトウェア プログラムを実行します。 このように、VDI を使用すると、従来のデスクトップ ワークロードを集中管理されたサーバー上で実行できます。 ビジネスの場での VDI の利点としては、機密性の高い企業アプリケーションとデータを安全なデータセンターに保持できることや、個人データと会社の資産との混合を懸念せずに BYOD ポリシーに対応できることがあります。 また、VDI は、リモート オフィスやブランチ オフィスの従業員をサポートし、請負業者やパートナーにアクセスを提供するための標準にもなっています。

Azure Stack HCI は、VDI に最適なプラットフォームを提供します。 検証済みの Azure Stack HCI ソリューションと Microsoft [リモート デスクトップ サービス (RDS)](/windows-server/remote/remote-desktop-services/welcome-to-rds) を組み合わせることによって、可用性や拡張性が高いアーキテクチャを実現できます。

さらに、Azure Stack HCI VDI には、VDI のワークロードとクライアントを保護するために、次のような固有のクラウドベースの機能が用意されています。
- Azure Update Management を使用する一元管理された更新プログラム
- VDI クライアント向けの統合されたセキュリティ管理および高度な脅威防止

## <a name="deploy-vdi"></a>VDI のデプロイ
このセクションでは、Azure Stack HCI 上に VDI をデプロイするためのハードウェアを取得する方法と、管理のために Windows Admin Center を使用する方法について説明します。 また、VDI をサポートする RDS の展開についても説明します。

### <a name="step-1-acquire-hardware-for-vdi-on-azure-stack-hci"></a>手順 1:Azure Stack HCI 上で VDI 用のハードウェアを取得する
最初に、ハードウェアを調達する必要があります。 最も簡単な方法としては、[Azure Stack HCI カタログ](https://hcicatalog.azurewebsites.net)で適切な Microsoft ハードウェア パートナーを見つけて、Azure Stack HCI オペレーティング システムがプレインストールされている統合システムを購入します。 カタログでは、フィルターを適用して、この種類のワークロードに最適化されたベンダーのハードウェアを表示できます。 ハードウェア パートナーに問い合わせて、クラスター上でホストしようとする仮想デスクトップ数がハードウェアでサポートできることを確認してください。

それ以外の場合は、Azure Stack HCI オペレーティング システムを独自のハードウェアにデプロイする必要があります。 Azure Stack HCI デプロイ オプションの詳細および Windows Admin Center のインストールについては、「[Azure Stack HCI オペレーティング システムのデプロイ](./operating-system.md)」を参照してください。

次に、Windows Admin Center を使用して、[Azure Stack HCI クラスターを作成します](./create-cluster.md)。

### <a name="step-2-set-up-azure-update-management-in-windows-admin-center"></a>手順 2:Windows Admin Center で Azure Update Management を設定する
Windows Admin Center で [Azure Update Management](/windows-server/manage/windows-admin-center/azure/azure-update-management) を設定し、利用可能な更新プログラムの状態をすばやく評価したり、必要な更新をスケジュールしたり、デプロイ結果を確認して適用された更新を確認したりできます。

Azure Update Management を使用するには、Microsoft Azure のサブスクリプションが必要です。 サブスクリプションがない場合は、[無料試用版](https://azure.microsoft.com/free)にサインアップできます。

Windows Admin Center を使用して、Backup、File Sync、Site Recovery、Point-to-Site VPN、Azure Security Center などの追加の Azure ハイブリッド サービスを設定することもできます。

### <a name="step-3-deploy-remote-desktop-services-rds-for-vdi-support"></a>手順 3: VDI サポート用にリモート デスクトップ サービス (RDS) を展開する
Azure Stack HCI のデプロイを完了し、Update Management を使用するように Azure に登録したら、このセクションのガイダンスを使用して、VDI をサポートする RDS を構築して展開することができます。

RDS を構築して展開するには:
1. [リモート デスクトップ環境を展開する](/windows-server/remote/remote-desktop-services/rds-deploy-infrastructure)
1. アプリとリソースを共有するための [RDS セッション コレクションを作成する](/windows-server/remote/remote-desktop-services/rds-create-collection)
1. [RDS 展開のライセンス](/windows-server/remote/remote-desktop-services/rds-client-access-license)
1. アプリとリソースにアクセスするために [リモート デスクトップ クライアント](/windows-server/remote/remote-desktop-services/clients/remote-desktop-clients)をインストールするようユーザーに指示する
1. 高可用性を実現するために接続ブローカーとセッション ホストを追加する
    - [RD セッション ホスト ファームを使用して既存の RDS コレクションをスケールアウトする](/windows-server/remote/remote-desktop-services/rds-scale-rdsh-farm)
    - [RD 接続ブローカー インフラストラクチャに高可用性を追加する](/windows-server/remote/remote-desktop-services/rds-connection-broker-cluster)
    - [RD Web および RD ゲートウェイ Web フロントに高可用性を追加する](/windows-server/remote/remote-desktop-services/rds-rdweb-gateway-ha)
    - [UPD 記憶域用に 2 ノードの記憶域スペース ダイレクト ファイル システムを展開する](/windows-server/remote/remote-desktop-services/rds-storage-spaces-direct-deployment)

## <a name="next-steps"></a>次のステップ
VDI に関する詳細については、「[リモート デスクトップ サービスにおいてサポートされる構成](/windows-server/remote/remote-desktop-services/rds-supported-config)」を参照してください。
