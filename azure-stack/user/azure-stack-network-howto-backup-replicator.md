---
title: 複数の Azure Stack Hub サブスクリプション間でリソースをレプリケートする
description: Azure Stack Hub サブスクリプション レプリケーターの一連のスクリプトを使用してリソースをレプリケートする方法について説明します。
author: mattbriggs
ms.topic: how-to
ms.date: 08/24/2020
ms.author: mabrigg
ms.reviewer: rtiberiu
ms.lastreviewed: 11/07/2019
ms.openlocfilehash: 14f86b63e8089069d53e7b849d4bfea55007f34e
ms.sourcegitcommit: 3e2460d773332622daff09a09398b95ae9fb4188
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/15/2020
ms.locfileid: "90571696"
---
# <a name="replicate-resources-using-the-azure-stack-hub-subscription-replicator"></a>Azure Stack Hub サブスクリプション レプリケーターを使用してリソースをレプリケートする

Azure Stack Hub サブスクリプション レプリケーターの PowerShell スクリプトを使用して、Azure Stack Hub サブスクリプション間、Azure Stack Hub スタンプ間、または Azure Stack Hub と Azure の間でリソースをコピーできます。 レプリケーター スクリプトでは、さまざまな Azure および Azure Stack Hub サブスクリプションから Azure Resource Manager リソースを読み取り、再構築します。 この記事では、スクリプトの動作とスクリプトの使用方法について説明し、スクリプト操作の参照情報を示します。

この記事で使用されるスクリプトは [Azure Intelligent Edge Patterns GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns) リポジトリにあります。 スクリプトは [ssubscription replicator](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/subscription%20replicator) フォルダーにあります。

## <a name="subscription-replicator-overview"></a>サブスクリプション レプリケーターの概要

Azure サブスクリプション レプリケーターはモジュール式に設計されました。 このツールには、リソースのレプリケーションを調整するコアプロセッサが使用されます。 また、このツールは、さまざまな種類のリソースをコピーするためのテンプレートとして機能するカスタマイズ可能なプロセッサをサポートしています。 

コア プロセッサは、次の 3 つのスクリプトで構成されています。

- **resource_retriever.ps1**

    - 出力ファイルを格納するフォルダーを生成します。

    - ソース サブスクリプションにコンテキストを設定します。

    - リソースを取得し、**resource_processor.ps1** に渡します。

- **resource_processor.ps1**

    - **resource_retriever.ps1** から渡されたリソースを処理します。

    - 使用するカスタマイズ済みプロセッサを決定し、リソースを渡します。

- **post_process.ps1**

    - カスタマイズ済みプロセッサによって生成された出力を後処理して、ターゲット サブスクリプションにデプロイする準備をします。

    - ターゲット サブスクリプション内のリソースをデプロイするデプロイ コードを生成します。

3 つのスクリプトでは、より高い柔軟性を実現するために、標準的な方法で情報の流れを制御します。 たとえば、新しいリソースのサポートを追加する場合でも、コア プロセッサのコードを変更する必要はありません。

前述のカスタマイズ済みプロセッサは、特定の種類のリソースの処理方法を指示する `ps1` ファイルです。 カスタマイズ済みプロセッサの名前は、常にリソースの型データを使用して名前が付けられます。 たとえば、`$vm` に仮想マシン オブジェクトが保持されている場合、`$vm`.Type を実行すると `Microsoft.Compute/virtualMachines` が生成されます。 つまり、仮想マシンのプロセッサには `virtualMachines_processor.ps1` という名前が付けられ、必ずリソース メタデータに出現する名前とまったく同じになります。これにより、コア プロセッサでは、使用するカスタマイズ済みプロセッサを決定します。

カスタマイズ済みプロセッサでは、重要な情報を決定し、その情報をリソース メタデータから引き出す方法を指示することにより、リソースのレプリケート方法を指示します。 次に、カスタマイズ済みプロセッサでは、抽出されたすべてのデータを取得し、それを使用し、Azure Resource Manager テンプレートと組み合わせて使用されるパラメーター ファイルを生成して、ターゲット サブスクリプションにリソースをデプロイします。 このパラメーター ファイルは、post_process.ps1 によって後処理された後、**Parameter_Files** に保存されます。

