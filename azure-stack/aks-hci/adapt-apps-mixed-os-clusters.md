---
title: 混合 OS Kubernetes クラスターでの使用のためにアプリケーションを適応させる
description: Azure Kubernetes Service でノード セレクターまたはテイントと容認を使用して、適切なワーカー ノード オペレーティング システムで、Azure Stack HCI で実行される混合 OS Kubernetes クラスター内のアプリケーションが確実にスケジュールされるようにする方法
author: abha
ms.topic: how-to
ms.date: 10/20/2020
ms.author: abha
ms.reviewer: ''
ms.openlocfilehash: 04b103fee921cf8bdab82a4004c6c80afd54d687
ms.sourcegitcommit: be445f183d003106192f039990d1fb8ee151c8d7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/20/2020
ms.locfileid: "92253947"
---
# <a name="adapt-apps-for-mixed-os-kubernetes-clusters-using-node-selectors-or-taints-and-tolerations"></a>ノード セレクターまたはテイントと容認を使用して、混合 OS Kubernetes クラスターにアプリを適応させます

Azure Stack HCI 上の Azure Kubernetes Service を使用すると、Linux と Windows の両方のノードで Kubernetes クラスターを実行できますが、これらの混合 OS クラスターで使用するためには、アプリに対して小さな編集を加える必要があります。 この攻略ガイドでは、ノード セレクターまたはテイントと容認のどちらかを使用して、適切なホスト OS 上でアプリケーションが確実にスケジュールされるようにする方法について説明します。

この攻略ガイドは、Kubernetes の概念についての基礎知識が前提となっています。 詳細については、「[Azure Stack HCI 上の Azure Kubernetes Service における Kubernetes の中心概念](kubernetes-concepts.md)」を参照してください。

## <a name="node-selector"></a>ノード セレクター

*ノード セレクター*は、ポッド仕様の YAML に含まれる簡単なフィールドであり、オペレーティング システムが一致する正常なノードにのみスケジュールされるように、ポッドが制約されます。 ポッド仕様の YAML では、以下の例に示すように、Windows または Linux として `nodeSelector` を指定します。 

```yaml
kubernetes.io/os = Windows
```
または、

```yaml
kubernetes.io/os = Linux
```

nodeSelectors の詳細については、[ノード セレクター](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/)に関するページを参照してください。 

## <a name="taints-and-tolerations"></a>テイントと容認

*テイント*と*容認*は、連携動作して、ポッドが意図せずノードにスケジュールされないようにします。 ノードを "テイント" すると、ポッド仕様の YAML の "toleration" を通して明示的にテイントを容認していないポッドを受け入れないようにすることができます。

Azure Stack HCI 上の Azure Kubernetes Service の Windows OS ノードは、次のキーと値のペアでテイントすることができます。 ユーザーは異なるものを使用してはいけません。

```yaml
node.kubernetes.io/os=Windows:NoSchedule
```
`kubectl get` を実行し、テイントする Windows ワーカー ノードを識別します。

```PowerShell
kubectl get nodes --all-namespaces -o=custom-columns=NAME:.metadata.name,OS:.status.nodeInfo.operatingSystem
```
出力:
```output
NAME                                     OS
my-aks-hci-cluster-control-plane-krx7j   linux
my-aks-hci-cluster-md-md-1-5h4bl         windows
my-aks-hci-cluster-md-md-1-5xlwz         windows
```

`kubectl taint node` を使用して Windows サーバー ワーカー ノードをテイントします。

```PowerShell
kubectl taint node my-aks-hci-cluster-md-md-1-5h4bl node.kubernetes.io/os=Windows:NoSchedule
kubectl taint node my-aks-hci-cluster-md-md-1-5xlwz node.kubernetes.io/os=Windows:NoSchedule
```

ポッドの容認は、ポッド仕様の YAML 内で指定します。 次の容認は、上の kubectl taint 行によって作成されたテイントと "一致" します。したがって、この容認が指定されたポッドは、my-aks-hci-cluster-md-md-1-5h4bl 上または my-aks-hci-cluster-md-md-1-5xlwz 上にスケジュールできるようになります。

```yaml
tolerations:
- key: node.kubernetes.io/os
  operator: Equal
  value: Windows
  effect: NoSchedule
```
テイントと容認の詳細については、「[テイントと容認](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)」を参照してください。 

## <a name="next-steps"></a>次の手順

この攻略ガイドでは、kubectl を使用して、Kubernetes クラスターにノード セレクターまたはテイントと容認を追加する方法について説明しました。 次に、以下を実行できます。
- [Kubernetes クラスターに Linux アプリケーションをデプロイします](./deploy-linux-application.md)。
- [Kubernetes クラスターに Windows Server アプリケーションをデプロイします](./deploy-windows-application.md)。
