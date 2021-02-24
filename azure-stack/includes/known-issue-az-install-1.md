---
author: mattbriggs
ms.author: mabrigg
ms.service: azure-stack
ms.topic: include
ms.date: 12/9/2020
ms.reviewer: sijuman
ms.lastreviewed: 12/9/2020
ms.openlocfilehash: 003f6801209d5d5ab1c9c4bd1df0fa47578004ee
ms.sourcegitcommit: 50b362d531c2d35a3a935811fee71252971bd5d8
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/09/2020
ms.locfileid: "96935170"
---
### <a name="error-thrown-when-installing-the-az-modules"></a>Az モジュールをインストールすると、エラーがスローされる

- 適用先:この問題は、2002 以降に適用されます。
- 原因:モジュールをインストールすると、エラーがスローされます。 エラー メッセージは次の文字列で始まります。`Register-PacakgeSource : A parameter cannot be found that matches parameter name. 'PackageManagementProvider'.`。または、エラー メッセージに次のテキストが含まれている可能性があります。`PackageManagement\Install-Package : Cannot convert value "2.0.1-preview" to type "System.Version". Error: "Input string was not in a correct format."`
- 修復: 同じセッションで、次のコマンドレットを実行します。  
    `Install-Module PowershellGet -MinimumumVersion 2.3.0 -Force`  
セッションを閉じ、新しい管理者特権の PowerShell セッションを開始します。
- 発生頻度: 共通