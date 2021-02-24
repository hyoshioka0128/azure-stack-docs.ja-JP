---
title: Windows PowerShell を使用して Azure Stack HCI 上に Kubernetes クラスターを作成するクイックスタート
description: Windows PowerShell を使用して Azure Stack HCI 上に Kubernetes クラスターを作成する方法について説明します
author: jessicaguan
ms.topic: quickstart
ms.date: 12/02/2020
ms.author: jeguan
ms.openlocfilehash: 6a215308e5eab1fa7991f912f1cd4aaff2391f5e
ms.sourcegitcommit: 0efffe1d04a54062a26d5c6ce31a417f511b9dbf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2020
ms.locfileid: "96612439"
---
# <a name="quickstart-create-kubernetes-clusters-on-azure-stack-hci-using-windows-powershell"></a>クイック スタート:Windows PowerShell を使用して Azure Stack HCI 上に Kubernetes クラスターを作成する

> 適用対象:AKS on Azure Stack HCI、AKS runtime on Windows Server 2019 Datacenter

このクイックスタートでは、Windows PowerShell を使用して Azure Stack HCI 上で Kubernetes クラスターを作成する方法について説明します。 次に、Kubernetes クラスターをスケーリングし、Kubernetes バージョンのクラスターをアップグレードする方法について説明します。 代わりに Windows Admin Center を使用するには、「[Windows Admin Center を使用して Azure Stack HCI 上で Azure Kubernetes Service を設定する](setup.md)」を参照してください。

