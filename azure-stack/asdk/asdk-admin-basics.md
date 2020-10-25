---
title: ASDK 管理の基礎
description: Azure Stack Development Kit (ASDK) の基本的な管理作業を行う方法について説明します。
author: justinha
ms.topic: article
ms.date: 02/19/2019
ms.author: justinha
ms.reviewer: misainat
ms.lastreviewed: 10/15/2019
ms.openlocfilehash: c320b0fa20746da3e14a3cb4af2c8fcd1a9775ea
ms.sourcegitcommit: e9a1dfa871e525f1d6d2b355b4bbc9bae11720d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86489829"
---
# <a name="asdk-admin-basics"></a>ASDK 管理の基礎
Azure Stack Development Kit (ASDK) の管理経験がない場合は、知っておく必要があることがいくつかあります。 このガイダンスでは、評価環境での Azure Stack オペレーターとしての役割の概要を示します。 この情報をよく理解することで、テスト ユーザーの生産性が可能な限り迅速に向上することになります。

まず、「[What is Azure Stack Development Kit?](asdk-what-is.md)」(Azure Stack Development Kit とは) という記事に目を通し、ASDK の目的と制限事項について確実に理解しておく必要があります。 開発キットは "サンド ボックス" として使用する必要があります。その中では、Azure Stack を評価し、非運用環境でアプリの開発とテストを行うことができます。 

Azure と同様に、Azure Stack はイノベーションが迅速であるため、ASDK の新しいビルドを定期的にリリースします。 ただし、Azure Stack 統合システムのデプロイとは異なり、ASDK をアップグレードすることはできません。 そのため、最新のビルドに移行する場合は、[ASDK を再度デプロイする](asdk-redeploy.md)必要があります。 更新プログラム パッケージは適用できません。 このプロセスは時間がかかりますが、最新の機能が使用可能になったらすぐに試すことができるというメリットがあります。 

## <a name="what-account-should-i-use"></a>使用するアカウントについて
Azure Stack を管理する場合に注意するいくつかのアカウントに関する考慮事項があります。 これは特に、Azure Active Directory (Azure AD) ではなく ID プロバイダーとして Windows Server Active Directory Federation Services (AD FS) を使用するデプロイで当てはまります。 次のアカウントに関する考慮事項は、Azure Stack 統合システムと ASDK 展開の両方に適用されます。

|Account|Azure AD|AD FS|
|-----|-----|-----|
|ローカル管理者 (.\Administrator)|ASDK ホスト管理者|ASDK ホスト管理者|
|AzureStack\AzureStackAdmin|ASDK ホスト管理者<br><br>Azure Stack 管理者ポータルにサインインするために使用できます<br><br>Service Fabric リングを表示および管理するためのアクセス権|ASDK ホスト管理者<br><br>Azure Stack 管理者ポータルにはアクセスできません<br><br>Service Fabric リングを表示および管理するためのアクセス権<br><br>既定のプロバイダー サブスクリプション (DPS) の所有者ではなくなりました|
|AzureStack\CloudAdmin|特権エンドポイント内で許可されたコマンドにアクセス、実行できます|特権エンドポイント内で許可されたコマンドにアクセス、実行できます<br><br>ASDK ホストにサインインできません<br><br>既定のプロバイダー サブスクリプション (DPS) の所有者|
|Azure AD 全体管理者|インストール時に使用<br><br>既定のプロバイダー サブスクリプション (DPS) の所有者|適用なし|
|

## <a name="what-tools-do-i-use-to-manage"></a>管理に使用するツールについて
Azure Stack 管理者ポータル (`https://adminportal.local.azurestack.external`) または PowerShell を使って Azure Stack を管理できます。 基本的概念を学ぶためには、ポータルを介するのが最も容易な方法です。 PowerShell を使う場合は、[PowerShell for Azure Stack](asdk-post-deploy.md#install-azure-stack-powershell) をインストールし、[GitHub から Azure Stack ツールをダウンロードする](asdk-post-deploy.md#download-the-azure-stack-tools)必要があります。

Azure Stack はデプロイ、管理、整理のための基礎となるメカニズムとして、Azure Resource Manager を使用します。 Azure Stack の管理とユーザーのサポートを行う予定であれば、Azure Resource Manager について学習する必要があります。 詳しくは、「[Azure Resource Manager の概要](https://download.microsoft.com/download/E/A/4/EA4017B5-F2ED-449A-897E-BD92E42479CE/Getting_Started_With_Azure_Resource_Manager_white_paper_EN_US.pdf)」ホワイトペーパーをご覧ください。

## <a name="your-typical-responsibilities"></a>クラウド オペレーターの通常の担当範囲
ユーザーはサービスを使用する必要があります。 ユーザーの観点からは、クラウド オペレーターの主な役割は、ユーザーがこれらのサービスを使用できるようにすることです。 ASDK を使って、提供するサービスと、[プラン、オファー、クォータを作成する](../operator/tutorial-offer-services.md?view=azs-2002)ことで、それらのサービスを使用できるようにする方法を確認できます。 仮想マシン (VM) のイメージなど、マーケットプレースに項目を追加する必要もあります。 Azure から Azure Stack に[マーケットプレースの項目をダウンロードする](../operator/azure-stack-create-and-publish-marketplace-item.md)のが最も容易な方法です。

> [!NOTE]
> プラン、オファー、およびサービスをテストする場合は、管理者ポータル (`https://adminportal.local.azurestack.external`) ではなくユーザー ポータル (`https://portal.local.azurestack.external`) を使用する必要があります。

サービスを提供することに加えて、Azure Stack オペレーターとして Azure を稼働させ続けるという通常の業務にすべて対応する必要があります。 これらの業務には、次のものが含まれます。
- Azure AD または AD FS デプロイ用のユーザー アカウントを追加します。
- ロールベースのアクセス制御 (RBAC) のロールを割り当てます (これは管理者に限られません)。
- インフラストラクチャの正常性を監視します。
- ネットワークとストレージのリソースを管理します。
- 開発キットのホスト コンピューターで問題のあるハードウェアを交換します。

## <a name="where-to-get-support"></a>サポートが受けられる場所
ASDK については、[Azure Stack MSDN フォーラム](https://social.msdn.microsoft.com/Forums/azure/home?forum=azurestack)でサポート関連の質問をすることができます。

また、管理者ポータルの右上隅の **[ヘルプ]** (疑問符) をクリックして、フォーラムにアクセスすることもできます。 次に、 **[ヘルプとサポート]** をクリックすると、フォーラムへのリンクが含まれるヘルプとサポートの **[概要]** が開きます。 MSDN フォーラムは定期的にチェックされています。  

> [!IMPORTANT]
> ASDK は評価環境であるため、Microsoft サポートを通じた正式なサポートは提供されていません。

## <a name="next-steps"></a>次のステップ
[ASDK をデプロイする](asdk-install.md)
