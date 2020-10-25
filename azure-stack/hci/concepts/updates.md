---
title: 更新とアップグレード
description: Azure Stack HCI に更新プログラムとアップグレードを適用する方法の概要について説明します。
author: khdownie
ms.author: v-kedow
ms.topic: conceptual
ms.date: 07/21/2020
ms.openlocfilehash: f56853ce64575d32ae10ccf7ac2aa0a1a640ec5b
ms.sourcegitcommit: 0e52f460295255b799bac92b40122a22bf994e27
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/21/2020
ms.locfileid: "86868176"
---
# <a name="updates-and-upgrades"></a>更新とアップグレード

> 適用対象:Azure Stack HCI バージョン 20H2

Azure Stack HCI ソリューションは、更新とアップグレードのエクスペリエンスが予測可能なように設計されています。

## <a name="support-period"></a>サポート期間

少なくとも 5 年間は Microsoft ハードウェア ソリューション パートナーからハードウェア サービス、サポート、セキュリティ更新プログラムを期待できます。 Azure Stack HCI のオペレーティング システムは、Microsoft によって 2 年間サポートされ、お客様は年間機能更新プログラムを最大 1 年間デプロイできます。

:::image type="content" source="media/updates/lifecycle-versions.png" alt-text="Azure Stack HCI の年間リリース ロードマップ" border="false":::

## <a name="updating-azure-stack-hci"></a>Deploy Azure Stack HCI の更新

Windows 10 と同様に、Azure Stack HCI は、Microsoft からの毎月の品質とセキュリティの更新プログラムによって、常に最新の状態に保たれます。 Windows Server の場合と同様に、クラスター対応更新を使用して更新を開始します。

ハードウェア パートナーとソリューション ビルダーは、Windows Admin Center に接続して拡張機能を開発し、サーバーのファームウェア、ドライバー、BIOS を最新かつクラスター ノード間で整合した状態に保つことができます。 Azure Stack HCI がプレインストールされた統合システムを購入したお客様は、これらの拡張機能を通してソリューションの更新プログラムをインストールできます。検証済みハードウェア ノードを単に購入したお客様は、ハードウェア ベンダーの推奨事項に従って、更新プログラムを個別に実行することが必要になる場合があります。

仮想マシンも定期的に更新する必要があります。 Windows Update と Windows Server Update Services に加えて、Azure Update Management を使用して VM を更新することもできます。

## <a name="upgrading-to-azure-stack-hci"></a>Azure Stack HCI へのアップグレード

Windows Server 2019 から Azure Stack HCI にアップグレードできますが、シームレスではありません。現時点ではインプレース アップグレードはありません。 アップグレードを行うには、メンテナンス期間とサーバーごとにクリーン インストールが必要であり、ローリング クラスター アップグレードの一部として各ノードを個別に再構築します。

## <a name="cluster-aware-updating"></a>クラスター対応更新

クラスターを更新する場合、一部の更新プログラムではオペレーティング システムの再起動が必要であり、クラスターの可用性を確保するためにサーバーを 1 台ずつ再起動する必要があるため、オーケストレーションが必要です。 クラスター対応更新 (CAU) は Microsoft の既定のクラスター オーケストレーターであり、Windows Admin Center の統合更新エクスペリエンスから、または PowerShell コマンドを使用して手動で、更新を実行できます。 クラスター対応更新は、ファームウェアとドライバーの更新プログラムを取得するパートナー プラグインをサポートするように拡張できます。

## <a name="next-steps"></a>次のステップ

関連情報については、以下も参照してください。

- [Azure Stack HCI クラスターを更新する](../manage/update-cluster.md)
