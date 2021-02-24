---
title: Azure Stack 1905 リリース ノート | Microsoft Docs
description: Azure Stack 統合システムの 1905 更新プログラムについて説明します
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/14/2019
ms.author: sethm
ms.reviewer: hectorl
ms.lastreviewed: 06/14/2019
ROBOTS: NOINDEX
ms.openlocfilehash: 0f26f27cea7fabaaec9c6fd1507294f9bcbe3372
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99248781"
---
# <a name="azure-stack-1905-update"></a>Azure Stack 1905 更新プログラム

*適用対象:Azure Stack 統合システム*

この記事では、1905 更新プログラム パッケージの内容について説明します。 この更新プログラムには、このリリースの Azure Stack に対する新機能、機能強化、および修正が含まれています。 この記事には、次の情報が含まれています。

- [新機能、機能強化、修正、およびセキュリティ更新プログラムの説明](#whats-in-this-update)
- [計画の更新](#update-planning)

> [!IMPORTANT]  
> 更新プログラム パッケージは、Azure Stack 統合システム専用です。 Azure Stack Development Kit にこの更新プログラム パッケージは適用しないでください。

## <a name="build-reference"></a>ビルドのリファレンス

Azure Stack 1905 更新プログラムのビルド番号は **1.1905.0.40** です。

### <a name="update-type"></a>更新の種類

Azure Stack 1905 更新プログラムのビルドの種類は **完全** です。 そのため、1905 更新プログラムは、1903 や 1904 のような高速更新プログラムよりもランタイムが長くなります。 完全な更新プログラムの正確なランタイムは、Azure Stack インスタンスに含まれているノード数、テナントのワークロードごとにシステムで使用される容量、システムのネットワーク接続 (インターネットに接続されている場合)、システムのハードウェア構成によって異なります。 内部テストでは、1905 更新プログラムのランタイムが次のように予測されました。4 ノード - 35 時間、8 ノード - 45 時間、12 ノード - 55 時間、16 ノード - 70 時間。 1905 のランタイムがこの予測値よりも長くなることは一般的ではなく、更新が失敗した場合を除き、Azure Stack オペレーターによるアクションは不要です。 更新プログラムのビルドの種類については、「[Azure Stack での更新プログラムの管理概要](../azure-stack-updates.md)」を参照してください。

## <a name="whats-in-this-update"></a>この更新プログラムの新機能

<!-- The current theme (if any) of this release. -->

<!-- What's new, also net new experiences and features. -->

- この更新プログラムでは、Azure Stack の更新エンジンはスケール ユニット ノードのファームウェアを更新できます。 これには、ハードウェア パートナーからの準拠している更新プログラム パッケージが必要です。 使用できるかどうか詳しくは、ハードウェア パートナーに問い合わせてください。

- Windows Server 2019 がサポートされるようになっており、Azure Stack Marketplace を通した配信に利用できます。
この更新プログラムでは、2016 ホスト上で Windows Server 2019 を正常にアクティブにできるようになりました。

- 新しい [Azure アカウント Visual Studio Code 拡張機能](../../user/azure-stack-dev-start-vscode-azure.md)を使うと、開発者はサブスクリプションにログインして表示することで、Azure Stack や他の多数のサービスをターゲットにできます。 Azure アカウント拡張機能は、Azure Active Directory (Azure AD) 環境と AD FS 環境の両方で動作し、Visual Studio Code ユーザー設定で必要な変更はわずかです。 Visual Studio Code には、この環境で実行するために、サービス プリンシパルにアクセス許可を付与する必要があります。 これを行うには、ID スクリプトをインポートし、[Azure Stack のマルチテナント](../azure-stack-enable-multitenancy.md)に指定されているコマンドレットを実行します。 これには、ホーム ディレクトリの更新と、各ディレクトリのゲスト テナント ディレクトリの登録が必要です。 1905 以降に更新後、Visual Studio Code サービス プリンシパルが含まれているホーム ディレクトリ テナントを更新するように、アラートが表示されます。 

### <a name="improvements"></a>機能強化

<!-- Changes and product improvements with tangible customer-facing value. -->
- Azure Stack で TLS 1.2 を適用する一部として、次の拡張機能がこれらのバージョンに更新されています。

  - microsoft.customscriptextension-arm-1.9.3
  - microsoft.iaasdiagnostics-1.12.2.2
  - microsoft.antimalware-windows-arm-1.5.5.9
  - microsoft.dsc-arm-2.77.0.0
  - microsoft.vmaccessforlinux-1.5.2

  これらのバージョンの拡張機能をすぐにダウンロードして、将来のリリースで TLS 1.2 が適用されるときに、拡張機能の新しいデプロイが失敗しないようにしてください。 常に **autoUpgradeMinorVersion=true** を設定し、拡張機能に対するマイナー バージョンの更新プログラム (たとえば、1.8 から 1.9) が自動的に実行されるようにします。

- Azure Stack ポータルの新しい **ヘルプとサポートの概要** により、オペレーターは簡単にサポート オプションを確認したり、専門家の支援を得たり、Azure Stack の詳細について学習したりできます。 統合システムでは、サポート リクエストを作成するときは Azure Stack サービスが事前に選択されます。 グローバルな Azure portal を使うのではなく、このエクスペリエンスを使ってチケットを送信することをお客様に強くお勧めします。 詳しくは、「[Azure Stack Help and Support](../azure-stack-help-and-support-overview.md)」(Azure Stack のヘルプとサポート) をご覧ください。

- 複数の Azure Active Directory を ([このプロセス](../azure-stack-enable-multitenancy.md)で) オンボードすると、特定の更新プログラムが発生したとき、または Azure AD サービス プリンシパルの認可に対する変更によって権限がなくなったときに、スクリプトの再実行が無視されることがあります。 これにより、特定の機能に対するアクセスのブロックから、元の問題まで追跡することが困難なさらに個別の障害まで、さまざまな問題が発生する可能性があります。 これを防ぐため、1905 では、これらのアクセス許可をチェックし、特定の構成の問題が見つかったときはアラートを作成する、新しい機能が導入されています。 この検証は 1 時間ごとに実行され、問題を解決するために必要な修復アクションが表示されます。 すべてのテナントが正常な状態になると、アラートは閉じます。

- サービスのフェールオーバーの間の、インフラストラクチャのバックアップ操作の信頼性の向上。

- 認証に [Azure Active Directory 認証ライブラリ](/azure/active-directory/develop/active-directory-authentication-libraries) (ADAL) を使用する、新しいバージョンの [Azure Stack Nagios プラグイン](../azure-stack-integrate-monitor.md#integrate-with-nagios)を利用できます。 プラグインでは、Azure AD と Active Directory フェデレーション サービス (ADFS) での Azure Stack のデプロイもサポートされるようになっています。 詳しくは、[Nagios プラグインの交換](https://exchange.nagios.org/directory/Plugins/Cloud/Monitoring-AzureStack-Alerts/details)に関するサイトをご覧ください。

- Azure Stack のすべての最新機能がサポートされている新しいハイブリッド プロファイル **2019-03-01-Hybrid** がリリースされました。 **2019-03-01-Hybrid** プロファイルは、Azure PowerShell と Azure CLI の両方でサポートされています。 .NET、Ruby、Node.js、Go、Python の SDK で、**2019-03-01-Hybrid** プロファイルをサポートするパッケージが公開されています。 その変更を反映するように、それぞれのドキュメントといくつかのサンプルが更新されています。

- [Node.js SDK](https://www.npmjs.com/search?q=2019-03-01-hybrid) では、API プロファイルがサポートされるようになっています。 **2019-03-01-Hybrid** プロファイルをサポートするパッケージが、公開されます。

- 1905 Azure Stack 更新プログラムでは、プラットフォームの信頼性とサポート性を向上させるために次の 2 つの新しいインフラストラクチャ ロールを追加しています。

  - **インフラストラクチャ リング**: 現在、xrp などは独自に指定したインフラストラクチャ VM が必要ですが、今後はインフラストラクチャ リングがこのような既存のインフラストラクチャ ロールのコンテナー化されたバージョンをホストします。 これにより、プラットフォームの信頼性が向上し、Azure Stack で必要とするインフラストラクチャ VM の数が少なくなります。 これにより、Azure Stack のインフラストラクチャ ロールの全体的なリソース消費が今後大幅に削減されます。
  - **サポート リング**: 今後は、お客様向けに強化されたサポートのシナリオを処理するためにサポート リングが使用されます。  

  さらに、このロールの可用性向上のため、ドメイン コントローラー VM にインスタンスを追加しました。

  この変更により、Azure Stack インフラストラクチャのリソースの消費量が次のように増加します。
  
    | Azure Stack SKU | コンピューティング消費量の増加 | メモリ消費量の増加 |
    | -- | -- | -- |
    |4 ノード|22 vCPU|28 GB|
    |8 ノード|38 vCPU|44 GB|
    |12 ノード|54 vCPU|60 GB|
    |16 ノード|70 vCPU|76 GB|
  
### <a name="changes"></a>[変更点]

- 計画的および非計画的なメンテナンス シナリオでの信頼性と可用性を向上させるため、Azure Stack では、ドメイン サービス用に新しいインフラストラクチャ ロール インスタンスが追加されています。

- この更新プログラムでは、修復および追加のノード操作の間に、ハードウェアが検証されて、スケール ユニット内のスケール ユニット ノードが同種であることが確認されます。

- スケジュールされたバックアップが完了できず、定義されている保持期間を超えた場合、インフラストラクチャ バックアップ コントローラーにより、少なくとも 1 つの成功したバックアップが保持されていることが確認されます。 

### <a name="fixes"></a>修正

<!-- Product fixes that came up from customer deployments worth highlighting, especially if there is an SR/ICM associated to it. -->

- スケール ユニット内のノードの再起動後に **コンピューティング ホスト エージェント** の警告が表示される問題が修正されました。

- フィルター適用時に正しくない結果が表示され、発行元フィルターで発行元の名前が重複して表示されるという、管理者ポータルでのマーケットプレース管理の問題が修正されました。 また、パフォーマンスが強化されて、結果の表示が速くなりました。

- 外部ストレージの場所へのアップロードが完了する前に、新しい利用可能なバックアップが一覧表示される、利用可能バックアップ ブレードでの問題が修正されました。 利用可能なバックアップは、ストレージの場所に正常にアップロードされた後で、一覧に表示されるようになります。 

<!-- ICM: 114819337; Task: 4408136 -->
- バックアップ操作中の回復キーの取得に関する問題が修正されました。 

<!-- Bug: 4525587 -->
- オペレーター ポータルでバージョンが "未定義" と表示される、OEM 更新プログラムでの問題が修正されました。

### <a name="security-updates"></a>セキュリティ更新プログラム

Azure Stack のこの更新でのセキュリティ更新プログラムについては、「[Azure Stack security updates](../release-notes-security-updates.md)」 (Azure Stack のセキュリティ更新プログラム) をご覧ください。

## <a name="update-planning"></a>計画の更新

更新プログラムを適用する前に、必ず次の情報を確認してください。

- [既知の問題](known-issues-1905.md)
- [セキュリティ更新プログラム](../release-notes-security-updates.md)
- [更新プログラム適用前後のアクティビティのチェックリスト](../release-notes-checklist.md)

## <a name="download-the-update"></a>更新プログラムをダウンロードする

Azure Stack 1905 更新プログラム パッケージは、[Azure Stack ダウンロード ページ](https://aka.ms/azurestackupdatedownload)からダウンロードできます。 ダウンローダー ツールを使用する場合は、ダウンロード ディレクトリからキャッシュされたコピーではなく最新バージョンを使用してください。

## <a name="hotfixes"></a>修正プログラム

Azure Stack では、修正プログラムが定期的にリリースされます。 Azure Stack を 1905 に更新する前に、必ず 1904 用の最新の Azure Stack 修正プログラムをインストールしてください。

Azure Stack 修正プログラムを適用できるのは Azure Stack 統合システムのみです。ASDK には修正プログラムをインストールしないでください。

### <a name="before-applying-the-1905-update"></a>1905 更新プログラムを適用する前

Azure Stack の 1905 リリースは、次の修正プログラムが適用された 1904 リリースに適用する必要があります。

<!-- One of these. Either no updates at all, nothing is required, or the LATEST hotfix that is required-->
- [Azure Stack 修正プログラム 1.1904.4.45](https://support.microsoft.com/help/4505688)

### <a name="after-successfully-applying-the-1905-update"></a>1905 更新プログラムの適用に成功した後

この更新プログラムをインストールした後、適用可能な修正プログラムがあればインストールします。 詳細については、[サービス ポリシー](../azure-stack-servicing-policy.md)に関する記事を参照してください。

<!-- One of these. Either no updates at all, nothing is required, or the LATEST hotfix that is required-->
- [Azure Stack 修正プログラム 1.1905.3.48](https://support.microsoft.com/help/4510078)

## <a name="automatic-update-notifications"></a>自動更新通知

インフラストラクチャ ネットワークからインターネットにアクセスできるシステムをご利用の場合は、オペレーター ポータルに "**利用可能な更新プログラムがあります**" というメッセージが表示されます。 インターネットにアクセスできないシステムでは、対応する .xml を含む .zip ファイルをダウンロードしてインポートできます。

> [!TIP]  
> 以下の *RSS* または *Atom* フィードに登録して、Azure Stack 修正プログラムの最新情報を入手してください。
>
> - [RSS](https://support.microsoft.com/app/content/api/content/feeds/sap/en-us/32d322a8-acae-202d-e9a9-7371dccf381b/rss)
> - [Atom](https://support.microsoft.com/app/content/api/content/feeds/sap/en-us/32d322a8-acae-202d-e9a9-7371dccf381b/atom)

## <a name="next-steps"></a>次のステップ

- Azure Stack での更新プログラム管理の概要については、「[Azure Stack での更新プログラムの管理概要](../azure-stack-updates.md)」を参照してください。  
- Azure Stack に更新プログラムを適用する方法については、「[Azure Stack で更新を適用する](../azure-stack-apply-updates.md)」を参照してください。
- Azure Stack 統合システムのサービス ポリシーについて、およびサポートを受けられる状態にシステムを維持するために必要な作業について確認するには、「[Azure Stack サービス ポリシー](../azure-stack-servicing-policy.md)」を参照してください。  
- 特権エンドポイント (PEP) を使用して更新プログラムを監視および再開するには、「[特権エンドポイントを使用して Azure Stack での更新プログラムをモニターする](../azure-stack-monitor-update.md)」をご覧ください。  