Replicator ファイル構造には、**Standardized_ARM_Templates** という名前のフォルダーがあります。 ソース環境に応じて、デプロイではこれらの標準化された Azure Resource Manager テンプレートのいずれかを使用するか、カスタマイズされた Azure Resource Manager テンプレートを生成する必要があります。 このケースでは、カスタマイズ済みプロセッサから Azure Resource Manager テンプレート ジェネレーターを呼び出す必要があります。 前述の例では、仮想マシン用の Azure Resource Manager テンプレート ジェネレーターの名前は **virtualMachines_ARM_Template_Generator.ps1** という名前になります。 Azure Resource Manager テンプレート ジェネレーターは、リソースのメタデータに含まれる情報に基づいて、カスタマイズされた Azure Resource Manager テンプレートを作成する役割を担います。 たとえば、仮想マシン リソースに可用性セットのメンバーであることを指定するメタデータがある場合、Azure Resource Manager テンプレート ジェネレーターによって、仮想マシンが属する可用性セットの ID を指定するコードで Azure Resource Manager テンプレートが作成されます。 これにより、仮想マシンが新しいサブスクリプションにデプロイされると、デプロイ時に可用性セットに自動的に追加されます。 これらのカスタマイズされた Azure Resource Manager テンプレートは、**Standardized_ARM_Templates** フォルダー内にある **Custom_ARM_Templates** フォルダーに保存されます。 post_processor.ps1 は、デプロイに標準化された Azure Resource Manager テンプレートを使用するか、カスタマイズしたテンプレートを使用するかを決定し、対応するデプロイ コードを生成する役割を担います。

スクリプト **post-process.ps1** は、パラメーター ファイルをクリーン アップし、ユーザーが新しいリソースをデプロイするために使用するスクリプトを作成する役割を担います。 クリーンアップ フェーズでは、スクリプトにより、ソースのサブスクリプション ID、テナント ID、および場所へのすべての参照が、対応するターゲット値に置き換えられます。 次に、パラメーター ファイルが **Parameter_Files** フォルダーに出力されます。 次に、処理対象のリソースにカスタマイズされた Azure Resource Manager テンプレートを使用するかどうかが決定され、対応するデプロイ コードが生成されます。これには **New-AzureRmResourceGroupDeployment** コマンドレットが使用されます。 このデプロイ コードは、**Deployment_Files** フォルダーに保存されている **DeployResources.ps1** という名前のファイルに追加されます。 最後に、スクリプトによってリソースが属するリソース グループが決定され、**DeployResourceGroups.ps1** スクリプトがチェックされ、そのリソース グループをデプロイするデプロイ コードが既に存在するかどうかが確認されます。 しない場合はそのスクリプトにリソース グループをデプロイするコードが追加され、する場合は何も実行されません。

### <a name="dynamic-api-retrieval"></a>動的 API の取得

このツールには動的 API 取得が組み込まれているため、ソース サブスクリプションで入手できる最新のリソース プロバイダー API バージョンが使用され、ターゲット サブスクリプションにリソースがデプロイされます。

![API の取得の図](./media/azure-stack-network-howto-backup-replicator/image1.png)

**resource_processor.ps1** での API の取得の図。

ただし、ターゲット サブスクリプションのリソース プロバイダー API バージョンがソース サブスクリプションのバージョンよりも古く、ソース サブスクリプションから提供されているバージョンがサポートされていない可能性があります。 この場合、デプロイの実行時にエラーがスローされます。 これを解決するには、ターゲット サブスクリプションのリソース プロバイダーを更新して、ソース サブスクリプションと一致するようにします。

### <a name="parallel-deployments"></a>並列デプロイ

