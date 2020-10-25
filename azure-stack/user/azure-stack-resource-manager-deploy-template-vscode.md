---
title: Visual Studio Code を使用して Azure Stack Hub にデプロイする
description: Visual Studio Code で Azure Resource Manager テンプレートを作成し、デプロイ スキーマを使用して、ご使用のバージョンの Azure Stack Hub と互換性のあるテンプレートを準備します。
author: mattbriggs
ms.topic: article
ms.date: 5/27/2020
ms.author: mabrigg
ms.reviewer: sijuman
ms.lastreviewed: 09/30/2019
ms.openlocfilehash: 3f97ec57900119543e37b0d99a7d82e30c900f42
ms.sourcegitcommit: 3e2460d773332622daff09a09398b95ae9fb4188
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/15/2020
ms.locfileid: "90574246"
---
# <a name="deploy-with-visual-studio-code-to-azure-stack-hub"></a>Visual Studio Code を使用して Azure Stack Hub にデプロイする

Visual Studio Code と Azure Resource Manager ツールの拡張機能を使用して、お使いのバージョンの Azure Stack Hub で動作する Azure Resource Manager テンプレートを作成および編集できます。 Visual Studio Code では、拡張機能を使わずに Resource Manager テンプレートを作成することもできますが、拡張機能を利用すれば、オートコンプリート機能によってテンプレートの開発を省力化することができます。 さらに、Azure Stack Hub で使用可能なリソースの理解に役立つデプロイ スキーマを指定できます。

この記事では、Windows 仮想マシンをデプロイします。

## <a name="concepts-for-azure-stack-hub-resource-manager"></a>Azure Stack Hub Resource Manager の概念

### <a name="azure-stack-hub-resource-manager"></a>Azure Stack Hub Resource Manager

Azure Stack Hub での Azure ソリューションのデプロイと管理に関する概念を理解するには、「[Azure Stack Hub で Azure Resource Manager テンプレートを使用する](azure-stack-arm-templates.md)」を参照してください。

### <a name="api-profiles"></a>API プロファイル
Azure Stack Hub でのリソース プロバイダーの調整に関する概念を理解するには、「[Azure Stack Hub での API バージョンのプロファイルの管理](azure-stack-version-profiles.md)」を参照してください。

### <a name="the-deployment-schema"></a>デプロイ スキーマ

Azure Stack Hub デプロイ スキーマでは、Visual Studio Code での Azure Resource Manager テンプレートを使用したハイブリッド プロファイルがサポートされています。 JSON テンプレート内の 1 行を変更してスキーマを参照できます。その後、IntelliSense を使用して、Azure と互換性のあるリソースを確認できます。 このスキーマを使用して、お使いのバージョンの Azure Stack Hub 内でサポートされているリソース プロバイダー、種類、API のバージョンを確認します。 このスキーマは、お使いのバージョンの Azure Stack Hub でサポートされているリソース プロバイダー内の API エンドポイントの特定バージョンを取得する API プロファイルに依存します。 type と apiVersion に単語補完を使用すると、API プロファイルで使用可能な apiVersion とリソースの種類に制限されます。

## <a name="prerequisites"></a>前提条件

- [Visual Studio Code](https://code.visualstudio.com/)
- Azure Stack Hub へのアクセス
- 管理エンドポイントに到達するマシンに、[Azure Stack Hub PowerShell がインストールされている](../operator/azure-stack-powershell-install.md?toc=https%3A%2F%2Fdocs.microsoft.com%2Fen-us%2Fazure-stack%2Fuser%2FTOC.json&bc=https%3A%2F%2Fdocs.microsoft.com%2Fen-us%2Fazure-stack%2Fbreadcrumb%2Ftoc.json)

## <a name="install-resource-manager-tools-extension"></a>Resource Manager ツール拡張機能のインストール

Resource Manager ツール拡張機能をインストールするには、次の手順を使用します。

1. Visual Studio Code を開きます。
2. Ctrl + Shift + X キーを押して、拡張機能ウィンドウを開きます
3. `Azure Resource Manager Tools` を検索し、 **[インストール]** を選択します。
4. **[再読み込み]** を選択して、拡張機能のインストールを完了します。

## <a name="get-a-template"></a>テンプレートの取得

ゼロからテンプレートを作成するのではなく、AzureStack-QuickStart-Templates (https://github.com/Azure/AzureStack-QuickStart-Templates) からテンプレートを開きます。 AzureStack-QuickStart-Templates は、リソースを Azure Stack Hub にデプロイする Resource Manager テンプレートのリポジトリです。 

この記事のテンプレートの名前は `101-vm-windows-create` です。 このテンプレートでは、Azure Stack Hub への Windows VM の基本的なデプロイを定義します。  このテンプレートでは、仮想ネットワーク (DNS 付き)、ネットワーク セキュリティ グループ、およびネットワーク インターフェイスもデプロイされます。

1. Visual Studio Code を開き、お使いのマシン上の作業フォルダーに移動します。
2. Visual Studio Code で Git bash ターミナルを開きます。
3. 次のコマンドを実行して、Azure Stack Hub クイック スタート リポジトリを取得します。
    ```bash  
    Git clone https://github.com/Azure/AzureStack-QuickStart-Templates.git
    ```
4. リポジトリが格納されているディレクトリを開きます。
    ```bash  
    CD AzureStack-QuickStart-Templates
    ```
5. **[開く]** を選択して、リポジトリ内の `/101-vm-windows-create/azuredeploy.json` のファイルを開きます。
6. そのファイルを自分のワークスペースに保存します。または、リポジトリのブランチを作成した場合は、その場で作業できます。
7. そのファイルを開いたまま、`$Schema` フィールドを `https://schema.management.azure.com/schemas/2019-03-01-hybrid/deploymentTemplate.json#` に変更します。
8. apiProfile フィールドの値をクリアすることで、デプロイ スキーマが機能することを確認できます。
    ```JSON  
    "apiProfile": ""
    ```
9. 空の引用符の間にカーソルを置き、CTRL + SPACE キーを押します。 Azure Stack Hub のデプロイ スキーマ内の有効な API プロファイルから選択できます。 この操作は、テンプレート内の各リソース プロバイダーで実行できます。

    ![Azure Stack Hub Resource Manager のデプロイ スキーマ](./media/azure-stack-resource-manager-deploy-template-vscode/azure-stack-resource-manager-vscode-schema.png)

10. 準備ができたら、PowerShell を使用してテンプレートをデプロイできます。 [PowerShell を使用したデプロイ](azure-stack-deploy-template-powershell.md)に関する記事の手順に従います。 スクリプトのテンプレートの場所を指定します。
11. Windows VM をデプロイした後、Azure Stack Hub ポータルに移動し、リソース グループを見つけます。 この演習の結果を Azure Stack Hub から消去する場合は、このリソース グループを削除します。

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub Resource Manager テンプレート](azure-stack-arm-templates.md)について詳しく確認します。  
- [Azure Stack Hub の API プロファイル](azure-stack-version-profiles.md)について詳しく確認します。
