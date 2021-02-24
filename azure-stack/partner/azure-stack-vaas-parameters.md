---
title: VaaS の共通ワークフロー パラメーター
titleSuffix: Azure Stack Hub
description: Azure Stack Hub のサービスとしての検証の共通ワークフロー パラメーターについて説明します。
author: mattbriggs
ms.topic: article
ms.date: 12/16/2020
ms.author: mabrigg
ms.reviewer: johnhas
ms.lastreviewed: 11/11/2019
ROBOTS: NOINDEX
ms.openlocfilehash: 51821c70153f2b4f091b0e8ccb659dddf863bb5e
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97910859"
---
# <a name="workflow-common-parameters-for-azure-stack-hub-validation-as-a-service"></a>Azure Stack Hub のサービスとしての検証のためのワークフロー共通パラメーター

[!INCLUDE [Azure_Stack_Partner](./includes/azure-stack-partner-appliesto.md)]

共通パラメーターには、サービスとしての検証 (VaaS) のすべてのテストに必要な環境変数やユーザー資格情報などの値が含まれます。 これらの値は、ワークフローを作成または変更するときにワークフロー レベルで定義されます。 テストをスケジュールするときに、ワークフローの各テストにこれらの値がパラメーターとして渡されます。

> [!NOTE]
> 各テストでは、独自のパラメーター セットが定義されています。 スケジュール時に、テストで共通パラメーターとは別に値を入力することが必要な場合があります。また、共通パラメーターの値を上書きできる場合もあります。

## <a name="environment-parameters"></a>環境パラメーター

環境パラメーターは、テスト対象の Azure Stack Hub 環境を表します。 テスト対象の特定のインスタンスの Azure Stack Hub スタンプ情報ファイルを生成してアップロードすることによって、これらの値を提供する必要があります。

> [!NOTE]
> 公式の検証ワークフローでは、ワークフローの作成後に環境パラメーターを変更することはできません。

### <a name="generate-the-stamp-information-file"></a>スタンプ情報ファイルを生成する

1. DVM または Azure Stack Hub 環境にアクセスできる任意のマシンにサインインします。
2. 管理者特権の PowerShell ウィンドウで次のコマンドを実行します。

    ```powershell  
    $CloudAdminUser = "<cloud admin username>"
    $CloudAdminPassword = ConvertTo-SecureString "<cloud admin password>" -AsPlainText -Force
    $stampInfoCreds = New-Object System.Management.Automation.PSCredential($CloudAdminUser, $CloudAdminPassword)
    $params = Invoke-RestMethod -Method Get -Uri 'https://ASAppGateway:4443/ServiceTypeId/4dde37cc-6ee0-4d75-9444-7061e156507f/CloudDefinition/GetStampInformation' -Credential $stampInfoCreds
    ConvertTo-Json $params > stampinfoproperties.json
    ```

### <a name="locate-values-in-the-ece-configuration-file"></a>ECE 構成ファイルで値を検索する

環境パラメーター値は、DVM の `C:\EceStore\403314e1-d945-9558-fad2-42ba21985248\80e0921f-56b5-17d3-29f5-cd41bf862787` にある **ECE 構成ファイル** で手動で検索することもできます。

## <a name="test-parameters"></a>テスト パラメーター

共通のテスト パラメーターには、構成ファイルに格納できない機密情報が含まれます。 これらのパラメーターは手動で指定する必要があります。

パラメーター    | 説明
-------------|-----------------
テナント管理者ユーザー                            | AAD ディレクトリのサービス管理者によってプロビジョニングされた Azure Active Directory (Azure AD) テナント管理者。 このユーザーは、リソース (VM、ストレージ アカウントなど) を設定したり、ワークロードを実行したりするためのテンプレートのデプロイなどのテナント レベルのアクションを実行します。 テナント アカウントのプロビジョニングの詳細については、[新しい Azure Stack Hub テナントの追加](../operator/azure-stack-add-new-user-aad.md)に関する記事をご覧ください。
サービス管理者ユーザー             | Azure Stack Hub のデプロイ中に指定される Azure AD ディレクトリ テナントの Azure AD 管理者。 ECE 構成ファイルで `AADTenant` を検索し、`UniqueName` 要素の値を選択します。
クラウド管理者ユーザー               | Azure Stack Hub ドメイン管理者アカウント (例: `contoso\cloudadmin`)。 ECE 構成ファイルで `User Role="CloudAdmin"` を検索し、`UserName` 要素の値を選択します。
診断接続文字列          | テストの実行中に診断ログのコピー先となる Azure ストレージ アカウントの SAS URL。 SAS URL を生成する手順については、「[診断接続文字列を生成する](#generate-the-diagnostics-connection-string)」をご覧ください。 |

> [!IMPORTANT]
> 先に進むには、**診断接続文字列** が有効である必要があります。

### <a name="generate-the-diagnostics-connection-string"></a>診断接続文字列を生成する

テストの実行中に診断ログを保存するために、診断接続文字列が必要となります。 セットアップ時に作成した Azure Storage アカウント ([サービスとしての検証のリソースの設定](azure-stack-vaas-set-up-resources.md)に関する記事を参照) を使用して、VaaS でログをストレージ アカウントにアップロードする際にアクセスする Shared Access Signature (SAS) URL を作成します。

1. [!INCLUDE [azure-stack-vaas-sas-step_navigate](includes/azure-stack-vaas-sas-step_navigate.md)]

1. **[使用できるサービス]** のオプションから **[BLOB]** を選択します。 残りのオプションについてはすべて選択を解除します。

1. **[使用できるリソースの種類]** で、 **[サービス]** 、 **[コンテナー]** 、 **[オブジェクト]** を選択します。

1. **[与えられているアクセス許可]** で、 **[読み取り]** 、 **[書き込み]** 、 **[リスト]** 、 **[追加]** 、 **[作成]** を選択します。 残りのオプションについてはすべて選択を解除します。

1. **[開始時間]** を現在の時刻に設定し、 **[終了時刻]** を現在の時刻から 3 か月後に設定します。

1. [!INCLUDE [azure-stack-vaas-sas-step_generate](includes/azure-stack-vaas-sas-step_generate.md)]

> [!NOTE]  
> SAS URL の生成時に指定した終了時刻になると、URL の有効期限が切れます。 テストをスケジュールするときは、URL の有効期間が 30 日以上であり、なおかつテストの実行に必要な期間有効であることを確認してください (3 か月を推奨)。

## <a name="next-steps"></a>次のステップ

- [サービスとしての検証の主要概念](azure-stack-vaas-key-concepts.md)を確認する