---
title: サービスを使用する場合やアプリを作成する場合の Azure Stack Hub と Azure の違い
description: サービスを使用する場合やアプリを作成する場合の Azure と Azure Stack Hub の違いについて説明します。
author: sethmanheim
ms.topic: overview
ms.date: 09/21/2020
ms.author: sethm
ms.lastreviewed: 12/27/2019
ms.openlocfilehash: 25d836bece262f881901df6c62b5dc8f4aeaf11d
ms.sourcegitcommit: dabbe44c3208fbf989b7615301833929f50390ff
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/22/2020
ms.locfileid: "90946457"
---
# <a name="differences-between-azure-stack-hub-and-azure-when-using-services-and-building-apps"></a>サービスを使用する場合やアプリを作成する場合の Azure Stack Hub と Azure の違い

Azure Stack Hub のサービスを使用する場合やアプリを作成する場合、Azure Stack Hub と Azure の違いを理解しておくことが重要です。 この記事では、Azure Stack Hub をハイブリッド クラウド開発環境として使用する際のさまざまな機能と重要な考慮事項について説明します。

## <a name="overview"></a>概要

Azure Stack Hub は、会社またはサービス プロバイダーのデータセンターからの Azure サービスの使用を可能にするハイブリッド クラウド プラットフォームです。 Azure Stack Hub でアプリを作成し、Azure Stack Hub、Azure、または Azure ハイブリッド クラウドにデプロイすることができます。

Azure Stack Hub オペレーターは、ユーザーに使用可能なサービスと、サポートを受ける方法を知らせます。 これらのサービスはカスタマイズされたプランやオファーを通じて提供されます。

[Azure に関する技術的なドキュメントの内容](/azure)は、Azure Stack Hub ではなく、Azure サービス用にアプリが開発されていることを前提としています。 アプリを作成して Azure Stack Hub にデプロイする場合は、次のようないくつかの主な違いを理解する必要があります。

* Azure Stack Hub では、Azure で利用可能なサービスと機能のサブセットが提供されます。
* 会社またはサービス プロバイダーは、提供するサービスを選択できます。 使用可能なオプションには、カスタマイズされたサービスまたはアプリケーションが含まれる場合があります。 独自のカスタマイズされたドキュメントが提供される場合もあります。
* 適切な Azure Stack Hub 固有のエンドポイント (ポータル アドレスや Azure Resource Manager エンドポイントの URL など) を使用します。
* Azure Stack Hub でサポートされている PowerShell および API のバージョンを使用する必要があります。 サポートされているバージョンを使用すると、確実に Azure Stack Hub と Azure の両方でアプリが動作するようになります。

## <a name="cheat-sheet-high-level-differences"></a>チート シート:大まかな違い

次の表では、Azure Stack Hub と Azure の大まかな違いを示します。 Azure Stack Hub 向けに開発する場合や、Azure Stack Hub サービスを使用する場合は以下の相違点に注意してください。

