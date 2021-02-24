---
title: Azure Backup を使用して Windows Server をバックアップする
description: この記事では、Windows Admin Center から Azure Backup を使用して Windows Server をバックアップする方法のガイダンスを提供します。
author: thomasmaurer
ms.author: thmaure
ms.topic: how-to
ms.date: 07/21/2020
ms.openlocfilehash: a4668e4d78a2ea3fb9f94913863d3471313034e5
ms.sourcegitcommit: a15a0f955bac922cebb7bf90a72384fd84ddfe56
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/22/2020
ms.locfileid: "86947505"
---
# <a name="use-azure-backup-to-back-up-windows-servers"></a>Azure Backup を使用して Windows Server をバックアップする

>適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019

Windows Admin Center では、Windows Server の Microsoft Azure へのバックアップが簡素化されています。 このプロセスを使用すると、偶発的または悪意のある削除、破損、さらにはランサムウェアからも保護されます。 セットアップを自動化するには、Windows Admin Center を Azure に接続して Azure Backup サービスを使用します。

この記事では、Azure Backup を構成し、Windows Admin Center からサーバー ボリュームと Windows システム状態のバックアップ ポリシーを作成する方法について説明します。 このガイダンスは、Azure Stack HCI オペレーティング システムの仮想マシン (VM) で実行されているワークロードを Azure にバックアップすることを目的としています。

Windows Admin Center と Azure の統合の詳細については、「[Windows Server を Azure ハイブリッド サービスに接続する](/windows-server/manage/windows-admin-center/azure/)」を参照してください。

## <a name="how-azure-backup-works-with-windows-admin-center"></a>Windows Admin Center での Azure Backup のしくみ
**Azure Backup** は、Azure 内のデータのバックアップ、保護、復元に使用できるサービスです。 既存のオンプレミスまたはオフプレミスのバックアップ ソリューションを、信頼性の高い、セキュリティで保護された、コスト競争力のあるクラウド ベースのソリューションに置き換えます。

詳細については、「[Azure Backup サービスとは](/azure/backup/backup-overview)」を参照してください。

Azure Backup には複数のコンポーネントが用意されており、これを適切なコンピューター、サーバー、またはクラウドにダウンロードしてデプロイします。 デプロイするコンポーネント (エージェント) は、何を保護するかによって決まります。 すべての Azure Backup コンポーネントは、保護対象がオンプレミス データか Azure のデータかに関係なく、Azure の Recovery Services コンテナーにデータをバックアップできます。

Azure Backup と Windows Admin Center との統合は、オンプレミスの Windows Server または仮想サーバーのいずれかのボリュームと Windows システム状態のバックアップに最適です。 包括的なプロセスで、ファイル サーバー、ドメイン コントローラー、および IIS Web サーバーをバックアップします。

Windows Admin Center で Azure Backup にアクセスするには、**バックアップ** ツールを使用します。 **バックアップ** ツールのセットアップ、管理、および監視機能を使用すると、サーバーのバックアップをすぐに開始し、バックアップと復元の操作を実行し、Windows Server の全体的なバックアップ正常性を監視できます。

## <a name="prerequisites"></a>前提条件
Azure Backup を使用するには、次の前提条件があります。
- 少なくとも 1 つのアクティブなサブスクリプションを持つ Azure アカウント
- ターゲット Windows Server から Azure へのインターネット アクセス
- Windows Admin Center のゲートウェイから Azure への接続

    詳細については、[Azure 統合の構成](/windows-server/manage/windows-admin-center/azure/azure-integration)に関する記事をご覧ください。

## <a name="getting-started-with-azure-backup"></a>Azure Backup の概要
Windows Admin Center で **バックアップ** ツールを初めて選択して、Azure へのサーバー接続を確立すると、 **[Azure Backup へようこそ]** のページが表示されます。 Azure Backup セットアップ ウィザードを起動するには、 **[Azure Backup のセットアップ]** を選択します。 次のセクションでは、ウィザードの各手順について詳しく説明します。

