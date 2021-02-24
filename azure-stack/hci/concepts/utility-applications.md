---
title: Azure Stack HCI 用のユーティリティ アプリケーション
description: このトピックでは、Azure Stack HCI オペレーティング システムをサポートする Microsoft パートナーのユーティリティ アプリケーションを紹介します。
author: burcuerdemir
ms.author: burcue
ms.topic: conceptual
ms.date: 01/22/2021
ms.openlocfilehash: 79490100adaef813af6b8df05779620b315765b3
ms.sourcegitcommit: ea4bb7bf0ba1bd642c769013a0280f24e71550bc
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2021
ms.locfileid: "98718000"
---
# <a name="utility-applications-for-azure-stack-hci"></a>Azure Stack HCI 用のユーティリティ アプリケーション

>適用対象:Azure Stack HCI バージョン 20H2

Azure Stack HCI により、ハイパーコンバージド インフラストラクチャ (HCI) テクノロジが Azure Stack ファミリに統合されているので、オンプレミスで使い慣れた方法でユーティリティ アプリケーションを実行し、Azure サービスに簡単にアクセスすることができます。 Azure Stack は、クラウド機能をオンプレミスのデータセンターと統合する Microsoft 独自のハイブリッド クラウド オファリングです。 詳細については、「[Azure Stack HCI ソリューションの概要](../overview.md)」を参照してください。

Azure Stack HCI 用の独立系ソフトウェア ベンダー (ISV) プログラムは、有効なライセンス アプリケーションを実行する有効なライセンスの仮想マシン (VM) のホスト、管理、およびサービス提供を行う、Azure Stack HCI オペレーティング システム上で直接実行されるユーティリティを対象としています。 このトピックでは、このオペレーティング システムをサポートする Microsoft パートナーのユーティリティ アプリケーションを紹介します。

このオペレーティング システムには Microsoft Defender ウイルス対策を使用することをお勧めします。 他のウイルス対策およびマルウェア検出ソフトウェアおよびサービスの場合、更新プログラムを受信するオペレーティング システムの機能に影響する可能性があります。 このガイダンスは、オペレーティング システムの今後のリリースで変更される可能性があります。

## <a name="partner-spotlight"></a>パートナーの紹介
このオペレーティング システムをサポートするユーティリティ アプリケーションは、次の Microsoft パートナーから入手できます。

:::image type="content" source="./media/utility-applications/altaro-logo.png" alt-text="Altaro 社のロゴ" border="false":::

Altaro VM Backup は、Hyper-V および VMware 環境用の受賞歴のある VM バックアップおよびレプリケーション ソリューションです。 このソリューションには、IT 部門、IT リセラーおよびコンサルタント、マネージド サービス プロバイダー (MSP) 向けに設計されており、堅牢で合理化されたエンタープライズ レベルの機能が提供されています。

- アプリケーションの一覧:[Virtual Machine Backup and Replication for Hyper-V and VMware](https://www.altaro.com/vm-backup/)
- アプリケーションのサポート:[Altaro による Azure Stack HCI のサポート](https://www.altaro.com/news/single/News-Altaro-applies-its-expertise-in-Hyper-V-backup-to-support-Microsoft.php)
- レプリケーションのサポート: [Altaro WAN-Optimized Replication](https://www.altaro.com/vm-backup/wan-optimized-replication.php)

:::image type="content" source="./media/utility-applications/veeam-logo.png" alt-text="Veeam 社のロゴ" border="false":::

Veeam Backup & Replication は、すべてのクラウド、仮想、および物理のワークロードを、シンプルで信頼性が高く柔軟な方法で保護するための 1 つのソリューションです。 データ保護、ランサムウェア防止、コンプライアンスの課題の管理によるストレスを解消できます。

- アプリケーションの一覧:[Veeam Backup & Replication](https://www.veeam.com/vm-backup-recovery-replication-software.html)
- アプリケーションのサポート:[Veeam による Azure Stack HCI のサポート](https://www.veeam.com/kb4047)
- [Replication による Hyper-V VM サポート](https://www.veeam.com/vm-advanced-replication.html?ad=in-text-link)

:::image type="content" source="./media/utility-applications/commvault-logo.png" alt-text="Commvault 社のロゴ" border="false":::

Commvault は、オンプレミスとクラウドの場所全体でデータを格納、保護、管理、および使用する機能が一式そろったデータ管理プラットフォームです。

- アプリケーションの一覧:[Microsoft Azure Stack](https://www.commvault.com/supported-technologies/microsoft/azurestack)
- アプリケーションのサポート:[Azure Stack HCI](https://documentation.commvault.com/11.21/essential/132799_microsoft_azure_stack_hci.html)
- [Commvault のバックアップと復旧で Microsoft Azure Stack HCI がサポートされるようになりました](https://www.commvault.com/blogs/commvault-backup-and-recovery-now-supports-microsoft-azure-stack-hci)

:::image type="content" source="./media/utility-applications/veritas-logo.png" alt-text="Veritas 社のロゴ" border="false":::

Veritas Backup Exec は、環境の仮想、物理、およびクラウドのリソースを保護するために役立つ、使いやすいバックアップおよび回復ソリューションです。

- アプリケーションの一覧:[Backup Exec](https://www.veritas.com/protection/backup-exec)
- アプリケーションのサポート:[Protect Azure Stack HCI with Backup Exec (Backup Exec を使用して Azure Stack HCI を保護する)](https://www.veritas.com/support/en_US/article.100048860)
- [Veritas Backup Exec による Microsoft Azure のサポート](https://www.veritas.com/protection/backup-exec/azure)

:::image type="content" source="./media/utility-applications/datadog-logo.png" alt-text="Datadag 社のロゴ" border="false":::

Datadog は、インフラストラクチャの監視、アプリケーション パフォーマンスの監視、ログ管理、ユーザー エクスペリエンスの監視などを網羅する完全に統合されたプラットフォームです。

- アプリケーションの一覧:[モダンモニタリングとセキュリティ](https://www.datadoghq.com/)
- アプリケーションのサポート:[Microsoft Azure Stack HCI を Datadog を使用して監視する](https://www.datadoghq.com/blog/monitor-azure-stack-hci-datadog)

## <a name="next-steps"></a>次のステップ
詳細については、次のトピックも参照してください。
- [Azure Stack HCI](https://azure.microsoft.com/products/azure-stack/hci/)
- [Microsoft Defender ウイルス対策](/windows/security/threat-protection/microsoft-defender-antivirus/microsoft-defender-antivirus-in-windows-10)