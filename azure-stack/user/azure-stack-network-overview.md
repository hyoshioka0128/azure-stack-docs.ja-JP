---
title: Azure Stack Hub ネットワークの概要
description: Azure Stack Hub ネットワークについて
author: mattbriggs
ms.topic: conceptual
ms.date: 2/1/2021
ms.author: mabrigg
ms.reviewer: scottnap
ms.lastreviewed: 01/14/2020
ms.openlocfilehash: ca8c93e983290e925d3af38bef0799d11c4a745a
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99247610"
---
# <a name="introduction-to-azure-stack-hub-networking"></a>Azure Stack Hub ネットワークの概要

Azure Stack Hub には、単独でまたは組み合わせて使用できる、さまざまな種類のネットワーク機能が用意されています。

- **Azure Stack Hub リソースの間の接続**  
    クラウド内の安全でプライベートな仮想ネットワークで Azure のリソースを接続します。
- **インターネット接続**  
    インターネットを介して双方向で Azure Stack Hub リソースと通信します。
- **オンプレミスの接続**  
    インターネット上の仮想プライベート ネットワーク (VPN) を介して、または Azure Stack Hub への専用接続を介して、オンプレミスのネットワークを Azure Stack Hub リソースに接続します。 
    > [!IMPORTANT]
    > オンプレミスのリソースにアクセスするには、VPN またはパブリック IP 接続を作成する必要があります。
- **負荷分散とトラフィックの方向**  
    同じ場所のサーバー間でトラフィックを負荷分散したり、異なる場所のサーバーにトラフィックを誘導したりします。
- **Security**  
    ネットワーク サブネット間や個々の VM 間のネットワーク トラフィックをフィルタリングします。
- **ルーティング**  
    Azure Stack Hub リソースとオンプレミス リソースとの間で既定のルーティングまたは完全制御のルーティングを使用します。
- **管理の容易性**  
    Azure Stack Hub のネットワーク リソースを監視したり管理したりします。
- **デプロイ ツールと構成ツール**  
    Web ベースのポータルまたはクロスプラットフォームのコマンド ライン ツールを使って、ネットワーク リソースのデプロイと構成を行います。


## <a name="next-steps"></a>次の手順

* [Azure Stack Hub ネットワークに関する考慮事項](azure-stack-network-differences.md)