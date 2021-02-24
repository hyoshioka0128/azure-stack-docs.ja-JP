---
title: Azure Stack Hub の Get-AzureStackStampInformation 特権エンドポイント
description: PowerShell Azure Stack の特権エンドポイントのリファレンス - Get-AzureStackStampInformation
author: mattbriggs
ms.topic: reference
ms.date: 04/27/2020
ms.author: mabrigg
ms.reviewer: fiseraci
ms.lastreviewed: 04/27/2020
ms.openlocfilehash: ec2fb2a7e23cbce90d0ab8ebd75ed33a3e446013
ms.sourcegitcommit: e9a1dfa871e525f1d6d2b355b4bbc9bae11720d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86487211"
---
# <a name="get-azurestackstampinformation"></a>Get-AzureStackStampInformation

## <a name="synopsis"></a>概要
スタンプ情報を取得します。

## <a name="syntax"></a>構文

```
Get-AzureStackStampInformation [-AsJob]
```

## <a name="description"></a>説明
ECE クライアント モジュールをインポートし、ECE サービス上のクラウド ロールに対してアクション プランを開始します。

## <a name="examples"></a>例

### <a name="example-1"></a>例 1
```
Get-AzureStackStampInformation
```

## <a name="parameters"></a>パラメーター

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
