---
author: BryanLa
ms.author: bryanla
ms.service: azure-stack
ms.topic: include
ms.date: 10/10/2020
ms.reviewer: bryanla
ms.lastreviewed: 10/20/2020
ms.openlocfilehash: 900d7ac882a37e229bec6dc916653cf55992cccb
ms.sourcegitcommit: 81e2d627c9dc4cc365deb4a0e0674b5ab3a7efbf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/21/2020
ms.locfileid: "93050328"
---
次に、付加価値リソース プロバイダーのエンドポイントを保護するための TLS 証明書を作成または更新します。

1. ご利用のリソース プロバイダーに対し、[証明書更新のための証明書署名要求 (CSR) を生成する](../operator/azure-stack-get-pki-certs.md#generate-certificate-signing-requests-for-certificate-renewal)手順を実行します。 ここでは、Azure Stack Hub 適合性チェッカー ツールを使用して CSR を作成します。 "他の Azure Stack Hub サービスへの証明書要求を生成する" 手順では、必ず、ご利用のリソース プロバイダーに合った適切なコマンドレットを実行してください。 たとえば、Event Hubs には `New-AzsHubEventHubsCertificateSigningRequest` を使用します。 完了したら、生成された .REQ ファイルを証明機関 (CA) に送信して、新しい証明書を申請します。

2. CA から証明書ファイルを受け取ったら、[デプロイまたはローテーションのために証明書を準備する](../operator/azure-stack-prepare-pki-certs.md)手順を実行します。 再度、適合性チェッカー ツールを使用して、CA から返されたファイルを処理します。

3. 最後に、「[Azure Stack Hub PKI 証明書の検証](../operator/azure-stack-validate-pki-certs.md)」の手順を実行します。 もう一度、適合性チェッカー ツールを使用して、新しい証明書の検証テストを実行します。


