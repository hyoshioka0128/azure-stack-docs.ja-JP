---
title: Infrastructure Backup サービスを使用した Azure Stack のデータの回復 | Microsoft Docs
description: Infrastructure Backup サービスを使用して、Azure Stack の構成とサービス データをバックアップおよび復元する方法について学習します。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
ms.service: azure-stack
ms.workload: tzl
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/29/2020
ms.author: sethm
ms.reviewer: hectorl
ms.lastreviewed: 07/29/2020
ms.openlocfilehash: 1941a1347e7b6ce3943a8cfae02e07a6b41fa46f
ms.sourcegitcommit: 50b362d531c2d35a3a935811fee71252971bd5d8
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/09/2020
ms.locfileid: "96941295"
---
# <a name="recover-data-in-azure-stack-with-the-infrastructure-backup-service"></a>Infrastructure Backup サービスを使用した Azure Stack のデータの回復

*適用対象:Modular Data Center および Azure Stack Hub ラグド*

Azure Stack の Infrastructure Backup サービスを使用して、インフラストラクチャ サービス メタデータをバックアップできます。 これらのバックアップは、インフラストラクチャ サービスの低下を修復するために使用されます。 バックアップには、システム内部のインフラストラクチャ サービスからのデータのみが含まれます。 バックアップ データには、ユーザー データとアプリケーション データは含まれません。 ユーザー データとアプリケーション データは個別に保護する必要があります。

既定では、インフラストラクチャのバックアップはデプロイ時に有効にされます。 これらのバックアップはシステム内部に格納され、高度なサポート ケースでのみアクセスできます。 システムが外部の保存場所にアクセスできる場合は、バックアップがセカンダリ コピーとしてその保存場所にエクスポートされるように、Infrastructure Backup サービスに指示できます。

バックアップ サービスを有効にする前に、[要件が満たされている](../../operator/azure-stack-backup-reference.md#backup-controller-requirements)ことを確認してください。

> [!NOTE]
> Infrastructure Backup サービスには、ユーザー データとアプリは含まれません。 IaaS VM ベースのアプリを保護する方法の詳細については、次の記事をご覧ください。
>
> - [Azure Stack にデプロイされた VM の保護](../../user/azure-stack-manage-vm-protect.md)
> - Azure Stack での Event Hubs 時系列データの保護
> - Azure Stack での BLOB データの保護

## <a name="the-infrastructure-backup-service"></a>インフラストラクチャ バックアップ サービス

このサービスには以下の機能が含まれています。

| 機能                                            | 説明                                                                                                                                                |
|----------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------|
| バックアップ インフラストラクチャ サービス                     | Azure Stack で Infrastructure Backup サービスのサブセット全体のバックアップを調整します。 |
| バックアップ データの圧縮と暗号化 | バックアップ データは、システムによって圧縮および暗号化されてから、内部的に保存されるか、オペレーターが提供する外部の保存場所にエクスポートされます。                |
| バックアップ ジョブの監視                              | バックアップ ジョブが失敗した場合、システムによって問題の解決方法が通知されます。                                                                                                |

## <a name="verify-requirements-for-the-infrastructure-backup-service"></a>インフラストラクチャ バックアップ サービスの要件を確認する

- **保存場所** 外部の保存場所に対する信頼性の高いアクセス権がある場合は、複数のバックアップを格納できる Azure Stack インフラストラクチャからアクセス可能なファイル共有が必要です。 Infrastructure Backup サービスの保存場所の選択の詳細については、「[バックアップ コントローラーの要件](../../operator/azure-stack-backup-reference.md#backup-controller-requirements)」をご覧ください。

- **資格情報** 保存場所にアクセスできるユーザー アカウントの資格情報が必要です。

## <a name="next-steps"></a>次のステップ

- [管理者ポータルで Azure Stack のバックアップを有効にする](../../operator/azure-stack-backup-enable-backup-console.md)方法を学習します。

- [PowerShell で Azure Stack のバックアップを有効にする](../../operator/azure-stack-backup-enable-backup-powershell.md)方法を学びます。
