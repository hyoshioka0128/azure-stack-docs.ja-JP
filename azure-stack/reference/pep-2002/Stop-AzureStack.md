---
title: Azure Stack Hub の Stop-AzureStack 特権エンドポイント
description: PowerShell Azure Stack の特権エンドポイントのリファレンス - Stop-AzureStack
author: mattbriggs
ms.topic: reference
ms.date: 04/27/2020
ms.author: mabrigg
ms.reviewer: fiseraci
ms.lastreviewed: 04/27/2020
ms.openlocfilehash: ae57a7845084e251429b2504c741305bfac9f690
ms.sourcegitcommit: e9a1dfa871e525f1d6d2b355b4bbc9bae11720d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86486004"
---
# <a name="stop-azurestack"></a>Stop-AzureStack

## <a name="synopsis"></a>概要
すべての Azure Stack Hub サービスを停止します。

## <a name="syntax"></a>構文

```
Stop-AzureStack [[-TimeoutInSecs] <Object>] [-AsJob]
```

## <a name="description"></a>説明
すべての Azure Stack サービスを停止し、Azure Stack Hub が実行されている物理コンピューターを停止します。

## <a name="parameters"></a>パラメーター

### <a name="-timeoutinsecs"></a>-TimeoutInSecs
実行が停止されるまでの最大時間。

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: False
Position: 1
Default value: 2400
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
