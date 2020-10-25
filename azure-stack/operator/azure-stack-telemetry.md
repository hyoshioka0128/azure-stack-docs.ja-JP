---
title: Azure Stack Hub のテレメトリを構成する
titleSuffix: Azure Stack
description: Azure Stack Hub のテレメトリと、PowerShell を使用してテレメトリ設定を構成する方法について学習します。
author: IngridAtMicrosoft
ms.topic: conceptual
ms.date: 1/16/2020
ms.author: inhenkel
ms.reviewer: comartin
ms.lastreviewed: 10/15/2019
ms.openlocfilehash: afaf8f4716443631ae0d2bbe1b82cd84b9d0e424
ms.sourcegitcommit: e9a1dfa871e525f1d6d2b355b4bbc9bae11720d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86487738"
---
# <a name="configure-azure-stack-hub-telemetry"></a>Azure Stack Hub のテレメトリを構成する

Azure Stack Hub のテレメトリでは、接続ユーザー エクスペリエンスを介してシステム データが自動的に Microsoft にアップロードされます。 Microsoft チームでは、Azure Stack Hub のテレメトリで収集されるデータを使用して、カスタマー エクスペリエンスを向上させます。 また、このデータはセキュリティ、正常性、品質、パフォーマンスの解析にも使用されます。

Azure Stack Hub オペレーターの場合、テレメトリによって、企業でのデプロイに関する重要な分析情報を得ることができ、将来のバージョンの Azure Stack Hub を形作るのに役立つ声が得られます。

> [!NOTE]
> また、課金のための使用状況情報を Azure に転送するように Azure Stack Hub を構成することもできます。 マルチノードの Azure Stack Hub をご利用で従量課金制を選択されたお客様には、これが必要となります。 使用状況のレポートは、テレメトリとは別に制御され、マルチノードをご利用の場合でも容量モデルを選択されたお客様や、Azure Stack Development Kit のユーザーには必要ありません。 そのような場合は、使用状況のレポートを[登録スクリプトを使って](azure-stack-usage-reporting.md)無効にすることができます。

Azure Stack Hub テレメトリは、Windows Server 2016 の接続ユーザー エクスペリエンスとテレメトリ コンポーネントに基づいています。 このコンポーネントでは [Windows イベント トレーシング (ETW)](/windows/win32/tracelogging/trace-logging-about) の TraceLogging テクノロジを使用して、イベントとデータを収集して格納します。 Azure Stack のコンポーネントにも同じテクノロジが使用されており、パブリックなオペレーティング システムのイベント ログとトレース API を使用して収集されたイベントとデータが発行されます。 これらの Azure Stack Hub コンポーネントの例として、ネットワーク リソース プロバイダー、ストレージ リソース プロバイダー、監視リソース プロバイダー、更新リソース プロバイダーがあります。 接続ユーザー エクスペリエンスとテレメトリ コンポーネントは、SSL を使用してデータを暗号化し、証明書のピン留めを使用してデータを HTTPS で Microsoft Data Management サービスに送信します。

> [!IMPORTANT]
> テレメトリ データ フローを有効にするには、ネットワークでポート 443 (HTTPS) を開く必要があります。 "接続ユーザー エクスペリエンスとテレメトリ" コンポーネントから Microsoft Data Management サービスへの接続には、`https://v10.events.data.microsoft.com` が使われます。 また、"接続ユーザー エクスペリエンスとテレメトリ" コンポーネントは、構成情報をダウンロードする目的で `https://settings-win.data.microsoft.com` にも接続します。 その他の診断データ サービスは、エラーのレポートのために `https://watson.telemetry.microsoft.com` を接続します。

## <a name="privacy-considerations"></a>プライバシーに関する考慮事項

