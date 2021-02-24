---
title: Azure Stack Hub で Windows 仮想マシンを実行する
description: Azure Stack Hub で Windows 仮想マシンを実行する方法について学習します。
author: mattbriggs
ms.topic: how-to
ms.date: 2/1/2021
ms.author: mabrigg
ms.reviewer: kivenkat
ms.lastreviewed: 3/9/2020
ms.openlocfilehash: 5bbad22a6a0998cd56fd3fefc309c14f665b2faa
ms.sourcegitcommit: a6f62a6693e48eb05272c01efb5ca24372875173
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/02/2021
ms.locfileid: "99247542"
---
# <a name="run-a-windows-virtual-machine-on-azure-stack-hub"></a>Azure Stack Hub で Windows 仮想マシンを実行する

Azure Stack Hub での仮想マシン (VM) のプロビジョニングには、VM 自体の他に、ネットワーク リソースやストレージ リソースなどの追加コンポーネントがいくつか必要です。 この記事では、Azure 上で Windows VM を実行するためのベスト プラクティスを示します。

![Azure Stack Hub の Windows VM のアーキテクチャ](./media/iaas-architecture-vm-windows/image1.png)

## <a name="resource-group"></a>Resource group

[リソース グループ](/azure/azure-resource-manager/resource-group-overview)は、関連する Azure Stack Hub リソースを保持する論理コンテナーです。 一般には、リソースの有効期間や管理者に基づいて、リソースをグループ化します。

同じライフサイクルを共有する密接に関連付けられたリソースを同じ[リソース グループ](/azure/azure-resource-manager/resource-group-overview)に配置します。 リソース グループを使用すると、リソースをグループとしてデプロイおよび監視したり、リソース グループ別に課金コストを追跡したりできます。 セットとしてリソースを削除することもできます。これはテスト デプロイの場合に便利です。 特定のリソースの検索やその役割の理解を簡略化するために、意味のあるリソース名を割り当ててください。 詳細については、「[Azure リソースの推奨される名前付け規則](/azure/architecture/best-practices/naming-conventions)」をご覧ください。

## <a name="virtual-machine"></a>仮想マシン

VM は、発行されたイメージの一覧や、Azure Stack Hub BLOB ストレージにアップロードされたカスタムのマネージド イメージまたは仮想ハード ディスク (VHD) ファイルからプロビジョニングできます。

Azure Stack Hub には、Azure とは異なる仮想マシン サイズが用意されています。 詳細については、[Azure Stack Hub の仮想マシンのサイズ](./azure-stack-vm-sizes.md)に関するページを参照してください。 既存のワークロードを Azure Stack Hub に移動する場合は、オンプレミスのサーバーまたは Azure に最も適合性が高い VM サイズから開始します。 次に、CPU、メモリ、およびディスクの 1 秒あたりの入出力操作 (IOPS) について、実際のワークロードのパフォーマンスを測定し、必要に応じてサイズを調整します。

## <a name="disks"></a>ディスク

コストは、プロビジョニングされたディスクの容量に基づきます。 IOPS とスループット (つまり、データ転送速度) は VM サイズによって異なるため、ディスクをプロビジョニングする場合は、3 つの要素 (容量、IOPS、スループット) すべてを考慮してください。

Azure Stack Hub のディスク IOPS (1 秒あたりの入出力操作数) は、ディスクの種類ではなく [VM サイズ](./azure-stack-vm-sizes.md)の関数です。 つまり、Standard_Fs シリーズの VM では、ディスクの種類として SSD と HDD のどちらを選択した場合でも、1 つの追加データ ディスクの IOPS 制限は 2,300 IOPS です。 課される IOPS 制限は、周囲へのノイズを防ぐための上限 (可能な最大値) です。 これは特定の VM サイズで得られる IOPS を保証するものではありません。

[Managed Disks](./azure-stack-managed-disk-considerations.md) を使用することもお勧めします。 マネージド ディスクでは、ストレージが自動的に処理されることでディスク管理が簡素化されます。 マネージド ディスクでは、ストレージ アカウントは必要ありません。 単にディスクのサイズと種類を指定するだけで、可用性の高いリソースとしてデプロイされます。

OS ディスクは Azure Stack Hub BLOB ストレージに格納された VHD であるため、ホスト マシンが停止している場合でも維持されます。 また、[データ ディスク](./azure-stack-manage-vm-disks.md)を 1 つ以上作成することもお勧めします。データ ディスクは、アプリケーション データ用に使用される永続的な VHD です。 可能であれば、OS ディスクではなく、データ ディスクにアプリケーションをインストールします。 一部のレガシー アプリケーションでは、C: ドライブへのコンポーネントのインストールが必要になることがあります。その場合は、PowerShell を使用して [OS ディスクのサイズを変更](/azure/virtual-machines/windows/expand-os-disk)できます。