Azure Backup でサーバー接続が既に構成されている場合は、**バックアップ** ツールを選択すると、**バックアップ ダッシュボード**が開きます。 ダッシュボードから実行できる操作とタスクの詳細については、「[管理と監視](#management-and-monitoring)」セクションを参照してください。

### <a name="step-1-log-on-to-the-microsoft-azure-portal"></a>手順 1:Microsoft Azure portal にログオンする
Azure アカウントにサインインします。

> [!NOTE]
> Windows Admin Center ゲートウェイを Azure に既に接続している場合は、ポータルに自動的にログオンしているはずです。 **[サインアウト]** を選択し、別のユーザーとしてサインインします。

### <a name="step-2-set-up-azure-backup"></a>手順 2:Azure Backup をセットアップする
Azure Backup に対して次の設定を選択します。
- **サブスクリプション ID:** Windows Server を Azure にバックアップするために使用する Azure サブスクリプション。 Azure リソース グループや Recovery Services コンテナーなどのすべての Azure 資産が作成され、選択したサブスクリプションに関連付けられます。
- **コンテナー:** Recovery Services コンテナーは、サーバーのバックアップが格納される場所です。 既存のコンテナーを使用することも、Windows Admin Center で新たに作成することもできます。  
- **[リソース グループ]:** Azure リソース グループは、リソースのコレクションのコンテナーです。 Recovery Services コンテナーが作成されるか、指定されたリソース グループに含まれています。 既存のリソース グループを使用することも、Windows Admin Center で新しいリソース グループを作成することもできます。
- **[場所]:** Recovery Services コンテナーが作成される Azure リージョン。 バックアップする Windows Server に最も近い Azure リージョンを選択することをお勧めします。

### <a name="step-3-select-backup-items-and-schedule"></a>手順 3:バックアップ項目とスケジュールを選択する
1. サーバーからバックアップする項目を選択します。 Windows Admin Center では、ユーザーはボリュームと Windows システム状態の組み合わせを選択でき、バックアップ対象として選択したデータの推定サイズが示されます。

    > [!NOTE]
    > 最初のバックアップは、選択したすべてのデータの完全バックアップです。 その後のバックアップは増分バックアップで、前のバックアップ以降に発生したデータに対する変更のみが転送されます。

1. Windows システムの状態とボリュームに対して、複数の事前設定された**バックアップ スケジュール**から選択します。

### <a name="step-4-enter-an-encryption-passphrase"></a>手順 4:暗号化パスフレーズを入力する
1. 16 文字以上の**暗号化パスフレーズ**を入力します。 ユーザーが作成して管理する暗号化パスフレーズを使用してバックアップ データが保護されます。 Azure Backup からデータを回復するには、暗号化パスフレーズが必要です。

    > [!NOTE]
    > パスフレーズは、別のサーバーや [Azure Key Vault](/azure/key-vault/quick-create-portal) などの安全なオフサイトの場所に格納します。 Microsoft はパスフレーズを保存しません。パスフレーズを紛失したり忘れたりした場合、パスフレーズを取得またはリセットすることはできません。

1. ウィザード ページですべての設定を確認し、 **[適用]** を選択します。

Windows Admin Center でその後、次の操作が実行されます。
1. Azure リソース グループがまだ存在していない場合は作成します。
1. 指定されたとおりに Azure Recovery Services コンテナーを作成します。
1. Microsoft Azure Recovery Services エージェントをコンテナーにインストールして登録します。
1. 選択された、Windows Server に関連付けられているオプションに従って、バックアップと保持のスケジュールを作成します。

## <a name="management-and-monitoring"></a>管理と監視
Azure Backup が正常にセットアップされると、既存のサーバー接続に対して**バックアップ** ツールを開いたときに **バックアップ ダッシュボード**が表示されます。 このダッシュボードからは、次の作業を実行できます。
- **Azure のコンテナーへのアクセス:** **[概要]** タブで、 **[Recovery Services コンテナー]** を選択してコンテナーにアクセスし、多数の管理操作を実行できます。 詳細については、「[Recovery Services コンテナーの監視と管理](/azure/backup/backup-azure-manage-windows-server)」を参照してください。
- **アドホック バックアップの実行:** **[今すぐバックアップ]** を選択して、アドホック バックアップを作成します。 
- **ジョブを監視しアラート通知を構成する:** **[ジョブ]** タブに移動して、進行中または過去のジョブを監視します。また、失敗したジョブやその他のバックアップ アラートに関する電子メールを受信するには、[アラート通知を構成](/azure/backup/backup-azure-manage-windows-server#configuring-notifications-for-alerts)します。
- **復旧ポイントの表示とデータの回復:** 復旧ポイントを表示するには **[復旧ポイント]** タブを選択します。 **[データの回復]** を選択すると、Azure からデータを復旧する手順が表示されます。

## <a name="next-steps"></a>次のステップ
詳細については、次のトピックも参照してください。
- [Azure Site Recovery を使用して Azure Stack HCI VM を保護する](./azure-site-recovery.md)
