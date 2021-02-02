---
title: Azure Stack HCI と Windows Server の比較
description: このトピックは、Azure Stack HCI または Windows Server が組織に適しているかどうかを判断するのに役立ちます。
ms.topic: conceptual
author: khdownie
ms.author: v-kedow
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 01/21/2021
ms.openlocfilehash: c691424251d096794315880106c131ecaaea1f57
ms.sourcegitcommit: 925351b77490364b3d52746f788c4c1b93343631
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/22/2021
ms.locfileid: "98705201"
---
# <a name="compare-azure-stack-hci-to-windows-server-2019"></a>Azure Stack HCI と Windows Server 2019 を比較する

> 適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019

このトピックでは、Azure Stack HCI と Windows Server 2019 の主な違いについて説明し、それぞれの使用に適した状況に関するガイダンスを示します。 どちらの製品も、Microsoft によって活発にサポートおよび管理されています。 これらは異なる補完的な目的を意図されているため、多くの組織では、両方とも展開することを選択できます。

## <a name="when-to-use-azure-stack-hci"></a>Azure Stack HCI を使用する局面

Azure Stack HCI は、オンプレミスで VM または仮想デスクトップを実行するための Microsoft の Premier ハイパーコンバージド インフラストラクチャ プラットフォームであり、Azure ハイブリッド サービスに接続できます。 データセンターやブランチ オフィスを最新化してセキュリティで保護し、業界最高のパフォーマンスと低遅延およびデータ主権を実現するための、簡単な方法です。


:::image type="content" source="media/compare-windows-server/hci-scenarios.png" alt-text="Windows Server 2019 より Azure Stack HCI を使用する方がよい場合" border="false" lightbox="media/compare-windows-server/hci-scenarios.png":::


以下の場合は、Azure Stack HCI を使用します。

- コア データセンターの既存のワークロード用、またはブランチ オフィスやエッジの場所での新しい要件用に、インフラストラクチャを最新化するための最適な仮想化ホスト
- Azure サブスクリプションから定期的に送られてくるイノベーションと、ツールとエクスペリエンスの一貫したセットによる、クラウドの簡単な拡張性
- ハイパーコンバージド インフラストラクチャのすべてのベネフィット: 高速のストレージとネットワークを使用する、よりシンプルで統合されたデータセンター アーキテクチャ

  >[!NOTE]
  >Azure Stack HCI は、最新のハイパーコンバージド アーキテクチャ用の Hyper-V 仮想化ホストとして使用することが意図されているため、ゲスト権限は含まれません。 このため、Azure Stack HCI は、少数のサーバーの役割を直接実行するためだけにライセンスを付与されます。他のロールはすべて、VM 内で実行する必要があります。

## <a name="when-to-use-windows-server-2019"></a>Windows Server 2019 を使用する局面

Windows Server 2019 は、非常に汎用性のある多目的なオペレーティング システムであり、ゲスト権限など、数十の役割と数百の機能を備えています。 Windows Server マシンは、クラウドでもオンプレミスでも使用でき、Azure Stack HCI での仮想化も含まれます。


:::image type="content" source="media/compare-windows-server/windows-server-scenarios.png" alt-text="Azure Stack HCI より Windows Server 2019 を使用する方がよい場合" border="false" lightbox="media/compare-windows-server/windows-server-scenarios.png":::


以下の場合は、Windows Server 2019 を使用します。

- 仮想マシン (VM) またはコンテナー内のゲスト オペレーティング システム
- Windows アプリケーション用のランタイム
- Active Directory、ファイル サービス、DNS、DHCP、インターネット インフォメーション サービス (IIS) など、1 つ以上の組み込みサーバー ロールを使用する場合
- ベアメタル ドメイン コントローラーや SQL Server インストールなどの従来のサーバー
- ファイバー チャネル SAN ストレージに接続された VM などの従来のインフラストラクチャ

## <a name="compare-product-positioning"></a>製品の位置づけを比較する

次の表は、Azure Stack HCI と Windows Server 2019 の高レベルの製品パッケージを示したものです。

