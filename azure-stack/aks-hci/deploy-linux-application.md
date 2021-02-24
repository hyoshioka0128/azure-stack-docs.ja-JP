---
title: チュートリアル - Azure Stack HCI 上の AKS に Linux アプリケーションをデプロイする
description: このチュートリアルでは、Azure Container Registry に格納されているカスタム イメージを使用して、マルチコンテナー Linux アプリケーションをクラスターにデプロイします。
author: abha
ms.topic: tutorial
ms.date: 12/02/2020
ms.author: abha
ms.reviewer: ''
ms.openlocfilehash: e5f7f96956248dc7cc2c92ae678970b40951ece4
ms.sourcegitcommit: 0efffe1d04a54062a26d5c6ce31a417f511b9dbf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2020
ms.locfileid: "96612303"
---
# <a name="tutorial-deploy-linux-applications-in-azure-kubernetes-service-on-azure-stack-hci"></a>チュートリアル:Azure Stack HCI 上の Azure Kubernetes Service に Linux アプリケーションをデプロイする

> 適用対象:AKS on Azure Stack HCI、AKS runtime on Windows Server 2019 Datacenter

このチュートリアルでは、Azure Stack HCI クラスター上の Azure Kubernetes Service に Web フロント エンドと Redis データベース インスタンスを含むマルチコンテナー アプリケーションをデプロイします。 次に、アプリケーションをテストおよびスケーリングする方法を確認します。 

このチュートリアルでは、Kubernetes の概念の基本的な理解を前提としています。 詳細については、「[Azure Stack HCI 上の Azure Kubernetes Service における Kubernetes の中心概念](kubernetes-concepts.md)」を参照してください。

## <a name="before-you-begin"></a>開始する前に

次の要件の準備ができていることを確認します。

* 少なくとも 1 つの Linux ワーカー ノードが稼働している Azure Stack HCI クラスター上の Azure Kubernetes Service。 
* クラスターにアクセスするための kubeconfig ファイル。
* Azure Stack HCI PowerShell モジュールに Azure Kubernetes Service がインストールされている。
* PowerShell 管理ウィンドウでこのドキュメントのコマンドを実行する。
* OS 固有のワークロードが適切なコンテナー ホストに配置されていることを確認する。 Linux と Windows のワーカー ノードが混在する Kubernetes クラスターがある場合は、ノード セレクター、またはテイントと容認のいずれかを使用できます。 詳細については、[ノード セレクター、テイントと容認の使用](adapt-apps-mixed-os-clusters.md)に関するページを参照してください。

## <a name="deploy-the-application"></a>アプリケーションをデプロイする

