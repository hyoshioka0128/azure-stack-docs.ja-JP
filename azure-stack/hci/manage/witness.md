---
title: クラスター監視のセットアップ
description: クラスター監視のセットアップ方法を学習します
author: v-dasis
ms.topic: how-to
ms.date: 01/21/2021
ms.author: v-dasis
ms.reviewer: JasonGerend
ms.openlocfilehash: cb964bafae7dd9252b386c30a12251e89fc8ead5
ms.sourcegitcommit: e772df8ac78c86d834a68d1a8be83b7f738019b7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/26/2021
ms.locfileid: "98782025"
---
# <a name="set-up-a-cluster-witness"></a>クラスター監視のセットアップ

> 適用対象: Azure Stack HCI バージョン 20H2、Windows Server 2019

監視リソースのセットアップはすべてのクラスターで必須であり、クラスターを作成した直後にセットアップする必要があります。 2 ノード クラスターでは、どちらかのサーバーがオフラインになったときに他のノードも使用できなくなることがないように監視が必要です。 3 つ以上のノードのクラスターでは、2 台のサーバーが障害が発生するかそれらがオフラインになったときに備えて監視が必要です。  

SMB ファイル共有を監視として使用するか、Azure クラウド監視を使用することができます。 クラスター内のすべてのサーバー ノードに信頼性の高いインターネット接続がある場合は、Azure クラウド監視が推奨されます。 詳しくは、「[フェールオーバー クラスターにクラウド監視を展開する](/windows-server/failover-clustering/deploy-cloud-witness)」をご覧ください。

ファイル共有監視の場合、ファイル サーバーには要件があります。 詳細については、[システム要件](../concepts/system-requirements.md)に関するページを参照してください。

## <a name="set-up-a-witness-using-windows-admin-center"></a>Windows Admin Center を使用して監視をセットアップする

1. Windows Admin Center で、上部のドロップダウン矢印から **[Cluster Manager]\(クラスター マネージャー\)** を選択します。
1. **[Cluster connections]\(クラスター接続\)** の下で、クラスターを選択します。
1. **[ツール]** の **[設定]** を選択します。
1. 右側のペインで **[Witness]\(監視\)** を選択します。
1. **[Witness type]\(監視の種類\)** ボックスの一覧で、次のいずれかを選択します。
      - **[クラウド監視]** - Azure ストレージ アカウント名、アクセス キー、エンドポイント URL を入力します (下記参照)
      - **[ファイル共有監視]** - ファイル共有パス "(//server/share)" を入力します

> [!NOTE]
> 3 番目のオプションである **[ディスク監視]** は、ストレッチ クラスターでの使用に適していません。

## <a name="create-an-azure-storage-account-to-use-as-a-cloud-witness"></a>クラウド監視として使用する Azure Storage アカウントを作成する

このセクションでは、ストレージ アカウントを作成し、そのアカウントのエンドポイント URL とアクセス キーを表示およびコピーする方法について説明します。

クラウド監視を構成するには、アービトレーション用の BLOB ファイルの格納に使用できる有効な Azure ストレージ アカウントが必要です。 クラウド監視では、Microsoft ストレージ アカウントの下に、よく知られた **msft-cloud-witness** というコンテナーが作成されます。 クラウド監視により、**msft-cloud-witness** コンテナーの下に 1 つの BLOB ファイルが書き込まれます。この BLOB ファイルの名前には、対応するクラスターの一意の ID が使用されます。 つまり、同じ Microsoft Azure ストレージ アカウントを使用して、複数の異なるクラスターに対してクラウド監視を構成できるということです。

複数の異なるクラスターに対して同じ Azure ストレージ アカウントを使用してクラウド監視を構成すると、1 つの **msft-cloud-witness** コンテナーが自動的に作成されます。 このコンテナーには、クラスターごとに 1 つの BLOB ファイルが格納されます。

> [!NOTE]  
> クラウド監視では、HTTPS (既定のポートは 443) を使用して Azure BLOB サービスとの通信を確立します。 ネットワーク プロキシ経由で HTTPS ポートにアクセスできることを確認してください。

### <a name="to-create-an-azure-storage-account"></a>Azure ストレージ アカウントを作成するには

