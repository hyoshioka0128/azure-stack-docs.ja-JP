---
title: ローカル エージェントをデプロイする
titleSuffix: Azure Stack Hub
description: Azure Stack Hub のサービスとしての検証に使用するローカル エージェントをデプロイする方法について説明します。
author: mattbriggs
ms.topic: quickstart
ms.date: 08/24/2020
ms.author: mabrigg
ms.reviewer: johnhas
ms.lastreviewed: 11/11/2019
ROBOTS: NOINDEX
ms.openlocfilehash: 7661177ec292d7c0b678f05d95c33d90f8e57c2c
ms.sourcegitcommit: 4922a14fdbc8a3b67df065336e8a21a42f224867
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/24/2020
ms.locfileid: "88764785"
---
# <a name="deploy-the-local-agent"></a>ローカル エージェントをデプロイする

[!INCLUDE [Azure_Stack_Partner](./includes/azure-stack-partner-appliesto.md)]

サービスとしての検証 (VaaS) のローカル エージェントを使って検証テストを実行する方法について説明します。 検証テストを実行する前にローカル エージェントをデプロイする必要があります。

> [!Note]  
> ローカル エージェントを実行するマシンのインターネットへの送信アクセスが失われていないことを確認してください。 テナントに代わって VaaS を使用する権限が付与されたユーザーのみがこのマシンにアクセスできるようにする必要があります。

ローカル エージェントをデプロイするには:

1. ローカル エージェントをダウンロードしてインストールします。
2. テストを開始する前にサニティ チェックを実行します。
3. ローカル エージェントを実行します。

## <a name="download-and-start-the-local-agent"></a>ローカル エージェントのダウンロードと起動

データセンター内の前提条件を満たしたマシンのうち、Azure Stack Hub のすべてのエンドポイントにアクセスできるマシンにエージェントをダウンロードします。 このマシンは、Azure Stack Hub システムに属していないこと、また Azure Stack Hub クラウドでホストされていないことが必要です。

### <a name="machine-prerequisites"></a>マシンの前提条件

対象のマシンが次の条件を満たしていることを確認します。

- すべての Azure Stack Hub エンドポイントへのアクセス。
- .NET 4.6 と PowerShell 5.0 がインストール済み。
- 8 GB 以上の RAM。
- 8 コア以上のプロセッサ。
- 200 GB 以上のディスク領域。
- インターネットへの安定したネットワーク接続。

### <a name="download-and-install-the-local-agent"></a>ローカル エージェントのダウンロードとインストール

1. テストの実行に使用するマシンから、管理者特権でのコマンド プロンプトで Windows PowerShell を開きます。
2. 次のコマンドを実行して、ローカル エージェントの依存関係をダウンロードしてインストールし、Azure Stack Hub 環境にパブリック イメージ リポジトリ (PIR) イメージ (OS VHD) をコピーします。

    ```powershell
    # Review and update the following five parameters
    $RootFolder = "c:\VaaS"
    $CloudAdmindUserName = "<Cloud admin user name>"
    $CloudAdminPassword = "<Cloud admin password>"
    $AadServiceAdminUserName = "<AAD service admin user name>"
    $AadServiceAdminPassword = "<AAD service admin password>"

    if (-not(Test-Path($RootFolder))) {
        mkdir $RootFolder
    }
    Set-Location $RootFolder
    Invoke-WebRequest -Uri "https://storage.azurestackvalidation.com/packages/Microsoft.VaaSOnPrem.TaskEngineHost.latest.nupkg" -outfile "$rootFolder\OnPremAgent.zip"
    Expand-Archive -Path "$rootFolder\OnPremAgent.zip" -DestinationPath "$rootFolder\VaaSOnPremAgent" -Force
    Set-Location "$rootFolder\VaaSOnPremAgent\lib\net46"

    $cloudAdminCredential = New-Object System.Management.Automation.PSCredential($cloudAdmindUserName, (ConvertTo-SecureString $cloudAdminPassword -AsPlainText -Force))
    $getStampInfoUri = "https://ASAppGateway:4443/ServiceTypeId/4dde37cc-6ee0-4d75-9444-7061e156507f/CloudDefinition/GetStampInformation" 
    $stampInfo = Invoke-RestMethod -Method Get -Uri $getStampInfoUri -Credential $cloudAdminCredential -ErrorAction Stop
    $serviceAdminCreds = New-Object System.Management.Automation.PSCredential $aadServiceAdminUserName, (ConvertTo-SecureString $aadServiceAdminPassword -AsPlainText -Force)
    Import-Module .\VaaSPreReqs.psm1 -Force
    Install-VaaSPrerequisites -AadTenantId $stampInfo.AADTenantID `
                            -ServiceAdminCreds $serviceAdminCreds `
                            -ArmEndpoint $stampInfo.AdminExternalEndpoints.AdminResourceManager `
                            -Region $stampInfo.RegionName
    ```

> [!Note]  
> `Install-VaaSPrerequisites` コマンドレットにより、大きな VM イメージ ファイルがダウンロードされます。 ネットワーク速度の低下が発生している場合は、ローカル ファイル サーバーにファイルをダウンロードし、テスト環境に VM イメージを手動で追加することができます。 詳細については、「[Handle slow network connectivity (低速なネットワーク接続への対応)](azure-stack-vaas-troubleshoot.md#handle-slow-network-connectivity)」を参照してください。

**パラメーター**

| パラメーター | 説明 |
| --- | --- |
| `AadServiceAdminUser` | Azure AD テナントの全体管理者ユーザー (例: vaasadmin@contoso.onmicrosoft.com)。 |
| `AadServiceAdminPassword` | 全体管理者ユーザーのパスワード。 |
| `CloudAdminUserName` | 特権エンドポイント内で許可されたコマンドにアクセスし、実行できるクラウド管理者ユーザー。 次に例を示します。AzusreStack\CloudAdmin。 詳細については、[VaaS の一般的なワークフロー パラメーター](azure-stack-vaas-parameters.md)に関する記事を参照してください。 |
| `CloudAdminPassword` | クラウド管理者アカウントのパスワード。|

![ローカル エージェントの前提条件をダウンロードする](media/installing-prereqs.png)

## <a name="perform-sanity-checks-before-starting-the-tests"></a>テストを開始する前のサニティ チェックを実行する

テストではリモート操作が実行されます。 テストを実行するマシンは、Azure Stack Hub エンドポイントにアクセスできる必要があります。そうでないと、テストは機能しません。 VaaS のローカル エージェントを使用する場合は、エージェントを実行するマシンを使用します。 次のチェックを実行すると、マシンが Azure Stack Hub エンドポイントにアクセスできることを確認できます。

1. ベース URI に到達できることを確認します。 CMD プロンプトまたは bash シェルを開きます。`<EXTERNALFQDN>` をお使いの環境の外部完全修飾ドメイン名 (FQDN) に置き換えて、次のコマンドを実行します。

    ```bash
    nslookup adminmanagement.<EXTERNALFQDN>
    ```

2. MAS ポータルに到達できることを確認するために、ブラウザーを開き、`https://adminportal.<EXTERNALFQDN>` に移動します。

