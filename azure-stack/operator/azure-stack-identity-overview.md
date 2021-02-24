---
title: Azure Stack Hub の ID プロバイダーの概要
description: Azure Stack Hub に使用できる ID プロバイダーについて学習します。
author: BryanLa
ms.topic: conceptual
ms.date: 04/10/2020
ms.author: bryanla
ms.reviewer: fiseraci
ms.lastreviewed: 01/14/2019
ms.openlocfilehash: dac1902747c79b68116c0341f50c47c3e0998c0f
ms.sourcegitcommit: e9a1dfa871e525f1d6d2b355b4bbc9bae11720d2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/20/2020
ms.locfileid: "86488792"
---
# <a name="overview-of-identity-providers-for-azure-stack-hub"></a>Azure Stack Hub の ID プロバイダーの概要

Azure Stack Hub には、Azure Active Directory (Azure AD)、または ID プロバイダーとして Active Directory を使用する Active Directory フェデレーション サービス (AD FS) が必要です。 プロバイダーの選択は、Azure Stack Hub を初めてデプロイするときに、一度だけ行います。 この記事での概念と承認についての詳細情報は、ID プロバイダーの選択に役立ちます。

Azure AD にするか AD FS にするかの選択は、Azure Stack Hub をデプロイするモードによって決まります。

- 接続モードでデプロイする場合は、Azure AD と AD FS のどちらも使用できます。
- インターネットに接続せずに切断モードでデプロイする場合は、AD FS のみがサポートされます。

Azure Stack Hub 環境に依存するオプションの詳細については、以下の記事を参照してください。

