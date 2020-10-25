---
title: Azure Stack Hub の Set-TLSPolicy 特権エンドポイント
description: PowerShell Azure Stack の特権エンドポイントのリファレンス - Set-TLSPolicy
author: mattbriggs
ms.topic: reference
ms.date: 04/27/2020
ms.author: mabrigg
ms.reviewer: fiseraci
ms.lastreviewed: 04/27/2020
ms.openlocfilehash: 8c6d4f69435e3bd81c74225d4051809a7511bfc1
ms.sourcegitcommit: e9a1dfa871e525f1d6d2b355b4bbc9bae11720d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86485884"
---
# <a name="set-tlspolicy"></a>Set-TLSPolicy

## <a name="syntax"></a>構文

```
Set-TLSPolicy [[-Version] <Object>] [-AsJob]
```

## <a name="parameters"></a>パラメーター

### <a name="-version"></a>-Version
 

```yaml
Type: Object
Parameter Sets: (All)
Aliases:
Accepted values: TLS_1.2, TLS_All

Required: False
Position: 0
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
