---
title: Azure Stack Hub での AzureRM から Az への Azure PowerShell スクリプトの移行
description: Azure Stack Hub で AzureRM モジュールから新しい Az モジュールにスクリプトを移行するための手順とツールについて説明します。
author: mattbriggs
ms.author: mabrigg
ms.topic: conceptual
ms.date: 12/2/2020
ms.reviewer: sijuman
ms.lastreviewed: 12/2/2020
ms.openlocfilehash: 8a7d1f270cdb611749e07da310fef484ca8bcd82
ms.sourcegitcommit: 50b362d531c2d35a3a935811fee71252971bd5d8
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/09/2020
ms.locfileid: "96934626"
---
# <a name="migrate-from-azurerm-to-azure-powershell-az-in-azure-stack-hub"></a>Azure Stack Hub での AzureRM から Azure PowerShell Az への移行

Az モジュールには AzureRM との機能パリティがありますが、より短く一貫性のあるコマンドレット名が使用されます。
AzureRM コマンドレット用に記述されたスクリプトは、新しいモジュールで自動的に機能するわけではありません。 移行を容易にするために、Az には AzureRM を使用する既存のスクリプトを実行できるようにするツールが用意されています。 新しいコマンド セットへの移行がないに越したことはありませんが、この記事は、新しいモジュールへの切り替えを開始する際に役立ちます。

AzureRM と Az の間の破壊的変更の完全な一覧については、「[Migration guide for Az 1.0.0 (Az 1.0.0 の移行ガイド)](/powershell/azure/migrate-az-1.0.0)」を参照してください。

## <a name="check-for-installed-versions-of-azurerm"></a>インストールされている AzureRM のバージョンを確認する

移行手順を実行する前に、システムにインストールされている AzureRM のバージョンを確認します。 これにより、スクリプトが最新のリリースで既に実行中であることを確認し、AzureRM をアンインストールせずにコマンドの別名を有効にできるかどうかを把握することができます。

インストールされている AzureRM のバージョンを確認するには、次のコマンドを実行します。

```powershell-interactive
Get-InstalledModule -Name AzureRM -AllVersions
```

## <a name="check-current-scripts-work-with-azurerm"></a>現在のスクリプトが AzureRM で機能することを確認する

これは最も重要な手順です。 既存のスクリプトを実行し、AzureRM の "_最新_" リリース (__2.5.0__) で機能することを確認します。 スクリプトが機能しない場合は、必ず [AzureRM 移行ガイド](/powershell/azure/azurerm/migration-guide.6.0.0)をお読みください。

## <a name="install-the-azure-powershell-az-module"></a>Azure PowerShell Az モジュールをインストールする

最初の手順では、お使いのプラットフォームに Az モジュールをインストールします。 Az をインストールするときは、AzureRM をアンインストールすることをお勧めします。 次の手順では、既存のスクリプトを引き続き実行できるようにする方法と、古いコマンドレット名の互換性を有効にする方法について説明します。

Azure PowerShell Az モジュールをインストールするには、次の手順を実行します。

