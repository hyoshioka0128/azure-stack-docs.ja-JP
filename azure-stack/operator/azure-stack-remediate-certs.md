---
title: PKI 証明書に関する一般的な問題を修正する
titleSuffix: Azure Stack Hub
description: Azure Stack Hub 適合性チェッカー ツールを使用して Azure Stack Hub PKI 証明書に関する一般的な問題を修します。
author: IngridAtMicrosoft
ms.topic: how-to
ms.date: 03/04/2020
ms.author: inhenkel
ms.reviewer: unknown
ms.lastreviewed: 11/19/2019
ms.openlocfilehash: c7f17c603a6b54474db4036953f0fbd755d496cf
ms.sourcegitcommit: e72145ebb5eac17a47ba1c9119fd31de545fdace
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/21/2020
ms.locfileid: "88724848"
---
# <a name="fix-common-issues-with-azure-stack-hub-pki-certificates"></a>Azure Stack Hub PKI 証明書に関する一般的な問題を修復する

この記事の情報は、Azure Stack Hub PKI 証明書の一般的な問題について理解し、解決するうえで役立ちます。 Azure Stack Hub 適合性チェッカー ツールを使用して [Azure Stack Hub PKI 証明書を検証](azure-stack-validate-pki-certs.md)すると、問題を見つけることができます。 このツールは、証明書が Azure Stack Hub デプロイと Azure Stack Hub シークレット ローテーションの PKI 要件を満たしていることを確認し、その結果のログを [report.json ファイル](azure-stack-validation-report.md)に出力します。  

## <a name="pfx-encryption"></a>PFX の暗号化

**問題** - PFX 暗号化が TripleDES-SHA1 ではない。

**解決策** - **TripleDES-SHA1** 暗号化を使用して PFX ファイルをエクスポートします。 これは、証明書スナップインからエクスポートするか `Export-PFXCertificate` を使用するときの、すべての Windows 10 クライアントの既定の暗号化の設定です。

## <a name="read-pfx"></a>PFX の読み取り

**警告** - 証明書の個人情報がパスワードのみで保護されています。  

**解決策** - **[証明書のプライバシーを有効にする]** オプション設定を使用して PFX ファイルをエクスポートします。  

**問題** - PFX ファイルが無効です。  

**解決策** - 「[デプロイ用の Azure Stack Hub PKI 証明書の準備](azure-stack-prepare-pki-certs.md)」の手順を使用して、証明書を再度エクスポートします。

## <a name="signature-algorithm"></a>署名アルゴリズム

**問題** - 署名アルゴリズムが SHA1 です。

**解決策** - 「Azure Stack Hub 証明書署名要求の生成」の手順を使用して、署名アルゴリズム SHA256 で証明書署名要求 (CSR) を再生成します。 その後、CSR を証明機関に再送信して、証明書を再発行します。

## <a name="private-key"></a>秘密キー

**問題** - 秘密キーがないか、秘密キーにローカル コンピューター属性が含まれていません。  

