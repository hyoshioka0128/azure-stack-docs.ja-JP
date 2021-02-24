---
title: Azure Stack Hub 検証レポート
titleSuffix: Azure Stack Hub
description: Azure Stack Hub 適合性チェッカー ツールを使用して、検証レポートを生成します。
author: PatAltimore
ms.topic: how-to
ms.date: 03/04/2020
ms.author: patricka
ms.reviewer: unknown
ms.lastreviewed: 10/23/2019
ms.openlocfilehash: ec94d4112bc1739aad974220aa4438c99874795d
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97868932"
---
# <a name="azure-stack-hub-validation-report"></a>Azure Stack Hub 検証レポート

[Azure Stack 適合性チェッカー ツール](https://www.powershellgallery.com/packages/Microsoft.AzureStack.ReadinessChecker/1.2002.1111.69)を使用して、Azure Stack Hub 環境のデプロイとサービスをサポートする検証を実行します。 このツールでは、結果が .json レポート ファイルに書き込まれます。 レポートには、Azure Stack Hub デプロイの前提条件の状態に関する詳細データと概要データが表示されます。 また、既存の Azure Stack Hub デプロイのシークレット ローテーションについての情報も表示されます。  

## <a name="where-to-find-the-report"></a>レポートの場所

ツールを実行すると、結果のログが **AzsReadinessCheckerReport.json** に出力されます。 また、ツールにより、**AzsReadinessChecker.log** という名前のログも作成されます。 これらのファイルの場所は、PowerShell に検証結果と共に表示されます。

![Azure Stack Hub 適合性チェッカーの run-validation 結果](./media/azure-stack-validation-report/validation.png)

両方のファイルに、同じコンピューター上で実行された以降の検証チェックの結果が保持されます。 たとえば、ツールを実行して証明書を検証した後、再度実行して Azure ID を検証し、3 回目に実行では登録を検証することができます。 この 3 回の検証の結果はすべて、結果として生成される .json レポートで確認できます。  

既定では、両方のファイルが `C:\Users\username\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessCheckerReport.json` に書き込まれます  

- 別のレポートの場所を指定するには、コマンド ラインの末尾に `-OutputPath <path>` パラメーターを使用します。
- ツールの以前の実行に関する情報を **AzsReadinessCheckerReport.json** からクリアするには、コマンド ラインの末尾に `-CleanReport` パラメーターを使用します。

## <a name="view-the-report"></a>レポートを表示する

PowerShell でレポートを表示するには、レポートへのパスを `-ReportPath` の値として指定します。 このコマンドにより、レポートのコンテンツが表示されます。また、結果がまだ出力されていない検証が特定されます。

たとえば、レポートがある場所で開かれている PowerShell プロンプトからレポートを表示するには、次のコマンドを実行します。

```powershell
Read-AzsReadinessReport -ReportPath .\AzsReadinessReport.json
```

出力は次の例のようになります。

```shell
Reading All Validation(s) from Report C:\Contoso-AzsReadinessCheckerReport.json

############### Certificate Validation Summary ###############

Certificate Validation results not available.

############### Registration Validation Summary ###############

Azure Registration Validation results not available.

############### Azure Identity Results ###############

Test                          : ServiceAdministrator
Result                        : OK
AAD Service Admin             : admin@contoso.onmicrosoft.com
Azure Environment             : AzureCloud
Azure Active Directory Tenant : contoso.onmicrosoft.com
Error Details                 : 

############### Azure Identity Validation Summary ###############

Azure Identity Validation found no errors or warnings.

############### Azure Stack Hub Graph Validation Summary ###############

Azure Stack Hub Graph Validation results not available.

############### Azure Stack Hub ADFS Validation Summary ###############

Azure Stack Hub ADFS Validation results not available.

############### AzsReadiness Job Summary ###############

Index             : 0
Operations        : 
StartTime         : 2018/10/22 14:24:16
EndTime           : 2018/10/22 14:24:19
Duration          : 3
PSBoundParameters :
```

## <a name="view-the-report-summary"></a>レポートの概要を表示する

レポートの概要を表示するには、PowerShell コマンドの末尾に `-summary` パラメーターを追加します。 次に例を示します。

```powershell
Read-AzsReadinessReport -ReportPath .\Contoso-AzsReadinessReport.json -summary
```

概要には、結果がない検証が表示されます。この概要には、完了した結果が成功したか失敗したかが示されます。 出力は次の例のようになります。

```shell
Reading All Validation(s) from Report C:\Contoso-AzsReadinessCheckerReport.json

############### Certificate Validation Summary ###############

Certificate Validation found no errors or warnings.

############### Registration Validation Summary ###############

Registration Validation found no errors or warnings.

############### Azure Identity Validation Summary ###############

Azure Identity Validation found no errors or warnings.

############### Azure Stack Hub Graph Validation Summary ###############

Azure Stack Hub Graph Validation results not available.

############### Azure Stack Hub ADFS Validation Summary ###############

Azure Stack Hub ADFS Validation results not available.
```

## <a name="view-a-filtered-report"></a>フィルター処理されたレポートを表示する

1 種類の検証でフィルター処理されたレポートを表示するには、`-ReportSections` パラメーターを使用して、次のいずれかの値を指定します。

- Certificate
- AzureRegistration
- AzureIdentity
- グラフ
- ADFS
- ジョブ
- All  

たとえば、証明書のレポート概要のみを表示するには、次の PowerShell コマンド ラインを使用します。

```powershell
Read-AzsReadinessReport -ReportPath .\Contoso-AzsReadinessReport.json -ReportSections Certificate - Summary
```
