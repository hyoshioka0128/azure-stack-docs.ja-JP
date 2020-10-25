---
title: Azure Stack Hub での API バージョンのプロファイルの管理
description: Azure Stack Hub での API バージョンのプロファイルについて説明します。
author: sethmanheim
ms.topic: article
ms.date: 08/28/2020
ms.author: sethm
ms.reviewer: sijuman
ms.lastreviewed: 07/24/2019
ms.openlocfilehash: 44638ca39dc5604ee436af5c0fa09c58fadf59eb
ms.sourcegitcommit: 28894abb31423046f6775aadef490c467f1b1820
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/29/2020
ms.locfileid: "89089645"
---
# <a name="manage-api-version-profiles-in-azure-stack-hub"></a>Azure Stack Hub での API バージョンのプロファイルの管理

API プロファイルでは、Azure リソース プロバイダーと Azure REST エンドポイントの API バージョンが指定されます。 API プロファイルを使用して、さまざまな言語でカスタム クライアントを作成できます。 各クライアントは API プロファイルを使用して、適切なリソース プロバイダーと Azure Stack Hub の API バージョンに連絡します。

各リソース プロバイダー API のどのバージョンが Azure Stack Hub と互換性があるかが厳密にわからなくても、Azure リソース プロバイダーと連携するアプリを作成できます。 アプリをプロファイルに整合させるだけで、SDK によって適切な API バージョンに戻されます。

このトピックは、以下に役立ちます。

- Azure Stack Hub の API プロファイルの理解
- API プロファイルを使用してソリューションを開発する方法
- コード固有のガイダンスが提供されている場所

## <a name="summary-of-api-profiles"></a>API プロファイルの概要

- API プロファイルは、一連の Azure リソース プロバイダーとその API バージョンを表すために使用されます。
- API プロファイルは、複数の Azure クラウド間でテンプレートを作成するユーザー向けに作成されました。 プロファイルは、互換性のある安定したインターフェイスを提供します。
- プロファイルは、1 年に 4 回リリースされます。
- 次の 3 つのプロファイル名前付け規則が使用されます。
  - **latest**  
        グローバル Azure でリリースされた最新の API バージョンを含みます。
  - **yyyy-mm-dd-hybrid**  
    年に 2 回リリースされます。このリリースでは、複数のクラウド間での一貫性と安定性に重点が置かれています。 このプロファイルは Azure Stack Hub との最適な互換性を目指しています。
  - **yyyy-mm-dd-profile** <br>
    最適な安定性と最新の機能のバランスを取ります。

## <a name="azure-api-profiles-and-azure-stack-hub-compatibility"></a>Azure API プロファイルと Azure Stack Hub の互換性

最新の API プロファイルは Azure Stack Hub との互換性がありません。 次の名前付け規則を使用して、Azure Stack Hub ソリューションにどのプロファイルを使用するか識別します。

**最新**  
このプロファイルはグローバル Azure にある最新の API バージョンであり、Azure Stack Hub では機能しません。 **最新** このプロファイルには､最も多くの重大な変更が含まれています｡ このプロファイルでは､安定性および他のクラウドとの互換性は配慮されていません｡ 最新の API バージョンを使用する場合は､**最新**プロファイルを使用してください。

**yyyy-mm-dd-hybrid**  
このプロファイルは､毎年 3月と 9 月にリリースされます｡ 最適な安定性と、さまざまなクラウドとの互換性があり、グローバル Azure と Azure Stack Hub を対象に設計されています。 このプロファイルに記載されている Azure API バージョンは、Azure Stack Hub に記載されているものと同じです。 このプロファイルを利用して､ハイブリッド クラウド ソリューション向けのコードを開発します。

**yyyy-mm-dd-profile**  
このプロファイルは､グローバル Azure 向けに 6 月と 12 月にリリースされます｡ Azure Stack Hub では機能せず、一般的に、重大な変更が多数含まれています。 最適な安定性と最新の機能のバランスを取っていますが、**最新**プロファイルとこのプロファイルの違いは､API のリリース時期に関係なく、**最新**のプロファイルは常に最新の API バージョンで構成されることです。 たとえば､Compute API 用に新しい API バージョンが明日作成されるとすると､**最新**プロファイルには､その API バージョンが記載されますが､このプロファイルはすでに存在するため、**yyyy-mm-dd-profile** プロファイルには記載されません｡ **yyyy-mm-dd-profile** は、6 月または 12 月の前にリリースされた最新のバージョンが対応します｡

## <a name="azure-resource-manager-api-profiles"></a>Azure Resource Manager API プロファイル

Azure Stack Hub は、グローバル Azure にある API バージョンのうちの最新バージョンを使用しません。 ソリューションを作成する場合は、Azure Stack Hub と互換性のある Azure の各リソース プロバイダーの API バージョンを特定する必要があります。

