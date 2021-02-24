---
author: BryanLa
ms.author: bryanla
ms.service: azure-stack
ms.topic: include
ms.date: 12/09/2019
ms.reviewer: bryanla
ms.lastreviewed: 12/09/2019
ms.openlocfilehash: 2a980becc3bf749987759f90a9ab84050d033a04
ms.sourcegitcommit: 81e2d627c9dc4cc365deb4a0e0674b5ab3a7efbf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/21/2020
ms.locfileid: "92297851"
---
既にリソース プロバイダーをインストールしている場合は、次の前提条件を満たしていると考えられるため、このセクションは省略してもかまいません。 それ以外の場合は、続ける前に次の手順を完了してください。 

1. まだの場合は、[Azure Stack Hub インスタンスを Azure に登録します](../operator/azure-stack-registration.md)。 この手順は、Azure から Marketplace に接続して項目をダウンロードする際に必要になります。

2. Azure Stack Hub 管理者ポータルの **Marketplace Management** 機能に慣れていない場合は、 [Azure から Marketplace 項目をダウンロードして Azure Stack Hub に発行する方法](../operator/azure-stack-download-azure-marketplace-item.md)に関するページを確認してください。 この記事では、Azure から Azure Stack Hub Marketplace に項目をダウンロードする手順について説明しています。 接続されている場合と接続されていない場合、両方のシナリオについての説明があります。 Azure Stack Hub インスタンスが切断されているか、部分的にしか接続されていない場合は、インストールの準備として追加の前提条件を満たす必要があります。

3. Azure Active Directory (Azure AD) ホーム ディレクトリを更新します。 ビルド 1910 以降の場合、ホーム ディレクトリ テナントに新しいアプリケーションを登録する必要があります。 このアプリを使用すると、Azure Stack Hub で新しいリソース プロバイダー (Event Hubs、IoT Hub など) を正常に作成し、Azure AD テナントに登録することができます。 これは、ビルド 1910 以降にアップグレードした後で実行する必要がある 1 回限りの操作です。 このステップを完了しないと、マーケットプレースのリソース プロバイダーのインストールが失敗します。 

   - Azure Stack Hub インスタンスが 1910 以上に正常に更新されたら、[Azure Stack Hub Tools リポジトリのクローンまたはダウンロードに関する手順](../operator/azure-stack-powershell-download.md)に従います。 
   - 次に、[(リソース プロバイダーの更新プログラムまたは新しいリソース プロバイダーのインストール後の) Azure Stack Hub Azure AD ホーム ディレクトリの更新](https://github.com/Azure/AzureStack-Tools/tree/master/Identity#updating-the-azure-stack-aad-home-directory-after-installing-updates-or-new-resource-providers)に関するページの指示に従います。 