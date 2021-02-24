---
title: Azure Stack Hub を使用して iSCSI ストレージに接続する方法
description: Azure Stack Hub を使用して iSCSI ストレージに接続する方法について説明します。
author: mattbriggs
ms.topic: how-to
ms.date: 12/2/2020
ms.author: mabrigg
ms.reviewer: sijuman
ms.lastreviewed: 12/2/2020
ms.openlocfilehash: f6e02aa1185070d641d946ef99a44858ce8a1e5b
ms.sourcegitcommit: 9ef2cdc748cf00cd3c8de90705ea0542e29ada97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/02/2020
ms.locfileid: "96525746"
---
# <a name="connect-to-iscsi-storage-with-azure-stack-hub"></a>Azure Stack Hub を使用して iSCSI ストレージに接続する

この記事のテンプレートを使用すると、Azure Stack Hub 仮想マシン (VM) をオンプレミスの iSCSI ターゲットに接続し、Azure Stack Hub の外部やデータセンターの他の場所でホストされるストレージを使用するように VM を設定できます。 この記事では、Windows マシンを iSCSI ターゲットとして使用する方法について説明します。

このテンプレートは、[Azure Intelligent Edge Patterns GitHub](https://github.com/lucidqdreams/azure-intelligent-edge-patterns) リポジトリの **lucidqdreams** フォークにあります。 このテンプレートは **storage-iSCSI** フォルダーにあります。 このテンプレートは、Azure Stack Hub 側で iSCSI ターゲットに接続するために必要なインフラストラクチャを設定するように設計されています。 これには、iSCSI イニシエーターとして機能する仮想マシンと、それに付随する VNet、NSG、PIP、およびストレージが含まれます。 テンプレートをデプロイした後は、2 つの PowerShell スクリプトを実行し、構成を完了する必要があります。 1 つのスクリプトはオンプレミスの VM (ターゲット) 上で実行され、1 つは Azure Stack Hub VM (イニシエーター) 上で実行されます。 これらの処理が完了すると、オンプレミスのストレージが Azure Stack Hub VM に追加されます。 

## <a name="overview"></a>概要

この図は、オンプレミスの Windows マシン (物理または仮想) から iSCSI マウント ディスクを使用して Azure Stack Hub でホストされている VM を示しています。これにより、Azure Stack Hub の外部ストレージを iSCSI プロトコルで Azure Stack Hub ホスト VM 内にマウントすることができます。

![この図は、外部 iSCSI マウント ディスクにアクセスする Azure Stack Hub 上でホストされている VM を示しています。](./media/azure-stack-network-howto-iscsi-storage/overview-iscsi2.svg)

### <a name="requirements"></a>必要条件

- Windows Server 2016 Datacenter または Windows Server 2019 Datacenter を実行するオンプレミスのマシン (物理または仮想)。
- 次の Azure Stack Hub Marketplace 必須項目。
    -  Windows Server 2016 Datacenter または Windows Server 2019 Datacenter (最新のビルドを推奨)。
    -  PowerShell DSC 拡張機能。
    -  カスタム スクリプト拡張機能。
    -  既存の仮想マシンまたは物理マシン。 このマシンには、2 つのネットワーク アダプターが搭載されることが理想的です。 これは、たとえば SAN などの別の iSCSI ターゲットの可能性もあります。

### <a name="things-to-consider"></a>考慮事項

- ネットワーク セキュリティ グループは、テンプレートのサブネットに適用されます。 これを確認し、必要に応じて追加の許容量を設定します。
- RDP 拒否規則はトンネル NSG に適用され、パブリック IP アドレスを使用して VM にアクセスする場合は許可に設定する必要があります。
- このソリューションでは、DNS 解決を考慮していません。
- Chapusername と Chappassword を変更する必要があります。 Chappassword の長さは 12 から 16 文字にする必要があります。
- iSCSI 接続では構成でローカル アドレスが使用されるため、このテンプレートでは VM に静的 IP アドレスを使用しています。
- このテンプレートには BYOL Windows ライセンスが使用されています。
- また、Linux ベースのシステムを iSCSI ターゲットに接続することもできます。 手順については、ubuntu のドキュメントの「[iSCSI Initiator (iSCSI イニシエーター)](https://help.ubuntu.com/lts/serverguide/iscsi-initiator.html)」を参照してください。

### <a name="options"></a>Options

- **_artifactsLocation** および **_artifactsLocationSasToken** パラメーターを使用してご自分の Blob ストレージ アカウントと SAS トークンを使用し、SAS トークンでご自分のストレージ BLOB を使用することができます。
- このテンプレートには、VNet の名前付けと IP アドレス指定の既定値が指定されています。
- この構成では、iSCSI クライアントから取得される iSCSI NIC は 1 つだけです。 個別のサブネットと NIC を使用する複数の構成をテストしましたが、複数のゲートウェイで問題が発生したため、トラフィックを分離して本当の意味で冗長性を確保するために個別のストレージ サブネットを作成しようとしました。 
- デプロイが失敗する可能性があるので、これらの値が有効なサブネットおよびアドレスの範囲内に収まるように注意します。 
- PowerShell DSC パッケージの主な目的は、再起動が保留されているかどうかを確認することです。 この DSC は、必要に応じてさらにカスタマイズできます。 詳細については、「[omputerManagementDsc](https://github.com/PowerShell/ComputerManagementDsc/)」を参照してください。

### <a name="resource-group-template-iscsi-client"></a>リソース グループ テンプレート (iSCSI クライアント)

この図は、iSCSI ターゲットへの接続に使用できる iSCSI クライアントを作成するために、テンプレートからデプロイされるリソースを示しています。 このテンプレートでは、VM とその他のリソースがデプロイされ、さらに prepare-iSCSIClient.ps1 が実行され、VM が再起動されます。

![この図は、iSCSI ターゲットに接続する iSCSI クライアントを作成するために、テンプレートからデプロイされるリソースを示しています。 内部サブネットと NIC (ネットワーク カード)、内部 PIP (プライベート インターネット プロトコル)、NSG (ネットワーク セキュリティ グループ) を備えたファイル サーバーが示されています。](./media/azure-stack-network-howto-iscsi-storage/iscsi-file-server.svg)

### <a name="the-deployment-process"></a>デプロイ プロセス

リソース グループ テンプレートによって出力が生成されます。出力は次の手順の入力となります。 主な焦点は、iSCSI トラフィックの発信元のサーバー名と Azure Stack Hub パブリック IP アドレスです。 この例の場合は次のとおりです。

1. インフラストラクチャ テンプレートをデプロイします。
2. データセンター内の他の場所でホストされている VM に Azure Stack Hub VM をデプロイします。 
3. テンプレートからの IP アドレスとサーバー名の出力を、iSCSI ターゲット (仮想マシンまたは物理サーバー) のスクリプトの入出力パラメーターとして使用して、`Create-iSCSITarget.ps1` を実行します。
4. `Connect-toiSCSITarget.ps1` スクリプトを実行するための入力として、iSCSI ターゲット サーバーの外部 IP アドレスを使用します。 

![この図には、上記の 4 つの手順のうちの最初の 3 つが示されており、入力と出力が含まれます。 手順は次のとおりです。インフラストラクチャのデプロイ、iSCSI ターゲットの作成、iSCSI への接続。](./media/azure-stack-network-howto-iscsi-storage/process.svg)

### <a name="inputs-for-azuredeployjson"></a>azuredeploy.json の入力

|**パラメーター**|**既定値**|**description**|
|------------------|---------------|------------------------------|
|WindowsImageSKU         |2019-Datacenter   |ベース Windows VM イメージを選択してください
|VMSize                  |Standard_D2_v2    |VM サイズを入力してください
|VMName                  |FileServer        |VM 名
|adminUsername           |storageadmin      |新しい VM の管理者の名前
|adminPassword           |                  |新しい VM の管理者アカウントのパスワード。 既定値はサブスクリプション ID です
|VNetName                |ストレージ           |VNet の名前。 これはリソースのラベル付けに使用されます
|VNetAddressSpace        |10.10.0.0/23      |VNet のアドレス空間
|VNetInternalSubnetName  |内部          |VNet の内部サブネット名
|VNetInternalSubnetRange |10.10.1.0/24      |VNet の内部サブネットのアドレス範囲
|InternalVNetIP          |10.10.1.4         |ファイル サーバーの内部 IP の静的アドレス。
|_artifactsLocation      ||
|_artifactsLocationSasToken||

### <a name="deployment-steps"></a>デプロイメントの手順

1. `azuredeploy.json` を使用して iSCSI クライアント インフラストラクチャをデプロイします
2. オンプレミス サーバーの iSCSI ターゲット上で `Create-iSCSITarget.ps1` を実行します。 テンプレートが完成したら、最初の手順の出力を使用して、オンプレミス サーバーの iSCSI ターゲット上で Create-iSCSITarget.ps1 を実行する必要があります。
3. iSCSI クライアント上で `Connect-toiSCSITarget.ps1` を実行します。 iSCSI クライアント上で、iSCSI ターゲットの詳細を含む Connect-toiSCSITarget.ps1 を実行します

## <a name="adding-iscsi-storage-to-existing-vms"></a>既存の VM への iSCSI ストレージの追加

既存の仮想マシン上でスクリプトを実行し、iSCSI クライアントから iSCSI ターゲットに接続することもできます。 このフローは、iSCSI ターゲットを自分で作成する場合に使用します。 次の図は、PowerShell スクリプトの実行フローを示しています。 これらのスクリプトは、Script ディレクトリにあります。

![この図には、以下で説明する 3 つのスクリプトが示されています。 実行順に示すと、Prepare-iSCSIClient.ps1 (クライアントで実行)、Create iSCSITarget.ps1 (ターゲットで実行)、Connect-toiSCSITarget.ps1 (クライアントで実行) です。](./media/azure-stack-network-howto-iscsi-storage/script-flow.svg)

### <a name="prepare-iscsiclientps1"></a>Prepare-iSCSIClient.ps1

`Prepare-iSCSIClient.ps1` スクリプトによって、iSCSI クライアントに前提条件がインストールされます。これには次のものが含まれます。
- マルチパス IO サービスのインストール
- iSCSI イニシエーター サービスを自動的に開始するように設定する
- マルチパス MPIO の iSCSI へのサポートを有効にする
- すべての iSCSI ボリュームの自動要求を有効にする
- ディスクのタイムアウトを 60 秒に設定する

これらの前提条件のインストール後にシステムを再起動することが重要です。 MPIO 負荷分散ポリシーを設定するには、再起動が必要です。

### <a name="create-iscsitargetps1"></a>Create-iSCSITarget.ps1

`Create-iSCSITarget.ps1` スクリプトは、ストレージ サーバーで実行されます。 イニシエーターによって制限される複数のディスクとターゲットを作成できます。 このスクリプトを複数回実行して多数の仮想ディスクを作成し、さまざまなターゲットにアタッチすることができます。 複数のディスクを 1 つのターゲットに接続できます。 

|**入力**|**既定値**|**description**|
|------------------|---------------|------------------------------|
|RemoteServer         |FileServer               |iSCSI ターゲットに接続しているサーバーの名前
|RemoteServerIPs      |1.1.1.1                  |iSCSI トラフィックの発信元の IP アドレス
|DiskFolder           |C:\iSCSIVirtualDisks     |仮想ディスクが格納されるフォルダーとドライブ
|DiskName             |DiskName                 |ディスク VHDX ファイルの名前
|DiskSize             |5GB                      |VHDX ディスク サイズ
|TargetName           |RemoteTarget01           |iSCSI クライアントのターゲット構成を定義するために使用されるターゲット名。 
|ChapUsername         |username                 |Chap 認証のユーザー名
|ChapPassword         |userP@ssw0rd!            |Chap 認証のパスワード名。 12 から 16 文字にする必要があります

### <a name="connect-toiscsitargetps1"></a>Connect-toiSCSITarget.ps1

`Connect-toiSCSITarget.ps1` は最終的なスクリプトです。iSCSI クライアント上で実行され、iSCSI ターゲットによって提示されたディスクが iSCSI クライアントにマウントされます。

|**入力**|**既定値**|**description**|
|------------------|---------------|------------------------------|
|TargetiSCSIAddresses   |"2.2.2.2"、"2.2.2.3"    |iSCSI ターゲットの IP アドレス
|LocalIPAddresses       |"10.10.1.4"            |これは iSCSI トラフィックが発信される内部 IP アドレスです。
|LoadBalancePolicy      |C:\iSCSIVirtualDisks   |iSCSI トラフィックの発信元の IP アドレス
|ChapUsername           |username               |Chap 認証のユーザー名
|ChapPassword           |userP@ssw0rd!          |Chap 認証のパスワード名。 12 から 16 文字にする必要があります

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub ネットワークの違いと考慮事項](azure-stack-network-differences.md)  