- Azure Stack Hub のデプロイ キット: [ID に関する考慮事項](azure-stack-datacenter-integration.md#identity-considerations)。
- Azure Stack Hub 統合システム: [Azure Stack Hub 統合システムのデプロイ計画の決定](azure-stack-connection-models.md)。

## <a name="common-concepts-for-identity-providers"></a>ID プロバイダーに関する一般的な概念

次のセクションでは、ID プロバイダーの一般的な概念と、Azure Stack Hub での使用について説明します。

![ID プロバイダーに関する用語](media/azure-stack-identity-overview/terminology.svg)

### <a name="directory-tenants-and-organizations"></a>ディレクトリ テナントと組織

ディレクトリとは、*ユーザー*、*アプリケーション*、*グループ*、および*サービス プリンシパル*に関する情報を保持するコンテナーのことです。

ディレクトリ テナントとは、Microsoft や自分の会社のような、"*組織*" のことです。

- Azure AD では複数のテナントがサポートされており、複数の組織をそれぞれ独自のディレクトリでサポートすることができます。 Azure AD を使用しており、複数のテナントがある場合、1 つのテナントのアプリとユーザーに、同じディレクトリの他のテナントへのアクセス権を付与できます。
- AD FS では、単一のテナントのみがサポートされます。そのため、単一の組織のみがサポートされます。

### <a name="users-and-groups"></a>ユーザーとグループ

ユーザー アカウント (ID) は、ユーザー ID とパスワードを使用して個人を認証する標準的なアカウントです。 グループには、ユーザーまたは他のグループを含めることができます。

ユーザーとグループを作成および管理する方法は、使用する ID ソリューションによって異なります。

Azure Stack Hub では、ユーザー アカウントに次のような特徴があります。

- *username\@domain* の形式で作成されています。 AD FS はユーザー アカウントを Active Directory インスタンスにマッピングしますが、AD FS では *\\\<domain>\\\<alias>* 形式の使用はサポートされていません。
- 多要素認証を使用するようにセットアップすることができます。
- 最初に登録するディレクトリ、つまり組織のディレクトリに制限されています。
- オンプレミスのディレクトリからインポートすることができます。 詳細については、「[オンプレミスのディレクトリと Azure Active Directory の統合](/azure/active-directory/connect/active-directory-aadconnect)」を参照してください。

組織のユーザー ポータルにサインインするときは、*https:\//portal.local.azurestack.external* の URL を使用します。 Azure Stack Hub の登録に使用したもの以外のドメインから Azure Stack Hub ポータルにサインインする場合は、Azure Stack Hub の登録に使用したドメイン名をポータルの URL に追加する必要があります。 たとえば、Azure Stack Hub が fabrikam.onmicrosoft.com に登録されていて、ログインするユーザー アカウントが admin@contoso.com である場合、ユーザー ポータルへのログインに使用する URL は https:\//portal.local.azurestack.external/fabrikam.onmicrosoft.com になります。

### <a name="guest-users"></a>ゲスト ユーザー

ゲスト ユーザーとは、あるディレクトリ内のリソースへのアクセスを許可されている、他のディレクトリ テナントのユーザー アカウントのことです。 ゲスト ユーザーをサポートするには、Azure AD を使用し、マルチテナントのサポートを有効にします。 サポートが有効になると、自社のディレクトリ テナント内のリソースにアクセスするようにゲスト ユーザーを招待することができます。これにより、外部組織との共同作業が可能になります。

ゲスト ユーザーを招待するには、クラウド オペレーターおよびユーザーが [Azure AD B2B コラボレーション](/azure/active-directory/active-directory-b2b-what-is-azure-ad-b2b)を使用できます。 招待されたユーザーは、ディレクトリのドキュメント、リソース、およびアプリにアクセスできるようになり、独自のリソースおよびデータに対する制御は元の管理者が保持し続けます。

ゲスト ユーザーは、他の組織のディレクトリ テナントにサインインすることができます。 そうするには、その組織のディレクトリ名をポータル URL に追加します。 たとえば、Contoso という組織に属していて、Fabrikam ディレクトリにサインインする場合は、https:\//portal.local.azurestack.external/fabrikam.onmicrosoft.com を使用します。

### <a name="apps"></a>アプリケーション

Azure AD または AD FS にアプリを登録し、組織内のユーザーにそのアプリを提供することができます。

アプリには以下が含まれます。

- **Web アプリ**:たとえば、Azure portal、Azure Resource Manager などです。 これらでは、Web API 呼び出しがサポートされています。
- **ネイティブ クライアント**: たとえば、Azure PowerShell、Visual Studio、Azure CLI などです。

アプリでは、次の 2 種類のテナントをサポートできます。

- **シングルテナント**: アプリが登録されている同じディレクトリのユーザーとサービスだけをサポートします。

  > [!NOTE]
  > AD FS では単一のディレクトリしかサポートされないため、AD FS トポロジ内で作成するアプリは、設計上、シングルテナント アプリになります。

- **マルチテナント**: アプリが登録されているディレクトリと追加のテナント ディレクトリの両方のユーザーとサービスによって、使用をサポートします。 マルチテナント アプリでは、別のテナント ディレクトリ (別の Azure AD テナント) のユーザーがアプリにサインインすることができます。

  マルチテナントの詳細については、[マルチテナントの有効化](azure-stack-enable-multitenancy.md)に関するページを参照してください。

  マルチテナント アプリの開発の詳細については、[マルチテナント アプリ](/azure/active-directory/develop/active-directory-devhowto-multi-tenant-overview)に関するページを参照してください。

アプリを登録するときに、次の 2 つのオブジェクトを作成します。

- **アプリケーション オブジェクト**: すべてのテナントにわたる、アプリのグローバルな表現です。 この関係はソフトウェア アプリと 1 対 1 であり、アプリが最初に登録されたディレクトリだけに存在します。

- **サービス プリンシパル オブジェクト**: アプリが最初に登録されたディレクトリ内でそのアプリのために作成される資格情報です。 また、サービス プリンシパルは、そのアプリが使用される追加の各テナントのディレクトリにも作成されます。 この関係は、ソフトウェア アプリと 1 対多にすることができます。

アプリとサービス プリンシパルのオブジェクトの詳細については、「[Azure Active Directory のアプリケーション オブジェクトとサービス プリンシパル オブジェクト](/azure/active-directory/develop/active-directory-application-objects)」を参照してください。

### <a name="service-principals"></a>サービス プリンシパル

サービス プリンシパルは、Azure Stack Hub 内のリソースへのアクセスを許可する、アプリまたはサービスの*資格情報*のセットです。 サービス プリンシパルを使用すると、アプリのアクセス許可と、アプリのユーザーのアクセス許可が分離されます。

サービス プリンシパルは、アプリが使用される各テナントに作成されます。 そのテナントによって保護されているリソース (ユーザーなど) へのサインインとアクセスのために、サービス プリンシパルは ID を確立します。

- シングルテナント アプリでは、最初に作成されたディレクトリ内に、サービス プリンシパルが 1 つだけ保持されます。 このサービス プリンシパルは、アプリの登録時に作成されて、使用が承認されます。
- マルチテナント Web アプリまたは API には、該当のテナントのユーザーがアプリの使用に同意した各テナント内に、作成されたサービス プリンシパルがあります。

サービス プリンシパルの資格情報は、Azure Portal を通じて生成されるキー、または証明書です。 証明書は、キーよりも安全であると考えられるため、自動化に適しています。

> [!NOTE]
> Azure Stack Hub で AD FS を使用する場合、管理者だけがサービス プリンシパルを作成できます。 AD FS では、サービス プリンシパルは証明書を必要とし、特権エンドポイント (PEP) を通じて作成されます。 詳細については、「[アプリ ID を使用してリソースにアクセスする](azure-stack-create-service-principals.md)」を参照してください。

Azure Stack Hub のサービス プリンシパルについては、[サービス プリンシパルの作成](azure-stack-create-service-principals.md)に関するページを参照してください。

### <a name="services"></a>サービス

ID プロバイダーとやりとりする Azure Stack Hub 内のサービスは、ID プロバイダーにアプリとして登録されます。 アプリと同様に、登録されることで、サービスは ID システムによって認証できるようになります。

すべての Azure サービスは、[OpenID Connect](/azure/active-directory/develop/active-directory-protocols-openid-connect-code) プロトコルと [JSON Web トークン](/azure/active-directory/develop/active-directory-token-and-claims)を使用して ID を確立します。 Azure AD と AD FS はプロトコルを同じように使用するため、Azure [Active Directory Authentication Library](/azure/active-directory/develop/active-directory-authentication-libraries) (ADAL) を使用して、オンプレミスまたは Azure (接続シナリオの場合) で認証することができます。 ADAL では、クロスクラウドおよびオンプレミスのリソース管理に Azure PowerShell や Azure CLI などのツールを使用することもできます。

### <a name="identities-and-your-identity-system"></a>ID と ID システム

Azure Stack Hub の ID には、ユーザー アカウント、グループ、サービス プリンシパルが含まれます。

Azure Stack Hub をインストールすると、いくつかの組み込みのアプリとサービスが、ディレクトリ テナント内の ID プロバイダーに自動的に登録されます。 登録される一部のサービスは、管理用に使用されます。 その他のサービスは、ユーザーが使用できます。 既定の登録では、各コア サービスに ID が与えられます。この ID では、相互に対話することも、後で追加する ID と対話することもできます。

マルチテナントを利用して Azure AD をセットアップすると、一部のアプリは新しいディレクトリに反映されます。

## <a name="authentication-and-authorization"></a>認証と権限承認

### <a name="authentication-by-apps-and-users"></a>アプリとユーザーによる認証

![Azure Stack Hub のレイヤー間の ID](media/azure-stack-identity-overview/identity-layers.svg)

アプリとユーザーにとって、Azure Stack Hub のアーキテクチャは 4 つのレイヤーで表されます。 各レイヤー間の対話には、さまざまな種類の認証を使用できます。

|レイヤー    |レイヤー間の認証  |
|---------|---------|
|管理者ポータルなどのツールとクライアント     | Azure Stack Hub のリソースにアクセスしたり、変更したりするために、ツールとクライアントでは [JSON Web トークン](/azure/active-directory/develop/active-directory-token-and-claims)を使用して Azure Resource Manager を呼び出します。 <br>Azure Resource Manager では JSON Web トークンを検証し、発行されたトークン内の "*要求*" を読み取って、ユーザーまたはサービス プリンシパルが Azure Stack Hub で持つ承認のレベルを見積もります。 |
|Azure Resource Manager とそのコア サービス     |Azure Resource Manager は、リソース プロバイダーと通信して、ユーザーからの通信を転送します。 <br> 転送では、[Azure Resource Manager テンプレート](../user/azure-stack-arm-templates.md)を通じて、"*直接命令*" 呼び出しまたは "*宣言*" 呼び出しが使用されます。|
|リソース プロバイダー     |リソース プロバイダーに渡された呼び出しは、証明書ベースの認証によって保護されます。 <br>Azure Resource Manager とリソース プロバイダーは、API を介した通信を継続します。 Azure Resource Manager から受信したすべての呼び出しを、リソース プロバイダーはその証明書で検証します。|
|インフラストラクチャとビジネス ロジック     |リソース プロバイダーは、任意の認証モードを使用して、ビジネス ロジックおよびインフラストラクチャと通信します。 Azure Stack Hub 付属の既定のリソース プロバイダーは、この通信をセキュリティで保護するために Windows 認証を使用します。|

![認証に必要な情報](media/azure-stack-identity-overview/authentication.svg)

### <a name="authenticate-to-azure-resource-manager"></a>Azure Resource Manager への認証

ID プロバイダーで認証して JSON Web トークンを受け取るには、次の情報が必要です。

1. **ID システム (機関) の URL**: ID プロバイダーに到達できる URL。 例: *https:\//login.windows.net*
2. **Azure Resource Manager のアプリ ID URI**: ID プロバイダーに登録された、Azure Resource Manager の一意の識別子。 各 Azure Stack Hub のインストールに対しても一意です。
3. **資格情報**:ID プロバイダーでの認証に使用する資格情報。
4. **Azure Resource Manager の URL**: Azure Resource Manager サービスの場所を示す URL。 例: *https:\//management.azure.com* または *https:\//management.local.azurestack.external*

プリンシパル (クライアント、アプリ、またはユーザー) がリソースにアクセスするために認証要求を行う場合、その要求には以下のものが含まれている必要があります。

- プリンシパルの資格情報。
- プリンシパルがアクセスするリソースのアプリ ID URI。

資格情報は、ID プロバイダーによって検証されます。 ID プロバイダーでは、アプリ ID URI が登録済みアプリに対応していることと、プリンシパルがそのリソースのトークンを取得するための正しい特権を持っていることも検証します。 要求が有効な場合は、JSON Web トークンが付与されます。

その後、トークンは要求のヘッダーで Azure Resource Manager に渡す必要があります。 Azure Resource Manager は、不特定の順序で以下の操作を行います。

- トークンが正しい ID プロバイダーからのものであることを確認するために、*issuer* (iss) 要求を検証します。
- トークンが Azure Resource Manager に発行されたことを確認するために、*audience* (aud) 要求を検証します。
- JSON Web トークンが OpenID を通じて構成された証明書によって署名され、Azure Resource Manager に認識されていることを検証します。
- トークンがアクティブであり、承認可能であることを確認するために、*issued at* (iat) および *expiration* (exp) 要求を見直します。

すべての検証が完了すると、Azure Resource Manager で *object id* (oid) および *groups* 要求を使用して、プリンシパルがアクセスできるリソースの一覧が作成されます。

![トークン交換プロトコルの図](media/azure-stack-identity-overview/token-exchange.svg)

> [!NOTE]
> デプロイ後は、Azure Active Directory の全体管理者のアクセス許可は必要ありません。 ただし、一部の操作では、全体管理者の資格情報が必要な場合があります (たとえば、リソース プロバイダーのインストーラー スクリプトや、アクセス許可を付与する必要のある新機能です)。 アカウントの全体管理者のアクセス許可を一時的に復元するか、*既定のプロバイダー サブスクリプション*の所有者である別の全体管理者アカウントを使用します。

### <a name="use-role-based-access-control"></a>ロールベースのアクセス制御を使用する

Azure Stack Hub のロールベースのアクセス制御 (RBAC) は、Microsoft Azure での実装と一致しています。 適切な RBAC ロールをユーザー、グループ、およびアプリに割り当てることによって、リソースへのアクセスを管理することができます。 Azure Stack Hub で RBAC を使用する方法については、以下の記事を参照してください。

- [Azure Portal でのロールベースの Access Control の基礎を確認する](/azure/role-based-access-control/overview)
- [ロールベースのアクセス制御を使用して Azure サブスクリプション リソースへのアクセスを管理する](/azure/role-based-access-control/role-assignments-portal)
- [Azure のロールベースのアクセス制御のためのカスタム ロールを作成する](/azure/role-based-access-control/custom-roles)
- Azure Stack Hub で[ロールベースのアクセス制御を管理する](azure-stack-manage-permissions.md)

### <a name="authenticate-with-azure-powershell"></a>Azure PowerShell で認証する

Azure PowerShell を使用して Azure Stack Hub で認証する方法の詳細については、[Azure Stack Hub ユーザーの PowerShell 環境の構成](../user/azure-stack-powershell-configure-user.md)に関するページで見つけることができます。

### <a name="authenticate-with-azure-cli"></a>Azure CLI で認証する

Azure PowerShell を使用して Azure Stack Hub で認証する方法については、[Azure Stack Hub で使用する Azure CLI のインストールと構成](../user/azure-stack-version-profiles-azurecli2.md)に関するページを参照してください。

## <a name="next-steps"></a>次のステップ

- [ID アーキテクチャ](azure-stack-identity-architecture.md)
- [データセンターの統合 - ID](azure-stack-integrate-identity.md)
