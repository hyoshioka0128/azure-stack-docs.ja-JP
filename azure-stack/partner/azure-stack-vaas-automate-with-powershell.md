---
title: PowerShell を使用した Azure Stack の検証の自動化
titleSuffix: Azure Stack Hub
description: PowerShell を使って Azure Stack の検証を自動化する方法について説明します。
author: mattbriggs
ms.topic: tutorial
ms.date: 08/24/2020
ms.author: mabrigg
ms.reviewer: johnhas
ms.lastreviewed: 11/26/2019
ROBOTS: NOINDEX
ms.openlocfilehash: 4d6cb5cdfe6cd294e54ae9f2fa5fc78e6d71548a
ms.sourcegitcommit: 4922a14fdbc8a3b67df065336e8a21a42f224867
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/24/2020
ms.locfileid: "88764734"
---
# <a name="automate-azure-stack-hub-validation-with-powershell"></a>PowerShell を使用した Azure Stack Hub の検証の自動化

サービスとしての検証 (VaaS) には、**RunVaaSAutomation.ps1** スクリプトを使ってテストの開始を自動化する機能があります。

このスクリプトを使用して次のことをおこなえます。

> [!div class="checklist"]
> * 必須コンポーネントをインストールする。
> * ローカル エージェントをインストールして起動する。
> * テスト パス、ソリューション検証、およびパッケージ検証ワークフローで VaaS テストを起動する。
> * テスト結果を報告する。

次のリンクには、VaaS ポータルを使用してテストを実行する方法に関する情報が含まれています。 スクリプトを使用する前に、必須パラメーターとその値について学習してください。

* SolutionValidation ワークフロー:[新しい Azure Stack Hub ソリューションの検証](azure-stack-vaas-validate-solution-new.md)
* PackageValidation ワークフロー:[OEM パッケージの検証](azure-stack-vaas-validate-oem-package.md)
* TestPass ワークフロー:[テストのスケジュール設定](azure-stack-vaas-schedule-test-pass.md)

## <a name="download-the-automation-scripts"></a>自動化スクリプトをダウンロードする

1. 管理者特権の PowerShell プロンプトを開きます。

2. 次のスクリプトを実行して、オートメーション スクリプトをダウンロードします。

```powershell
# Review and update the $RootFolder parameter
$RootFolder = "c:\VaaS"

if (-not(Test-Path($RootFolder))) {
    mkdir $RootFolder
}
Invoke-WebRequest -Uri https://storage.azurestackvalidation.com/packages/Microsoft.VaaS.Scripts.latest.nupkg -OutFile "$RootFolder\RunVaaSAutomation.zip"
Expand-Archive -Path "$RootFolder\RunVaaSAutomation.zip" -DestinationPath "$RootFolder\RunVaaSAutomation" -Force
Set-Location "$RootFolder\RunVaaSAutomation"
```

## <a name="launch-the-solution-validation-workflow"></a>ソリューション検証ワークフローを起動する

VaaS ポータルを使用してソリューション検証ワークフローを実行する方法については、「[OEM パッケージの検証](azure-stack-vaas-validate-oem-package.md)」を参照してください。

適切なパラメーター値を使用して、次のスクリプトを実行します。

```powershell
# Review and update the following parameters
$VaaSAccountUserName = ""
$VaaSAccountPassword = ""
$VaaSAccountTenantId = ""
$ServiceAdminUserName = ""
$ServiceAdminPassword = ""
$TenantAdminUserName  = ""
$TenantAdminPassword  = ""
$CloudAdminUserName   = ""
$CloudAdminPassword   = ""
$SolutionName   = ''
$ProjectName    = ''
$DiagnosticsStorageConnection=''
$VaaSProject_SolutionValidation_Configuration='Min' # enter 'Min' or 'Max'

# No need to modify the following lines
Import-Module .\VaaSAutomation.psm1 -verbose -force
$stampInfo = Get-StampInfo -cloudAdminUserName $CloudAdminUserName -cloudAdminPassword $CloudAdminPassword -retryCount 3 -retryPeriod 30 -outputFolder "."
$AgentName = "$((Get-WmiObject win32_computersystem).DNSHostName).$((Get-WmiObject win32_computersystem).Domain)".ToLower()
$VaaSAccountCredentail = New-Object System.Management.Automation.PSCredential ($VaaSAccountUserName, (ConvertTo-SecureString $VaaSAccountPassword -AsPlainText -Force))

$testParameters = @{}
$projectParameters = @{
    "Configuration" = $VaaSProject_SolutionValidation_Configuration;
}
$scriptParameters = @{
    'VaaSAccountCredentail' = $VaaSAccountCredentail;
    'VaaSAccountTenantId' = $VaaSAccountTenantId;
    'VaaSSolutionName' = $SolutionName;
    'VaaSProjectType' = 'SolutionValidation';
    'VaaSProjectName' = $ProjectName;
    'VaaSProjectParameterHashTable'= $projectParameters;
    'VaaSTestFilter' = 'Test';
    'VaaSTestFilterValue' = '';
    'VaaSTestParameterHashTable'= $testParameters;
    'VaaSOnPremAgentName'= $AgentName;
    'MaxScriptWaitTimeInHours'=48;
    'ServiceAdminUserName' = $ServiceAdminUserName;
    'ServiceAdminPassword' = $ServiceAdminPassword;
    'TenantAdminUserName' = $TenantAdminUserName;
    'TenantAdminPassword' = $TenantAdminPassword;
    'CloudAdminUserName' = $CloudAdminUserName;
    'CloudAdminPassword' = $CloudAdminPassword;
    'DiagnosticsStorageConnection' = $DiagnosticsStorageConnection;
}
& .\RunVaaSAutomation.ps1 @scriptParameters
```

