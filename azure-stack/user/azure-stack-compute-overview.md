---
title: Azure Stack Hub VM の概要
description: Azure Stack Hub VM について説明します。
author: sethmanheim
ms.topic: conceptual
ms.date: 02/08/2021
ms.author: sethm
ms.reviewer: kivenkat
ms.lastreviewed: 01/05/2020
ms.openlocfilehash: 70bbb35b8433e8542fc58303f1e8cc56c28c84d7
ms.sourcegitcommit: 824fd33fd5d6aa0c0dac06c21b592bdb60378940
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2021
ms.locfileid: "99850807"
---
# <a name="introduction-to-azure-stack-hub-vms"></a>Azure Stack Hub VM の概要

Azure Stack Hub では、オンデマンドでスケーラブルなコンピューティング リソースの一種として仮想マシン (VM) が提供されます。 コンピューティング環境をより細かく管理する必要がある場合は、VM を選択できます。 この記事では、最初の VM の作成について詳しく説明します。

Azure Stack Hub VM により柔軟な仮想化が可能になります。クラスターや個別のコンピューターを管理する必要はありません。 ただし、VM のメンテナンス、つまり VM 上で動作するソフトウェアの構成、その修正プログラム/更新プログラムの適用、インストールは必要です。

Azure Stack Hub VM は、いくつかの方法で利用できます。 次に例を示します。

- **開発とテスト**:Azure Stack Hub VM を使用すると、アプリケーションのコーディングとテストに必要な特定の構成でコンピューターを作成できます。

- **クラウドのアプリケーション**:アプリケーションの需要は変動する可能性があるため、Azure Stack Hub 内の VM でアプリケーションを実行することは経済的に理に適っています。 VM が必要になったら追加分の料金を支払い、不要になったらシャットダウンすることができます。

- **データセンターの拡張**:Azure Stack Hub 仮想ネットワーク内の VM は、組織のネットワークや Azure に接続できます。

アプリケーションで使用する VM は、ニーズに応じてスケールアップまたはスケールアウトできます。

## <a name="before-creating-a-vm"></a>VM を作成する前に

Azure Stack Hub でアプリケーション インフラストラクチャを構築するときは、設計上の考慮事項が必ず存在します。 インフラストラクチャの作成を開始する前に、VM の次の側面を考慮することが重要です。

- アプリケーション リソースの名前。
- VM のサイズ。
- 作成できる VM の最大数。
- VM で実行されるオペレーティング システム。
- 開始した後の VM の構成。
- VM で必要な関連リソース。

### <a name="names"></a>名前

VM には名前が割り当てられ、コンピューター名がオペレーティング システムの一部として構成されます。 VM の名前は最大で 15 文字です。

Azure Stack Hub を使用してオペレーティング システム ディスクを作成する場合、コンピューター名と VM 名は同じになります。 以前構成されたオペレーティング システムが含まれる独自のイメージをアップロードして使用し、それを基に VM を作成する場合は、別の名前にすることができます。 独自のイメージ ファイルをアップロードするときは、ベスト プラクティスとして、オペレーティング システムのコンピューター名と VM 名が同じであることを確認します。

### <a name="vm-size"></a>VM サイズ

使用する VM のサイズは、実行するワークロードによって決まります。 さらに、選択したサイズによって、処理能力、メモリ、ストレージの容量などの要素が決まります。 Azure Stack Hub では、さまざまな種類の使用をサポートするために、さまざまなサイズが用意されています。

### <a name="vm-limits"></a>VM の制限

サブスクリプションにはそれぞれ既定のクォータ制限が設けられており、プロジェクトで VM をデプロイする場合に、その点が影響する可能性があります。 現在は、リージョンあたり 20 VM の制限がサブスクリプションごとに設けられています。

### <a name="operating-system-disks-and-images"></a>オペレーティング システム ディスクおよびイメージ

Azure Stack Hub の VM は、第 1 世代の仮想ハードディスク (VHD/VHDX) 形式に制限されます。 VHD を使用して、コンピューターのオペレーティング システム (OS) とデータを格納できます。 VHD は、OS をインストールするためのイメージにも使用されます。 Azure Stack Hub には、オペレーティング システムのさまざまなバージョンと種類で使用できるマーケットプレースが用意されています。 Marketplace イメージは、イメージの発行元、オファー、SKU、およびバージョン (通常、最新バージョンは **最新** として指定) によって識別されます。

次の表に、イメージに関する情報を見つける方法を示します。

|Method|説明|
|---------|---------|
|Azure Stack Hub ポータル|値は、使用するイメージを選択する際に自動的に指定されます。|
|Azure Stack Hub PowerShell|`Get-AzVMImagePublisher -Location "location"`<br>`Get-AzVMImageOffer -Location "location" -Publisher "publisherName"`<br>`Get-AzVMImageSku -Location "location" -Publisher "publisherName" -Offer "offerName"`|
|REST API     |[イメージ発行元の一覧表示](/rest/api/compute/platformimages/platformimages-list-publishers)<br>[イメージ プランの一覧表示](/rest/api/compute/platformimages/platformimages-list-publisher-offers)<br>[イメージ SKU の一覧表示](/rest/api/compute/platformimages/platformimages-list-publisher-offer-skus)|

