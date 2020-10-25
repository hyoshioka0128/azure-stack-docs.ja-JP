---
title: Azure Stack Hub でマネージド ディスクの Resource Manager テンプレートを使用する
description: Azure Resource Manager テンプレートを使用する際のマネージド ディスクとアンマネージド ディスクの違いについて説明します。
author: sethmanheim
ms.author: sethm
ms.date: 8/25/2020
ms.topic: conceptual
ms.reviewer: wellsluo
ms.lastreviewed: 8/25/2020
ms.openlocfilehash: 2d2f274fffd1a07857b79255587b659003a3a1ce
ms.sourcegitcommit: 65a115d1499b5fe16b6fe1c31cce43be21d05ef8
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/25/2020
ms.locfileid: "88823272"
---
# <a name="use-vm-managed-disks-templates"></a>VM マネージド ディスク テンプレートを使用する

この記事では、Azure Resource Manager テンプレートを使って Azure Stack Hub で仮想マシンをプロビジョニングする際のマネージド ディスクとアンマネージド ディスクの違いについて説明します。 以下の例は、アンマネージド ディスクが使用されている既存のテンプレートをマネージド ディスクに変換する際にお役立てください。

## <a name="unmanaged-disks-template-formatting"></a>アンマネージド ディスクのテンプレートの書式

最初に、非管理対象ディスクのデプロイ方法について見てみましょう。 非管理対象ディスクを作成するときは、VHD ファイルを保持するストレージ アカウントが必要です。 ストレージ アカウントは新たに作成しても、既にあるものを使用してもかまいません。 次に示すように、テンプレートのリソース ブロックで新しいストレージ アカウント リソースを作成します。

```json
{
    "type": "Microsoft.Storage/storageAccounts",
    "apiVersion": "2017-10-01",
    "name": "[variables('storageAccountName')]",
    "location": "[resourceGroup().location]",
    "sku": {
        "name": "Standard_LRS"
    },
    "kind": "Storage"
}
```

仮想マシン オブジェクト内で、ストレージ アカウントへの依存関係を追加します。ストレージ アカウントが仮想マシンより前に作成されるようにするためです。 `storageProfile` セクション内に、VHD がある場所の完全 URI を指定します。この URI はストレージ アカウントを参照するもので、OS ディスク用とデータ ディスク用に必要となります。 次の例では、イメージから 1 つの OS ディスクと、1023GB のサイズの 1 つの空のデータ ディスクを作成します。

```json
{
    "type": "Microsoft.Compute/virtualMachines",
    "apiVersion": "2017-12-01",
    "name": "[variables('vmName')]",
    "location": "[resourceGroup().location]",
    "dependsOn": [
    "[resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
    "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
    ],
    "properties": {
        "hardwareProfile": {...},
        "osProfile": {...},
        "storageProfile": {
            "imageReference": {
                "publisher": "MicrosoftWindowsServer",
                "offer": "WindowsServer",
                "sku": "[parameters('windowsOSVersion')]",
                "version": "latest"
            },
            "osDisk": {
                "name": "osdisk",
                "vhd": {
                    "uri": "[concat(reference(resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))).primaryEndpoints.blob, 'vhds/osdisk.vhd')]"
                },
                "caching": "ReadWrite",
                "createOption": "FromImage"
            },
            "dataDisks": [
                {
                    "name": "datadisk1",
                    "diskSizeGB": 1023,
                    "lun": 0,
                    "vhd": {
                        "uri": "[concat(reference(resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))).primaryEndpoints.blob, 'vhds/datadisk1.vhd')]"
                    },
                    "createOption": "Empty"
                }
            ]
        },
        "networkProfile": {...},
        "diagnosticsProfile": {...}
    }
}
```

## <a name="managed-disks-template-formatting"></a>マネージド ディスクのテンプレートの書式

Azure マネージド ディスクを使用した場合、そのディスクが最上位のリソースとなり、ユーザーによって作成されるストレージ アカウントは不要となります。 マネージド ディスクは `2017-03-30` API バージョンで初めて導入されました。 以降のセクションでは、既定の設定とディスクのカスタマイズ方法について説明します。

### <a name="default-managed-disk-settings"></a>マネージド ディスクの既定の設定

今後、マネージド ディスクを使用して VM を作成する場合、ストレージ アカウント リソースを作成する必要はありません。 以下のテンプレート例では、前出のアンマネージド ディスクの例とは、いくつかの違いがあります。

- `apiVersion` は、マネージド ディスクをサポートする "virtualMachines" のリソースの種類のバージョンです。
- `osDisk` と `dataDisks` は、VHD の具体的な URI を参照していません。
- 追加のプロパティを指定せずにデプロイすると、ディスクでは VM のサイズに基づいてストレージの種類が使用されます。 たとえば、Premium Storage をサポートする VM サイズ (Standard_DS2_v2 など、名前に "s" が含まれるサイズ) を使用する場合、既定で Premium ディスクが構成されます。 これは、ディスクの sku 設定でストレージの種類を指定することによって変更できます。
- ディスクの名前を指定しなかった場合、OS ディスクには `<VMName>_OsDisk_1_<randomstring>` 形式の名前が、各データ ディスクには `<VMName>_disk<#>_<randomstring>` 形式の名前が付きます。
  - カスタム イメージから VM を作成する場合、ストレージ アカウントの種類とディスクの名前に使用される既定の設定は、カスタム イメージ リソースに定義されているディスクのプロパティから取得されます。 これらは、テンプレートで対応する値を指定することでオーバーライドすることができます。