3. テスト成功の作成時に指定した Azure AD サービス管理者の名前とパスワードの値を使用してサインインします。

4. 「[Azure Stack Hub の検証テストを実行する](../operator/azure-stack-diagnostic-test.md)」の説明に従って、**Test-AzureStack** PowerShell コマンドレットを実行してシステムの正常性を確認します。 警告やエラーがあれば、テストを開始する前にすべて解消しておいてください。

## <a name="run-the-local-agent"></a>ローカル エージェントを実行する

1. 管理者特権のプロンプトで Windows PowerShell を開きます。

2. 次のコマンドを実行します。

    ```powershell
   # Review and update the following five parameters
    $RootFolder = "c:\VAAS"
    $CloudAdmindUserName = "<Cloud admin user name>"
    $CloudAdminPassword = "<Cloud admin password>"
    $VaaSUserId = "<VaaS user ID>"
    $VaaSTenantId = "<VaaS tenant ID>"

    Set-Location "$rootFolder\VaaSOnPremAgent\lib\net46"
    .\Microsoft.VaaSOnPrem.TaskEngineHost.exe -u $VaaSUserId -t $VaaSTenantId -x $CloudAdmindUserName -y $CloudAdminPassword
    ```

      **パラメーター**  

    | パラメーター | 説明 |
    | --- | --- |
    | `CloudAdminUserName` | 特権エンドポイント内で許可されたコマンドにアクセスし、実行できるクラウド管理者ユーザー。 次に例を示します。AzusreStack\CloudAdmin。 詳細については、[VaaS の一般的なワークフロー パラメーター](azure-stack-vaas-parameters.md)に関する記事を参照してください。 |
    | `CloudAdminPassword` | クラウド管理者アカウントのパスワード。|
    | `VaaSUserId` | Azure Stack Hub 検証ポータルにサインインするために使用するユーザー ID。 次に例を示します。UserName\@Contoso.com)。 |
    | `VaaSTenantId` | サービスとしての検証に登録された Azure アカウントの Azure AD テナント ID。 |

    > [!Note]  
    > エージェントを実行するときは、タスク エンジン ホストの実行可能ファイル (**Microsoft.VaaSOnPrem.TaskEngineHost.exe**) が置かれている場所を現在の作業ディレクトリとしてください。

エラーが一切報告されなければ、ローカル エージェントは正常に実行されたことになります。 次の例に示すテキストがコンソール ウィンドウに表示されます。

`Heartbeat was sent successfully.`

![起動されたエージェント](media/started-agent.png)

エージェントは、その名前で一意に識別されます。 既定では、その起動元となったマシンの FQDN が使用されます。 うっかり選択してしまうことのないようウィンドウは最小化してください。フォーカスが変わると他のアクションがすべて一時停止されます。

## <a name="next-steps"></a>次のステップ

- [サービスとしての検証のトラブルシューティング](azure-stack-vaas-troubleshoot.md)
- [サービスとしての検証の主要概念](azure-stack-vaas-key-concepts.md)
- [クイック スタート: Azure Stack Hub 検証ポータルを使用して初めてのテストをスケジュールする](azure-stack-vaas-schedule-test-pass.md)