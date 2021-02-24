---
title: 特権アクセス ワークステーションと特権エンドポイント アクセス
description: 特権アクセス ワークステーションと特権エンドポイント アクセスについて学習します
author: PatAltimore
ms.topic: how-to
ms.date: 11/13/2020
ms.author: patricka
ms.reviewer: ''
ms.lastreviewed: ''
ms.openlocfilehash: 96a4e944b6b86c8b5db314141fd3473a8512d518
ms.sourcegitcommit: 9b0e1264ef006d2009bb549f21010c672c49b9de
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2021
ms.locfileid: "98256202"
---
# <a name="privileged-access-workstation-and-privileged-endpoint-access"></a>特権アクセス ワークステーションと特権エンドポイント アクセス

## <a name="overview"></a>概要

この手順では、特権アクセス ワークステーション (PAW) に接続する必要があります。 顧客は、リモート デスクトップを使用して PAW に接続する機能を提供する必要があります。

## <a name="configuring-the-winrm"></a>WinRM の構成

PAW から特権エンドポイントへの接続を許可するには、Azure Stack Hub 管理ポータルで定義されている特権エンドポイントの IP アドレスが、PAW の信頼されたホストとして設定されていることを確実にします。 これらの IP アドレスを管理者ポータルから取得する手順は、16 ページのスケール ユニット ノードのアクセスと正常性の検証にあります。

WinRM の信頼されたホストを表示または編集するには、管理者特権で次の PowerShell セッションを起動します。

-   信頼されたホストの表示

現在信頼されているホストを表示するには、PowerShell で次を実行します。

-   信頼されたホストの編集

Emergency Recovery Console Server (ERCS) の IP が存在しない場合は、次を実行して信頼されたホストに新しい値を設定します。*\<ERCS01_IP\*、*\<ERCS02_IP\*、*\<ERCS03_IP\* を Azure Stack Hub 管理ポータル内で定義されている 3 つの特権エンドポイント IP に置き換えます。

## <a name="connect-to-the-privileged-endpoint"></a>特権エンドポイントに接続する

PAW 上で、管理者特権で PowerShell セッションを開き、次の 2 つのコマンドを実行します。 *\<ERCS_IP\* を、この手順で前述したいずれかの特権エンドポイント インスタンスの IP に置き換えます。 指示に従い、顧客から提供された特権エンドポイント (PEP) の資格情報を入力します。

## <a name="close-the-privileged-endpoint"></a>特権エンドポイントを閉じる

特権エンドポイント セッションを閉じるには、次を実行します。

## <a name="further-reading"></a>参考資料

特権エンドポイントに接続して使用する方法については、「[Azure Stack Hub での特権エンドポイントの](../../operator/azure-stack-privileged-endpoint.md)
[使用](../../operator/azure-stack-privileged-endpoint.md)」を参照してください。