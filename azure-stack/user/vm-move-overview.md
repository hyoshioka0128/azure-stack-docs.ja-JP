---
title: VM を Azure Stack Hub に移動する
description: VM を Azure Stack Hub に移動するさまざまな方法について説明します。
author: mattbriggs
ms.topic: conceptual
ms.date: 9/8/2020
ms.author: mabrigg
ms.reviewer: kivenkat
ms.lastreviewed: 9/8/2020
ms.openlocfilehash: 932fed2c6097fbbf41606f365b0c5a424bf1381b
ms.sourcegitcommit: 9a340b383dcf42c85bc6ec0d01ff3c9ae29dfe4c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/09/2020
ms.locfileid: "89609904"
---
# <a name="move-a-vm-to-azure-stack-hub-overview"></a>Azure Stack Hub への VM の移動の概要

仮想マシン (VM) をご使用の環境から Azure Stack Hub に移動できます。 ワークロードの移動を計画する場合、考慮する必要がある制限事項がいくつかあります。 この記事では、Azure Stack Hub での仮想ハード ディスク (VHD) の要件の一覧を示します。 Azure Stack Hub では、第 1 世代の VHD が必要です。 VM は汎用化または特殊化する必要があります。 Azure Stack で作成される VMS の基本イメージとして汎用化された VM を使用します。 特殊化された VM には、ユーザー アカウントが含まれます。 VHD を移行、準備、ダウンロードするには、VHD が要件を満たしていることを確認し、Azure Stack Hub のストレージ アカウントにイメージをアップロードしてから、クラウドに VM を作成します。 移行タスクがより複雑な場合は、「*Azure Stack Hub への移行*」のホワイトペーパーで詳細を確認できます。

カスタム イメージには、**汎用化**と**特殊化**の 2 つの形式があります。

- **汎用化イメージ**  
  汎用化ディスク イメージとは、ユーザー アカウントなどの固有の情報を削除するために **Sysprep** を使用して準備されたもので、これを再利用して複数の VM を作成することができます。 Azure Stack Hub クラウド オペレーターが Marketplace 項目として使用するイメージを作成する場合は、汎用化 VHD が適しています。 管理者ポータルまたは管理者エンドポイントを介して提供されるイメージは、**プラットフォーム イメージ**です。

- **特殊化イメージ**  
  特殊化ディスク イメージとは、既存の VM の仮想ハードディスク (VHD) のコピーで、元の VM のユーザー アカウント、アプリケーション、その他の状態データが含まれます。 これは通常、VM を Azure Stack Hub に移行するときに使用する形式です。 オンプレミスから Azure Stack Hub に VM を移行する必要がある場合は、特殊化 VHD が適しています。

イメージを Azure Stack Hub に移動する場合は、イメージをどのように使用するかを考慮します。

- **個人のワークロード**  
    開発または特定のタスクに使用するマシンをオンプレミス環境またはグローバル Azure に配置でき、Azure Stack Hub を使用したプライベート クラウドでそのマシンをホストするという利点を得ることができます。 マシンでデータとユーザー アカウントを保持することが望ましいです。 このマシンは特殊化イメージとして Azure Stack Hub に移動できます。

- **ゴールデン イメージ**  
    ワークグループ内で共通の VM 構成と一連のアプリケーションを共有したい場合があります。 Azure Stack Hub ドメイン (ディレクトリ テナント) の外部のユーザーとイメージを共有する必要はありません。 この場合は、データとユーザー アカウントを削除してイメージを汎用化できます。 その後、このイメージをテナント内の他のユーザーと共有できます。

- **Azure Stack Hub Marketplace オファリング**  
    クラウド オペレーターは、Marketplace オファリングの基礎として汎用化イメージを使用できます。 イメージを準備した後、クラウド オペレーターはカスタム イメージを使用して Azure Stack Hub インスタンスの Marketplace オファリングを作成できます。 ユーザーは、Marketplace の他のオファリングと同様に、イメージから独自の VM を作成できます。 このオファリングを作成するには、クラウド オペレーターと協力する必要があります。

## <a name="verify-vhd-requirements"></a>VHD の要件を確認する

> [!IMPORTANT]  
> VHD を準備するときは、次の要件が満たされている必要があります。そうでない場合、Azure Stack Hub で VHD を使用できません。
> イメージをアップロードする前に、以下を考慮します。
> - Azure Stack Hub が第 1 世代の VM からのイメージのみをサポートしている。
> - VHD が固定型である。 Azure Stack Hub では、ダイナミック ディスク VHD はサポートされていません。
> - VHD の最小仮想サイズが 20 MB 以上である。
> - VHD がアラインされている。つまり、仮想サイズを 1 MB の倍数にする必要があります。
> - VHD BLOB の長さ = 仮想サイズ + VHD フッターの長さ (512)。 BLOB 末尾の小さなフッターに、VHD のプロパティが記述されます。 

VHD を修復する手順については、「[VHD を検証する](vm-move-from-azure.md#verify-your-vhd)」を参照してください。

## <a name="methods-of-moving-a-vm"></a>VM を移動する方法

次のシナリオでは、VM を Azure Stack Hub に手動で移動できます。

| シナリオ | Instructions |
| --- | --- |
| グローバル Azure から Azure Stack Hub に移動 | グローバル Azure で VHD を準備し、Azure Stack Hub にアップロードします。 詳細については、「[VM を Azure から Azure Stack Hub に移動する](vm-move-from-azure.md)」を参照してください。 |
| ローカルで汎用化して Azure Stack Hub に移動 | VHD を準備し、その VHD を Hyper-V を使用してローカルで汎用化し、Azure Stack Hub にアップロードします。 詳細については、「[汎用化 VM をオンプレミスから Azure Stack Hub に移動する](vm-move-generalized.md)」を参照してください。 |
| ローカルで特殊化して Azure Stack Hub に移動 | Hyper-V を使用してローカルで特殊化された VHD を準備し、Azure Stack Hub にアップロードします。 詳細については、「[特殊化 VM をオンプレミスから Azure Stack Hub に移動する](vm-move-specialized.md)」を参照してください。 |

## <a name="migrate-to-azure-stack-hub"></a>Azure Stack Hub に移行する

Azure グローバルの AzureCAT の専門家が記述したガイドで、ワークロードを計画し、Azure Stack Hub に一括で移行するための詳細、チェックリスト、ベスト プラクティスを確認できます。 このガイドは、物理サーバーまたは既存の仮想化プラットフォームで実行される既存のアプリケーションの移行について重点的に説明しています。 これらのワークロードを Azure Stack Hub の IaaS 環境に移行することで、チームは、スムーズな運用、セルフサービス デプロイ、標準化されたハードウェア構成、Azure 整合性の恩恵を受けることができます。

[ホワイト ペーパーを入手してください。](https://azure.microsoft.com/resources/migrate-to-azure-stack-hub-patterns-and-practices-checklists/)

クラウド導入フレームワークでも移行に関するガイダンスを確認できます。 詳細については、「[Azure Stack Hub 移行の計画](/azure/cloud-adoption-framework/scenarios/azure-stack/plan)」を参照してください。 

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub VM の概要](azure-stack-compute-overview.md)

[Azure Stack Hub にカスタム VM イメージを追加する](../operator/azure-stack-add-vm-image.md)