- 既定では、ディスク キャッシュが、OS ディスクでは**読み取り/書き込み**に、データ ディスクでは**なし**になっています。
- 下の例を見るとわかるように、ストレージ アカウントの依存関係は依然として存在します。ただし、これはあくまで診断のストレージ用であって、ディスク ストレージに必要なものではありません。

```json
{
    "type": "Microsoft.Compute/virtualMachines",
    "apiVersion": "2017-12-01",
    "name": "[variables('vmName')]",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
        "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
    ],
    "properties": {
        "hardwareProfile": {...},
        "osProfile": {...},
        "storageProfile": {
            "imageReference": {
                "publisher": "MicrosoftWindowsServer",
                "offer": "WindowsServer",
                "sku": "[parameters('windowsOSVersion')]",
                "version": "latest"
            },
            "osDisk": {
                "createOption": "FromImage"
            },
            "dataDisks": [
                {
                    "diskSizeGB": 1023,
                    "lun": 0,
                    "createOption": "Empty"
                }
            ]
        },
        "networkProfile": {...},
        "diagnosticsProfile": {...}
    }
}
```

### <a name="use-a-top-level-managed-disk-resource"></a>最上位のマネージド ディスク リソースを使用する

仮想マシン オブジェクトにディスク構成を指定する代わりに、最上位のディスク リソースを作成して、それを仮想マシン作成の過程でアタッチすることもできます。 必ず、`disks` リソース API バージョンとして `2017-03-30` を使用してください。 たとえば、データ ディスクとして使用するディスク リソースは次のように作成できます。 この例では、ディスク名の一部として `vmName` が使用されています。

```json
{
    "type": "Microsoft.Compute/disks",
    "apiVersion": "2017-03-30",
    "name": "[concat(variables('vmName'),'-datadisk1')]",
    "location": "[resourceGroup().location]",
    "sku": {
        "name": "Standard_LRS"
    },
    "properties": {
        "creationData": {
            "createOption": "Empty"
        },
        "diskSizeGB": 1023
    }
}
```

VM オブジェクト内で、アタッチするディスク オブジェクトを参照します。 作成した管理ディスクのリソース ID を `managedDisk` プロパティに指定しておけば、VM が作成されるときにディスクがアタッチされます。 VM リソースの `apiVersion` は `2017-12-01` に設定されています。 VM の作成前にディスク リソースが作成されるように、ディスク リソースへの依存関係が追加されます。

```json
{
    "type": "Microsoft.Compute/virtualMachines",
    "apiVersion": "2017-12-01",
    "name": "[variables('vmName')]",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
        "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]",
        "[resourceId('Microsoft.Compute/disks/', concat(variables('vmName'),'-datadisk1'))]"
    ],
    "properties": {
        "hardwareProfile": {...},
        "osProfile": {...},
        "storageProfile": {
            "imageReference": {
                "publisher": "MicrosoftWindowsServer",
                "offer": "WindowsServer",
                "sku": "[parameters('windowsOSVersion')]",
                "version": "latest"
            },
            "osDisk": {
                "createOption": "FromImage"
            },
            "dataDisks": [
                {
                    "lun": 0,
                    "name": "[concat(variables('vmName'),'-datadisk1')]",
                    "createOption": "attach",
                    "managedDisk": {
                        "id": "[resourceId('Microsoft.Compute/disks/', concat(variables('vmName'),'-datadisk1'))]"
                    }
                }
            ]
        },
        "networkProfile": {...},
        "diagnosticsProfile": {...}
    }
}
```

### <a name="create-managed-availability-sets-with-vms-using-managed-disks"></a>マネージド ディスクを使った VM で構成される管理された可用性セットを作成する

マネージド ディスクを使った VM で構成される管理された可用性セットを作成するには、可用性セット リソースに `sku` オブジェクトを追加し、その `name` プロパティを `Aligned` に設定します。 このプロパティにより、各 VM のディスクが単一障害点とならないよう互いに分離されます。 また、可用性セット リソース の `apiVersion` は `2017-12-01` に設定されています。

```json
{
    "type": "Microsoft.Compute/availabilitySets",
    "apiVersion": "2017-12-01",
    "location": "[resourceGroup().location]",
    "name": "[variables('avSetName')]",
    "properties": {
        "PlatformUpdateDomainCount": 1,
        "PlatformFaultDomainCount": 2
    },
    "sku": {
        "name": "Aligned"
    }
}
```

## <a name="next-steps"></a>次のステップ

<!--
* For full templates that use managed disks visit the following Azure Quickstart Repo links.
    * [Windows VM with managed disk](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-windows)
    * [Linux VM with managed disk](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-linux)
-->

- マネージド ディスクの詳細については、[Azure Stack Hub マネージド ディスク](azure-stack-managed-disk-considerations.md)に関する記事を参照してください。
- [Microsoft.Compute/virtualMachines テンプレート リファレンス](/azure/templates/microsoft.compute/2017-12-01/virtualmachines) で、仮想マシン リソースのテンプレート リファレンス ドキュメントを参照してください。
- [Microsoft.Compute/disks テンプレート リファレンス](/azure/templates/microsoft.compute/2017-03-30/disks)で、ディスク リソースのテンプレート リファレンス ドキュメントを参照してください。
