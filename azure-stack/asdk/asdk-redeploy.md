---
title: ASDK の再デプロイ
description: Azure Stack Development Kit (ASDK) を再デプロイする方法について説明します。
author: myoungerman
ms.topic: article
ms.date: 02/12/2019
ms.author: v-myoung
ms.reviewer: misainat
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 55a35a509b4fe3fadbe9a58e705cc53cfd18a027
ms.sourcegitcommit: 3e2460d773332622daff09a09398b95ae9fb4188
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/15/2020
ms.locfileid: "90572172"
---
# <a name="redeploy-the-asdk"></a>ASDK の再デプロイ
この記事では、非運用環境で Azure Stack Development Kit (ASDK) を再デプロイする方法について説明します。 ASDK のアップグレードはサポートされていないため、新しいバージョンに移行するには、ASDK を完全に再デプロイする必要があります。 最初から始める場合はいつでも、ASDK を再デプロイすることもできます。

> [!IMPORTANT]
> 新しいバージョンへの ASDK のアップグレードはサポートされていません。 Azure Stack の新しいバージョンを評価するたびに、ASDK ホスト コンピューターに ASDK を再デプロイする必要があります。

## <a name="remove-azure-registration"></a>Azure の登録の削除 
以前に ASDK インストールを Azure に登録したことがある場合は、ASDK を再デプロイする前に、登録リソースを削除する必要があります。 ASDK を再デプロイした場合は、マーケットプレースにある項目を入手できるようにするために ASDK を再登録します。 以前に ASDK を Azure サブスクリプションに登録していない場合は、このセクションをスキップできます。

登録リソースを削除するには、**Remove-AzsRegistration** コマンドレットを使用して Azure Stack の登録を解除します。 次に、**Remove-AzureRMResourceGroup** コマンドレットを使用して、Azure サブスクリプションから Azure Stack リソース グループを削除します。

1. 特権エンドポイントにアクセスできるコンピューターで、管理者として PowerShell コンソールを開きます。 ASDK の場合は、ASDK のホスト コンピューターです。

2. 次の PowerShell コマンドを実行して、ASDK のインストールを登録解除し、Azure サブスクリプションから **azurestack** リソース グループを削除します。

   ```powershell    
   #Import the registration module that was downloaded with the GitHub tools
   Import-Module C:\AzureStack-Tools-master\Registration\RegisterWithAzure.psm1

   # Provide Azure subscription admin credentials
   Add-AzureRmAccount

   # Provide ASDK admin credentials
   $CloudAdminCred = Get-Credential -UserName AZURESTACK\CloudAdmin -Message "Enter the cloud domain credentials to access the privileged endpoint"

   # Unregister Azure Stack
   Remove-AzsRegistration `
      -PrivilegedEndpointCredential $CloudAdminCred `
      -PrivilegedEndpoint AzS-ERCS01
      -RegistrationName $RegistrationName

   # Remove the Azure Stack resource group
   Remove-AzureRmResourceGroup -Name azurestack -Force
   ```

3. スクリプトの実行時に、Azure サブスクリプションとローカル ASDK インストールの両方にサインインするように求めるメッセージが表示されます。
4. コマンドが完了すると、次の例のようなメッセージが表示されます。

    `De-Activating Azure Stack (this may take up to 10 minutes to complete).` `Your environment is now unable to syndicate items and is no longer reporting usage data.`
    `Remove registration resource from Azure...`
    `"Deleting the resource..." on target "/subscriptions/<subscription information>"`
    `********** End Log: Remove-AzsRegistration *********`



これで、Azure Stack の登録が Azure サブスクリプションから正常に解除されました。 azurestack リソース グループも削除する必要があります。 このリソース グループは、ASDK を Azure に初めて登録したときに作成されたものです。

## <a name="deploy-the-asdk"></a>ASDK のデプロイ
Azure Stack を再デプロイするには、次に説明するように、最初から開始する必要があります。 ASDK のインストールに Azure Stack インストーラー (asdk-installer.ps1) スクリプトを使用したかどうかによって、手順が異なります。

### <a name="redeploy-the-asdk-using-the-installer-script"></a>インストーラー スクリプトを使用して ASDK を再デプロイする
1. ASDK コンピューターで、管理者特権で PowerShell コンソールを開き、非システム ドライブにある **AzureStack_Installer** ディレクトリの asdk-installer.ps1 スクリプトに移動します。 スクリプトを実行し、 **[再起動]** をクリックします。

   ![asdk-installer.ps1 スクリプトを実行する](media/asdk-redeploy/1.png)

2. ベース オペレーティング システム (**Azure Stack** でなく) を選択し、 **[次へ]** をクリックします。

   ![ホスト オペレーティング システムで再起動する](media/asdk-redeploy/2.png)

3. ASDK ホストがベース オペレーティング システムで再起動したら、ローカル管理者としてログインします。以前のデプロイの一部として使用されていた **C:\CloudBuilder.vhdx** ファイルを探して削除します。

4. 最初に [ASDK をデプロイ](asdk-install.md)したときと同じ手順を繰り返します。

### <a name="redeploy-the-asdk-without-using-the-installer"></a>インストーラーを使用せずに ASDK を再デプロイする
ASDK をインストールするために asdk-installer.ps1 スクリプトを使用しなかった場合は、ASDK を再デプロイする前に、ASDK ホスト コンピューターを手動で再構成する必要があります。

1. ASDK コンピューターで **msconfig.exe** を実行して、システム構成ユーティリティを起動します。 **[ブート]** タブでホスト コンピューターのオペレーティング システム (Azure Stack ではない) を選択し、 **[既定値に設定する]** をクリックし、 **[OK]** をクリックします。 確認を求めるメッセージが表示されたら、 **[再起動]** をクリックします。

      ![ブート構成を設定する](media/asdk-redeploy/4.png)

2. ASDK ホストがベース オペレーティング システムで再起動したら、ASDK ホスト コンピューターにローカル管理者としてログインします。 以前のデプロイの一部として使用されていた **C:\CloudBuilder.vhdx** ファイルを探して削除します。

3. 最初に [PowerShell を使用して ASDK をデプロイ](asdk-deploy-powershell.md)したときと同じ手順を繰り返します。


## <a name="next-steps"></a>次のステップ
[ASDK デプロイ後のタスク](asdk-post-deploy.md)




