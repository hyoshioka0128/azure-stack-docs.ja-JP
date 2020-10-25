---
title: Azure Stack Hub の Set-GraphApplication 特権エンドポイント
description: PowerShell Azure Stack の特権エンドポイントのリファレンス - Set-GraphApplication
author: mattbriggs
ms.topic: reference
ms.date: 04/27/2020
ms.author: mabrigg
ms.reviewer: fiseraci
ms.lastreviewed: 04/27/2020
ms.openlocfilehash: 44d1f75f243bb23e3a1823d96e7a3d877568abb9
ms.sourcegitcommit: e9a1dfa871e525f1d6d2b355b4bbc9bae11720d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86486242"
---
# <a name="set-graphapplication"></a>Set-GraphApplication

## <a name="synopsis"></a>概要
Set-GraphApplication は、AD FS 上で AD FS Graph コマンドレットを呼び出すラッパー関数です。

## <a name="syntax"></a>構文

```
Set-GraphApplication [-ClientCertificates <Object>] [-ClientRedirectUris <Object>]
 [-ApplicationIdentifier <Object>] [-ChangeClientSecret] [-ResetClientSecret] [-AsJob]
```

## <a name="description"></a>説明
AD FS 上で Set-GraphApplicationGroup を呼び出して、AD FS マシン上のアプリケーションを変更します。

## <a name="examples"></a>例

### <a name="example-1"></a>例 1
```
New-GraphApplication -Name $ApplicationName -ClientRedirectUris $redirectUri -ClientCertificates $certificate
```

## <a name="parameters"></a>パラメーター

### <a name="-applicationidentifier"></a>-ApplicationIdentifier
 

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

### <a name="-resetclientsecret"></a>-ResetClientSecret
 

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

### <a name="-changeclientsecret"></a>-ChangeClientSecret
 

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
