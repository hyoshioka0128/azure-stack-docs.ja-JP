---
title: Azure Stack HCI のリリース ノート
description: Azure Stack HCI バージョン 20H2 のリリース ノート。
author: khdownie
ms.author: v-kedow
ms.topic: conceptual
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 10/13/2020
ms.openlocfilehash: 2432a7fb28ba65f08b0540113ec5d3f90f742509
ms.sourcegitcommit: 64060ff02d2450c6cf91cb21cdefdcf6b720a75f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/13/2020
ms.locfileid: "92009847"
---
# <a name="release-notes-for-azure-stack-hci-public-preview"></a>Azure Stack HCI パブリック プレビューのリリース ノート

> 適用対象:Azure Stack HCI バージョン 20H2

この記事では、Azure Stack HCI パブリック プレビューの更新プログラム パッケージの内容について説明します。

## <a name="october-13-2020-security-update-kb4580363"></a>2020 年 10 月 13 日のセキュリティ更新プログラム (KB4580363)

更新プログラムには、Azure Stack HCI の最新のリリースに対する機能強化と修正が含まれています。

### <a name="improvements-and-fixes"></a>機能強化と修正
この更新プログラムには、内部の OS 機能に対するさまざまなセキュリティ強化が含まれています。 今回のリリースに関して別途文書化された問題はありません。

