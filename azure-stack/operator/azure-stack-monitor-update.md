---
title: 特権エンドポイントを使用して Azure Stack Hub での更新プログラムをモニターする
description: 特権エンドポイントを使用して Azure Stack Hub 統合システムの更新プログラムの状態をモニターする方法について説明します。
author: PatAltimore
ms.topic: how-to
ms.date: 03/04/2020
ms.author: patricka
ms.reviewer: fiseraci
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: fd7c1740d96ecea92bd4414ae1b4481e38594202
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99245978"
---
# <a name="monitor-updates-in-azure-stack-hub-using-the-privileged-endpoint"></a>特権エンドポイントを使用して Azure Stack Hub での更新プログラムをモニターする

[特権エンドポイント](azure-stack-privileged-endpoint.md)を使用して、Azure Stack Hub 更新プログラム実行の進行状況をモニターすることができます。 また、Azure Stack Hub ポータルが利用できなくなった場合は、実行が失敗した更新プログラムを、特権エンドポイントを使用して、最後に成功した手順から再開することもできます。 Azure Stack Hub ポータルの使用は、Azure Stack Hub で更新プログラムを管理するための推奨される方法です。

次に示す更新プログラム管理用の新しい PowerShell コマンドレットは、Azure Stack Hub 統合システムの 1710 更新プログラムに含まれています。

| コマンドレット  | 説明  |
|---------|---------|
| `Get-AzureStackUpdateStatus` | 現在実行中、完了済み、または失敗した更新プログラムの状態を返します。 更新操作の状態の詳細と、現在のステップと対応する状態の両方について説明する XML ドキュメントを提供します。 |
| `Resume-AzureStackUpdate` | 失敗した更新プログラムを失敗した時点から再開します。 特定のシナリオでは、更新プログラムを再開する前に、リスク軽減の手順を完了しなければならない場合があります。         |
| | |

## <a name="verify-the-cmdlets-are-available"></a>コマンドレットが利用可能なことを確認する
これらのコマンドレットは、Azure Stack Hub の 1710 更新プログラム パッケージの新機能であるため、1710 更新プロセスを、モニター機能が使用可能となる前の特定の時点まで進める必要があります。 通常は、管理者ポータルの状態で、1710 更新が「**ストレージ ホストの再起動**」の手順に達していることが示された場合に、コマンドレットを使用できるようになります。 具体的には、コマンドレットの更新は、「**手順: 手順 2.6 の実行 - PrivilegedEndpoint 許可リストの更新**」の間に行われます。