1. [Azure portal](https://portal.azure.com) にサインインします。
1. [ハブ] メニューで、[新規]、[データ + ストレージ]、[ストレージ アカウント] の順に選択します。
1. [ストレージ アカウントを作成する] ページで、次の手順を実行します。
    1. ストレージ アカウントの名前を入力します。
    <br>ストレージ アカウント名の長さは 3 から 24 文字である必要があり、数字と小文字のみを使用できます。 ストレージ アカウント名は、Azure 内で一意である必要もあります。
    1. **[アカウントの種類]** で **[汎用]** を選択します。
    <br>クラウド監視に BLOB ストレージ アカウントを使用することはできません。
    1. **[パフォーマンス]** には **[Standard]** を選択します
    <br>クラウド監視に Azure Premium Storage を使用することはできません。
    1. **[レプリケーション]** には **[ローカル冗長ストレージ (LRS)]** を選択します。
    <br>フェールオーバー クラスタリングでは、アービトレーション ポイントとして BLOB ファイルを使用します。ここでは、データの読み取り時に一定の整合性が保証される必要があります。 したがって、**レプリケーション** の種類として **ローカル冗長ストレージ** を選択する必要があります。

### <a name="view-and-copy-storage-access-keys-for-your-azure-storage-account"></a>Azure ストレージ アカウントのストレージ アクセス キーを表示およびコピーする

Microsoft Azure ストレージ アカウントを作成すると、自動的に生成される 2 つのアクセス キー (プライマリ アクセス キーとセカンダリ アクセス キー) に関連付けられます。 クラウド監視を初めて作成した場合は、**プライマリ アクセス キー** を使用します。 クラウド監視に使用するキーに関する制限はありません。  

#### <a name="to-view-and-copy-storage-access-keys"></a>ストレージ アクセス キーを表示およびコピーするには、次のようにします

Azure portal で、お使いのストレージ アカウントに移動し、 **[すべての設定]** 、 **[アクセス キー]** の順にクリックすると、アカウントのアクセス キーを表示、コピー、再生成できます。 また、 [アクセス キー] ブレードには、コピーしてアプリケーションで使用できるプライマリ キーとセカンダリ キーを使用してあらかじめ構成された接続文字列も含まれています。

:::image type="content" source="media/witness/cloud-witness-1.png" alt-text="クラウド監視のアクセス キー" lightbox="media/witness/cloud-witness-1.png":::

### <a name="view-and-copy-endpoint-url-links"></a>エンドポイント URL リンクを表示およびコピーする

ストレージ アカウントを作成すると、次の形式で URL が生成されます: `https://<Storage Account Name>.<Storage Type>.<Endpoint>`  

クラウド監視では、ストレージの種類として常に **BLOB** が使用されます。 Azure では、エンドポイントとして **.core.windows.net** が使用されます。 クラウド監視を構成する際は、シナリオに従って別のエンドポイントで構成することもできます (たとえば、中国の Microsoft Azure データセンターに異なるエンドポイントがある場合など)。  

> [!NOTE]  
> エンドポイント URL は、クラウド監視リソースによって自動的に生成されます。URL に対して追加の構成手順を行う必要はありません。  

#### <a name="to-view-and-copy-endpoint-url-links"></a>エンドポイントの URL リンクを表示およびコピーするには、次のようにします

Azure portal でお使いのストレージ アカウントに移動し、 **[すべての設定]** 、 **[プロパティ]** の順にクリックすると、エンドポイントの URL を表示およびコピーできます。  

:::image type="content" source="media/witness/cloud-witness-2.png" alt-text="クラウド監視エンドポイント URL" lightbox="media/witness/cloud-witness-2.png":::  

## <a name="set-up-a-witness-using-windows-powershell"></a>Windows PowerShell を使用して監視をセットアップする

PowerShell を使用してクラスター監視をセットアップするには、次のいずれかのコマンドレットを実行します。

Azure クラウド監視を作成するには、次のコマンドレットを使用します。

```powershell
Set-ClusterQuorum –Cluster "Cluster1" -CloudWitness -AccountName "AzureStorageAccountName" -AccessKey "AzureStorageAccountAccessKey"
```

ファイル共有監視を作成するには、次のコマンドレットを使用します。

```powershell
Set-ClusterQuorum -FileShareWitness "\\fileserver\share" -Credential (Get-Credential)
```

## <a name="next-steps"></a>次のステップ

- クラスター クォーラムの詳細については、「[Azure Stack HCI のクラスターとプールのクォーラムについて](../concepts/quorum.md)」を参照してください。

- Azure ストレージ アカウントを作成および管理する方法の詳細については、[Azure ストレージ アカウント](/azure/storage/common/storage-account-create)に関するページを参照してください。