## <a name="launch-package-validation-workflow"></a>パッケージ検証ワークフローを起動する

VaaS ポータルを使用してパッケージ検証ワークフローを実行する方法については、「[OEM パッケージの検証](azure-stack-vaas-validate-oem-package.md)」を参照してください。

適切なパラメーター値を使用して、次のスクリプトを実行します。

```powershell
# Review and update the following parameters
$VaaSAccountUserName = ""
$VaaSAccountPassword = ""
$VaaSAccountTenantId = ""
$ServiceAdminUserName = ""
$ServiceAdminPassword = ""
$TenantAdminUserName  = ""
$TenantAdminPassword  = ""
$CloudAdminUserName   = ""
$CloudAdminPassword   = ""
$SolutionName   = ''
$ProjectName    = ''
$DiagnosticsStorageConnection=''
$VaaSProject_PackageValidation_PackageBlobUri=''
$VaaSProject_PackageValidation_RequireDigitalSignature = "false" # enter 'true' or 'false' string
$VaaSProject_PackageValidation_LocalPathtoAzureStackUpdatePkgs = ""
$VaaSProject_PackageValidation_LocalPathtoOEMUpdatePkgs = ""

# No need to modify the following lines
Import-Module .\VaaSAutomation.psm1 -verbose -force
$stampInfo = Get-StampInfo -cloudAdminUserName $CloudAdminUserName -cloudAdminPassword $CloudAdminPassword -retryCount 3 -retryPeriod 30 -outputFolder "."
$AgentName = "$((Get-WmiObject win32_computersystem).DNSHostName).$((Get-WmiObject win32_computersystem).Domain)".ToLower()
$VaaSAccountCredentail = New-Object System.Management.Automation.PSCredential ($VaaSAccountUserName, (ConvertTo-SecureString $VaaSAccountPassword -AsPlainText -Force))

$testParameters = @{
    "RequireDigitalSignature" = $VaaSProject_PackageValidation_RequireDigitalSignature;
    "LocalPathtoAzureStackUpdatePkgs" = $VaaSProject_PackageValidation_LocalPathtoAzureStackUpdatePkgs;
    "LocalPathtoOEMUpdatePkgs" = $VaaSProject_PackageValidation_LocalPathtoOEMUpdatePkgs;
}

$projectParameters = @{
    "PackageBlobUri" = $VaaSProject_PackageValidation_PackageBlobUri;
}
$scriptParameters = @{
    'VaaSAccountCredentail' = $VaaSAccountCredentail;
    'VaaSAccountTenantId' = $VaaSAccountTenantId;
    'VaaSSolutionName' = $SolutionName;
    'VaaSProjectType' = 'PackageValidation';
    'VaaSProjectName' = $ProjectName;
    'VaaSProjectParameterHashTable' = $projectParameters;
    'VaaSTestFilter' = 'Test';
    'VaaSTestFilterValue' = '';
    'VaaSTestParameterHashTable'= $testParameters;
    'VaaSOnPremAgentName'= $AgentName;
    'MaxScriptWaitTimeInHours'=96;
    'ServiceAdminUserName' = $ServiceAdminUserName;
    'ServiceAdminPassword' = $ServiceAdminPassword;
    'TenantAdminUserName' = $TenantAdminUserName;
    'TenantAdminPassword' = $TenantAdminPassword;
    'CloudAdminUserName' = $CloudAdminUserName;
    'CloudAdminPassword' = $CloudAdminPassword;
    'DiagnosticsStorageConnection' = $DiagnosticsStorageConnection;
}
& .\RunVaaSAutomation.ps1 @scriptParameters
```

## <a name="launch-the-test-pass-workflow"></a>テスト パス ワークフローの起動

VaaS ポートを使用してテスト パス ワークフローを実行する方法については、「[テストのスケジュール設定](azure-stack-vaas-schedule-test-pass.md)」を参照してください。

適切なパラメーター値を使用して、次のスクリプトを実行します。