| **属性**    | **Azure Stack HCI** | **Windows Server 2019** |
| ---------------- | ------------------- | ----------------------- |
| 製品の種類     | オペレーティング システムなどが含まれるクラウド サービス | オペレーティング システム |
| 法務            | Microsoft 顧客契約またはオンライン サブスクリプション契約の対象 | 固有の使用許諾契約 |
| ライセンス        | Azure サブスクリプションに対して課金 | 固有の有料ライセンス |
| サポート          | Azure サポートの対象 | Microsoft Premier サポートなど、異なるサポート契約の対象にすることが可能 |
| 情報の入手元  | [Azure.com/HCI](https://azure.com/hci) からのダウンロード、または統合システムにプレインストール | Microsoft ボリューム ライセンス サービス センターまたは評価センター |
| VM での実行      | 評価の場合のみ、ホスト OS として意図 | 可能、クラウド内またはオンプレミス |
| ハードウェア         | [Azure Stack HCI カタログ](https://hcicatalog.azurewebsites.net)の 200 以上の事前検証済みソリューションのいずれでも実行可能 | "Certified for Windows Server 2019" のロゴが付いている任意のハードウェアで実行可能 |
| ライフサイクル ポリシー | 常に最新の機能で更新 | [Windows Server サービス チャネル](/windows-server/get-started-19/servicing-channels-19)を選択: 長期サービス チャネル (LTSC) または半期チャネル (SAC) |

## <a name="compare-technical-features"></a>技術的な機能を比較する

次の表は、Azure Stack HCI と Windows Server 2019 の技術的機能を比較したものです。

| **属性** | **Azure Stack HCI** | **Windows Server 2019** |
| ------------- | ------------------- | ----------------------- |
| コア Hyper-V | はい | はい |
| コア記憶域スペース ダイレクト | はい | はい |
| コア SDN | はい | はい |
| ディザスター リカバリーのための拡張クラスタリング | はい | - |
| 4 から 5 倍高速の記憶域スペース修復 | はい | - |
| 統合されたドライバーとファームウェアの更新プログラム | はい (統合システムのみ) | - |
| ガイド付き展開 | はい | - |

## <a name="compare-management-options"></a>管理オプションを比較する

次の表は、Azure Stack HCI と Windows Server 2019 の管理オプションを比較したものです。 どちらの製品もリモート管理向けに設計されており、多くの同じツールで管理できます。

| **属性** | **Azure Stack HCI** | **Windows Server 2019** |
| ------------- | ------------------- | ----------------------- |
| デスクトップ エクスペリエンス | - | はい |
| Windows Admin Center | はい | はい |
| Microsoft System Center | はい (個別に販売) | はい (個別に販売) |
| Azure portal | はい (ネイティブ) | Arc エージェントが必要 |
| サードパーティ製のツール | はい | はい |

## <a name="compare-product-pricing"></a>製品の価格を比較する

次の表は、Azure Stack HCI と Windows Server 2019 の製品価格を比較したものです。 詳細については、「[Azure Stack HCI の価格](https://azure.microsoft.com/pricing/details/azure-stack/hci/)」を参照してください。

| **属性** | **Azure Stack HCI** | **Windows Server 2019** |
| ------------- | ------------------- | ----------------------- |
| 価格の種類 | 購読サービス | さまざま: ほとんどの場合、1 回限りのライセンス |
| 価格体系 | コアあたり、月あたり | さまざま: 通常はコアあたり |
| 価格 | コアあたり $10 USD/月 | 「[Windows Server 2019 の価格とライセンス体系](https://www.microsoft.com/windows-server/pricing)」を参照 |
| 評価および試用の期間 | 登録後に 30 日間の無料試用 | 180 日の評価版コピー |
| チャネル | Enterprise Agreement、クラウド サービス プロバイダー、または直接 | Enterprise Agreement/ボリューム ライセンス、OEM、サービス プロバイダー ライセンス契約 (SPLA) |

## <a name="next-steps"></a>次のステップ

- [Azure Stack HCI と Azure Stack Hub の比較](compare-azure-stack-hub.md)
