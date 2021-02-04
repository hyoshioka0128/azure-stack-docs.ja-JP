---
title: Azure Stack Hub ユーザーに対して Azure CLI を有効にする
description: Azure Stack Hub でクロスプラットフォーム コマンド ライン インターフェイス (CLI) を使用してリソースの管理およびデプロイを行えるようにする方法について学習します。
author: mattbriggs
ms.topic: article
ms.date: 2/1/2021
ms.author: mabrigg
ms.lastreviewed: 05/16/2019
ms.openlocfilehash: b9df823c61a9c0c54975188d15046ea4d73f9ca4
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99246046"
---
# <a name="enable-azure-cli-for-azure-stack-hub-users"></a>Azure Stack Hub ユーザーに対して Azure CLI を有効にする

Azure Stack Hub のユーザーに CA ルート証明書を提供することで、Azure Stack Hub のユーザーは自分の開発マシン上で Azure CLI を有効にすることができます。 ユーザーが CLI を使用してリソースを管理するには証明書が必要です。

 - ユーザーが Azure Stack Development Kit (ASDK) の外部のワークステーションから CLI を使用する場合は、**Azure Stack Hub の CA ルート証明書** が必要です。  

 - **仮想マシン (VM) のエイリアス エンドポイント** では、VM をデプロイするときに、1 つのパラメーターとしてイメージ発行者、プラン、SKU、バージョンを参照する "UbuntuLTS" または "Win2012Datacenter" などのエイリアスを提供します。  

これらの値の取得方法については、以下のセクションで説明します。

## <a name="export-the-azure-stack-hub-ca-root-certificate"></a>Azure Stack Hub の CA ルート証明書をエクスポートする

統合システムを使用している場合は、CA ルート証明書をエクスポートする必要はありません。 ASDK で CA ルート証明書をエクスポートする必要があります。

PEM 形式で ASDK ルート証明書をエクスポートするには、サインインして次のスクリプトを実行します。

```powershell
$label = "AzureStackSelfSignedRootCert"
Write-Host "Getting certificate from the current user trusted store with subject CN=$label"
$root = Get-ChildItem Cert:\CurrentUser\Root | Where-Object Subject -eq "CN=$label" | select -First 1
if (-not $root)
{
    Write-Error "Certificate with subject CN=$label not found"
    return
}

Write-Host "Exporting certificate"
Export-Certificate -Type CERT -FilePath root.cer -Cert $root

Write-Host "Converting certificate to PEM format"
certutil -encode root.cer root.pem
```

## <a name="set-up-the-vm-aliases-endpoint"></a>VM のエイリアス エンドポイントを設定する

Azure Stack Hub オペレーターは、VM のエイリアス ファイルをホストする、公開されているエンドポイントを設定する必要があります。 VM のエイリアス ファイルは、イメージの共通名が指定された JSON ファイルです。 この名前は、Azure CLI パラメーターとして VM をデプロイするときに使用します。  

エイリアス ファイルにエントリを追加する前に、[Azure Marketplace からイメージをダウンロード](azure-stack-download-azure-marketplace-item.md)するか、[独自のカスタム イメージを発行](azure-stack-add-vm-image.md)してください。 カスタム イメージを発行する場合は、発行時に指定した発行者、オファー、SKU、バージョン情報をメモしておいてください。 マーケットプレースのイメージである場合は、`Get-AzureVMImage` コマンドレットを使用して情報を表示できます。  

多くの一般的なイメージの別名を含む[サンプル エイリアス ファイル](https://raw.githubusercontent.com/Azure/azure-rest-api-specs/master/arm-compute/quickstart-templates/aliases.json)が使用できます。 それを出発点として使用できます。 このファイルを CLI クライアントがアクセスできる場所にホストします。 そのためには、BLOB ストレージ アカウント内でファイルをホストし、ユーザーと URL を共有する方法があります。

1. GitHub から[サンプル ファイル](https://raw.githubusercontent.com/Azure/azure-rest-api-specs/master/arm-compute/quickstart-templates/aliases.json)をダウンロードします。
2. Azure Stack Hub にストレージ アカウントを作成します。 完了したら、BLOB コンテナーを作成します。 アクセス ポリシーを [パブリック] に設定します。  
3. その新しいコンテナーに JSON ファイルをアップロードします。 完了すると、BLOB の URL を表示できるようになります。 BLOB 名を選択してから、BLOB プロパティから URL を選択します。

## <a name="next-steps"></a>次のステップ

- [Azure CLI を使用したテンプレートのデプロイ](../user/azure-stack-deploy-template-command-line.md )
- [PowerShell で接続する](powershell-install-az-module.md)
- [ユーザー アクセス許可の管理](azure-stack-manage-permissions.md)
