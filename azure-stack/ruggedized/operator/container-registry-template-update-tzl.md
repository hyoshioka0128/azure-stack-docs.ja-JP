---
title: Azure Stack Hub でコンテナー レジストリを更新する | Microsoft Docs
titleSuffix: Azure Stack
description: Azure Stack Hub でコンテナー レジストリを更新する方法を学習します。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/16/2020
ms.author: mabrigg
ms.reviewer: chasat
ms.lastreviewed: 12/17/2019
ms.openlocfilehash: f63f0d550a841902e1d7c27d9c7688a8b5373149
ms.sourcegitcommit: d719f148005e904fa426a001a687e80730c91fda
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/06/2021
ms.locfileid: "97910586"
---
# <a name="update-the-container-registry-in-azure-stack-hub"></a>Azure Stack Hub でコンテナー レジストリを更新する

Azure Stack Hub のユーザーは、以下の手順を使用して、コンテナー レジストリ デプロイを、より新しい AKS 基本イメージ SKU に更新できます。 コンテナー レジストリ テンプレートの VM とサービスは、すべての状態とコンテナーのイメージが BLOB ストレージに格納されるためステートレスです。 更新は、より新しいバージョンの AKS 基本イメージ VHD を使用してコンテナー レジストリ テンプレートをデプロイし、DNS を新しい VM に再指定するのと同じくらい簡単です。 新旧のコンテナー レジストリ テンプレートの DNS 値を更新する操作を行うと、値が伝達されているときに、ほんの短い間、断続的なレジストリ接続が発生します。

## <a name="prerequisites"></a>前提条件

### <a name="operator"></a>演算子

1.  Azure Stack Marketplace から最新の AKS 基本イメージを配信します。 AKS 基本イメージは、月 1 回のペースで更新されます。

> !["AKS Base Ubu" の検索結果が表示された [Azure から追加する] ページを示すスクリーンショット。](./media/container-registry-template-updating-tzl/image1.png)

### <a name="user"></a>User

1.  リソース グループ内のデプロイ レコードを参照して、コンテナー レジストリ テンプレートのデプロイに使用された AKS 基本イメージの SKU を確認し、 **[入力]** を選択します。

    ![[入力] ページを示すスクリーンショット。](./media/container-registry-template-updating-tzl/image2.png)

2.  **Get-VMImageSku** 関数を使用して、使用可能な AKS 基本イメージのより新しい SKU があるかどうかを確認します。コンテナー レジストリ テンプレート スクリプトから `Import-Module .\pre-reqs.ps1` が必要です。

    ```powershell  
    PS C:\azurestack-galler-master\registry\Scripts> Get-VMImageSku -Location Shanghai
    
    Skus                  
    ----                  
    aks-ubuntu-1604-201909
    aks-ubuntu-1604-201910 
    ```

## <a name="parameters-required"></a>必須のパラメーター

| パラメーター | 詳細 |
| --- | --- |
| ユーザー名 | VM にログインするためのユーザー名を指定します。 |
| SSH 公開キー | SSH プロトコルを使用して、VM での認証に使用する SSH 公開キーを指定します。 |
| サイズ | デプロイする VM のサイズを選択します。 |
| パブリック IP アドレス | この VM の IP アドレス (動的/静的) の名前と種類を指定します。 |
| ドメイン名のラベル | レジストリの DNS プレフィックスを指定します。 この FQDN 全体が、レジストリに対して作成された PFX 証明書の CN 値と一致している必要があります。 |
| レプリカ | 開始するコンテナー レプリカの数を指定します。 |
| イメージ SKU | デプロイに使用するイメージ SKU を指定します。 AKS 基本イメージに対して使用可能な SKU は、**Get-VMImageSku** PowerShell コマンドレットによって一覧表示されます。 |
| サービス プリンシパルのクライアント ID | 前のデプロイで使用したサービス プリンシパル (SPN) アプリ ID を指定します。 |
| サービス プリンシパルのパスワード/パスワードの確認 | 前のデプロイで使用した SPN アプリ ID シークレットを指定します。 |
| 既存の拡張ストレージ アカウントのリソース ID | 前のデプロイで使用したストレージ アカウントのリソース ID を指定します。 |
| 既存のバックエンド BLOB コンテナー | 前のデプロイで使用した BLOB コンテナーを指定します。 |
| PFX 証明書 Key Vault のリソース ID | 前のデプロイで使用した Microsoft Azure Key Vault のリソース ID を指定します。 |
| PFX 証明書 Key Vault のシークレット URL | 前のデプロイで使用した証明書 URL を指定します。 |
| PFX 証明書のサムプリント | 前のデプロイで使用した証明書のサムプリントを指定します。 |

## <a name="installation"></a>インストール

1.  コンテナー レジストリ テンプレートの新しいインスタンスを新しいリソース グループにインストールします。

    ![[Create Container Registry Template]\(コンテナー レジストリ テンプレートの作成\) - [基本] ページを示すスクリーンショット。](./media/container-registry-template-updating-tzl/image3.png)

2.  `Get-VMImage` スクリプトから最新の SKU 出力を指定して、仮想マシン構成内の初期インストールから一意の **dnsname** パラメーターを使用し、初期インストールと同じサービス プリンシパルとシークレットを使用します。

    ![[Create Container Registry Template]\(コンテナー レジストリ テンプレートの作成\) - [仮想マシンの構成] ページを示すスクリーンショット。](./media/container-registry-template-updating-tzl/image4.png)

3.  ストレージと Key Vault 構成の初期インストールと同じストレージおよび Key Vault パラメーターを使用します。

    ![[Create Container Registry Template]\(コンテナー レジストリ テンプレートの作成\) - [ストレージと Key Vault の構成] ページを示すスクリーンショット。](./media/container-registry-template-updating-tzl/image5.png)

1.  新しいコンテナー レジストリ テンプレートがデプロイされたら、最初のリソース グループに移動し、パブリック IP アドレス リソースを選択します。

    ![パブリック IP アドレス リソースの一覧を示すスクリーンショット。](./media/container-registry-template-updating-tzl/image6.png)

1.  パブリック IP アドレス リソース内で [構成] に移動し、新しくデプロイするリソースに使用できるように DNS 名のラベルを変更します。 DNS 名のラベルを変更し、 **[保存]** を選択すると、Container Registry への呼び出しが失敗するようになることに注意してください。

    ![[Static]\(静的\) が選択された [パブリック IP アドレス] リソース ページを示すスクリーンショット。](./media/container-registry-template-updating-tzl/image7.png)
    
    ![[DNS 名ラベル (オプション)] が強調表示された [パブリック IP アドレス] リソース ページを示すスクリーンショット。](./media/container-registry-template-updating-tzl/image8.png)

2.  コンテナー レジストリ テンプレートへの新しいインスタンスのデプロイに使用する新しいリソース グループに移動し、パブリック IP リソース、構成を選択してから、DNS 名のラベルを、元のデプロイで使用されている正しい名前 (この例では `myreg`) に更新し、 **[保存]** を選択します。

    ![元の DNS 名ラベルが入力された [パブリック IP アドレス] リソース ページを示すスクリーンショット。](./media/container-registry-template-updating-tzl/image9.png)
    
    ![コンテナー レジストリ テンプレート](./media/container-registry-template-updating-tzl/image10.png)

3.  次の 30分間、DNS レコードが伝達されると、コンテナー レジストリへの断続的なアクセスが発生します。 Docker レジストリにログインし、イメージをプル/プッシュすることで、接続を検証します。

## <a name="next-steps"></a>次のステップ

[Azure Stack Marketplace の概要](../../operator/azure-stack-marketplace.md)
