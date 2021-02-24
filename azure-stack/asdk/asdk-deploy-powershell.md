---
title: PowerShell を使用してコマンド ラインから ASDK をデプロイする
description: PowerShell を使用してコマンド ラインから ASDK デプロイする方法について説明します。
author: PatAltimore
ms.topic: article
ms.date: 10/14/2020
ms.author: patricka
ms.reviewer: misainat
ms.lastreviewed: 10/14/2020
ms.openlocfilehash: a42149fd878a56b306104e5f200b6b3a8eaf07a3
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97873250"
---
# <a name="deploy-asdk-from-the-command-line-using-powershell"></a>PowerShell を使用してコマンド ラインから ASDK をデプロイする

Azure Stack Development Kit (ASDK) は、Azure Stack の機能やサービスを評価したり、実演したりするためにデプロイできるテスト/開発環境です。 それを稼働させるには、環境ハードウェアを用意し、いくつかのスクリプトを実行する必要があります。 このスクリプトは実行に数時間かかります。 その後、管理者ポータルとユーザー ポータルにサインインし、Azure Stack の使用を開始することができます。

## <a name="prerequisites"></a>前提条件

ASDK ホスト コンピューターを準備します。 ハードウェア、ソフトウェア、およびネットワークを計画します。 ASDK をホストするコンピューターでは、ハードウェア、ソフトウェア、ネットワークの要件を満たす必要があります。 Azure Active Directory (Azure AD) と Active Directory フェデレーション サービス (AD FS) のどちらを使用するかを選択します。 インストール プロセスが滞りなく進行するように、デプロイの開始前にこれらの前提条件に従ってください。

ASDK をデプロイする前に、計画している ASDK ホスト コンピューターのハードウェア、オペレーティング システム、アカウント、およびネットワーク構成が、ASDK をインストールするための最小要件を満たしていることを確認してください。

**[ASDK のデプロイの要件と考慮事項を確認してください](asdk-deploy-considerations.md)** 。


## <a name="download-and-extract-the-deployment-package"></a>デプロイ パッケージをダウンロードし、展開する
ASDK ホスト コンピューターが ASDK の基本的なインストール要件を満たしていることを確認したら、次の手順は ASDK デプロイ パッケージをダウンロードして展開することです。 デプロイ パッケージには、Cloudbuilder.vhdx ファイルが含まれています。このファイルは、起動可能なオペレーティング システムと Azure Stack インストール ファイルを含む仮想ハード ドライブです。

ASDK ホストや別のコンピューターに、デプロイ パッケージをダウンロードできます。 展開されたデプロイ ファイルは 60 GB の空きディスク容量を占めます。そのため、別のコンピューターを利用することで、ASDK ホストのハードウェア要件を緩和できます。

**[Azure Stack Development Kit (ASDK) をダウンロードして展開する](asdk-download.md)**

## <a name="prepare-the-asdk-host-computer"></a>ASDK ホスト コンピューターの準備
ASDK をホスト コンピューターにインストールする前に、環境を準備し、VHD から起動するようにシステムを構成する必要があります。 この手順の後、ASDK ホストは Cloudbuilder.vhdx (起動可能なオペレーティング システムと Azure Stack インストール ファイルを含む仮想ハード ドライブ) から起動します。

PowerShell を使用して、CloudBuilder.vhdx から起動するように ASDK ホスト コンピューターを構成します。 これらのコマンドは、ダウンロードして展開した Azure Stack 仮想ハード ディスク (CloudBuilder.vhdx) から起動するように ASDK ホスト コンピューターを構成します。 以下の手順の完了後、ASDK ホスト コンピューターを再起動します。

CloudBuilder.vhdx から起動するように ASDK ホスト コンピューターを構成するには、次の手順に従います。

  1. 管理者としてコマンド プロンプトを起動します。
  2. `bcdedit /copy {current} /d "Azure Stack"` を実行します。
  3. 返された CLSID 値を、必要な中かっこ (`{}`) を含めてコピー (Ctrl + C) します。 この値は `{CLSID}` と呼ばれ、後の手順で貼り付ける (Ctrl + V または右クリック) 必要があります。
  4. `bcdedit /set {CLSID} device vhd=[C:]\CloudBuilder.vhdx` を実行します。
  5. `bcdedit /set {CLSID} osdevice vhd=[C:]\CloudBuilder.vhdx` を実行します。
  6. `bcdedit /set {CLSID} detecthal on` を実行します。
  7. `bcdedit /default {CLSID}` を実行します。
  8. 起動設定を確認するには、`bcdedit` を実行します。
  9. CloudBuilder.vhdx ファイルが C:\ ドライブのルートに移動されている (`C:\CloudBuilder.vhdx`) ことを確認し、ASDK ホスト コンピューターを再起動します。 ASDK ホスト コンピューターが再起動されるときに、ASDK のデプロイを開始するには、CloudBuilder.vhdx 仮想マシン (VM) のハード ドライブから起動する必要があります。

