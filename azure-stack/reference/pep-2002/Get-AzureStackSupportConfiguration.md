---
title: Azure Stack Hub の Get-AzureStackSupportConfiguration 特権エンドポイント
description: PowerShell Azure Stack の特権エンドポイントのリファレンス - Get-AzureStackSupportConfiguration
author: mattbriggs
ms.topic: reference
ms.date: 04/27/2020
ms.author: mabrigg
ms.reviewer: fiseraci
ms.lastreviewed: 04/27/2020
ms.openlocfilehash: 97f6bd0fb751885a2a4a4553aba0d7a9329e5458
ms.sourcegitcommit: e9a1dfa871e525f1d6d2b355b4bbc9bae11720d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86487228"
---
# <a name="get-azurestacksupportconfiguration"></a>Get-AzureStackSupportConfiguration

## <a name="synopsis"></a>概要
サポート サービス構成設定を取得します。

## <a name="syntax"></a>構文

```
Get-AzureStackSupportConfiguration [-IncludeRegistrationObjectId] [-AsJob]
```

## <a name="description"></a>説明
サポート サービス構成設定。

## <a name="examples"></a>例

### <a name="example-1"></a>例 1
次の例では、スタンプが登録されている場合、または null の場合に登録の詳細を取得します。

```
PS  C:\> Get-AzureStackSupportConfiguration
```

## <a name="parameters"></a>パラメーター

### <a name="-includeregistrationobjectid"></a>-IncludeRegistrationObjectId
省略可能。
インターネット接続が必要です。
登録 ID のオブジェクト ID を取得します。

```yaml
Type: SwitchParameter
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: False
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

## <a name="notes"></a>Notes
インターネットに接続するにはサポート VM が必要です。

## <a name="next-steps"></a>次のステップ

特権エンドポイントにアクセスして使用する方法については、「[Azure Stack Hub で特権エンドポイントを使用する](../../operator/azure-stack-privileged-endpoint.md)」を参照してください。