独自のイメージをアップロードして使用することもできます。 その場合、発行元の名前、オファー、および SKU は使用されません。

### <a name="extensions"></a>拡張機能

VM の拡張機能は、デプロイ後の構成と自動化タスクを通じて VM に追加の機能を提供します。 拡張機能を利用し、次の共通タスクを実行できます。

- **カスタム スクリプトの実行**:カスタム スクリプト拡張機能を利用すれば、VM のプロビジョニングの際にスクリプトを実行して、VM でワークロードを容易に構成できます。

- **構成のデプロイと管理**:PowerShell Desired State Configuration (DSC) 拡張機能を利用すれば、VM で DSC を容易に設定して構成と環境を管理できます。

- **診断データの収集**:Azure Diagnostics 拡張機能を利用すれば、アプリケーションの正常性を監視するために使用できる診断データを収集するように VM を容易に構成できます。

### <a name="related-resources"></a>関連リソース

次の表のリソースは VM によって使用されるため、VM の作成時に存在するか、作成する必要があります。

|リソース|必須|説明|
|---------|---------|---------|
|Resource group|はい|VM は、リソース グループに含まれる必要があります。|
|ストレージ アカウント|いいえ|マネージド ディスクを使用する場合、VM には仮想ハード ディスクを格納するためのストレージ アカウントは必要ありません。 |
|仮想ネットワーク|はい|VM は、仮想ネットワークのメンバーである必要があります。|
|パブリック IP アドレス|いいえ|VM には、リモートでアクセスするためのパブリック IP アドレスを割り当てることができます。|
|ネットワーク インターフェイス|はい|VM には、ネットワークで通信するためのネットワーク インターフェイスが必要です。|
|データ ディスク|いいえ|VM には、ストレージ容量を拡張するためのデータ ディスクを含めることができます。|

## <a name="create-your-first-vm"></a>最初の VM の作成

VM はいくつかの方法で作成できます。 どれを選ぶかは環境によって異なります。 次の表は、VM の作成を開始するのに役立つ情報を提供します。

|Method|[アーティクル]|
|---------|---------|
|Azure Stack Hub ポータル|[Azure Stack Hub ポータルを使用して Windows VM を作成する](azure-stack-quick-windows-portal.md)。<br>[Azure Stack Hub ポータルを使用して Linux VM を作成する](azure-stack-quick-linux-portal.md)。|
|テンプレート|Azure Stack Hub のクイックスタート テンプレートは次の場所にあります。<br> [https://github.com/Azure/AzureStack-QuickStart-Templates](https://aka.ms/aa6z60s)|
|PowerShell|[Azure Stack Hub で PowerShell を使用して Windows VM を作成する](azure-stack-quick-create-vm-windows-powershell.md)<br>[Azure Stack Hub で PowerShell を使用して Linux VM を作成する](azure-stack-quick-create-vm-linux-powershell.md)|
|CLI|[Azure Stack Hub で CLI を使用して Windows VM を作成する](azure-stack-quick-create-vm-windows-cli.md)<br>[Azure Stack Hub で CLI を使用して Linux VM を作成する](azure-stack-quick-create-vm-linux-cli.md)|

## <a name="manage-your-vm"></a>VM の管理

VM は、ブラウザーベースのポータル、スクリプトがサポートされるコマンドライン ツール、または直接 API を使用して管理できます。 一般的な管理タスクは次のとおりです。

- VM に関する情報を取得する。
- VM に接続する。
- 可用性を管理する。
- バックアップを作成する。

### <a name="get-information-about-your-vm"></a>VM に関する情報を取得する

次の表に、VM に関する情報の取得方法の一部を示します。

|Method|説明|
|---------|---------|
|Azure Stack Hub ポータル|ハブ メニューの **[仮想マシン]** をクリックし、一覧から VM を選択します。 その VM のページで、概要情報を確認したり、値の設定やメトリックの監視を実行したりできます。|
|Azure PowerShell|VM の管理は、Azure と Azure Stack Hub とで似ています。 PowerShell を使用する方法については、Azure トピックの「[Azure PowerShell モジュールを使用して Windows VM を作成および管理する](/azure/virtual-machines/windows/tutorial-manage-vm#understand-vm-sizes)」を参照してください。|
|クライアント SDK|C# による VM の管理は、Azure と Azure Stack Hub とで似ています。 詳細については、「[C# を使用して Azure で Windows VM を作成および管理する](/azure/virtual-machines/windows/csharp)」を参照してください。|

### <a name="connect-to-your-vm"></a>VM に接続する

Azure Stack Hub ポータルの **[接続]** オプションを使用して、VM に接続できます。

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub の VM に関する考慮事項](azure-stack-vm-considerations.md)
