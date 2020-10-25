---
title: Azure Stack Hub のリリース ノート
description: 更新プログラムやバグ修正プログラムを含む、Azure Stack Hub 統合システムのリリース ノート。
author: sethmanheim
ms.topic: article
ms.date: 09/28/2020
ms.author: sethm
ms.reviewer: sranthar
ms.lastreviewed: 08/11/2020
ms.openlocfilehash: cff17c92a44bec23d0a10620d6c182ad5240c451
ms.sourcegitcommit: 703be61f2f1565bf478b8c184753869c29e5c33c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/29/2020
ms.locfileid: "91495798"
---
# <a name="azure-stack-hub-release-notes"></a>Azure Stack Hub のリリース ノート

この記事では、Azure Stack Hub 更新プログラム パッケージの内容について説明します。 更新プログラムには、Azure Stack Hub の最新のリリースに対する機能強化と修正が含まれています。

別のバージョンのリリース ノートにアクセスするには、左側の目次の上部にあるバージョン セレクターのドロップダウンを使用します。

::: moniker range=">=azs-1910"
> [!IMPORTANT]  
> この更新プログラム パッケージは、Azure Stack Hub 統合システム専用です。 Azure Stack Development Kit (ASDK) にこの更新プログラム パッケージを適用しないでください。
::: moniker-end
::: moniker range="<azs-1910"
> [!IMPORTANT]  
> お使いの Azure Stack Hub インスタンスが 2 つ前の更新プログラムより古い場合、コンプライアンスに対応していないとみなされます。 [サポートを受けるためには、少なくともサポートされる最小バージョンまで更新する](azure-stack-servicing-policy.md#keep-your-system-under-support)必要があります。
::: moniker-end

## <a name="update-planning"></a>計画の更新

更新プログラムを適用する前に、必ず次の情報を確認してください。

- [既知の問題](known-issues.md)
- [セキュリティ更新プログラム](release-notes-security-updates.md)
- [更新プログラム適用前後のアクティビティのチェックリスト](release-notes-checklist.md)

更新プログラムのトラブルシューティングと更新プロセスの詳細については、[Azure Stack Hub の修正プログラムと更新プログラムについての問題のトラブルシューティング](azure-stack-troubleshooting.md)に関するページを参照してください。

## <a name="download-the-update"></a>更新プログラムをダウンロードする

Azure Stack Hub 更新プログラム パッケージは、[Azure Stack Hub 更新プログラム ダウンローダー ツール](https://aka.ms/azurestackupdatedownload)を使用してダウンロードできます。

<!---------------------------------------------------------->
<!------------------- SUPPORTED VERSIONS ------------------->
<!---------------------------------------------------------->
::: moniker range="azs-2005"
## <a name="2005-build-reference"></a>2005 ビルドのリファレンス

Azure Stack Hub 2005 更新プログラムのビルド番号は **1.2005.6.53** です。

### <a name="update-type"></a>更新の種類

Azure Stack Hub 2005 更新プログラムのビルドの種類は**完全**です。

2005 更新プログラム パッケージは、以前の更新プログラムと比較してサイズが大きくなっています。 サイズが大きいため、ダウンロード時間が長くなります。 この更新プログラムは**準備中**状態の時間が長くなります。オペレーターは以前の更新プログラムよりもこのプロセスの時間が長くなることを想定してください。 内部テストでは、2005 更新プログラムの実行時間が 4 ノードの場合は 13 から 20 時間、8 ノードの場合は 16 から 26 時間、12 ノードの場合は 19 から 32 時間、16 ノードの場合は 22 から 38 時間と予測されました。 更新プログラムの正確なランタイムは一般的に、ご使用のシステムでテナント ワークロードによって使用されている容量、システム ネットワーク接続 (インターネットに接続されている場合)、およびシステム ハードウェアの仕様に左右されます。 実行時間がこの予測値よりも短くなったり長くなったりすることは一般的ではなく、更新が失敗した場合を除き、Azure Stack Hub オペレーターによるアクションは不要です。 この実行時間の概算は 2005 更新プログラムに固有であるため、他の Azure Stack Hub 更新プログラムと比較しないでください。

更新プログラムのビルドの種類については、「[Azure Stack Hub での更新プログラム管理の概要](azure-stack-updates.md)」を参照してください。

<!-- ## What's in this update -->

<!-- The current theme (if any) of this release. -->

### <a name="whats-new"></a>新機能

<!-- What's new, also net new experiences and features. -->
- このビルドでは、3 つの新しい GPU VM の種類がサポートされています。NCv3 (Nvidia V100)、NVv4 (AMD MI25)、NCas_v4 (NVIDIA T4) の各 VM サイズです。 適切なハードウェアを使用し、Azure Stack Hub GPU プレビュー プログラムにオンボードされていれば、VM のデプロイは成功します。 関心がある場合は、 https://aka.ms/azurestackhubgpupreview で GPU プレビュー プログラムにサインアップしてください。 詳細については、[こちら](../user/gpu-vms-about.md)を参照してください。
- このリリースには、エラーを検出し、影響を評価して、システムの問題を安全に軽減する自律復旧機能を有効にする新機能が用意されています。 この機能により、Microsoft は手動による介入なしでシステムの可用性を向上させることを目指しています。 リリース 2005 以降では、アラートの数が減少します。 このパイプラインでエラーが発生しても、通知されない限り、Azure Stack Hub オペレーターによるアクションは不要です。
- Azure Stack Hub 管理ポータルに、エアギャップまたは切断された Azure Stack Hub のお客様がログをローカルに保存するための新しいオプションが追加されました。 Azure Stack Hub が Azure から切断されている場合、ログをローカル SMB 共有に保存できます。
- システム操作が既に進行中の場合に Azure Stack Hub 管理ポータルで特定の操作がブロックされるようになりました。 たとえば、更新が進行中の場合、新しいスケール ユニット ノードを追加することはできません。
- このリリースでは、1910 より前に作成された VM で、Azure とのファブリックの一貫性が向上します。 1910 で、Microsoft は、新しく作成されたすべての VM で wireserver プロトコルが使用されることを発表しました。これにより、お客様は Azure と同じ WALA エージェントおよび Windows ゲスト エージェントを使用できるようになり、Azure Stack Hub でより簡単に Azure イメージを使用できるようになりました。 今回のリリースでは、1910 より前に作成されたすべての VM が、wireserver プロトコルを使用するよう自動的に移行されます。 これにより、より信頼性の高い VM の作成、VM 拡張機能のデプロイ、安定状態のアップタイムの向上も実現します。
- Azure Stack Hub ストレージで、Azure Storage サービスの API バージョン 2019-02-02 がサポートされるようになりました。 Azure クライアント ライブラリでは、これは新しい REST API バージョンと互換性があります。 詳細については、[Azure Stack Hub ストレージ開発ツール](../user/azure-stack-storage-dev.md#azure-client-libraries)に関する記事をご覧ください。
- Azure Stack Hub で、[CreateUiDefinition (バージョン 2)](/azure/azure-resource-manager/managed-applications/create-uidefinition-overview) の最新バージョンがサポートされるようになりました。
- バッチ処理された VM デプロイに関する新しいガイダンス。 詳細については、[こちらの記事](../operator/azure-stack-capacity-planning-compute.md)をご覧ください。
- Azure Stack Hub Marketplace CoreOS Container Linux 項目は、[まもなくサポート終了になります](https://azure.microsoft.com/updates/flatcar-in-azure/)。 詳細については、[CoreOS Container Linux からの移行](https://docs.flatcar-linux.org/os/migrate-from-container-linux/)に関するページをご覧ください。

### <a name="improvements"></a>改善

<!-- Changes and product improvements with tangible customer-facing value. -->

- ストレージ インフラストラクチャ クラスター サービスのログとイベントの機能強化。 ストレージ インフラストラクチャ クラスター サービスのログとイベントは、診断とトラブルシューティングの向上のために最大 14 日間保持されます。
- Azure Stack Hub の開始と停止の信頼性を向上させる機能強化。
- 分散を使用し、依存関係を削除することで、更新プログラムの実行時間を短縮する機能強化。 2002 更新プログラムと比較すると、4 ノードのスタンプ更新時間が、15 から 42 時間から、13 から 20 時間に短縮されます。 8 ノードでは、20 から 50 時間から、16 から 26 時間に短縮されます。 12 ノードでは、20 から 60 時間から、19 から 32 時間に短縮されます。 16 ノードでは、25 から 70 時間から、22 から 38 時間に短縮されます。 更新プログラムの正確なランタイムは一般的に、ご使用のシステムでテナント ワークロードによって使用されている容量、システム ネットワーク接続 (インターネットに接続されている場合)、およびシステム ハードウェアの仕様に左右されます。
- 特定の回復不能なエラーが発生した場合、更新が早期に失敗するようになりました。
- インターネットからのダウンロード中に更新プログラム パッケージの回復性が向上しました。
- VM の停止 - 割り当て解除中の回復性が向上しました。
- ネットワーク コントローラーのホスト エージェントの回復性が向上しました。
- 特権エンドポイントと復旧エンドポイントへの接続に使用されるソース IP とアカウントを報告するために、syslog メッセージの CEF ペイロードにフィールドが追加されました。 詳細については、「[Syslog 転送を使用して Azure Stack Hub と監視ソリューションを統合する](azure-stack-integrate-security.md)」を参照してください。
- syslog クライアントによって生成されるイベントのリストに Windows Defender イベント (イベント ID 5001、5010、5012) が追加されました。
- Defender プラットフォームとシグネチャのバージョンの不一致、および検出されたマルウェアに対するアクションの失敗を報告するために、Azure Stack 管理者ポータルに Windows Defender 関連のイベントのアラートが追加されました。
- Azure Stack Hub をデータセンターに統合するときに、4 つの境界デバイスのサポートが追加されました。

### <a name="changes"></a>[変更点]

- インフラストラクチャ ロール インスタンスを停止、シャットダウン、再起動するアクションが、管理ポータルから削除されました。 また、ファブリック リソース プロバイダーの対応する API も削除されました。 Azure Stack Hub 用の管理 RM モジュールと AZ プレビューの次の PowerShell コマンドレットは機能しなくなりました: **Stop-AzsInfrastructureRoleInstance**、**Disable-InfrastructureRoleInstance**、**Restart-InfrastructureRoleInstance**。 これらのコマンドレットは、Azure Stack Hub 用管理 AZ モジュールの次回リリースから削除される予定です。
- Azure Stack Hub 2005 では、[App Service on Azure Stack Hub 2020 (バージョン 87.x)](app-service-release-notes-2020-Q2.md) のみがサポートされるようになりました。

### <a name="fixes"></a>修正

<!-- Product fixes that came up from customer deployments worth highlighting, especially if there is an SR/ICM associated to it. -->

- ベース OS イメージへのパスが見つからないためスケール ユニット ノードの修復が失敗する原因となる問題を修正しました。
- スケール ユニット ノードの修復に連鎖的な影響を及ぼすサポート インフラストラクチャ ロールのスケールインとスケールアウトに関する問題を修正しました。
- オペレーターが **[すべてのサービス] > [コンピューティング] > [VM イメージ] > [追加]** で Azure Stack Hub 管理者ポータルに独自のイメージを追加するときに (.vhd ではなく) .VHD 拡張子が許可されない問題を修正しました。
- 以前の VM 再起動操作によって、他の VM 更新操作 (ディスク、タグなどの追加) 後に予期しない再起動が発生する問題を修正しました。
- 重複する DNS ゾーンを作成するとポータルが応答しなくなる問題を修正しました。 適切なエラーが表示されるようになりました。
- **Get-AzureStackLogs** で、ネットワークの問題のトラブルシューティングに必要なログが収集されていない問題を修正しました。 
- ポータルで接続が許可される NIC の数が、実際に許可されている数よりも少ないという問題を修正しました。 
- 特定の内部ソフトウェアに対して違反イベントが生成されないようにコードの整合性ポリシーを修正しました。 これにより、syslog クライアントによって生成されるコードの整合性違反イベントのノイズが減少します。
- https サービスの再起動またはホストの再起動を求めることなく新しいポリシーを適用するように **Set-TLSPolicy** コマンドレットを修正しました。
- Linux NTP サーバーを使用したときに管理ポータルで誤ってアラートが生成される問題を修正しました。  
- バックアップ コントローラー サービス インスタンスのフェールオーバーによって自動バックアップが無効になる問題を修正しました。
- インフラストラクチャ サービスがインターネットに接続されていないときに内部シークレットのローテーションが失敗する問題を修正しました。
- ユーザーが、Azure Stack Hub ポータルを使用して、サブスクリプションのアクセス許可を表示できない問題を修正しました。

## <a name="security-updates"></a>セキュリティ更新プログラム

Azure Stack Hub のこの更新でのセキュリティ更新プログラムについては、「[Azure Stack Hub のセキュリティ更新プログラム](release-notes-security-updates.md)」を参照してください。

## <a name="hotfixes"></a>修正プログラム

Azure Stack Hub では、修正プログラムが定期的にリリースされます。 2005 リリース以降では、新しいメジャー バージョンに更新すると (たとえば、1.2002.x から 1.2005.x)、その新しいメジャー バージョン内の最新の修正プログラム (存在する場合) が自動的にインストールされます。 それ以降は、ビルドの修正プログラムがリリースされたら、それをインストールする必要があります。

> [!NOTE]
> Azure Stack Hub 修正プログラムのリリースは累積的です。そのバージョンに対する以前の修正プログラムのリリースに含まれるすべての修正を取得するには、最新の修正プログラムをインストールするだけで済みます。

詳細については、[サービス ポリシー](azure-stack-servicing-policy.md)に関する記事を参照してください。

Azure Stack Hub 修正プログラムを適用できるのは Azure Stack Hub 統合システムのみです。ASDK には修正プログラムをインストールしないでください。

### <a name="prerequisites-before-applying-the-2005-update"></a>前提条件:2005 更新プログラムを適用する前

Azure Stack Hub の 2005 リリースは、次の修正プログラムが適用された 2002 リリースに適用する必要があります。

- [Azure Stack Hub 修正プログラム 1.2002.56.152](https://support.microsoft.com/help/4582983)

### <a name="after-successfully-applying-the-2005-update"></a>2005 更新プログラムが正常に適用された後

2005 リリース以降では、新しいメジャー バージョンに更新すると (たとえば、1.2002.x から 1.2005.x)、その新しいメジャー バージョン内の最新の修正プログラム (存在する場合) が自動的にインストールされます。

2005 のインストール後に、2005 修正プログラムがリリースされた場合は、それらをインストールする必要があります。

- [Azure Stack Hub 修正プログラム 1.2005.13.68](https://support.microsoft.com/help/4583399)
::: moniker-end

::: moniker range="azs-2002"
## <a name="2002-build-reference"></a>2002 ビルドのリファレンス

Azure Stack Hub 2002 更新プログラムのビルド番号は **1.2002.0.35** です。

> [!IMPORTANT]  
> Azure Stack Hub 2002 更新プログラムでは、Microsoft は [Azure Stack Hub サポート ポリシー ステートメント](azure-stack-servicing-policy.md)を一時的に延長しています。  Microsoft は現在、COVID-19 に対応中のユーザーで、Azure Stack Hub システムとその更新および管理の方法に関する重要な意思決定を行う可能性がある世界中のお客様と連携しています。その結果、お客様のデータセンターのビジネス運営が引き続き通常通り運営できるようにしています。 Microsoft では、お客様をサポートするために、3 つ前までの更新プログラムのバージョンを含めるように一時的なサポート ポリシー変更の延長を行っています。  その結果、新しくリリースされた 2002 更新プログラムと、3 つ前までの更新プログラムのバージョン (1910、1908、1907 など) のいずれもがサポートされるようになります。

### <a name="update-type"></a>更新の種類

Azure Stack Hub 2002 更新プログラムのビルドの種類は**完全**です。

2002 更新プログラム パッケージは、以前の更新プログラムよりも大きなサイズです。 サイズが大きいため、ダウンロード時間が長くなります。 この更新プログラムは**準備中**状態の時間が長くなります。オペレーターは以前の更新プログラムよりもこのプロセスの時間が長くなることを想定してください。 内部テストでは、2002 更新プログラムの実行時間が次のように予測されました。4 ノード:15 から 42 時間、8 ノード:20 から 50 時間、12 ノード:20 から 60 時間、16 ノード:25 から 70 時間。 更新プログラムの正確なランタイムは一般的に、ご使用のシステムでテナント ワークロードによって使用されている容量、システム ネットワーク接続 (インターネットに接続されている場合)、およびシステム ハードウェアの仕様に左右されます。 実行時間がこの予測値よりも短くなったり長くなったりすることは一般的ではなく、更新が失敗した場合を除き、Azure Stack Hub オペレーターによるアクションは不要です。 このおおよその実行時間は 2002 更新プログラムに固有であり、他の Azure Stack Hub 更新プログラムと比較することはできません。

更新プログラムのビルドの種類については、「[Azure Stack Hub での更新プログラム管理の概要](azure-stack-updates.md)」を参照してください。

<!-- ## What's in this update -->

<!-- The current theme (if any) of this release. -->

### <a name="whats-new"></a>新機能

<!-- What's new, also net new experiences and features. -->

- AzureRM に基づく Azure Stack Hub 管理 PowerShell モジュールの新しいバージョン (1.8.1) がリリースされました。
- Azure Stack Hub 管理 REST API の新しいバージョンがリリースされました。 エンドポイントと破壊的変更の詳細については、[API リファレンス](/rest/api/azure-stack/)でご確認ください。
- 新しい Azure PowerShell テナント モジュールは、2020 年 4 月 15 日に Azure Stack Hub 用にリリースされます。 現在使用されている Azure RM モジュールは引き続き機能しますが、ビルド 2002 の後は更新されません。
- 構成された syslog サーバーの接続の問題について報告するために Azure Stack Hub 管理者ポータルに新しい警告アラートが追加されました。 アラートのタイトルは、**The Syslog client encountered a networking issue while sending a Syslog message** (syslog クライアントは syslog メッセージの送信中にネットワークの問題を検出しました) です。
- ネットワーク タイム プロトコル (NTP) サーバーの接続の問題について報告するために Azure Stack Hub 管理者ポータルに新しい警告アラートが追加されました。 アラートのタイトルは、**Invalid Time Source on [node name]** ([ノード名] の時間ソースが無効です) です。
- 2002 での TLS 制限に関連する破壊的変更により、[Java SDK](https://azure.microsoft.com/develop/java/)の新しいパッケージがリリースされました。 新しい Java SDK 依存関係をインストールする必要があります。 手順については、「[Java と API バージョンのプロファイル](../user/azure-stack-version-profiles-java.md?view=azs-2002#java-and-api-version-profiles)」を参照してください。
- System Center Operations Manager - Azure Stack Hub MP の新しいバージョン (1.0.5.10) が利用できます。これは、API の破壊的変更により、2002 を実行しているすべてのシステムで必要となります。 この API の変更は、バックアップとストレージのパフォーマンス ダッシュボードに影響します。最初にすべてのシステムを 2002 に更新してから MP を更新することをお勧めします。

### <a name="improvements"></a>機能強化

<!-- Changes and product improvements with tangible customer-facing value. -->

- この更新プログラムには、今後の完全な更新のパフォーマンスを大幅に向上させる更新プロセスの変更が含まれています。 これらの変更は、2002 リリース後の次の完全な更新で有効になり、特にホスト オペレーティング システムが更新される完全な更新のフェーズのパフォーマンスを向上させます。 ホスト オペレーティング システムの更新のパフォーマンスを向上させると、完全な更新中にテナントのワークロードが影響を受ける時間が大幅に短縮されます。
- Azure Stack Hub 適合性チェッカー ツールは、AD Graph に割り当てられているすべての TCP IP ポートを使用して AD Graph の統合を検証するようになりました。
- オフライン シンジケーション ツールは、信頼性に関する機能強化によって更新されました。 このツールは GitHub では入手できなくなり、[PowerShell ギャラリーに移動されました](https://www.powershellgallery.com/packages/Azs.Syndication.Admin/)。 詳細については、「[Azure Stack Hub に Marketplace の項目をダウンロードする](azure-stack-download-azure-marketplace-item.md)」を参照してください。
- 新しい監視機能が導入されています。 物理ホストとインフラストラクチャ VM のディスク領域不足のアラートは、プラットフォームによって自動的に修復されます。この操作が失敗した場合にのみ、オペレーターがアクションを実行するために、Azure Stack Hub 管理者ポータルにアラートが表示されます。
- [診断ログの収集](./azure-stack-diagnostic-log-collection-overview.md?view=azs-2002)に対する改善。 新しいエクスペリエンスでは、BLOB ストレージ アカウントを事前に構成する必要がなくなるため、診断ログの収集が合理化されて簡素化されます。 ストレージ環境が事前構成されるため、サポート ケースを開く前にログを送信でき、サポート コールにかかる時間が短縮されます。
- [事前ログ収集とオンデマンドのログ収集](./azure-stack-diagnostic-log-collection-overview.md?view=azs-2002) の両方にかかる時間が 80% 削減されました。 ログ収集時間は、この予想値より長くなることがありますが、ログ収集が失敗しない限り、Azure Stack Hub オペレーターによる操作は必要ありません。
- 更新が開始された後、Azure Stack Hub 更新パッケージのダウンロードの進行状況が更新ブレードに表示されるようになりました。 これは、[自動ダウンロードを使用して更新パッケージを準備する](azure-stack-update-prepare-package.md#automatic-download-and-preparation-for-update-packages)ことを選択した、接続済みの Azure Stack Hub システムに対してのみ適用されます。
- ネットワーク コントローラーのホスト エージェントの信頼性に関する機能強化。
- 修正プログラムや更新プログラムを適用中の内部 DNS サービスの回復性ロジックを向上させる、DNS Orchestrator と呼ばれる新しいマイクロサービスが導入されました。
- VM の作成中にブート診断ストレージ アカウントパラメーターの無効な BLOB URI をエラーとする新しい要求検証を追加しました。
- VM の CRUD 操作を容易にするホスト上の 2 つのサービスである Rdagent とホスト エージェントの自動修復とログ作成の機能強化が追加されました。
- Azure Stack のバージョンや課金モデルなどのさまざまなプロパティにより、利用している Azure Stack と互換性のないマーケットプレース製品を管理者にダウンロードさせないようにする属性を Microsoft が追加できるようにする新しい機能がマーケットプレース管理に追加されました。 これらの属性を追加できるのは Microsoft だけです。 詳細については、「[ポータルを使用して Marketplace 項目をダウンロードする](azure-stack-download-azure-marketplace-item.md#use-the-portal-to-download-marketplace-items)」を参照してください。

### <a name="changes"></a>[変更点]

- 管理者ポータルで、操作が進行中かどうかが Azure Stack 領域の横にあるアイコンで表示されるようになりました。 アイコンの上にマウスポインターを置くと、操作の名前が表示されます。 これにより、何時間も実行されることがあるバックアップ ジョブやストレージの拡張など、実行中のシステム バックグラウンド操作を識別することができます。

- 次の管理者 API が非推奨となりました。

  | リソース プロバイダー       | リソース              | Version            |
  |-------------------------|-----------------------|--------------------|
  | Microsoft.Storage.Admin | farms                 | 2015-12-01-preview |
  | Microsoft.Storage.Admin | farms/acquisitions    | 2015-12-01-preview |
  | Microsoft.Storage.Admin | farms/shares          | 2015-12-01-preview |
  | Microsoft.Storage.Admin | farms/storageaccounts | 2015-12-01-preview |

- 次の管理者 API は、新しいバージョン (2018-09-01) に置き換えられました。

  | リソース プロバイダー      | リソース              | Version    |
  |------------------------|-----------------------|------------|
  | Microsoft.Backup.Admin | backupLocation         | 2016-05-01 |
  | Microsoft.Backup.Admin | backups                | 2016-05-01 |
  | Microsoft.Backup.Admin | operations             | 2016-05-01 |

- PowerShell を使用して Windows VM を作成するときに、VM で拡張機能をデプロイする場合は、必ず `provisionvmagent` フラグを追加してください。 このフラグがない場合、VM はゲスト エージェントなしで作成され、VM 拡張機能をデプロイする機能が削除されます。

   ```powershell
   $VirtualMachine = Set-AzureRmVMOperatingSystem `
     -VM $VirtualMachine `
     -Windows `
     -ComputerName "MainComputer" `
     -Credential $Credential -ProvisionVMAgent
  ```

### <a name="fixes"></a>修正

<!-- Product fixes that came up from customer deployments worth highlighting, especially if there is an SR/ICM associated to it. -->

- 仮想マシンの同じ NIC に複数のパブリック IP を追加すると、インターネット接続の問題が発生する問題を修正しました。 これで、2 つのパブリック IP を持つ NIC は正常に動作するようになります。
- Azure AD のホーム ディレクトリを構成する必要があることを示すアラートがシステムによって生成される原因となった問題を修正しました。
- アラートが自動的に閉じない原因となった問題を修正しました。 このアラートは、Azure AD のホーム ディレクトリが構成されている必要があることを示していましたが、問題が軽減された後も閉じませんでした。
- 更新リソース プロバイダーの内部エラーの結果として、更新準備フェーズ中に更新が失敗する原因となった問題を修正しました。
- Azure Stack Hub のシークレット ローテーションの実行後にアドオン リソース プロバイダーの操作が失敗する原因となる問題を修正しました。
- ERCS ロールのメモリ不足のために Azure Stack Hub の更新エラーの一般的な原因となった問題を修正しました。
- Azure Stack Hub の更新の準備フェーズ中に更新ステータスが **[準備中]** ではなく **[インストール中]** と表示されていた、更新ブレードのバグを修正しました。
- 仮想スイッチ上の RSC 機能で不整合が発生し、ロード バランサーを通過するトラフィックが破棄される問題を修正しました。 RSC 機能は既定で無効化されるようになりました。
- NIC の複数の IP 構成が原因でトラフィックが誤ってルーティングされ、送信接続が妨げられていた問題を修正しました。 
- NIC の MAC アドレスがキャッシュされているときに、そのアドレスを別のリソースに割り当てると VM のデプロイ エラーが発生する問題が修正されました。
- RETAIL チャネルからの Windows VM イメージが、AVMA によってライセンス認証を行うことができない問題を修正しました。
- VM によって要求された仮想コアの数がノードの物理コアと等しい場合に VM が作成されないという問題を修正しました。 VM の仮想コアがノードの物理コアの数以下でも許可されるようになりました。
- ライセンスの種類を "null" に設定して、従量課金制イメージを BYOL に切り替えることができない問題を修正しました。
- VM スケール セットに拡張機能を追加できるようにするために問題を修正しました。

## <a name="security-updates"></a>セキュリティ更新プログラム

Azure Stack Hub のこの更新でのセキュリティ更新プログラムについては、「[Azure Stack Hub のセキュリティ更新プログラム](release-notes-security-updates.md)」を参照してください。

## <a name="hotfixes"></a>修正プログラム

Azure Stack Hub では、修正プログラムが定期的にリリースされます。 Azure Stack Hub を 2002 に更新する前に、必ず 1910 用の最新の Azure Stack Hub 修正プログラムをインストールしてください。

> [!NOTE]
> Azure Stack Hub 修正プログラムのリリースは累積的です。そのバージョンに対する以前の修正プログラムのリリースに含まれるすべての修正を取得するには、最新の修正プログラムをインストールするだけで済みます。

Azure Stack Hub 修正プログラムを適用できるのは Azure Stack Hub 統合システムのみです。ASDK には修正プログラムをインストールしないでください。

修正プログラムの詳細については、「[Azure Stack Hub サービス ポリシー](azure-stack-servicing-policy.md#hotfixes)」を参照してください。

### <a name="prerequisites-before-applying-the-2002-update"></a>前提条件:2002 更新プログラムを適用する前

Azure Stack Hub の 2002 リリースは、以下の修正プログラムが適用された 1910 リリースに適用する必要があります。

<!-- One of these. Either no updates at all, nothing is required, or the LATEST hotfix that is required-->
- [Azure Stack Hub 修正プログラム 1.1910.63.186](https://support.microsoft.com/help/4574735)

### <a name="after-successfully-applying-the-2002-update"></a>2002 更新プログラムの適用に成功した後

この更新プログラムをインストールした後、適用可能な修正プログラムがあればインストールします。

<!-- One of these. Either no updates at all, nothing is required, or the LATEST hotfix that is required-->
- [Azure Stack Hub 修正プログラム 1.2002.56.152](https://support.microsoft.com/help/4582983)
::: moniker-end

::: moniker range="azs-1910"
## <a name="1910-build-reference"></a>1910 ビルドのリファレンス

Azure Stack Hub 1910 更新プログラムのビルド番号は **1.1910.0.58** です。

### <a name="update-type"></a>更新の種類

1908 以降、Azure Stack Hub が実行される基盤のオペレーティング システムは Windows Server 2019 に更新されました。 この更新プログラムによって、核となる基本的な機能強化と、Azure Stack Hub に機能を追加する機能が使用可能になります。

Azure Stack Hub 1910 更新プログラムのビルドの種類は**高速**です。

1910 更新プログラム パッケージは、以前の更新プログラムと比較してサイズが大きくなっており、ダウンロードにより長い時間がかかります。 更新プログラムは長い時間、**準備中**状態のままになり、オペレーターは以前の更新プログラムよりもこのプロセスに長い時間がかかることを予想できます。 1910 更新プログラムが完了するまでの予測所要時間は、ご使用の Azure Stack Hub 環境内の物理ノード数に関係なく、約 10 時間です。 更新プログラムの正確なランタイムは一般的に、ご使用のシステムでテナント ワークロードによって使用されている容量、システム ネットワーク接続 (インターネットに接続されている場合)、およびシステム ハードウェアの仕様に左右されます。 ランタイムが予測値よりも長くなることは珍しいわけではなく、更新が失敗した場合を除いて、Azure Stack Hub オペレーターによるアクションは必要ありません。 このおおよその実行時間は、1910 更新プログラムに固有であり、他の Azure Stack Hub 更新プログラムと比較することはできません。

更新プログラムのビルドの種類については、「[Azure Stack Hub での更新プログラム管理の概要](azure-stack-updates.md)」を参照してください。

<!-- ## What's in this update -->

<!-- The current theme (if any) of this release. -->

### <a name="whats-new"></a>新機能

<!-- What's new, also net new experiences and features. -->

- 管理者ポータルでは、リージョンのプロパティ メニューに特権エンドポイントの IP アドレスが表示され、簡単に見つけられるようになりました。 さらに、現在構成されているタイム サーバーと DNS フォワーダーも表示されます。 詳細については、「[Azure Stack Hub で特権エンドポイントを使用する](azure-stack-privileged-endpoint.md)」を参照してください。

- Azure Stack Hub の正常性および監視システムでは、エラーが発生した場合、さまざまなハードウェア コンポーネントに対してアラートを生成できるようになりました。 これらのアラートには、追加の構成が必要になります。 詳細については、「[Azure Stack Hub のハードウェア コンポーネントを監視する](azure-stack-hardware-monitoring.md)」を参照してください。

- [Azure Stack Hub の cloud-init のサポート](/azure/virtual-machines/linux/using-cloud-init):cloud-Init は、Linux VM を初回起動時にカスタマイズするために広く使用されているアプローチです。 cloud-init を使って、パッケージをインストールしてファイルを書き込んだり、ユーザーとセキュリティを構成したりすることができます。 cloud-init は初回起動プロセスの間に呼び出されるので、構成を適用するために追加の手順や必要なエージェントはありません。 マーケットプレースの Ubuntu イメージは、プロビジョニング用の cloud-init をサポートするように更新されました。

- Azure Stack Hub では、すべての Windows Azure Linux エージェント バージョンバージョンが Azure としてサポートされるようになりました。

- Azure Stack Hub 管理 PowerShell モジュールの新しいバージョンがリリースされました。 <!-- For more information, see -->

- 2020 年 4 月 15 日に、Azure Stack Hub 用の新しい Azure PowerShell テナント モジュールがリリースされました。 現在使用されている Azure RM モジュールは引き続き機能しますが、ビルド 2002 の後は更新されません。

- Azure Stack Hub インフラストラクチャの Windows Defender 定義の手動更新を構成するために、特権エンドポイント (PEP) に **Set-AzSDefenderManualUpdate** コマンドレットを追加しました。 詳細については、「[Azure Stack Hub 上で Windows Defender ウイルス対策を更新する](azure-stack-security-av.md)」を参照してください。

- Azure Stack Hub インフラストラクチャの Windows Defender 定義の手動更新の構成を取得するために、特権エンドポイント (PEP) に **Get-AzSDefenderManualUpdate** コマンドレットを追加しました。 詳細については、「[Azure Stack Hub 上で Windows Defender ウイルス対策を更新する](azure-stack-security-av.md)」を参照してください。

- 特権エンドポイント (PEP) に Azure Stack Hub の DNS サーバーのフォワーダー設定を変更する **Set-AzSDnsForwarder** コマンドレットを追加しました。 DNS 構成の詳細については、「[Azure Stack Hub データセンターの DNS の統合](azure-stack-integrate-dns.md)」を参照してください。

- 特権エンドポイント (PEP) に Azure Stack Hub の DNS サーバーのフォワーダー設定を取得する **Get-AzSDnsForwarder** コマンドレットを追加しました。 DNS 構成の詳細については、「[Azure Stack Hub データセンターの DNS の統合](azure-stack-integrate-dns.md)」を参照してください。

- [AKS エンジン](../user/azure-stack-kubernetes-aks-engine-overview.md)を使用する **Kubernetes クラスター**の管理のサポートを追加しました。 この更新プログラムから、運用環境の Kubernetes クラスターをデプロイできるようになりました。 AKS エンジンによって、以下のことが可能になっています。
  - Kubernetes クラスターのライフ サイクルを管理します。 クラスターの作成、更新、およびスケールを行うことができます。
  - AKS チームおよび Azure Stack Hub チームによって作成されたマネージド イメージを使用して、クラスターを維持できます。
  - Azure Resource Manager に統合された Kubernetes クラウド プロバイダーを利用し、ネイティブの Azure リソースを使用してクラスターを構築できます。
  - 接続または切断された Azure Stack Hub スタンプでクラスターをデプロイおよび管理できます。
  - Azure ハイブリッド機能を使用します。
    - Azure Arc との統合。
    - Azure Monitor for Containers との統合。
  - Windows コンテナーを AKS エンジンと共に使用します。
  - デプロイに対して Microsoft サポートおよびエンジニアリグ サポートを受けられます。

### <a name="improvements"></a>機能強化

<!-- Changes and product improvements with tangible customer-facing value. -->

- Azure Stack Hub では、以前に更新の失敗を引き起こしたり、オペレーターによる Azure Stack Hub の更新開始を妨げたりしていた修正プログラムや更新プログラムの問題を自動修復する機能が向上しています。 その結果、**Test-AzureStack -UpdateReadiness** グループに含まれるテストの数が減っています。 詳細については、「[Azure Stack Hub システムの状態を検証する](azure-stack-diagnostic-test.md#groups)」を参照してください。 次の 3 つのテストは **UpdateReadiness** グループに残っています。

  - **AzSInfraFileValidation**
  - **AzSActionPlanStatus**
  - **AzsStampBMCSummary**

- 外部デバイス (USB キーなど) がいつ Azure Stack Hub インフラストラクチャのノードにマウントされたかを報告する監査ルールが追加されました。 監査ログは syslog を介して出力され、"**Microsoft-Windows-Security-Auditing:6416|Plug and Play Events** (Microsoft-Windows-Security-Auditing: 6416|プラグ アンド プレイ イベント)" と表示されます。 syslog クライアントの構成方法の詳細については、[Syslog の転送](azure-stack-integrate-security.md)に関する記事を参照してください。

- 内部証明書については、Azure Stack Hub では 4096 ビット RSA キーに移行中です。 内部シークレットのローテーションを実行すると、以前の 2048 ビット証明書は 4096 ビット長の証明書に置き換えられます。 Azure Stack Hub のシークレット ローテーションの詳細については、「[Azure Stack Hub でシークレットをローテーションする](azure-stack-rotate-secrets.md)」を参照してください。

- Committee on National Security Systems - Policy 15 (CNSSP-15) (安全な情報共有のための公開標準の使用に関するベスト プラクティスが記載されています) に準拠するために、いくつかの内部コンポーネントについて暗号化アルゴリズムの複雑さとキー強度をアップグレードします。 強化された点として、Kerberos 認証用の AES256 と、VPN 暗号化用の SHA384 があります。 CNSSP-15 の詳細については、[Committee on National Security Systems の「Policies (ポリシー)」ページ](http://www.cnss.gov/CNSS/issuances/Policies.cfm)を参照してください。

- 上記のアップグレードに伴い、Azure Stack Hub には IPsec/IKEv2 構成の新しい既定値が追加されました。 Azure Stack Hub 側で使用される新しい既定値は次のとおりです。

   **IKE フェーズ 1 (メイン モード) のパラメーター**

   | プロパティ              | 値|
   |-|-|
   | IKE のバージョン           | IKEv2 |
   |Diffie-hellman グループ   | ECP384 |
   | 認証方法 | 事前共有キー |
   |暗号化とハッシュ アルゴリズム | AES256、SHA384 |
   |SA の有効期間 (時間)     | 28,800 秒|

   **IKE フェーズ 2 (クイック モード) のパラメーター**

   | プロパティ| 値|
   |-|-|
   |IKE のバージョン |IKEv2 |
   |暗号化とハッシュ アルゴリズム (暗号化)     | GCMAES256|
   |暗号化とハッシュ アルゴリズム (認証) | GCMAES256|
   |SA の有効期間 (時間)  | 27,000 秒  |
   |SA の有効期間 (キロバイト単位) | 33,553,408     |
   |Perfect Forward Secrecy (PFS) | ECP384 |
   |Dead Peer Detection | サポートされています|

   これらの変更は、[既定の IPsec/IKE 提案](../user/azure-stack-vpn-gateway-settings.md#ipsecike-parameters)のドキュメントにも反映されます。

- インフラストラクチャ バックアップ サービスのロジックは、固定のしきい値を利用するのではなく、バックアップに必要な空き領域を計算するように改善されました。 このサービスでは、バックアップのサイズ、アイテム保持ポリシー、予約、および外部ストレージの場所の現在の使用率が使用され、オペレーターに警告を発する必要があるかどうかが決定されます。

### <a name="changes"></a>[変更点]

- Azure から Azure Stack Hub にマーケットプレース項目をダウンロードするときに、複数のバージョンが存在する場合、項目のバージョンを指定できる新しいユーザー インターフェイスが追加されました。 新しい UI は、接続されたシナリオと切断されたシナリオの両方で使用できます。 詳細については、「[Azure から Azure Stack Hub に Marketplace の項目をダウンロードする](azure-stack-download-azure-marketplace-item.md)」を参照してください。  

- 1910 リリース以降、Azure Stack Hub システムには追加で /20 プライベート内部 IP 空間が**必要**になりました。 詳細については、「[Azure Stack のためのネットワーク統合計画](azure-stack-network.md)」を参照してください。
  
- アップロード手順中に外部ストレージの場所の容量が不足すると、インフラストラクチャ バックアップ サービスによって、部分的にアップロードされたバックアップ データが削除されます。  

- インフラストラクチャ バックアップ サービスでは、AAD デプロイのためのバックアップ ペイロードに ID サービスが追加されました。  

- 1910 リリースでは、Azure Stack Hub PowerShell モジュールがバージョン 1.8.0 に更新されました。<br>変更内容:
   - **新しい DRP 管理モジュール**:デプロイ リソース プロバイダー (DRP) を使用すると、Azure Stack Hub に対するリソース プロバイダーの調整されたデプロイが可能になります。 これらのコマンドを使うと、DRP とやり取りする Azure Resource Manager レイヤーとやり取りできます。
   - **BRP**: <br />
           - Azure Stack インフラストラクチャ バックアップの 1 つのロールの復元をサポートします。 <br />
           - パラメーター `RoleName` をコマンドレット `Restore-AzsBackup` に追加します。
   - **FRP**:API バージョン `2019-05-01` の**ドライブ** リソースと**ボリューム** リソースの重大な変更。 これらの機能は Azure Stack Hub 1910 以降でサポートされています。 <br />
            - `ID`、`Name`、`HealthStatus`、および `OperationalStatus` の値が変更されました。 <br />
            - **ドライブ** リソースの新しいプロパティ `FirmwareVersion`、`IsIndicationEnabled`、`Manufacturer`、`StoragePool` がサポートされるようになりました。 <br />
            - **ドライブ** リソースのプロパティ `CanPool` および `CannotPoolReason` は廃止されました。代わりに `OperationalStatus` を使用してください。

### <a name="fixes"></a>修正

<!-- Product fixes that came up from customer deployments worth highlighting, especially if there's an SR/ICM associated to it. -->

- Azure Stack Hub 1904 リリースより前にデプロイされた環境では TLS 1.2 ポリシーの適用が妨げられる問題を修正しました。
- SSH の承認を有効にして作成された Ubuntu 18.04 VM では、SSH キーを使用してサインインできない問題を修正しました。
- 仮想マシン スケール セットの UI から **[パスワードのリセット]** を削除しました。
- ポータルからロード バランサーを削除しても、インフラストラクチャ レイヤーにあるオブジェクトが削除されない問題を修正しました。
- 管理者ポータル上でゲートウェイ プール使用率アラートの割合が正しく表示されない問題を修正しました。
<!-- Fixed an issue where adding more than one public IP on the same NIC on a Virtual Machine resulted in internet connectivity issues. Now, a NIC with two public IPs should work as expected.[This fix actually didn't go in 1910 due to build issues, commenting out until next build (2002) ] -->

## <a name="security-updates"></a>セキュリティ更新プログラム

Azure Stack Hub のこの更新でのセキュリティ更新プログラムについては、「[Azure Stack Hub のセキュリティ更新プログラム](release-notes-security-updates.md)」を参照してください。

このリリースの Qualys 脆弱性レポートは、[Qualys Web サイト](https://www.qualys.com/azure-stack/)からダウンロードできます。

## <a name="hotfixes"></a>修正プログラム

Azure Stack Hub では、修正プログラムが定期的にリリースされます。 Azure Stack Hub を 1910 に更新する前に、必ず 1908 用の最新の Azure Stack Hub 修正プログラムをインストールしてください。

> [!NOTE]
> Azure Stack Hub 修正プログラムのリリースは累積的です。そのバージョンに対する以前の修正プログラムのリリースに含まれるすべての修正を取得するには、最新の修正プログラムをインストールするだけで済みます。

Azure Stack Hub 修正プログラムを適用できるのは Azure Stack Hub 統合システムのみです。ASDK には修正プログラムをインストールしないでください。

### <a name="prerequisites-before-applying-the-1910-update"></a>前提条件:1910 更新プログラムを適用する前

Azure Stack Hub の 1910 リリースは、以下の修正プログラムが適用された 1908 リリースに適用する必要があります。

<!-- One of these. Either no updates at all, nothing is required, or the LATEST hotfix that is required-->
- [Azure Stack Hub 修正プログラム 1.1908.51.133](https://support.microsoft.com/help/4574734)

### <a name="after-successfully-applying-the-1910-update"></a>1910 更新プログラムの適用に成功した後

この更新プログラムをインストールした後、適用可能な修正プログラムがあればインストールします。 詳細については、[サービス ポリシー](azure-stack-servicing-policy.md)に関する記事を参照してください。

<!-- One of these. Either no updates at all, nothing is required, or the LATEST hotfix that is required-->
- [Azure Stack Hub 修正プログラム 1.1910.63.186](https://support.microsoft.com/help/4574735)
::: moniker-end

<!------------------------------------------------------------>
<!------------------- UNSUPPORTED VERSIONS ------------------->
<!------------------------------------------------------------>
::: moniker range="azs-1908"
## <a name="1908-archived-release-notes"></a>1908 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1907"
## <a name="1907-archived-release-notes"></a>1907 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1906"
## <a name="1906-archived-release-notes"></a>1906 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1905"
## <a name="1905-archived-release-notes"></a>1905 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1904"
## <a name="1904-archived-release-notes"></a>1904 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1903"
## <a name="1903-archived-release-notes"></a>1903 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1902"
## <a name="1902-archived-release-notes"></a>1902 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1901"
## <a name="1901-archived-release-notes"></a>1901 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1811"
## <a name="1811-archived-release-notes"></a>1811 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1809"
## <a name="1809-archived-release-notes"></a>1809 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1808"
## <a name="1808-archived-release-notes"></a>1808 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1807"
## <a name="1807-archived-release-notes"></a>1807 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1805"
## <a name="1805-archived-release-notes"></a>1805 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1804"
## <a name="1804-archived-release-notes"></a>1804 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1803"
## <a name="1803-archived-release-notes"></a>1803 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1802"
## <a name="1802-archived-release-notes"></a>1802 アーカイブされたリリース ノート
::: moniker-end

::: moniker range="<azs-1910"
[以前のバージョンの Azure Stack Hub のリリース ノートは TechNet ギャラリー](https://aka.ms/azsarchivedrelnotes)でアクセスできます。 これらのアーカイブされたドキュメントは、参照のみを目的に提供されており、これらのバージョンのサポートを意味しているわけではありません。 Azure Stack のサポートについては、「[Azure Stack Hub サービス ポリシー](azure-stack-servicing-policy.md)」を参照してください。 さらにサポートが必要な場合は、Microsoft カスタマー サポート サービスにお問い合わせください。
::: moniker-end
