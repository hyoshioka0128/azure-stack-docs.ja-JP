---
author: BryanLa
ms.author: bryanla
ms.service: azure-stack
ms.topic: include
ms.date: 10/10/2020
ms.reviewer: bryanla
ms.lastreviewed: 10/20/2020
ms.openlocfilehash: 294f0f5b1f6ac45d53f5d3eebc222b3edba23776
ms.sourcegitcommit: 81e2d627c9dc4cc365deb4a0e0674b5ab3a7efbf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/21/2020
ms.locfileid: "93050329"
---
> [!NOTE]
> 付加価値リソース プロバイダーに対するシークレットのローテーションは、現在、PowerShell でのみサポートされています。 

Azure Stack Hub インフラストラクチャと同様、付加価値リソース プロバイダーには、内部シークレットと外部シークレットの両方が使用されています。 シークレットには、X509 証明書によって管理される暗号化キーやパスワードなど複数の形態があります。 オペレーターは、次の作業を担当します。

- 最新の外部シークレットを提供する (リソース プロバイダー エンドポイントの保護に使用される新しい TLS 証明書など)。
- リソース プロバイダーのシークレットのローテーションを定期的に管理する。

ローテーション プロセスに備えて次の作業を行います。

1. X509 証明書を取得または更新する前に、「[Azure Stack Hub 公開キー インフラストラクチャ (PKI) 証明書の要件](../operator/azure-stack-pki-certs.md#certificate-requirements)」を読んで重要な前提条件の情報を確認します (必要な PFX 形式の詳細を含む)。 特定の付加価値リソース プロバイダーについて、[「オプションの PaaS 証明書」セクション](../operator/azure-stack-pki-certs.md#optional-paas-certificates)に記載された要件も確認してください。

2. 続行する前に、[Azure Stack Hub 用の PowerShell Az モジュールをインストール](../operator/powershell-install-az-module.md)します (まだ済んでいない場合)。 Azure Stack Hub のシークレットのローテーションには、2.0.2-preview 以降のバージョンが必要です。 詳細については、「[Azure Stack Hub での AzureRM から Azure PowerShell Az への移行](../operator/migrate-azurerm-az.md)」を参照してください。
