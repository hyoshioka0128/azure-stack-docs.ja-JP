---
title: Azure Graph 統合を検証する
titleSuffix: Azure Stack Hub
description: Azure Stack Hub 適合性チェッカーを使用して、Azure Stack Hub の Graph 統合を検証します。
author: BryanLa
ms.topic: how-to
ms.date: 10/19/2020
ms.author: bryanla
ms.reviewer: jerskine
ms.lastreviewed: 10/19/2020
ms.openlocfilehash: 1abb44838e0c70764d08a4aa47a0fdf73660072d
ms.sourcegitcommit: 695f56237826fce7f5b81319c379c9e2c38f0b88
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/12/2020
ms.locfileid: "94545603"
---
# <a name="validate-graph-integration-for-azure-stack-hub"></a>Azure Stack Hub の Graph 統合を検証する

Azure Stack Hub 適合性チェッカー ツール (AzsReadinessChecker) を使用して、対象の環境で Azure Stack Hub と Graph を統合する準備が整っていることを検証します。 データ センターの統合を開始したり、Azure Stack Hub をデプロイしたりする前に、Graph 統合を検証します。

適合性チェッカーは以下を検証します。

* Graph 統合用に作成したサービス アカウントの資格情報に、Active Directory に対してクエリを実行するための適切な権限が含まれていること。
* " *グローバル カタログ* " を解決できること。また、グローバル カタログに接続できること。
* KDC を解決できること。また、KDC に接続できること。
* 必要なネットワーク接続が存在すること。

Azure Stack Hub とデータ センターの統合の詳細については、「[Azure Stack Hub とデータセンターの統合 - ID](azure-stack-integrate-identity.md)」を参照してください。

## <a name="get-the-readiness-checker-tool"></a>適合性チェッカー ツールを取得する

最新バージョンの Azure Stack Hub 適合性チェッカー ツール (AzsReadinessChecker) を [PowerShell ギャラリー](https://aka.ms/AzsReadinessChecker)からダウンロードします。

## <a name="prerequisites"></a>前提条件

次の前提条件を満たす必要があります。

**ツールを実行するコンピューター:**

* ドメインに接続された Windows 10 または Windows Server 2016。
* PowerShell 5.1 以降。 お使いのバージョンを確認するには、次の PowerShell コマンドを実行し、" *メジャー* " バージョンと " *マイナー* " バージョンを確かめます。
    ```powershell
    $PSVersionTable.PSVersion
    ```
* Active Directory PowerShell モジュール。
* 最新バージョンの [Microsoft Azure Stack Hub 適合性チェッカー](https://aka.ms/AzsReadinessChecker) ツール。

**Active Directory の環境:**

* 既存の Active Directory インスタンスで Graph サービスのアカウントのユーザー名とパスワードを特定します。
* Active Directory フォレストの ルート FQDN を特定します。

## <a name="validate-the-graph-service"></a>Graph サービスを検証する

1. 前提条件を満たしているコンピューターで、管理 PowerShell プロンプトを開き、次のコマンドを実行して、AzsReadinessChecker をインストールします。

    ```powershell
    Install-Module Microsoft.AzureStack.ReadinessChecker -Force -AllowPrerelease
    ```

1. PowerShell プロンプトから次のコマンドを実行して、 *$graphCredential* 変数を Graph アカウントに設定します。 `domain\username` の形式を使用して `contoso\graphservice` をご自身のアカウントで置き換えます。

    ```powershell
    $graphCredential = Get-Credential contoso\graphservice -Message "Enter Credentials for the Graph Service Account"
    ```

1. PowerShell プロンプトから次のコマンドを実行して、Graph サービスの検証を開始します。 `-ForestFQDN` の値をフォレスト ルートの FQDN として指定します。

    ```powershell
    Invoke-AzsGraphValidation -ForestFQDN contoso.com -Credential $graphCredential
    ```

1. ツールの実行後、出力を確認します。 Graph 統合の要件について、状態が OK であることを確認します。 検証が成功した場合は、次の例と同様になります。

    ```powershell
    Testing Graph Integration (v1.0)
            Test Forest Root:            OK
            Test Graph Credential:       OK
            Test Global Catalog:         OK
            Test KDC:                    OK
            Test LDAP Search:            OK
            Test Network Connectivity:   OK

    Details:

    [-] In standalone mode, some tests should not be considered fully indicative of connectivity or readiness the Azure Stack Hub Stamp requires prior to Datacenter Integration.

    Additional help URL: https://aka.ms/AzsGraphIntegration

    AzsReadinessChecker Log location (contains PII): C:\Users\username\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessChecker.log

    AzsReadinessChecker Report location (contains PII): C:\Users\username\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessCheckerReport.json

    Invoke-AzsGraphValidation Completed
    ```

運用環境では、オペレーターのワークステーションからのネットワーク接続をテストしても、その接続を Azure Stack Hub で使用できることを完全に示しているわけではありません。 Azure Stack Hub スタンプのパブリック VIP ネットワークでは、ID の統合を実行する LDAP トラフィック用の接続が必要です。

## <a name="report-and-log-file"></a>レポートとログ ファイル

検証を実行するたびに、結果のログが **AzsReadinessChecker.log** と **AzsReadinessCheckerReport.json** に出力されます。 これらのファイルの場所は、PowerShell に検証結果と共に表示されます。

これらの検証ファイルは、Azure Stack Hub をデプロイする前、または検証に関する問題を調査する前に、状態を共有するときに役立ちます。 両方のファイルに、以降の各検証チェックの結果が保持されます。 デプロイ チームはこのレポートを使用して ID 構成を確認できます。 デプロイ チームやサポート チームは、検証の問題を調査する際に、このログ ファイルを役立たせることができます。

既定では、両方のファイルが `C:\Users\<username>\AppData\Local\Temp\AzsReadinessChecker\` に書き込まれます

次のコマンドを使用します。

* `-OutputPath`:別のレポートの場所を指定するには、実行コマンドの末尾に *path* パラメーターを使用します。
* `-CleanReport`:前のレポート情報から *AzsReadinessCheckerReport.json* をクリアするために、実行コマンドの末尾に付けるパラメーターです。 詳細については、「[Azure Stack Hub 検証レポート](azure-stack-validation-report.md)」を参照してください。

## <a name="validation-failures"></a>検証エラー

検証チェックに失敗した場合は、エラーの詳細が PowerShell ウィンドウに表示されます。 また、ツールによって、 *AzsGraphIntegration.log* にログ情報が記録されます。

## <a name="next-steps"></a>次のステップ

[対応状況レポートを表示する](azure-stack-validation-report.md)  
[Azure Stack Hub の統合に関する一般的な考慮事項](azure-stack-datacenter-integration.md)  
