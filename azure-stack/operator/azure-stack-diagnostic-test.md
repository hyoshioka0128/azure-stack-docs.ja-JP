---
title: Azure Stack Hub 検証ツールを使用してシステムの状態を検証する
description: Azure Stack Hub 検証ツールを使用してシステムの状態を検証する方法について説明します。
author: justinha
ms.topic: article
ms.date: 01/10/2020
ms.author: justinha
ms.reviewer: adshar
ms.lastreviewed: 01/10/2020
ms.openlocfilehash: 40a916a282f4808d9897cc5c23ea953739b0a0cf
ms.sourcegitcommit: 3e2460d773332622daff09a09398b95ae9fb4188
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/15/2020
ms.locfileid: "90572937"
---
# <a name="validate-azure-stack-hub-system-state"></a>Azure Stack Hub システムの状態を検証する

Azure Stack Hub のオペレーターは、使用しているシステムの正常性と状態をオンデマンドで判断できる必要があります。 Azure Stack Hub の検証ツール (**Test-AzureStack**) は、お使いのシステムで実行した一連のテストから、存在する場合に障害を特定する PowerShell コマンドレットです。 通常このツールは、問題について Microsoft カスタマー サービス サポート (Microsoft サポート) に問い合わせるときに、[特権エンドポイント (PEP)](azure-stack-privileged-endpoint.md) から実行するよう求められます。 手元にシステム全体の正常性および状態情報があれば、Microsoft サポートは詳細ログを収集して分析し、エラーが発生した領域に焦点を当て、お客様と連携して問題を修正できます。

## <a name="running-the-validation-tool-and-accessing-results"></a>検証ツールを実行して結果にアクセスする

前述のとおり、この検証ツールは PEP 経由で実行されます。 各テストは、PowerShell ウィンドウで**合格/不合格**のいずれかの状態を返します。 次に示すのは、エンド ツー エンド検証テスト プロセスの概要です。

1. 信頼関係を確立します。 統合システムで、管理者特権の Windows PowerShell セッションから次のコマンドを実行して、ハードウェア ライフサイクル ホストまたは特権アクセス ワークステーションで実行されているセキュリティ強化された VM の信頼されたホストとして PEP を追加します。

   ```powershell
   winrm s winrm/config/client '@{TrustedHosts="<IP Address of Privileged Endpoint>"}'
   ```

   Azure Stack Development Kit (ASDK) を実行している場合、開発キットのホストにサインインします。

1. PEP にアクセスします。 PEP セッションを確立するために、次のコマンドを実行します。

   ```powershell
   Enter-PSSession -ComputerName "<ERCS VM-name/IP address>" -ConfigurationName PrivilegedEndpoint -Credential $localcred 
   ```

   > [!TIP]
   > Azure Stack Development Kit (ASDK) ホスト コンピューターで PEP にアクセスするには、-ComputerName に対して AzS-ERCS01 を使用します。