* **[推奨]** : [AzureRM モジュールをアンインストールする](/powershell/azure/uninstall-az-ps#uninstall-the-azurerm-module)。
  最新のバージョンだけでなく、インストールした "_すべて_" のバージョンの AzureRM を必ず削除します。
* [Az モジュールをインストールする](/powershell/azure/install-az-ps)

## <a name="enable-azurerm-compatibility-aliases"></a>AzureRM と互換性のあるエイリアスを有効にする 

> [!IMPORTANT]
>
> AzureRM の "_すべて_" のバージョンをアンインストールした場合のみ、互換モードを有効にしてください。 AzureRM コマンドレットがまだ使用できる状態で互換モードを有効にすると、予期しない動作が発生する可能性があります。 AzureRM をインストールしたままにする場合はこの手順をスキップしてください。ただし、AzureRM コマンドレットは、Az コマンドレットを呼び出さずに、古いモジュールを使用することに注意してください。

AzureRM をアンインストールして、スクリプトが最新バージョンの AzureRM で機能するようになったら、次の手順で Az モジュールの互換モードを有効にします。 次のコマンドを使用して、互換性を有効にします。

```powershell-interactive
Enable-AzureRmAlias -Scope CurrentUser
```

別名を使用することによって、Az モジュールがインストールされた状態で、古いコマンドレット名を使用できるようになります。 これらの別名は、選択したスコープのユーザー プロファイルに書き込まれます。 ユーザー プロファイルが存在しない場合は、ユーザー プロファイルが作成されます。

> [!WARNING]
>
> このコマンドに別の `-Scope` を使用することができますが、これはお勧めしません。 別名は選択したスコープのユーザー プロファイルに書き込まれるので、できるだけ限られたスコープに対して有効にしてください。 別名をシステム全体で有効にすると、ローカル スコープに AzureRM をインストールしている他のユーザーに問題が生じることもあります。

別名モードを有効にしたら、、スクリプトを再度実行して、スクリプトが引き続き想定どおりに機能することを確認します。 

## <a name="change-module-and-cmdlet-names"></a>モジュール名とコマンドレット名を変更する

一般的に、モジュール名は `AzureRM` および `Azure` が `Az` になるように変更されています。また、コマンドレットについても同様です。
たとえば、`AzureRM.Compute` モジュールは名前が `Az.Compute` に変更されています。 `New-AzureRMVM` は `New-AzVM` になり、`Get-AzureStorageBlob` は `Get-AzStorageBlob` になっています。

この名前付けの変更には例外があるため、注意する必要があります。 `AzureRM` または `Azure` から `Az` への変更以外に、一部のモジュールは、コマンドレットのサフィックスに影響を与えることなく、名前が変更されたか、既存のモジュールにマージされています。 それ以外の場合は、コマンドレットの完全なサフィックスは、新しいモジュール名を反映するように変更されました。

| AzureRM モジュール | Az モジュール | コマンドレットのサフィックスの変更 |
|----------------|-----------|------------------------|
| AzureRM.Profile | Az.Accounts | はい |
| AzureRM.Insights | Az.Monitor | はい |
| AzureRM.Tags | Az.Resources | いいえ |
| AzureRM.UsageAggregates | Az.Billing | いいえ |
| AzureRM.Consumption | Az.Billing | いいえ |

## <a name="summary"></a>まとめ

これらの手順に従うと、既存のスクリプトをすべて更新して、新しいモジュールを使用できるようになります。 これらの手順に関する質問や、移行を困難にしている問題がある場合は、手順を改善できるようにこの記事にコメントを追加してください。

## <a name="breaking-changes-for-az-100"></a>Az 1.0.0 の破壊的変更

このドキュメントでは、AzureRM 6.x と新しい Az モジュール、バージョン 1.x 以降の間の変更について詳しく説明します。 目次は、スクリプトに影響する可能性のあるモジュール固有の変更など、完全な移行パスのすべての段階で役立ちます。

## <a name="general-breaking-changes"></a>重大な変更 - 全般

このセクションでは、Az モジュールの再設計の一部である、一般的な重大な変更について説明します。

### <a name="cmdlet-noun-prefix-changes"></a>コマンドレットの名詞プレフィックスの変更

AzureRM モジュールでは、コマンドレットの名詞プレフィックスとして `AzureRM` または `Azure` が使用されていました。  Az ではコマンドレット名が簡略化され、正規化されています。そのため、すべてのコマンドレットでコマンドレット名詞プレフィックスとして "Az" が使用されます。 次に例を示します。

```powershell  
Get-AzureRMVM
Get-AzureKeyVaultSecret
```

次のように変更されました。

```powershell  
Get-AzVM
Get-AzKeyVaultSecret
```

これらの新しいコマンドレット名に簡単に移行できるように、Az では [Enable-AzureRmAlias](/powershell/module/az.accounts/enable-azurermalias) と [Disable-AzureRmAlias](/powershell/module/az.accounts/disable-azurermalias) の 2 つの新しいコマンドレットが導入されています。  `Enable-AzureRmAlias` では、新しい Az コマンドレット名に対応する、AzureRM の古いコマンドレット名のエイリアスが作成されます。 `Enable-AzureRmAlias` で `-Scope` 引数を使用すると、エイリアスを有効にする場所を選択することができます。

たとえば、AzureRM の次のスクリプトがあるとします。

```powershell  
#Requires -Modules AzureRM.Storage
Get-AzureRmStorageAccount | Get-AzureStorageContainer | Get-AzureStorageBlob
```

`Enable-AzureRmAlias` を使用すると、最小限の変更で実行できます。

```powershell  
#Requires -Modules Az.Storage
Enable-AzureRmAlias -Scope Process
Get-AzureRmStorageAccount | Get-AzureStorageContainer | Get-AzureStorageBlob
```

`Enable-AzureRmAlias -Scope CurrentUser` を実行すると、現在のユーザーが開くすべての PowerShell セッションでエイリアスが有効になるため、このコマンドレットの実行後は、次のようなスクリプトを変更する必要はありません。

```powershell  
Get-AzureRmStorageAccount | Get-AzureStorageContainer | Get-AzureStorageBlob
```

エイリアス コマンドレットの使用方法の詳細については、[Enable-AzureRmAlias リファレンス](/powershell/module/az.accounts/enable-azurermalias)を参照してください。

エイリアスを無効にする準備ができたら、`Disable-AzureRmAlias` により作成されたエイリアスが削除されます。 詳細については、[Disable-AzureRmAlias リファレンス](/powershell/module/az.accounts/disable-azurermalias)を参照してください。

> [!IMPORTANT]
> エイリアスを無効にする場合は、必ず、エイリアスが有効になっている _すべて_ のスコープに対して無効にしてください。

### <a name="module-name-changes"></a>モジュール名の変更

次のモジュールを除き、モジュール名が `AzureRM.*` から `Az.*` に変更されました。

| AzureRM モジュール | Az モジュール |
|----------------|-----------|
| Azure.Storage | Az.Storage |
| Azure.AnalysisServices | Az.AnalysisServices |
| AzureRM.Profile | Az.Accounts |
| AzureRM.Insights | Az.Monitor |
| AzureRM.RecoveryServices.Backup | Az.RecoveryServices |
| AzureRM.RecoveryServices.SiteRecovery | Az.RecoveryServices |
| AzureRM.Tags | Az.Resources |
| AzureRM.MachineLearningCompute | Az.MachineLearning |
| AzureRM.UsageAggregates | Az.Billing |
| AzureRM.Consumption | Az.Billing |

モジュール名が変更されたので、`#Requires` または `Import-Module` を使用して特定のモジュールを読み込むスクリプトは、新しいモジュールを代わりに使用するように変更する必要があります。 コマンドレットのサフィックスが変更されていないモジュールの場合、これは、モジュール名が変更されていても操作スペースを示すサフィックスは _変更されていない_ ことを意味します。

#### <a name="migrating-requires-and-import-module-statements"></a>#Requires ステートメントと Import-Module ステートメントの移行

`#Requires` または `Import-Module` を使用して AzureRM モジュールへの依存関係を宣言しているスクリプトは、新しいモジュール名を使用するように更新する必要があります。 次に例を示します。

```powershell  
#Requires -Module AzureRM.Compute
```

次のように変更する必要があります。

```powershell  
#Requires -Module Az.Compute
```

`Import-Module` の場合:

```powershell  
Import-Module -Name AzureRM.Compute
```

次のように変更する必要があります。

```powershell  
Import-Module -Name Az.Compute
```

### <a name="migrating-fully-qualified-cmdlet-invocations"></a>完全修飾コマンドレットの呼び出しの移行

次のように、モジュール修飾コマンドレットの呼び出しを使用するスクリプトは、

```powershell  
AzureRM.Compute\Get-AzureRmVM
```

新しいモジュール名とコマンドレット名を使用するように変更する必要があります。

```powershell  
Az.Compute\Get-AzVM
```

### <a name="migrating-module-manifest-dependencies"></a>モジュール マニフェスト依存関係の移行

モジュール マニフェスト (.psd1) ファイルを使用して AzureRM モジュールへの依存関係を表すモジュールは、`RequiredModules` セクションのモジュール名を更新する必要があります。

```powershell
RequiredModules = @(@{ModuleName="AzureRM.Profile"; ModuleVersion="5.8.2"})
```

次のように変更する必要があります。

```powershell
RequiredModules = @(@{ModuleName="Az.Accounts"; ModuleVersion="1.0.0"})
```

### <a name="removed-modules"></a>削除されたモジュール

次のモジュールが削除されました。

- `AzureRM.Backup`
- `AzureRM.Compute.ManagedService`
- `AzureRM.Scheduler`

これらのサービス用のツールは積極的にサポートされなくなりました。  都合がつき次第、代替サービスに移行することをお勧めします。

### <a name="windows-powershell-51-and-net-472"></a>Windows PowerShell 5.1 と .NET 4.7.2

Windows 用の PowerShell 5.1 で Az を使用するには、.NET Framework 4.7.2 をインストールする必要があります。 PowerShell Core 6.x 以降を使用する場合、.NET Fremework は不要です。

### <a name="temporary-removal-of-user-login-using-pscredential"></a>PSCredential を使用したユーザー ログインの一時的な削除

.NET Standard の認証フローの変更により、PSCredential を使用したユーザー ログインが一時的に削除されます。 この機能は、Windows 用の PowerShell 5.1 の 2019/1/15 リリースで再導入されます。 詳細については、[こちらの GitHub の問題](https://github.com/Azure/azure-powershell/issues/7430)をご覧ください。

### <a name="default-device-code-login-instead-of-web-browser-prompt"></a>Web ブラウザー プロンプトの代わりにデバイス コード ログインを既定で使用

.NET Standard の認証フローの変更により、対話型ログイン時の既定のログイン フローとしてデバイス ログインが使用されます。 Web ブラウザー ベースのログインは、Windows 用の PowerShell 5.1 の 2019/1/15 リリースで既定のログインとして再導入されます。 その時点で、ユーザーは Switch パラメーターを使用してデバイス ログインを選択できるようになります。

## <a name="module-breaking-changes"></a>モジュールの破壊的変更

このセクションでは、個々のモジュールとコマンドレットの特定の重大な変更について説明します。

### <a name="azapimanagement-previously-azurermapimanagement"></a>Az.ApiManagement (以前の AzureRM.ApiManagement)

- 次のコマンドレットが削除されました。
  - New-AzureRmApiManagementHostnameConfiguration
  - Set-AzureRmApiManagementHostnames
  - Update-AzureRmApiManagementDeployment
  - Import-AzureRmApiManagementHostnameCertificate
  - 代わりに、**Set-AzApiManagement** コマンドレットを使用してこれらのプロパティを設定します
- 次のプロパティが削除されました。
  - `PsApiManagementContext` から、`PsApiManagementHostnameConfiguration` 型の `PortalHostnameConfiguration`、`ProxyHostnameConfiguration`、`ManagementHostnameConfiguration`、`ScmHostnameConfiguration` の各プロパティが削除されました。 代わりに、`PsApiManagementCustomHostNameConfiguration` 型の `PortalCustomHostnameConfiguration`、`ProxyCustomHostnameConfiguration`、`ManagementCustomHostnameConfiguration`、`ScmCustomHostnameConfiguration` を使用します。
  - PsApiManagementContext から `StaticIPs` プロパティが削除されました。 このプロパティは、`PublicIPAddresses` と `PrivateIPAddresses` に分割されました。
  - New-AzureApiManagementVirtualNetwork コマンドレットから、必須の `Location` プロパティが削除されました。

### <a name="azbilling-previously-azurermbilling-azurermconsumption-and-azurermusageaggregates"></a>Az.Billing (以前の AzureRM.Billing、AzureRM.Consumption、および AzureRM.UsageAggregates)

- `Get-AzConsumptionUsageDetail` コマンドレットから、`InvoiceName` パラメーターが削除されました。  スクリプトでは、請求書に他の ID パラメーターを使用する必要があります。

### <a name="azcompute-previously-azurermcompute"></a>Az.Compute (以前の AzureRM.Compute)

- `PSVirtualMachine` および `PSVirtualMachineScaleSet` オブジェクトの `Identity` プロパティから `IdentityIds` が削除されました。スクリプトでは、このフィールドの値を使用して処理を決定することはできなくなりました。
- `PSVirtualMachineScaleSetVM` オブジェクトの `InstanceView` プロパティの型が、`VirtualMachineInstanceView` から `VirtualMachineScaleSetVMInstanceView` に変更されました。
- `UpgradePolicy` プロパティから、`AutoOSUpgradePolicy` および `AutomaticOSUpgrade` プロパティが削除されました。
- `PSSnapshotUpdate` オブジェクトの `Sku` プロパティの型が、`DiskSku` から `SnapshotSku` に変更されました。
- `Add-AzVMDataDisk` コマンドレットから、`VmScaleSetVMParameterSet` が削除されました。スケールセット VM にデータ ディスクを個別に追加することはできなくなりました。

### <a name="azkeyvault-previously-azurermkeyvault"></a>Az.KeyVault (以前の AzureRM.KeyVault)

- `PSKeyVaultKeyAttributes`、`PSKeyVaultKeyIdentityItem`、`PSKeyVaultSecretAttributes` の各オブジェクトから、`PurgeDisabled` プロパティが削除されました。スクリプトでは、```PurgeDisabled``` プロパティを参照して処理を決定することはできなくなりました。

### <a name="azmonitor-previously-azurerminsights"></a>Az.Monitor (以前の AzureRM.Insights)

- 単数形のパラメーター名を優先して、`Set-AzDiagnosticSetting` コマンドレットから複数形のパラメーター名 `Categories` と `Timegrains` が削除されました。次のコマンドレットを使用するスクリプトは
  ```powershell  
  Set-AzureRmDiagnosticSetting -Timegrains PT1M -Categories Category1, Category2
  ```

  次のように変更する必要があります。
  ```powershell  
  Set-AzDiagnosticSetting -Timegrain PT1M -Category Category1, Category2
  ```

### <a name="aznetwork-previously-azurermnetwork"></a>Az.Network (以前の AzureRM.Network)

- `Get-AzServiceEndpointPolicyDefinition` コマンドレットから、非推奨の `ResourceId` パラメーターが削除されました。
- `PSVirtualNetwork` オブジェクトから、非推奨の `EnableVmProtection` プロパティが削除されました。
- 非推奨の `Set-AzVirtualNetworkGatewayVpnClientConfig` コマンドレットが削除されました。

スクリプトでは、これらのフィールドの値に基づいて処理を決定することはできなくなりました。

### <a name="azresources-previously-azurermresources"></a>Az.Resources (以前の AzureRM.Resources)

- `New/Set-AzPolicyAssignment` コマンドレットから、`Sku` パラメーターが削除されました。
- `New-AzADServicePrincipal` および `New-AzADSpCredential` コマンドレットから、`Password` パラメーターが削除されました。パスワードは自動的に生成されます。次のようにパスワードを指定しているスクリプトは

  ```powershell  
  New-AzAdSpCredential -ObjectId 1f99cf81-0146-4f4e-beae-2007d0668476 -Password $secPassword
  ```

  出力からパスワードを取得するように変更する必要があります。

  ```powershell  
  $credential = New-AzAdSpCredential -ObjectId 1f99cf81-0146-4f4e-beae-2007d0668476
  $secPassword = $credential.Secret
  ```


### <a name="azstorage-previously-azurestorage-and-azurermstorage"></a>Az.Storage (以前の Azure.Storage and AzureRM.Storage)

- ストレージ アカウント名のみを使用した Oauth ストレージ コンテキストの作成をサポートするために、既定のパラメーター セットが `OAuthParameterSet` に変更されました。
  - 例: `$ctx = New-AzureStorageContext -StorageAccountName $accountName`
- `Get-AzStorageUsage` コマンドレットで `Location` パラメーターが必須になりました。
- Storage API のメソッドで、同期 API 呼び出しの代わりに、タスク ベースの非同期パターン (TAP) が使用されるようになりました。 次の例は、新しい非同期コマンドを示しています。

#### <a name="blob-snapshot"></a>BLOB スナップショット

AzureRM:

```powershell  
$b = Get-AzureStorageBlob -Container $containerName -Blob $blobName -Context $ctx
$b.ICloudBlob.Snapshot()
```

Az:

```powershell  
$b = Get-AzStorageBlob -Container $containerName -Blob $blobName -Context $ctx
$task = $b.ICloudBlob.SnapshotAsync()
$task.Wait()
$snapshot = $task.Result
```

#### <a name="share-snapshot"></a>スナップショットの共有

AzureRM:

```powershell  
$Share = Get-AzureStorageShare -Name $containerName -Context $ctx
$snapshot = $Share.Snapshot()
```

Az:

```powershell  
$Share = Get-AzStorageShare -Name $containerName -Context $ctx
$task = $Share.SnapshotAsync()
$task.Wait()
$snapshot = $task.Result
```

#### <a name="undelete-soft-deleted-blob"></a>論理的に削除された BLOB の削除の取り消し

AzureRM:

```powershell  
$b = Get-AzureStorageBlob -Container $containerName -Blob $blobName -IncludeDeleted -Context $ctx
$b.ICloudBlob.Undelete()
```

Az:

```powershell  
$b = Get-AzStorageBlob -Container $containerName -Blob $blobName -IncludeDeleted -Context $ctx
$task = $b.ICloudBlob.UndeleteAsync()
$task.Wait()
```

#### <a name="set-blob-tier"></a>BLOB レベルの設定

AzureRM:

```powershell  
$blockBlob = Get-AzureStorageBlob -Container $containerName -Blob $blockBlobName -Context $ctx
$blockBlob.ICloudBlob.SetStandardBlobTier("hot")

$pageBlob = Get-AzureStorageBlob -Container $containerName -Blob $pageBlobName -Context $ctx
$pageBlob.ICloudBlob.SetPremiumBlobTier("P4")
```

Az:

```powershell  
$blockBlob = Get-AzStorageBlob -Container $containerName -Blob $blockBlobName -Context $ctx
$task = $blockBlob.ICloudBlob.SetStandardBlobTierAsync("hot")
$task.Wait()

$pageBlob = Get-AzStorageBlob -Container $containerName -Blob $pageBlobName -Context $ctx
$task = $pageBlob.ICloudBlob.SetPremiumBlobTierAsync("P4")
$task.Wait()
```

### <a name="azwebsites-previously-azurermwebsites"></a>Az.Websites (以前の AzureRM.Websites)

- `PSAppServicePlan`、`PSCertificate`、`PSCloningInfo`、`PSSite` の各オブジェクトから非推奨のプロパティが削除されました。

## <a name="next-steps"></a>次のステップ

- Azure Stack Hub での PowerShell の詳細については、「[Azure Stack Hub 内の PowerShell の概要](../user/azure-stack-powershell-overview.md)」を参照してください
- PowerShell Az モジュールをインストールします。[Azure Stack Hub 用の PowerShell Az モジュールのインストール](powershell-install-az-module.md)に関する記事を参照してください
