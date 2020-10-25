---
title: Azure Stack Hub の AKS エンジンとは
description: AKS エンジンのコマンド ライン ツールを使用して、Azure および Azure Stack Hub で Kubernetes クラスターをデプロイおよび管理する方法を学習します。
author: mattbriggs
ms.topic: article
ms.date: 09/02/2020
ms.author: mabrigg
ms.reviewer: waltero
ms.lastreviewed: 09/02/2020
ms.openlocfilehash: da77872f4b80d3885a903bbd62497af68e148c01
ms.sourcegitcommit: b80d529ff47b15b8b612d8a787340c7b0f68165b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/04/2020
ms.locfileid: "89473012"
---
# <a name="what-is-the-aks-engine-on-azure-stack-hub"></a>Azure Stack Hub の AKS エンジンとは

AKS エンジンのコマンド ライン ツールを使用して、Azure および Azure Stack Hub で Kubernetes クラスターをデプロイおよび管理することができます。 AKS エンジンを使用して、Azure Resource Manager ネイティブ クラスターを作成、アップグレード、スケーリングします。 このエンジンを使用して、接続されている環境と接続されていない環境の両方でクラスターをデプロイできます。 この記事では、AKS エンジンの概要、Azure Stack Hub でエンジンを使用する際にサポートされるシナリオ、デプロイ、アップグレード、スケーリングなどの操作の概要について示します。

## <a name="overview-of-the-aks-engine"></a>AKS エンジンの概要

[AKS エンジン](https://github.com/Azure/aks-engine)には、Azure および Azure Stack Hub 上で Kubernetes クラスターをブートストラップするためのコマンド ライン ツールが用意されています。 Azure Resource Manager を使用することで、AKS エンジンは、VM、仮想ネットワーク、Azure Stack Hub 内のその他のサービスとしてのインフラストラクチャ (IaaS) リソース上で実行されているクラスターを作成し、維持するのに役立ちます。

## <a name="aks-engine-on-azure-stack-hub-considerations"></a>Azure Stack Hub の AKS エンジンの考慮事項

Azure Stack Hub で AKS エンジンを使用する前に、Azure Stack Hub と Azure の違いを理解しておくことが重要です。 このセクションでは、Azure Stack Hub と AKS エンジンを使用して Kubernetes クラスターを管理する際のさまざまな機能と主な考慮事項について確認します。

Azure Stack Hub の AKS エンジンの詳細と Azure に関する違いの詳細については、[Azure Stack Hub の AKS エンジン](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md)に関する記事を参照してください。

## <a name="supported-scenarios-with-the-aks-engine"></a>AKS エンジンでサポートされるシナリオ

Azure Stack Hub サポート チームによって、次のシナリオがサポートされています。

1.  AKS エンジンでは、このドキュメントのガイドラインに従って、[次のテンプレート](https://github.com/Azure/aks-engine/tree/master/examples/azure-stack)を使用して、すべてのクラスター成果物がデプロイされます。
2.  AKS エンジンにより、既存の VNET にクラスターがデプロイされます。 詳細については、「[Using a custom virtual network with AKS engine](https://github.com/Azure/aks-engine/blob/master/docs/tutorials/custom-vnet.md)」 (AKS エンジンでのカスタム仮想ネットワークの使用) を参照してください。
3.  [アップグレード](azure-stack-kubernetes-aks-engine-upgrade.md)と[スケール](azure-stack-kubernetes-aks-engine-scale.md)操作。

AKS エンジンと Azure Stack Hub の詳細については、「[Azure Stack Hub の AKS エンジンのサポート ポリシー](azure-stack-kubernetes-aks-engine-support.md)」を参照してください。

## <a name="install-the-aks-engine-and-deploy-a-kubernetes-cluster"></a>AKS エンジンをインストールし、Kubernetes クラスターをデプロイする

AKS エンジンを使用して Azure Stack Hub に Kubernetes クラスターをデプロイするには

1. [AKS エンジンの前提条件を設定する](azure-stack-kubernetes-aks-engine-set-up.md)
2. ご使用の Azure Stack Hub 環境にアクセスできるマシンに AKS エンジンをインストールします
     - [Azure Stack Hub の Windows に AKS エンジンをインストールする](azure-stack-kubernetes-aks-engine-deploy-windows.md)
     - [Azure Stack Hub の Linux に AKS エンジンをインストールする](azure-stack-kubernetes-aks-engine-deploy-linux.md)
3. [AKS エンジンを使用して Azure Stack Hub に Kubernetes クラスターをデプロイする](azure-stack-kubernetes-aks-engine-deploy-cluster.md)

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [AKS エンジンの前提条件を設定する](azure-stack-kubernetes-aks-engine-set-up.md)