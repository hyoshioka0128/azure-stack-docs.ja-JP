---
author: mattbriggs
ms.author: mabrigg
ms.service: azure-stack
ms.topic: include
ms.date: 12/9/2020
ms.reviewer: sijuman
ms.lastreviewed: 12/9/2020
ms.openlocfilehash: e456f17c75b129a3f45c64b9b55e75d21ecb2973
ms.sourcegitcommit: 50b362d531c2d35a3a935811fee71252971bd5d8
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/09/2020
ms.locfileid: "96935167"
---
### <a name="when-installing-az-module-falsely-throws-admin-rights-required-error"></a>Az モジュールをインストールすると、管理者権限が必要であるというエラーが間違ってスローされる

- 適用先:この問題は、2002 以降に適用されます。
- 原因:管理者特権のプロンプトからモジュールをインストールすると、エラーがスローされます。 エラーは `Administrator rights required` と表示されます。
- 修復: セッションを閉じ、新しい管理者特権で PowerShell セッションを開始します。 既存の Az が存在しないことを確認します。 アカウント モジュールがセッションに読み込まれました。
- 発生頻度: 共通