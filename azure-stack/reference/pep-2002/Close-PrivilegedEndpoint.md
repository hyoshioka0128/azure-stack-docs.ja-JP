---
title: Azure Stack Hub の Close-PrivilegedEndpoint 特権エンドポイント
description: PowerShell Azure Stack の特権エンドポイントのリファレンス - Close-PrivilegedEndpoint
author: mattbriggs
ms.topic: reference
ms.date: 04/27/2020
ms.author: mabrigg
ms.reviewer: fiseraci
ms.lastreviewed: 04/27/2020
ms.openlocfilehash: c7c02385d723c34841627850c9f656cd2885de57
ms.sourcegitcommit: e9a1dfa871e525f1d6d2b355b4bbc9bae11720d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86487313"
---
# <a name="close-privilegedendpoint"></a>Close-PrivilegedEndpoint

## <a name="syntax"></a>構文

```
Close-PrivilegedEndpoint [[-TranscriptsPathDestination] <Object>] [[-Credential] <Object>] [-AsJob]
```

## <a name="parameters"></a>パラメーター

### <a name="-credential"></a>-Credential


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

### <a name="-transcriptspathdestination"></a>-TranscriptsPathDestination
 

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

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
