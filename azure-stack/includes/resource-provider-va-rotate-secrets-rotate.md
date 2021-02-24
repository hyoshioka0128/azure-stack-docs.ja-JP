---
author: BryanLa
ms.author: bryanla
ms.service: azure-stack
ms.topic: include
ms.date: 10/10/2020
ms.reviewer: bryanla
ms.lastreviewed: 10/20/2020
ms.openlocfilehash: 0df920ef0c8063332a290ca5e95e1c01d755e548
ms.sourcegitcommit: 8c745b205ea5a7a82b73b7a9daf1a7880fd1bee9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/24/2020
ms.locfileid: "95517560"
---
最後に、リソース プロバイダーの最新のデプロイ プロパティを決定し、それを使用してシークレットのローテーション プロセスを完了します。

### <a name="determine-deployment-properties"></a>デプロイ プロパティを確認する

リソース プロバイダーは、バージョン管理された製品パッケージとして、お使いの Azure Stack Hub 環境にデプロイされます。 パッケージには、`'<product-id>.<installed-version>'` の形式で一意のパッケージ ID が割り当てられます。 ここで `<product-id>` はリソース プロバイダーを表す一意の文字列で、`<installed-version>` は特定のバージョンを表します。 各パッケージに関連付けられているシークレットは、Azure Stack Hub Key Vault サービスに格納されます。 

管理者特権の PowerShell コンソールを開き、次の手順を実行して、リソース プロバイダーのシークレットのローテーションに必要なプロパティを確認します。

1. オペレーターの資格情報を使用して Azure Stack Hub 環境にサインインします。 PowerShell サインイン スクリプトについては、「[PowerShell を使用して Azure Stack Hub に接続する](../operator/azure-stack-powershell-configure-admin.md)」をご覧ください。 (AzureRM ではなく) PowerShell Az コマンドレットを使用して、エンドポイント URL やディレクトリ テナント名など、すべてのプレースホルダー値を置き換えます。

2. `Get-AzsProductDeployment` コマンドレットを実行して、最新のリソース プロバイダー デプロイの一覧を取得します。 返された `"value"` コレクションには、デプロイされたリソース プロバイダーごとに要素が含まれます。 目的のリソース プロバイダーを見つけて、これらのプロパティの値をメモします。
   - `"name"` - 値の 2 番目のセグメント内にリソース プロバイダーの製品 ID が含まれています。 
   - `"properties"."deployment"."version"` - 現在デプロイされているバージョン番号が含まれています。 

   次の例で、コレクション内の最初の要素の Event Hubs RP デプロイに注目してください。その製品 ID は `"microsoft.eventhub"`、バージョンは `"1.2003.0.0"` です。

   ```powershell
   PS C:\WINDOWS\system32> Get-AzsProductDeployment -AsJson
   VERBOSE: GET https://adminmanagement.myregion.mycompany.com/subscriptions/ze22ca96-z546-zbc6-z566-z35f68799816/providers/Microsoft.Deployment.Admin/locations/global/productDeployments?api-version=2019-01-01 with 0-char payload
   VERBOSE: Received 2656-char response, StatusCode = OK
   {
       "value":  [
                     {
                         "id":  "/subscriptions/ze22ca96-z546-zbc6-z566-z35f68799816/providers/Microsoft.Deployment.Admin/locations/global/productDeployments/microsoft.eventhub",
                         "name":  "global/microsoft.eventhub",
                         "type":  "Microsoft.Deployment.Admin/locations/productDeployments",
                         "properties":  {
                                            "status":  "DeploymentSucceeded",
                                            "subscriptionId":  "b37ae55a-a6c6-4474-ba97-81519412adf5",
                                            "deployment":  {
                                                               "version":  "1.2003.0.0",
                                                               "actionPlanInstanceResourceId":"/subscriptions/ze22ca96-z546-zbc6-z566-z35f68799816/providers/Microsoft.Deployment.Admin/locations/global/actionplans/abcdfcd3-fef0-z1a3-z85d-z6ceb0f31e36",
                                                               "parameters":  {
   
                                                                              }
                                                           },
                                            "lastSuccessfulDeployment":  {
                                                                             "version":  "1.2003.0.0",
                                                                             "actionPlanInstanceResourceId":"/subscriptions/ze22ca96-z546-zbc6-z566-z35f68799816/providers/Microsoft.Deployment.Admin/locations/global/actionplans/abcdfcd3-fef0-z1a3-z85d-z6ceb0f31e36",
                                                                             "parameters":  {
   
                                                                                            }
                                                                         },
                                            "provisioningState":  "Succeeded"
                                        }
                     },
                     {
                     ...
                     }
                 ]
   }
   ```

3. リソース プロバイダーの製品 ID とバージョンを連結して、リソース プロバイダーのパッケージ ID とバージョンを作成します。 たとえば、前の手順の派生した値を使用すると、Event Hubs RP パッケージ ID は `microsoft.eventhub.1.2003.0.0` です。 

