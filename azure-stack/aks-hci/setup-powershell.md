---
title: Windows PowerShell を使用して Azure Stack HCI で Azure Kubernetes Service ホストを設定するためのクイックスタート
description: Windows PowerShell を使用して Azure Stack HCI で Azure Kubernetes Service ホストを設定する方法について説明します
author: jessicaguan
ms.topic: quickstart
ms.date: 09/23/2020
ms.author: jeguan
ms.openlocfilehash: b4b128c5d51d7f916e0936102224283dd77a971d
ms.sourcegitcommit: 849be7ebd02a1e54e8d0ec59736c9917c67e309e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/24/2020
ms.locfileid: "91134663"
---
# <a name="quickstart-set-up-an-azure-kubernetes-service-host-on-azure-stack-hci-using-powershell"></a>クイック スタート:PowerShell を使用して Azure Stack HCI で Azure Kubernetes Service ホストを設定する

> 適用対象:Azure Stack HCI

このクイックスタートでは、PowerShell を使用して Azure Stack HCI で Azure Kubernetes Service ホストを設定する方法について説明します。 Windows 管理センターを使用する場合は、[Windows 管理センターを使用した設定](setup.md)に関するページを参照してください。

## <a name="before-you-begin"></a>開始する前に

開始する前に、2 から 4 ノードの Azure Stack HCI クラスターまたは単一ノードの Azure Stack HCI があることを確認してください。 **2 から 4 ノードの Azure Stack HCI クラスターを使用することをお勧めします。** それ以外の場合は、[こちら](./system-requirements.md)の手順に従ってください。

