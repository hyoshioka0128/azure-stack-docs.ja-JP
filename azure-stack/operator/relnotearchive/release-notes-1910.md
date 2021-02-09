---
title: Azure Stack Hub 1910 リリース ノート
description: 更新プログラムやバグ修正プログラムを含む、Azure Stack Hub 統合システムの 1910 リリース ノート。
author: sethmanheim
ms.topic: article
ms.date: 01/25/2021
ms.author: sethm
ms.reviewer: sranthar
ms.lastreviewed: 09/09/2020
ROBOTS: NOINDEX
ms.openlocfilehash: ced055a2983fb4aabb1c31f314ccb4587f3801cc
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99248769"
---
# <a name="azure-stack-hub-1910-release-notes"></a>Azure Stack Hub 1910 リリース ノート

この記事では、Azure Stack Hub 更新プログラム パッケージの内容について説明します。 更新プログラムには、Azure Stack Hub の最新のリリースに対する機能強化と修正が含まれています。

> [!IMPORTANT]  
> この更新プログラム パッケージは、Azure Stack Hub 統合システム専用です。 Azure Stack Development Kit (ASDK) にこの更新プログラム パッケージを適用しないでください。

> [!IMPORTANT]  
> お使いの Azure Stack Hub インスタンスが 2 つ前の更新プログラムより古い場合、コンプライアンスに対応していないとみなされます。 [サポートを受けるためには、少なくともサポートされる最小バージョンまで更新する](../azure-stack-servicing-policy.md#keep-your-system-under-support)必要があります。

## <a name="update-planning"></a>計画の更新

更新プログラムを適用する前に、必ず次の情報を確認してください。

- [更新プログラム適用前後のアクティビティのチェックリスト](../release-notes-checklist.md)
- [既知の問題](../known-issues.md)
- [修正プログラム](#hotfixes)
- [セキュリティ更新プログラム](../release-notes-security-updates.md)

更新プログラムのトラブルシューティングと更新プロセスの詳細については、[Azure Stack Hub の修正プログラムと更新プログラムについての問題のトラブルシューティング](../azure-stack-troubleshooting.md)に関するページを参照してください。

## <a name="download-the-update"></a>更新プログラムをダウンロードする

Azure Stack Hub 更新プログラム パッケージは、[Azure Stack Hub 更新プログラム ダウンローダー ツール](https://aka.ms/azurestackupdatedownload)を使用してダウンロードできます。

## <a name="1910-build-reference"></a>1910 ビルドのリファレンス

Azure Stack Hub 1910 更新プログラムのビルド番号は **1.1910.0.58** です。

### <a name="update-type"></a>更新の種類

1908 以降、Azure Stack Hub が実行される基盤のオペレーティング システムは Windows Server 2019 に更新されました。 この更新プログラムによって、核となる基本的な機能強化と、Azure Stack Hub に機能を追加する機能が使用可能になります。

Azure Stack Hub 1910 更新プログラムのビルドの種類は **高速** です。

1910 更新プログラム パッケージは、以前の更新プログラムと比較してサイズが大きくなっており、ダウンロードにより長い時間がかかります。 更新プログラムは長い時間、**準備中** 状態のままになり、オペレーターは以前の更新プログラムよりもこのプロセスに長い時間がかかることを予想できます。 1910 更新プログラムが完了するまでの予測所要時間は、ご使用の Azure Stack Hub 環境内の物理ノード数に関係なく、約 10 時間です。 更新プログラムの正確なランタイムは一般的に、ご使用のシステムでテナント ワークロードによって使用されている容量、システム ネットワーク接続 (インターネットに接続されている場合)、およびシステム ハードウェアの仕様に左右されます。 ランタイムが予測値よりも長くなることは珍しいわけではなく、更新が失敗した場合を除いて、Azure Stack Hub オペレーターによるアクションは必要ありません。 このおおよその実行時間は、1910 更新プログラムに固有であり、他の Azure Stack Hub 更新プログラムと比較することはできません。

更新プログラムのビルドの種類については、「[Azure Stack Hub での更新プログラム管理の概要](../azure-stack-updates.md)」を参照してください。

<!-- ## What's in this update -->

<!-- The current theme (if any) of this release. -->

### <a name="whats-new"></a>新機能

<!-- What's new, also net new experiences and features. -->

- 管理者ポータルでは、リージョンのプロパティ メニューに特権エンドポイントの IP アドレスが表示され、簡単に見つけられるようになりました。 さらに、現在構成されているタイム サーバーと DNS フォワーダーも表示されます。 詳細については、「[Azure Stack Hub で特権エンドポイントを使用する](../azure-stack-privileged-endpoint.md)」を参照してください。

- Azure Stack Hub の正常性および監視システムでは、エラーが発生した場合、さまざまなハードウェア コンポーネントに対してアラートを生成できるようになりました。 これらのアラートには、追加の構成が必要になります。 詳細については、「[Azure Stack Hub のハードウェア コンポーネントを監視する](../azure-stack-hardware-monitoring.md)」を参照してください。

- [Azure Stack Hub の cloud-init のサポート](/azure/virtual-machines/linux/using-cloud-init):cloud-Init は、Linux VM を初回起動時にカスタマイズするために広く使用されているアプローチです。 cloud-init を使って、パッケージをインストールしてファイルを書き込んだり、ユーザーとセキュリティを構成したりすることができます。 cloud-init は初回起動プロセスの間に呼び出されるので、構成を適用するために追加の手順や必要なエージェントはありません。 マーケットプレースの Ubuntu イメージは、プロビジョニング用の cloud-init をサポートするように更新されました。

- Azure Stack Hub では、すべての Windows Azure Linux エージェント バージョンバージョンが Azure としてサポートされるようになりました。

- Azure Stack Hub 管理 PowerShell モジュールの新しいバージョンがリリースされました。 <!-- For more information, see -->

- 2020 年 4 月 15 日に、Azure Stack Hub 用の新しい Azure PowerShell テナント モジュールがリリースされました。 現在使用されている Azure RM モジュールは引き続き機能しますが、ビルド 2002 の後は更新されません。

- Azure Stack Hub インフラストラクチャの Windows Defender 定義の手動更新を構成するために、特権エンドポイント (PEP) に **Set-AzSDefenderManualUpdate** コマンドレットを追加しました。 詳細については、「[Azure Stack Hub 上で Windows Defender ウイルス対策を更新する](../azure-stack-security-av.md)」を参照してください。

- 特権エンドポイント (PEP) に Azure Stack Hub の DNS サーバーのフォワーダー設定を変更する **Set-AzSDnsForwarder** コマンドレットを追加しました。 DNS 構成の詳細については、「[Azure Stack Hub データセンターの DNS の統合](../azure-stack-integrate-dns.md)」を参照してください。

- [AKS エンジン](../../user/azure-stack-kubernetes-aks-engine-overview.md)を使用する **Kubernetes クラスター** の管理のサポートを追加しました。 この更新プログラムから、運用環境の Kubernetes クラスターをデプロイできるようになりました。 AKS エンジンによって、以下のことが可能になっています。
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

- Azure Stack Hub では、以前に更新の失敗を引き起こしたり、オペレーターによる Azure Stack Hub の更新開始を妨げたりしていた修正プログラムや更新プログラムの問題を自動修復する機能が向上しています。 その結果、**Test-AzureStack -UpdateReadiness** グループに含まれるテストの数が減っています。 詳細については、「[Azure Stack Hub システムの状態を検証する](../azure-stack-diagnostic-test.md#groups)」を参照してください。 次の 3 つのテストは **UpdateReadiness** グループに残っています。

  - **AzSInfraFileValidation**
  - **AzSActionPlanStatus**
  - **AzsStampBMCSummary**

- 外部デバイス (USB キーなど) がいつ Azure Stack Hub インフラストラクチャのノードにマウントされたかを報告する監査ルールが追加されました。 監査ログは syslog を介して出力され、"**Microsoft-Windows-Security-Auditing:6416|Plug and Play Events** (Microsoft-Windows-Security-Auditing: 6416|プラグ アンド プレイ イベント)" と表示されます。 syslog クライアントの構成方法の詳細については、[Syslog の転送](../azure-stack-integrate-security.md)に関する記事を参照してください。

- 内部証明書については、Azure Stack Hub では 4096 ビット RSA キーに移行中です。 内部シークレットのローテーションを実行すると、以前の 2048 ビット証明書は 4096 ビット長の証明書に置き換えられます。 Azure Stack Hub のシークレット ローテーションの詳細については、「[Azure Stack Hub でシークレットをローテーションする](../azure-stack-rotate-secrets.md)」を参照してください。

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

   これらの変更は、[既定の IPsec/IKE 提案](../../user/azure-stack-vpn-gateway-settings.md#ipsecike-parameters)のドキュメントにも反映されます。

- インフラストラクチャ バックアップ サービスのロジックは、固定のしきい値を利用するのではなく、バックアップに必要な空き領域を計算するように改善されました。 このサービスでは、バックアップのサイズ、アイテム保持ポリシー、予約、および外部ストレージの場所の現在の使用率が使用され、オペレーターに警告を発する必要があるかどうかが決定されます。

### <a name="changes"></a>[変更点]

- Azure から Azure Stack Hub にマーケットプレース項目をダウンロードするときに、複数のバージョンが存在する場合、項目のバージョンを指定できる新しいユーザー インターフェイスが追加されました。 新しい UI は、接続されたシナリオと切断されたシナリオの両方で使用できます。 詳細については、「[Azure から Azure Stack Hub に Marketplace の項目をダウンロードする](../azure-stack-download-azure-marketplace-item.md)」を参照してください。  

- 1910 リリース以降、Azure Stack Hub システムには追加で /20 プライベート内部 IP 空間が **必要** になりました。 詳細については、「[Azure Stack のためのネットワーク統合計画](../azure-stack-network.md)」を参照してください。
  
- アップロード手順中に外部ストレージの場所の容量が不足すると、インフラストラクチャ バックアップ サービスによって、部分的にアップロードされたバックアップ データが削除されます。  

- インフラストラクチャ バックアップ サービスでは、AAD デプロイのためのバックアップ ペイロードに ID サービスが追加されました。  

- 1910 リリースでは、Azure Stack Hub PowerShell モジュールがバージョン 1.8.0 に更新されました。<br>変更内容:
   - **新しい DRP 管理モジュール**:デプロイ リソース プロバイダー (DRP) を使用すると、Azure Stack Hub に対するリソース プロバイダーの調整されたデプロイが可能になります。 これらのコマンドを使うと、DRP とやり取りする Azure Resource Manager レイヤーとやり取りできます。
   - **BRP**: <br />
           - Azure Stack インフラストラクチャ バックアップの 1 つのロールの復元をサポートします。 <br />
           - パラメーター `RoleName` をコマンドレット `Restore-AzsBackup` に追加します。
   - **FRP**:API バージョン `2019-05-01` の **ドライブ** リソースと **ボリューム** リソースの重大な変更。 これらの機能は Azure Stack Hub 1910 以降でサポートされています。 <br />
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

Azure Stack Hub のこの更新でのセキュリティ更新プログラムについては、「[Azure Stack Hub のセキュリティ更新プログラム](../release-notes-security-updates.md)」を参照してください。

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

この更新プログラムをインストールした後、適用可能な修正プログラムがあればインストールします。 詳細については、[サービス ポリシー](../azure-stack-servicing-policy.md)に関する記事を参照してください。

<!-- One of these. Either no updates at all, nothing is required, or the LATEST hotfix that is required-->
- [Azure Stack Hub 修正プログラム 1.1910.63.186](https://support.microsoft.com/help/4574735)
