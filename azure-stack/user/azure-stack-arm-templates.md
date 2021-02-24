---
title: Azure Stack Hub で Azure Resource Manager テンプレートを使用する
description: Azure Stack Hub で Azure Resource Manager テンプレートを使用してリソースをプロビジョニングする方法を説明します。
author: mattbriggs
ms.topic: article
ms.date: 2/1/2021
ms.author: mabrigg
ms.reviewer: justini
ms.lastreviewed: 11/14/2019
ms.openlocfilehash: 21c6fd7408179c71e6ff2ab64bfc94afba0140b2
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99247015"
---
# <a name="use-azure-resource-manager-templates-in-azure-stack-hub"></a>Azure Stack Hub で Azure Resource Manager テンプレートを使用する

Azure Resource Manager テンプレートを使用して、お使いのアプリケーションのすべてのリソースを、単一の連携した操作でデプロイしてプロビジョニングできます。 テンプレートを再デプロイして、リソース グループ内のリソースを変更することもできます。

これらのテンプレートは、Microsoft Azure Stack Hub ポータル、PowerShell、コマンド ライン、および Visual Studio を使用してデプロイできます。

以下のクイックスタート テンプレートを [GitHub](https://aka.ms/azurestackgithub) で入手できます。

## <a name="deploy-sharepoint-server-non-high-availability-deployment"></a>SharePoint サーバーのデプロイ (非高可用性デプロイ)

PowerShell [Desired State Configuration](/powershell/scripting/dsc/overview/overview) (DSC) 拡張機能を使用して、以下のリソースを含む [SharePoint Server 2013 ファームを作成](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/sharepoint-2013-non-ha)します。

* 仮想ネットワーク
* 3 つのストレージ アカウント
* 2 つの外部ロード バランサー
* 単一のドメインを使用する新しいフォレスト用のドメイン コントローラーとして構成された 1 台の仮想マシン (VM)
* SQL Server 2014 のスタンドアロン サーバーとして構成された 1 台の VM
* ワンマシン SharePoint Server 2013 ファームとして構成された 1 台の VM

## <a name="deploy-ad-non-high-availability-deployment"></a>AD のデプロイ (非高可用性デプロイ)

PowerShell DSC 拡張機能を使用して、以下のリソースを含む [AD ドメイン コントローラーを作成](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/ad-non-ha)します。

* 仮想ネットワーク
* 1 つのストレージ アカウント
* 1 つの外部ロード バランサー
* 単一のドメインを使用する新しいフォレスト用のドメイン コントローラーとして構成された 1 台の仮想マシン (VM)

## <a name="deploy-adsql-non-high-availability-deployment"></a>AD/SQL のデプロイ (非高可用性デプロイ)

PowerShell DSC 拡張機能を使用して、以下のリソースを含む [SQL Server 2014 スタンドアロン サーバーを作成](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/sql-2014-non-ha)します。

* 仮想ネットワーク
* 2 つのストレージ アカウント
* 1 つの外部ロード バランサー
* 単一のドメインを使用する新しいフォレスト用のドメイン コントローラーとして構成された 1 台の仮想マシン (VM)
* SQL Server 2014 のスタンドアロン サーバーとして構成された 1 台の VM

## <a name="vm-dsc-extension-azure-automation-pull-server"></a>VM-DSC-Extension-Azure-Automation-Pull-Server

PowerShell DSC 拡張機能を使用して、既存の仮想マシンのローカル構成マネージャー (LCM) を構成し、それを Azure Automation Account DSC Pull Server に登録します。

## <a name="create-a-virtual-machine-from-a-user-image"></a>ユーザー イメージからの仮想マシンの作成

[カスタム ユーザー イメージから仮想マシンを作成](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/101-vm-create-from-customimage)します。 このテンプレートは、仮想ネットワーク (DNS 付き)、パブリック IP アドレス、およびネットワーク インターフェイスもデプロイします。

## <a name="basic-virtual-machine"></a>基本的な仮想マシン

仮想ネットワーク (DNS 付き)、パブリック IP アドレス、およびネットワーク インターフェイスを含む [Windows VMをデプロイ](https://aka.ms/aa6zdzx)します。

## <a name="cancel-a-running-template-deployment"></a>実行中のテンプレートのデプロイのキャンセル

実行中のテンプレートのデプロイをキャンセルするには、[Stop-AzResourceGroupDeployment](/powershell/module/Az.resources/stop-Azresourcegroupdeployment) PowerShell [コマンドレット](/powershell/scripting/developer/cmdlet/cmdlet-overview)を使用します。

## <a name="next-steps"></a>次のステップ

* [ポータルを使用してテンプレートをデプロイする](azure-stack-deploy-template-portal.md)
* [PowerShell を使用したテンプレートのデプロイ](azure-stack-deploy-template-powershell.md)
* [Visual Studio を使用したテンプレートのデプロイ](azure-stack-deploy-template-visual-studio.md)
* [Azure リソース マネージャーの概要](/azure/azure-resource-manager/resource-group-overview)