また、AksHci PowerShell モジュールがインストールされていることを確認する必要もあります。 [こちら](https://aka.ms/AKS-HCI-Evaluate)にあるダウンロード パッケージでは、zip ファイル内にこのモジュールが含まれます。 必ず正しい場所 (`%systemdrive%\program files\windowspowershell\modules`) で zip ファイルを抽出し、PowerShell 管理ウィンドウで次のコマンドを実行してください。

   ```powershell
   Import-Module AksHci
   ```

上記のコマンドを実行した後、すべての PowerShell ウィンドウを閉じ、管理セッションを再度開いて、次の手順でコマンドを実行します。

## <a name="step-1-prepare-your-machines-for-deployment"></a>手順 1:デプロイ用にマシンを準備する

まず、すべての物理ノードでチェックを実行して、Azure Stack HCI 上に Azure Kubernetes Service をインストールするためのすべての要件が満たされているかどうかを確認します。

管理者として PowerShell を開き、次のコマンドを実行します。

   ```powershell
   Initialize-AksHciNode
   ```

チェックが完了すると、緑色のテキストで "Done" と表示されます。

## <a name="step-2-configure-your-deployment"></a>手順 2:デプロイを構成する

Azure Kubernetes Service ホストの構成を設定します。 **2 から 4 ノードの Azure Stack HCI クラスターの場合、`-deploymentType`、`wssdImageDir`、および `cloudConfigLocation` パラメーターに `MultiNode` を指定する必要があります。** 単一ノードの Azure Stack HCI クラスターの場合、すべてのパラメーターは省略可能で、既定値に設定されます。 ただし、最適なパフォーマンスを得るには、**2 から 4 ノードの Azure Stack HCI クラスターのデプロイを使用することをお勧めします。**

次のコマンドを使用して、デプロイを構成します。

   ```powershell
   Set-AksHciConfig [-deploymentType {SingleNode, MultiNode}]
                    [-wssdImageDir]
                    [-cloudConfigLocation]
                    [-nodeConfigLocation]
                    [-vnetName]
                    [-controlPlaneVmSize]
                    [-loadBalancerVmSize]
                    [-sshPublicKey]
                    [-vipPoolStartIp]
                    [-vipPoolEndIp]
                    [-macPoolStart]
                    [-macPoolEnd]
                    [-wssdDir]
                    [-akshciVersion]
                    [-vnetType]
                    [-nodeAgentPort]
                    [-nodeAgentAuthorizerPort]
                    [-clusterRoleName]
                    [-skipHostLimitChecks]
                    [-insecure]
                    [-skipUpdates]
                    [-forceDnsReplication]

   ```

### <a name="optional-parameters"></a>省略可能なパラメーター

`-deploymentType`

配置タイプ。 指定可能な値:SingleNode、MultiNode。 既定値は SingleNode です。

`-wssdImageDir`

Azure Stack HCI 上の Azure Kubernetes Service によって VHD イメージが格納されるディレクトリへのパス。 単一ノードのデプロイの場合、既定値は `%systemdrive%\wssdimagestore` です。 "*複数ノードのデプロイの場合、このパラメーターを指定する必要があります*"。 パスは、共有ストレージ パス ( `C:\ClusterStorage\Volume2\ImageStore`  など) または SMB 共有 ( `\\FileShare\ImageStore` など) を指す必要があります。

`-cloudConfigLocation`

クラウド エージェントによって構成が格納される場所。 単一ノードのデプロイの場合、既定値は `%systemdrive%\wssdimagestore` です。 この場所は、上記の  `-wssdImageDir` のパスと同じにすることができます。 "*複数ノードのデプロイの場合、このパラメーターを指定する必要があります*"。

`-nodeConfigLocation`

ノード エージェントによって構成が格納される場所。 これはローカル パスである必要があります。

`-vnetName`

仮想マシンの接続先となる仮想スイッチの名前。 既定値は "外部" 名です。 スイッチが存在しない場合は作成されます。  

`-controlPlaneVmSize`

コントロール プレーン用に作成する VM のサイズ。 使用できる VM サイズの一覧を取得するには、`Get-AksHciVmSize` を実行します。

`-loadBalancerVmSize`

ロード バランサー VM 用に作成する VM のサイズ。 使用できる VM サイズの一覧を取得するには、`Get-AksHciVmSize` を実行します。

`-sshPublicKey`

SSH 公開キー ファイルへのパス。 この公開キーを使用すると、Azure Stack HCI デプロイ上の Azure Kubernetes Service によって作成された任意の VM にログインできます。 キーが指定されていない場合は、 `%systemdrive%\Users\<username>\.ssh\id_rsa.pub` 下を確認します。 ファイルが存在しない場合は、上の場所で SSH キー ペアが生成されて使用されます。  

`-vipPoolStartIp`

デプロイに VIP プールを使用している場合、このパラメーターでプールのネットワーク開始を指定します。 既定値は none です。

`-vipPoolEndIp`

デプロイに VIP プールを使用している場合、このパラメーターでプールのネットワーク終了を指定します。 既定値は none です。

`-macPoolStart`

これは、Azure Kubernetes Service ホスト VM に使用する MAC プールの MAC アドレスの先頭を指定するために使用されます。 MAC アドレスの構文では、先頭のバイトの最下位ビットが常に 0 である必要があります。また、先頭のバイトは常に偶数 (つまり、00、02、04、06...) である必要があります。一般的な MAC アドレスは、02:1E:2B:78:00:00 のようになります。 既定値は none です。

`-macPoolEnd`

これは、Azure Kubernetes Service ホスト VM に使用する MAC プールの MAC アドレスの末尾を指定するために使用されます。 MAC アドレスの構文では、先頭のバイトの最下位ビットが常に 0 である必要があります。また、先頭のバイトは常に偶数 (つまり、00、02、04、06...) である必要があります。`-macPoolEnd` として渡されたアドレスの先頭のバイトは、`-macPoolStart` として渡されたアドレスの先頭のバイトと同じである必要があります。 既定値は none です。

`-wssdDir`

これは、小さいファイルを格納するためにモジュールで使用する作業ディレクトリです。 既定値は `%PROGRAMFILES%\AksHci`  であり、ほとんどのデプロイでは変更できません。  

`-akshciVersion`

デプロイする Azure Stack HCI 上の Azure Kubernetes Service のバージョン。 既定値は最新バージョンです。

`-vnetType`

接続または作成する仮想スイッチの種類。 既定値は "外部" スイッチの種類です。

`-nodeAgentPort`

ノード エージェントでリッスンする必要がある TCP/IP ポート番号。 既定値は 45000 です。  

`-nodeAgentAuthorizerPort`

ノード エージェントで承認ポートに使用する必要がある TCP/IP ポート番号。 既定値は 45001 です。  

`-clusterRoleName`

これにより、クラスター内の汎用サービスとしてクラウド エージェントを作成するときに使用する名前が指定されます。 これは、既定では、プレフィックス ca- と GUID サフィックスを持つ一意の名前になります (例: "ca-9e6eb299-bc0b-4f00-9fd7-942843820c26")

`-skipHostLimitChecks`

デプロイを続行する前に、メモリとディスク領域が使用可能であることを確認するために実行するすべてのチェックをスキップするよう、スクリプトに要求します。

`-insecure`

セキュリティ保護されていないモード (接続が TLS によって保護されていない) で、クラウド エージェントやノード エージェントなどの Azure Stack HCI コンポーネント上に Azure Kubernetes Service をデプロイします。  運用環境では、セキュリティ保護されていないモードを使用しないことをお勧めします。

`-skipUpdates`

利用可能な更新プログラムをスキップする場合は、このフラグを使用します。

`-forceDnsReplication`

一部のシステムでは、DNS レプリケーションに最大 1 時間かかることがあります。 これにより、デプロイの速度が低下します。 この問題が発生した場合は、Install-AksHci がループに入っていることがわかります。 この問題を解決するには、このフラグを使用してみてください。 `-forceDnsReplication` フラグは、保証されている修正プログラムではありません。 フラグの背後にあるロジックが失敗した場合、エラーは非表示になり、フラグが指定されなかったかのようにコマンドが実行されます。

### <a name="reset-the-azure-kubernetes-service-on-azure-stack-hci-configuration"></a>Azure Stack HCI 構成で Azure Kubernetes Service をリセットする

Azure Stack HCI 構成で Azure Kubernetes Service をリセットするには、次のコマンドを実行します。 このコマンドを単独で実行すると、構成が既定値にリセットされます。

```powershell
Set-AksHciConfig
```

## <a name="step-3-start-a-new-deployment"></a>手順 3:新しいデプロイを開始する

デプロイを構成したら、デプロイを開始する必要があります。 これにより、Azure Stack HCI エージェントまたはサービスおよび Azure Kubernetes Service ホスト上に Azure Kubernetes Service がインストールされます。

デプロイを開始するには、次のコマンドを実行します。

```powershell
Install-AksHci
```

### <a name="check-your-deployed-clusters"></a>デプロイされたクラスターを確認する

デプロイされた Azure Kubernetes Service ホストの一覧を取得するには、次のコマンドを実行します。 また、デプロイ後に同じコマンドを使用して Kubernetes クラスターを取得することもできます。

```powershell
Get-AksHciCluster
```

## <a name="step-4-access-your-clusters-using-kubectl"></a>手順 4:kubectl を使用してクラスターにアクセスする

kubectl を使用して Azure Kubernetes Service ホストまたは Kubernetes クラスターにアクセスするには、次のコマンドを実行します。 これにより、指定したクラスターの kubeconfig ファイルが kubectl の既定の kubeconfig ファイルとして使用されます。

```powershell
Set-AksHciKubeConfig -clusterName
```

## <a name="get-logs"></a>ログを取得する

すべてのポッドからログを取得するには、次のコマンドを実行します。 このコマンドを実行すると、パス `C:\wssd\akshcilogs` に、`akshcilogs` という名前の出力 zip 形式フォルダーが作成されます。

```powershell
Get-AksHciLogs
```

## <a name="reinstall-azure-kubernetes-service-on-azure-stack-hci"></a>Azure Stack HCI 上に Azure Kubernetes Service を再インストールする

Azure Stack HCI 上に Azure Kubernetes Service を再インストールすると、すべての Kubernetes クラスター (ある場合)、および Azure Kubernetes Service ホストが削除されます。 また、ノードから Azure Stack HCI エージェントおよびサービス上の Azure Kubernetes Service もアンインストールされます。 その後、ホストが再作成されるまで、元のインストール プロセスのステップが再実行されます。 `Set-AksHciConfig` を使用して構成した Azure Stack HCI 構成上の Azure Kubernetes Service と、ダウンロードした VHDX イメージは保持されます。

Azure Stack HCI 上に Azure Kubernetes Service を再インストールするには、次のコマンドを実行します。

```powershell
Restart-AksHci
```

## <a name="remove-azure-kubernetes-service-on-azure-stack-hci"></a>Azure Stack HCI で Azure Kubernetes Service を削除する

Azure Stack HCI で Azure Kubernetes Service を削除するには、次のコマンドを実行します。

```powershell
Uninstall-AksHci
```

## <a name="next-steps"></a>次の手順

- [アプリケーションの Kubernetes クラスターを作成する](create-kubernetes-cluster-powershell.md)