VM も一時ディスク (Windows の D: ドライブ) を使用して作成されます。 このディスクは、Azure Stack Hub ストレージ インフラストラクチャの一時ボリュームに格納されます。 再起動やその他の VM ライフサイクル イベント中に削除される可能性があります。 ページ ファイルやスワップ ファイルなどの一時的なデータにのみ、このディスクを使用してください。

## <a name="network"></a>ネットワーク

ネットワーク コンポーネントには、次のリソースが含まれます。

-   **Virtual network**。 すべての VM が、複数のサブネットにセグメント化できる仮想ネットワーク内にデプロイされます。

-   **ネットワーク インターフェイス (NIC)** 。 NIC を使用すると、VM は仮想ネットワークと通信できます。 VM 用に複数の NIC が必要な場合は、[VM サイズ](./azure-stack-vm-sizes.md)ごとに NIC の最大数が定義されることに注意してください。

-   **パブリック IP アドレス/VIP**。 パブリック IP アドレスは、リモート デスクトップ (RDP) 経由などで VM と通信するために必要です。 パブリック IP アドレスは、動的でも静的でもかまいません。 既定では、動的アドレスになっています。

-   変化しない固定 IP アドレスが必要な場合 (たとえば、DNS "A" レコードを作成するか、またはセーフ リストに IP アドレスを追加する必要がある場合) は、[静的 IP アドレス](/azure/virtual-network/virtual-networks-reserved-public-ip)を予約します。