特権エンドポイントからコマンド一覧のクエリを実行することによって、コマンドレットが使用可能かどうかをプログラムによって判別することもできます。 このクエリを実行するには、ハードウェア ライフ サイクル ホストまたは特権アクセス ワークステーションから、次のコマンドを実行します。 さらに、特権エンドポイントが信頼されたホストであることを確認してください。 詳細については、「[特権エンドポイントへのアクセス](azure-stack-privileged-endpoint.md#access-the-privileged-endpoint)」の手順 1 を参照してください。

1. Azure Stack Hub 環境内のいずれかの ERCS 仮想マシン (VM) で PowerShell セッションを作成します ("*プレフィックス*"-ERCS01、"*プレフィックス*"-ERCS02、または "*プレフィックス*"-ERCS03)。 *プレフィックス* を、ご使用の環境に固有の VM のプレフィックス文字列に置換します。

   ```powershell
   $cred = Get-Credential

   $pepSession = New-PSSession -ComputerName <Prefix>-ercs01 -Credential $cred -ConfigurationName PrivilegedEndpoint 
   ```
   資格情報を要求するプロンプトが表示されたら、&lt;"*Azure Stack Hub ドメイン*"&gt;\cloudadmin アカウント、または CloudAdmins グループのメンバーであるアカウントを使用します。 CloudAdmin アカウントの場合、インストール中に AzureStackAdmin ドメイン管理者アカウントのパスワードとして指定したものと同じパスワードを入力します。

2. 特権エンドポイントで使用できるコマンドの完全な一覧を取得します。

   ```powershell
   $commands = Invoke-Command -Session $pepSession -ScriptBlock { Get-Command } 
   ```
3. 特権エンドポイントが更新されたかどうかを判別します。

   ```powershell
   $updateManagementModuleName = "Microsoft.Azurestack.UpdateManagement"
    if (($commands | ? Source -eq $updateManagementModuleName)) {
   Write-Host "Privileged endpoint was updated to support update monitoring tools."
    } else {
   Write-Host "Privileged endpoint has not been updated yet. Please try again later."
    } 
   ```

4. Microsoft.AzureStack.UpdateManagement モジュールに固有のコマンドを一覧表示します。

   ```powershell
   $commands | ? Source -eq $updateManagementModuleName 
   ```
   次に例を示します。
   ```powershell
   $commands | ? Source -eq $updateManagementModuleName
   
   CommandType     Name                                               Version    Source                                                  PSComputerName
    -----------     ----                                               -------    ------                                                  --------------
   Function        Get-AzureStackUpdateStatus                         0.0        Microsoft.Azurestack.UpdateManagement                   Contoso-ercs01
   Function        Resume-AzureStackUpdate                            0.0        Microsoft.Azurestack.UpdateManagement                   Contoso-ercs01
   ``` 

## <a name="use-the-update-management-cmdlets"></a>更新管理コマンドレットを使用する

> [!NOTE]
> ハードウェア ライフ サイクル ホストまたは特権アクセス ワークステーションから、次のコマンドを実行します。 さらに、特権エンドポイントが信頼されたホストであることを確認してください。 詳細については、「[特権エンドポイントへのアクセス](azure-stack-privileged-endpoint.md#access-the-privileged-endpoint)」の手順 1 を参照してください。

### <a name="connect-to-the-privileged-endpoint-and-assign-session-variable"></a>特権エンドポイントに接続し、セッション変数を割り当てる

次のコマンドを実行し、Azure Stack Hub 環境内のいずれかの ERCS VM で PowerShell セッションを作成して ("*プレフィックス*"-ERCS01、"*プレフィックス*"-ERCS02、または "*プレフィックス*"-ERCS03)、セッション変数を割り当てます。

```powershell
$cred = Get-Credential

$pepSession = New-PSSession -ComputerName <Prefix>-ercs01 -Credential $cred -ConfigurationName PrivilegedEndpoint 
```
 資格情報を要求するプロンプトが表示されたら、&lt;"*Azure Stack Hub ドメイン*"&gt;\cloudadmin アカウント、または CloudAdmins グループのメンバーであるアカウントを使用します。 CloudAdmin アカウントの場合、インストール中に AzureStackAdmin ドメイン管理者アカウントのパスワードとして指定したものと同じパスワードを入力します。

### <a name="get-high-level-status-of-the-current-update-run"></a>現在の更新実行の詳細な状態を取得する

現在の更新実行の詳細な状態を取得するには、次のコマンドを実行します。

```powershell
$statusString = Invoke-Command -Session $pepSession -ScriptBlock { Get-AzureStackUpdateStatus -StatusOnly }

$statusString.Value 
```

指定できる値は、次のとおりです。

- 実行中
- 完了
- 失敗 
- Canceled

これらのコマンドを繰り返し実行することで、最新の状態を表示することができます。 チェックする接続を再確立する必要はありません。

### <a name="get-the-full-update-run-status-with-details"></a>完全な更新実行の状態とその詳細を取得する

完全な更新実行の概要は XML 文字列として取得できます。 この文字列は、調査用にファイルに書き込むか、XML ドキュメントに変換し、PowerShell を使用して解析することができます。 次のコマンドでは、XML を解析して現在実行中の手順の階層リストが取得されます。

```powershell
[xml]$updateStatus = Invoke-Command -Session $pepSession -ScriptBlock { Get-AzureStackUpdateStatus }

$updateStatus.SelectNodes("//Step[@Status='InProgress']")
```

次の例にある大まかな手順 (クラウド更新) には、ストレージ ホストを更新して再起動するための子計画が含まれています。 これは、[ストレージ ホストの再起動] プランが、ホストの 1 つで Blob Storage サービスを更新していることを示しています。

```powershell
[xml]$updateStatus = Invoke-Command -Session $pepSession -ScriptBlock { Get-AzureStackUpdateStatus }

$updateStatus.SelectNodes("//Step[@Status='InProgress']") 

    FullStepIndex : 2
    Index         : 2
    Name          : Cloud Update
    Description   : Perform cloud update.
    StartTimeUtc  : 2017-10-13T12:50:39.9020351Z
    Status        : InProgress
    Task          : Task
    
    FullStepIndex  : 2.9
    Index          : 9
    Name           : Restart Storage Hosts
    Description    : Restart Storage Hosts.
    EceErrorAction : Stop
    StartTimeUtc   : 2017-10-13T15:44:06.7431447Z
    Status         : InProgress
    Task           : Task
    
    FullStepIndex : 2.9.2
    Index         : 2
    Name          : PreUpdate ACS Blob Service
    Description   : Check function level, update deployment artifacts, configure Blob service settings
    StartTimeUtc  : 2017-10-13T15:44:26.0708525Z
    Status        : InProgress
    Task          : Task
```

### <a name="resume-a-failed-update-operation"></a>失敗した更新操作を再開する

更新が失敗した場合は、失敗した場所から更新実行を再開できます。

```powershell
Invoke-Command -Session $pepSession -ScriptBlock { Resume-AzureStackUpdate } 
```

## <a name="troubleshoot"></a>トラブルシューティング

特権エンドポイントは、Azure Stack Hub 環境内のすべての ERCS VM で使用できます。 高可用性エンドポイントへの接続は確立されないため、中断したり、警告またはエラー メッセージが表示されたりすることがあります。 これらのメッセージは、セッションが切断されたか、または ECE サービスとの通信エラーがあったことを示している場合があります。 これは正しい動作です。 しばらく待ってから操作を再試行するか、他の ERCS VM のいずれかで新しい特権エンドポイント セッションを作成できます。

更新プログラムのトラブルシューティングの詳細については、[Azure Stack のトラブルシューティング](azure-stack-troubleshooting.md)に関するページをご覧ください

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub での更新プログラム管理](azure-stack-updates.md)


