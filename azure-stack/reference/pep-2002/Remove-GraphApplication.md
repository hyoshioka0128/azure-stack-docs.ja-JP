---
title: Azure Stack Hub の Remove-GraphApplication 特権エンドポイント
description: PowerShell Azure Stack の特権エンドポイントのリファレンス - Remove-GraphApplication
author: mattbriggs
ms.topic: reference
ms.date: 04/27/2020
ms.author: mabrigg
ms.reviewer: fiseraci
ms.lastreviewed: 04/27/2020
ms.openlocfilehash: b91d363626bc600027b37a1b9c56e69f6c40c3aa
ms.sourcegitcommit: e9a1dfa871e525f1d6d2b355b4bbc9bae11720d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86486392"
---
# <a name="remove-graphapplication"></a>Remove-GraphApplication

## <a name="synopsis"></a>概要
Remove-GraphApplication は、AD FS 上で AD FS Graph コマンドレットを呼び出すラッパー関数です。

## <a name="syntax"></a>構文

```
Remove-GraphApplication [[-ApplicationIdentifier] <Object>] [-AsJob]
```

## <a name="description"></a>説明
AD FS 上で Remove-GraphApplicationGroup を呼び出して、指定されたアプリケーションを AD FS マシンから削除します。

## <a name="examples"></a>例

### <a name="example-1"></a>例 1
```
Remove-GraphApplication -ApplicationIdentifier "Application-Identifier-123456"
```

## <a name="parameters"></a>パラメーター

### <a name="-applicationidentifier"></a>-ApplicationIdentifier
削除するアプリケーションの識別子

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
