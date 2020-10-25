---
title: Azure Stack Hub の Key Vault の概要
description: Azure Stack Hub での Key Vault によるキーとシークレットの管理方法について説明します
author: sethmanheim
ms.topic: conceptual
ms.date: 08/10/2020
ms.author: sethm
ms.lastreviewed: 08/10/2020
ms.openlocfilehash: 7387778055a50b138766cc7f8c416b206f4b9db9
ms.sourcegitcommit: 77f53d8f4188feea7dd2197650ee860104b1e2aa
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/17/2020
ms.locfileid: "88501077"
---
# <a name="introduction-to-key-vault-in-azure-stack-hub"></a>Azure Stack Hub の Key Vault の概要

## <a name="prerequisites"></a>前提条件

* Azure Key Vault サービスを含むオファーをサブスクライブします。  
* PowerShell をインストールし、[Azure Stack Hub で使うように構成します](azure-stack-powershell-configure-user.md)。

## <a name="key-vault-basics"></a>Key Vault の基礎

Azure Stack Hub の Key Vault は、クラウド アプリやクラウド サービスで使用される暗号化キーとシークレットを保護するのに役立ちます。 Key Vault を使用すると、キーやシークレットを暗号化することができます。その例を次に示します。

* 認証キー
* ストレージ アカウント キー
* データ暗号化キー
* .pfx ファイル
* パスワード

Key Vault は、キー管理プロセスを合理化し、データにアクセスして暗号化するキーの制御を維持できます。 開発者は、開発やテスト用のキーを数分で作成し、それらをシームレスに実稼働キーに移行できます。 セキュリティ管理者は、必要に応じて、キーに権限を付与する (取り消す) ことができます。

Azure Stack Hub サブスクリプションを持つユーザーはだれでも、キー コンテナーを作成して使用することができます。 Key Vault は開発者とセキュリティ管理者にとってメリットがありますが、組織の他の Azure Stack Hub サービスを管理するオペレーターが実装して管理できます。 たとえば、Azure Stack Hub オペレーターは、Azure Stack Hub サブスクリプションを使用してサインインし、キーを格納する組織用のコンテナーを作成できます。 完了すると、次のことができるようになります。

* キーやシークレットの作成やインポート。
* キーやシークレットの取り消しや削除。
* ユーザーまたはアプリがキーとシークレットを管理または使用できるように、キー コンテナーへのアクセスを許可する。
* キーの使用状況の構成 (署名や暗号化など)。

オペレーターは、アプリから呼び出す Uniform Resource Identifier (URI) を開発者に提供できるようになります。

開発者は、API を使用してキーを直接管理することもできます。 詳細については、「[Key Vault 開発者ガイド](/azure/key-vault/key-vault-developers-guide)」を参照してください。

## <a name="scenarios"></a>シナリオ

次のシナリオでは、Key Vault が開発者やセキュリティ管理者のニーズを満たすのに役立つシナリオについて説明します。

### <a name="developer-for-an-azure-stack-hub-app"></a>Azure Stack Hub アプリの開発者

**問題**:キーを使って署名と暗号化を行う Azure Stack Hub アプリを作成したい。 ソリューションが地理的に分散したアプリに合うように、これらのキーをアプリの外部に設定したい。

**説明:** キーはコンテナーに格納され、必要に応じて URI で呼び出されます。

### <a name="developer-for-software-as-a-service-saas"></a>サービスとしてのソフトウェア (SaaS) の開発者

**問題**:顧客のキーやシークレットに対して義務や潜在的責任を負いたくない。 顧客がキーを自分で所有して管理すれば、オペレーターは自分の専門分野である、中心的なソフトウェア機能を提供することに集中できます。

**説明:** 顧客は Azure Stack Hub で独自のキーをインポートして管理できます。

### <a name="chief-security-officer-cso"></a>最高セキュリティ責任者 (CSO)

**問題**: 組織が、キーのライフ サイクルを管理し、キーの使用状況を確実に監視できるようにしたい。

**説明:** Key Vault は、Microsoft がキーを確認または抽出しないように作られています。 アプリが顧客のキーを使って暗号化操作を実行する必要がある場合は、Key Vault がアプリに代わってこれを行います。 アプリからは顧客のキーが見えません。 複数の Azure Stack Hub サービスとリソースが使用されていますが、Azure Stack Hub の 1 つの場所からキーを管理することができます。 コンテナーでは、Azure Stack Hub にあるコンテナーの数、サポートするリージョン、使用するアプリに関係なく、1 つのインターフェイスが提供されます。

## <a name="next-steps"></a>次のステップ

* [ポータルを使用して Azure Stack Hub の Key Vault を管理する](azure-stack-key-vault-manage-portal.md)  
* [PowerShell を使用して Azure Stack Hub の Key Vault を管理する](azure-stack-key-vault-manage-powershell.md)
