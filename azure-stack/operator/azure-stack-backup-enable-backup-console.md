---
title: 管理者ポータルで Azure Stack Hub のバックアップを有効にする
description: エラーが発生した場合に Azure Stack Hub を復元できるように、管理者ポータルで Infrastructure Backup サービスを有効にする方法について説明します。
author: PatAltimore
ms.topic: article
ms.date: 08/21/2019
ms.author: patricka
ms.reviewer: hectorl
ms.lastreviewed: 08/21/2019
ms.openlocfilehash: 4ec0aebf0fcf46973a4f371d659aece8e51eb2c7
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97871856"
---
# <a name="enable-backup-for-azure-stack-hub-from-the-administrator-portal"></a>管理者ポータルで Azure Stack Hub のバックアップを有効にする

Azure Stack Hub でインフラストラクチャのバックアップを生成できるように、管理者ポータルで Infrastructure Backup サービスを有効にすることができます。 ハードウェア パートナーは、[壊滅的な障害](./azure-stack-backup-recover-data.md)が発生した場合、これらのバックアップとクラウドの復旧を使って環境を復元できます。 クラウドを復旧する目的は、復旧の完了後に、作業者とユーザーがポータルに再度ログインできるということを確認することです。 ユーザーには、次のようなサブスクリプションが復元されます。

- ロール ベースのアクセス許可とロール
- 元のプランとオファー
- 以前に定義したコンピューティング、ストレージ、ネットワーク クォータ
- Key Vault のシークレット

ただし、Infrastructure Backup サービスでは、IaaS VM、ネットワーク構成、ストレージ リソース (ストレージ アカウント、BLOB、テーブルなど) はバックアップされません。 クラウドの回復後にログインしたユーザーには、以前に存在していたリソースは一切表示されません。 サービスとしてのプラットフォーム (PaaS) のリソースとデータも、サービスではバックアップされません。

管理者とユーザーは、IaaS および PaaS のリソースのバックアップと復元を、インフラストラクチャ バックアップ プロセスとは別に行う責任があります。 IaaS と PaaS のリソースのバックアップについては、次のリンクをご覧ください。

- [Azure Stack Hub にデプロイされた VM の保護](../user/azure-stack-manage-vm-protect.md)
- [Azure でのアプリのバックアップ](/azure/app-service/manage-backup)
- [Azure Virtual Machines 上の SQL Server とは何か (Windows)](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-iaas-overview)


## <a name="enable-or-reconfigure-backup"></a>バックアップの有効化または再構成

1. [Azure Stack Hub 管理者ポータル](azure-stack-manage-portals.md)を開きます。
2. **[すべてのサービス]** を選択し、 **[管理]** カテゴリで **[インフラストラクチャ バックアップ]** を選びます。 **[Infrastructure backup]\(インフラストラクチャ バックアップ\)** ブレードで **[構成]** を選択します。
3. **バックアップ ストレージの場所** のパスを入力します。 別のデバイスでホストされるファイル共有へのパスの場合、汎用名前付け規則 (UNC) の文字列を使用します。 UNC 文字列は、共有ファイルやデバイスといった、リソースの場所を指定します。 サービスの場合は、IP アドレスを使用できます。 障害発生後にバックアップ データを確実に利用できるようにするためには、保存デバイスは別の場所に配置する必要があります。

    > [!Note]  
    > 使用する環境で Azure Stack Hub インフラストラクチャ ネットワークからエンタープライズ環境への名前解決がサポートされている場合は、IP ではなく、完全修飾ドメイン名 (FQDN) を使用できます。

4. ファイルを読み書きするための十分なアクセス権を持つドメインとユーザー名を使用して **ユーザー名** を入力します。 たとえば、「 `Contoso\backupshareuser` 」のように入力します。
5. ユーザーの **パスワード** を入力します。
6. **[パスワードの確認]** にもう一度パスワードを入力します。
7. **頻度 (時間単位)** はバックアップの作成頻度を決定します。 既定値は 12 です。 Scheduler では、最大値 12 から最小値 4 までをサポートします。 
8. **保有期間 (日単位)** は、バックアップが外部の保存場所に保持される日数を決定します。 既定値は 7 です。 Scheduler では、最大値 14 から最小値 2 までをサポートします。 保有期間より前のバックアップは、外部の保存場所から自動的に削除されます。

    > [!Note]  
    > 保有期間より前のバックアップをアーカイブする場合は、スケジューラでバックアップを削除する前に必ずファイルをバックアップしてください。 バックアップの保有期間を短縮すると (たとえば、7 日を 5 日にする)、スケジューラは新しい保有期間より前のすべてのバックアップを削除します。 この値を更新する前に、バックアップが削除されても問題がないことを確認してください。

