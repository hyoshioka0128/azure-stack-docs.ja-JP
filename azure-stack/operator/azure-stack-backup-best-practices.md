---
title: インフラストラクチャ バックアップ サービスのベスト プラクティス - Azure Stack Hub
description: Azure Stack Hub をデプロイして管理するときのこれらのベスト プラクティスに従って、致命的な障害が発生した場合のデータ損失を軽減できます。
author: PatAltimore
ms.topic: article
ms.date: 02/08/2019
ms.author: patricka
ms.reviewer: hectorl
ms.lastreviewed: 02/08/2019
ms.openlocfilehash: f1208b2a683923e4cf09951d9812e8cbcfcd475f
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97871822"
---
# <a name="infrastructure-backup-service-best-practices"></a>インフラストラクチャ バックアップ サービスのベスト プラクティス

Azure Stack Hub をデプロイして管理するときのこれらのベスト プラクティスに従って、致命的な障害が発生した場合のデータ損失を軽減できます。

ベスト プラクティスは定期的に確認して、操作フローに変更が加えられた場合でもインストールがコンプライアンスに対応していることを確認します。 これらのベスト プラクティスを実装する過程で問題が生じた場合は、Microsoft サポートにお問い合わせください。

## <a name="configuration-best-practices"></a>構成のベスト プラクティス

### <a name="deployment"></a>デプロイ

各 Azure Stack Hub クラウドのデプロイ後に、Infrastructure Backup を有効にします。 Azure Stack Hub PowerShell を使用することで、オペレーターの管理 API エンドポイントにアクセスできるクライアントやサーバーから、バックアップをスケジュールできます。

### <a name="networking"></a>ネットワーク

パスの汎用名前付け規則 (UNC) 文字列には、完全修飾ドメイン名 (FQDN) を使用する必要があります。 名前解決ができない場合は、IP アドレスを使用することができます。 UNC 文字列は、共有ファイルやデバイスといった、リソースの場所を指定します。

### <a name="encryption"></a>暗号化

#### <a name="version-1901-and-newer"></a>バージョン 1901 以降

暗号化証明書を使用して、外部ストレージにエクスポートされるバックアップ データを暗号化します。 証明書はキーの転送にしか使用されないため、自己署名証明書でかまいません。 証明書の作成方法の詳細については、New-SelfSignedCertificate を参照してください。
  
キーは (グローバルの Azure Key Vault 証明書などの) 安全な場所に保存する必要があります。 証明書の CER 形式はデータの暗号化に使用されます。 Azure Stack Hub のクラウド復旧デプロイの間は、バックアップ データを暗号化解除するために PFX 形式を使用する必要があります。

![証明書を安全な場所に保存しました。](media/azure-stack-backup/azure-stack-backup-encryption-store-cert.png)

#### <a name="1811-and-older"></a>1811 およびそれ以前

暗号化キーを使用して、外部ストレージにエクスポートされるバックアップ データを暗号化します。 キーは、[PowerShell で Azure Stack Hub のバックアップを有効にする](azure-stack-backup-enable-backup-powershell.md)一環として生成されます。

キーは (グローバルの Azure Key Vault シークレットなどの) 安全な場所に保存する必要があります。 このキーは、Azure Stack Hub の再デプロイ時に使用します。

![安全な場所にキーを格納する。](media/azure-stack-backup/azure-stack-backup-encryption2.png)

## <a name="operational-best-practices"></a>運用上のベスト プラクティス

### <a name="backups"></a>バックアップ

 - バックアップ ジョブはシステムの稼働中に実行されるため、管理エクスペリエンスやユーザーのアプリにダウンタイムが発生しません。 それなりに負荷がかかっているソリューションでは、バックアップ ジョブに 20 から 40 分かかると想定してください。
 - パッチ、更新プログラム、FRU の操作中は、自動バックアップは開始されません。 スケジュールされたバックアップ ジョブは、既定ではスキップされます。 これらの操作中は、バックアップに対するオンデマンド要求もブロックされます。    
 - OEM 提供の手順を使用して、ネットワーク スイッチを手動でバックアップします。また、Infrastructure Backup コントローラーでコントロール プレーン バックアップ データが格納されている場所と同じバックアップ共有上に、ハードウェア ライフサイクル ホスト (HLH) を格納する必要があります。 スイッチと HLH 構成は、リージョンのフォルダーに格納することを検討してください。 同一リージョンに Azure Stack Hub インスタンスが複数ある場合は、スケール ユニットに属している構成ごとに識別子を使用することを検討してください。