Kubernetes のマニフェスト ファイルでは、どのコンテナー イメージを実行するかなど、クラスターの望ましい状態を定義します。 このクイックスタートでは、マニフェストを使用して、[Azure 投票アプリケーション](https://github.com/Azure-Samples/azure-voting-app-redis)を実行するために必要なすべてのオブジェクトを作成します。 このマニフェストには、 Kubernetes デプロイ が 2 つ含まれます。サンプル Azure Vote Python アプリケーション用と Redis インスタンス用です。 さらに、Kubernetes サービスが 2 つ作成されます。Redis インスタンスに使用される内部サービスと、Azure 投票アプリケーションにインターネットからアクセスするための外部サービスです。

`azure-vote.yaml` という名前のファイルを作成し、以下の YAML 定義をコピーします。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: azure-vote-back
spec:
  replicas: 1
  selector:
    matchLabels:
      app: azure-vote-back
  template:
    metadata:
      labels:
        app: azure-vote-back
    spec:
      nodeSelector:
        "beta.kubernetes.io/os": linux
      containers:
      - name: azure-vote-back
        image: redis
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
        ports:
        - containerPort: 6379
          name: redis
---
apiVersion: v1
kind: Service
metadata:
  name: azure-vote-back
spec:
  ports:
  - port: 6379
  selector:
    app: azure-vote-back
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: azure-vote-front
spec:
  replicas: 1
  selector:
    matchLabels:
      app: azure-vote-front
  template:
    metadata:
      labels:
        app: azure-vote-front
    spec:
      nodeSelector:
        "beta.kubernetes.io/os": linux
      containers:
      - name: azure-vote-front
        image: mcr.microsoft.com/azuredocs/azure-vote-front:v1
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
        ports:
        - containerPort: 80
        env:
        - name: REDIS
          value: "azure-vote-back"
---
apiVersion: v1
kind: Service
metadata:
  name: azure-vote-front
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: azure-vote-front
```

`kubectl apply` コマンドを使用してアプリケーションをデプロイし、ご利用の YAML マニフェストの名前を指定します。

```PowerShell
kubectl apply -f azure-vote.yaml
```

次の出力例は、正常に作成されたデプロイおよびサービスを示しています。

```output
deployment "azure-vote-back" created
service "azure-vote-back" created
deployment "azure-vote-front" created
service "azure-vote-front" created
```

## <a name="test-the-application"></a>アプリケーションをテストする

アプリケーションが実行されると、Kubernetes サービスによってアプリケーション フロント エンドがインターネットに公開されます。 このプロセスが完了するまでに数分かかることがあります。

進行状況を監視するには、`kubectl get service` コマンドを `--watch` 引数と一緒に使用します。

```PowerShell
kubectl get service azure-vote-front --watch
```

最初に、*azure-vote-front* サービスの *EXTERNAL-IP* が "*保留中*" として表示されます。

```output
NAME               TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
azure-vote-front   LoadBalancer   10.0.37.27      <pending>     80:30572/TCP   22m
```

*EXTERNAL-IP* アドレスが "*保留中*" から実際のパブリック IP アドレスに変わったら、`CTRL-C` を使用して `kubectl` ウォッチ プロセスを停止します。 次の出力例は、サービスに割り当てられている有効なパブリック IP アドレスを示しています。

```output
NAME               TYPE           CLUSTER-IP   EXTERNAL-IP     PORT(S)        AGE
azure-vote-front   LoadBalancer   10.0.37.27   52.179.23.131   80:30572/TCP   24m
```

Azure Vote アプリが動作していることを確認するには、Web ブラウザーを開いてサービスの外部 IP アドレスにアクセスします。

![Azure 上の Kubernetes クラスターの図](media/deploy-linux-application/azure-vote.png)

## <a name="scale-application-pods"></a>アプリケーション ポッドをスケーリングする

Azure 投票フロントエンドと Redis インスタンスの 1 つのレプリカを作成しました。 ご利用のクラスターに存在するポッドの数と状態を確認するには、次のように `kubectl get` コマンドを使用します。

```console
kubectl get pods -n default
```

次の出力例を見ると、1 つのフロントエンド ポッドと 1 つのバックエンド ポッドが存在することがわかります。

```
NAME                                READY     STATUS    RESTARTS   AGE
azure-vote-back-6bdcb87f89-g2pqg    1/1       Running   0          25m
azure-vote-front-84c8bf64fc-cdq86   1/1       Running   0          25m
```

*azure-vote-front* デプロイのポッド数を変更するには、`kubectl scale` コマンドを使用します。 次の例では、フロントエンド ポッドの数を *5* に増やしています。

```console
kubectl scale --replicas=5 deployment/azure-vote-front
```

`kubectl get pods` を再度実行して、追加のポッドが作成されたことを確認します。 しばらくすると、追加したポッドがクラスターで利用できる状態になります。

```console
kubectl get pods -n default

Name                                READY   STATUS    RESTARTS   AGE
azure-vote-back-6bdcb87f89-g2pqg    1/1     Running   0          31m
azure-vote-front-84c8bf64fc-cdq86   1/1     Running   0          31m
azure-vote-front-84c8bf64fc-56h64   1/1     Running   0          80s
azure-vote-front-84c8bf64fc-djkp8   1/1     Running   0          80s
azure-vote-front-84c8bf64fc-jmmvs   1/1     Running   0          80s
azure-vote-front-84c8bf64fc-znc6z   1/1     Running   0          80s
```

## <a name="next-steps"></a>次の手順

* [Azure Monitor を使用してクラスターとアプリケーションを監視します](/azure/azure-monitor/insights/container-insights-enable-arc-enabled-clusters)。