> [!IMPORTANT]
> 再起動する前に、ASDK ホスト コンピューターへの直接の物理的アクセスまたは KVM アクセスがあることを確認してください。 VM が初めて起動するときに、Windows Server セットアップを完了するように求めるメッセージが表示されます。 ASDK ホスト コンピューターにログインするときに使用したのと同じ管理者資格情報を入力します。

### <a name="prepare-the-asdk-host-using-powershell"></a>PowerShell を使用して ASDK ホストを準備する 
ASDK ホスト コンピューターが CloudBuilder.vhdx イメージから正常に起動したら、ASDK ホスト コンピューターへのログインに使用した同じローカル管理者資格情報を使用してサインインします。 さらに、ホスト コンピューターが VHD から起動したときに Windows Server セットアップの実行の一環として指定した同じ資格情報もあります。

> [!NOTE]
> 必要に応じて、ASDK のインストール "*前*" に [Azure Stack のテレメトリ設定](asdk-telemetry.md#set-telemetry-level-in-the-windows-registry)を構成することもできます。

管理者特権の PowerShell コンソールを開いてこのセクションのコマンドを実行し、ASDK を ASDK ホストにデプロイします。

> [!IMPORTANT]
> ASDK のインストールでは、ネットワーク用に 1 つだけの NIC (ネットワーク インターフェイス カード) がサポートされています。 NIC が複数ある場合は、デプロイ スクリプトを実行する前に、1 つのみが有効になっている (他はすべて無効になっている) ことを確認します。

Azure Stack と、ID プロバイダーとして Azure AD または Windows Server AD FS をデプロイできます。 どちらを使用しても、Azure Stack、リソース プロバイダー、およびその他のアプリは同様に動作します。

> [!TIP]
> セットアップ パラメーター (InstallAzureStackPOC.ps1 のオプションのパラメーターと以下の例を参照) を指定しない場合、必須パラメーターの入力を求めるメッセージが表示されます。

### <a name="deploy-azure-stack-using-azure-ad"></a>Azure AD を使用して Azure Stack をデプロイする 
**Azure AD を ID プロバイダーとして使用** して Azure Stack をデプロイするには、インターネットに直接または透過プロキシ経由で接続する必要があります。 

Azure AD を使用して ASDK をデプロイするには、次の PowerShell コマンドを実行します。

  ```powershell
  cd C:\CloudDeployment\Setup     
  $adminpass = Get-Credential Administrator     
  .\InstallAzureStackPOC.ps1 -AdminPassword $adminpass.Password
  ```

ASDK のインストールを開始して数分後に、Azure AD 資格情報の入力を求めるメッセージが表示されます。 Azure AD テナントのグローバル管理者の資格情報を入力します。

デプロイの後、Azure Active Directory の全体管理者のアクセス許可は必要ありません。 ただし、一部の操作では、全体管理者の資格情報が必要な場合があります。 そのような操作の例には、リソース プロバイダーのインストーラー スクリプトや、アクセス許可を付与する必要のある新機能などがあります。 アカウントの全体管理者のアクセス許可を一時的に復元するか、*既定のプロバイダー サブスクリプション* の所有者である別の全体管理者アカウントを使用します。

### <a name="deploy-azure-stack-using-ad-fs"></a>AD FS を使用して Azure Stack をデプロイする 
**AD FS を ID プロバイダーとして使用** して ASDK をデプロイするには、次の PowerShell コマンドを実行します (-UseADFS パラメーターを追加するだけです)。

  ```powershell
  cd C:\CloudDeployment\Setup     
  $adminpass = Get-Credential Administrator 
  .\InstallAzureStackPOC.ps1 -AdminPassword $adminpass.Password -UseADFS
  ```

AD FS デプロイでは、既定のスタンプ ディレクトリ サービスが ID プロバイダーとして使用されます。 サインインに使用する既定のアカウントは azurestackadmin@azurestack.local で、パスワードは PowerShell のセットアップ コマンドの一部として指定したものに設定されます。

デプロイ処理には数時間かかる場合があります。その間に、システムは自動的に 1 回再起動されます。 デプロイが成功すると、PowerShell コンソールに次のように表示されます:**COMPLETE:Action 'Deployment' (完了: アクション 'デプロイ')** 。 デプロイが失敗した場合は、-rerun パラメーターを使用してスクリプトを再実行してみてください。 または、最初から [ASDK を再デプロイ](asdk-redeploy.md)することもできます。

> [!IMPORTANT]
> ASDK ホストの再起動後にデプロイの進行状況を監視する場合は、AzureStack\AzureStackAdmin としてサインインする必要があります。 ホスト コンピューターが再起動され、azurestack.local ドメインに参加した後、ローカル管理者としてサインインすると、デプロイの進行状況は表示されません。 デプロイは再実行せず、代わりにローカル管理者と同じパスワードを使用して AzureStack\AzureStackAdmin としてサインインし、セットアップが実行中であることを確認します。


#### <a name="azure-ad-deployment-script-examples"></a>Azure AD のデプロイ スクリプトの例
Azure AD のデプロイ全体をスクリプト化することができます。 オプションのパラメーターを含む、コメント付きの例をいくつか示します。

Azure AD ID が **1 つ** の Azure AD ディレクトリのみに関連付けられている場合:

```powershell
cd C:\CloudDeployment\Setup 
$adminpass = Get-Credential Administrator 
$aadcred = Get-Credential "<Azure AD global administrator account name>" 
.\InstallAzureStackPOC.ps1 -AdminPassword $adminpass.Password -InfraAzureDirectoryTenantAdminCredential $aadcred -TimeServer 52.168.138.145 #Example time server IP address.
```

Azure AD ID が **2 つ以上** の Azure AD ディレクトリに関連付けられている場合:
```powershell
cd C:\CloudDeployment\Setup 
$adminpass = Get-Credential Administrator 
$aadcred = Get-Credential "<Azure AD global administrator account name>" #Example: user@AADDirName.onmicrosoft.com 
.\InstallAzureStackPOC.ps1 -AdminPassword $adminpass.Password -InfraAzureDirectoryTenantAdminCredential $aadcred -InfraAzureDirectoryTenantName "<Azure AD directory in the form of domainname.onmicrosoft.com or an Azure AD verified custom domain name>" -TimeServer 52.168.138.145 #Example time server IP address.
```

自分の環境で DHCP を有効にしていない場合は、上のいずれかのオプションに以下の追加パラメーターを含める必要があります (使用例を示しています)。 

```powershell
.\InstallAzureStackPOC.ps1 -AdminPassword $adminpass.Password -InfraAzureDirectoryTenantAdminCredential $aadcred -TimeServer 10.222.112.26
```

### <a name="asdk-installazurestackpocps1-optional-parameters"></a>ASDK InstallAzureStackPOC.ps1 の省略可能なパラメーター

|パラメーター|必須/省略可能|説明|
|-----|-----|-----|
|AdminPassword|必須|ASDK のデプロイの一環として作成されたすべての仮想マシンのローカル管理者アカウントと他のすべてのユーザー アカウントを設定します。 このパスワードは、ホスト上の現在のローカル管理者パスワードと一致する必要があります。|
|InfraAzureDirectoryTenantName|必須|テナント ディレクトリを設定します。 このパラメーターを使用して、Azure AD アカウントに複数のディレクトリを管理するアクセス許可がある特定のディレクトリを指定します。 Azure AD テナントのフル ネーム (.onmicrosoft.com 形式) または Azure AD 確認済みカスタム ドメイン名。|
|TimeServer|必須|このパラメーターを使用して、特定のタイム サーバーを指定します。 このパラメーターは、有効なタイム サーバーの IP アドレスとして指定する必要があります。 サーバー名はサポートされていません。|
|InfraAzureDirectoryTenantAdminCredential|省略可能|Azure Active Directory のユーザー名とパスワードを設定します。 これらの Azure 資格情報は、組織 ID である必要があります。|
|InfraAzureEnvironment|省略可能|この Azure Stack デプロイを登録する Azure 環境を選択します。 オプションには、グローバル Azure、Azure China、Azure US Government などがあります。|
|DNSForwarder|省略可能|DNS サーバーが Azure Stack のデプロイの一部として作成されます。 ソリューション内のコンピューターにスタンプ外の名前解決を許可するには、既存のインフラストラクチャの DNS サーバーを提供します。 スタンプ内の DNS サーバーが、このサーバーに不明な名前解決の要求を送信します。|
|Rerun|省略可能|このフラグを使用して、デプロイを再実行します。 以前のすべての入力が使用されます。 以前に指定したデータの再入力は、サポートされていません。いくつかの一意の値が生成され、デプロイに使用されるためです。|


## <a name="perform-post-deployment-configurations"></a>デプロイ後の構成を実行する
ASDK をインストールした後、推奨されるインストール後の確認と構成変更がいくつかあります。 test-AzureStack コマンドレットを使用してインストールが正常に行われたことを検証してから、Azure Stack PowerShell および GitHub ツールをインストールします。

評価期間が終了する前に ASDK ホストのパスワードが期限切れにならないように、パスワードの有効期限のポリシーをリセットすることをお勧めします。

> [!NOTE]
> 必要に応じて、ASDK のインストール "*後*" に [Azure Stack のテレメトリ設定](asdk-telemetry.md#enable-or-disable-telemetry-after-deployment)を構成することもできます。

**[ASDK デプロイ後のタスク](asdk-post-deploy.md)**

## <a name="register-with-azure"></a>Azure に登録する
Azure Stack に [Azure Marketplace 項目をダウンロード](../operator/azure-stack-create-and-publish-marketplace-item.md)できるように、Azure Stack を Azure に登録する必要があります。

**[Azure Stack を Azure に登録する](asdk-register.md)**

## <a name="next-steps"></a>次のステップ
お疲れさまでした。 これらの手順を完了すると、管理ポータル (`https://adminportal.local.azurestack.external`) とユーザー ポータル (`https://portal.local.azurestack.external`) の両方で ASDK 環境を利用できるようになります。 

[ASDK インストール後の構成タスク](asdk-post-deploy.md)

