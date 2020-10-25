---
title: Azure Stack Hub でのテンプレート検証ツールの使用
description: テンプレート検証ツールを使用して、Azure Stack Hub にデプロイするテンプレートをチェックします。
author: sethmanheim
ms.topic: article
ms.date: 10/01/2020
ms.author: sethm
ms.reviewer: sijuman
ms.lastreviewed: 12/27/2019
ms.openlocfilehash: 82eb8ce8b8ddc1b4d357b784814c95809924fc15
ms.sourcegitcommit: a1e2003fb9c6dacdc76f97614ff5a26a5b197b49
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/01/2020
ms.locfileid: "91623253"
---
# <a name="use-the-template-validation-tool-in-azure-stack-hub"></a>Azure Stack Hub でのテンプレート検証ツールの使用

テンプレート検証ツールを使用して、Azure Stack Hub に Azure Resource Manager [テンプレート](azure-stack-arm-templates.md)をデプロイする準備ができているかどうかをチェックします。 テンプレート検証ツールは Azure Stack Hub ツールの GitHub リポジトリで入手できます。 Azure Stack Hub ツールをダウンロードするには、「[GitHub からの Azure Stack Hub ツールのダウンロード](../operator/azure-stack-powershell-download.md)」で説明されている手順を行います。

## <a name="overview"></a>概要

テンプレートを検証するには、最初にクラウド機能ファイルを作成してから検証ツールを実行する必要があります。 Azure Stack Hub ツールから、次の PowerShell モジュールを使用します。

- **CloudCapabilities** フォルダー内:**AzureRM.CloudCapabilities.psm1** は、Azure Stack Hub クラウド内のサービスとバージョンを表すクラウド機能 JSON ファイルを作成します。
- **TemplateValidator** フォルダー内:**AzureRM.TemplateValidator.psm1** は、クラウド機能 JSON ファイルを使用して、Azure Stack Hub へのデプロイ用テンプレートをテストします。

## <a name="build-the-cloud-capabilities-file"></a>クラウド機能ファイルをビルドする

テンプレート検証ツールを使用する前に、**AzureRM.CloudCapabilities** PowerShell モジュールを実行して JSON ファイルを構築します。

> [!NOTE]
> 統合システムを更新する場合や、新しいサービスまたは仮想拡張機能を追加する場合は、このモジュールの再実行が必要になります。

1. Azure Stack Hub への接続があることを確認してください。 これらの手順は Azure Stack Development Kit (ASDK) ホストから実行するか、または、[VPN](../asdk/asdk-connect.md#connect-to-azure-stack-using-vpn) を使用してワークステーションから接続できます。
2. **AzureRM.CloudCapabilities** PowerShell モジュールをインポートします。

    ```powershell
    Import-Module .\CloudCapabilities\AzureRM.CloudCapabilities.psm1
    ```

3. **Get-CloudCapabilities** コマンドレットを使用して、サービスのバージョンを取得し、クラウド機能 JSON ファイルを作成します。 `-OutputPath` を指定しない場合、ファイル **AzureCloudCapabilities.json** は現在のディレクトリに作成されます。 実際の Azure の場所を使用してください。

    ```powershell
    Get-AzureRMCloudCapability -Location <your location> -Verbose
    ```

## <a name="validate-templates"></a>テンプレートの検証

次の手順を使用して、**AzureRM.TemplateValidator** PowerShell モジュールを使用してテンプレートを検証します。 独自のテンプレートを使用するか、[Azure Stack Hub クイック スタート テンプレート](https://github.com/Azure/AzureStack-QuickStart-Templates)を使用できます。

1. **AzureRM.TemplateValidator.psm1** PowerShell モジュールをインポートします。

    ```powershell
    cd "c:\AzureStack-Tools-master\TemplateValidator"
    Import-Module .\AzureRM.TemplateValidator.psm1
    ```

2. テンプレート検証ツールを実行します。

    ```powershell
    Test-AzureRMTemplate -TemplatePath <path to template.json or template folder> `
    -CapabilitiesPath <path to cloudcapabilities.json> `
    -Verbose
    ```

テンプレート検証の警告やエラーが、PowerShell コンソールに表示され、ソース ディレクトリ内の HTML ファイルに書き込まれます。 以下のスクリーンショットは、検証レポートの例です。

![テンプレートの検証レポート](./media/azure-stack-validate-templates/image1.png)

### <a name="parameters"></a>パラメーター

テンプレート検証コマンドレットでは、次のパラメーターがサポートされます。

| パラメーター | 説明 | 必須 |
| ----- | -----| ----- |
| `TemplatePath` | Azure Resource Manager テンプレートを再帰的に検索するパスを指定します。 | はい |
| `TemplatePattern` | 照合するテンプレート ファイルの名前を指定します。 | いいえ |
| `CapabilitiesPath` | クラウド機能 JSON ファイルへのパスを指定します。 | はい |
| `IncludeComputeCapabilities` | VM サイズや VM 拡張機能などの IaaS リソースの評価が含まれます。 | いいえ |
| `IncludeStorageCapabilities` | SKU の種類などのストレージ リソースの評価が含まれます。 | いいえ |
| `Report` | 生成される HTML レポートの名前を指定します。 | いいえ |
| `Verbose` | エラーと警告をコンソールに記録します。 | いいえ|

### <a name="examples"></a>例

この例では、ローカル ストレージにダウンロードした [Azure Stack Hub クイック スタート テンプレート](https://github.com/Azure/AzureStack-QuickStart-Templates)をすべて検証します。 また、この例では、仮想マシン (VM) のサイズと ASDK 機能に対する拡張機能も検証します。

```powershell
test-AzureRMTemplate -TemplatePath C:\AzureStack-Quickstart-Templates `
-CapabilitiesPath .\TemplateValidator\AzureStackCloudCapabilities_with_AddOns_20170627.json `
-TemplatePattern MyStandardTemplateName.json `
-IncludeComputeCapabilities `
-Report TemplateReport.html
```

## <a name="next-steps"></a>次のステップ

- [テンプレートを Azure Stack Hub にデプロイする](azure-stack-arm-templates.md)
- [Azure Stack Hub のテンプレートの開発](azure-stack-develop-templates.md)
