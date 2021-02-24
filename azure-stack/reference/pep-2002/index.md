---
title: Azure Stack Hub の特権エンドポイント リファレンス
description: PowerShell Azure Stack 特権エンドポイントのリファレンス
author: mattbriggs
ms.topic: reference
ms.date: 04/27/2020
ms.author: mabrigg
ms.reviewer: fiseraci
ms.lastreviewed: 04/27/2020
ms.openlocfilehash: 76eb340da04e9254bcf8d8a626822c65362f44d2
ms.sourcegitcommit: e9a1dfa871e525f1d6d2b355b4bbc9bae11720d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86486038"
---
# <a name="azure-stack-hub-privileged-endpoint-reference"></a>Azure Stack Hub の特権エンドポイント リファレンス

PowerShell Azure Stack 特権エンドポイント (PEP) は、あらかじめ構成されたリモート PowerShell コンソールであり、必要なタスクを実行するために十分な機能だけを提供します。 エンドポイントでは PowerShell JEA (Just Enough Administration) を使用して、コマンドレットの限定的なセットのみが公開されます。

## <a name="privilege-endpoint-cmdlets"></a>特権エンドポイントのコマンドレット

| コマンドレット | 説明 |
| --- | --- |
| [Close-PrivilegedEndpoint](Close-PrivilegedEndpoint.md) | 説明なし。 |
| [Get-ActionStatus](Get-ActionStatus.md) | 指定された関数名を持つ操作の最新のアクションの状態を取得します。 |
| [Get-AzureStackLog](Get-AzureStackLog.md) | タイムアウトが指定された AzureStack のさまざまなロールのログを取得します。 |
| [Get-AzureStackStampInformation](Get-AzureStackStampInformation.md) | スタンプ情報を取得します。 |
| [Get-AzureStackSupportConfiguration](Get-AzureStackSupportConfiguration.md) | サポート サービス構成設定を取得します。 |
| [Get-CloudAdminPasswordRecoveryToken](Get-CloudAdminPasswordRecoveryToken.md) | 説明なし。 |
| [Get-CloudAdminUserList](Get-CloudAdminUserList.md) | 説明なし。 |
| [Get-ClusterLog](Get-ClusterLog.md) | 説明なし。 |
| [Get-GraphApplication](Get-GraphApplication.md) | Get-GraphApplication は、指定されたアプリケーションの名前または識別子のグラフ アプリケーション情報を取得するためのラッパー関数です。 |
| [Get-StorageJob](Get-StorageJob.md) | 説明なし。 |
| [Get-SupportSessionInfo](Get-SupportSessionInfo.md) | 説明なし。 |
| [Get-SupportSessionToken](Get-SupportSessionToken.md) | 説明なし。 |
| [Get-SyslogClient](Get-SyslogClient.md) | syslog クライアントの設定を取得します。 |
| [Get-SyslogServer](Get-SyslogServer.md) | syslog サーバー エンドポイントを取得します。 |
| [Get-ThirdPartyNotices](Get-ThirdPartyNotices.md) | 説明なし。 |
| [Get-TLSPolicy](Get-TLSPolicy.md) | 説明なし。 |
| [Get-VirtualDisk](Get-VirtualDisk.md) | 説明なし。 |
| [Invoke-AzureStackOnDemandLog](Invoke-AzureStackOnDemandLog.md) | 必要に応じて、AzureStack ロールのオンデマンド ログを生成します。 |
| [New-AzureBridgeServicePrincipal](New-AzureBridgeServicePrincipal.md) | Azure Active Directory に新しいサービス プリンシパルを作成します。 |
| [New-AzureStackActivation](New-AzureStackActivation.md) | Azure Stack をアクティブ化します。 |
| [New-CloudAdminUser](New-CloudAdminUser.md) | 説明なし。 |
| [New-GraphApplication](New-GraphApplication.md) | New-GraphApplication は、AD FS 上で ADFS Graph コマンドレットを呼び出すラッパー関数です。 |
| [New-RegistrationToken](New-RegistrationToken.md) | 新しい登録トークンを作成します。 |
| [Register-CustomAdfs](Register-CustomAdfs.md) | カスタム Active Directory フェデレーション サービス (ADFS) をクレーム プロバイダーとして Azure Stack AD FS に登録するスクリプト。 |
| [Register-CustomDnsServer](Register-CustomDnsServer.md) | カスタム DNS サーバーを Azure Stack DNS に登録するスクリプト。 |
| [Register-DirectoryService](Register-DirectoryService.md) | 顧客の Active Directory を Graph サービスに登録するためのスクリプト。 |
| [Remove-AzureStackActivation](Remove-AzureStackActivation.md) | 説明なし。 |
| [Remove-CloudAdminUser](Remove-CloudAdminUser.md) | 説明なし。 |
| [Remove-GraphApplication](Remove-GraphApplication.md) | Remove-GraphApplication は、AD FS 上で ADFS Graph コマンドレットを呼び出すラッパー関数です。 |
| [Repair-VirtualDisk](Repair-VirtualDisk.md) | 説明なし。 |
| [Reset-DatacenterIntegrationConfiguration](Reset-DatacenterIntegrationConfiguration.md) | データセンターの統合の変更をリセットするためのスクリプトです。 |
| [Send-AzureStackDiagnosticLog](Send-AzureStackDiagnosticLog.md) | Azure Stack 診断ログを Microsoft に送信します。 |
| [Set-CloudAdminUserPassword](Set-CloudAdminUserPassword.md) | 説明なし。 |
| [Set-GraphApplication](Set-GraphApplication.md) | Set-GraphApplication は、AD FS 上で ADFS Graph コマンドレットを呼び出すラッパー関数です。 |
| [Set-ServiceAdminOwner](Set-ServiceAdminOwner.md) | サービス管理者を更新するためのスクリプト。 |
| [Set-SyslogClient](Set-SyslogClient.md) | syslog クライアント エンドポイント証明書をインポートして適用します。 |
| [Set-SyslogServer](Set-SyslogServer.md) | syslog サーバー エンドポイントを設定します。 |
| [Set-Telemetry](Set-Telemetry.md) | Microsoft への利用統計情報の転送を有効または無効にします。 |
| [Set-TLSPolicy](Set-TLSPolicy.md) | 説明なし。 |
| [Start-AzureStack](Start-AzureStack.md) | すべての Azure Stack サービスを開始します。 |
| [Start-SecretRotation](Start-SecretRotation.md) | スタンプのシークレット ローテーションをトリガーします。 |
| [Stop-AzureStack](Stop-AzureStack.md) | すべての Azure Stack サービスを停止します。 |
| [Test-AzureStack](Test-AzureStack.md) | Azure Stack の状態を検証します。 |
| [Unlock-SupportSession](Unlock-SupportSession.md) | 説明なし。 |

## <a name="next-steps"></a>次のステップ

Azure Stack Hub 上の特権エンドポイントの詳細については、[Azure Stack での特権エンドポイントの使用](../../operator/azure-stack-privileged-endpoint.md)に関するページを参照してください。
