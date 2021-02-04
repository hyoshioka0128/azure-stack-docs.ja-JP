---
title: Azure Stack 仮想マシン (VM) の機能 | Microsoft Docs
description: MDC で仮想マシン (VM) を操作する際のさまざまな機能と考慮事項について説明します。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 2/1/2021
ms.author: mabrigg
ms.reviewer: kivenkat
ms.lastreviewed: 12/20/2019
ms.openlocfilehash: 37e5004bd31ba37546abeee67feafe280b39a48e
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99246284"
---
# <a name="azure-stack-vm-features---modular-data-center-mdc"></a>Azure Stack VM の機能 - Modular Data Center (MDC)

Azure Stack 仮想マシン (VM) では、オンデマンドのスケーラブルなコンピューティング リソースが提供されます。 VM を展開する前に、Azure Stack と Microsoft Azure で使用できる VM の機能の相違点について学ぶ必要があります。 この記事では、これらの相違点について説明し、VM の展開を計画する際の重要な考慮事項を特定します。 Azure Stack と Azure の違いの概要については、「[主な考慮事項](../user/azure-stack-considerations.md)」をご覧ください。

## <a name="vm-differences"></a>VM の相違点

| 特徴量 | Azure (グローバル) | Azure Stack |
| --- | --- | --- |
| 仮想マシン イメージ | Azure Marketplace には、VM の作成に使用できるイメージがあります。 Azure Marketplace で入手できるイメージの一覧を参照するには、[Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/compute?subcategories=virtual-machine-images&page=1) のページを参照してください。 | 既定では、Azure Stack Marketplace に使用可能なイメージはありません。 Azure Stack のクラウド管理者は、ユーザーがイメージを使用する前に、Azure Stack Marketplace に対してイメージの発行またはダウンロードを行う必要があります。 |
| 仮想マシン サイズ | Azure では、幅広いサイズの VM がサポートされます。 提供されているサイズとオプションの詳細については、[Windows VM のサイズ](/azure/virtual-machines/sizes)と [Linux VM のサイズ](/azure/virtual-machines/linux/sizes)に関するトピックを参照してください。 | Azure Stack は、Azure で使用できる VM のサイズのサブセットをサポートしています。 サポートされているサイズの一覧を参照するには、この記事の [VM のサイズ](#vm-sizes)についてのセクションを参照してください。 |
| 仮想マシンのクォータ | [クォータ制限](/azure/azure-subscription-service-limits#service-specific-limits)は Microsoft によって設定されます | Azure Stack のクラウド管理者は、VM をユーザーに提供する前にクォータを割り当てる必要があります。 |
| 仮想マシン拡張機能 |Azure では、幅広い VM 拡張機能がサポートされます。 使用できる拡張機能については、[VM の拡張機能と機能](/azure/virtual-machines/windows/extensions-features)に関する記事を参照してください。| Azure Stack は、Azure で使用できる拡張機能のサブセットをサポートしており、拡張機能それぞれに特定のバージョンがあります。 Azure Stack のクラウド管理者は、ユーザーにどの拡張機能を提供するかを選択することができます。 サポートされている拡張機能の一覧を参照するには、この記事の [VM の拡張機能](#vm-extensions)についてのセクションを参照してください。 |
| 仮想マシンのネットワーク | テナントの VM に割り当てられたパブリック IP アドレスは、インターネット経由でアクセスできます。<br><br><br>Azure VM には固定の DNS 名があります。 | テナントの VM に割り当てられたパブリック IP アドレスにアクセスできるのは、Azure Stack Development Kit 環境内のみです。 ユーザーは、Azure Stack で作成される VM に接続するためには、[RDP](../asdk/asdk-connect.md#connect-to-azure-stack-using-rdp) または [VPN](../asdk/asdk-connect.md#connect-to-azure-stack-using-vpn) 経由で Azure Stack Development Kit にアクセスできる必要があります。<br><br>特定の Azure Stack インスタンス内に作成された VM には、クラウド管理者が構成した値に基づく DNS 名があります。 |
| 仮想マシンのストレージ | [マネージド ディスク](/azure/virtual-machines/windows/managed-disks-overview)をサポートしています。 | マネージド ディスクは、バージョン 1808 以降の Azure Stack でサポートされています。 |
| 仮想マシン ディスクのパフォーマンス | ディスクの種類とサイズによって異なります。 | ディスクが接続されている VM の VM サイズによって異なります。 詳細については、「[Azure Stack でサポートされている VM のサイズ](azure-stack-vm-sizes.md)」の記事を参照してください。
| API のバージョン | Azure では常に、すべての VM 機能について最新の API のバージョンが用意されます。 | Azure Stack では特定の Azure サービスがサポートされ、それらのサービスについて特定の API バージョンがサポートされます。 サポートされている API バージョンの一覧を参照するには、この記事の [API バージョン](#api-versions)についてのセクションを参照してください。 |
| Azure Instance Metadata Service | Azure Instance Metadata Service は、実行中の VM インスタンスに関する情報を提供します。これらの情報を使用して VM の管理とセットアップを行うことができます。  | Azure Stack では、Azure Instance Metadata Service がサポートされません。 |
| 仮想マシン可用性セット|複数の障害ドメイン (リージョンあたり 2 または 3)。<br>複数の更新ドメイン。|複数の障害ドメイン (リージョンあたり 2 または 3)。<br>更新中のワークロードを保護するためのライブ マイグレーションによる単一の更新ドメイン。 テンプレートの互換性のためにサポートされている 20 個の更新ドメイン。<br>VM と可用性セットは、同じ場所とリソース グループに存在する必要があります。|
| 仮想マシン スケール セット|自動スケーリングがサポートされます。|自動スケーリングはサポートされません。<br><br>ポータル、Resource Manager テンプレート、または PowerShell を使用してスケール セットにより多くのインスタンスを追加します。 |
| クラウド監視 | Azure Stack で使用できるストレージ アカウントのプロパティからエンドポイントを選択します。 | [クラウド監視](/windows-server/failover-clustering/deploy-cloud-witness)はフェールオーバー クラスター クォーラム監視の一種であり、Microsoft Azure を使用してクラスター クォーラムで投票を提供します。<br>グローバル Azure のエンドポイントと Azure Stack の比較は次のようになります。<br>グローバル Azure の場合:<br>`https://mywitness.blob.core.windows.net/`<br>Azure Stack の場合:<br>`https://mywitness.blob.<region>.<FQDN>/`|
| 仮想マシンの診断 | Linux VM の診断がサポートされます。 | Linux VM の診断は、Azure Stack でサポートされません。 VM 診断を有効にして Linux VM を展開すると、展開が失敗します。 診断設定で Linux VM の基本メトリックを有効にした場合も、展開が失敗します。 |

## <a name="vm-sizes"></a>VM サイズ

Azure Stack は、リソースの過剰消費を防ぐため、(サーバー ローカルおよびサービス レベルで) リソースの制限を適用します。これらの制限は、他のテナントによるリソース消費の影響を軽減してテナントのエクスペリエンスを改善します。

- VM からのネットワーク送信には、帯域幅の上限が設けられます。 Azure Stack での上限は、Azure での上限と同様です。
- ストレージ リソースの場合、テナントによるストレージ使用のためのリソースの基本的な過剰消費を防ぐため、Azure Stack によってストレージ IOPS (1 秒あたりの入力/出力操作) 制限が実装されます。
- VM ディスクの場合、Azure Stack 上のディスクの IOPS はディスクの種類ではなく VM サイズの関数です。 つまり、Standard_Fs シリーズの VM では、ディスクの種類として SSD と HDD のどちらを選択した場合でも、2 番目のデータ ディスクの IOPS 制限は 2,300 IOPS です。

次の表は、Azure Stack でサポートされている VM とその構成の一覧です。

| Type            | サイズ          | サポートされるサイズの範囲 |
| ----------------| ------------- | ------------------------ |
|汎用  |Basic A        |[A0 - A4](azure-stack-vm-sizes.md#basic-a)                   |
|汎用  |Standard A     |[A0 - A7](azure-stack-vm-sizes.md#standard-a)              |
|汎用  |Av2 シリーズ     |[A1_v2 - A8m_v2](azure-stack-vm-sizes.md#av2-series)     |
|汎用  |D シリーズ       |[D1 - D4](azure-stack-vm-sizes.md#d-series)              |
|汎用  |Dv2 シリーズ     |[D1_v2 - D5_v2](azure-stack-vm-sizes.md#ds-series)        |
|汎用  |DS シリーズ      |[DS1 - DS4](azure-stack-vm-sizes.md#dv2-series)            |
|汎用  |DSv2 シリーズ    |[DS1_v2 - DS5_v2](azure-stack-vm-sizes.md#dsv2-series)      |
|メモリ最適化 |D シリーズ       |[D11 - D14](azure-stack-vm-sizes.md#mo-d)            |
|メモリ最適化 |DS シリーズ      |[DS11 - DS14](azure-stack-vm-sizes.md#mo-ds)|
|メモリ最適化 |Dv2 シリーズ     |[D11_v2 - DS14_v2](azure-stack-vm-sizes.md#mo-dv2)     |
|メモリ最適化 |DSv2 シリーズ    |[DS11_v2 - DS14_v2](azure-stack-vm-sizes.md#mo-dsv2)    |
|コンピューティング最適化|F シリーズ       |[F1 - F16](azure-stack-vm-sizes.md#f-series)    |
|コンピューティング最適化|Fs シリーズ      |[F1s - F16s](azure-stack-vm-sizes.md#fs-series)    |
|コンピューティング最適化|Fsv2 シリーズ    |[F2s_v2 - F64s_v2](azure-stack-vm-sizes.md#fsv2-series)    |
|GPU 対応      |NVv4 シリーズ    |[NVv4](azure-stack-vm-sizes.md#nvv4-series) |

VM のサイズと、それに関連付けられるリソースの量は、Azure Stack と Azure の間で一貫しています。 この一貫性には、作成できるメモリの量、コアの数、データ ディスクの数やサイズが含まれます。 ただし、同じサイズの VM のパフォーマンスは、基になっている特定の Azure Stack 環境の特性によって異なります。

## <a name="vm-extensions"></a>VM 拡張機能

Azure Stack には、拡張機能の小規模なセットが含まれています。 更新プログラムおよび追加の拡張機能は、Marketplace シンジケーションを通じて入手できます。

実際の Azure Stack 環境で利用できる VM 拡張機能の一覧を取得するには、次の PowerShell スクリプトを使用します。

### <a name="az-modules"></a>[Az モジュール](#tab/az1)

```powershell
Get-AzVmImagePublisher -Location local | `
  Get-AzVMExtensionImageType | `
  Get-AzVMExtensionImage | `
  Select Type, Version | `
  Format-Table -Property * -AutoSize
```
### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm1)

```powershell
Get-AzureRmVmImagePublisher -Location local | `
  Get-AzureRmVMExtensionImageType | `
  Get-AzureRmVMExtensionImage | `
  Select Type, Version | `
  Format-Table -Property * -AutoSize
```

---

VM のデプロイで拡張機能のプロビジョニングに時間がかかりすぎる場合は、プロセスを停止して VM の割り当て解除または削除を試みるのではなく、プロビジョニングをタイムアウトさせる必要があります。

## <a name="api-versions"></a>API のバージョン

Azure Stack の VM 機能は、次の API バージョンをサポートしています。

"2017-12-01"、"2017-03-30"、"2016-03-30"、"2015-06-15"

次の PowerShell スクリプトを使用して、実際の Azure Stack 環境で利用できる VM 機能の API バージョンを取得することができます。

### <a name="az-modules"></a>[Az モジュール](#tab/az)

```powershell
Get-AzResourceProvider | `
  Select ProviderNamespace -Expand ResourceTypes | `
  Select * -Expand ApiVersions | `
  Select ProviderNamespace, ResourceTypeName, @{Name="ApiVersion"; Expression={$_}} | `
  where-Object {$_.ProviderNamespace -like "Microsoft.compute"}
```
### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm)

```powershell
Get-AzureRmResourceProvider | `
  Select ProviderNamespace -Expand ResourceTypes | `
  Select * -Expand ApiVersions | `
  Select ProviderNamespace, ResourceTypeName, @{Name="ApiVersion"; Expression={$_}} | `
  where-Object {$_.ProviderNamespace -like "Microsoft.compute"}
```

---

サポートされるリソースの種類と API バージョンの一覧は、クラウド オペレーターが Azure Stack 環境を新しいバージョンに更新した場合は異なっている可能性があります。

## <a name="windows-activation"></a>Windows のライセンス認証

Windows の製品は、製品使用権利およびマイクロソフのライセンス条項に従って使用する必要があります。 Azure Stack は [VM の自動ライセンス認証](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn303421(v%3dws.11)) (AVMA) を使用して Windows Server VM をアクティブ化します。

- Azure Stack のホストは、Windows Server 2016 の AVMA キーを使用して Windows をアクティブ化します。 Windows Server 2012 R2 またはそれ以降を実行する VM はすべて、自動的にアクティブ化されます。
- Windows Server 2012 以前を実行する VM は、自動的にアクティブ化されないので、[MAK ライセンス認証](/previous-versions/tn-archive/ff793438(v=technet.10))を使用してアクティブ化する必要があります。 MAK ライセンス認証を使用するには、自分のプロダクト キーを指定する必要があります。

Microsoft Azure では、KMS ライセンス認証を使用して、Windows VM をアクティブ化します。 Azure Stack から Azure に VM を移行して、アクティブ化の問題があった場合は、[Azure Windows VM のライセンス認証に関する問題のトラブルシューティング](/azure/virtual-machines/windows/troubleshoot-activation-problems)を参照してください。 追加の情報については、Azure サポート チームのブログ記事「[Troubleshooting Windows activation failures on Azure VMs (Azure VM での Windows のライセンス認証の失敗をトラブルシューティングする)](/archive/blogs/mast/troubleshooting-windows-activation-failures-on-azure-vms)」を参照してください。

## <a name="next-steps"></a>次のステップ

[Azure Stack 内で PowerShell を使用して Windows VM を作成する](../user/azure-stack-quick-create-vm-windows-powershell.md)