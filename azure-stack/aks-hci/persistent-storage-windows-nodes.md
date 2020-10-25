---
title: Windows コンテナーで永続的ストレージを使用する
description: Windows コンテナーで永続的ストレージを使用し、グループの管理サービス アカウント用に Windows ノードを準備する
author: abha
ms.topic: how-to
ms.date: 09/21/2020
ms.author: abha
ms.reviewer: ''
ms.openlocfilehash: 91f7249beb34e5afee808d299df48611a5ce26bb
ms.sourcegitcommit: 868887e4b13b1572f15004a9db2c334e60d8add2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/06/2020
ms.locfileid: "91778124"
---
# <a name="use-persistent-storage-in-a-windows-container-and-prepare-windows-nodes-for-group-managed-service-accounts"></a>Windows コンテナーで永続的ストレージを使用し、グループの管理サービス アカウント用に Windows ノードを準備する

永続ボリュームとは、Kubernetes ポッドで使用するためにプロビジョニングされているストレージの一部です。 永続ボリュームは、1 つまたは複数のポッドで使用でき、長期の保管用に向いています。 また、ポッドやノードのライフサイクルからは独立しています。  このセクションでは、永続ボリュームを作成する方法と、Windows アプリケーションでこのボリュームを使用する方法について説明します。

## <a name="before-you-begin"></a>開始する前に

使用を開始するために必要なものは次のとおりです。

* Windows ワーカー ノードが少なくとも 1 つある Kubernetes クラスター。
* Kubernetes クラスターにアクセスするための kubeconfig ファイル。


## <a name="create-a-persistent-volume-claim"></a>永続ボリューム要求の作成

永続ボリューム要求を使用して、ストレージ クラスに基づいてストレージを自動的にプロビジョニングします。 ボリューム要求を作成するには、まず `pvc-akshci-csi.yaml` というファイルを作成し、次の YAML 定義にコピーします。 要求では、 *ReadWriteOnce* アクセスがあるサイズ 10 GB のディスクを要求します。  *default* ストレージ クラスは、ストレージ クラス (vhdx) として指定されています。  

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


```PowerShell
kubectl create -f pvc-akshci-csi.yaml 
```
次の出力に、永続ボリューム要求が正常に作成されたことが示されます。

**出力:**
```PowerShell
persistentvolumeclaim/pvc-akshci-csi created
```

## <a name="use-persistent-volume"></a>永続ボリュームの使用

永続ボリュームを使用するには、winwebserver.yaml という名前のファイルを作成し、次の YAML 定義にコピーします。 次に、永続ボリューム要求と vhdx にアクセスできるポッドを作成します。 

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

```PowerShell
Kubectl create -f winwebserver.yaml 
```

ポッドが実行されていることを確認するには、次のコマンドを実行します。 イメージをプルするのに時間がかかるため、ポッドが実行中の状態になるまで数分待機します。

```PowerShell
kubectl get pods -o wide 
```
ポッドが実行中になったら、次のコマンドを実行してポッドの状態を表示します。 

```PowerShell
kubectl.exe describe pod %podName% 
```

ボリュームがポッドにマウントされていることを確認するには、次のコマンドを実行します。

```PowerShell
kubectl exec -it %podname% cmd.exe 
```

## <a name="delete-a-persistent-volume-claim"></a>永続ボリューム要求の削除

永続ボリューム要求を削除する前に、次を実行してアプリのデプロイを削除する必要があります。

```PowerShell
kubectl.exe delete deployments win-webserver
```

その後で、次を実行して永続ボリューム要求を削除できます。

```PowerShell
kubectl.exe delete PersistentVolumeClaim pvc-akshci-csi
```

## <a name="prepare-windows-nodes-for-group-managed-service-account-support-on-windows-nodes"></a>Windows ノードでグループの管理サービス アカウントをサポートするための Windows ノードの準備

グループの管理サービス アカウントは、パスワードの自動管理、簡略化されたサービス プリンシパル名 (SPN) の管理、および複数のサーバー間で管理を他の管理者に委任する機能を提供する、特定の種類の Active Directory アカウントです。 Windows ノードで実行されるポッドとコンテナーに対してグループの管理サービス アカウント (gMSA) を構成するには、最初に、Windows ノードを Active Directory ドメインに参加させる必要があります。

グループの管理サービス アカウントのサポートを有効にするには、Kubernetes クラスター名が 4 文字未満である必要があります。 これは、ドメインに参加したサーバーの名前でサポートされる最大長が 15 文字であり、Azure Stack HCI Kubernetes クラスター上の AKS の、ワーカー ノードについての名前付け規則により、いくつかの定義済み文字がノード名に追加されるためです。

Windows ワーカー ノードをドメインに参加させるには、`kubectl get` を実行して `EXTERNAL-IP` の値を書き留めることにより、Windows ワーカー ノードにログインします。

```PowerShell
kubectl get nodes -o wide
``` 

次に `ssh Administrator@ip` を使用して、SSH でノードに接続できます。 

Windows ワーカー ノードに正常にログインしたら、次の PowerShell コマンドを実行してノードをドメインに参加させます。 **ドメイン管理者アカウント**の資格情報を入力するよう求められます。 特定のドメインにコンピューターを参加させる権限が付与されている、昇格されたユーザー資格情報を使用することもできます。 その後、Windows ワーカー ノードを再起動する必要があります。

```PowerShell
add-computer --domainame "YourDomainName" -restart
```

すべての Windows ワーカー ノードをドメインに参加させたら、[gMSA の構成](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa)に関するページに詳述されている手順に従って、Kubernetes クラスターに Kubernetes gMSA カスタム リソース定義と Webhook を適用します。

gMSA を使用する Windows コンテナーの詳細については、[Windows コンテナーと gMSA](/virtualization/windowscontainers/manage-containers/manage-serviceaccounts) に関するページを参照してください。 

## <a name="next-steps"></a>次の手順
- [Kubernetes クラスターに Windows アプリケーションをデプロイします](./deploy-windows-application.md)。
- [Kubernetes クラスターを Azure Arc for Kubernetes に接続します](./connect-to-arc.md)。
