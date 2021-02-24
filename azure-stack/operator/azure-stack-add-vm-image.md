---
title: Azure Stack Hub にカスタム VM イメージを追加する
description: Azure Stack Hub にカスタム VM イメージを追加または削除する方法について説明します。
author: sethmanheim
ms.topic: how-to
ms.date: 02/08/2021
ms.author: sethm
ms.reviewer: kivenkat
ms.lastreviewed: 9/8/2020
ms.openlocfilehash: 6b80ea5a25ab83eb5b62842fe1c21e7e698ab51a
ms.sourcegitcommit: 824fd33fd5d6aa0c0dac06c21b592bdb60378940
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2021
ms.locfileid: "99850940"
---
# <a name="add-and-remove-a-custom-vm-image-to-azure-stack-hub"></a>Azure Stack Hub に対してカスタム VM イメージを追加または削除する

Azure Stack Hub では、オペレーターとして、カスタム仮想マシン (VM) のイメージをマーケットプレースに追加して、ユーザーに提供することができます。 VM イメージは、管理者ポータルまたは Windows PowerShell を介して Azure Stack Hub Marketplace に追加できます。 グローバル Microsoft Azure Marketplace のイメージをカスタム イメージのベースとして使用するか、Hyper-V を使用して独自のものを作成します。

## <a name="add-an-image"></a>イメージの追加

一般化イメージおよび特殊化イメージを追加する手順については、ユーザー ガイドの **コンピューティング** に関するセクションを参照してください。 ユーザーにイメージを提供する前に、一般化イメージを作成することをお勧めします。 手順については、[Azure Stack Hub への VM の移動の概要](../user/vm-move-overview.md)に関するページを参照してください。 テナントに使用できるイメージを作成するときは、ユーザー ポータルまたはテナント ディレクトリ エンドポイントではなく、Azure Stack Hub 管理ポータルまたは管理者エンドポイントを使用します。

ユーザーがイメージを使用できるようにするには、次の 2 つの方法があります。

- **Azure Resource Manager からのみアクセスできるイメージを提供する**  
  Azure Stack Hub 管理ポータルの **[コンピューティング]**  >  **[イメージ]** でイメージを追加すると、すべてのテナントがイメージにアクセスできます。 ただし、ユーザーは Azure Resource Manager テンプレートを使用してそれにアクセスする必要があります。 それは、Azure Stack Hub Marketplace には表示されません。

- **Azure Stack Hub Marketplace を使用してイメージを提供する**  
    Azure Stack Hub 管理ポータルを使用してイメージを追加すると、マーケットプレース オファリングを作成できます。 手順については、「[Azure Stack Hub でカスタム Marketplace アイテムを作成して発行する](azure-stack-create-and-publish-marketplace-item.md)」を参照してください。

## <a name="add-a-platform-image"></a>プラットフォーム イメージを追加する

プラットフォーム イメージを Azure Stack Hub に追加するには、PowerShell を使用して、Azure Stack Hub 管理者ポータルまたはエンドポイントを使用します。 まず、汎用化した VHD を作成する必要があります。 詳細については、[Azure Stack Hub への VM の移動の概要](../user/vm-move-overview.md)に関するページを参照してください。

### <a name="portal"></a>[ポータル](#tab/image-add-portal)

Azure Stack Hub オペレーターとしてポータルを使用して VM イメージを追加します。

1. Azure Stack Hub にオペレーターとしてサインインします。 左側のナビゲーションから **[ダッシュボード]** を選択します。