1. PEP にアクセスしたら、次を実行します。

   ```powershell
   Test-AzureStack
   ```

   詳細については、「[パラメーターに関する考慮事項](azure-stack-diagnostic-test.md#parameter-considerations)」と「[ユース ケースの例](azure-stack-diagnostic-test.md#use-case-examples)」を参照してください。

1. いずれかのテストで **FAIL** が報告された場合は、`Get-AzureStackLog` を実行します。 統合システムでの手順については、[Azure Stack Hub 統合システムでの Get-AzureStackLog](azure-stack-get-azurestacklog.md)の実行方法を参照してください。

   このコマンドレットは、Test-azurestack によって生成されたログを収集します。 テストで **WARN** が報告される場合は、ログを収集せずに Microsoft サポートに連絡することをお勧めします。

1. Microsoft サポートにより検証ツールを実行するように指示された場合、Microsoft サポートの担当者から、問題のトラブルシューティングを続行するために、お客様が収集したログの提出が求められます。

## <a name="tests-available"></a>利用可能なテスト

検証ツールを使用すると、一連のシステム レベルのテストと基本的なクラウド シナリオを実行して、現在の状態を把握し、システムの問題を修正できます。

### <a name="cloud-infrastructure-tests"></a>クラウド インフラストラクチャのテスト

これらの影響度が低いテストは、インフラストラクチャ レベルで動作し、さまざまなシステム コンポーネントや機能に関する情報を提供します。 現時点では、テストは、次のカテゴリにグループ化されます。

| テスト カテゴリ                                        | -Include および -Ignore の引数 |
| :--------------------------------------------------- | :-------------------------------- |
| Azure Stack Hub ACS の概要                              | AzsAcsSummary                     |
| Azure Stack Hub Active Directory の概要                 | AzsAdSummary                      |
| Azure Stack Hub のアラートの概要                            | AzsAlertSummary                   |
| Azure Stack Hub アプリケーションのクラッシュの概要                | AzsApplicationCrashSummary        |
| Azure Stack Hub バックアップ共有のアクセシビリティの概要       | AzsBackupShareAccessibility       |
| Azure Stack Hub BMC の概要                              | AzsStampBMCSummary                |
| Azure Stack Hub クラウド ホスティング インフラストラクチャの概要     | AzsHostingInfraSummary            |
| Azure Stack Hub クラウド ホスティング インフラストラクチャの利用 | AzsHostingInfraUtilization        |
| Azure Stack Hub コントロール プレーンの概要                    | AzsControlPlane                   |
| Azure Stack Hub Defender の概要                         | AzsDefenderSummary                |
| Azure Stack Hub のホスティング インフラストラクチャ ファームウェアの概要  | AzsHostingInfraFWSummary          |
| Azure Stack Hub インフラストラクチャの容量                  | AzsInfraCapacity                  |
| Azure Stack Hub インフラストラクチャのパフォーマンス               | AzsInfraPerformance               |
| Azure Stack Hub インフラストラクチャ ロールの概要              | AzsInfraRoleSummary               |
| Azure Stack Hub ネットワーク インフラストラクチャ                            | AzsNetworkInfra                   |
| Azure Stack Hub ポータルおよび API の概要                   | AzsPortalAPISummary               |
| Azure Stack Hub スケール ユニットの VM イベント                     | AzsScaleUnitEvents                |
| Azure Stack Hub スケール ユニットの VM リソース                  | AzsScaleUnitResources             |
| Azure Stack Hub のシナリオ                                | AzsScenarios                      |
| Azure Stack Hub SDN 検証の概要                   | AzsSDNValidation                  |
| Azure Stack Hub サービス ファブリック ロールの概要              | AzsSFRoleSummary                  |
| Azure Stack Hub ストレージのデータ プレーン                       | AzsStorageDataPlane               |
| Azure Stack Hub のストレージ サービスの概要                 | AzsStorageSvcsSummary             |
| Azure Stack Hub SQL ストアの概要                        | AzsStoreSummary                   |
| Azure Stack Hub 更新プログラムの概要                           | AzsInfraUpdateSummary             |
| Azure Stack Hub の VM 配置の概要                     | AzsVmPlacement                    |

### <a name="cloud-scenario-tests"></a>クラウド シナリオのテスト

上記のインフラストラクチャ テストに加えて、クラウド シナリのオテストを実行して、インフラストラクチャ コンポーネント全体の機能を確認することもできます。 リソースのデプロイが関係するため、これらのテストを実行するは、クラウド管理者の資格情報が必要です。

> [!NOTE]
> 現時点では、Active Directory フェデレーション サービス (AD FS) の資格情報を使用してクラウド シナリオのテストを実行することはできません。

検証ツールでは、次のクラウドのシナリオがテストされます。
- リソース グループの作成
- プランの作成
- オファーの作成
- ストレージ アカウントの作成
- 仮想マシン (VM) の作成
- Blob ストレージの操作
- キュー ストレージの操作
- テーブル ストレージの操作

## <a name="parameter-considerations"></a>パラメーターに関する考慮事項

- パラメーター **List** は、利用可能なすべてのテスト カテゴリを表示するために使用できます。

- パラメーター **Include** と **Ignore** は、テスト カテゴリを含めたり除外したりするために使用できます。 これらの引数の詳細については、次のセクションを参照してください。

  ```powershell
  Test-AzureStack -Include AzsSFRoleSummary, AzsInfraCapacity
  ```

  ```powershell
  Test-AzureStack -Ignore AzsInfraPerformance
  ```

- テナント VM はクラウド シナリオ テストの一部としてデプロイされます。 この VM のデプロイを無効にするには、**DoNotDeployTenantVm** を使用します。

- クラウド シナリオ テストを実行するには、**ServiceAdminCredential** パラメーターを指定する必要があります。これについては「[ユース ケースの例](azure-stack-diagnostic-test.md#use-case-examples)」のセクションで説明しています。

- **BackupSharePath** と **BackupShareCredential** は、インフラストラクチャ バックアップ設定をテストするときに使用されます。これについては「[ユース ケースの例](azure-stack-diagnostic-test.md#use-case-examples)」セクションで説明しています。

- **DetailedResults** を使用すると、実行全体だけでなく、各テストの合格/不合格/警告の情報を取得できます。 指定しないと、失敗がない場合は **$true** が、失敗がある場合は **$false** が **Test-AzureStack** から返されます。
- **TimeoutSeconds** を使用すると、各グループでの完了に対する特定の時刻を設定できます。

- 検証ツールは、Verbose、Debug、ErrorAction、ErrorVariable、WarningAction、WarningVariable、OutBuffer、PipelineVariable、および OutVariable などの一般的な PowerShell パラメーターもサポートします。 詳細については、「[About Common Parameters (一般的なパラメーターについて)](https://go.microsoft.com/fwlink/?LinkID=113216)」を参照してください。  

## <a name="use-case-examples"></a>ユース ケースの例

### <a name="run-validation-without-cloud-scenarios"></a>クラウド シナリオがない検証の実行

クラウド シナリオ テストの実行をスキップするには、**ServiceAdminCredential** パラメーターを指定せずに検証ツールを実行します。 

```powershell
New-PSSession -ComputerName "<ERCS VM-name/IP address>" -ConfigurationName PrivilegedEndpoint -Credential $localcred
Test-AzureStack
```

### <a name="run-validation-with-cloud-scenarios"></a>クラウド シナリオがある検証の実行

検証ツールに **ServiceAdminCredentials** パラメーターを指定すると、既定でクラウド シナリオ テストが実行されます。 

```powershell
Enter-PSSession -ComputerName "<ERCS VM-name/IP address>" -ConfigurationName PrivilegedEndpoint -Credential $localcred 
Test-AzureStack -ServiceAdminCredential "<Cloud administrator user name>" 
```

残りのテストを実行せずにクラウド シナリオのみを実行したい場合は、**Include** パラメーターを使用してそのようにすることができます。 

```powershell
Enter-PSSession -ComputerName "<ERCS VM-name/IP address>" -ConfigurationName PrivilegedEndpoint -Credential $localcred 
Test-AzureStack -ServiceAdminCredential "<Cloud administrator user name>" -Include AzsScenarios   
```

クラウド管理者のユーザー名は UPN 形式serviceadmin@contoso.onmicrosoft.com (Azure AD) で入力する必要があります。 入力を求められたら、クラウド管理者のアカウントのパスワードを入力します。

### <a name="groups"></a>グループ

オペレーターの操作性を向上させるため、複数のテスト カテゴリを同時に実行できるように **Group** パラメーターが有効になっています。 現時点では、次の 3 つのグループが定義されています。**既定**、**UpdateReadiness**、および **SecretRotationReadiness**。

- **既定**:**Test-AzureStack** の標準実行と見なされます。 他のグループが選択されていない場合、このグループが既定で実行されます。
- **UpdateReadiness**:Azure Stack Hub インスタンスが更新可能かどうかを確認するためのチェック。 **UpdateReadiness** グループが実行されると、コンソール出力にエラーとして警告が表示されます。これは更新の妨げと見なします。 Azure Stack Hub バージョン 1910 の時点では、次のカテゴリは、**UpdateReadiness** グループの一部です。

  - **AzsInfraFileValidation**
  - **AzsActionPlanStatus**
  - **AzsStampBMCSummary**

- **SecretRotationReadiness**:Azure Stack Hub のインスタンスが、シークレット ローテーションを実行できる状態にあるかどうかを確認するためのチェック。 **SecretRotationReadiness** グループが実行されると、コンソール出力にエラーとして警告が表示されます。これはシークレット ローテーションの妨げと見なします。 次のカテゴリは、SecretRotationReadiness グループの一部です。

  - **AzsAcsSummary**
  - **AzsDefenderSummary**
  - **AzsHostingInfraSummary**
  - **AzsInfraCapacity**
  - **AzsInfraRoleSummary**
  - **AzsPortalAPISummary**
  - **AzsSFRoleSummary**
  - **AzsStorageSvcsSummary**
  - **AzsStoreSummary**

#### <a name="group-parameter-example"></a>Group パラメーターの例

次の例では、**Group** を使用して更新プログラムまたは修正プログラムをインストールする前に、**Test-AzureStack** を実行してシステムの準備状況をテストします。 更新プログラムまたは修正プログラムのインストールを開始する前に、**Test-AzureStack** を実行してお使いの Azure Stack Hub の状態を確認します。

```powershell
Test-AzureStack -Group UpdateReadiness
```

お使いの Azure Stack Hub で 1811 より前のバージョンを実行している場合、次の PowerShell コマンドを使用して **Test-AzureStack** を実行してください。

```powershell
New-PSSession -ComputerName "<ERCS VM-name/IP address>" -ConfigurationName PrivilegedEndpoint -Credential $localcred 
Test-AzureStack -Include AzsControlPlane, AzsDefenderSummary, AzsHostingInfraSummary, AzsHostingInfraUtilization, AzsInfraCapacity, AzsInfraRoleSummary, AzsPortalAPISummary, AzsSFRoleSummary, AzsStampBMCSummary
```

### <a name="run-validation-tool-to-test-infrastructure-backup-settings"></a>インフラストラクチャのバックアップ設定をテストするには、検証ツールを実行します。

インフラストラクチャのバックアップを構成する*前に*、**AzsBackupShareAccessibility** テストを使用して、バックアップ共有パスと資格情報をテストできます。

  ```powershell
  Enter-PSSession -ComputerName "<ERCS VM-name/IP address>" -ConfigurationName PrivilegedEndpoint -Credential $localcred 
  Test-AzureStack -Include AzsBackupShareAccessibility -BackupSharePath "\\<fileserver>\<fileshare>" -BackupShareCredential $using:backupcred
  ```

バックアップを構成した*後*、PEP セッションの実行から **AzsBackupShareAccessibility** を実行して、ERCS から共有にアクセスできることを検証できます。

  ```powershell
  Enter-PSSession -ComputerName "<ERCS VM-name/IP address>" -ConfigurationName PrivilegedEndpoint -Credential $localcred 
  Test-AzureStack -Include AzsBackupShareAccessibility
  ```

構成済みのバックアップ共有で新しい資格情報をテストするには、以下を実行します。 

  ```powershell
  Enter-PSSession -ComputerName "<ERCS VM-name/IP address>" -ConfigurationName PrivilegedEndpoint -Credential $localcred 
  Test-AzureStack -Include AzsBackupShareAccessibility -BackupShareCredential "<PSCredential for backup share>"
  ```

### <a name="run-validation-tool-to-test-network-infrastructure"></a>検証ツールを実行してネットワーク インフラストラクチャをテストする

このテストでは、Azure Stack Hub ソフトウェア定義ネットワーク (SDN) をバイパスするネットワーク インフラストラクチャの接続を確認します。 パブリック VIP から構成済みの DNS フォワーダー、NTP サーバー、および認証エンドポイントへの接続が示されます。 これには、ID プロバイダーとして Azure AD を使用する場合の Azure への接続、または ID プロバイダーとして AD FS を使用する場合のフェデレーション サーバーへの接続が含まれます。

コマンドの詳細な出力を取得するには、debug パラメーターを含めます。

```powershell
Test-AzureStack -Include AzsNetworkInfra -Debug
```

## <a name="next-steps"></a>次のステップ

Azure Stack Hub 診断ツールと問題のログ記録の詳細については、[Azure Stack Hub の診断ツール](./azure-stack-diagnostic-log-collection-overview.md?view=azs-2002)に関する記事を参照してください。

トラブルシューティングの詳細については、[Microsoft Azure Stack Hub でのトラブルシューティング](azure-stack-troubleshooting.md)に関する記事を参照してください。