このツールには **parallel** という名前のパラメーターが必要です。 このパラメーターは、取得したリソースを並行してデプロイするかどうかを指定するブール値を受け取ります。 値が **true** に設定されている場合、**New-AzureRmResourceGroupDeployment** への各呼び出しには **-asJob** フラグが含まれ、リソースの種類に基づいて、並列ジョブが終了するまで待つコード ブロックがリソース デプロイのセットの間に追加されます。 これにより、次の種類のリソースがデプロイされる前に、ある種類のすべてのリソースが確実にデプロイされます。 **parallel** パラメーター値が **false** に設定されている場合は、すべてのリソースがシリアルでデプロイされます。

## <a name="add-additional-resource-types"></a>新しいリソースの種類の追加

新しいリソースの種類の追加は簡単です。 開発者は、カスタマイズ済みプロセッサと、Azure Resource Manager テンプレートまたは Azure Resource Manager テンプレート ジェネレーターを作成する必要があります。 それが完了したら、開発者は **$resourceType** パラメーターと resource_retriever.ps1 の **$resourceTypes** 配列の ValidateSet にリソースの種類を追加する必要があります。 リソースの種類を **$resourceTypes** 配列に追加する場合は、正しい順序で追加する必要があります。 配列の順序によってリソースがデプロイされる順序が決まるため、依存関係を考慮してください。 最後に、カスタマイズ済みプロセッサで Azure Resource Manager テンプレート ジェネレーターを利用する場合、**post_process.ps1** の **$customTypes** 配列にリソースの種類名を追加する必要があります。

## <a name="run-azure-subscription-replicator"></a>Azure サブスクリプション レプリケーターの実行

Azure サブスクリプション レプリケーター (v3) ツールを実行するには、すべてのパラメーターを指定して resource_retriever.ps1 を開始する必要があります。 **resourceType** パラメーターには、1 つのリソースの種類ではなく **All** を選択するオプションがあります。 **All** が選択されている場合、resource_retriever.ps1 では、デプロイの実行時に依存するリソースが最初にデプロイされるように、すべてのリソースが順番に処理されます。 たとえば、仮想マシンを適切にデプロイするには VNet が必要であるため、VNet は仮想マシンの前にデプロイされます。

スクリプトの実行が完了すると、3 つの新しいフォルダー **Deployment_Files**、**Parameter_Files**、および **Custom_ARM_Templates** が作成されます。

 > [!NOTE]  
 > 生成されたスクリプトを実行する前に、適切な環境を設定し、ターゲット サブスクリプション (新しい Azure Stack Hub など) にログインして、作業ディレクトリを **Deployment_Files** フォルダーに設定する必要があります。

Deployment_Files には、2 つのファイル **DeployResourceGroups.ps1** と **DeployResources.ps1** が保持されます。 DeployResourceGroups.ps1 を実行すると、リソース グループがデプロイされます。 DeployResources.ps1 を実行すると、処理されたすべてのリソースがデプロイされます。 リソースの種類として **All** または **Microsoft.Compute/virtualMachines** を指定してツールを実行した場合、DeployResources.ps1 では仮想マシンの管理者パスワードを入力するようにユーザーに求められます。これはすべての仮想マシンの作成に使用されます。

### <a name="example"></a>例

1.  スクリプトを実行します。

    ![スクリプトを実行する](./media/azure-stack-network-howto-backup-replicator/image2.png)

    > [!NOTE]  
    > PS インスタンスのソース環境とサブスクリプション コンテキストを必ず構成してください。 

2.  新しく作成されたフォルダーを確認します。

    ![フォルダーの確認](./media/azure-stack-network-howto-backup-replicator/image4.png)

3.  コンテキストをターゲット サブスクリプションに設定し、フォルダーを **Deployment_Files** に変更して、リソース グループをデプロイしてから (DeployResourceGroups.ps1 スクリプトを実行)、リソースのデプロイを開始します (DeployResources.ps1 スクリプトを実行)。

    ![デプロイの構成と開始](./media/azure-stack-network-howto-backup-replicator/image6.png)

