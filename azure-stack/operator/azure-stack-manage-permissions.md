---
title: ロールベースのアクセス制御を使用してアクセス許可を設定する
description: Azure Stack Hub で、ロールベースのアクセス制御 (RBAC) を使用してアクセス許可を設定する方法について説明します。
author: PatAltimore
ms.topic: article
ms.date: 12/23/2019
ms.author: patricka
ms.reviewer: thoroet
ms.lastreviewed: 12/23/2019
ms.openlocfilehash: 852aa0283e3bd88a196c4bcf9e56951142bc4d60
ms.sourcegitcommit: 733a22985570df1ad466a73cd26397e7aa726719
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2021
ms.locfileid: "97870122"
---
# <a name="set-access-permissions-using-role-based-access-control"></a>ロールベースのアクセス制御を使用してアクセス許可を設定する

Azure Stack Hub のユーザーは、サブスクリプション、リソース グループ、またはサービスの各インスタンスの閲覧者、所有者、または共同作成者になることができます。 たとえば、ユーザー A にはサブスクリプション 1 に対する閲覧者アクセス許可と、仮想マシン 7 に対する所有者アクセス許可がある可能性があります。

 - 閲覧者: ユーザーはすべてを閲覧できますが、変更を加えることはできません。
 - 共同作成者: ユーザーはリソースへのアクセス以外のすべてを管理できます。
 - 所有者: ユーザーは、リソースへのアクセスを含め、すべてを管理できます。
 - カスタム:ユーザーのリソースへのアクセスは、特定のものに制限されています。

 カスタム ロールの作成の詳細については、「[Azure リソースのカスタム ロール](/azure/role-based-access-control/custom-roles)」を参照してください。

## <a name="set-access-permissions-for-a-user"></a>ユーザーのアクセス権限の設定

1. 管理するリソースへの所有者アクセス許可があるアカウントでサインインします。
2. リソースのブレードで、 **[アクセス]** アイコン ![[アクセス] アイコンは、2 名の人物の頭と肩の輪郭線です。](media/azure-stack-manage-permissions/image1.png) をクリックします。
3. **[ユーザー]** ブレードで、 **[役割]** をクリックします。
4. **[役割]** ブレードで、 **[追加]** をクリックし、ユーザーのアクセス許可を追加します。

## <a name="set-access-permissions-for-a-universal-group"></a>ユニバーサル グループのアクセス許可を設定する 

> [!Note]
> Active Directory フェデレーション サービス (AD FS) にのみ適用されます。

1. 管理するリソースへの所有者アクセス許可があるアカウントでサインインします。
2. リソースのブレードで、 **[アクセス]** アイコン ![[アクセス] アイコンは、2 名の人物の頭と肩の輪郭線です。](media/azure-stack-manage-permissions/image1.png) をクリックします。
3. **[ユーザー]** ブレードで、 **[役割]** をクリックします。
4. **[役割]** ブレードで、 **[追加]** をクリックして、ユニバーサル グループ Active Directory グループのアクセス許可を追加します。

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub テナントの追加](azure-stack-add-new-user-aad.md)
