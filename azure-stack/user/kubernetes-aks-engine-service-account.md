---
title: Azure Stack Hub で AKS エンジンのサービス アカウント トークン ボリューム予測を有効にする
description: Azure Stack Hub で AKS エンジンのサービス アカウント トークン ボリューム予測を有効にする方法について説明します。
author: mattbriggs
ms.topic: article
ms.date: 10/23/2020
ms.author: mabrigg
ms.reviewer: waltero
ms.lastreviewed: 10/23/2020
ms.openlocfilehash: 5e712577bb01f7f084c24eadfd3931ebb67f23d9
ms.sourcegitcommit: 716ca50bd198fd51a4eec5b40d5247f6f8c16530
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/28/2020
ms.locfileid: "92906183"
---
# <a name="enabling-service-account-token-volume-projection-for-the-aks-engine-on-azure-stack-hub"></a>Azure Stack Hub で AKS エンジンのサービス アカウント トークン ボリューム予測を有効にする

Istio は、Kubernetes クラスターのコンテナーを接続し、監視し、セキュリティで保護する、構成可能な、オープン ソースのサービスメッシュ レイヤーです。 Istio 1.3 以降では、" *サービス アカウント トークン ボリューム プロジェクション* " という名称の Kubernetes の機能が使用されます。 この機能は、AKS エンジンでデプロイされる Kubernetes クラスターでは既定で有効になっていません。 この記事では、クラスターのサービス アカウント トークン ボリューム予測を有効にするために必要な Kubernetes API サーバー フラグを表示する `apiServerConfig` 要素の API モデル json プロパティを紹介します。

サービス アカウント トークン ボリューム予測の詳細については、「[Service Account Token Volume Projection](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#service-account-token-volume-projection)」 (サービス アカウント トークン ボリューム予測) を参照してください。

## <a name="enable-service-account-token-volume-projection"></a>サービス アカウント トークンのボリューム予測を有効にする

サービス アカウント トークンのボリューム予測を有効にするには、API モデル json ファイルに次の設定を追加します。 

```json
{
    "kubernetesConfig": {
        "apiServerConfig": {
            "--service-account-api-audiences": "api,istio-ca",
            "--service-account-issuer": "kubernetes.default.svc",
            "--service-account-signing-key-file": "/etc/kubernetes/certs/apiserver.key"
        }
    }
}
```

> [!Note]  
> 場合によっては、特定のユース ケースに合わせて `--service-account-api-audiences` と `--service-account-issuer` を調整する必要があります。

API モデルの完全なサンプルが必要であれば、[istio.json](https://github.com/Azure/aks-engine/blob/master/examples/service-mesh/istio.json) を参照してください。

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub 上の AKS エンジン](azure-stack-kubernetes-aks-engine-overview.md)を確認してください
- [Azure Stack Hub で Kubernetes クラスターをアップグレードする](azure-stack-kubernetes-aks-engine-upgrade.md)
