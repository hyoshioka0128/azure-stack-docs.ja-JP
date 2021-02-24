---
title: Azure Stack で Commvault Marketplace アイテムを提供する | Microsoft Docs
description: Azure Stack Marketplace から Commvault をデプロイする
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: tzl
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/14/2019
ms.author: sethm
ms.reviewer: hectorl
ms.lastreviewed: 12/14/2019
ms.openlocfilehash: c0220a7e1d5f6de6ba00b89ef64c7496e482992f
ms.sourcegitcommit: 50b362d531c2d35a3a935811fee71252971bd5d8
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/09/2020
ms.locfileid: "96941236"
---
# <a name="offer-commvault-marketplace-item-in-azure-stack"></a>Azure Stack で Commvault Marketplace アイテムを提供する

*適用対象:Modular Data Center、Azure Stack Hub ラグド*

Commvault は、Azure Stack Hub 上の次の種類のリソースを対象にバックアップと復元をサポートします。

- VM レベルのバックアップ
  - IaaS VM
  - 非管理対象ディスク
  - マネージド ディスク
  - 詳細については、「[仮想マシンのバックアップ](https://documentation.commvault.com/commvault/v11/article?p=86503.htm)」を参照してください。

- ストレージ アカウントのバックアップ
  - BLOB
  - 詳細については、「[Azure Blob Storage の概要](https://documentation.commvault.com/commvault/v11/article?p=30063.htm)」を参照してください。

- エージェントベースのバックアップ
  - ゲスト OS -- Windows および Linux
  - アプリケーション -- SQL、MySQL
  - 詳細については、「[バックアップ エージェント](https://documentation.commvault.com/commvault/v11/article?p=14333.htm)」を参照してください。

Commvault を外部のマシンにデプロイし、Azure Stack Hub 上のリソースをリモートから保護することができます。 加えて、Azure Stack Hub 上の仮想アプライアンスとして Commvault をデプロイすることもできます。 Commvault が提供する詳細なガイダンスは、[Azure Stack Hub](https://documentation.commvault.com/commvault/v11/article?p=86486.htm) について取り上げた同社のドキュメント サイトから入手できます。 また、[Microsoft Azure の機能を網羅した一覧](https://documentation.commvault.com/commvault/v11/article?p=109795_1.htm)も公開されています。ご参考ください。

## <a name="deploy-from-azure-stack-hub-marketplace"></a>Azure Stack Hub Marketplace からデプロイする

Commvault は、Azure Marketplace で BYOL イメージを公開しており、そのイメージは Azure Stack Hub へのシンジケーションが可能です。 Azure Stack 上の VM をバックアップするために必要な最小バージョンは SP16 です。 仮想アプライアンスの使用を予定している場合は必ず、SP16 (最新の[長期サポート リリース](https://documentation.commvault.com/commvault/v11/article?p=2617.htm)) または SP17 (提供されている最新のメインストリームリリース) に更新してください。

| クラウド        | Version | シンジケーション対応 | 次の更新 |
|--------------|---------|---------------------------|-------------|
| Azure Public | SP13    | はい                       | TBD         |
| Azure Gov    | SP13    | TBD                       | TBD         |

## <a name="download-from-azure-marketplace"></a>Azure Marketplace からダウンロードする

Azure Stack Hub のオペレーターは、接続環境および非接続環境のローカル Azure Stack Marketplace にアイテムをダウンロードすることができます。 接続環境では、Azure から追加できる利用可能なアイテムの一覧をオペレーターが参照できます。

![Azure から追加する](media/azure-stack-commvault-offer-tzl/add-from-azure.png)

## <a name="upload-and-publish-manually"></a>手動でアップロードして公開する

非接続環境では、Azure からアイテムをダウンロードしたうえで、Azure Stack Hub に手動でアップロードする必要があります。 詳細については、[接続環境と非接続環境向けの詳しい手順](../../operator/azure-stack-download-azure-marketplace-item.md)を参照してください。

## <a name="deployment-considerations"></a>デプロイに関する考慮事項

- Azure Stack Hub 外のデプロイ
- Azure Stack Hub に仮想アプライアンスをデプロイする
- ディスク ライブラリとクラウド ライブラリ
- ネットワーク関連の考慮事項
- サブスクリプション レベルの分離

## <a name="next-steps"></a>次のステップ

- IaaS VM の保護について詳しくは、Azure Stack Hub 上の VM の保護に関するページを参照してください。
