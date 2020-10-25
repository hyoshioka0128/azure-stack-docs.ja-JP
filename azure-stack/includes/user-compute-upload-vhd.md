---
author: mattbriggs
ms.author: mabrigg
ms.service: azure-stack
ms.topic: include
ms.date: 08/04/2020
ms.reviewer: thoroet
ms.lastreviewed: 08/04/2020
ms.openlocfilehash: 28fe06ca182b2df8104c2d8b23ce9f5c27448064
ms.sourcegitcommit: 362081a8c19e7674c3029c8a44d7ddbe2deb247b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/09/2020
ms.locfileid: "91936089"
---
ポータルを使用して、またはポータルで作成したコンテナーを使用して VHD をアップロードすること、AzCopy を使用することができます。

### <a name="portal-to-generate-sas-url-and-upload-vhd"></a>SAS URL を生成して VHD をアップロードするためのポータル

1. Azure Stack Hub のユーザー ポータルにサインインします。

2. **[ストレージ アカウント]** を選択し、既存のストレージ アカウントを選択するか、新しいストレージ アカウントを作成します。

3. ストレージ アカウントの [ストレージ アカウント] ブレードで **[BLOB]** を選択します。 [コンテナー] を選択して、新しいコンテナーを作成します。

4. コンテナーの名前を入力し、 **[BLOB (BLOB 専用の匿名読み取りアクセスのみ)]** を選択します。

5. ポータルではなく AzCopy を使用してイメージをアップロードする場合は、SAS トークンを作成します。 ストレージ アカウントで **[共有アクセス署名]** を選択し、 **[SAS と接続文字列を生成する]** を選択します。 **[Blob service の SAS URL]** をコピーしてメモを作成します。 AzCopy を使用して VHD をアップロードするときに、この URL を使用します。

6. コンテナーを選択し、 **[アップロード]** を選択します。 VHD をアップロードします。

### <a name="azcopy-vhd"></a>VHD を AzCopy する

Azure Storage Explorer または AzCopy を使用すると、アップロード プロセスで VHD が破損する可能性が減少し、アップロードが高速になります。 次の手順では、Windows 10 コンピューターで AzCopy を使用します。 AzCopy は、ストレージ アカウント間の BLOB またはファイル コピーに利用できるコマンドライン ユーティリティです。

1. AzCopy がインストールされていない場合はインストールしてください。 AzCopy をダウンロードして使用開始する手順については、記事「[AzCopy を使ってみる](/azure/storage/common/storage-use-azcopy-v10)」を参照してください。 バイナリを保存する場所をメモします。 [AzCopy をパスに追加](https://www.architectryan.com/2018/03/17/add-to-the-path-on-windows-10/)して、PowerShell コマンド ラインから使用できるようにします。

2. PowerShell を開いて、シェルから AzCopy を使用します。

3. AzCopy を使用して、ストレージ アカウントのコンテナーに VHD をアップロードします。

    ```powershell  
    set AZCOPY_DEFAULT_SERVICE_API_VERSION=2017-11-09
    azcopy cp "/path/to/file.vhd" "https://[account].blob.core.windows.net/[container]/[path/to/blob]?[SAS] --blob-type=PageBlob
    ```

> [!NOTE]  
> 1 つのファイルを仮想ディレクトリにアップロードする場合と同様の構文を使用して VHD をアップロードします。 `--blob-type=PageBlob` を追加して、VHD が**ブロック**ではなく既定で**ページ BLOB** としてアップロードされるようにします。

AzCopy とその他のストレージ ツールの使用の詳細については、「[Azure Stack Hub Storage でのデータ転送ツールの使用](../user/azure-stack-storage-transfer.md)」を参照してください。