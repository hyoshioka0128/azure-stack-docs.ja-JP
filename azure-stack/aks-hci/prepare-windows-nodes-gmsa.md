---
title: グループの管理サービス アカウントをサポートするための Windows ノードの準備
description: Windows ノード上のコンテナーに対してグループの管理サービス アカウントを構成する方法について説明します
author: v-susbo
ms.topic: how-to
ms.date: 11/30/2020
ms.author: v-susbo
ms.openlocfilehash: fd08f288d816468610dadf7e5c063c08b63c0cf2
ms.sourcegitcommit: 9b0e1264ef006d2009bb549f21010c672c49b9de
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98254706"
---
# <a name="prepare-windows-nodes-for-group-managed-service-account-support"></a>グループの管理サービス アカウントをサポートするための Windows ノードの準備

> 適用対象:Azure Stack HCI 上の AKS、Windows Server 2019 Datacenter 上の AKS ランタイム

グループの管理サービス アカウントは、以下を提供する種類の Active Directory アカウントです。
- 自動的なパスワード管理
- 簡略化されたサービス プリンシパル名 (SPN) の管理
- 複数のサーバー間で管理を他の管理者に委任する機能 

Windows ノードで実行されるポッドとコンテナーに対してグループの管理サービス アカウント (gMSA) を構成するには、最初に、Windows ノードを Active Directory ドメインに参加させる必要があります。

グループの管理サービス アカウントのサポートを有効にするには、Kubernetes クラスター名を 4 文字未満にする必要があります。 4 文字未満にする必要があるのは、ドメインに参加しているサーバー名でサポートされる最大長は 15 文字であるためです。 ワーカー ノードでの Azure Stack HCI Kubernetes クラスター上の AKS の名前付け規則によって、いくつかの定義済みの文字がノード名に追加されます。

## <a name="join-the-worker-nodes-to-a-domain"></a>ワーカー ノードをドメインに参加させる

Windows ワーカー ノードをドメインに参加させるには、`kubectl get` を実行して `EXTERNAL-IP` の値を書き留めることにより、Windows ワーカー ノードにログインします。

```
kubectl get nodes -o wide
```  

次に `ssh Administrator@ip` を使用して、SSH でノードに接続できます。 SSH を使用してログインする方法の詳細については、[SSH を使用した Windows ワーカー ノードのトラブルシューティング](troubleshoot.md#troubleshooting-windows-worker-nodes)に関する記事を参照してください。

SSH を使用して Windows ワーカー ノードに正常にログインしたら、次のコマンドを実行して、ノードをドメインに参加させます。 その後、Windows ワーカー ノードを再起動する必要があります。 

```
netdom.exe join %computername% /domain:DomainName /UserD:DomainName\UserName /PasswordD:Password
```  

すべての Windows ワーカー ノードがドメインに参加したら、[gMSA の構成](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa/)に関する記事で詳しく説明されている手順に従います。 これらの手順は、Kubernetes クラスターに Kubernetes gMSA カスタム リソース定義と Webhook を適用するのに役立ちます。

gMSA を使用する Windows コンテナーの詳細については、[Windows コンテナーと gMSA](/virtualization/windowscontainers/manage-containers/manage-serviceaccounts) に関する記事を参照してください。 

## <a name="next-steps"></a>次の手順

* [Azure Monitor を使用してクラスターとアプリケーションを監視します](/azure/azure-monitor/insights/container-insights-enable-arc-enabled-clusters)。
* [Kubernetes クラスターで永続ボリュームを使用します](persistent-volume.md)。