| 領域 | Azure (グローバル) | Azure Stack Hub |
| -------- | ------------- | ----------|
| 運用担当(オペレーター) | Microsoft | 組織またはサービス プロバイダー。|
| サポートに関する連絡先 | Microsoft | 統合システムについては、(組織またはサービス プロバイダーの) Azure Stack Hub オペレーターにお問い合わせください。<br><br>Azure Stack Development Kit (ASDK) のサポートについては、[Microsoft フォーラム](https://social.msdn.microsoft.com/Forums/en-US/home?forum=AzureStack)にアクセスして確認してください。 開発キットは評価環境であるため、Microsoft サポートを通じた正式なサポートは提供されていません。
| 利用可能なサービス | [Azuer 製品](https://azure.microsoft.com/services/?b=17.04b)の一覧を参照してください。 利用可能なサービスは Azure リージョンによって異なります。 | Azure Stack Hub では Azure サービスのサブセットがサポートされます。 提供される実際のサービスは、組織またはサービス プロバイダーによる選択内容によって異なります。
| Azure Resource Manager のエンドポイント* | `https://management.azure.com` | Azure Stack Hub 統合システムの場合は、Azure Stack Hub オペレーターによって提供されたエンドポイントを使用します。<br><br>開発キットの場合は、`https://management.local.azurestack.external` を使用します。
| ポータル URL* | [https://portal.azure.com](https://portal.azure.com) | Azure Stack Hub 統合システムの場合は、Azure Stack Hub オペレーターによって提供される URL を使用します。<br><br>開発キットの場合は、`https://portal.local.azurestack.external` を使用します。
| リージョン | デプロイ先のリージョンを選択することができます。 | Azure Stack Hub 統合システムの場合は、システムで利用可能なリージョンを使用します。<br><br>Azure Stack Development Kit (ASDK) の場合、リージョンは常に**ローカル**になります。
| リソース グループ | リソース グループは複数のリージョンにまたがることができます。 | 統合システムと開発キットのいずれも、リージョンは 1 つのみです。
|サポートされている名前空間、リソースの種類、および API のバージョン | 最新 (またはまだ非推奨ではない以前のバージョン)。 | Azure Stack Hub では特定のバージョンがサポートされます。 この記事の「[バージョンの要件](#version-requirements)」セクションを参照してください。
| | |

*Azure Stack Hub オペレーターの場合、詳細については、[管理者ポータルの使用](../operator/azure-stack-manage-portals.md)に関する記事および[管理の基本](../operator/azure-stack-manage-basics.md)に関する記事をご覧ください。

## <a name="helpful-tools-and-best-practices"></a>便利なツールとベスト プラクティス

Microsoft では、Azure Stack Hub 向けの開発に役立つツールとガイダンスが提供されています。

| 推奨 | References |
| -------- | ------------- |
| 開発者用ワークステーションに適切なツールをインストールします。 | - [PowerShell のインストール](../operator/azure-stack-powershell-install.md)<br>- [ツールのダウンロード](../operator/azure-stack-powershell-download.md)<br>- [PowerShell の構成](azure-stack-powershell-configure-user.md)<br>- [Visual Studio のインストール](azure-stack-install-visual-studio.md)
| 次の項目に関する情報を確認します。<br>- Azure Resource Manager テンプレートに関する考慮事項。<br>- クイックスタート テンプレートを見つける方法。<br>- Azure Stack Hub 向けの開発で Azure を使用する場合に役立つポリシー モジュールの使用。 | [Azure Stack Hub 向けの開発](azure-stack-developer.md) |
| テンプレートのベスト プラクティスを確認して、それに従います。 | [Resource Manager のクイックスタート テンプレート](https://aka.ms/aa6yz42)
| | |

## <a name="version-requirements"></a>バージョンの要件

Azure Stack Hub では、特定のバージョンの Azure PowerShell と Azure サービス API がサポートされます。 アプリを Azure Stack Hub と Azure の両方にデプロイできるようにするには、サポートされているバージョンを使用してください。

正しいバージョンの Azure PowerShell を使用していることを確認するには、[API バージョン プロファイル](azure-stack-version-profiles.md)を使用します。 使用可能な最新の API バージョン プロファイルを確認するには、使用している Azure Stack Hub のビルドを調べます。 この情報は、Azure Stack Hub 管理者から取得できます。

> [!NOTE]
> Azure Stack Development Kit を使用されていて、管理アクセス権がある場合は、「[現在のバージョンの判断](../operator/azure-stack-updates.md)」セクションを参照して、Azure Stack Hub のビルドを確認してください。

その他の API の場合は、次の PowerShell コマンドを実行し、名前空間、リソースの種類、および Azure Stack Hub サブスクリプションでサポートされている API のバージョンを出力します (プロパティ レベルでも違いがある場合があります)。 このコマンドを機能させるには、Azure Stack Hub 環境用に PowerShell が既に[インストール](../operator/azure-stack-powershell-install.md)され、[構成](azure-stack-powershell-configure-user.md)されている必要があります。 Azure Stack Hub オファーのサブスクリプションも必要です。

```powershell
Get-AzureRmResourceProvider | Select ProviderNamespace -Expand ResourceTypes | Select * -Expand ApiVersions | `
Select ProviderNamespace, ResourceTypeName, @{Name="ApiVersion"; Expression={$_}} 
```

出力例 (抜粋):![Get-AzureRmResourceProvider コマンドの出力例](media/azure-stack-considerations/image1.png)

## <a name="next-steps"></a>次のステップ

サービス レベルでの違いの詳細については、以下を参照してください。

* [Azure Stack Hub の VM に関する考慮事項](azure-stack-vm-considerations.md)
* [Azure Stack Hub のストレージに関する考慮事項](azure-stack-acs-differences.md)
* [Azure Stack Hub ネットワークに関する考慮事項](azure-stack-network-differences.md)
* [Azure Stack Hub SQL リソース プロバイダーに関する考慮事項](../operator/azure-stack-sql-resource-provider.md)
