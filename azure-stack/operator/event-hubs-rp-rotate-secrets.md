---
title: Azure Stack Hub リソース プロバイダーで Event Hubs のシークレットをローテーションする方法
description: Azure Stack Hub リソース プロバイダーで Event Hubs のシークレットをローテーションする方法について説明します
author: BryanLa
ms.author: bryanla
ms.service: azure-stack
ms.topic: how-to
ms.date: 10/15/2020
ms.reviewer: jfggdl
ms.lastreviewed: 10/15/2020
ms.openlocfilehash: fe238576814d1aae5d7469e9438f8921f912f0ea
ms.sourcegitcommit: f56a5b287c90b2081ae111385c8b7833931d4059
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/11/2020
ms.locfileid: "97349472"
---
# <a name="how-to-rotate-secrets-for-event-hubs-on-azure-stack-hub"></a>Azure Stack Hub で Event Hubs のシークレットをローテーションする方法

この記事では、Event Hubs リソース プロバイダーで使用されるシークレットをローテーションする方法について説明します。

## <a name="overview-and-prerequisites"></a>概要と前提条件

[!INCLUDE [prereqs](../includes/resource-provider-va-rotate-secrets-prereqs.md)]

## <a name="prepare-a-new-tls-certificate"></a>新しい TLS 証明書を準備する

[!INCLUDE [prepare](../includes/resource-provider-va-rotate-secrets-prepare.md)]

## <a name="rotate-secrets"></a>シークレットの切り替え

[!INCLUDE [rotate](../includes/resource-provider-va-rotate-secrets-rotate.md)]

## <a name="troubleshooting"></a>トラブルシューティング

シークレットのローテーションは、エラーが発生することなく正常に完了する必要があります。 管理者ポータルで次のいずれかの症状が発生した場合は、[サポート リクエストを開いて](azure-stack-manage-basics.md#where-to-get-support)ください。

   - 認証の問題 (Event Hubs リソース プロバイダーへの接続の問題を含む)。
   - リソース プロバイダーをアップグレードできない、または構成パラメーターを編集できない。
   - 使用状況のメトリックが表示されない。
   - 請求書が生成されない。
   - バックアップが実行されない。

## <a name="next-steps"></a>次のステップ

Azure Stack Hub インフラストラクチャのシークレットのローテーションについて詳しくは、「[Azure Stack Hub でシークレットをローテーションする](azure-stack-rotate-secrets.md)」を参照してください。