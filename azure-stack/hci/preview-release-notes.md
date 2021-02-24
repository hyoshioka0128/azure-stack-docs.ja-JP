---
title: Azure Stack HCI のリリース ノート アーカイブ
description: Azure Stack HCI バージョン 20H2 パブリック プレビュー リリースのアーカイブされたリリース ノート。
author: khdownie
ms.author: v-kedow
ms.topic: conceptual
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 12/21/2020
ms.openlocfilehash: 5125e9b27fbb9cc17e9381a9617fefce6409418f
ms.sourcegitcommit: a745662c7a5a18f135accf3f70d8508b57e83e2b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/22/2020
ms.locfileid: "97737948"
---
# <a name="archived-release-notes-for-azure-stack-hci-version-20h2-public-preview"></a>Azure Stack HCI バージョン 20H2 パブリック プレビューのアーカイブされたリリース ノート

> 適用対象:Azure Stack HCI バージョン 20H2

この記事では、2020 年 12 月 10 日の一般提供 (GA) リリースより前の Azure Stack HCI バージョン 20H2 パブリック プレビュー リリースに対する更新プログラム パッケージの内容について説明します。 最新の [Azure Stack HCI のリリース ノート](https://support.microsoft.com/help/4595086/)を表示することもできます。

## <a name="december-8-2020-security-update-kb4592441"></a>2020 年 12 月 8 日のセキュリティ更新プログラム (KB4592441)

この更新プログラムには、Azure Stack HCI のパブリック プレビュー リリースに対する機能強化と修正が含まれています。 

### <a name="improvements-and-fixes"></a>機能強化と修正
この更新プログラムには、内部の OS 機能に対するさまざまなセキュリティ強化が含まれています。 SYSTEM アカウントとして実行されるアプリケーションが "FILE:" ポートに出力されないようにすることで、セキュリティの脆弱性に対処しています。 今後、この問題に対処するには、特定のユーザーまたはサービス アカウントとしてアプリケーションまたはサービスを実行するようにします。

今回のリリースに関して別途文書化された問題はありません。

解決済みのセキュリティの脆弱性について詳しくは、[セキュリティ更新プログラム ガイド](https://portal.msrc.microsoft.com/security-guidance)を参照してください。

### <a name="known-issues-in-this-update"></a>この更新プログラムの既知の問題
現在、この更新プログラムに関して Microsoft が把握している問題はありません。

### <a name="how-to-get-this-update"></a>この更新プログラムを入手する方法
[Azure Stack HCI プレビュー](https://azure.microsoft.com/products/azure-stack/hci/hci-download/)向けの 2020 年 12 月 8 日のセキュリティ更新プログラム (KB4592441) は、Windows Update で配信されます。 Azure Stack HCI クラスターへのインストール方法については、「[Azure Stack HCI クラスターを更新する](manage/update-cluster.md)」を参照してください。

### <a name="file-information"></a>ファイル情報
この更新プログラム (OS ビルド 17784.1408) で提供されるファイルの一覧については、[累積的な更新プログラム 4592441 のファイル情報](https://download.microsoft.com/download/2/7/2/272ea75f-1657-43ce-a7a4-a17d51463a94/4592441.csv)をダウンロードしてください。

   > [!NOTE]
   > この CSV ファイルの "File version" 列に、一部のファイルで誤って "Not applicable" と表示されています。 そのことが原因で、サードパーティのスキャン検出ツールを使用してビルドを検証したときに擬陽性または偽陰性の結果が生じる可能性があります。

## <a name="november-23-2020-preview-update-kb4586852"></a>2020 年 11 月 23 日のプレビュー更新プログラム (KB4586852)

この更新プログラムには、Azure Stack HCI のパブリック プレビュー リリースに対する機能強化と修正が含まれています。 

   > [!IMPORTANT]
   > パブリック プレビュー イメージを使用して Azure Stack HCI クラスターを構成および登録している場合、更新プログラムの提供する新機能を使用するには、KB4586852 更新プログラムのインストール後に Azure の登録を修復する必要があります。 この更新プログラムをインストールした後、クラスターごとに次の手順を実行します。
   >
   > 1. 必ずクラスター内のすべてのサーバーを KB4586852 に更新してください。 そうしないと、修復は失敗し、更新が必要なノードが示されます。
   >
   > 2. ローカルで、または `Enter-PSSession <server-name>` を使用して、クラスター ノードのいずれかに接続します
   >
   > 3. PowerShell ギャラリーから AzStackHCI v0.4.1 (以降の) 登録モジュールをダウンロードします。 `Install-Module -Name Az.StackHCI` を実行して、最新のモジュールを取得します。
   >
   > 4. 次のコマンドを実行して、登録を修復します。 最初にクラスターを登録するために使用したサブスクリプション ID を使用します。 `Get-AzureStackHCI` を使用すると、サブスクリプション情報が含まれている現在の Azure Resource Manager の URI が表示されます。
   >
   >   ```PowerShell
   >   Register-AzStackHCI -SubscriptionId "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" -RepairRegistration
   >   ```
   > 

### <a name="improvements-and-fixes"></a>機能強化と修正
セキュリティ関連でないこの更新プログラムには、品質上の機能強化が含まれています。 重要な変更点は、次のとおりです。 

- この更新プログラムを適用すると、有効な Windows Server 2019 Datacenter Edition ライセンスをお持ちの Azure Stack HCI のお客様は、それらを使用して、Azure Stack HCI でホストされている仮想マシン (VM) を簡単にアクティブ化することができ、VM ごとにプロダクト キーを管理する必要はありません。 具体的には、Windows 管理センターまたは PowerShell を使用して、未使用の Windows Server 2019 Datacenter エディションのアクティブ化キーを Azure Stack HCI ホストに直接入力して、VM の自動ライセンス認証 (AVMA) を有効にすることができます。 Windows Server 2019 以前を実行している VM は、ホストからライセンス認証を継承できます。 入力に使用できるキーは、ボリューム ライセンス センターから取得したマルチ ライセンス認証キー (MAK)、OEM サーバーに適用された Certificate of Authenticity (COA) ステッカーに印刷されたキー、または Windows Server 2019 Datacenter エディションの製品版のキーです。 このリリースでは、汎用ボリューム ライセンス キー (GVLK) はサポートされていません。

- Azure Stack HCI では、必要な診断データが収集されるようになりました。これは、デバイスのセキュリティを維持し、最新の状態に保ち、想定どおりに実行するために必要な最小限のデータです。 必要な診断データとして、デバイスとその構成を理解するために重要な限られたデータ セットが収集されます。 このデータは、特定のハードウェアまたはソフトウェア構成で発生する可能性のある問題を特定するために役立ちます。  

### <a name="known-issues-in-this-update"></a>この更新プログラムの既知の問題
現在、この更新プログラムに関して Microsoft が把握している問題はありません。

### <a name="how-to-get-this-update"></a>この更新プログラムを入手する方法 
[Azure Stack HCI プレビュー](https://azure.microsoft.com/products/azure-stack/hci/hci-download/)用の 2020 年 11 月 23 日のセキュリティ更新プログラム (KB4586852) は、Windows Update で配信されます。 Azure Stack HCI クラスターへのインストール方法については、「[Azure Stack HCI クラスターを更新する](manage/update-cluster.md)」を参照してください。

### <a name="file-information"></a>ファイル情報
この更新プログラム (OS ビルド 17784.1381) で提供されるファイルの一覧については、 [累積的な更新プログラム 4586852 のファイル情報](https://download.microsoft.com/download/5/c/6/5c6f8c37-3e0b-4239-a6d9-9c709e18e869/4586852.csv)をダウンロードしてください。

   > [!NOTE]
   > この CSV ファイルの "File version" 列に、一部のファイルで誤って "Not applicable" と表示されています。 そのことが原因で、サードパーティのスキャン検出ツールを使用してビルドを検証したときに擬陽性または偽陰性の結果が生じる可能性があります。

## <a name="november-10-2020-security-update-kb4586811"></a>2020 年 10 月 10 日のセキュリティ更新プログラム (KB4586811)

この更新プログラムには、Azure Stack HCI のパブリック プレビュー リリースに対する機能強化と修正が含まれています。

### <a name="improvements-and-fixes"></a>機能強化と修正
この更新プログラムには、内部の OS 機能に対するさまざまなセキュリティ強化が含まれています。 SYSTEM アカウントとして実行されるアプリケーションが、ファイルを指すローカル ポートに出力されないようにすることで、セキュリティの脆弱性に対処しています。 PrintService\Admin イベント ログのイベント ID 372 に含まれる出力ジョブの失敗ログ エラー 50 "この要求は、サポートされていません"。 今後、この問題に対処するには、特定のユーザーまたはサービス アカウントとしてアプリケーションまたはサービスを実行するようにします。

今回のリリースに関して別途文書化された問題はありません。

解決済みのセキュリティの脆弱性について詳しくは、[セキュリティ更新プログラム ガイド](https://portal.msrc.microsoft.com/security-guidance)を参照してください。

### <a name="known-issues-in-this-update"></a>この更新プログラムの既知の問題
現在、この更新プログラムに関して Microsoft が把握している問題はありません。

### <a name="how-to-get-this-update"></a>この更新プログラムを入手する方法
[Azure Stack HCI](https://azure.microsoft.com/products/azure-stack/hci/hci-download/) 向けの 2020 年 11 月 10 日のセキュリティ更新プログラム (KB4586811) は、Windows Update を介して配信されます。 Azure Stack HCI クラスターへのインストール方法については、「[Azure Stack HCI クラスターを更新する](manage/update-cluster.md)」を参照してください。

### <a name="file-information"></a>ファイル情報
この更新プログラム (OS ビルド 17784.1345) で提供されるファイルの一覧については、[累積的な更新プログラム 4586811 のファイル情報](https://download.microsoft.com/download/8/f/2/8f2ce4bb-e113-4abc-b3ff-f0f4c4c71403/4586811.csv)をダウンロードしてください。

   > [!NOTE]
   > この CSV ファイルの "File version" 列に、一部のファイルで誤って "Not applicable" と表示されています。 そのことが原因で、サードパーティのスキャン検出ツールを使用してビルドを検証したときに擬陽性または偽陰性の結果が生じる可能性があります。

## <a name="november-10-2020-servicing-stack-update-kb4590242"></a>2020 年 10 月 10 日のサービス スタック更新プログラム (KB4590242)

この更新プログラムには、Azure Stack HCI のパブリック プレビュー リリースの品質向上が含まれています。

### <a name="improvements-and-fixes"></a>機能強化と修正
この更新プログラムにより、更新プログラムをインストールするコンポーネントであるサービス スタックの品質が向上します。 サービス スタック更新プログラム (SSU) により、堅牢で信頼性の高いサービス スタックを使用して、お使いのデバイスで Microsoft 更新プログラムを受信し、インストールできるようになります。

### <a name="how-to-get-this-update"></a>この更新プログラムを入手する方法
[Azure Stack HCI](https://azure.microsoft.com/products/azure-stack/hci/hci-download/) 向けの 2020 年 11 月のサービス スタック更新プログラム (KB4590242) は Windows Update を介して配信されます。 Azure Stack HCI クラスターへのインストール方法については、「[Azure Stack HCI クラスターを更新する](manage/update-cluster.md)」を参照してください。

### <a name="restart-information"></a>再起動情報
この更新プログラムを適用した後に、コンピューターを再起動する必要はありません。

### <a name="removal-information"></a>削除情報
サービス スタック更新プログラム (SSU) は、更新プログラムのインストール方法を変更します。これをデバイスからアンインストールすることはできません。

### <a name="file-information"></a>ファイル情報
この更新プログラム (OS ビルド 17784.1342) で提供されるファイルの一覧については、[累積的な更新プログラム 4590242 のファイル情報](https://download.microsoft.com/download/b/b/4/bb4fb4f5-c0ba-4e55-bada-d72310857982/4590242.csv)をダウンロードしてください。

### <a name="references"></a>References

SSU の詳細については、次の記事を参照してください。

- [サービス スタック更新プログラム](/windows/deployment/update/servicing-stack-updates)
- [サービス スタック更新プログラム (SSU): よく寄せられる質問](https://support.microsoft.com/help/4535697)

ソフトウェア更新プログラムを説明するために Microsoft が使用する[用語](https://support.microsoft.com/help/824684)についてご確認ください。

## <a name="october-20-2020-preview-update-kb4580388"></a>2020 年 10 月 20 日のプレビュー更新プログラム (KB4580388)

この更新プログラムには、Azure Stack HCI のパブリック プレビュー リリースに対する機能強化と修正が含まれています。

### <a name="improvements-and-fixes"></a>機能強化と修正
セキュリティ関連でないこの更新プログラムには、品質上の機能強化が含まれています。 重要な変更点は、次のとおりです。

- この更新プログラムを適用すると、有効な Windows Server 2019 Datacenter Edition ライセンスをお持ちの Azure Stack HCI のお客様は、それらを使用して、Azure Stack HCI でホストされている仮想マシン (VM) を簡単にアクティブ化することができ、VM ごとにプロダクト キーを管理する必要はありません。

### <a name="known-issues-in-this-update"></a>この更新プログラムの既知の問題

Microsoft は、この更新プログラムに関して 1 つの問題を把握しています。

#### <a name="symptom"></a>症状
ライブ マイグレーションを使用して Windows Server と Azure Stack HCI オペレーティング システムの間で VM を移動すると、次のエラーが表示されることがあります。"Blocked a migration operation for virtual machine <vmname> because VM migration between differing Windows editions is not supported (virtual machine ID)." (仮想マシン <vmname> の移行操作がブロックされました。異なる Windows エディション間での VM の移行はサポートされていません (仮想マシン ID)。)

または、いずれかの VM がクラスター対応更新 (CAU) の間にライブ マイグレーションを実行することが予想される場合、CAU 操作が失敗することもあります。

#### <a name="workaround"></a>回避策

ライブ マイグレーションの代わりにクイック マイグレーションを使用します。 CAU を使用している場合は、CAU でクイック マイグレーションが使用されるように、既定の動作を一時的に変更します。

例:

```powershell
Get-ClusterResourceType "Virtual Machine" | Set-ClusterParameter MoveTypeThreshold 3001
```

CAU が正常に完了した後、前の `MoveTypeThreshold` の値に戻すことをお勧めします。

詳細については、「[ノードがドレインされたときに VM を移動する方法の構成](https://techcommunity.microsoft.com/t5/failover-clustering/configuring-how-vms-are-moved-when-a-node-is-drained/ba-p/371848)」を参照してください。

### <a name="how-to-get-this-update"></a>この更新プログラムを入手する方法
[Azure Stack HCI](https://azure.microsoft.com/products/azure-stack/hci/hci-download/) 向けの 2020 年 10 月 20 日のセキュリティ更新プログラム (KB4580388) は、Windows Update を介して配信されます。 Azure Stack HCI クラスターへのインストール方法については、「[Azure Stack HCI クラスターを更新する](manage/update-cluster.md)」を参照してください。

### <a name="file-information"></a>ファイル情報
この更新プログラム (OS ビルド 17784.1321) で提供されるファイルの一覧については、[累積的な更新プログラム 4580388 のファイル情報](https://download.microsoft.com/download/2/f/b/2fb766d3-c4c8-4279-8718-8efbd0b6f211/4580388.csv)をダウンロードしてください。

   > [!NOTE]
   > この CSV ファイルの "File version" 列に、一部のファイルで誤って "Not applicable" と表示されています。 そのことが原因で、サードパーティのスキャン検出ツールを使用してビルドを検証したときに擬陽性または偽陰性の結果が生じる可能性があります。

## <a name="october-13-2020-security-update-kb4580363"></a>2020 年 10 月 13 日のセキュリティ更新プログラム (KB4580363)

この更新プログラムには、Azure Stack HCI のパブリック プレビュー リリースに対する機能強化と修正が含まれています。

### <a name="improvements-and-fixes"></a>機能強化と修正
この更新プログラムには、内部の OS 機能に対するさまざまなセキュリティ強化が含まれています。 今回のリリースに関して別途文書化された問題はありません。

解決済みのセキュリティの脆弱性について詳しくは、[セキュリティ更新プログラム ガイド](https://portal.msrc.microsoft.com/security-guidance)を参照してください。

### <a name="known-issues-in-this-update"></a>この更新プログラムの既知の問題
現在、この更新プログラムに関して Microsoft が把握している問題はありません。

### <a name="how-to-get-this-update"></a>この更新プログラムを入手する方法
[Azure Stack HCI](https://azure.microsoft.com/products/azure-stack/hci/hci-download/) 向けの 2020 年 10 月 13 日のセキュリティ更新プログラム (KB4580363) は、Windows Update を介して配信されます。 Azure Stack HCI クラスターへのインストール方法については、「[Azure Stack HCI クラスターを更新する](manage/update-cluster.md)」を参照してください。

### <a name="file-information"></a>ファイル情報
この更新プログラム (OS ビルド 17784.1288) で提供されるファイルの一覧については、[累積的な更新プログラム 4580363 のファイル情報](https://download.microsoft.com/download/f/7/8/f78801f0-e7e5-4a3d-a971-f642ccd24ee4/4580363.csv)をダウンロードしてください。

   > [!NOTE]
   > この CSV ファイルの "File version" 列に、一部のファイルで誤って "Not applicable" と表示されています。 そのことが原因で、サードパーティのスキャン検出ツールを使用してビルドを検証したときに擬陽性または偽陰性の結果が生じる可能性があります。

## <a name="october-13-2020-servicing-stack-update-kb4583287"></a>2020 年 10 月 13 日のサービス スタック更新プログラム (KB4583287)

この更新プログラムには、Azure Stack HCI のパブリック プレビュー リリースの品質向上が含まれています。

### <a name="improvements-and-fixes"></a>機能強化と修正
この更新プログラムにより、更新プログラムをインストールするコンポーネントであるサービス スタックの品質が向上します。 サービス スタック更新プログラム (SSU) により、堅牢で信頼性の高いサービス スタックを使用して、お使いのデバイスで Microsoft 更新プログラムを受信し、インストールできるようになります。

### <a name="how-to-get-this-update"></a>この更新プログラムを入手する方法
[Azure Stack HCI](https://azure.microsoft.com/products/azure-stack/hci/hci-download/) 向けの 2020 年 10 月 13 日のサービス スタック更新プログラム (KB4583287) は Windows Update を介して配信されます。 Azure Stack HCI クラスターへのインストール方法については、「[Azure Stack HCI クラスターを更新する](manage/update-cluster.md)」を参照してください。

### <a name="restart-information"></a>再起動情報
この更新プログラムを適用した後に、コンピューターを再起動する必要はありません。

### <a name="removal-information"></a>削除情報
サービス スタック更新プログラム (SSU) は、更新プログラムのインストール方法を変更します。これをデバイスからアンインストールすることはできません。

### <a name="file-information"></a>ファイル情報
この更新プログラム (OS ビルド 17784.1287) で提供されるファイルの一覧については、[累積的な更新プログラム 4583287 のファイル情報](https://download.microsoft.com/download/b/8/5/b85160fb-85d9-49f9-b9d5-7dbc0158a944/4583287.csv)をダウンロードしてください。

### <a name="references"></a>References

SSU の詳細については、次の記事を参照してください。

- [サービス スタック更新プログラム](/windows/deployment/update/servicing-stack-updates)
- [サービス スタック更新プログラム (SSU): よく寄せられる質問](https://support.microsoft.com/help/4535697)

ソフトウェア更新プログラムを説明するために Microsoft が使用する[用語](https://support.microsoft.com/help/824684)についてご確認ください。

## <a name="september-17-2020-preview-update-kb4577629"></a>2020 年 9 月 17 日のプレビュー更新プログラム (KB4577629)

この更新プログラムには、Azure Stack HCI のパブリック プレビュー リリースに対する機能強化と修正が含まれています。

### <a name="improvements-and-fixes"></a>機能強化と修正
セキュリティ関連でないこの更新プログラムには、品質上の機能強化が含まれています。 重要な変更点は、次のとおりです。
- マルチプレクサーを経由するソフトウェア ロード バランサー (SLB) のトラフィックが、アプリケーション接続エラーを引き起こす可能性がある別のホストにリダイレクトされる可能性がある問題に対処しました。

### <a name="known-issues-in-this-update"></a>この更新プログラムの既知の問題
現在、この更新プログラムに関して Microsoft が把握している問題はありません。

### <a name="how-to-get-this-update"></a>この更新プログラムを入手する方法
[Azure Stack HCI](https://azure.microsoft.com/products/azure-stack/hci/hci-download/) 向けの 2020 年 9 月 17 日のセキュリティ更新プログラム (KB4577629) は、Windows Update を介して配信されます。 Azure Stack HCI クラスターへのインストール方法については、「[Azure Stack HCI クラスターを更新する](manage/update-cluster.md)」を参照してください。

### <a name="file-information"></a>ファイル情報
この更新プログラム (OS ビルド 17784.1259) で提供されるファイルの一覧については、[累積的な更新プログラム 4577629 のファイル情報](https://download.microsoft.com/download/9/1/a/91addcbb-2b36-408c-ab88-736de42edb98/4577629.csv)をダウンロードしてください

   > [!NOTE]
   > この CSV ファイルの "File version" 列に、一部のファイルで誤って "Not applicable" と表示されています。 そのことが原因で、サードパーティのスキャン検出ツールを使用してビルドを検証したときに擬陽性または偽陰性の結果が生じる可能性があります。

## <a name="september-8-2020-security-update-kb4577470"></a>2020 年 9 月 8 日のセキュリティ更新プログラム (KB4577470)

この更新プログラムには、Azure Stack HCI のパブリック プレビュー リリースに対する機能強化と修正が含まれています。

### <a name="improvements-and-fixes"></a>機能強化と修正
この更新プログラムには、内部の OS 機能に対するさまざまなセキュリティ強化が含まれています。 今回のリリースに関して別途文書化された問題はありません。

解決済みのセキュリティの脆弱性について詳しくは、[セキュリティ更新プログラム ガイド](https://portal.msrc.microsoft.com/security-guidance)を参照してください。

### <a name="known-issues-in-this-update"></a>この更新プログラムの既知の問題
現在、この更新プログラムに関して Microsoft が把握している問題はありません。

### <a name="how-to-get-this-update"></a>この更新プログラムを入手する方法
[Azure Stack HCI](https://azure.microsoft.com/products/azure-stack/hci/hci-download/) 向けの 2020 年 9 月 8 日のセキュリティ更新プログラム (KB4577470) は Windows Update を介して配信されます。 Azure Stack HCI クラスターへのインストール方法については、「[Azure Stack HCI クラスターを更新する](manage/update-cluster.md)」を参照してください。

### <a name="file-information"></a>ファイル情報
この更新プログラム (OS ビルド 17784.1226) で提供されるファイルの一覧については、[累積的な更新プログラム 4577470 のファイル情報](https://download.microsoft.com/download/3/c/4/3c468525-5867-4cc3-8d34-dba88989adab/4577470.csv)をダウンロードしてください。

   > [!NOTE]
   > この CSV ファイルの "File version" 列に、一部のファイルで誤って "Not applicable" と表示されています。 そのことが原因で、サードパーティのスキャン検出ツールを使用してビルドを検証したときに擬陽性または偽陰性の結果が生じる可能性があります。

## <a name="september-8-2020-servicing-stack-update-kb4577558"></a>2020 年 9 月 8 日のサービス スタック更新プログラム (KB4577558)

この更新プログラムには、Azure Stack HCI のパブリック プレビュー リリースの品質向上が含まれています。

### <a name="improvements-and-fixes"></a>機能強化と修正
この更新プログラムにより、更新プログラムをインストールするコンポーネントであるサービス スタックの品質が向上します。 サービス スタック更新プログラム (SSU) により、堅牢で信頼性の高いサービス スタックを使用して、お使いのデバイスで Microsoft 更新プログラムを受信し、インストールできるようになります。

### <a name="how-to-get-this-update"></a>この更新プログラムを入手する方法
[Azure Stack HCI](https://azure.microsoft.com/products/azure-stack/hci/hci-download/) 向けの 2020 年 9 月 8 日のサービス スタック更新プログラム (KB4577558) は Windows Update を介して配信されます。 Azure Stack HCI クラスターへのインストール方法については、「[Azure Stack HCI クラスターを更新する](manage/update-cluster.md)」を参照してください。

### <a name="restart-information"></a>再起動情報 
この更新プログラムを適用した後に、コンピューターを再起動する必要はありません。

### <a name="removal-information"></a>削除情報
サービス スタック更新プログラム (SSU) は、更新プログラムのインストール方法を変更します。これをデバイスからアンインストールすることはできません。

### <a name="file-information"></a>ファイル情報
この更新プログラム (OS ビルド 17784.1220) で提供されるファイルの一覧については、[累積的な更新プログラム 4577558 のファイル情報](https://download.microsoft.com/download/8/f/6/8f612a9b-cb4e-4832-9397-156760848592/4577558.csv)をダウンロードしてください。

### <a name="references"></a>References

SSU の詳細については、次の記事を参照してください。

- [サービス スタック更新プログラム](/windows/deployment/update/servicing-stack-updates)
- [サービス スタック更新プログラム (SSU): よく寄せられる質問](https://support.microsoft.com/help/4535697)

ソフトウェア更新プログラムを説明するために Microsoft が使用する[用語](https://support.microsoft.com/help/824684)についてご確認ください。

## <a name="august-11-2020-security-update-kb4574585"></a>2020 年 8 月 11 日のセキュリティ更新プログラム (KB4574585)

この更新プログラムには、Azure Stack HCI のパブリック プレビュー リリースに対する機能強化と修正が含まれています。

### <a name="improvements-and-fixes"></a>機能強化と修正
この更新プログラムには、内部の OS 機能に対するさまざまなセキュリティ強化が含まれています。 今回のリリースに関して別途文書化された問題はありません。

解決済みのセキュリティの脆弱性について詳しくは、[セキュリティ更新プログラム ガイド](https://portal.msrc.microsoft.com/security-guidance)を参照してください。

### <a name="known-issues-in-this-update"></a>この更新プログラムの既知の問題
現在、この更新プログラムに関して Microsoft が把握している問題はありません。

### <a name="how-to-get-this-update"></a>この更新プログラムを入手する方法
[Azure Stack HCI](https://azure.microsoft.com/products/azure-stack/hci/hci-download/) 向けの 2020 年 8 月 11 日のセキュリティ更新プログラム (KB4574585) は Windows Update を介して配信されます。 Azure Stack HCI クラスターへのインストール方法については、「[Azure Stack HCI クラスターを更新する](manage/update-cluster.md)」を参照してください。

### <a name="file-information"></a>ファイル情報
この更新プログラム (OS ビルド 17784.1162) で提供されるファイルの一覧については、[累積的な更新プログラム 4574585 のファイル情報](https://download.microsoft.com/download/7/f/4/7f451def-76c5-4cc0-9929-0c5efeb27d2f/4574585.csv)をダウンロードしてください。

   > [!NOTE]
   > この CSV ファイルの "File version" 列に、一部のファイルで誤って "Not applicable" と表示されています。 そのことが原因で、サードパーティのスキャン検出ツールを使用してビルドを検証したときに擬陽性または偽陰性の結果が生じる可能性があります。
