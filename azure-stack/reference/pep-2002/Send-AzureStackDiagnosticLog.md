---
title: Azure Stack Hub の Send-AzureStackDiagnosticLog 特権エンドポイント
description: PowerShell Azure Stack の特権エンドポイントのリファレンス - Send-AzureStackDiagnosticLog
author: mattbriggs
ms.topic: reference
ms.date: 04/27/2020
ms.author: mabrigg
ms.reviewer: fiseraci
ms.lastreviewed: 04/27/2020
ms.openlocfilehash: 2b62eb35f096ab5b38ee37ff776435bb90b7c210
ms.sourcegitcommit: e9a1dfa871e525f1d6d2b355b4bbc9bae11720d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86486191"
---
# <a name="send-azurestackdiagnosticlog"></a>Send-AzureStackDiagnosticLog

## <a name="synopsis"></a>概要
Azure Stack Hub 診断ログを Microsoft に送信します。

## <a name="syntax"></a>構文

```
Send-AzureStackDiagnosticLog [[-FromDate] <Object>] [[-FilterByResourceProvider] <Object>]
 [[-FilterByRole] <Object>] [[-ToDate] <Object>] [-AsJob]
```

## <a name="description"></a>説明
スタンプの診断ログをアップロードするように Support Bridge Service に要求します。

## <a name="examples"></a>例

### <a name="example-1"></a>例 1

次の例では、過去 4 時間のログを Microsoft に送信します。

```
PS  C:\> Send-AzureStackDiagnosticLog
```

### <a name="example-2"></a>例 2
次の例は、過去 4 時間のサポート サービスと WAS ログを Microsoft に送信します。
```
PS  C:\> Send-AzureStackDiagnosticLog -FilterByRole SupportBridgeController,WAS
```

## <a name="parameters"></a>パラメーター

### <a name="-filterbyrole"></a>-FilterByRole
省略可能。
インフラストラクチャ ロールの種類でフィルター処理します。

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: False
Position: 1
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-filterbyresourceprovider"></a>-FilterByResourceProvider
省略可能。
付加価値リソース プロバイダーの種類でフィルターします。

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: False
Position: 2
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-fromdate"></a>-FromDate
省略可能。
収集するログに使用する開始時刻。

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: False
Position: 3
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-todate"></a>-ToDate
省略可能。
収集するログに使用する終了時刻。

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: False
Position: 4
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-asjob"></a>-AsJob


```yaml
Type: SwitchParameter
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

## <a name="next-steps"></a>次のステップ

特権エンドポイントにアクセスして使用する方法については、「[Azure Stack Hub で特権エンドポイントを使用する](../../operator/azure-stack-privileged-endpoint.md)」を参照してください。