解決済みのセキュリティの脆弱性について詳しくは、[セキュリティ更新プログラム ガイド](https://portal.msrc.microsoft.com/security-guidance)を参照してください。

### <a name="known-issues-in-this-update"></a>この更新プログラムの既知の問題
現在、この更新プログラムに関して Microsoft が把握している問題はありません。

### <a name="how-to-get-this-update"></a>この更新プログラムを入手する方法
[Azure Stack HCI プレビュー](https://azure.microsoft.com/products/azure-stack/hci/hci-download/)向けの 2020 年 10 月 13 日のセキュリティ更新プログラム (KB4580363) は、Windows Update で配信されます。 Azure Stack HCI クラスターへのインストール方法については、「[Azure Stack HCI クラスターを更新する](manage/update-cluster.md)」を参照してください。

### <a name="file-information"></a>ファイル情報
この更新プログラム (OS ビルド 17784.1288) で提供されるファイルの一覧については、[累積的な更新プログラム 4580363 のファイル情報](https://download.microsoft.com/download/f/7/8/f78801f0-e7e5-4a3d-a971-f642ccd24ee4/4580363.csv)をダウンロードしてください。

   > [!NOTE]
   > この CSV ファイルの "File version" 列に、一部のファイルで誤って "Not applicable" と表示されています。 そのことが原因で、サードパーティのスキャン検出ツールを使用してビルドを検証したときに擬陽性または偽陰性の結果が生じる可能性があります。

## <a name="october-13-2020-servicing-stack-update-kb4583287"></a>2020 年 10 月 13 日のサービス スタック更新プログラム (KB4583287)

この更新プログラムには、Azure Stack HCI の最新リリースの品質向上が含まれています。

### <a name="improvements-and-fixes"></a>機能強化と修正
この更新プログラムにより、更新プログラムをインストールするコンポーネントであるサービス スタックの品質が向上します。 サービス スタック更新プログラム (SSU) により、堅牢で信頼性の高いサービス スタックを使用して、お使いのデバイスで Microsoft 更新プログラムを受信し、インストールできるようになります。

### <a name="how-to-get-this-update"></a>この更新プログラムを入手する方法
[Azure Stack HCI プレビュー](https://azure.microsoft.com/products/azure-stack/hci/hci-download/)向けの 2020 年 10 月 13 日のサービス スタック更新プログラム (KB4583287) は Windows Update で配信されます。 Azure Stack HCI クラスターへのインストール方法については、「[Azure Stack HCI クラスターを更新する](manage/update-cluster.md)」を参照してください。

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

更新プログラムには、Azure Stack HCI の最新のリリースに対する機能強化と修正が含まれています。

### <a name="improvements-and-fixes"></a>機能強化と修正
セキュリティ関連でないこの更新プログラムには、品質上の機能強化が含まれています。 重要な変更点は、次のとおりです。
- マルチプレクサーを経由するソフトウェア ロード バランサー (SLB) のトラフィックが、アプリケーション接続エラーを引き起こす可能性がある別のホストにリダイレクトされる可能性がある問題に対処しました。

### <a name="known-issues-in-this-update"></a>この更新プログラムの既知の問題
現在、この更新プログラムに関して Microsoft が把握している問題はありません。

### <a name="how-to-get-this-update"></a>この更新プログラムを入手する方法
[Azure Stack HCI プレビュー](https://azure.microsoft.com/products/azure-stack/hci/hci-download/)向けの 2020 年 9 月 17 日のセキュリティ更新プログラム (KB4577629) は、Windows Update で配信されます。 Azure Stack HCI クラスターへのインストール方法については、「[Azure Stack HCI クラスターを更新する](manage/update-cluster.md)」を参照してください。

### <a name="file-information"></a>ファイル情報
この更新プログラム (OS ビルド 17784.1259) で提供されるファイルの一覧については、[累積的な更新プログラム 4577629 のファイル情報](https://download.microsoft.com/download/9/1/a/91addcbb-2b36-408c-ab88-736de42edb98/4577629.csv)をダウンロードしてください

   > [!NOTE]
   > この CSV ファイルの "File version" 列に、一部のファイルで誤って "Not applicable" と表示されています。 そのことが原因で、サードパーティのスキャン検出ツールを使用してビルドを検証したときに擬陽性または偽陰性の結果が生じる可能性があります。

## <a name="september-8-2020-security-update-kb4577470"></a>2020 年 9 月 8 日のセキュリティ更新プログラム (KB4577470)

更新プログラムには、Azure Stack HCI の最新のリリースに対する機能強化と修正が含まれています。

### <a name="improvements-and-fixes"></a>機能強化と修正
この更新プログラムには、内部の OS 機能に対するさまざまなセキュリティ強化が含まれています。 今回のリリースに関して別途文書化された問題はありません。

解決済みのセキュリティの脆弱性について詳しくは、[セキュリティ更新プログラム ガイド](https://portal.msrc.microsoft.com/security-guidance)を参照してください。

### <a name="known-issues-in-this-update"></a>この更新プログラムの既知の問題
現在、この更新プログラムに関して Microsoft が把握している問題はありません。

### <a name="how-to-get-this-update"></a>この更新プログラムを入手する方法
[Azure Stack HCI プレビュー](https://azure.microsoft.com/products/azure-stack/hci/hci-download/)向けの 2020 年 9 月 8 日のセキュリティ更新プログラム (KB4577470) は Windows Update で配信されます。 Azure Stack HCI クラスターへのインストール方法については、「[Azure Stack HCI クラスターを更新する](manage/update-cluster.md)」を参照してください。

### <a name="file-information"></a>ファイル情報
この更新プログラム (OS ビルド 17784.1226) で提供されるファイルの一覧については、[累積的な更新プログラム 4577470 のファイル情報](https://download.microsoft.com/download/3/c/4/3c468525-5867-4cc3-8d34-dba88989adab/4577470.csv)をダウンロードしてください。

   > [!NOTE]
   > この CSV ファイルの "File version" 列に、一部のファイルで誤って "Not applicable" と表示されています。 そのことが原因で、サードパーティのスキャン検出ツールを使用してビルドを検証したときに擬陽性または偽陰性の結果が生じる可能性があります。

## <a name="september-8-2020-servicing-stack-update-kb4577558"></a>2020 年 9 月 8 日のサービス スタック更新プログラム (KB4577558)

この更新プログラムには、Azure Stack HCI の最新リリースの品質向上が含まれています。

### <a name="improvements-and-fixes"></a>機能強化と修正
この更新プログラムにより、更新プログラムをインストールするコンポーネントであるサービス スタックの品質が向上します。 サービス スタック更新プログラム (SSU) により、堅牢で信頼性の高いサービス スタックを使用して、お使いのデバイスで Microsoft 更新プログラムを受信し、インストールできるようになります。

### <a name="how-to-get-this-update"></a>この更新プログラムを入手する方法
[Azure Stack HCI プレビュー](https://azure.microsoft.com/products/azure-stack/hci/hci-download/)向けの 2020 年 9 月 8 日のサービス スタック更新プログラム (KB4577558) は Windows Update で配信されます。 Azure Stack HCI クラスターへのインストール方法については、「[Azure Stack HCI クラスターを更新する](manage/update-cluster.md)」を参照してください。

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

更新プログラムには、Azure Stack HCI の最新のリリースに対する機能強化と修正が含まれています。

### <a name="improvements-and-fixes"></a>機能強化と修正
この更新プログラムには、内部の OS 機能に対するさまざまなセキュリティ強化が含まれています。 今回のリリースに関して別途文書化された問題はありません。

解決済みのセキュリティの脆弱性について詳しくは、[セキュリティ更新プログラム ガイド](https://portal.msrc.microsoft.com/security-guidance)を参照してください。

### <a name="known-issues-in-this-update"></a>この更新プログラムの既知の問題
現在、この更新プログラムに関して Microsoft が把握している問題はありません。

### <a name="how-to-get-this-update"></a>この更新プログラムを入手する方法
[Azure Stack HCI プレビュー](https://azure.microsoft.com/products/azure-stack/hci/hci-download/)向けの 2020 年 8 月 11 日のセキュリティ更新プログラム (KB4574585) は Windows Update で配信されます。 Azure Stack HCI クラスターへのインストール方法については、「[Azure Stack HCI クラスターを更新する](manage/update-cluster.md)」を参照してください。

### <a name="file-information"></a>ファイル情報
この更新プログラム (OS ビルド 17784.1162) で提供されるファイルの一覧については、[累積的な更新プログラム 4574585 のファイル情報](https://download.microsoft.com/download/7/f/4/7f451def-76c5-4cc0-9929-0c5efeb27d2f/4574585.csv)をダウンロードしてください。

   > [!NOTE]
   > この CSV ファイルの "File version" 列に、一部のファイルで誤って "Not applicable" と表示されています。 そのことが原因で、サードパーティのスキャン検出ツールを使用してビルドを検証したときに擬陽性または偽陰性の結果が生じる可能性があります。