```powershell
# Review and update the following parameters
$VaaSAccountUserName = ""
$VaaSAccountPassword = ""
$VaaSAccountTenantId = ""
$ServiceAdminUserName = ""
$ServiceAdminPassword = ""
$TenantAdminUserName  = ""
$TenantAdminPassword  = ""
$CloudAdminUserName   = ""
$CloudAdminPassword   = ""
$SolutionName   = ''
$ProjectName    = ''
$DiagnosticsStorageConnection=''

# No need to modify the following lines
# The following code is an example of running the "Cloud Simulation Engine" test
Import-Module .\VaaSAutomation.psm1 -verbose -force
$stampInfo = Get-StampInfo -cloudAdminUserName $CloudAdminUserName -cloudAdminPassword $CloudAdminPassword -retryCount 3 -retryPeriod 30 -outputFolder "."
$AgentName = "$((Get-WmiObject win32_computersystem).DNSHostName).$((Get-WmiObject win32_computersystem).Domain)".ToLower()
$VaaSAccountCredentail = New-Object System.Management.Automation.PSCredential ($VaaSAccountUserName, (ConvertTo-SecureString $VaaSAccountPassword -AsPlainText -Force))

$VaaSTestFilter='Test'
$VaaSTestFilterValue='cloudsimulation'
$testParameters = @{
    'ServiceAdminUser' = $ServiceAdminUserName;
    'ServiceAdminPassword' = $ServiceAdminPassword;
    'TenantAdminUser' = $TenantAdminUserName;
    'TenantAdminPassword' = $TenantAdminPassword;
    'CloudAdminUser' = $CloudAdminUserName;
    'CloudAdminPassword' = $CloudAdminPassword;
    'DomainFQDN' = "";
    'DomainAdminUser' = "";
    'DomainAdminPassword' = "";
    'TenantId' = $stampInfo.AadTenantId;
    'ExternalFqdn' = $stampInfo.ExternalDomainFQDN;
    'NumberOfNodes' = "$($stampInfo.NumberOfNodes)";
    'Region' = $stampInfo.RegionName;
    'RunDurationInHours' = 24;
    'EnableSuBR' = "false";
    'Faults' = "";
    'Resources' = "";
    'FaultControllerSettings' = "default";
    'DiagnosticsStorageConnection' = $diagnosticsStorageConnection;
    'DiagnosticsContainerName' = "$(New-Guid).ToString().ToLower()";
    'MaxFiddlerCaptureSizeInMB' = "0";
    'PackageHashCode' = "";
}

$projectParameters = @{}

$scriptParameters = @{
    'VaaSAccountCredentail' = $VaaSAccountCredentail;
    'VaaSAccountTenantId' = $VaaSAccountTenantId
    'VaaSSolutionName' = $SolutionName;
    'VaaSProjectType' = 'TestPass';
    'VaaSProjectName' = $ProjectName;
    'VaaSProjectParameterHashTable'= $projectParameters;
    'VaaSTestFilter'= $VaaSTestFilter;
    'VaaSTestFilterValue' = $VaaSTestFilterValue;
    'VaaSTestParameterHashTable'= $testParameters;
    'VaaSOnPremAgentName' = $AgentName;
    'MaxScriptWaitTimeInHours' = 24;
    'ServiceAdminUserName' = $ServiceAdminUserName;
    'ServiceAdminPassword' = $ServiceAdminPassword;
    'TenantAdminUserName' = $TenantAdminUserName;
    'TenantAdminPassword' = $TenantAdminPassword;
    'CloudAdminUserName' = $CloudAdminUserName;
    'CloudAdminPassword' = $CloudAdminPassword;
    'DiagnosticsStorageConnection' = $DiagnosticsStorageConnection;
}
& .\RunVaaSAutomation.ps1 @scriptParameters
```

## <a name="parameter-table"></a>パラメーター テーブル

詳細については、「[一般的なワークフロー パラメーター](azure-stack-vaas-parameters.md)」を参照してください。

| パラメーター | 説明 |
| --- | --- |
| VaaSAccountUserName | VaaS ポータルの VaaS ユーザー名。 |
| VaaSAccountPassword | VaaS ポータルの VaaS パスワード。 |
| VaaSAccountTenantId | VaaS テナントの GUID。 |
| ServiceAdminUserName | Azure Stack Hub サービスの管理者アカウント。  |
| ServiceAdminPassword | Azure Stack Hub サービスのパスワード。  |
| TenantAdminUserName | プライマリ テナントの管理者。  |
| TenantAdminPassword | プライマリ テナントのパスワード。  |
| CloudAdminUserName | クラウド管理者のユーザー名。  |
| CloudAdminPassword | クラウド管理者のパスワード。  |
| SolutionName | VaaS ソリューションの名前。 |
| ProjectName | VaaS ワークフローの名前。 |
| DiagnosticsStorageConnection | テストの実行中に診断ログのコピー先となる Azure Storage アカウントの SAS URL。 SAS URL を生成する手順については、「[診断接続文字列を生成する](azure-stack-vaas-parameters.md)」をご覧ください。 |

## <a name="review-the-results"></a>結果の確認

テスト ログとレポートは、現在の作業フォルダーの下に保存されます。 

他のオプションについては、「[VaaS ポータルでのテストの監視と管理](azure-stack-vaas-monitor-test.md)」を参照してください。

## <a name="next-steps"></a>次のステップ

Azure Stack Hub での PowerShell の詳細については、最新のモジュールを参照してください。

- [Azure Stack Hub モジュール](/powershell/azure/azure-stack/overview?view=azurestackps-1.6.0)
