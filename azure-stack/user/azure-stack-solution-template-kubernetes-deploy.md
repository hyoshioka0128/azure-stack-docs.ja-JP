---
title: Azure Stack Hub コンテナーを使用するための Kubernetes をデプロイする
description: Azure Stack Hub でコンテナーを使用するための Kubernetes をデプロイする方法について説明します。
author: mattbriggs
ms.topic: article
ms.date: 2/1/2021
ms.author: mabrigg
ms.reviewer: waltero
ms.lastreviewed: 06/18/2019
ms.openlocfilehash: ead1d1cad49876643080e57ba45b0c430596113a
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99247712"
---
# <a name="deploy-kubernetes-to-use-containers-with-azure-stack-hub"></a>Azure Stack Hub でコンテナーを使用するための Kubernetes をデプロイする

> [!NOTE]  
> Kubernetes Azure Stack Marketplace 項目のみを使用して、概念実証としてクラスターをデプロイします。 Azure Stack でサポートされている Kubernetes クラスターの場合は、[AKS エンジン](azure-stack-kubernetes-aks-engine-overview.md)を使用します。

この記事の手順に従えば、1 回の連携した操作で Kubernetes のリソースをデプロイおよび設定することができます。 この手順では、Azure Resource Manager ソリューション テンプレートを使用します。 Azure Stack Hub のインストールに関する必要な情報を収集し、テンプレートを生成してから、クラウドにデプロイする必要があります。 Azure Stack Hub テンプレートでは、グローバルな Azure で提供されているのと同じマネージド型の AKS サービスは使用されません。

## <a name="kubernetes-and-containers"></a>Kubernetes とコンテナー

Azure Stack Hub 上の AKS エンジンによって生成された Azure Resource Manager テンプレートを使用して、Kubernetes をインストールできます。 [Kubernetes](https://kubernetes.io) はコンテナー内でのアプリケーションのデプロイ、スケーリング、管理を自動化するオープンソース システムです。 [コンテナー](https://www.docker.com/what-container)はイメージに含まれます。 コンテナー イメージは仮想マシン (VM) と似ていますが、VM とは違って、コンテナーには、コード、コードを実行するためのランタイム、特定のライブラリ、および設定など、アプリケーションを実行するために必要なリソースのみが含まれています。

Kubernetes は、次の目的で使用できます。

- 数秒でデプロイできる極めてスケーラブルかつアップグレード可能なアプリケーションを開発する。 
- アプリケーションの設計を簡素化し、異なる Helm アプリケーションによって信頼性を向上させる。 [Helm](https://github.com/kubernetes/helm) は、Kubernetes アプリケーションのライフサイクルをインストールおよび管理するのに役立つオープン ソースのパッケージ化ツールです。
- アプリケーションの正常性を簡単に監視および診断する。

クラスターをサポートするノードに必要なコンピューティングの使用量にのみ課金されます。 詳細については、「[Azure Stack Hub での使用量と請求](../operator/azure-stack-billing-and-chargeback.md)」を参照してください。

## <a name="deploy-kubernetes-to-use-containers"></a>コンテナーを使用するための Kubernetes をデプロイする

Azure Stack Hub で Kubernetes クラスターをデプロイする手順は、ご使用の ID 管理サービスによって異なります。 Azure Stack Hub のインストールで使用される ID 管理ソリューションを確認します。 ご使用の ID 管理サービスを確認するには、Azure Stack Hub 管理者に問い合わせてください。

- **Azure Active Directory (Azure AD)**  
Azure AD を使用した場合のクラスターのインストール手順については、「[Azure Active Directory (Azure AD) を使用して Azure Stack Hub に Kubernetes をデプロイする](azure-stack-solution-template-kubernetes-azuread.md)」を参照してください。

- **Active Directory フェデレーション サービス (AD FS)**  
AD FS を使用した場合のクラスターのインストール手順については、「[Active Directory フェデレーション サービス (AD FS) を使用して Azure Stack Hub に Kubernetes をデプロイする](azure-stack-solution-template-kubernetes-adfs.md)」を参照してください。

## <a name="connect-to-your-cluster"></a>クラスターへの接続

これで、クラスターに接続する準備ができました。 マスターはクラスター リソース グループ内にあり、`k8s-master-<sequence-of-numbers>` という名前です。 マスターに接続するには SSH クライアントを使います。 マスターでは、Kubernetes のコマンドライン クライアントである **kubectl** を使ってクラスターを管理できます。 方法については、[Kubernetes.io](https://kubernetes.io/docs/reference/kubectl/overview) をご覧ください。

クラスターへのアプリのインストールと展開には、**Helm** パッケージ マネージャーも役に立ちます。 クラスターでの Helm のインストールと使用の方法については、[helm.sh](https://helm.sh/) をご覧ください。

## <a name="next-steps"></a>次のステップ

[Kubernetes ダッシュボードの有効化](azure-stack-solution-template-kubernetes-dashboard.md)

[Kubernetes を Marketplace に追加する (Azure Stack Hub のオペレーター)](../operator/azure-stack-solution-template-kubernetes-cluster-add.md)

[Azure Active Directory (Azure AD) を使用して Azure Stack Hub に Kubernetes をデプロイする](azure-stack-solution-template-kubernetes-azuread.md)

[Active Directory フェデレーション サービス (AD FS) を使用して Azure Stack Hub に Kubernetes をデプロイする](azure-stack-solution-template-kubernetes-adfs.md)

[Azure における Kubernetes](/azure/container-service/kubernetes/container-service-kubernetes-walkthrough)