4.  `Get-Job` を実行して状態を確認します。 Get-Job | Receive-Job から結果が返されます。

## <a name="clean-up"></a>クリーンアップ

replicatorV3 フォルダー内には、**cleanup_generated_items.ps1** という名前のファイルがあります。これにより、**Deployment_Files**、**Parameter_Files**、および **Custom_ARM_Templates** フォルダーとそのすべての内容が削除されます。

## <a name="subscription-replicator-operations"></a>サブスクリプション レプリケーターの操作

現在、Azure サブスクリプション レプリケーター (v3) では、次の種類のリソースをレプリケートできます。

- Microsoft.Compute/availabilitySets

- Microsoft.Compute/virtualMachines

- Microsoft.Network/loadBalancers

- Microsoft.Network/networkSecurityGroups

- Microsoft.Network/publicIPAddresses

- Microsoft.Network/routeTables

- Microsoft.Network/virtualNetworks

- Microsoft.Network/virtualNetworkGateways

- Microsoft.Storage/storageAccounts

リソースの種類として **All** を指定してツールを実行すると、レプリケートおよびデプロイ時に次の順序で実行されます (以下では、sku、offer など、すべてのリソースの構成がレプリケートされます)。

- Microsoft.Network/virtualNetworks

    - レプリケート: - すべてのアドレス空間 - すべてのサブネット

- Microsoft.Network/virtualNetworkGateways

    - レプリケート: - パブリック IP の構成 - サブネットの構成 - VPN の種類 - ゲートウェイの種類

- Microsoft.Network/routeTables

- Microsoft.Network/networkSecurityGroups

    - レプリケート: - すべてのセキュリティ規則の受信と送信

- Microsoft.Network/publicIPAddresses

- Microsoft.Network/loadBalancers

    - レプリケート: - プライベート IP アドレス - パブリック IP アドレスの構成 - サブネットの構成
    
- Microsoft.Compute/availabilitySets

    - レプリケート: - 障害ドメインの数 - 更新ドメインの数

- Microsoft.Storage/storageAccounts

- Microsoft.Compute/virtualMachines
    - レプリケート:  
            - データ ディスク (データなし)  
            - 仮想マシンのサイズ  
            - オペレーティング システム  
            - 診断ストレージ アカウントの構成  
            - パブリック IP の構成  
            - ネットワーク インターフェイス  
            - ネットワーク インターフェイスのプライベート IP アドレス  
            - ネットワーク セキュリティ グループの構成  
            - 可用性セットの構成  

> [!NOTE]  
> OS ディスクとデータ ディスクのマネージド ディスクのみが作成されます。 現時点では、ストレージ アカウントの使用はサポートされていません。 

### <a name="limitations"></a>制限事項

このツールでは、ターゲット サブスクリプションのリソース プロバイダーがソース サブスクリプションからレプリケートされるすべてのリソースとオプションをサポートしている限り、あるサブスクリプションから別のサブスクリプションにリソースをレプリケートできます。

レプリケーションを正常に行うには、ターゲット サブスクリプションのリソース プロバイダーのバージョンがソース サブスクリプションのバージョンと一致していることを確認します。

商用 Azure から商用 Azure にレプリケートする場合、または Azure Stack Hub 内の 1 つのサブスクリプションから同じ Azure Stack Hub 内の別のサブスクリプションにレプリケートする場合は、ストレージ アカウントをレプリケートするときに問題が発生します。 これは、すべてのストレージ アカウント名がすべての商用 Azure または Azure Stack Hub リージョンまたはインスタンスのすべてのサブスクリプションで一意であるというストレージ アカウントの名前付け要件によるものです。 Stack は別々のリージョンまたはインスタンスであるため、異なる Azure Stack Hub インスタンス間でストレージ アカウントをレプリケートできます。



## <a name="next-steps"></a>次のステップ

[Azure Stack Hub ネットワークの違いと考慮事項](azure-stack-network-differences.md)  
