---
title: 管理者ポータルで Azure Stack のバックアップを有効にする - MDC
description: エラーが発生した場合に Azure Stack を復元できるように、管理者ポータルで Infrastructure Backup サービスを有効にする方法を学習します。 Modular Data Center の場合。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: tzl
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/16/2019
ms.author: sethm
ms.reviewer: hectorl
ms.lastreviewed: 12/16/2019
ms.openlocfilehash: db0980511b2272a6c92c1644401ce1d577ddcf8d
ms.sourcegitcommit: d719f148005e904fa426a001a687e80730c91fda
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/06/2021
ms.locfileid: "97910944"
---
# <a name="configure-backup-for-azure-stack-hub-from-the-administrator-portal---modular-data-center-mdc"></a>管理者ポータルで Azure Stack Hub のバックアップを構成する - Modular Data Center (MDC)

*適用対象:Modular Data Center、Azure Stack Hub ラグド*

管理者ポータルから Infrastructure Backup サービスを構成して、インフラストラクチャのバックアップを外部の保存場所にエクスポートできます。 インフラストラクチャのバックアップは、サービスの低下を修正するためにサポートが使用できます。

## <a name="configure-backup"></a>バックアップの構成

1. [Azure Stack 管理者ポータル](../../operator/azure-stack-manage-portals.md)を開きます。

2. **[すべてのサービス]** を選択し、 **[管理]** カテゴリで **[インフラストラクチャ バックアップ]** を選択します。 **[Infrastructure backup]\(インフラストラクチャ バックアップ\)** ブレードで **[構成]** を選択します。

3. **バックアップ ストレージの場所** のパスを入力します。 別のデバイスでホストされるファイル共有へのパスの場合、汎用名前付け規則 (UNC) の文字列を使用します。 UNC 文字列は、共有ファイルやデバイスといった、リソースの場所を指定します。 サービスの場合は、IP アドレスを使用できます。 障害発生後にバックアップ データを確実に利用できるようにするためには、保存デバイスは別の場所に配置する必要があります。

    > [!NOTE]  
    > 使用する環境で Azure Stack インフラストラクチャ ネットワークからエンタープライズ環境への名前解決がサポートされている場合は、IP ではなく、完全修飾ドメイン名 (FQDN) を使用できます。

4. ファイルを読み書きするための十分なアクセス権を持つドメインとユーザー名を使用して **ユーザー名** を入力します。例: **Contoso\backupshareuser**

5. ユーザーの **パスワード** を入力します。

6. **[パスワードの確認]** にもう一度パスワードを入力します。

7. **頻度 (時間単位)** はバックアップの作成頻度を決定します。 既定値は 12 です。 スケジューラによって、最小値 4 から最大値 12 までがサポートされます。

8. **保有期間 (日単位)** は、バックアップが外部の保存場所に保持される日数を決定します。 既定値は 7 です。 スケジューラによって、最小値 2 から最大値 14 までがサポートされます。 保有期間より前のバックアップは、外部の保存場所から自動的に削除されます。

   > [!NOTE]
   > 保有期間より前のバックアップをアーカイブする場合は、スケジューラでバックアップを削除する前に必ずファイルをバックアップしてください。 バックアップの保有期間を短縮すると (たとえば、7 日を 5 日にする)、スケジューラは新しい保有期間より前のすべてのバックアップを削除します。 この値を更新する前に、バックアップが削除されても問題がないことを確認してください。

9. **[暗号化の設定]** では、公開証明書のサムプリントは、デプロイ中に指定された証明書を対象としています。 既存の証明書を更新する必要はありません。

10. **[OK]** を選択して、バックアップ コントローラーの設定を保存します。

    ![Azure Stack - バックアップ コントローラーの設定](media/azure-stack-backup-enable-backup-console-tzl/backup-controller-settings-certificate.png)

## <a name="enable-or-disable-automatic-backups"></a>自動バックアップの有効化または無効化

バックアップは、デプロイ中に自動的に有効にされます。 保存場所を構成すると、バックアップは、自動バックアップによって、スケジュールに従って頻度の設定に基づいて外部の保存場所にエクスポートされます。 **[要点]** ブレードで、次回のスケジュールされたバックアップ時間をご確認いただけます。

![Azure Stack - オンデマンド バックアップ](media/azure-stack-backup-enable-backup-console-tzl/on-demand-backup.png)

スケジュールされた今後のバックアップを無効にする必要がある場合は、 **[自動バックアップを無効にする]** を選択します。 自動バックアップを無効にしても、バックアップ設定の構成は保たれ、バックアップ スケジュールは保持されます。 このアクションは、将来のバックアップをスキップするようスケジューラに指示します。

![Azure Stack - スケジュールされたバックアップの無効化](media/azure-stack-backup-enable-backup-console-tzl/disable-auto-backup.png)

**[基本]** で、スケジュールされた将来のバックアップが無効にされたことを確認します。

![Azure Stack - バックアップが無効にされたことの確認](media/azure-stack-backup-enable-backup-console-tzl/confirm-disable.png)

**[自動バックアップを有効にする]** を選択して、スケジュールされた時間に今後のバックアップを開始するようにスケジューラに通知します。

![Azure Stack - スケジュールされたバックアップの有効化](media/azure-stack-backup-enable-backup-console-tzl/enable-auto-backup.png)

## <a name="update-backup-settings"></a>バックアップ設定の更新

バックアップ データの暗号化に使用される証明書を更新するには、公開キー部分が含まれる新しい .CER ファイルをアップロードし、[OK] を選択して設定を保存します。

新しいバックアップから、新しい証明書の公開キーの使用が開始されます。 以前の証明書で作成されたすべての既存のバックアップには影響ありません。 クラウドの復旧が必要になったときのため、古い証明書を安全な場所に保管しておいてください。

![Azure Stack - 証明書のサムプリントを表示する](media/azure-stack-backup-enable-backup-console-tzl/encryption-settings-thumbprint.png)

## <a name="next-steps"></a>次のステップ

バックアップが実行されたことを確認するには、[管理者ポータルでのバックアップ完了の確認](../../operator/azure-stack-backup-back-up-azure-stack.md)に関するページをご覧ください。