9. [暗号化設定] では、[.Cer 証明書ファイル] ボックスで証明書を指定します。 バックアップ ファイルは証明書内のこの公開キーを使用して暗号化されます。 バックアップの設定を構成するときは、公開キー部分のみを含む証明書を指定します。 この証明書を初めて設定するとき、または後で証明書を交換するときは、証明書のサムプリントのみを表示できます。 アップロードした証明書ファイルをダウンロードまたは表示することはできません。 証明書ファイルを作成するには、次の PowerShell コマンドを実行して公開キーと秘密キーを含む自己署名証明書を作成した後、公開キー部分のみを含む証明書をエクスポートします。 管理ポータルからアクセスできる任意の場所に証明書を保存できます。

    ```powershell

        $cert = New-SelfSignedCertificate `
            -DnsName "www.contoso.com" `
            -CertStoreLocation "cert:\LocalMachine\My"

        New-Item -Path "C:\" -Name "Certs" -ItemType "Directory" 
        Export-Certificate `
            -Cert $cert `
            -FilePath c:\certs\AzSIBCCert.cer 
    ```

   > [!Note]
   > **1901 以降**:Azure Stack Hub では、インフラストラクチャ バックアップ データを暗号化するための証明書が受け入れられます。 公開キーと秘密キーを含む証明書を安全な場所に保管してください。 セキュリティ上の理由から、公開キーと秘密キーを含む証明書を使用して、バックアップ設定を構成することはお勧めできません。 この証明書のライフサイクルを管理する方法について詳しくは、「[インフラストラクチャ バックアップ サービスのベスト プラクティス](azure-stack-backup-best-practices.md)」をご覧ください。
   > 
   > **1811 以前**:Azure Stack Hub では、インフラストラクチャのバックアップ データを暗号化するための対称キーが受け入れられます。 [キーの作成には、New-AzsEncryptionKey64 コマンドレットを使用します](/powershell/module/azs.backup.admin/new-azsencryptionkeybase64)。 1811 から 1901 にアップグレードした後は、バックアップ設定に暗号化キーが保持されます。 証明書を使用するように、バックアップ設定を更新することをお勧めします。 暗号化キーのサポートは現在、非推奨となっています。 証明書を使用するような設定への更新が必要になるまでには、リリースが少なくともあと 3 回あります。

10. **[OK]** を選択して、バックアップ コントローラーの設定を保存します。

![Azure Stack Hub - バックアップ コントローラーの設定](media/azure-stack-backup/backup-controller-settings-certificate.png)


## <a name="start-backup"></a>バックアップの開始
バックアップを開始するには、 **[今すぐバックアップ]** をクリックして、オンデマンド バックアップを開始します。 オンデマンド バックアップでは、次回のスケジュールされたバックアップの時間は変更されません。 タスクの完了後、 **[基本]** で設定を確認できます。

![オンデマンド バックアップを開始する方法を示すスクリーンショット。](media/azure-stack-backup/scheduled-backup.png)

また、Azure Stack Hub 管理コンピューターで PowerShell コマンドレット **Start-AzsBackup** を実行することもできます。 詳細については、「[Azure Stack Hub のバックアップ](azure-stack-backup-back-up-azure-stack.md)」を参照してください。

## <a name="enable-or-disable-automatic-backups"></a>自動バックアップの有効化または無効化
バックアップを有効にすると、バックアップが自動的にスケジュールされます。 **[基本]** で次回のスケジュールのバックアップ時間を確認できます。 

![Azure Stack Hub - オンデマンド バックアップ](media/azure-stack-backup/on-demand-backup.png)