2. **[リソース プロバイダー]** リストから **[コンピューティング]** を選びます。

   [![[コンピューティング] を選択](./media/azure-stack-add-vm-image/dash-small.png)](./media/azure-stack-add-vm-image/dash.png#lightbox)

3. **[VM イメージ]** を選択し、 **[追加]** を選択します。

   [![VM イメージを追加する](./media/azure-stack-add-vm-image/tca4-small.png)](./media/azure-stack-add-vm-image/tca4.png#lightbox)

2. **[イメージの作成]** で **[発行元]** 、 **[オファー]** 、 **[SKU]** 、 **[バージョン]** 、[OS ディスクの BLOB URI] に入力します。 その後 **[作成]** をクリックすると VM イメージの作成が開始されます。

   [![カスタム イメージ サイドローディング UI](./media/azure-stack-add-vm-image/tca5-small.png)](./media/azure-stack-add-vm-image/tca5.png#lightbox)

   イメージが正常に作成されると、VM イメージの状態が **[成功]** に変わります。

3. イメージを追加すると、そのイメージは Azure Resource Manager ベースのテンプレートと PowerShell のデプロイのみで使用できるようになります。 イメージを Marketplace 項目としてユーザーが使用できるようにするには、「[Marketplace アイテムを作成および発行する](azure-stack-create-and-publish-marketplace-item.md)」の手順を使用して Marketplace 項目を公開してください。 **Publisher**、**Offer**、**SKU**、**Version** の値を書き留めておいてください。 これらは、カスタム .azpkg 内の Resource Manager テンプレートと Manifest.json を編集するときに必要になります。

### <a name="powershell"></a>[PowerShell](#tab/image-add-ps)

 Azure Stack Hub オペレーターとして PowerShell を使用して VM イメージを追加します。

1. [PowerShell for Azure Stack Hub をインストールします](azure-stack-powershell-install.md)。  

2. Azure Stack Hub にオペレーターとしてサインインします。 手順については、[オペレーターとしての Azure Stack Hub へのサインイン](azure-stack-powershell-configure-admin.md)に関するページをご覧ください。

3. 管理者特権のプロンプトで PowerShell を開き、次を実行します。

   ```powershell
    Add-AzsPlatformimage -publisher "<publisher>" `
      -offer "<Offer>" `
      -sku "<SKU>" `
      -version "<#.#.#>" `
      -OSType "<OS type>" `
      -OSUri "<OS URI>"
   ```

   **Add-AzsPlatformimage** コマンドレットでは、VM イメージを参照するために Azure Resource Manager テンプレートによって使用される値が指定されます。 値は次のとおりです。
   - **publisher**  
     例: `Canonical`  
     イメージをデプロイするときにユーザーが使用する VM イメージの **発行元** 名のセグメント。 このフィールドではスペースや他の特殊文字は使用できません。  
   - **offer**  
     例: `UbuntuServer`  
     VM イメージをデプロイするときにユーザーが使用する VM イメージの **オファー** 名のセグメント。 このフィールドではスペースや他の特殊文字は使用できません。  
   - **sku**  
     例: `14.04.3-LTS`  
     VM イメージをデプロイするときにユーザーが使用する VM イメージの **SKU** 名のセグメント。 このフィールドではスペースや他の特殊文字は使用できません。  
   - **version**  
     例: `1.0.0`  
     VM イメージをデプロイするときにユーザーが使用する VM イメージのバージョン。 このバージョンの形式は **\#.\#.\#** です。 このフィールドではスペースや他の特殊文字は使用できません。  
   - **osType**  
     例: `Linux`  
     イメージの **osType** には **Windows** または **Linux** を指定する必要があります。  
   - **OSUri**  
     例: `https://storageaccount.blob.core.windows.net/vhds/Ubuntu1404.vhd`  
     `osDisk` の Blob Storage URI を指定できます。  

     詳細については、[Add-AzsPlatformimage](/powershell/module/azs.compute.admin/add-azsplatformimage) コマンドレットの PowerShell リファレンスを参照してください。

4. イメージを追加すると、そのイメージは Azure Resource Manager ベースのテンプレートと PowerShell のデプロイのみで使用できるようになります。 イメージを Marketplace 項目としてユーザーが使用できるようにするには、「[Marketplace アイテムを作成および発行する](azure-stack-create-and-publish-marketplace-item.md)」の手順を使用して Marketplace 項目を公開してください。 **Publisher**、**Offer**、**SKU**、**Version** の値を書き留めておいてください。 これらは、カスタム .azpkg 内の Resource Manager テンプレートと Manifest.json を編集するときに必要になります。

---

## <a name="remove-a-platform-image"></a>プラットフォーム イメージを削除する

ポータルまたは PowerShell を使用して、プラットフォーム イメージを削除できます。

### <a name="portal"></a>[ポータル](#tab/image-rem-portal)

Azure Stack Hub オペレーターとして Azure Stack Hub ポータルを使用して VM イメージを削除するには、次の手順を行います。

1. [Azure Stack Hub 管理者ポータル](https://portal.azure.com/signin/index)を開きます。

2. VM イメージに Marketplace 項目が関連付けられている場合は、 **[Marketplace management]\(Marketplace 管理\)** を選択し、削除する VM Marketplace 項目を選択します。

3. VM イメージに Marketplace 項目が関連付けられていない場合は、 **[すべてのサービス] > [コンピューティング] > [VM イメージ]** に移動し、VM イメージの横にある省略記号 ( **...** ) を選択します。

4. **[削除]** を選択します。

### <a name="powershell"></a>[PowerShell](#tab/image-rem-ps)

Azure Stack Hub オペレーターとして PowerShell を使用して VM イメージを削除するには、次の手順を行います。

1. [PowerShell for Azure Stack Hub をインストールします](azure-stack-powershell-install.md)。

2. Azure Stack Hub にオペレーターとしてサインインします。

3. 管理者特権のプロンプトで PowerShell を開き、次を実行します。

   ```powershell  
   Remove-AzsPlatformImage `
    -publisher "<Publisher>" `
    -offer "<Offer>" `
    -sku "<SKU>" `
    -version "<Version>" `
   ```

   **Remove-AzsPlatformImage** コマンドレットでは、VM イメージを参照するために Azure Resource Manager テンプレートによって使用される値が指定されます。 値は次のとおりです。
   - **publisher**  
     例: `Canonical`  
     イメージをデプロイするときにユーザーが使用する VM イメージの **発行元** 名のセグメント。 このフィールドではスペースや他の特殊文字は使用できません。  
   - **offer**  
     例: `UbuntuServer`  
     VM イメージをデプロイするときにユーザーが使用する VM イメージの **オファー** 名のセグメント。 このフィールドではスペースや他の特殊文字は使用できません。  
   - **sku**  
     例: `14.04.3-LTS`  
     VM イメージをデプロイするときにユーザーが使用する VM イメージの **SKU** 名のセグメント。 このフィールドではスペースや他の特殊文字は使用できません。  
   - **version**  
     例: `1.0.0`  
     VM イメージをデプロイするときにユーザーが使用する VM イメージのバージョン。 このバージョンの形式は **\#.\#.\#** です。 このフィールドではスペースや他の特殊文字は使用できません。  

     **Remove-AzsPlatformImage** コマンドレットの詳細については、Microsoft PowerShell の [Azure Stack Hub オペレーター モジュールのドキュメント](/powershell/azure/azure-stack/overview)を参照してください。

---

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub Marketplace のカスタム アイテムを作成して発行する](azure-stack-create-and-publish-marketplace-item.md)
- [仮想マシンのプロビジョニング](../user/azure-stack-create-vm-template.md)