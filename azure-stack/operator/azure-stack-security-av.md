---
title: Windows Defender ウイルス対策を更新する
titleSuffix: Azure Stack Hub
description: Azure Stack Hub 上で Windows Defender ウイルス対策を更新する方法を説明します
author: justinha
ms.topic: article
ms.date: 12/04/2019
ms.author: justinha
ms.reviewer: fiseraci
ms.lastreviewed: 12/04/2019
ms.openlocfilehash: 31ef55554646065a5d61cc21abf4e72b7fcb4eda
ms.sourcegitcommit: e9a1dfa871e525f1d6d2b355b4bbc9bae11720d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86488095"
---
# <a name="update-windows-defender-antivirus-on-azure-stack-hub"></a>Azure Stack Hub 上で Windows Defender ウイルス対策を更新する

[Windows Defender ウイルス対策](/windows/security/threat-protection/windows-defender-antivirus/windows-defender-antivirus-in-windows-10)はセキュリティとウイルス対策を提供するマルウェア対策ソリューションです。 すべての Azure Stack Hub インフラストラクチャ コンポーネント (Hyper-V ホストと仮想マシン) は、Windows Defender ウイルス対策によって保護されています。 保護を最新に保つために、Windows Defender ウイルス対策の定義、エンジン、プラットフォームに定期的な更新が必要です。 更新を適用する方法は、構成によって変わります。

## <a name="connected-scenario"></a>接続されているシナリオ

Azure Stack Hub の[更新リソース プロバイダー](azure-stack-updates.md#the-update-resource-provider)によって、1 日に複数回、マルウェア対策の定義とエンジンの更新プログラムがダウンロードされます。 各 Azure Stack Hub インフラストラクチャ コンポーネントは、更新リソース プロバイダーから更新を取得し、それを自動的に適用します。

パブリック インターネットに接続されている Azure Stack Hub のデプロイの場合は、[Azure Stack Hub の月次更新](azure-stack-apply-updates.md)を適用します。 Azure Stack Hub の月次更新には、月ごとの Windows Defender ウイルス対策プラットフォームの更新が含まれています。

## <a name="disconnected-scenario"></a>切断されたシナリオ

パブリック インターネットに接続されていない Azure Stack Hub のデプロイ (エアギャップのあるデータ センターなど) では、1910 リリース以降、マルウェア対策の定義とエンジン更新プログラムを発行時に適用することができます。 

更新プログラムを Azure Stack Hub ソリューションに適用するには、最初に Microsoft サイト (以下のリンク) からそれらをダウンロードした後、*updateadminaccount* の下のストレージ BLOB コンテナーにインポートします。 スケジュールされたタスクにより、BLOB コンテナーが 30 分ごとにスキャンされます。新しい Defender の定義とエンジンの更新プログラムが見つかった場合は、それが Azure Stack Hub インフラストラクチャに適用されます。 

まだ 1910 以降を使用していない、または Defender の定義とエンジンの更新プログラムを毎日ダウンロードできない、切断されたデプロイの場合、Azure Stack Hub の月次更新プログラムには、Windows Defender ウイルス対策のその月の定義、エンジン、プラットフォームの更新プログラムが含まれます。 


### <a name="set-up-windows-defender-for-manual-updates"></a>手動更新用に Windows Defender をセットアップする 

1910 リリースでは、Azure Stack Hub 内で Windows Defender の手動更新を構成するために、2 つの新しいコマンドレットが特権エンドポイントに追加されました。 

```powershell 
### cmdlet to configure the storage blob container for the Defender updates 
Set-AzsDefenderManualUpdate [-Container <string>] [-Remove]  
### cmdlet to retrieve the configuration of the Defender manual update settings 
Get-AzsDefenderManualUpdate  
``` 

次の手順は、Windows Defender の手動更新を設定する方法を示しています。 

1. 特権エンドポイントに接続し、次のコマンドレットを実行して、Defender の更新プログラムがアップロードされるストレージ BLOB コンテナーの名前を指定します。 

   > [!NOTE] 
   > 以下で説明する手動更新プロセスは、"go.microsoft.com" へのアクセスが許可されていない切断された環境でのみ機能します。 接続された環境でコマンドレット Set-AzsDefenderManualUpdate を実行しようとすると、エラーが発生します。 

   ```powershell 
   ### Configure the storage blob container for the Defender updates 
   Set-AzsDefenderManualUpdate -Container <yourContainerName>
   ``` 

2. 2 つの Windows Defender 更新パッケージをダウンロードし、Azure Stack Hub 管理ポータルからアクセスできる場所に保存します。  

   * [https://go.microsoft.com/fwlink/?LinkId=121721&arch=x64](https://go.microsoft.com/fwlink/?LinkId=121721&arch=x64) からの mpam-fe.exe 
   * [https://go.microsoft.com/fwlink/?LinkId=197094](https://go.microsoft.com/fwlink/?LinkId=197094) からの nis_full.exe 

   > [!NOTE] 
   > この 2 つのファイルは、Defender の署名を更新する**たびに**ダウンロードする必要があります。 

3. 管理者ポータルで、 **[すべてのサービス]** を選択します。 次に、 **[DATA + STORAGE]\(データ + ストレージ\)** カテゴリで、 **[ストレージ アカウント]** を選択します (または、フィルター ボックスで「**ストレージ アカウント**」の入力を開始して選択します)。 

   ![Azure Stack Hub Defender - すべてのサービス](./media/azure-stack-security-av/image1.png)  

4. フィルター ボックスに「**更新**」と入力し、**updateadminaccount** ストレージ アカウントを選択します。 

5. ストレージ アカウントの詳細で、 **[サービス]** の **[BLOB]** を選択します。 

   ![Azure Stack Hub Defender - BLOB](./media/azure-stack-security-av/image2.png) 

6. **[Blob service]** で、 **[+ コンテナー]** を選択してコンテナーを作成します。 Set-AzsDefenderManualUpdate で指定した名前 (この例では *defenderupdates*) を入力し、 **[OK]** を選択します。 

   ![Azure Stack Hub Defender - コンテナー](./media/azure-stack-security-av/image3.png) 

7. コンテナーが作成されたら、コンテナー名をクリックした後、 **[アップロード]** をクリックしてパッケージ ファイルをコンテナーにアップロードします。 

   ![Azure Stack Hub Defender - アップロード](./media/azure-stack-security-av/image4.png) 

8. **[BLOB のアップロード]** で、フォルダー アイコンをクリックし、Windows Defender 更新プログラムの *mpam-fe.exe* ファイルを参照し、ファイル エクスプローラー ウィンドウで **[開く]** をクリックします。 

9. **[BLOB のアップロード]** で、 **[アップロード]** をクリックします。 

   ![Azure Stack Hub Defender - blob1 のアップロード](./media/azure-stack-security-av/image5.png) 

1. *nis_full.exe* ファイルに対して手順 8. と 9. を繰り返します。 

   ![Azure Stack Hub Defender - blob2 のアップロード](./media/azure-stack-security-av/image6.png)

スケジュールされたタスクにより、BLOB コンテナーが 30 分おきにスキャンされ、新しい Windows Defender パッケージが適用されます。  

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub のセキュリティについて詳しく学習する](azure-stack-security-foundations.md)
