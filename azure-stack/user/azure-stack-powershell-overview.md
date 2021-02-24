---
title: Azure Stack Hub での PowerShell
description: Azure Stack Hub 内の PowerShell には、さまざまなモジュールやコンテキストがあります。
author: mattbriggs
ms.topic: article
ms.date: 12/2/2020
ms.author: mabrigg
ms.reviewer: sijuman
ms.lastreviewed: 12/2/2020
ms.openlocfilehash: 3e00be01c56b9689e29681e01919feb0f432d7b9
ms.sourcegitcommit: 9ef2cdc748cf00cd3c8de90705ea0542e29ada97
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/02/2020
ms.locfileid: "96524658"
---
# <a name="get-started-with-powershell-in-azure-stack-hub"></a>Azure Stack Hub 内の PowerShell の概要

PowerShell は、コマンド ラインからリソースを管理できるように設計されています。 Azure Resource Manager モデルを使用する自動化ツールをビルドする場合に PowerShell を使用できます。 PowerShell モジュールは、特定の領域のすべての側面を管理するためにグループ化された一連の PowerShell 関数として定義できます。 Azure Stack Hub を操作するには、一連のさまざまな PowerShell コマンドレットを使いこなす必要があります。

この記事は、Azure Stack Hub 内で使用されるさまざまな PowerShell モジュールを習得するのに役立ちます。 Azure Stack Hub 内で PowerShell を使用するときは、次の表に示す 4 セットの API のいずれかとやりとりすることができます。