テレメトリ データは、ETW サービスによって、保護されたクラウド ストレージに戻されます。 テレメトリ データへのアクセスには、最小特権の原則が適用されます。 テレメトリ データへのアクセスは、業務上の正当な必要性がある Microsoft の担当者にのみ認められます。 お客様の判断による場合や、[Microsoft のプライバシーに関する声明](https://privacy.microsoft.com/PrivacyStatement)に記載されている限定的な目的を除き、Microsoft がお客様の個人データを第三者と共有することはありません。 OEM やパートナーと共有されるビジネス レポートには、集計されて匿名化されたデータが含まれています。 データ共有の決定は、プライバシー、法務、データ管理の関係者を含む Microsoft 社内のチームが行います。

Microsoft は情報量の最小化に賛同し、実践しています。 必要な情報だけを収集するよう努め、サービスまたは解析に必要な期間のみ保存しています。 Azure Stack Hub システムや Azure サービスがどのように機能しているかについての情報は、その多くが 6 か月以内に削除されます。 要約されたデータや集計されたデータの保存期間は、それよりも長くなります。

Microsoft は、お客様の情報のプライバシーとセキュリティがいかに重要であるかを理解しています。  Microsoft では、お客様のプライバシーと Azure Stack Hub 内のお客様のデータの保護に関して、熟慮を重ねながら包括的な取り組みを行っています。 各種機能やプライバシー設定は、IT 管理者の権限でいつでもカスタマイズできます。 透明性と信用を確保するために、Microsoft は以下のことを表明しています。

- 収集するデータの種類をありのままお客様に伝えます。
- 企業ユーザーによる独自管理を可能にします。お客様は自社のプライバシー設定を独自にカスタマイズすることができます。
- お客様のプライバシーとセキュリティを最優先事項とします。
- テレメトリ データの用途について透明性を確保します。
- テレメトリ データは、カスタマー エクスペリエンスの向上を目的として使用されます。

Microsoft では、意図して機微なデータ (クレジット カード番号、ユーザー名とパスワード、メール アドレスなどの、機密情報) を収集することはありません。 意図せず機密情報を受け取ったと Microsoft が判断した場合には、削除します。

## <a name="examples-of-how-microsoft-uses-the-telemetry-data"></a>Microsoft によるテレメトリ データの使用例

テレメトリは、信頼性にかかわる重大な問題をお客様のデプロイと構成からいち早く特定し、修正するうえで重要な役割を果たします。 テレメトリ データから得られる分析情報は、サービスの問題やハードウェア構成の問題を速やかに把握するために役立てられます。 Microsoft ではお客様からこうしたデータを得て、エコシステム向上の原動力とすることができます。これにより、統合された Azure Stack Hub ソリューションの品質水準が上がります。

テレメトリは、お客様がどのようにコンポーネントをデプロイし、各種機能を使用しているか、またお客様がどのようにサービスを使用してそのビジネス目標を達成しているかについて、Microsoft が理解を深めるうえでも役立ちます。 これらの分析情報は、お客様のエクスペリエンスやワークロードに直接影響する領域に対して優先的にエンジニアリングの投資を行うのに役立ちます。

たとえば、Azure Stack Hub ロールに関連付けられているコンテナー、ストレージ、ネットワーク構成が、お客様によってどのように使用されているか、といった情報が挙げられます。 そのような分析情報も、Azure Stack Hub の管理および監視ソリューションの改善やインテリジェンスに活かされます。 これらの改善により、お客様は問題を診断しやすくなり、また、Microsoft サポートへの問い合わせの件数が減ることでコストの削減にもつながります。

## <a name="manage-telemetry-collection"></a>テレメトリの収集を管理する

組織でテレメトリをオフにすることはお勧めできません。 しかし、シナリオによってはそれが必要になることもあります。

そのようなシナリオでは、Microsoft に送信されるテレメトリのレベルを構成することができます。その場合、Azure Stack Hub のデプロイ前はレジストリ設定を、Azure Stack Hub のデプロイ後はテレメトリ エンドポイントを使用します。

### <a name="telemetry-levels-and-data-collection"></a>テレメトリのレベルと収集されるデータ

テレメトリの設定を変更する前に、テレメトリのレベルと、各レベルで収集されるデータについて理解してください。

テレメトリの設定は累積で 4 つのレベル (0 ～3) にグループ化され、次のように分類されます。

**0 (Security)**</br>
セキュリティ データのみ。 オペレーティング システムのセキュリティを確保するために必要な情報。 これには、"接続ユーザー エクスペリエンスとテレメトリ" コンポーネントの設定と Windows Defender に関するデータが含まれています。 Azure Stack Hub に固有のテレメトリは、このレベルでは出力されません。

**1 (Basic)**</br>
Security データに加え、基本的な正常性データと品質データ。 **Security** レベルからのデータに加え、品質関連のデータ、アプリの互換性、アプリの利用状況データなど、基本的なデバイス情報が対象となります。 テレメトリ レベルを Basic に設定すると、Azure Stack Hub のテレメトリが有効になります。 このレベルで収集されるデータの例を次に示します。

- "*基本的なデバイス情報*": エコシステムに存在する Windows Server 2016 のネイティブ インスタンスと仮想インスタンスの構成や種類が明らかとなります。 これには次のものが含まれます

  - マシン属性 (OEM、モデルなど)。
  - ネットワーク属性 (ネットワーク アダプターの数や速度など)。
  - プロセッサとメモリの属性 (コア数、インストールされたメモリの量など)。
  - ストレージ属性 (ドライブの数、種類、サイズなど)。

- "*テレメトリの機能*": アップロードされたイベントの割合、ドロップされたイベントの割合、前回のデータのアップロード時刻などが収集されます。
- "*品質関連の情報*": Microsoft が、Azure Stack Hub がどのように実行されているかについての基本的な理解を深めるのに役立ちます。 たとえば、特定のハードウェア構成に関する重要なアラートの件数が該当します。
- "*互換性データ*": システムや仮想マシン (VM) にインストールされているリソース プロバイダーについて理解するのに役立ちます。 これは互換性に関する問題のリスクを特定します。

**2 (Enhanced)**</br>
その他の分析情報: オペレーティング システムや Azure Stack Hub サービスがどのように使用されているか、これらのサービスがどのように実行されているか、詳細な信頼性データ、**Security** および **Basic** レベルからのデータなど。

> [!NOTE]
> これはテレメトリの既定の設定です。

**3 (Full)**</br>
**Security**、**Basic**、**Enhanced** の各レベルからのデータに加え、問題を特定して修正するために必要なあらゆるデータ。

> [!IMPORTANT]
> これらのテレメトリ レベルが適用されるのは、Microsoft Azure Stack Hub のコンポーネントのみです。 Azure Stack Hub ハードウェア パートナーのハードウェア ライフサイクル ホストで動作する、Microsoft 以外のソフトウェア コンポーネントやサービスでは、これらのテレメトリ レベル外でそのクラウド サービスと通信を行うことが考えられます。 テレメトリ ポリシーについてと、どのようにオプトインまたはオプトアウトできるかについては、ご利用の Azure Stack Hub ハードウェア ソリューション プロバイダーにお問い合わせください。

Windows と Azure Stack Hub のテレメトリをオフにすると、SQL のテレメトリも無効になります。 Windows Server のテレメトリ設定の作用に関する詳細については、[Windows テレメトリのホワイトペーパー](https://aka.ms/winservtelemetry)を参照してください。

### <a name="asdk-set-the-telemetry-level-in-the-windows-registry"></a>ASDK: Windows レジストリでテレメトリ レベルを設定する

Azure Stack Hub をデプロイする前に、Windows レジストリ エディターを使用して、物理ホスト コンピューターに対してテレメトリ レベルを手動で設定できます。 グループ ポリシーなどの管理ポリシーが既に存在する場合、そのポリシーがこのレジストリ設定をオーバーライドします。

開発キットのホストに Azure Stack Hub をデプロイする前に、CloudBuilder.vhdx を起動し、管理者特権の PowerShell ウィンドウで次のスクリプトを実行します。

```powershell
### Get current AllowTelemetry value on DVM Host
(Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\DataCollection" `
-Name AllowTelemetry).AllowTelemetry
### Set & Get updated AllowTelemetry value for ASDK-Host
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\DataCollection" `
-Name "AllowTelemetry" -Value '0' # Set this value to 0,1,2,or3.  
(Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\DataCollection" `
-Name AllowTelemetry).AllowTelemetry
```

### <a name="asdk-and-multi-node-enable-or-disable-telemetry-after-deployment"></a>ASDK とマルチノード: デプロイ後にテレメトリを有効または無効にする

デプロイ後にテレメトリを有効または無効にするには、ERCS VM に公開されている特権エンドポイント (PEP) へのアクセスが必要です。

- 有効にする場合: `Set-Telemetry -Enable`
- 無効にする場合: `Set-Telemetry -Disable`

パラメーターの詳細:
- `.PARAMETER Enable` - テレメトリ データのアップロードをオンにする
- `.PARAMETER Disable` - テレメトリ データのアップロードをオフにする  

**テレメトリを有効にするためのスクリプト:**

```powershell
$ip = "<IP ADDRESS OF THE PEP VM>" # You can also use the machine name instead of IP here.
$pwd= ConvertTo-SecureString "<CLOUD ADMIN PASSWORD>" -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential ("<DOMAIN NAME>\CloudAdmin", $pwd)
$psSession = New-PSSession -ComputerName $ip -ConfigurationName PrivilegedEndpoint -Credential $cred
Invoke-Command -Session $psSession {Set-Telemetry -Enable}
if($psSession)
{
    Remove-PSSession $psSession
}
```

**テレメトリを無効にするためのスクリプト:**

```powershell
$ip = "<IP ADDRESS OF THE PEP VM>" # You can also use the machine name instead of IP here.
$pwd= ConvertTo-SecureString "<CLOUD ADMIN PASSWORD>" -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential ("<DOMAIN NAME>\CloudAdmin", $pwd)
$psSession = New-PSSession -ComputerName $ip -ConfigurationName PrivilegedEndpoint -Credential $cred
Invoke-Command -Session $psSession {Set-Telemetry -Disable}
if($psSession)
{
    Remove-PSSession $psSession
}
```

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub を Azure に登録する](azure-stack-registration.md)