すべてのリソース プロバイダーや Azure Stack Hub でサポートされている特定のバージョンを調査する代わりに、API プロファイルを使用できます。 プロファイルは、リソース プロバイダーと API バージョンのセットを指定します。 SDK、または SDK でビルドされたツールによって、プロファイル内で指定されたターゲット `api-version` に戻されます。 API プロファイルでは、テンプレート全体に適用されるプロファイル バージョンを指定できます。 実行時に、Azure Resource Manager によってリソースの適切なバージョンが選択されます。

API プロファイルは、Azure Resource Manager を使用するツール (PowerShell、Azure CLI、SDK で提供されるコード、Microsoft Visual Studio など) と連携します。 ツールや SDK はプロファイルを使用して、アプリのビルド時にインクルードするモジュールやライブラリのバージョンを読み取ります。

たとえば、PowerShell を使用して **Microsoft.Storage** リソース プロバイダーを使ってストレージ アカウントを作成する場合 (ここでは **api-version** 2016-03-30 と、**Microsoft.Compute** リソース プロバイダーと **api-version** 2015-12-01 を使用した VM がサポートされます)、ストレージ用にどの PowerShell モジュールが 2016-03-30 をサポートしているか、コンピューティング用にどのモジュールが 2015-02-01 をサポートしているか調べてからインストールする必要があります。 その代わりに、プロファイルを使用することができます。 コマンドレット `Install-Profile <profilename>` を使用すると、PowerShell によって適切なバージョンのモジュールが読み込まれます。

同様に、Python SDK を使用して Python ベースのアプリをビルドする場合、プロファイルを指定できます。 SDK によって、スクリプトで指定済みのリソース プロバイダーの適切なモジュールが読み込まれます。

開発者として、ソリューションの作成に集中できます。 API バージョン、リソース プロバイダー、連携させるクラウドを調査する代わりに、プロファイルを使用すれば、そのプロファイルに対応するすべてのクラウドにわたってコードが機能することを把握できます。

## <a name="api-profile-code-samples"></a>API プロファイルのコード サンプル

プロファイルを使用して、Azure Stack Hub を使って目的の言語でソリューションを統合するためのコード サンプルが用意されています。 現在、次の言語用のガイダンスとサンプルを使用できます。

- **.NET** <br>
.NET API プロファイルを使用して、リソース プロバイダーのパッケージに含まれる各リソースの種類の、最も安定した最新バージョンを取得します。 詳細については、[Azure Stack Hub での .NET による API バージョンのプロファイルの使用](azure-stack-version-profiles-net.md)についてのページを参照してください。
- **PowerShell**  
PowerShell ギャラリーから入手できる **AzureRM.Bootstrapper** モジュールを使用して、API バージョンのプロファイルを操作するために必要な PowerShell コマンドレットを取得します。 詳しくは、「[Use API version profiles for PowerShell (PowerShell 向け API バージョン プロファイルの使用)](../operator/azure-stack-powershell-install.md?view=azs-2002)」をご覧ください。
- **Azure CLI**  
Azure Stack Hub 固有の API バージョンのプロファイルを使用するようにお使いの環境の構成を更新します。 詳細については、[Azure CLI での API バージョンのプロファイルの使用](azure-stack-version-profiles-azurecli2.md)に関するページを参照してください。
- **Go**  
GO SDK では、プロファイルはさまざまな種類のリソースと多様なサービスのさまざまなバージョンの組み合わせが用意されています。 プロファイルは、**YYYY-MM-DD** 形式のバージョンと共に profiles/path で利用できます。 詳しくは、[GO による API バージョンのプロファイルの使用](azure-stack-version-profiles-go.md)に関する記事をご覧ください。
- **Ruby**  
Azure Stack Hub Resource Manager 向けの Ruby SDK には、インフラストラクチャの構築と管理に役立つツールが用意されています。 SDK のリソース プロバイダーには､Ruby 言語を使用したコンピュートや仮想ネットワーク､ストレージがあります。 詳しくは、[Ruby による API バージョンのプロファイルの使用](azure-stack-version-profiles-ruby.md)に関する記事をご覧ください。
- **Python**  
Python SDK では、Azure Stack Hub やグローバル Azure などの異なるクラウド プラットフォームをターゲットとする API バージョン プロファイルをサポートします。 ハイブリッド クラウド向けのソリューションを作成するには、API プロファイルを使用します。 詳しくは、[Python による API バージョンのプロファイルの使用](azure-stack-version-profiles-python.md)に関する記事をご覧ください。
- **Node.js**  
Azure Stack Hub Resource Manager 向けの Node.js SDK には、インフラストラクチャの構築と管理に役立つツールが用意されています。 詳細については、[Node.js での API バージョンのプロファイルの使用](azure-stack-version-profile-nodejs.md)に関するページを参照してください。

## <a name="next-steps"></a>次のステップ

- [PowerShell for Azure Stack Hub をインストールする](../operator/azure-stack-powershell-install.md)
- [Azure Stack Hub ユーザーの PowerShell 環境の構成](azure-stack-powershell-configure-user.md)
- [プロファイルでサポートされているリソース プロバイダーの API バージョンの詳細を確認します](azure-stack-profiles-azure-resource-manager-versions.md)。
