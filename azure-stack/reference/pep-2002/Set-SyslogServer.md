---
title: Set-SyslogServer
description: PowerShell Azure Stack の特権エンドポイントのリファレンス - Close-PrivilegedEndpoint
author: mattbriggs
ms.topic: reference
ms.date: 04/27/2020
ms.author: mabrigg
ms.reviewer: fiseraci
ms.lastreviewed: 04/27/2020
ms.openlocfilehash: 328862a4d6d598781a88597ea1d03fbaf50a4885
ms.sourcegitcommit: 4af79f4fa2598d57c81e994192c10f8c6be5a445
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/10/2020
ms.locfileid: "89742691"
---
# <a name="set-syslogserver"></a>Set-SyslogServer

## <a name="synopsis"></a>概要
syslog サーバー エンドポイントを設定します。

## <a name="syntax"></a>構文

```
Set-SyslogServer [-Remove] [[-ServerName] <Object>] [-NoEncryption] [-SkipCertificateCheck] [-UseUDP]
 [-SkipCNCheck] [[-ServerPort] <Object>] [-AsJob]
```


## <a name="examples"></a>例

### <a name="example-1"></a>例 1

```
Set-SyslogServer -ServerName <FQDN or IP address of Syslog server>
```

### <a name="example-2"></a>例 2
```
-NoEncryption
```

### <a name="example-3"></a>例 3
```
-SkipCertificateCheck
```

### <a name="example-4"></a>例 4
```
-SkipCNCheck
```

### <a name="example-5"></a>例 5
```
-UseUDP
```

### <a name="example-6"></a>例 6
```
Set-SyslogServer -Remove
```

## <a name="parameters"></a>パラメーター

### <a name="-servername"></a>-ServerName
syslog サーバーの FQDN または IPv4 アドレス。
他のパラメーターを指定しない場合、既定のプロトコルは暗号化、証明書の検証、証明書名の確認が有効な TCP です。

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

### <a name="-serverport"></a>-ServerPort
Syslog サーバーのサーバー ポート。

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: False
Position: 2
Default value: 0
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-noencryption"></a>-NoEncryption
TCP トラフィックの暗号化を無効にする場合は、NoEncryption を指定します。

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

### <a name="-skipcncheck"></a>-SkipCNCheck
証明書名の確認をスキップする場合は、SkipCNCheck を指定します。

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

### <a name="-skipcertificatecheck"></a>-SkipCertificateCheck
Syslog サーバー証明書の確認をスキップする場合は、SkipCertificateCheck を指定します。

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

### <a name="-useudp"></a>-UseUDP
Syslog サーバーで UDP プロトコルを使用する場合は、UseUDP を指定します。

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

### <a name="-remove"></a>-Remove
 

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