| API | PowerShell リファレンス | REST リファレンス |
| --- | --- | --- |
| グローバルな Azure Resource Manager | [Azure PowerShell モジュール](https://github.com/Azure/azure-powershell/blob/master/documentation/azure-powershell-modules.md) | [REST API ブラウザー](/rest/api/) |
| Azure Stack Hub Resource Manager | [Azure Stack Hub での API バージョンのプロファイルの管理](azure-stack-version-profiles.md) | [Azure Stack Hub での API バージョンのプロファイルの管理](azure-stack-version-profiles.md) |
| Azure Stack Hub の管理者エンドポイント | [Azure Stack Hub 管理モジュール](/powershell/azure/azure-stack/overview) | [REST API ブラウザー - Azure Stack Hub](/rest/api/?term=Azure Azure Stack Admin) |
| Azure Stack Hub の特権エンドポイント | [Azure Stack Hub で特権エンドポイントを使用する](../operator/azure-stack-privileged-endpoint.md) | |

各インターフェイスでは、グローバル Azure または Azure Stack Hub のリソース プロバイダーに接続します。 リソース プロバイダーにより、Azure の機能が有効になります。 たとえば、Azure コンピューティングのリソース プロバイダーでは、仮想マシンとそれに対応するリソースの作成および管理へのプログラムによるアクセスを提供します。

リソース プロバイダーは、リソースを管理および構成するための機能と制御の両方を提供します。 リソース プロバイダーには、Azure Resource Manager を使用してプログラムからアクセスできます。 さらに、そのインターフェイスは、PowerShell、Azure CLI、およびお客様独自の REST クライアントのサーフェイスを提供します。

## <a name="where-to-find-azure-stack-hub-powershell"></a>Azure Stack Hub PowerShell を探す場所

次のブロック図は、PowerShell モジュールのセット間の関係を示しています。 お使いのマシンから、PowerShell モジュールを読み込んで、グローバル Azure および Azure Stack Hub の両方を管理できます。

![Azure Stack Hub PowerShell](media/azure-stack-powershell-overview/azure-stack-powershell.svg)

### <a name="global-azure"></a>グローバル Azure

Azure PowerShell には、お使いの Azure リソースの操作に Azure Resource Manager の最新バージョンを使用する一連のコマンドレットが含まれています。 Azure PowerShell では .NET Standard バージョンが使用されます。つまり、Windows、macOS、Linux で PowerShell のバージョンを使用することができます。 Azure PowerShell は、Azure Cloud Shell 上で利用することもできます。 詳細については、[Azure PowerShell の概要](/powershell/azure/get-started-azureps)に関するページを参照してください。

### <a name="azure-stack-hub-resource-manager"></a>Azure Stack Hub Resource Manager

Azure Stack Hub PowerShell には、Azure Resource Manager の以前のバージョンを使用する一連のコマンドレットが用意されています。 これらのコマンドレットは、Azure Stack Hub のリソース プロバイダーと互換性があります。 Azure Stack Hub の各リソース プロバイダーでは、グローバル Azure にあるプロバイダーの古いバージョンを使用します。 Azure Stack Hub でサポートされている各プロバイダーのバージョンを調整しやすくするために、API プロファイルを使用できます。 詳細については、「[Azure Stack Hub での API バージョンのプロファイルの管理](azure-stack-version-profiles.md)」を参照してください。

### <a name="azure-stack-hub-administrator"></a>Azure Stack Hub 管理者

Azure Stack Hub では、クラウド オペレーターが Azure Stack Hub のインストールおよび管理を行うことができるように、一連のリソース プロバイダーがオペレーターに公開されます。 グローバル Azure では、この操作がユーザーから抽象化され、Azure の一部として、バックグラウンドで処理されます。 しかし、Azure Stack Hub を使用することで、企業はプライベート クラウドをサポートできます。 これらのタスクを行うには、オペレーターが Azure Stack Hub 管理 API を操作します。 詳細については、「[PowerShell for Azure Stack Hub をインストールする](../operator/powershell-install-az-module.md)」を参照してください。

### <a name="azure-stack-hub-privileged-endpoint"></a>Azure Stack Hub の特権エンドポイント

インストールのテスト、ログへのアクセスなど、Azure Stack Hub 内でのオペレーターのアクティビティでは、オペレーターは特権エンドポイント (PEP) を操作できます。 この PEP は、あらかじめ構成されたリモート PowerShell コンソールであり、特定のタスクを実行するのに十分なアクセスをオペレーターに提供します。 エンドポイントは PowerShell Just Enough Administration (JEA) を使用して、コマンドレットの限定的なセットを公開します。 詳細については、「[Azure Stack Hub で特権エンドポイントを使用する](../operator/azure-stack-privileged-endpoint.md)」を参照してください。

### <a name="azure-stack-hub-tools"></a>Azure Stack Hub ツール

Azure Stack Hub では、GitHub リポジトリの *AzureStack-Tools* でスクリプトと追加のコマンドレットを利用できます。 AzureStack-Tools では、Azure Stack Hub に対するリソースの管理とデプロイのための PowerShell モジュールをホストしています。 VPN 接続を確立しようとしている場合は、これらの PowerShell モジュールを Azure Stack Development Kit か、Windows ベースの外部クライアントにダウンロードできます。 詳細については、[AzureStack-Tools](https://github.com/Azure/AzureStack-Tools) ページを参照してください。

## <a name="work-with-powershell-in-azure-stack-hub"></a>Azure Stack Hub 内で PowerShell を操作する

PowerShell には、Azure Resource Manager をプログラムから操作するための方法が用意されています。 対話型のコマンド プロンプトを操作するか、タスクを自動化する場合は、スクリプトを記述できます。

Azure Stack Hub PowerShell の操作に多くの時間がかかる場合、モジュールのインストールや再インストールをすることがあります。 グローバル Azure を同時に操作している場合には、このルーチンは困難な場合があります。お客様の目的に合わせてお使いのモジュールのアンインストールや再インストールを行う必要があるためです。

Docker コンテナーを使用して、お使いのローカル コンピューター上の PowerShell をバージョンごとに分離できます。 PowerShell モジュール セットから PowerShell モジュール セットに切り替えられるように、Docker コンテナーを使用するには、[Docker を使用した PowerShell の実行](azure-stack-powershell-user-docker.md)に関するページを参照してください。


## <a name="next-steps"></a>次のステップ

- Azure Stack Hub の [PowerShell の API プロファイル](azure-stack-version-profiles.md)についてお読みください。
- [Azure Stack Hub PowerShell](../operator/powershell-install-az-module.md) をインストールします。
- クラウドの一貫性のための [Azure Resource Manager テンプレート](azure-stack-develop-templates.md)の作成についてお読みください。
