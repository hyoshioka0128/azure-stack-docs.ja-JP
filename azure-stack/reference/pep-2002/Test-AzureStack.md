---
title: Azure Stack Hub の Test-AzureStack 特権エンドポイント
description: PowerShell Azure Stack の特権エンドポイントのリファレンス - Test-AzureStack
author: mattbriggs
ms.topic: reference
ms.date: 04/27/2020
ms.author: mabrigg
ms.reviewer: fiseraci
ms.lastreviewed: 04/27/2020
ms.openlocfilehash: 4b64f3fa38b474fddb930915059f6406d25fd3c8
ms.sourcegitcommit: e9a1dfa871e525f1d6d2b355b4bbc9bae11720d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86485953"
---
# <a name="test-azurestack"></a>Test-AzureStack

## <a name="synopsis"></a>概要
Azure Stack Hub の状態を検証します。

## <a name="syntax"></a>構文

```
Test-AzureStack [-BackupShareCredential <Object>] [-Include <Object>] [-DetailedResults]
 [-ServiceAdminCredentials <Object>] [-List] [-Group <Object>] [-DoNotDeployTenantVm] [-Repair]
 [-Ignore <Object>] [-Confirm] [-TimeoutSeconds <Object>] [-AdminCredential <Object>]
 [-BackupSharePath <Object>] [-WhatIf] [-UseRunners] [-DisableConsoleOutput] [-AsJob]
```

## <a name="description"></a>説明
エラーが検出されない場合、Test-AzureStack からは `$true` が返されます。
エラーが検出されると、`$false` が返されます。

## <a name="examples"></a>例

### <a name="example-1"></a>例 1

オペレーターとユーザーなしのシナリオで Test-AzureStack を実行するには:
```
[azs-ercs01\]: PS\>Test-AzureStack
```

オペレーターとユーザーありのシナリオで Test-AzureStack を実行するには:
```
[azs-ercs01\]: PS\>Test-AzureStack -ServiceAdminCredentials \<PSCredential\>
```

## <a name="parameters"></a>パラメーター

### <a name="-serviceadmincredentials"></a>-ServiceAdminCredentials
 

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

### <a name="-donotdeploytenantvm"></a>-DoNotDeployTenantVm
 

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

### <a name="-admincredential"></a>-AdminCredential
 

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

### <a name="-backupsharepath"></a>-BackupSharePath
 

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

### <a name="-backupsharecredential"></a>-BackupShareCredential
 

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: [PSCredential]::Empty
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-list"></a>-List
 

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

### <a name="-ignore"></a>-Ignore
 

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

### <a name="-include"></a>-Include
 

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

### <a name="-detailedresults"></a>-DetailedResults
 

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

### <a name="-timeoutseconds"></a>-TimeoutSeconds
 

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: 1800
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-group"></a>-Group
 

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: Default
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-repair"></a>-Repair
 

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

### <a name="-userunners"></a>-UseRunners
 

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

### <a name="-disableconsoleoutput"></a>-DisableConsoleOutput
 

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

### <a name="-whatif"></a>-WhatIf
コマンドレットの実行時に発生する内容を示します。
このコマンドレットは実行されません。

```yaml
Type: SwitchParameter
Parameter Sets: (All)
Aliases: wi

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-confirm"></a>-Confirm
コマンドレットの実行前に確認を求めるメッセージが表示されます。

```yaml
Type: SwitchParameter
Parameter Sets: (All)
Aliases: cf

Required: False
Position: Named
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