4. 前の手順の派生したパッケージ ID を使用して、`Get-AzsProductSecret -PackageId` を実行し、リソース プロバイダーによって使用されているシークレットの種類の一覧を取得します。 返された `value` コレクションで、`"properties"."secretKind"` プロパティの `"Certificate"` の値を含む要素を見つけます。 この要素には、RP の証明書シークレットのプロパティが含まれています。 この証明書のシークレットに割り当てられている名前をメモしておきます。これは、`"properties"` のすぐ上にある `"name"` プロパティの最後のセグメントによって特定されます。 

   次の例では、Event Hubs RP に対して返されたシークレット コレクションに、`aseh-ssl-gateway-pfx` という名前の `"Certificate"` シークレットが含まれています。 

    ```powershell
    PS C:\WINDOWS\system32> Get-AzsProductSecret -PackageId 'microsoft.eventhub.1.2003.0.0' -AsJson
    VERBOSE: GET
    https://adminmanagement.myregion.mycompany.com/subscriptions/ze22ca96-z546-zbc6-z566-z35f68799816/providers/Microsoft.Deployment.Admin/locations/global/productPackages/microsoft.eventhub.1.2003.0.0/secrets?api-version=2019-01-01 with 0-char payload
    VERBOSE: Received 617-char response, StatusCode = OK
    {
        "value":  [
                        {
                            "id":  "/subscriptions/ze22ca96-z546-zbc6-z566-z35f68799816/providers/Microsoft.Deployment.Admin/locations/global/productPackages/microsoft.eventhub.1.2003.0.0/secrets/aseh-ssl-gateway-pfx",
                            "name":  "global/microsoft.eventhub.1.2003.0.0/aseh-ssl-gateway-pfx",
                            "type":  "Microsoft.Deployment.Admin/locations/productPackages/secrets",
                            "properties":  {
                                            "secretKind":  "Certificate",
                                            "description":  "Event Hubs gateway SSL certificate.",
                                            "expiresAfter":  "P730D",
                                            "secretDescriptor":  {
    
                                                                    },
                                            "secretState":  {
                                                                "status":  "Deployed",
                                                                "rotationStatus":  "None",
                                                                "expirationDate":  "2022-03-31T00:16:05.3068718Z"
                                                            },
                                            "provisioningState":  "Succeeded"
                                        }
                        },
                        ...
                    ]
    }
    ```

### <a name="rotate-the-secrets"></a>シークレットのローテーション

1. `Set-AzsProductSecret` コマンドレットを使用して、新しい証明書を Key Vault にインポートします。これは、ローテーション プロセスによって使用されます。 スクリプトを実行する前に、変数のプレースホルダーの値を適宜置き換えます。

   | プレースホルダー | 説明 | 値の例 |
   | ----------- | ----------- | --------------|
   | `<product-id>` | 最新のリソース プロバイダー デプロイの製品 ID。 | `microsoft.eventhub` |
   | `<installed-version>` | 最新のリソース プロバイダー デプロイのバージョン。 | `1.2003.0.0` |
   | `<cert-secret-name>` | 証明書シークレットが格納されている名前。 | `aseh-ssl-gateway-pfx` |
   | `<cert-pfx-file-path>` | 証明書 PFX ファイルへのパス。 | `C:\dir\eh-cert-file.pfx` |
   | `<pfx-password>` | 証明書 .PFX ファイルに割り当てられているパスワード。 | `strong@CertSecret6` |

   ```powershell
   $productId = '<product-id>'
   $packageId = $productId + '.' + '<installed-version>'
   $certSecretName = '<cert-secret-name>' 
   $pfxFilePath = '<cert-pfx-file-path>'
   $pfxPassword = ConvertTo-SecureString '<pfx-password>' -AsPlainText -Force   
   Set-AzsProductSecret -PackageId $packageId -SecretName $certSecretName -PfxFileName $pfxFilePath -PfxPassword $pfxPassword -Force
   ```

2. 最後に、`Invoke-AzsProductRotateSecretsAction` コマンドレットを使用して、内部および外部のシークレットをローテーションします。

   > [!NOTE]
   > ローテーション プロセスを完了するには、約 3.5 時間から 4 時間かかります。

   ```powershell
   Invoke-AzsProductRotateSecretsAction -ProductId $productId
   ```
   
   PowerShell コンソールまたは管理者ポータルでシークレット ローテーションの進行状況を監視するには、Marketplace サービスでリソース プロバイダーを選択します。

   [![secret-rotation-progress](media/resource-provider-va-rotate-secrets-rotate/secret-rotation-in-progress.png)](media/resource-provider-va-rotate-secrets-rotate/secret-rotation-in-progress.png#lightbox)

