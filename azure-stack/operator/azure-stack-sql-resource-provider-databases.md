---
title: SQL データベースの作成
titleSuffix: Azure Stack Hub
description: SQL リソース プロバイダー アダプター使用してプロビジョニングされた SQL データベースの作成と管理を行う方法について説明します。
author: bryanla
ms.topic: article
ms.date: 1/22/2020
ms.author: bryanla
ms.reviewer: xiaofmao
ms.lastreviewed: 10/16/2019
ms.openlocfilehash: 0f011fe3017d33627259f58e51795193257e3956
ms.sourcegitcommit: e9a1dfa871e525f1d6d2b355b4bbc9bae11720d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86487925"
---
# <a name="create-sql-databases"></a>SQL データベースの作成

ユーザー ポータルで、セルフサービス データベースを作成して管理できます。 Azure Stack Hub ユーザーは、SQL データベース サービスを含むオファーがあるサブスクリプションが必要です。

1. [Azure Stack Hub](azure-stack-overview.md) のユーザー ポータルにサインインします。

2. **[+ 新規]** &gt; **[データ + ストレージ]** &gt; **[SQL Server データベース]** &gt; **[追加]** を選択します。

3. **[データベースの作成]** で、 **[データベース名]** 、 **[最大サイズ (MB)]** などの必要な情報を入力します。

   >[!NOTE]
   >データベースのサイズは少なくとも 64 MB 必要であり、データベースの配置後に増やすことができます。

   必要に応じて、環境に合うように他の設定を構成します。

4. **[データベースの作成]** で、 **[SKU]** を選択します。 **[SKU の選択]** で、データベースの SKU を選択します。

   ![Azure Stack Hub ユーザー ポータルでのデータベースの作成。](./media/azure-stack-sql-rp-deploy/newsqldba.png)

   >[!NOTE]
   >ホスティング サーバーは、Azure Stack Hub に追加されるときに SKU が割り当てられます。 データベースは、ホスティング サーバー プールの SKU 内に作成されます。

5. **[ログイン]** を選択します。

6. **[ログインの選択]** で、既存のログインを選択するか、 **[+ 新しいログインの作成]** を選択します。

7. **[新しいログイン]** で、**データベースのログイン名**と**パスワード** を入力します。

   >[!NOTE]
   >これらの設定は、このデータベースにアクセスするためにのみ作成する SQL 認証資格情報です。 ログイン ユーザー名はグローバルに一意である必要があります。 同じ SKU を使用する他のデータベースのログイン設定を再利用できます。

   ![Azure Stack Hub ユーザー ポータルでの新しいデータベース ログインの作成](./media/azure-stack-sql-rp-deploy/create-new-login-a.png)

8. **[OK]** を選択して、データベースの配置を完了します。

データベースの配置後に表示される **[要点]** で、 **[接続文字列]** の情報を書き留めます。 この文字列は、SQL Server データベースにアクセスする必要があるすべてのアプリで使用できます。

![SQL Server データベースの接続文字列の取得](./media/azure-stack-sql-rp-deploy/sql-db-settings-a.png)

## <a name="sql-always-on-databases"></a>SQL AlwaysOn データベース

設計により、AlwaysOn データベースの処理は、スタンドアロン サーバー環境とは異なる方法で処理されます。 詳細については、[Azure 仮想マシンの SQL Server AlwaysOn 可用性グループの概要](/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-availability-group-overview)に関する記事を参照してください。

### <a name="verify-sql-always-on-databases"></a>SQL AlwaysOn データベースの確認

次の画面キャプチャは、SQL Server Management Studio を使用して、SQL AlwaysOn データベースの状態を確認する方法を示しています。

![SQL Server Management Studio での AlwaysOn データベースの状態](./media/azure-stack-sql-rp-deploy/verify-always-on.png)

AlwaysOn データベースは **[同期済み]** と表示され、すべての SQL インスタンスで使用でき、 **[可用性グループ]** 内に表示されます。 前のスクリーンショットでは、データベースの例は newdb1 であり、その状態は **newdb1 (同期済み)** です。

### <a name="delete-an-always-on-database"></a>Always On データベースの削除

リソース プロバイダーから SQL AlwaysOn データベースを削除すると、そのデータベースは SQL によって**プライマリ** レプリカと可用性グループから削除されます。

その後、データベースは SQL によって他のレプリカ上に **[復元中]** 状態で配置され、トリガーされない限り、削除されることはありません。 データベースが削除されない場合、セカンダリ レプリカは **[同期されていません]** 状態になります。

## <a name="next-steps"></a>次のステップ

[高可用性 SQL データベースの提供](azure-stack-tutorial-sql.md)方法を確認する
