---
title: Azure Stack Hub 用の特権エンドポイント コマンドレット Start-SecretRotation
description: PowerShell Azure Stack Hub の特権エンドポイントのリファレンス - Start-SecretRotation
author: mattbriggs
ms.topic: reference
ms.date: 07/29/2020
ms.author: mabrigg
ms.reviewer: fiseraci
ms.lastreviewed: 04/27/2020
ms.openlocfilehash: 8a3dee9ad26c37626c0f3383b874f149be4fac01
ms.sourcegitcommit: 4af79f4fa2598d57c81e994192c10f8c6be5a445
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/10/2020
ms.locfileid: "89742600"
---
# <a name="start-secretrotation"></a>Start-SecretRotation

## <a name="synopsis"></a>概要
スタンプのシークレット ローテーションをトリガーします。

## <a name="syntax"></a>構文

```
Start-SecretRotation [-PathAccessCredential <Object>] [-ReRun] [-CertificatePassword <Object>] [-Internal]
 [-PfxFilesPath <Object>] [-AsJob]
```

## <a name="description"></a>説明
Azure Stack Hub システムのインフラストラクチャ シークレット用のシークレット ローテーション プロセスを呼び出します。 既定では、外部ネットワーク インフラストラクチャ エンドポイントの証明書のみのローテーションを行います。 詳細については、「[Azure Stack Hub でシークレットをローテーションする](../../operator/azure-stack-rotate-secrets.md)」を参照してください。

## <a name="parameters"></a>パラメーター

### <a name="-internal"></a>-Internal
内部ネットワーク インフラストラクチャ エンドポイントのシークレットをローテーションします。

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

### <a name="-pfxfilespath"></a>-PfxFilesPath
外部証明書のローテーション用に共有される新しい pfx ファイルのパス。
外部証明書のローテーションが必要な場合は必須です。

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

### <a name="-pathaccesscredential"></a>-PathAccessCredential
PfxFilesPath にアクセスするための資格情報。
外部証明書のローテーションが必要な場合は必須です。

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

### <a name="-certificatepassword"></a>-CertificatePassword
指定された新しい pfx ファイルのパスワード。
外部証明書のローテーションが必要な場合は必須です。
最初のデプロイ時に指定された元の pfx パスワードとは異なる場合があります。
正しい CA パスワードを使用して pfx ファイルを再生成します。

使用法:: 

```console
    # Rotates external certificates only
    Start-SecretRotation -PfxFilesPath \<String\> -PathAccessCredential \<PSCredential\> -CertificatePassword \<SecureString\>

    # Rotates internal secrets only
    Start-SecretRotation -Internal  

    # Reruns internal secrets only
    Start-SecretRotation -Internal -ReRun 

    # Reruns external certificates only
    Start-SecretRotation -ReRun
```

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

### <a name="-rerun"></a>-ReRun
 

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
