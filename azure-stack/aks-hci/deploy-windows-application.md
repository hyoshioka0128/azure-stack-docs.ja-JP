---
title: AKS on Azure Stack HCI に Windows .NET アプリケーションをデプロイする
description: Azure Container Registry に格納されているカスタム イメージを使用して、Windows .NET アプリケーションをクラスターにデプロイする方法について説明します。
author: abha
ms.topic: tutorial
ms.date: 12/02/2020
ms.author: abha
ms.reviewer: ''
ms.openlocfilehash: 844bafbb497228d2e4527a5e974574dadb95ee7d
ms.sourcegitcommit: 0efffe1d04a54062a26d5c6ce31a417f511b9dbf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2020
ms.locfileid: "96612320"
---
# <a name="deploy-windows-applications-in-azure-kubernetes-service-on-azure-stack-hci"></a>Azure Stack HCI 上の Azure Kubernetes Service に Windows アプリケーションをデプロイする

> 適用対象:AKS on Azure Stack HCI、AKS runtime on Windows Server 2019 Datacenter

このチュートリアルでは、Windows Server コンテナー内の ASP.NET サンプル アプリケーションを Kubernetes クラスターにデプロイし、アプリケーションのテストとスケーリングの方法の確認の仕方について説明します。 また、Windows ノードを Active Directory ドメインに参加させる方法についても説明します。
このチュートリアルでは、Kubernetes の概念の基本的な理解を前提としています。 詳細については、「[Azure Stack HCI 上の Azure Kubernetes Service における Kubernetes の中心概念](kubernetes-concepts.md)」を参照してください。

## <a name="before-you-begin"></a>開始する前に

次の要件の準備ができていることを確認します。

* 少なくとも 1 つの Windows ワーカー ノードが稼働している Azure Stack HCI クラスター上の Azure Kubernetes Service。 
* クラスターにアクセスするための kubeconfig ファイル。
* Azure Stack HCI PowerShell モジュールに Azure Kubernetes Service がインストールされている。
* PowerShell 管理ウィンドウでこのドキュメントのコマンドを実行する。
* OS 固有のワークロードが適切なコンテナー ホストに配置されていることを確認する。 Linux と Windows のワーカー ノードが混在する Kubernetes クラスターがある場合は、ノード セレクター、またはテイントと容認のいずれかを使用できます。 詳細については、[ノード セレクター、テイントと容認の使用](adapt-apps-mixed-os-clusters.md)に関するページを参照してください。

## <a name="deploy-the-application"></a>アプリケーションをデプロイする

Kubernetes のマニフェスト ファイルでは、どのコンテナー イメージを実行するかなど、クラスターの望ましい状態を定義します。 この記事では、Windows Server コンテナー内で ASP.NET サンプル アプリケーションを実行するために必要なすべてのオブジェクトを作成するのにマニフェストを使用します。 このマニフェストには、ASP.NET サンプル アプリケーションの Kubernetes デプロイと、インターネットからアプリケーションにアクセスするための外部 Kubernetes サービスが含まれています。

ASP.NET サンプル アプリケーションは、.NET Framework のサンプルの一部として提供され、Windows Server コンテナー内で実行されます。 Azure Stack HCI 上の Azure Kubernetes Service では、Windows Server コンテナーが *Windows Server 2019* のイメージに基づいている必要があります。 

Kubernetes マニフェスト ファイルではまた、ノード セレクターを定義する必要があり、この定義では、Windows Server コンテナーを実行できるノード上でご利用の ASP.NET サンプル アプリケーションのポッドを実行するように AKS クラスターに指示します。

`sample.yaml` という名前のファイルを作成し、以下の YAML 定義をコピーします。 

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample
  labels:
    app: sample
spec:
  replicas: 1
  template:
    metadata:
      name: sample
      labels:
        app: sample
    spec:
      nodeSelector:
        "beta.kubernetes.io/os": windows
      containers:
      - name: sample
        image: mcr.microsoft.com/dotnet/framework/samples:aspnetapp
        resources:
          limits:
            cpu: 1
            memory: 800M
          requests:
            cpu: .1
            memory: 300M
        ports:
          - containerPort: 80
  selector:
    matchLabels:
      app: sample
---
apiVersion: v1
kind: Service
metadata:
  name: sample
spec:
  type: LoadBalancer
  ports:
  - protocol: TCP
    port: 80
  selector:
    app: sample
```

`kubectl apply` コマンドを使用してアプリケーションをデプロイし、ご利用の YAML マニフェストの名前を指定します。

```console
kubectl apply -f sample.yaml
```

次の出力例は、正常に作成されたデプロイおよびサービスを示しています。

```output
deployment.apps/sample created
service/sample created
```

## <a name="test-the-application"></a>アプリケーションをテストする

アプリケーションが実行されると、Kubernetes サービスによってアプリケーション フロント エンドがインターネットに公開されます。 このプロセスが完了するまでに数分かかることがあります。 場合によっては、サービスのプロビジョニングに数分以上かかることがあります。 このような場合は、最大 10 分と考えてください。

進行状況を監視するには、`kubectl get service` コマンドを `--watch` 引数と一緒に使用します。

```PowerShell
kubectl get service sample --watch
```

最初に、"*サンプル*" サービスの *EXTERNAL-IP* が "*保留中*" として表示されます。

```output
NAME    TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
sample  LoadBalancer   10.0.37.27   <pending>     80:30572/TCP   6s
```

*EXTERNAL-IP* アドレスが "*保留中*" から実際のパブリック IP アドレスに変わったら、`CTRL-C` を使用して `kubectl` ウォッチ プロセスを停止します。 次の出力例は、サービスに割り当てられている有効なパブリック IP アドレスを示しています。

```output
NAME    TYPE           CLUSTER-IP   EXTERNAL-IP     PORT(S)        AGE
sample  LoadBalancer   10.0.37.27   52.179.23.131   80:30572/TCP   2m
```

サンプル アプリが動作していることを確認するには、Web ブラウザーを開いてサービスの外部 IP アドレスにアクセスします。

![ASP.NET サンプル アプリケーションを参照している画像](media/deploy-windows-application/asp-net-sample-app.png)

ページを読み込もうとしたときに接続タイムアウトが発生した場合は、`kubectl get pods --watch` コマンドを使用してサンプル アプリの準備ができているかどうかを確認します。 Windows コンテナーが起動する前に、外部 IP アドレスを使用できる場合があります。

## <a name="scale-application-pods"></a>アプリケーション ポッドをスケーリングする

アプリケーション フロントエンドの 1 つのレプリカを作成しました。 ご利用のクラスターに存在するポッドの数と状態を確認するには、次のように `kubectl get` コマンドを使用します。

```console
kubectl get pods -n default
```

"*サンプル*" デプロイのポッドの数を変更するには、`kubectl scale` コマンドを使用します。 次の例では、フロントエンド ポッドの数を *3* に増やしています。

```console
kubectl scale --replicas=3 deployment/sample
```

`kubectl get pods` を再度実行して、追加のポッドが作成されたことを確認します。 しばらくすると、追加したポッドがクラスターで利用できる状態になります。

```console
kubectl get pods -n default
```

## <a name="next-steps"></a>次の手順

* [Azure Monitor を使用してクラスターとアプリケーションを監視します](/azure/azure-monitor/insights/container-insights-enable-arc-enabled-clusters)。
* [Kubernetes クラスターで永続ボリュームを使用します](persistent-volume.md)。
