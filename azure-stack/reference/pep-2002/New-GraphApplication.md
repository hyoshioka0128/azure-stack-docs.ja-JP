---
title: Azure Stack Hub の New-GraphApplication 特権エンドポイント
description: PowerShell Azure Stack の特権エンドポイントのリファレンス - New-GraphApplication
author: mattbriggs
ms.topic: reference
ms.date: 04/27/2020
ms.author: mabrigg
ms.reviewer: fiseraci
ms.lastreviewed: 04/27/2020
ms.openlocfilehash: b0a9aeb2e7ec1035f24d89409f5fa23ef13500fb
ms.sourcegitcommit: e9a1dfa871e525f1d6d2b355b4bbc9bae11720d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86486616"
---
# <a name="new-graphapplication"></a>New-GraphApplication

## <a name="synopsis"></a>概要
New-GraphApplication は、AD FS 上で AD FS Graph コマンドレットを呼び出すラッパー関数です。

## <a name="syntax"></a>構文

```
New-GraphApplication [-ClientCertificates <Object>] [-Name <Object>] [-ClientRedirectUris <Object>]
 [-GenerateClientSecret] [-AsJob]
```

## <a name="description"></a>説明
AD FS 上で New-GraphApplicationGroup を呼び出して、AD FS マシンに新しいアプリケーションを追加します。

## <a name="examples"></a>例

### <a name="example-1"></a>例 1
```
New-GraphApplication -Name $ApplicationName -ClientRedirectUris $redirectUri -ClientCertificates $certificate
```

## <a name="parameters"></a>パラメーター

### <a name="-name"></a>-Name
最大長が 50 文字のアプリケーションの名前。これは関数によって `Azurestack-$Name-$({guid}::{NewGuid}())` に変更され、返されます。

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-clientredirecturis"></a>-ClientRedirectUris
AD FS で新しいアプリケーションを作成するために使用されるリダイレクト URI。

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-clientcertificates"></a>-ClientCertificates
AD FS で新しいアプリケーションを作成するために使用される証明書。

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-generateclientsecret"></a>-GenerateClientSecret
 

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

## <a name="next-steps"></a>次のステップ

特権エンドポイントにアクセスして使用する方法については、「[Azure Stack Hub で特権エンドポイントを使用する](../../operator/azure-stack-privileged-endpoint.md)」を参照してください。