**解決策** - 「[デプロイ用の Azure Stack Hub PKI 証明書の準備](azure-stack-prepare-pki-certs.md#prepare-certificates-azure-stack-readiness-checker)」の手順を使用して、CSR を生成したコンピューターから証明書を再度エクスポートします。 この手順には、ローカル コンピューターの証明書ストアからのエクスポートが含まれます。

## <a name="certificate-chain"></a>証明書チェーン

**問題** - 証明書チェーンが完全ではありません。  

**解決策** - 証明書には、完全な証明書チェーンが含まれている必要があります。 「[デプロイ用の Azure Stack Hub PKI 証明書の準備](azure-stack-prepare-pki-certs.md#prepare-certificates-azure-stack-readiness-checker)」の手順を使用して証明書を再度エクスポートし、 **[証明のパスにある証明書を可能であればすべて含む]** オプションを選択します。

## <a name="dns-names"></a>DNS 名

**問題** - 証明書の **DNSNameList** に、Azure Stack Hub サービス エンドポイント名または有効なワイルドカード一致が含まれていません。 ワイルドカード一致は、DNS 名の左端の名前空間に対してのみ有効です。 たとえば、`*.region.domain.com` は `portal.region.domain.com` のみで有効であり、`*.table.region.domain.com` では有効ではありません。

**解決策** - 「Azure Stack Hub 証明書署名要求の生成」の手順を使用して、Azure Stack Hub エンドポイントをサポートする正しい DNS 名で CSR を再生成します。 CSR を証明機関に再送信します。 その後、「[デプロイ用の Azure Stack Hub PKI 証明書の準備](azure-stack-prepare-pki-certs.md#prepare-certificates-azure-stack-readiness-checker)」の手順に従って、CSR を生成したマシンから、証明書をエクスポートします。  

## <a name="key-usage"></a>キー使用法

**問題** - キー使用法に、デジタル署名またはキーの暗号化がありません。または拡張キー使用法に、サーバー認証またはクライアント認証がありません。  

**解決策** - 「[Azure Stack Hub 証明書署名要求の生成](azure-stack-get-pki-certs.md)」の手順を使用して、正しいキー使用法の属性で CSR を再生成します。 CSR を証明機関に再送信して、証明書テンプレートによって要求のキー使用法が上書きされていないことを確認します。

## <a name="key-size"></a>キー サイズ

**問題** - キー サイズが 2,048 未満です。

**解決策** - 「[Azure Stack Hub 証明書署名要求の生成](azure-stack-get-pki-certs.md)」の手順を使用して、正しいキーの長さ (2,048) で CSR を再生成し、CSR を証明機関に再送信します。

## <a name="chain-order"></a>チェーンの順序

**問題** - 証明書チェーンの順序が正しくありません。  

**解決策** - 「[デプロイ用の Azure Stack Hub PKI 証明書の準備](azure-stack-prepare-pki-certs.md#prepare-certificates-azure-stack-readiness-checker)」の手順を使用して証明書を再度エクスポートし、 **[証明のパスにある証明書を可能であればすべて含む]** オプションを選択します。 リーフ証明書のみがエクスポート用に選択されていることを確認します。

## <a name="other-certificates"></a>他の証明書

**問題** - PFX パッケージに、リーフ証明書または証明書チェーンの一部ではない証明書が含まれています。  

**解決策** - 「[デプロイ用の Azure Stack Hub PKI 証明書の準備](azure-stack-prepare-pki-certs.md#prepare-certificates-azure-stack-readiness-checker)」の手順を使用して証明書を再度エクスポートし、 **[証明のパスにある証明書を可能であればすべて含む]** オプションを選択します。 リーフ証明書のみがエクスポート用に選択されていることを確認します。

## <a name="fix-common-packaging-issues"></a>パッケージに関する一般的な問題の修正

**AzsReadinessChecker** ツールには、**Repair-AzsPfxCertificate** というヘルパー コマンドレットが含まれています。これを使用すると、PFX ファイルをインポートしてからエクスポートし、パッケージに関する次のような一般的な問題を修正できます。

- **PFX 暗号化**が TripleDES-SHA1 ではない。
- **秘密キー**にローカル コンピューター属性がない。
- "**証明書チェーン**" が完全ではないか、間違っている PFX パッケージに証明書チェーンが含まれない場合、ローカル コンピューターに証明書チェーンを含める必要があります。
- **他の証明書**

新しい CSR を生成して証明書を再発行する必要がある場合は、**Repair-AzsPfxCertificate** は役に立ちません。

### <a name="prerequisites"></a>前提条件

ツールが実行されるコンピューターで、次の前提条件を満たす必要があります。

- インターネットに接続された Windows 10 または Windows Server 2016。
- PowerShell 5.1 以降。 バージョンを確認するには、次の PowerShell コマンドレットを実行し、"**メジャー**" バージョンと "**マイナー**" バージョンを調べます。

   ```powershell
   $PSVersionTable.PSVersion
   ```

- [PowerShell for Azure Stack Hub](azure-stack-powershell-install.md) を構成します。
- 最新バージョンの [Azure Stack Hub 適合性チェッカー](https://aka.ms/AzsReadinessChecker) ツールをダウンロードします。

### <a name="import-and-export-an-existing-pfx-file"></a>既存の PFX ファイルのインポートとエクスポート

1. 前提条件を満たしているコンピューターで、管理者特権の PowerShell プロンプトを開き、次のコマンドを実行して、Azure Stack Hub 適合性チェッカーをインストールします。

   ```powershell
   Install-Module Microsoft.AzureStack.ReadinessChecker -Force
   ```

2. PowerShell プロンプトで次のコマンドレットを実行して、PFX パスワードを設定します。 `PFXpassword` を実際のパスワードで置き換えます。

   ```powershell
   $password = Read-Host -Prompt PFXpassword -AsSecureString
   ```

3. PowerShell プロンプトから次のコマンドを実行して、新しい PFX ファイルをエクスポートします。

   - `-PfxPath` には、操作する PFX ファイルへのパスを指定します。 次の例では、パスは `.\certificates\ssl.pfx` です。
   - `-ExportPFXPath` には、エクスポートする PFX ファイルの場所と名前を指定します。 次の例では、パスは `.\certificates\ssl_new.pfx` です。

   ```powershell
   Repair-AzsPfxCertificate -PfxPassword $password -PfxPath .\certificates\ssl.pfx -ExportPFXPath .\certificates\ssl_new.pfx
   ```  

4. ツールが完了したら、成功したかどうかを出力で確認します。

   ```shell
   Repair-AzsPfxCertificate v1.1809.1005.1 started.
   Starting Azure Stack Hub Certificate Import/Export
   Importing PFX .\certificates\ssl.pfx into Local Machine Store
   Exporting certificate to .\certificates\ssl_new.pfx
   Export complete. Removing certificate from the local machine store.
   Removal complete.
   Log location (contains PII): C:\Users\username\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessChecker.log
   Repair-AzsPfxCertificate Completed
   ```

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub のセキュリティについて詳しく学習する](azure-stack-rotate-secrets.md)
