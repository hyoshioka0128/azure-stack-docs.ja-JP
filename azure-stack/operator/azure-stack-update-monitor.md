---
title: Azure Stack Hub で PowerShell を使用して更新プログラムを監視する
description: Azure Stack Hub で PowerShell を使用して更新プログラムを監視する方法を説明します。
author: IngridAtMicrosoft
ms.topic: how-to
ms.date: 03/04/2020
ms.author: inhenkel
ms.lastreviewed: 08/23/2019
ms.reviewer: ppacent
ms.openlocfilehash: 315cf773c63e3154ce92472c15d18b5e4bbbe019
ms.sourcegitcommit: e9a1dfa871e525f1d6d2b355b4bbc9bae11720d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86487551"
---
# <a name="monitor-updates-with-powershell-in-azure-stack-hub"></a>Azure Stack Hub で PowerShell を使用して更新プログラムを監視する

Azure Stack Hub 管理エンドポイントを使用して、更新の監視と管理を行うことができます。 それらには、PowerShell でアクセスできます。 Azure Stack Hub で PowerShell を使用してセットアップする手順については、「[PowerShell for Azure Stack Hub をインストールする](azure-stack-powershell-install.md)」を参照してください。

次の PowerShell コマンドレットを使用して更新を管理することもできます。

| コマンドレット | 説明 |
|------------------------------------------------------|-------------|
| [Get-AzsUpdate](/powershell/module/azs.update.admin/get-azsupdate?view=azurestackps-1.8.0) | 使用可能な更新プログラムの一覧を取得します。 |
| [Get-AzsUpdateLocation](/powershell/module/azs.update.admin/get-azsupdatelocation?view=azurestackps-1.8.0)| 更新プログラムの場所の一覧を取得します。 |
| [Get-AzsUpdateRun](/powershell/module/azs.update.admin/get-azsupdaterun?view=azurestackps-1.8.0) | 更新プログラムの実行の一覧を取得します。  |
| [Install-AzsUpdate](/powershell/module/azs.update.admin/install-azsupdate?view=azurestackps-1.8.0) | 更新プログラムの場所にある特定の更新プログラムを適用します。 |
| [Resume-AzsUpdateRun](/powershell/module/azs.update.admin/resume-azsupdaterun?view=azurestackps-1.8.0) | 以前に開始され、失敗した更新実行を再開します。 |

## <a name="get-a-list-of-update-runs"></a>更新実行の一覧を取得する

更新実行の一覧を取得するには、次のコマンドを実行します。

```powershell
Get-AzsUpdateRun -UpdateName Microsoft1.0.180302.1
```

## <a name="resume-a-failed-update-operation"></a>失敗した更新操作を再開する

更新が失敗した場合は、次のコマンドを実行して、失敗した場所から更新実行を再開できます。

```powershell
Get-AzsUpdateRun -Name 5173e9f4-3040-494f-b7a7-738a6331d55c -UpdateName Microsoft1.0.180305.1 | Resume-AzsUpdateRun
```

## <a name="troubleshoot"></a>トラブルシューティング

更新プログラムのトラブルシューティングの詳細については、[Azure Stack のトラブルシューティング](azure-stack-troubleshooting.md)に関するページをご覧ください。

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub での更新プログラム管理](./azure-stack-updates.md)
