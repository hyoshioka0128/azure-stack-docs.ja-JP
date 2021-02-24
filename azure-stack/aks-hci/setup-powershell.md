---
title: Windows PowerShell を使用して Azure Stack HCI で Azure Kubernetes Service ホストを設定するためのクイックスタート
description: Windows PowerShell を使用して Azure Stack HCI で Azure Kubernetes Service ホストを設定する方法について説明します
author: jessicaguan
ms.topic: quickstart
ms.date: 12/02/2020
ms.author: jeguan
ms.openlocfilehash: 4211ec50ef0ea24ffb55f14791101c5d266ede2e
ms.sourcegitcommit: 0efffe1d04a54062a26d5c6ce31a417f511b9dbf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2020
ms.locfileid: "96612558"
---
# <a name="quickstart-set-up-an-azure-kubernetes-service-host-on-azure-stack-hci-using-powershell"></a>クイック スタート:PowerShell を使用して Azure Stack HCI で Azure Kubernetes Service ホストを設定する

> 適用対象:Azure Stack HCI、Windows Server 2019 Datacenter

このクイックスタートでは、PowerShell を使用して Azure Kubernetes Service ホストを設定する方法について説明します。 Windows Admin Center を使用する場合は、[Windows Admin Center を使用した設定](setup.md)に関するページを参照してください。

## <a name="before-you-begin"></a>開始する前に

以下のものがそろっていることを確認してください。
 - 2 から 4 ノードの Azure Stack HCI クラスター
 - Windows Server 2019 Datacenter フェールオーバー クラスター
 - 単一ノードの Windows Server 2019 Datacenter 
 