## <a name="before-you-begin"></a>開始する前に

 - Azure Stack Kubernetes ホストが設定されていることを確認します。 設定されていない場合は、「[クイックスタート: PowerShell を使用して Azure Stack HCI で Azure Kubernetes Service ホストを設定する](./setup-powershell.md)」を参照してください。
 - 最新の Aks-Hci PowerShell モジュールがインストールされていることを確認します。 インストールされていない場合は、[AksHci PowerShell モジュールをダウンロードしてインストール](./setup-powershell.md#step-1-download-and-install-the-akshci-powershell-module)します。

## <a name="step-1-create-a-kubernetes-cluster"></a>手順 1:Kubernetes クラスターを作成する

Azure Kubernetes Service ホストをインストールすると、Kubernetes クラスターをデプロイする準備は完了です。

管理者として PowerShell を開き、次のコマンドを実行します。

   ```powershell
   New-AksHciCluster -clusterName <String>
                    [-kubernetesVersion <String>]
                    [-controlPlaneNodeCount <int>]
                    [-linuxNodeCount <int>]
                    [-windowsNodeCount <int>]
                    [-controlPlaneVmSize <VmSize>]
                    [-loadBalancerVmSize <VmSize>]
                    [-linuxNodeVmSize <VmSize>]
                    [-windowsNodeVmSize <VmSize>]
   ```

### <a name="example"></a>例

   ```powershell
   New-AksHciCluster -clusterName mynewcluster -kubernetesVersion v1.18.8 -controlPlaneNodeCount 1 -linuxNodeCount 1 -windowsNodeCount 0 
   ```

### <a name="required-parameters"></a>必須のパラメーター

`-clusterName`

Kubernetes クラスターの英数字名。

### <a name="optional-parameters"></a>省略可能なパラメーター

`-kubernetesVersion`

デプロイする Kubernetes のバージョン。 既定値は v1.18.8 です。 使用できるバージョンの一覧を取得するには、`Get-AksHciKubernetesVersion` を実行します。

`-controlPlaneNodeCount`

コントロール プレーン内のノード数。 既定値は 1 です。

`-linuxNodeCount`

Kubernetes クラスター内の Linux ノードの数。 既定値は 1 です。

`-windowsNodeCount`

Kubernetes クラスター内の Windows ノードの数。 既定値は 0 です。 Windows ノードをデプロイできるのは、Kubernetes v1.18.8 を実行している場合のみです。

`-controlPlaneVmSize`

コントロール プレーン VM のサイズ。 既定値は Standard_A2_v2 です。 使用できる VM サイズの一覧を取得するには、`Get-AksHciVmSize` を実行します。

`-loadBalancerVmSize`

ロード バランサー VM のサイズ。 既定値は Standard_A2_V2 です。 使用できる VM サイズの一覧を取得するには、`Get-AksHciVmSize` を実行します。

`-linuxNodeVmSize`

Linux ノード VM のサイズ。 既定値は Standard_K8S3_v1 です。 使用できる VM サイズの一覧を取得するには、`Get-AksHciVmSize` を実行します。

`-windowsNodeVmSize`

Windows ノード VM のサイズ。 既定値は Standard_K8S3_v1 です。 使用できる VM サイズの一覧を取得するには、`Get-AksHciVmSize` を実行します。

### <a name="check-your-deployed-clusters"></a>デプロイされたクラスターを確認する

デプロイされている Azure Kubernetes Service ホストと Kubernetes クラスターの一覧を取得するには、次のコマンドを実行します。

```powershell
Get-AksHciCluster
```

## <a name="step-2-scale-a-kubernetes-cluster"></a>手順 2:Kubernetes クラスターをスケーリングする

クラスターをスケールアップまたはスケールダウンする必要がある場合は、コントロール プレーン ノード、Linux ワーカー ノード、または Windows ワーカー ノードの数を変更できます。

コントロール プレーン ノードをスケーリングするには、次のコマンドを実行します。

```powershell
Set-AksHciClusterNodeCount –clusterName <String>
                           -controlPlaneNodeCount <int>
```

ワーカー ノードをスケーリングするには、次のコマンドを実行します。

```powershell
Set-AksHciClusterNodeCount –clusterName <String>
                           -linuxNodeCount <int>
                           -windowsNodeCount <int>
```

コントロール プレーン ノードとワーカー ノードは、個別にスケーリングする必要があります。

### <a name="example"></a>例

```powershell
Set-AksHciClusterNodeCount –clusterName mynewcluster -controlPlaneNodeCount 3
```

```powershell
Set-AksHciClusterNodeCount –clusterName mynewcluster -linuxNodeCount 2 -windowsNodeCount 2 
```

## <a name="step-3-upgrade-kubernetes-version"></a>手順 3:Kubernetes バージョンをアップグレードする

使用可能な Kubernetes バージョンの一覧を表示するには、次のコマンドを実行します。

```powershell
Get-AksHciKubernetesVersion
```

次の Kubernetes バージョンに更新するには、次のコマンドを実行します。

```powershell
Update-AksHciCluster -clusterName <String>
                     [-patch]
```
すべての Kubernetes バージョンには、メジャー リリース、マイナー バージョン、およびパッチ バージョンがあります。 たとえば、v1.18.6 では、1 はメジャー リリース、18 はマイナー バージョン、6 はパッチ バージョンです。 時間の経過と共に、AKS-HCI では 1 つのメジャー リリース、3 つのマイナー リリース、およびマイナー リリースごとに 2 つのパッチがサポートされる予定で、サポート対象のバージョンは合計 6 つです。 ただし、このプレビュー リリースでは、合計 4 つのリリース - v1.16.10、v1.16.15、v1.17.11、v1.18.8 がサポートされています。 

`Update-AksHciCluster` の実行時にパラメーター `patch` が追加されると、このコマンドによってマイナー バージョンの次のパッチ バージョン (存在する場合) にアップグレードされます。 パラメーター `patch` を指定せずにコマンドを実行すると、既定のアップグレード エクスペリエンスは次のマイナー リリースになります。 わかりやすいように、次の表に、考えられるすべての更新エクスペリエンスを示します。

| 現在のリリース           | -patch なしでの Kubernetes 更新バージョン         | -patch による Kubernetes 更新バージョン
| ---------------------------- | ------------ | -------------------------------- |
| v1.16.10           |     v1.17.11      | v1.16.15
| v1.16.15            | v1.17.11 | インプレース アドオン アップグレード
| v1.17.11           |  v1.18.8          | インプレース アドオン アップグレード
| v1.18.8             | インプレース アドオン アップグレード   | インプレース アドオン アップグレード

インプレース アドオン アップグレードでは、AKS-HCI によって管理される CSI などのすべての Kubernetes アドオンが更新されます。 このアップグレードでは、ノードの OS バージョンは変更されません。 また、Kubernetes バージョンも変更されません。

### <a name="example---upgrade-kubernetes-version-to-the-next-minor-version"></a>例 - Kubernetes バージョンを次のマイナー バージョンにアップグレードする

```powershell
Update-AksHciCluster -clusterName mynewcluster
```

### <a name="example---upgrade-kubernetes-version-to-the-next-patch-version"></a>例 - Kubernetes バージョンを次のパッチ バージョンにアップグレードする

```powershell
Update-AksHciCluster -clusterName mynewcluster -patch
```


## <a name="step-4-access-your-clusters-using-kubectl"></a>手順 4:kubectl を使用してクラスターにアクセスする

kubectl を使用して Kubernetes クラスターにアクセスするには、次のコマンドを実行します。 これにより、指定したクラスターの kubeconfig ファイルが kubectl の既定の kubeconfig ファイルとして使用されます。

```powershell
Get-AksHciCredential -clusterName <String>
                     [-outputLocation <String>]
```

### <a name="example"></a>例

```powershell
Get-AksHciCredential -clusterName mynewcluster
```

### <a name="required-parameters"></a>必須のパラメーター

`clusterName`

クラスターの名前です。

### <a name="optional-parameters"></a>省略可能のパラメーター

`outputLocation`

kubeconfig をダウンロードする場所。 既定値は `%USERPROFILE%\.kube` です。

## <a name="delete-a-kubernetes-cluster"></a>Kubernetes クラスターを削除する

Kubernetes クラスターを削除する必要がある場合は、次のコマンドを実行します。

```powershell
Remove-AksHciCluster -clusterName
```

### <a name="example"></a>例

```powershell
Remove-AksHciCluster -clusterName mynewcluster
```

## <a name="get-logs"></a>ログを取得する

すべてのポッドからログを取得するには、次のコマンドを実行します。 このコマンドを実行すると、パス `C:\wssd\akshcilogs` に、`akshcilogs` という名前の出力 zip 形式フォルダーが作成されます。

```powershell
Get-AksHciLogs
```

このクイックスタートでは、PowerShell を使用して Kubernetes バージョンのクラスターを作成、スケーリング、アップグレードする方法について説明しました。

## <a name="next-steps"></a>次の手順

- [クラスターを Azure Arc for Kubernetes に接続します](./connect-to-arc.md)。
- [Kubernetes クラスターに Linux アプリケーションをデプロイします](./deploy-linux-application.md)。
- [Kubernetes クラスターに Windows アプリケーションをデプロイします](./deploy-windows-application.md)。
