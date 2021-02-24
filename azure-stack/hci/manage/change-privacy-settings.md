---
title: プライバシー設定を変更する
description: このトピックでは、Azure Stack HCI オペレーティング システムまたは Windows Server でプライバシー設定を変更する方法に関するガイダンスを提供します。
author: JohnCobb1
ms.author: v-johcob
ms.topic: how-to
ms.date: 10/05/2020
ms.openlocfilehash: 8055e02b702fb5e585b5570171c6dabb247b623f
ms.sourcegitcommit: 79be77eb2bebf314d956258fc9bc14b2014be190
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/05/2020
ms.locfileid: "91745165"
---
# <a name="change-privacy-settings-on-individual-servers"></a>個々のサーバーのプライバシー設定を変更する

>適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019、Windows Server 2016

ここでは、Azure Stack HCI オペレーティング システムまたは Windows Server が実行されているサーバー上でプライバシー設定を変更する方法を説明します。 一度に複数のサーバーで、またはエンタープライズ全体でグループ ポリシーを使ってプライバシー設定を管理するには、「[エンタープライズ診断データの管理](/windows/privacy/configure-windows-diagnostic-data-in-your-organization#manage-enterprise-diagnostic-data)」を参照してください。

## <a name="azure-stack-hci-or-server-core"></a>Azure Stack HCI または Server Core
サーバーで、Server Core インストール オプションにより Azure Stack HCI オペレーティング システムまたは Windows Server が使用されている場合は、次の手順を実行します。
1. リモート デスクトップ、リモート管理 (ヘッドレスまたは BMC) コントローラー、または KVM を使用するか、キーボードとモニターを使用してローカルでサインオンして、Azure Stack HCI クラスター内のサーバーに接続します。 
1. Azure Stack HCI を実行しているサーバーに接続すると、サーバー構成ツール (Sconfig) が自動的に開きます。 Server Core を使用して Windows Server を実行しているサーバーに接続する場合は、コマンド プロンプトで「`Sconfig`」と入力します。
1. **[Enter a number to select an option:]\(数値を入力してオプションを選択;\)** プロンプトで「**10**」と入力し、Enter キーを押します。
1. **[Change Telemetry]\(テレメトリの変更\)** 確認プロンプトで **[はい]** を選択して、次のオプションを表示します。

    使用可能なテレメトリの設定は次のとおりです。 **[1 Security]\(1 セキュリティ\)** 、 **[2 Basic]\(2 基本\)** 、 **[3 Enhanced]\(拡張\)** 、 **[4 Full]\(4 完全\)**

    >[!NOTE]
    > Azure Stack HCI の既定の設定は **[1 Security]\(1 セキュリティ\)** です。

1. **[Enter new telemetry setting:]\(新しいテレメトリ設定の入力:\)** プロンプトで、必要なオプションを入力し、Enter キーを押します。

## <a name="full-desktop"></a>完全なデスクトップ
サーバーが Windows Server を実行していて、完全なデスクトップ インストール オプションがある場合は、次の手順を実行します。
1. Windows Server のサーバー マネージャー ダッシュボードに接続します。

    キーボードとモニターを使用するか、リモート管理 (ヘッドレスまたは BMC) コントローラーを使用するか、リモート デスクトップを使用してローカルで接続できます。 

1. **[ダッシュボード]** の [Server Manage]\(サーバー管理\) で、 **[ローカル サーバー]** を選択します。
1. サーバーの **[プロパティ]** ページで、 **[Feedback & Diagnostics]\(フィードバックと診断\)** の横にある **[設定]** を選択します。

    **[設定]** ページに、 **[Feedback frequency]\(フィードバックの頻度\)** と **[Diagnostic and usage data]\(診断と使用状況のデータ\)** の設定が表示されます。 
 
1. **[Diagnostic and usage data]\(診断と使用状況のデータ\)** 設定を展開して、次のオプションのいずれかを選択します。
    - **[Required diagnostic data]\(必須の診断データ\)**
    - **拡張**
    - **[Optional diagnostic data]\(追加の診断データ\)**

    >[!NOTE]
    > **[設定]** ページで、通知 **[一部の設定は組織によって管理されています]** が表示される場合は、**診断と使用状況のデータ**の設定が使用可能でない可能性があります。

## <a name="next-steps"></a>次のステップ
一度に複数のサーバーで、またはエンタープライズ全体でグループ ポリシーを使ってプライバシー設定を管理するには、
-   「[エンタープライズ診断データの管理](/windows/privacy/configure-windows-diagnostic-data-in-your-organization#manage-enterprise-diagnostic-data)」を参照してください。