### <a name="folder-names"></a>フォルダー名

 - インフラストラクチャでは、MASBACKUP フォルダーが自動的に作成されます。 これは、Microsoft によって管理される共有です。 MASBACKUP と同じレベルの共有を作成することもできます。 Azure Stack Hub 以外で作成された MASBACKUP 内に、フォルダーやストレージ データを作成することは推奨されません。
 -  フォルダー名に FQDN とリージョンを使用して、バックアップ データを、他のクラウドと区別します。 Azure Stack Hub デプロイとエンドポイントの FQDN は、リージョン パラメーターと外部ドメイン名パラメーターの組み合わせです。 詳細については、「[Azure Stack Hub データセンターの統合 - DNS](azure-stack-integrate-dns.md)」を参照してください。

たとえば、バックアップ共有を、fileserver01.contoso.com 上でホストされている AzSBackups とします。 このファイル共有では、外部ドメイン名を使用する、Azure Stack Hub デプロイごとのフォルダーと、リージョン名を使用するサブフォルダーが存在する可能性があります。

FQDN: contoso.com  
リージョン: nyc

```console
    \\fileserver01.contoso.com\AzSBackups
    \\fileserver01.contoso.com\AzSBackups\contoso.com
    \\fileserver01.contoso.com\AzSBackups\contoso.com\nyc
    \\fileserver01.contoso.com\AzSBackups\contoso.com\nyc\MASBackup
```

MASBackup フォルダーは、Azure Stack Hub で Azure Stack Hub のバックアップ データを格納する場所です。 独自のデータを格納するために、このフォルダーは使用しないでください。 同様に OEM でも、バックアップ データの格納にはこのフォルダーを使用しないでください。

OEM は、各社のコンポーネントのバックアップ データは、リージョン フォルダー配下に格納することを推奨します。 各ネットワーク スイッチ、ハードウェア ライフサイクル ホスト (HLH) などは、それぞれのサブフォルダーに格納されることもあります。 次に例を示します。

```console
    \\fileserver01.contoso.com\AzSBackups\contoso.com\nyc\HLH
    \\fileserver01.contoso.com\AzSBackups\contoso.com\nyc\Switches
    \\fileserver01.contoso.com\AzSBackups\contoso.com\nyc\DeploymentData
    \\fileserver01.contoso.com\AzSBackups\contoso.com\nyc\Registration
```

### <a name="monitoring"></a>監視

システムでサポートされるアラートは、以下のとおりです。

| アラート:                                                   | 説明                                                                                     | Remediation                                                                                                                                |
|---------------------------------------------------------|-------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| ファイル共有の容量が不足しているため、バックアップに失敗しました。 | ファイル共有が容量不足のため、バックアップ コントローラーがバックアップ ファイルをその場所にエクスポートできません。 | ストレージ容量を追加して、バックアップをやり直します。 既存のバックアップを (最も古いものから) 削除して、空き容量を増やします。                    |
| 接続の問題のため、バックアップに失敗しました。             | Azure Stack Hub とファイル共有との間のネットワークに問題があります。                          | ネットワークの問題に対処して、バックアップをやり直します。                                                                                            |
| パスに障害があるため、バックアップに失敗しました。                | ファイル共有のパスを解決できません。                                                          | 別のコンピューターからこの共有をマップして、共有にアクセスできることを確認します。 パスが無効になっている場合は、更新が必要な場合があります。       |
| 認証の問題のため、バックアップに失敗しました。               | 資格情報に問題があるか、認証に影響を与えるネットワークの問題がある可能性があります。    | 別のコンピューターからこの共有をマップして、共有にアクセスできることを確認します。 資格情報が無効になっている場合は、更新が必要な場合があります。 |
| 一般的なエラーのため、バックアップに失敗しました。                    | 断続的な問題のため、要求が失敗した可能性があります。 バックアップをやり直してください。                    | サポートに問い合わせます。                                                                                                                               |

## <a name="next-steps"></a>次のステップ

[インフラストラクチャ バックアップ サービス](azure-stack-backup-reference.md)の参照資料を確認します。

[インフラストラクチャ バックアップ サービス](azure-stack-backup-enable-backup-console.md)を有効にします。
