---
title: Windows PowerShell を使用して Azure Stack HCI 上に Kubernetes クラスターを作成するクイックスタート
description: Windows PowerShell を使用して Azure Stack HCI 上に Kubernetes クラスターを作成する方法について説明します
author: jessicaguan
ms.topic: quickstart
ms.date: 09/22/2020
ms.author: jeguan
ms.openlocfilehash: 35d527b56a2429676d343ba8098fc6821835fb00
ms.sourcegitcommit: dabbe44c3208fbf989b7615301833929f50390ff
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/22/2020
ms.locfileid: "90949390"
---
# <a name="quickstart-create-kubernetes-clusters-on-azure-stack-hci-using-windows-powershell"></a>クイック スタート:Windows PowerShell を使用して Azure Stack HCI 上に Kubernetes クラスターを作成する

> 適用対象:Azure Stack HCI

このクイックスタートでは、Windows PowerShell を使用して Azure Stack HCI 上で Kubernetes クラスターを作成する方法について説明します。 代わりに Windows Admin Center を使用するには、「[Windows Admin Center を使用して Azure Stack HCI 上で Azure Kubernetes Service を設定する](setup.md)」を参照してください。

## <a name="before-you-begin"></a>開始する前に

開始する前に次の点を確認します。

- 2 から 4 ノードの Azure Stack HCI クラスターまたは単一ノードの Azure Stack HCI がある。 **2 から 4 ノードの Azure Stack HCI クラスターを使用することをお勧めします。** それ以外の場合は、[こちら](./system-requirements.md)の作成方法の手順に従ってください。
- Azure Stack Kubernetes ホストが設定されている。 それ以外の場合は、[こちら](./setup-powershell.md)の設定方法の手順に従ってください。

## <a name="step-1-create-a-kubernetes-cluster"></a>手順 1:Kubernetes クラスターを作成する

Azure Kubernetes Service ホストをインストールすると、Kubernetes クラスターをデプロイする準備は完了です。

管理者として PowerShell を開き、次のコマンドを実行します。

   ```powershell
   New-AksHciCluster -clusterName
                    [-kubernetesVersion]
                    [-controlPlaneNodeCount]
                    [-linuxNodeCount]
                    [-windowsNodeCount]
                    [-controlPlaneVmSize]
                    [-loadBalancerVmSize]
                    [-linuxNodeVmSize]
                    [-windowsNodeVmSize]
   ```

### <a name="required-parameters"></a>必須のパラメーター

`-clusterName`

Kubernetes クラスターの英数字名。

### <a name="optional-parameters"></a>省略可能なパラメーター

`-kubernetesVersion`

デプロイする Kubernetes のバージョン。 既定値は v1.18.6 です。 使用できるバージョンの一覧を取得するには、`Get-AksHciKubernetesVersion` を実行します。

`-controlPlaneNodeCount`

コントロール プレーン内のノード数。 既定値は 1 です。

`-linuxNodeCount`

Kubernetes クラスター内の Linux ノードの数。 既定値は 1 です。

`-windowsNodeCount`

Kubernetes クラスター内の Windows ノードの数。 既定値は 0 です。

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
Set-AksHciClusterNodeCount –clusterName
                           -controlPlaneNodeCount
```

ワーカー ノードをスケーリングするには、次のコマンドを実行します。

```powershell
Set-AksHciClusterNodeCount –clusterName
                           -linuxNodeCount
                           -windowsNodeCount
```

コントロール プレーン ノードとワーカー ノードは、個別にスケーリングする必要があります。

## <a name="step-3-upgrade-kubernetes-version"></a>手順 3:Kubernetes バージョンをアップグレードする

現在実行している Kubernetes バージョンを確認するには、次のコマンドを実行します。

```powershell
Get-AksHciKubernetesVersion
```

次の Kubernetes バージョンにアップグレードするには、次のコマンドを実行します。

```powershell
Update-AksHciCluster -clusterName
```

Windows ノードを使用する場合、最低限必要なバージョンは v1.1.8.6 です。

## <a name="step-4-access-your-clusters-using-kubectl"></a>手順 4:kubectl を使用してクラスターにアクセスする

kubectl を使用して Azure Kubernetes Service ホストまたは Kubernetes クラスターにアクセスするには、次のコマンドを実行します。 これにより、指定したクラスターの kubeconfig ファイルが kubectl の既定の kubeconfig ファイルとして使用されます。

```powershell
Set-AksHciKubeConfig -clusterName
```

## <a name="delete-a-kubernetes-cluster"></a>Kubernetes クラスターを削除する

Kubernetes クラスターを削除する必要がある場合は、次のコマンドを実行します。

```powershell
Remove-AksHciCluster -clusterName
```

## <a name="get-logs"></a>ログを取得する

すべてのポッドからログを取得するには、次のコマンドを実行します。 このコマンドを実行すると、パス `C:\wssd\akshcilogs` に、`akshcilogs` という名前の出力 zip 形式フォルダーが作成されます。

```powershell
Get-AksHciLogs
```

このクイックスタートでは、PowerShell を使用して Kubernetes クラスターを作成、スケーリング、アップグレードする方法について説明しました。

## <a name="next-steps"></a>次の手順

- [クラスターを Azure Arc for Kubernetes に接続します](./connect-to-arc.md)。
- [Kubernetes クラスターに Linux アプリケーションをデプロイします](./deploy-linux-application.md)。
- [Kubernetes クラスターに Windows アプリケーションをデプロイします](./deploy-windows-application.md)。
