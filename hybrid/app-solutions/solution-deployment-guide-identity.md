---
title: Azure 및 Azure Stack Hub 앱에 대 한 하이브리드 클라우드 id 구성
description: Azure 및 Azure Stack Hub 앱에 대 한 하이브리드 클라우드 id를 구성 하는 방법에 대해 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 1b3c683dd3e4a68413f83fd3cc129d6e6f594e1b
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911316"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a>Azure 및 Azure Stack Hub 앱에 대 한 하이브리드 클라우드 id 구성

Azure 및 Azure Stack Hub 앱에 대 한 하이브리드 클라우드 id를 구성 하는 방법에 대해 알아봅니다.

글로벌 Azure와 Azure Stack Hub 모두에서 앱에 대 한 액세스 권한을 부여 하는 두 가지 옵션이 있습니다.

 * Azure Stack 허브가 인터넷에 지속적으로 연결 되어 있는 경우 Azure Active Directory (Azure AD)를 사용할 수 있습니다.
 * Azure Stack 허브가 인터넷에서 연결이 끊기면 Azure AD FS (디렉터리 페더레이션 서비스)를 사용할 수 있습니다.

서비스 주체를 사용 하 여 Azure Stack 허브의 Azure Resource Manager를 사용 하 여 배포 또는 구성에 대 한 Azure Stack 허브 앱에 대 한 액세스 권한을 부여 합니다.

이 솔루션에서는 다음을 수행 하는 샘플 환경을 빌드합니다.

> [!div class="checklist"]
> - 글로벌 Azure 및 Azure Stack 허브에서 하이브리드 id 설정
> - Azure Stack 허브 API에 액세스 하는 토큰을 검색 합니다.

이 솔루션의 단계에 대 한 Azure Stack 허브 운영자 권한이 있어야 합니다.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack 허브는 Azure의 확장입니다. Azure Stack 허브는 온-프레미스 환경에 클라우드 컴퓨팅의 민첩성과 혁신을 제공 하 여 어디서 나 하이브리드 앱을 빌드하고 배포할 수 있는 유일한 하이브리드 클라우드를 사용 하도록 설정 합니다.  
> 
> [하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 문서에서는 하이브리드 앱을 디자인, 배포 및 운영 하기 위한 소프트웨어 품질 (배치, 확장성, 가용성, 복원 력, 관리 효율성 및 보안)의 핵심 요소을 검토 합니다. 디자인 고려 사항은 하이브리드 앱 디자인을 최적화 하 고 프로덕션 환경에서 문제를 최소화 하는 데 도움이 됩니다.

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a>포털에서 Azure AD에 대 한 서비스 주체 만들기

Azure AD를 id 저장소로 사용 하 여 Azure Stack 허브를 배포한 경우 Azure에 대해 수행 하는 것 처럼 서비스 사용자를 만들 수 있습니다. [앱 id를 사용 하 여 리소스에 액세스](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) 는 포털을 통해 단계를 수행 하는 방법을 보여 줍니다. 시작 하기 전에 [필요한 AZURE AD 권한이](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) 있어야 합니다.

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a>PowerShell을 사용 하 여 AD FS에 대 한 서비스 주체 만들기

AD FS를 사용 하 Azure Stack 허브를 배포한 경우 PowerShell을 사용 하 여 서비스 주체를 만들고, 액세스에 역할을 할당 하 고, 해당 id를 사용 하 여 PowerShell에서 로그인 할 수 있습니다. [앱 id를 사용 하 여 리소스에 액세스](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) 하는 방법은 PowerShell을 사용 하 여 필요한 단계를 수행 하는 방법을 보여 줍니다.

## <a name="using-the-azure-stack-hub-api"></a>Azure Stack 허브 API 사용

[Azure Stack HUB api](/azure-stack/user/azure-stack-rest-api-use.md) 솔루션은 AZURE STACK 허브 api에 액세스 하는 토큰을 검색 하는 과정을 안내 합니다.

## <a name="connect-to-azure-stack-hub-using-powershell"></a>PowerShell을 사용 하 여 Azure Stack 허브에 연결

[Azure Stack hub에서 PowerShell을 시작 하 고 실행 하](/azure-stack/operator/azure-stack-powershell-install.md) 는 빠른 시작은 Azure PowerShell를 설치 하 고 Azure Stack 허브 설치에 연결 하는 데 필요한 단계를 안내 합니다.

### <a name="prerequisites"></a>필수 조건

액세스할 수 있는 구독을 사용 하 여 Azure AD에 연결 된 Azure Stack 허브를 설치 해야 합니다. Azure Stack 허브를 설치 하지 않은 경우 다음 지침을 사용 하 여 [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install.md)를 설정할 수 있습니다.

#### <a name="connect-to-azure-stack-hub-using-code"></a>코드를 사용 하 여 Azure Stack 허브에 연결

코드를 사용 하 여 Azure Stack 허브에 연결 하려면 Azure Resource Manager 끝점 API를 사용 하 여 Azure Stack 허브 설치를 위한 인증 및 그래프 끝점을 가져옵니다. 그런 다음 REST 요청을 사용 하 여 인증 합니다. [GitHub](https://github.com/shriramnat/HybridARMApplication)에서 샘플 클라이언트 응용 프로그램을 찾을 수 있습니다.

>[!Note]
>선택 하는 언어에 대 한 Azure SDK가 Azure API 프로필을 지원 하지 않는 한 SDK는 Azure Stack 허브에서 작동 하지 않을 수 있습니다. Azure API 프로필에 대해 자세히 알아보려면 [api 버전 프로필 관리](/azure-stack/user/azure-stack-version-profiles.md) 문서를 참조 하세요.

## <a name="next-steps"></a>다음 단계

- Azure Stack Hub에서 id를 처리 하는 방법에 대 한 자세한 내용은 [Azure Stack 허브에 대 한 id 아키텍처](/azure-stack/operator/azure-stack-identity-architecture.md)를 참조 하세요.
- Azure Cloud 패턴에 대해 자세히 알아보려면 [클라우드 디자인 패턴](https://docs.microsoft.com/azure/architecture/patterns)을 참조 하세요.
