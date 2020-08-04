---
title: Azure 및 Azure Stack Hub 앱에 대한 하이브리드 클라우드 ID 구성
description: Azure 및 Azure Stack Hub 앱에 대한 하이브리드 클라우드 ID를 구성하는 방법에 대해 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 650eef0f144ecafab4586d93f72e1defdf4a61ce
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477255"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a>Azure 및 Azure Stack Hub 앱에 대한 하이브리드 클라우드 ID 구성

Azure 및 Azure Stack Hub 앱에 대한 하이브리드 클라우드 ID를 구성하는 방법에 대해 알아봅니다.

글로벌 Azure와 Azure Stack Hub 모두에서 앱에 대한 액세스 권한을 부여하는 두 가지 옵션이 있습니다.

 * Azure Stack Hub가 인터넷에 지속적으로 연결되어 있는 경우 Azure AD(Azure Active Directory)를 사용할 수 있습니다.
 * Azure Stack Hub가 인터넷에서 연결이 끊기면 Azure AD FS(디렉터리 페더레이션 서비스)를 사용할 수 있습니다.

서비스 주체를 사용하여 Azure Stack Hub의 Azure Resource Manager를 통해 배포 또는 구성을 위한 Azure Stack Hub 앱에 대한 액세스 권한을 부여합니다.

이 솔루션에서는 다음을 수행하는 샘플 환경을 빌드합니다.

> [!div class="checklist"]
> - 글로벌 Azure 및 Azure Stack Hub에서 하이브리드 ID 설정
> - Azure Stack Hub API에 액세스하는 토큰을 검색합니다.

이 솔루션의 단계에 대한 Azure Stack Hub 운영자 권한이 있어야 합니다.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub는 Azure의 확장입니다. Azure Stack Hub는 온-프레미스 환경에 클라우드 컴퓨팅의 민첩성과 혁신을 제공하여 어디서나 하이브리드 앱을 빌드하고 배포할 수 있는 유일한 하이브리드 클라우드를 사용하도록 설정합니다.  
> 
> [하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 문서는 하이브리드 앱 디자인, 배포 및 운영에 대한 소프트웨어 품질(배치, 확장성, 가용성, 복원력, 관리 효율성 및 보안)의 핵심 요소를 검토합니다. 디자인 고려 사항은 하이브리드 앱 디자인을 최적화하고 프로덕션 환경에서 문제를 최소화하는 데 도움이 됩니다.

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a>포털에서 Azure AD에 대한 서비스 주체 만들기

Azure AD를 ID 저장소로 사용하여 Azure Stack Hub를 배포한 경우 Azure에 대해 수행하는 것처럼 서비스 주체를 만들 수 있습니다. [앱 ID를 사용하여 리소스에 액세스](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity)는 포털을 통해 단계를 수행하는 방법을 보여 줍니다. 시작하기 전에 [필요한 Azure AD 사용 권한](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions)이 있어야 합니다.

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a>PowerShell을 사용하여 AD FS에 대한 서비스 주체 만들기

AD FS를 사용하여 Azure Stack Hub를 배포한 경우 PowerShell을 사용하여 서비스 주체를 만들고, 액세스에 대한 역할을 할당하고, 해당 ID를 사용하여 PowerShell에서 로그인할 수 있습니다. [앱 ID를 사용하여 리소스에 액세스](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity)는 PowerShell을 사용하여 필요한 단계를 수행하는 방법을 보여 줍니다.

## <a name="using-the-azure-stack-hub-api"></a>Azure Stack Hub API 사용

[Azure Stack Hub API](/azure-stack/user/azure-stack-rest-api-use.md) 솔루션은 Azure Stack Hub API에 액세스하는 토큰을 검색하는 과정을 안내합니다.

## <a name="connect-to-azure-stack-hub-using-powershell"></a>PowerShell을 사용하여 Azure Stack Hub에 연결

[Azure Stack Hub에서 PowerShell을 시작하고 실행하는](/azure-stack/operator/azure-stack-powershell-install.md) 빠른 시작에서는 Azure PowerShell을 설치하고 Azure Stack Hub 설치에 연결하는 데 필요한 단계를 안내합니다.

### <a name="prerequisites"></a>필수 구성 요소

액세스할 수 있는 구독을 사용하여 Azure AD에 연결된 Azure Stack Hub를 설치해야 합니다. Azure Stack Hub를 설치하지 않은 경우 다음 지침을 사용하여 [ASDK(Azure Stack Development Kit)](/azure-stack/asdk/asdk-install.md)를 설정할 수 있습니다.

#### <a name="connect-to-azure-stack-hub-using-code"></a>코드를 사용하여 Azure Stack Hub에 연결

코드를 사용하여 Azure Stack Hub에 연결하려면 Azure Resource Manager 엔드포인트 API를 사용하여 Azure Stack Hub 설치를 위한 인증 및 그래프 엔드포인트를 가져옵니다. 그런 다음, REST 요청을 사용하여 인증합니다. [GitHub](https://github.com/shriramnat/HybridARMApplication)에서 샘플 클라이언트 애플리케이션을 찾을 수 있습니다.

>[!Note]
>선택하는 언어에 대한 Azure SDK가 Azure API 프로필을 지원하지 않는 한 SDK는 Azure Stack Hub에서 작동하지 않을 수 있습니다. Azure API 프로필에 대해 자세히 알아보려면 [API 버전 프로필 관리](/azure-stack/user/azure-stack-version-profiles.md) 문서를 참조하세요.

## <a name="next-steps"></a>다음 단계

- Azure Stack Hub에서 ID를 처리하는 방법에 대한 자세한 내용은 [Azure Stack Hub에 대한 ID 아키텍처](/azure-stack/operator/azure-stack-identity-architecture.md)를 참조하세요.
- Azure Cloud 패턴에 대해 자세히 알아보려면 [클라우드 디자인 패턴](/azure/architecture/patterns)을 참조하세요.
