---
title: 資格情報を切り替える
description: Azure Stack Hub ラグドのスイッチの資格情報を更新する方法について説明します
author: troettinger
ms.author: thoroet
ms.service: azure-stack
ms.topic: article
ms.date: 10/14/2020
ms.reviewer: justinha
ms.lastreviewed: 10/14/2020
ms.openlocfilehash: a761544c8d514fd69364d917890d284ab06e9962
ms.sourcegitcommit: 50b362d531c2d35a3a935811fee71252971bd5d8
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/09/2020
ms.locfileid: "96941332"
---
# <a name="switch-credentials"></a>資格情報を切り替える

このトピックでは、すべてのスイッチに設定されている管理者 (admin) および簡易ネットワーク管理プロトコル (SNMP) の資格情報を変更する方法について説明します。 

## <a name="prerequisites"></a>前提条件

この手順を実行する前に:

- リモート デスクトップを使用して HLH に接続します。
- HLH 上にある PuTTY (通常は E:\Tools\Putty\putty.exe) を見つけます。 PuTTY が見つからない場合は、ダウンロードして HLH にコピーします。
- スイッチの現在の資格情報と新しい資格情報の両方を用意します。

## <a name="update-credentials-for-the-admin-and-enable-accounts"></a>管理者と有効化のアカウントの資格情報を更新します 

スケール ユニット (BMC、TOR1、および TOR2) の各スイッチについて:

1. HLH 上で PuTTY を使用し、現在の資格情報を使用してスイッチにログインします。 
1. 次のコマンドを実行します。\<new password\> を新しい管理者と有効化の資格情報に置き換えます。 
   ```ini
   enable
   configuration terminal
   username admin pass
   word <new_password> privilege 15
   enable password <new_password>
   ```
1. スイッチへの現在のセッションは開いたままにします。
1. HLH 上で PuTTY を使用し、新しい資格情報を使用してスイッチにログインします。
1. 次のコマンドを実行します。 有効化については、パスワードの入力が求められません。

   ```ini
   enable
   write
   dir flash:
   ```

1. スタートアップ構成が今日の日付であることを確認します。
1. 確認したら、このセッションと元のセッションの両方を閉じます。

## <a name="update-snmp-accounts"></a>SNMP アカウントを更新する

スケール ユニット (BMC、TOR1、および TOR2) の各スイッチについて:

1. HLH 上で PuTTY を使用してスイッチにログインします。
1. 次のコマンドを実行し、現在の SNMP 読み取りおよび書き込みのユーザーとグループを取得します。

   ```ini
   enable
   show run snmp | grep user
   ```

   次の例は、ユーザー、グループ名、パスワード ハッシュなど、返されるコンテンツの種類を示しています。

   ```ini
   snmp-server user <user> <group> 3 encrypted auth sha <password_hash>
   ```

1. 次のコマンドを実行し、\<user\> と \<group\> を前の手順で決定した詳細に置き換え、\<password\> を新しいパスワードに置き換えます。

   ```ini
   configuration terminal
   snmp-server user <user> <group> 3 auth sha <password_new>
   end
   write
   exit
   ```

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub でシークレットをローテーションする](../operator/azure-stack-rotate-secrets.md)