スケジュールされた将来のバックアップを無効にする必要がある場合は、 **[自動バックアップを無効にする]** をクリックします。 自動バックアップを無効にしても、バックアップ設定の構成は保たれ、バックアップ スケジュールは保持されます。 このアクションは、単に将来のバックアップをスキップするようスケジューラに指示するに過ぎません。

![Azure Stack Hub - スケジュールされたバックアップの無効化](media/azure-stack-backup/disable-auto-backup.png)

**[基本]** で、スケジュールされた将来のバックアップが無効にされたことを確認します。

![Azure Stack Hub - バックアップが無効にされたことの確認](media/azure-stack-backup/confirm-disable.png)

**[自動バックアップを有効にする]** をクリックして、スケジュールされた時刻に将来のバックアップを開始するようにスケジューラに通知します。 

![Azure Stack Hub - スケジュールされたバックアップの有効化](media/azure-stack-backup/enable-auto-backup.png)


> [!Note]  
> 1807 に更新する前にインフラストラクチャのバックアップを構成した場合、自動バックアップは無効になります。 このように、Azure Stack Hub によって開始されたバックアップは、外部タスク スケジュール エンジンによって開始されたバックアップとは競合しません。 任意の外部タスク スケジューラを無効にしたら、 **[自動バックアップを有効にする]** をクリックします。

## <a name="update-backup-settings"></a>バックアップ設定の更新
1901 の時点で、暗号化キーのサポートは非推奨になっています。 1901 で初めてバックアップを構成する場合は、証明書を使用する必要があります。 Azure Stack Hub では、1901 に更新する前にキーが構成されていた場合にのみ、暗号化キーがサポートされます。 下位互換性モードは引き続き 3 つのリリースが対象です。 その後は、暗号化キーはサポートされなくなります。

### <a name="default-mode"></a>既定モード
暗号化の設定では、1901 をインストールするか 1901 に更新した後で初めてインフラストラクチャのバックアップを構成する場合は、証明書を使用してバックアップを構成する必要があります。 暗号化キーの使用は、サポートされなくなっています。

バックアップ データの暗号化に使用される証明書を更新するには、公開キー部分が含まれる新しい .CER ファイルをアップロードし、[OK] を選択して設定を保存します。

新しいバックアップから、新しい証明書の公開キーの使用が開始されます。 以前の証明書で作成された既存のバックアップには、まったく影響はありません。 クラウドの復旧が必要になったときのため、古い証明書を安全な場所に保管しておいてください。

![Azure Stack Hub - 証明書のサムプリントを表示する](media/azure-stack-backup/encryption-settings-thumbprint.png)

### <a name="backwards-compatibility-mode"></a>下位互換性モード
1901 に更新する前にバックアップを構成した場合、設定は動作の変更なしで引き継がれます。 この場合、暗号化キーは下位互換性のためにサポートされます。 暗号化キーを更新するか、または証明書の使用に切り替えることができます。 少なくともあと 3 回のリリースの間は、引き続き暗号化キーを更新できます。 この期間を使用して、証明書に移行してください。 新しい暗号化キーを作成するには、[New-AzsEncryptionKeyBase64](/powershell/module/azs.backup.admin/new-azsencryptionkeybase64) を使用します。

![Azure Stack Hub - 下位互換性モードで暗号化キーを使用する](media/azure-stack-backup/encryption-settings-backcompat-encryption-key.png)

> [!Note]  
> 暗号化キーから証明書への更新は、一方向の操作です。 この変更を行った後、暗号化キーに戻すことはできません。 既存のすべてのバックアップは、前の暗号化キーで暗号化されたまま残ります。

![Azure Stack Hub - 下位互換性モードで暗号化証明書を使用する](media/azure-stack-backup/encryption-settings-backcompat-certificate.png)

## <a name="next-steps"></a>次のステップ

バックアップの実行について学習します。 「[Azure Stack Hub のバックアップ](azure-stack-backup-back-up-azure-stack.md)」を参照してください。

バックアップが実行されたことを確認する方法を学習します。 「[管理者ポータルでバックアップの完了を確認する](azure-stack-backup-back-up-azure-stack.md)」を参照してください。