作業を開始する前に、[システム要件](.\system-requirements.md)に関するページのすべての前提条件を満たしていることを確認してください。 
**2 から 4 ノードの Azure Stack HCI クラスターを使用することをお勧めします。** 上記のいずれもない場合は、[Azure Stack HCI の登録ページ](https://azure.microsoft.com/products/azure-stack/hci/hci-download/)の手順に従ってください。    

   > [!IMPORTANT]
   > Azure Kubernetes Service on Azure Stack HCI を削除する場合は、「[Azure Stack HCI で Azure Kubernetes Service を削除する](#remove-azure-kubernetes-service-on-azure-stack-hci)」を参照し、注意して手順に従ってください。 

## <a name="step-1-download-and-install-the-akshci-powershell-module"></a>手順 1:AksHci PowerShell モジュールをダウンロードしてインストールする

[Azure Kubernetes Service on Azure Stack HCI の登録ページ](https://aka.ms/AKS-HCI-Evaluate)から `AKS-HCI-Public-Preview-Dec-2020` をダウンロードします。 ZIP ファイル `AksHci.Powershell.zip` に、この PowerShell モジュールが含まれています。

PowerShell または Windows Admin Center を使用して Azure Kubernetes Service on Azure Stack HCI を既にインストール済みの場合、新しい PowerShell モジュールのインストールのフローは 2 つあります。
 - PowerShell モジュールのクリーン インストールを実行すると、クリーンなシステムから開始することになり、以前にデプロイしたワークロードが削除されます。 クリーン インストールを実行するには、手順 1.1 に進んでください。
 - システムとワークロードをそのまま維持する場合は、PowerShell モジュールをアップグレードします。 PowerShell モジュールをアップグレードするには、手順 1.2 に進みます。

### <a name="step-11-clean-install-of-the-akshci-powershell-module"></a>手順 1.1: AksHci PowerShell モジュールのクリーン インストール

続行する前に、次のコマンドを実行します。
   ```powershell
   Uninstall-AksHci
   ```

**すべての PowerShell ウィンドウを閉じます。** パス `%systemdrive%\program files\windowspowershell\modules` にある AksHci、AksHci.UI、MOC、MSK8sDownloadAgent の既存のディレクトリを削除します。 既存のディレクトリを削除したら、新しい ZIP ファイルの内容を展開できます。 必ず正しい場所 (`%systemdrive%\program files\windowspowershell\modules`) に ZIP ファイルを展開します。 次に、以下のコマンドを実行します。

   ```powershell
   Import-Module AksHci
   ```

すべての PowerShell ウィンドウを再度閉じ、管理セッションを開き直して、「手順 1.3 - アップグレードした PowerShell モジュールの検証」に進みます。

### <a name="step-12-upgrade-the-akshci-powershell-module"></a>手順 1.2: AksHci PowerShell モジュールのアップグレード

**すべての PowerShell ウィンドウを閉じます。** パス `%systemdrive%\program files\windowspowershell\modules` にある AksHci、AksHci.UI、MOC、MSK8sDownloadAgent の既存のディレクトリを削除します。 これらのディレクトリを削除したら、新しい ZIP ファイルの内容を展開できます。 必ず正しい場所 (`%systemdrive%\program files\windowspowershell\modules`) に ZIP ファイルを展開します。 次に、以下のコマンドを実行します。

   ```powershell
   Import-Module AksHci
   ```
  
上記のコマンドを実行した後、すべての PowerShell ウィンドウを閉じ、管理セッションを開き直して、以下に説明するように PowerShell モジュールのアップグレードを検証してから、このドキュメントで後述するとおり `Update-AksHci` コマンドを実行します。

## <a name="step-13-validate-upgraded-powershell-module"></a>手順 1.3: アップグレードした PowerShell モジュールの検証

**すべての PowerShell ウィンドウを閉じ**、新しい管理セッションを開き直して、使用している PowerShell モジュールが最新バージョンかどうかを確認します。  

   ```powershell
   Get-Command -Module AksHci
   ```
 
**出力:**
```
CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Alias           Initialize-AksHciNode                              0.2.12     AksHci
Function        Get-AksHciCluster                                  0.2.12     AksHci
Function        Get-AksHciConfig                                   0.2.12     AksHci
Function        Get-AksHciCredential                               0.2.12     AksHci
Function        Get-AksHciKubernetesVersion                        0.2.12     AksHci
Function        Get-AksHciLogs                                     0.2.12     AksHci
Function        Get-AksHciUpdates                                  0.2.12     AksHci
Function        Get-AksHciVersion                                  0.2.12     AksHci
Function        Get-AksHciVmSize                                   0.2.12     AksHci
Function        Install-AksHci                                     0.2.12     AksHci
Function        Install-AksHciAdAuth                               0.2.12     AksHci
Function        Install-AksHciArcOnboarding                        0.2.12     AksHci
Function        New-AksHciCluster                                  0.2.12     AksHci
Function        Remove-AksHciCluster                               0.2.12     AksHci
Function        Restart-AksHci                                     0.2.12     AksHci
Function        Set-AksHciClusterNodeCount                         0.2.12     AksHci
Function        Set-AksHciConfig                                   0.2.12     AksHci
Function        Uninstall-AksHci                                   0.2.12     AksHci
Function        Uninstall-AksHciAdAuth                             0.2.12     AksHci
Function        Uninstall-AksHciArcOnboarding                      0.2.12     AksHci
Function        Update-AksHci                                      0.2.12     AksHci
Function        Update-AksHciCluster                               0.2.12     AksHci
```

## <a name="step-2-prepare-your-machines-for-deployment"></a>手順 2:デプロイ用にマシンを準備する

すべての物理ノードでチェックを実行して、Azure Stack HCI 上に Azure Kubernetes Service をインストールするためのすべての要件が満たされているかどうかを確認します。

管理者として PowerShell を開き、次のコマンドを実行します。

   ```powershell
   Initialize-AksHciNode
   ```

チェックが完了すると、緑色のテキストで "Done" と表示されます。

## <a name="step-3-configure-your-deployment"></a>手順 3:デプロイを構成する

Azure Kubernetes Service ホストの構成を設定します。 **2 から 4 ノードの Azure Stack HCI クラスターまたは Windows Server 2019 Datacenter フェールオーバー クラスターにデプロイする場合は、`imageDir` および `cloudConfigLocation` パラメーターを指定する必要があります。** 単一ノードの Windows Server 2019 Datacenter の場合、すべてのパラメーターは省略可能で、既定値に設定されます。 ただし、最適なパフォーマンスを得るには、**2 から 4 ノードの Azure Stack HCI クラスターのデプロイを使用することをお勧めします。**

次のコマンドを使用して、デプロイを構成します。

   ```powershell
   Set-AksHciConfig [-imageDir <String>]
                    [-cloudConfigLocation <String>]
                    [-nodeConfigLocation <String>]
                    [-vnetName <String>]
                    [-controlPlaneVmSize <VmSize>]
                    [-loadBalancerVmSize <VmSize>]
                    [-sshPublicKey <String>]
                    [-vipPoolStartIp <String>]
                    [-vipPoolEndIp <String>]
                    [-macPoolStart <String>]
                    [-macPoolEnd <String>]
                    [-vlanID <int>]
                    [-kvaLoadBalancerType {unstacked_haproxy, stacked_kube_vip}]
                    [-kvaControlPlaneEndpoint <String>]
                    [-proxyServerHTTP <String>]
                    [-proxyServerHTTP <String>]
                    [-proxyServerNoProxy <String>]
                    [-proxyServerCredential <PSCredential>]
                    [-cloudServiceCidr <String>]
                    [-workingDir <String>]
                    [-version <String>]
                    [-vnetType <String>]
                    [-nodeAgentPort <int>]
                    [-nodeAgentAuthorizerPort <int>]
                    [-clusterRoleName <String>]
                    [-cloudLocation <String>]
                    [-skipHostLimitChecks]
                    [-insecure]
                    [-skipUpdates]
                    [-forceDnsReplication]

   ```

### <a name="example"></a>例

DHCP ネットワークを使用して 2 から 4 ノードのクラスターにデプロイするには:

   ```powershell
   Set-AksHciConfig -imageDir c:\clusterstorage\volume1\Images -cloudConfigLocation c:\clusterstorage\volume1\Config
   ```

仮想 IP プールを使用してデプロイするには:

   ```powershell
   Set-AksHciConfig -imageDir c:\clusterstorage\volume1\Images -cloudConfigLocation c:\clusterstorage\volume1\Config -vipPoolStartIp 10.0.0.20 -vipPoolEndIp 10.0.0.80
   ```

`stacked_kube_vip` ロード バランサーを使用してデプロイするには:

   ```powershell
   Set-AksHciConfig -imageDir c:\clusterstorage\volume1\Images -cloudConfigLocation c:\clusterstorage\volume1\Config -kvaLoadBalancerType stacked_kube_vip -kvaControlPlaneEndpoint 10.0.1.10
   ```

プロキシ サーバーを使用してデプロイするには:

   ```powershell
   Set-AksHciConfig -imageDir c:\clusterstorage\volume1\Images -cloudConfigLocation c:\clusterstorage\volume1\Config -proxyServerHttp "http://proxy.contoso.com:8888" -proxyServerHttps "http://proxy.contoso.com:8888" -proxyServerNoProxy "localhost,127.0.0.1,.svc,10.96.0.0/12,10.244.0.0/16,10.231.110.0/24,10.68.237.0/24" -proxyServerCredential $credential
   ```  

### <a name="optional-parameters"></a>省略可能なパラメーター

`-imageDir`

Azure Stack HCI 上の Azure Kubernetes Service によって VHD イメージが格納されるディレクトリへのパス。 単一ノードのデプロイの場合、既定値は `%systemdrive%\AksHciImageStore` です。 "*複数ノードのデプロイの場合、このパラメーターを指定する必要があります*"。 パスは、共有ストレージ パス ( `C:\ClusterStorage\Volume2\ImageStore`  など) または SMB 共有 ( `\\FileShare\ImageStore` など) を指す必要があります。

`-cloudConfigLocation`

クラウド エージェントによって構成が格納される場所。 単一ノードのデプロイの場合、既定値は `%systemdrive%\wssdcloudagent` です。 この場所は、上記の  `-imageDir` のパスと同じにすることができます。 "*複数ノードのデプロイの場合、このパラメーターを指定する必要があります*"。 パスは、共有ストレージ パス ( `C:\ClusterStorage\Volume2\ImageStore`  など) または SMB 共有 ( `\\FileShare\ImageStore` など) を指す必要があります。 ストレージに常にアクセスできるようにするために、この場所は可用性の高い共有に存在する必要があります。

`-nodeConfigLocation`

ノード エージェントによって構成が格納される場所。 すべてのノードにノード エージェントがあるため、その構成はノードごとにローカルになります。 この場所はローカル パスである必要があります。 既定では、すべてのデプロイで `%systemdrive%\programdata\wssdagent` になります。

`-vnetName`

仮想マシンの接続先となる仮想スイッチの名前。 ホストに既に外部スイッチがある場合は、スイッチの名前をここで渡す必要があります。 スイッチが存在しない場合は作成されます。既定値は "外部" 名です。  

`-controlPlaneVmSize`

コントロール プレーン用に作成する VM のサイズ。 使用できる VM サイズの一覧を取得するには、`Get-AksHciVmSize` を実行します。

`-loadBalancerVmSize`

ロード バランサー VM 用に作成する VM のサイズ。 使用できる VM サイズの一覧を取得するには、`Get-AksHciVmSize` を実行します。

`-sshPublicKey`

SSH 公開キー ファイルへのパス。 この公開キーを使用すると、Azure Stack HCI デプロイ上の Azure Kubernetes Service によって作成された任意の VM にログインできます。 独自の SSH 公開キーを使用する場合は、ここでその場所を渡します。 キーを指定しなかった場合は、`%systemdrive%\akshci\.ssh\akshci_rsa.pub` でそれが検索されます。 そのファイルが存在しない場合は、この場所で SSH キー ペアが生成されて使用されます。

`-vipPoolStartIp`

デプロイに VIP プールを使用している場合、このパラメーターでプールのネットワーク開始を指定します。 長期間維持されるデプロイの場合、IP アドレス プールの一貫性を保つために、VIP プールを使用する必要があります。 これは、常に到達可能である必要があるワークロードがある場合に便利です。 既定値は none です。

`-vipPoolEndIp`

デプロイに VIP プールを使用している場合、このパラメーターでプールのネットワーク終了を指定します。 長期間維持されるデプロイの場合、IP アドレス プールの一貫性を保つために、VIP プールを使用する必要があります。 これは、常に到達可能である必要があるワークロードがある場合に便利です。 既定値は none です。

`-macPoolStart` 

これは、Azure Kubernetes Service ホスト VM に使用する MAC プールの MAC アドレスの先頭を指定するために使用されます。 MAC アドレスの構文では、先頭のバイトの最下位ビットが常に 0 である必要があります。また、先頭のバイトは常に偶数 (つまり、00、02、04、06...) である必要があります。一般的な MAC アドレスは、02:1E:2B:78:00:00 のようになります。 割り当てられる MAC アドレスが一貫性のあるものになるように、長期間維持されるデプロイには MAC プールを使用します。 これは、VM に特定の MAC アドレスが割り当てられる必要がある場合に便利です。 既定値は none です。

`-macPoolEnd`

これは、Azure Kubernetes Service ホスト VM に使用する MAC プールの MAC アドレスの末尾を指定するために使用されます。 MAC アドレスの構文では、先頭のバイトの最下位ビットが常に 0 である必要があります。また、先頭のバイトは常に偶数 (つまり、00、02、04、06...) である必要があります。`-macPoolEnd` として渡されたアドレスの先頭のバイトは、`-macPoolStart` として渡されたアドレスの先頭のバイトと同じである必要があります。 割り当てられる MAC アドレスが一貫性のあるものになるように、長期間維持されるデプロイには MAC プールを使用します。 これは、VM に特定の MAC アドレスが割り当てられる必要がある場合に便利です。 既定値は none です。

`-vlandID`

これを使用すると、ネットワーク VLAN ID を指定できます。 Azure Kubernetes Service ホストおよび Kubernetes クラスター VM ネットワーク アダプターには、指定した VLAN ID がタグ付けされます。 これは、適切な接続を実現するためにタグ付けする必要がある特定の VLAN ID がある場合に使用します。 既定値は none です。

`-kvaLoadBalancerType`

これには、`unstacked_haproxy` または `stacked_kube_vip` のいずれかを渡します。 `unstacked_haproxy` は、別のロード バランサー VM が Azure Kubernetes Service ホストの API サーバー エンドポイントとして HAProxy を使用してデプロイされている場合の既定値です。 `stacked_kube_vip` は、Azure Kubernetes Service ホスト用のロード バランサー ソリューションである [Kubevip](https://kube-vip.io/) です。 これを使用すると、ホストの 1 つの静的 IP アドレスを複数のコントロール プレーン ノード全体のフローティング IP として指定し、その IP 経由で API サーバーの高可用性を保つことができます。 このオプションを選択した場合は、`kvaControlPlaneEndpoint` パラメーターで静的 IP アドレスを指定する必要があり、別個のロード バランサー VM はデプロイされません。

`stacked_kube_vip` には IP アドレスが必須であり、メモリ、CPU、デプロイ時間を節約することでリソースの負担が少なくなります。 フローティング IP として使用する IP アドレスがない場合は、`unstacked_haproxy` を使用する必要があります。 後者のオプションにはロード バランサー VM が必要です。 

`-kvaControlPlaneEndpoint`

これは、`kvaLoadBalancerType` パラメーターが `stacked_kube_vip` に設定されている場合に、Azure Kubernetes Service ホストの API サーバー アドレスとして使用する静的 IP アドレスを指定します。 `stacked_kube_vip` が使用されている場合は、このパラメーターを指定する必要があります。

`-proxyServerHTTP`

これにより、HTTP エンドポイントに接続する必要があるすべてのコンポーネントによって使用されるプロキシ サーバー URI が提供されます。 URI 形式には、URI スキーマ、サーバー アドレス、ポートが含まれます (つまり https://server.com:8888) )。 既定値は none です。

`-proxyServerHTTPS`

これにより、HTTPS エンドポイントに接続する必要があるすべてのコンポーネントで使用されるプロキシサーバー URI が提供されます。 URI 形式には、URI スキーマ、サーバー アドレス、ポートが含まれます (つまり https://server.com:8888) )。 既定値は none です。

`-proxyServerNoProxy`

これは、プロキシから除外されるアドレスのコンマ区切りの文字列です。 既定値は `localhost,127.0.0.1,.svc,10.96.0.0/12,10.244.0.0/16` です。 これにより、localhost トラフィック (localhost、127.0.0.1)、内部の Kubernetes サービス トラフィック (.svc)、Kubernetes Service CIDR (10.96.0.0/12)、および Kubernetes POD CIDR (10.244.0.0/16) がプロキシ サーバーから除外されます。 このパラメーターを使用して、サブネット範囲または名前の除外をさらに追加できます。 **このパラメーターの設定は非常に重要です。正しく構成されていないと、内部の Kubernetes クラスター トラフィックがプロキシに予期せずルーティングされる可能性があるためです。これにより、ネットワーク通信でさまざまなエラーが発生する可能性があります。**


`-proxyServerCredential`

これにより、HTTP または HTTPS プロキシ サーバーに対する認証のためのユーザー名とパスワードが提供されます。 このパラメーターに渡す `PSCredential` オブジェクトを生成するために `Get-Credential` を使用できます。 既定値は none です。

`-cloudServiceCidr`

これを使用すると、MOC CloudAgent サービスに割り当てられる静的 IP/ネットワーク プレフィックスを指定できます。 この値は、CIDR 形式を使用して指定する必要があります (例: 192.168.1.2/16)。 (例: 192.168.1.2/16)。 これを指定すると、IP アドレスが変更されないため、ネットワーク上で重要なすべてのものに常にアクセスできるようにすることができます。 既定値は none です。

`-workingDir`

これは、小さいファイルを格納するためにモジュールで使用する作業ディレクトリです。 既定値は `%PROGRAMFILES%\AksHci`  であり、ほとんどのデプロイでは変更できません。 既定値は変更しないことをお勧めします。 

`-version`

デプロイする Azure Stack HCI 上の Azure Kubernetes Service のバージョン。 既定値は最新バージョンです。 既定値は変更しないことをお勧めします。

`-vnetType`

接続または作成する仮想スイッチの種類。 既定値は "外部" スイッチの種類です。 既定値は変更しないことをお勧めします。

`-nodeAgentPort`

ノード エージェントでリッスンする必要がある TCP/IP ポート番号。 既定値は 45000 です。 既定値は変更しないことをお勧めします。 

`-nodeAgentAuthorizerPort`

ノード エージェントで承認ポートに使用する必要がある TCP/IP ポート番号。 既定値は 45001 です。 既定値は変更しないことをお勧めします。  

`-clusterRoleName`

これにより、クラスター内の汎用サービスとしてクラウド エージェントを作成するときに使用する名前が指定されます。 これは、既定では、プレフィックス ca- と GUID サフィックスを持つ一意の名前になります (例: "ca-9e6eb299-bc0b-4f00-9fd7-942843820c26")。 既定値は変更しないことをお勧めします。

`-cloudLocation` 

このパラメーターにより、Microsoft が運営するカスタム クラウドの場所の名前を提供します。 既定の名前は "MocLocation" です。 既定値は変更しないことをお勧めします。

`-skipHostLimitChecks`

デプロイを続行する前に、メモリとディスク領域が使用可能であることを確認するために実行するすべてのチェックをスキップするよう、スクリプトに要求します。 この設定を使用することは推奨されません。

`-insecure`

セキュリティ保護されていないモード (接続が TLS によって保護されていない) で、クラウド エージェントやノード エージェントなどの Azure Kubernetes Service on Azure Stack HCI コンポーネントをデプロイします。  運用環境に、セキュリティ保護されていないモードを使用しないことをお勧めします。

`-skipUpdates`

利用可能な更新プログラムをスキップする場合は、このフラグを使用します。 この設定を使用することは推奨されません。

`-forceDnsReplication`

一部のシステムでは、DNS レプリケーションに最大 1 時間かかることがあります。 これにより、デプロイの速度が低下します。 この問題が発生した場合は、Install-AksHci がループに入っていることがわかります。 この問題を解決するには、このフラグを使用してみてください。 `-forceDnsReplication` フラグは、保証されている修正プログラムではありません。 フラグの背後にあるロジックが失敗した場合、エラーは非表示になり、フラグが指定されなかったかのようにコマンドが実行されます。 

### <a name="reset-the-azure-kubernetes-service-on-azure-stack-hci-configuration"></a>Azure Stack HCI 構成で Azure Kubernetes Service をリセットする

Azure Kubernetes Service on Azure Stack HCI の構成をリセットするには、次のコマンドを実行します。 このコマンドを単独で実行すると、構成が既定値にリセットされます。

```powershell
Set-AksHciConfig
```

## <a name="step-4-start-a-new-deployment"></a>手順 4:新しいデプロイを開始する

デプロイを構成したら、デプロイを開始する必要があります。 これにより、Azure Stack HCI エージェントまたはサービスおよび Azure Kubernetes Service ホスト上に Azure Kubernetes Service がインストールされます。

デプロイを開始するには、次のコマンドを実行します。

```powershell
Install-AksHci
```

### <a name="verify-your-deployed-azure-kubernetes-service-host"></a>デプロイした Azure Kubernetes Service ホストを確認する

Azure Kubernetes Service ホストがデプロイされたことを確認するには、次のコマンドを実行します。 また、デプロイ後に同じコマンドを使用して Kubernetes クラスターを取得することもできます。

```powershell
Get-AksHciCluster
```

**出力:**
```

Name            : clustergroup-management
Version         : v1.18.8
Control Planes  : 1
Linux Workers   : 0
Windows Workers : 0
Phase           : provisioned
Ready           : True
```

## <a name="step-5-access-your-clusters-using-kubectl"></a>手順 5:kubectl を使用してクラスターにアクセスする

kubectl を使用して Azure Kubernetes Service ホストまたは Kubernetes クラスターにアクセスするには、次のコマンドを実行します。 これにより、指定したクラスターの kubeconfig ファイルが kubectl の既定の kubeconfig ファイルとして使用されます。

```powershell
Get-AksHciCredential -clusterName <String>
                     [-outputLocation <String>]
```

### <a name="example"></a>例

```powershell
Get-AksHciCredential -clusterName clustergroup-management
```

### <a name="required-parameters"></a>必須のパラメーター

`clusterName`

クラスターの名前です。

### <a name="optional-parameters"></a>省略可能のパラメーター

`outputLocation`

kubeconfig をダウンロードする場所。 既定値は `%USERPROFILE%\.kube` です。

## <a name="get-logs"></a>ログを取得する

すべてのポッドからログを取得するには、次のコマンドを実行します。 このコマンドを実行すると、パス `C:\wssd\akshcilogs` に、`akshcilogs` という名前の出力 zip 形式フォルダーが作成されます。

```powershell
Get-AksHciLogs
```

## <a name="update-to-the-latest-version-of-azure-kubernetes-service-on-azure-stack-hci"></a>Azure Kubernetes Service on Azure Stack HCI の最新バージョンに更新する

Azure Kubernetes Service on Azure Stack HCI の最新バージョンに更新するには、次のコマンドを実行します。 update コマンドは、Oct リリースがインストールされている場合にのみ機能します。 10 月のリリースより前のリリースでは機能しません。 この update コマンドにより、Azure Kubernetes Service ホストと、Microsoft が運営するオンプレミスのクラウド プラットフォームを更新します。 このプレビュー リリースについて、Kubernetes バージョンと AKS ホストの OS バージョンは、引き続き同じままです。 このコマンドによって既存のワークロード クラスターはアップグレードされません。 AKS ホストの更新後に作成された新しいワークロード クラスターは、Windows ノードの OS バージョンと Kubernetes バージョンに関して、既存のワークロード クラスターとは異なります。

   ```powershell
   Update-AksHci
   ```
   
Windows ノードを使用している Kubernetes クラスターでサポート対象外である Windows Server の OS バージョンが実行されないように、管理クラスターを更新した直後にワークロード クラスターを更新することをお勧めします。 ワークロード クラスターを更新するには、[ワークロード クラスターの更新](create-kubernetes-cluster-powershell.md)に関するページを参照してください。

## <a name="restart-azure-kubernetes-service-on-azure-stack-hci"></a>Azure Kubernetes Service on Azure Stack HCI を再起動する

Azure Kubernetes Service on Azure Stack HCI を再起動すると、すべての Kubernetes クラスター (ある場合)、および Azure Kubernetes Service ホストが削除されます。 また、ノードから Azure Stack HCI エージェントおよびサービス上の Azure Kubernetes Service もアンインストールされます。 その後、ホストが再作成されるまで、元のインストール プロセスのステップが再実行されます。 `Set-AksHciConfig` を使用して構成した Azure Stack HCI 構成上の Azure Kubernetes Service と、ダウンロードした VHDX イメージは保持されます。

同じ構成設定を使用して Azure Kubernetes Service on Azure Stack HCI を再起動するには、次のコマンドを実行します。

```powershell
Restart-AksHci
```

## <a name="reset-configuration-settings-and-reinstall-azure-kubernetes-service-on-azure-stack-hci"></a>構成設定をリセットして Azure Kubernetes Service on Azure Stack HCI を再インストールする

異なる構成設定を使用して Azure Kubernetes Service on Azure Stack HCI を再インストールするには、次のコマンドを実行します。

```powershell
Uninstall-AksHci
```

上記のコマンドを実行した後、次のコマンドを使用して構成設定を変更できます。 パラメーターは、手順 3 で説明したものと同じです。 パラメーターを指定せずにこのコマンドを実行すると、パラメーターは既定値にリセットされます。

```powershell
Set-AksHciConfig
```

構成を必要な設定に変更した後、次のコマンドを実行して、Azure Stack HCI に Azure Stack Kubernetes を再インストールします。

```powershell
Install-AksHci
```

## <a name="remove-azure-kubernetes-service-on-azure-stack-hci"></a>Azure Stack HCI で Azure Kubernetes Service を削除する

Azure Stack HCI で Azure Kubernetes Service を削除するには、次のコマンドを実行します。 **PowerShell を使用して Windows Admin Center の展開をアンインストールする場合は、`-Force` フラグを指定してコマンドを実行する必要があります。**

```powershell
Uninstall-AksHci
```

上記のコマンドを実行した後、`Install-AksHci` コマンドを実行して、以前と同じ構成で Azure Kubernetes Service ホストをインストールできます。 構成を変更する場合は、install コマンドを実行する前に、必要な変更を指定して `Set-AksHciConfig` を実行します。

古い構成を保持しない場合は、次のコマンドを実行します。

```powershell
Uninstall-AksHci -Force
```

以前に Windows Admin Center を使用してデプロイしたクラスターで PowerShell コマンドを実行すると、PowerShell モジュールによって Windows Admin Center 構成ファイルが存在するかどうかがチェックされます。 Windows Admin Center によって、すべてのノードに Windows Admin Center 構成ファイルが配置されます。 **uninstall コマンドを使用し、それから Windows Admin Center に戻る場合は、上記の uninstall コマンドに `-Force` フラグを付けて実行します。これが行われていない場合、PowerShell と Windows Admin Center が同期しなくなります。**

## <a name="next-steps"></a>次の手順

- [アプリケーションの Kubernetes クラスターを作成する](create-kubernetes-cluster-powershell.md)
