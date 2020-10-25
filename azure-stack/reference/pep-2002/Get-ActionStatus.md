---
title: Azure Stack Hub の Get-ActionStatusprivileged 特権エンドポイント
description: PowerShell Azure Stack の特権エンドポイントのリファレンス - Get-ActionStatus
author: mattbriggs
ms.topic: reference
ms.date: 04/27/2020
ms.author: mabrigg
ms.reviewer: fiseraci
ms.lastreviewed: 04/27/2020
ms.openlocfilehash: 35d857ff8f118e587246f6d10d7f1453b9dec193
ms.sourcegitcommit: e9a1dfa871e525f1d6d2b355b4bbc9bae11720d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86487267"
---
# <a name="get-actionstatus"></a>Get-ActionStatus

## <a name="synopsis"></a>概要
指定された関数名を持つ操作の最新のアクションの状態を取得します。

## <a name="syntax"></a>構文

```
Get-ActionStatus [[-ActionType] <Object>] [[-FunctionName] <Object>] [-AsJob]
```

## <a name="description"></a>説明
指定された関数名を持つ操作の最新のアクションの状態を取得します。

## <a name="examples"></a>例

### <a name="example-1"></a>例 1
```
Get-ActionStatus -FunctionName "Start-SecretRotation"
```

## <a name="parameters"></a>パラメーター

### <a name="-functionname"></a>-FunctionName
状態が照会されている関数の名前。

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

### <a name="-actiontype"></a>-ActionType
操作アクションの種類の名前。

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
