---
title: Azure Stack HCI クラスターの AKS で永続ボリュームを使用する
description: Windows コンテナーで永続的ストレージを使用し、グループの管理サービス アカウント用に Windows ノードを準備する
author: v-susbo
ms.topic: how-to
ms.date: 12/02/2020
ms.author: v-susbo
ms.reviewer: ''
ms.openlocfilehash: 4baaf83c16b3fc290d9dc1339aaf432dc05d89b7
ms.sourcegitcommit: 0efffe1d04a54062a26d5c6ce31a417f511b9dbf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2020
ms.locfileid: "96612609"
---
# <a name="use-persistent-volume-in-an-aks-on-azure-stack-hci-cluster"></a>Azure Stack HCI クラスターの AKS で永続ボリュームを使用する

> 適用対象:Azure Stack HCI 上の AKS、Windows Server 2019 Datacenter 上の AKS ランタイム

永続ボリュームとは、Kubernetes ポッドで使用するためにプロビジョニングされているストレージの一部です。 永続ボリュームは、1 つまたは複数のポッドで使用でき、長期の保管用に向いています。 また、ポッドやノードのライフサイクルからは独立しています。 Windows ノードと Linux ノードの両方に対して PVC をプロビジョニングできますが、このセクションでは、永続ボリュームを作成する方法と、Windows アプリケーションでこのボリュームを使用する方法について説明します。 詳細については、[Kubernetes での永続ボリューム](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)に関する記事を参照してください。

## <a name="before-you-begin"></a>開始する前に

使用を開始するために必要なものは次のとおりです。

* Windows ワーカー ノードが少なくとも 1 つある Kubernetes クラスター。
* Kubernetes クラスターにアクセスするための kubeconfig ファイル。

## <a name="create-a-persistent-volume-claim"></a>永続ボリューム要求の作成

永続ボリューム要求を使用して、ストレージ クラスに基づいてストレージを自動的にプロビジョニングします。  ボリューム要求を作成するには、まず `pvc-akshci-csi.yaml` というファイルを作成し、次の YAML 定義にコピーします。 要求では、 *ReadWriteOnce* アクセスがあるサイズ 10 GB のディスクを要求します。  *default* ストレージ クラスは、ストレージ クラス (vhdx) として指定されています。  

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
 name: pvc-akshci-csi
spec:
 accessModes:
 - ReadWriteOnce
 resources:
  requests:
   storage: 10Gi
```
Azure Stack HCI クラスター内のいずれかのサーバーの管理 PowerShell セッションで、以下のコマンドを実行して、ボリュームを作成します ( [Enter-PSSession](/powershell/module/microsoft.powershell.core/enter-pssession) などのメソッドまたはリモート デスクトップを使用してサーバーに接続します)。 

```
kubectl create -f pvc-akshci-csi.yaml 
```
次の出力に、永続ボリューム要求が正常に作成されたことが示されます。

**出力:**
```
persistentvolumeclaim/pvc-akshci-csi created
```

## <a name="use-persistent-volume"></a>永続ボリュームの使用

永続ボリュームを使用するには、winwebserver.yaml という名前のファイルを作成し、次の YAML 定義にコピーします。  次に、永続ボリューム要求と vhdx にアクセスできるポッドを作成します。 

以下の yaml 定義では、*mountPath* が、コンテナー内にボリュームをマウントするパスです。 ポッドの作成が正常に完了すると、*C:\\* 内にサブディレクトリ *mnt* が、*mnt* 内にサブディレクトリ *akshciscsi* が作成されたと表示されます。


```yaml
apiVersion: apps/v1 
kind: Deployment 
metadata: 
  labels: 
    app: win-webserver 
  name: win-webserver 
spec: 
  replicas: 1 
  selector: 
    matchLabels: 
      app: win-webserver 
  template: 
    metadata: 
      labels: 
        app: win-webserver 
      name: win-webserver 
    spec: 
     containers: 
      - name: windowswebserver 
        image: mcr.microsoft.com/windows/servercore/iis:windowsservercore-ltsc2019 
        ports:  
          - containerPort: 80    
        volumeMounts: 
            - name: akshciscsi 
              mountPath: "/mnt/akshciscsi" 
     volumes: 
        - name: akshciscsi 
          persistentVolumeClaim: 
            claimName:  pvc-akshci-csi 
     nodeSelector: 
      kubernetes.io/os: windows 
```

上の yaml 定義を使用してポッドを作成するには、次を実行します。
```
kubectl create -f winwebserver.yaml 
```

ポッドが実行されていることを確認するには、次のコマンドを実行します。 イメージをプルするのに時間がかかるため、ポッドが実行中の状態になるまで数分待機します。 
```
kubectl get pods -o wide 
```
ポッドが実行中になったら、次のコマンドを実行してポッドの状態を表示します。 
```
kubectl.exe describe pod %podName% 
```

ボリュームがポッドにマウントされていることを確認するには、次のコマンドを実行します。
```
kubectl exec -it %podname% cmd.exe 
```

## <a name="delete-a-persistent-volume-claim"></a>永続ボリューム要求の削除

永続ボリューム要求を削除する前に、次を実行してアプリのデプロイを削除する必要があります。
```
kubectl delete deployments win-webserver
```

その後で、次を実行して永続ボリューム要求を削除できます。
```
kubectl delete PersistentVolumeClaim pvc-akshci-csi
```

## <a name="next-steps"></a>次の手順
- [Kubernetes クラスターに Windows アプリケーションをデプロイします](./deploy-windows-application.md)。
- [Kubernetes クラスターを Azure Arc for Kubernetes に接続します](./connect-to-arc.md)。
