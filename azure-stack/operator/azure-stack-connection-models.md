---
title: Azure Stack Hub 統合システムの接続モデル
description: Azure Stack Hub 統合システムに対して接続モデルとその他のデプロイ計画を決定します。
author: PatAltimore
ms.topic: conceptual
ms.date: 03/04/2020
ms.author: patricka
ms.reviewer: wfayed
ms.lastreviewed: 02/21/2019
ms.openlocfilehash: 4d3cf019366b1e3b9b42f83948a170427ffbb1bd
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97871106"
---
# <a name="azure-stack-hub-integrated-systems-connection-models"></a>Azure Stack Hub 統合システムの接続モデル
Azure Stack Hub 統合システムの購入に関心をお持ちの場合は、このシステムがご利用のデータセンターにどのように適合するかを判断するために、Azure Stack Hub のデプロイに関する[データセンターの統合のいくつかの考慮事項](azure-stack-datacenter-integration.md)を理解する必要があります。 さらに、Azure Stack Hub をハイブリッド クラウド環境にどのように統合するかを決定する必要があります。 この記事では、Azure 接続モデル、ID ストア オプション、課金モデル オプションなど、これらの主な決定の概要について示します。

統合システムの購入を決定した場合、相手先ブランド供給 (OEM) ハードウェア ベンダーから計画プロセスについて詳しくガイドされます。 OEM ハードウェア ベンダーは、実際のデプロイも行います。

## <a name="choose-an-azure-stack-hub-deployment-connection-model"></a>Azure Stack Hub デプロイの接続モデルを選択する
Azure Stack Hub をインターネット (および Azure) に接続した状態でデプロイするか、接続していない状態でデプロイするかを選択できます。 Azure Stack Hub と Azure 間のハイブリッド シナリオを含めて、Azure Stack Hub を最大限に有効利用するには、Azure に接続した状態でデプロイします。 この選択では、次の図と表に要約されているように、ID ストアに使用可能なオプション (Azure Active Directory または Active Directory フェデレーション サービス) および課金モデル (従量制ベースの課金または容量ベースの課金) を定義します。

![Azure Stack Hub デプロイと課金のシナリオ](media/azure-stack-connection-models/azure-stack-scenarios.png)
  
> [!IMPORTANT]
> これは重要な意思決定のポイントです。 Active Directory フェデレーション サービス (AD FS) または Azure Active Directory (Azure AD) の選択は、デプロイ時に行う必要がある 1 回限りの決定です。 システム全体を再デプロイせずに、後でこれを変更することはできません。  


|オプション|Azure への接続|Azure からの切断|
|-----|:-----:|:-----:|
|Azure AD|![サポートされています](media/azure-stack-connection-models/check.png)| |
|AD FS|![サポートされています](media/azure-stack-connection-models/check.png)|![サポートされています](media/azure-stack-connection-models/check.png)|
|消費ベースの課金|![サポートされています](media/azure-stack-connection-models/check.png)| |
|容量ベースの課金|![サポートされています](media/azure-stack-connection-models/check.png)|![サポートされています](media/azure-stack-connection-models/check.png)|
|ライセンス| マイクロソフト エンタープライズ契約または Cloud Solution Provider | Enterprise Agreement |
|パッチと更新プログラム|更新プログラム パッケージはインターネットから Azure Stack Hub に直接ダウンロード可能 |  必須<br><br>リムーバブル メディアと<br> 別途接続されているデバイスも必要 |
| 登録 | 自動 | 必須<br><br>リムーバブル メディアと<br> 別途接続されているデバイスも必要 |

Azure Stack Hub デプロイに使用される Azure 接続モデルを決定した後、ID ストアと課金方法に関する、接続に依存した追加の決定を行う必要があります。

## <a name="next-steps"></a>次のステップ

[Azure に接続された Azure Stack Hub デプロイの決定](azure-stack-connected-deployment.md)

[Azure から切断された Azure Stack Hub デプロイの決定](azure-stack-disconnected-deployment.md)