-   IP アドレスの完全修飾ドメイン名 (FQDN) を作成することもできます。 これにより、その FQDN を参照する DNS で [CNAME レコード](https://en.wikipedia.org/wiki/CNAME_record)を登録できます。 詳細については、[Azure portal での完全修飾ドメイン名の作成](/azure/virtual-machines/windows/portal-create-fqdn)に関する記事をご覧ください。

-   **ネットワーク セキュリティ グループ (NSG)** 。 NSG は、VM へのネットワーク トラフィックを許可または拒否するために使用されます。 NSG は、サブネットまたは個々の VM インスタンスと関連付けることができます。

すべての NSG に[既定の規則](/azure/virtual-network/security-overview#default-security-rules) (すべての受信インターネット トラフィックをブロックする規則など) のセットが含まれています。 既定のルールを削除することはできませんが、他の規則でオーバーライドすることはできます。 インターネット トラフィックを有効にするには、特定のポート (HTTP のポート 80 など) への着信トラフィックを許可するルールを作成します。 RDP を有効にするには、TCP ポート 3389 への着信トラフィックを許可する NSG ルールを追加します。

## <a name="operations"></a>操作

**診断** 基本的な正常性メトリック、診断インフラストラクチャ ログ、[ブート診断](https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/)などの監視と診断を有効にします。 VM が起動不可能な状態になった場合は、起動エラーを診断するのにブート診断が役立ちます。 ログを格納するための Azure Storage アカウントを作成します。 診断ログには、標準的なローカル冗長ストレージ (LRS) アカウントがあれば十分です。 詳細については、「[監視と診断の有効化](./azure-stack-metrics-azure-data.md)」を参照してください。

**可用性**。 Azure Stack Hub オペレーターによってスケジュールされた計画メンテナンスにより、VM が再起動される場合があります。 Azure でのマルチ VM による実稼働システムの高可用性を実現するため、VM は、複数の障害ドメインと更新ドメインに分散される[可用性セット](/azure/virtual-machines/windows/manage-availability#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy)に配置されます。 より小さいスケールの Azure Stack Hub では、可用性セット内の障害ドメインは、スケール ユニット内の 1 つのノードとして定義されます。  

Azure Stack Hub のインフラストラクチャは既に障害に対する回復性を備えていますが、基盤となっているテクノロジ (フェールオーバー クラスタリング) では、ハードウェアが故障した場合にその影響を受ける物理サーバー上の VM に多少のダウンタイムが発生します。 Azure Stack Hub では、Azure との一貫性がある最大 3 つの障害ドメインを持つ可用性セットを用意することをサポートしています。

|                   |             |
|-------------------|-------------|
| **障害ドメイン** | 可用性セットに配置された VM は、複数の障害ドメイン (Azure Stack Hub ノード) にできる限り均等に分散させることによって、互いに物理的に分離されます。 ハードウェア障害が発生した場合、障害が発生した障害ドメインの VM は、他の障害ドメインで再起動されます。 これらは、他の VM とは別の障害ドメインに、ただし可能な場合、同じ可用性セットに保持されます。 ハードウェアがオンラインに戻ると、高可用性を維持するために VM の再配置が行われます。 |
| **更新ドメイン**| 更新ドメインも、Azure によって可用性セットに高可用性が提供される方法です。 更新ドメインは、メンテナンスを同時に実行できる、基盤となるハードウェアの論理グループです。 同じ更新ドメイン内の VM は、計画済みメンテナンス中に同時に再起動されます。 テナントが可用性セット内に VM を作成すると、Azure プラットフォームは、これらの更新ドメインに VM を自動的に分散します。 <br>Azure Stack Hub では、VM のホストが更新される前に、クラスター内の他のオンライン ホストに VM がライブ マイグレーションされます。 ホスト更新の際にテナントのダウンタイムは発生しないため、Azure Stack Hub の更新ドメイン機能は、Azure とテンプレートの互換性を保つためにのみ存在します。 可用性セット内の VM では、ポータル上でその更新ドメイン番号として 0 が表示されます。 |

**バックアップ**。Azure Stack Hub IaaS VM の保護に関する推奨事項については、「[Azure Stack Hub にデプロイされた VM の保護](azure-stack-manage-vm-protect.md)」を参照してください。

**VM の停止**。 Azure では、"停止" 状態と "割り当て解除済み" 状態が区別されます。 VM が割り当て解除されたときではなく、VM が停止状態のときに課金されます。 Azure Stack Hub ポータルの **[停止]** ボタンを使用すると、VM の割り当てが解除されます。 ログイン中に OS からシャットダウンした場合、VM は停止しますが割り当ては "**解除されない**" ため、引き続き課金されます。

**VM の削除**。 VM を削除しても VM ディスクは削除されません。 つまり、データを失うことなく安全に VM を削除できます。 ただし、Storage に対して引き続き課金されます。 VM ディスクを削除するには、マネージド ディスク オブジェクトを削除します。 誤って削除されないようにするために、*リソース ロック* を使用してリソース グループ全体をロックするか、または個別のリソース (VM など) をロックします。

## <a name="security-considerations"></a>セキュリティに関する考慮事項

VM を [Azure Security Center](/azure/security-center/quick-onboard-azure-stack) にオンボードすると、Azure リソースのセキュリティの状態を一元的に表示して把握できます。 Security Center は、潜在的なセキュリティ上の問題を監視し、デプロイのセキュリティの正常性を包括的に示します。 セキュリティ センターは、Azure サブスクリプションごとに構成されます。 [Azure サブスクリプションでの Security Center Standard の利用開始](/azure/security-center/security-center-get-started)に関するページの説明に従って、セキュリティ データの収集を有効にします。 データ収集を有効にすると、セキュリティ センターは、そのサブスクリプションに作成されているすべての VM を自動的にスキャンします。

**更新プログラムの管理**。 VM で修正プログラムの管理を構成するには、[この](./vm-update-management.md)記事を参照してください。 有効になっている場合、Security Center はセキュリティ更新プログラムや緊急更新プログラムが不足しているかどうかをチェックします。 VM で[グループ ポリシー設定](/windows-server/administration/windows-server-update-services/deploy/4-configure-group-policy-settings-for-automatic-updates)を使用して、システムの自動更新を有効にします。

**マルウェア対策**。 有効な場合、セキュリティ センターは、マルウェア対策ソフトウェアがインストールされているかどうかを確認します。 セキュリティ センターを使用して、Azure Portal 内からマルウェア対策ソフトウェアをインストールすることもできます。

**アクセスの制御**。 [ロールベースのアクセス制御 (RBAC)](/azure/active-directory/role-based-access-control-what-is) を使用して、Azure リソースへのアクセスを制御します。 RBAC を使用すると、DevOps チームのメンバーに承認の役割を割り当てることができます。 たとえば、閲覧者の役割では、Azure リソースを表示することはできますが、作成、削除、または管理することはできません。 一部のアクセス許可は、Azure リソースの種類に固有です。 たとえば、仮想マシンの共同作業者ロールは VM を再起動または割り当て解除したり、管理者パスワードをリセットしたり、新しい VM を作成したりできます。 このアーキテクチャで役立つ可能性のあるその他の[組み込みの RBAC ロール](/azure/active-directory/role-based-access-built-in-roles)には、[DevTest Labs ユーザー](/azure/active-directory/role-based-access-built-in-roles#devtest-labs-user)や[ネットワークの共同作業者](/azure/active-directory/role-based-access-built-in-roles#network-contributor)が含まれます。

> [!NOTE]  
> RBAC では、VM にログインしているユーザーが実行できる操作は制限されません。 これらのアクセス許可は、ゲスト OS のアカウントの種類によって決まります。

**監査ログ**。 プロビジョニング操作や他の VM イベントを確認するには、[アクティビティ ログ](./azure-stack-metrics-azure-data.md?#activity-log)を使用します。

**データの暗号化**。 Azure Stack Hub では、BitLocker 128 ビット AES 暗号化を使用して、ストレージ サブシステム内のユーザーとインフラストラクチャの保存データが保護されます。 詳細については、「[Azure Stack Hub における保存データの暗号化](../operator/azure-stack-security-bitlocker.md)」を参照してください。


## <a name="next-steps"></a>次のステップ

- Azure Stack Hub VM の詳細については、「[Azure Stack Hub VM の機能](azure-stack-vm-considerations.md)」を参照してください。  
- Azure のクラウド パターンの詳細については、「[Cloud Design Pattern (クラウド設計パターン)](/azure/architecture/patterns)」を参照してください。
