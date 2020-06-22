---
title: Azure 및 Azure Stack Hub에서 클라우드 간 크기를 조정 하는 앱 배포
description: Azure 및 Azure Stack Hub에서 클라우드 간 크기를 조정 하는 앱을 배포 하는 방법에 대해 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 740a8c0ec904fe8eb3f9744626bc9dd6655bdb52
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911526"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a>Azure 및 Azure Stack Hub를 사용 하 여 클라우드 간 크기를 조정 하는 앱 배포

Traffic manager를 통해 자동 크기 조정을 사용 하 여 Azure Stack Hub 호스팅 웹 앱에서 Azure 호스트 된 웹 앱으로 전환 하기 위한 수동으로 트리거된 프로세스를 제공 하는 클라우드 간 솔루션을 만드는 방법을 알아봅니다. 이 프로세스를 통해 부하 상태에서 유연 하 고 확장 가능한 클라우드 유틸리티를 사용할 있습니다.

이 패턴을 사용 하면 테 넌 트가 공용 클라우드에서 앱을 실행할 준비가 되지 않았을 수 있습니다. 그러나 앱에 대 한 급증 하는 요청을 처리 하기 위해 온-프레미스 환경에 필요한 용량을 유지 하는 것이 경제적으로 수 있는 것은 아닙니다. 테 넌 트는 온-프레미스 솔루션으로 공용 클라우드의 탄력성을 활용할 수 있습니다.

이 솔루션에서는 다음을 수행 하는 샘플 환경을 빌드합니다.

> [!div class="checklist"]
> - 다중 노드 웹 앱을 만듭니다.
> - CD (연속 배포) 프로세스를 구성 하 고 관리 합니다.
> - Azure Stack 허브에 웹 앱을 게시 합니다.
> - 릴리스를 만듭니다.
> - 배포를 모니터링 하 고 추적 하는 방법을 알아봅니다.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack 허브는 Azure의 확장입니다. Azure Stack 허브는 온-프레미스 환경에 클라우드 컴퓨팅의 민첩성과 혁신을 제공 하 여 어디서 나 하이브리드 앱을 빌드하고 배포할 수 있는 유일한 하이브리드 클라우드를 사용 하도록 설정 합니다.  
> 
> [하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 문서에서는 하이브리드 앱을 디자인, 배포 및 운영 하기 위한 소프트웨어 품질 (배치, 확장성, 가용성, 복원 력, 관리 효율성 및 보안)의 핵심 요소을 검토 합니다. 디자인 고려 사항은 하이브리드 앱 디자인을 최적화 하 고 프로덕션 환경에서 문제를 최소화 하는 데 도움이 됩니다.

## <a name="prerequisites"></a>필수 구성 요소

- 동작합니다. 필요한 경우 시작 하기 전에 [무료 계정](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 을 만듭니다.
- Azure Stack 허브 통합 시스템 또는 ASDK (Azure Stack Development Kit 배포).
  - Azure Stack 허브를 설치 하는 방법에 대 한 지침은 [ASDK 설치](/azure-stack/asdk/asdk-install.md)를 참조 하세요.
  - ASDK 배포 후 자동화 스크립트의 경우 다음으로 이동 합니다.[https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)
  - 이 설치를 완료 하는 데 몇 시간 정도 걸릴 수 있습니다.
- Azure Stack 허브에 [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS 서비스를 배포 합니다.
- Azure Stack 허브 환경 내에서 [계획/제품을 만듭니다](/azure-stack/operator/service-plan-offer-subscription-overview.md) .
- Azure Stack 허브 환경 내에서 [테 넌 트 구독을 만듭니다](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) .
- 테 넌 트 구독 내에서 웹 앱을 만듭니다. 나중에 사용 하기 위해 새 웹 앱 URL을 기록해 둡니다.
- 테 넌 트 구독 내에서 VM (가상 머신) Azure Pipelines 배포
- .NET 3.5이 있는 Windows Server 2016 VM이 필요 합니다. 이 VM은 Azure Stack 허브의 테 넌 트 구독에 전용 빌드 에이전트로 빌드됩니다.
- [SQL 2017 VM 이미지를 포함 하는 Windows Server 2016](/azure-stack/operator/azure-stack-add-vm-image.md) 는 Azure Stack Hub Marketplace에서 사용할 수 있습니다. 이 이미지를 사용할 수 없는 경우 Azure Stack 허브 운영자에 게 작업 하 여 환경에 추가 되었는지 확인 합니다.

## <a name="issues-and-considerations"></a>문제 및 고려 사항

### <a name="scalability"></a>확장성

클라우드 간 크기 조정의 핵심 구성 요소는 공용 및 온-프레미스 클라우드 인프라 간에 즉각적인 주문형 및 주문형 크기 조정을 제공 하 여 일관적이 고 안정적인 서비스를 제공 하는 기능입니다.

### <a name="availability"></a>가용성

로컬로 배포 된 앱이 온-프레미스 하드웨어 구성 및 소프트웨어 배포를 통해 고가용성으로 구성 되어 있는지 확인 합니다.

### <a name="manageability"></a>관리 효율

클라우드 간 솔루션은 환경 간에 원활한 관리 및 친숙 한 인터페이스를 보장 합니다. PowerShell은 플랫폼 간 관리에 권장 됩니다.

## <a name="cross-cloud-scaling"></a>클라우드 간 크기 조정

### <a name="get-a-custom-domain-and-configure-dns"></a>사용자 지정 도메인 가져오기 및 DNS 구성

도메인에 대 한 DNS 영역 파일을 업데이트 합니다. Azure AD에서 사용자 지정 도메인 이름의 소유권을 확인 합니다. Azure 내에서 Azure/Office 365/외부 DNS 레코드에 대 한 [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) 를 사용 하거나 [다른 DNS 등록자](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/)에 dns 항목을 추가 합니다.

1. 공용 등록자에 게 사용자 지정 도메인을 등록 합니다.
2. 도메인에 대한 도메인 이름 등록 기관에 로그인합니다. 승인 된 관리자는 DNS를 업데이트 해야 할 수 있습니다.
3. Azure AD에서 제공 하는 DNS 항목을 추가 하 여 도메인에 대 한 DNS 영역 파일을 업데이트 합니다. DNS 항목은 메일 라우팅 또는 웹 호스팅 동작에 영향을 주지 않습니다.

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a>Azure Stack 허브에서 기본 다중 노드 웹 앱 만들기

웹 앱을 Azure Stack Azure에 배포 하는 CI/CD (연속 통합 및 지속적인 배포)를 설정 하 고 두 클라우드 모두에 변경 내용을 autopush.

> [!Note]  
> 적절 한 이미지 게시를 실행 하는 Azure Stack 허브 (Windows Server 및 SQL) 및 App Service 배포가 필요 합니다. 자세한 내용은 [Azure Stack 허브에 App Service 배포에 대 한 App Service 설명서 필수 조건](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md)을 참조 하세요.

### <a name="add-code-to-azure-repos"></a>Azure Repos에 코드 추가

Azure Repos

1. Azure Repos에 대 한 프로젝트 만들기 권한이 있는 계정으로 Azure Repos에 로그인 합니다.

    하이브리드 CI/CD는 앱 코드와 인프라 코드 모두에 적용할 수 있습니다. 사설 및 호스 티 드 클라우드 개발 모두에 [Azure Resource Manager 템플릿을](https://azure.microsoft.com/resources/templates/) 사용 합니다.

    ![Azure Repos에서 프로젝트에 연결](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. 기본 웹 앱을 만들고 열어 **리포지토리를 복제** 합니다.

    ![Azure 웹 앱에서 리포지토리 복제](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>두 클라우드에서 App Services에 대 한 자체 포함 웹 앱 배포 만들기

1. **WebApplication .csproj** 파일을 편집 합니다. `Runtimeidentifier`을 선택 하 고 추가 `win10-x64` 합니다. [자체 포함 된 배포](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) 설명서를 참조 하세요.

    ![웹 앱 프로젝트 파일 편집](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. 팀 탐색기를 사용 하 여 Azure Repos 코드를 체크 인 합니다.

3. 앱 코드가 Azure Repos 체크 인 되었는지 확인 합니다.

## <a name="create-the-build-definition"></a>빌드 정의 만들기

1. Azure Pipelines에 로그인 하 여 빌드 정의를 만들 수 있는지 확인 합니다.

2. **-R win10-x64** 코드를 추가 합니다. 이러한 추가는 .NET Core를 사용 하 여 자체 포함 된 배포를 트리거하기 위해 필요 합니다.

    ![웹 앱에 코드 추가](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. 빌드를 실행 합니다. [자체 포함 배포 빌드](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) 프로세스는 Azure 및 Azure Stack Hub에서 실행 되는 아티팩트를 게시 합니다.

## <a name="use-an-azure-hosted-agent"></a>Azure 호스트 된 에이전트 사용

Azure Pipelines에서 호스팅된 빌드 에이전트를 사용 하는 것은 웹 앱을 빌드 및 배포 하는 편리한 옵션입니다. 유지 관리 및 업그레이드는 Microsoft Azure에 의해 자동으로 수행 되어 지속적이 고 중단 되지 않는 개발 주기를 가능 하 게 합니다.

### <a name="manage-and-configure-the-cd-process"></a>CD 프로세스 관리 및 구성

Azure Pipelines 및 Azure DevOps Services는 개발, 스테이징, QA 및 프로덕션 환경과 같은 여러 환경에 릴리스를 위한 매우 구성 가능 하 고 관리 하기 쉬운 파이프라인을 제공 합니다. 특정 단계에서 승인 요구를 포함 합니다.

## <a name="create-release-definition"></a>릴리스 정의 만들기

1. **더하기** 단추를 선택 **하 여 Azure DevOps Services** 의 **빌드 및 릴리스** 섹션에서 릴리스 탭 아래에 새 릴리스를 추가 합니다.

    ![릴리스 정의 만들기](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. Azure App Service 배포 템플릿을 적용 합니다.

   ![Azure App Service 배포 템플릿 적용](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. **아티팩트 추가**에서 Azure 클라우드 빌드 앱에 대 한 아티팩트를 추가 합니다.

   ![Azure 클라우드 빌드에 아티팩트 추가](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. 파이프라인 탭에서 환경의 **단계, 작업** 링크를 선택 하 고 Azure 클라우드 환경 값을 설정 합니다.

   ![Azure 클라우드 환경 값 설정](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. **환경 이름을** 설정 하 고 azure 클라우드 끝점에 대 한 **azure 구독** 을 선택 합니다.

      ![Azure Cloud 끝점에 대 한 Azure 구독 선택](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. **App service name**에서 필요한 Azure App service 이름을 설정 합니다.

      ![Azure app service 이름 설정](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. Azure 클라우드 호스팅 환경에 대 한 **에이전트 큐** 아래에 "Hosted VS2017"를 입력 합니다.

      ![Azure 클라우드 호스팅 환경에 대 한 에이전트 큐 설정](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. 배포 Azure App Service 메뉴에서 환경의 올바른 **패키지 또는 폴더** 를 선택 합니다. **폴더 위치**에 **확인** 을 선택 합니다.
  
      ![Azure App Service 환경을 위한 패키지 또는 폴더 선택](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![Azure App Service 환경을 위한 패키지 또는 폴더 선택](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. 모든 변경 내용을 저장 하 고 **릴리스 파이프라인**으로 돌아갑니다.

    ![릴리스 파이프라인의 변경 내용 저장](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. Azure Stack 허브 앱에 대 한 빌드를 선택 하는 새 아티팩트를 추가 합니다.

    ![Azure Stack Hub 앱에 대 한 새 아티팩트 추가](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. Azure App Service 배포를 적용 하 여 환경을 하나 더 추가 합니다.

    ![Azure App Service 배포에 환경 추가](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. 새 환경의 이름을 "Azure Stack"로 합니다.

    ![Azure App Service 배포의 이름 환경](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. **작업** 탭에서 Azure Stack 환경을 찾습니다.

    ![Azure Stack 환경](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. Azure Stack 끝점에 대 한 구독을 선택 합니다.

    ![Azure Stack 끝점에 대 한 구독을 선택 합니다.](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. Azure Stack 웹 앱 이름을 App service 이름으로 설정 합니다.
    ![Azure Stack 웹 앱 이름 설정](media/solution-deployment-guide-cross-cloud-scaling/image20.png)

16. Azure Stack 에이전트를 선택 합니다.

    ![Azure Stack 에이전트를 선택 합니다.](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. 배포 Azure App Service 섹션에서 환경의 올바른 **패키지 또는 폴더** 를 선택 합니다. 폴더 위치에 **확인** 을 선택 합니다.

    ![Azure App Service 배포를 위한 폴더 선택](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![Azure App Service 배포를 위한 폴더 선택](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. 변수 탭에서 라는 변수를 추가 하 `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` 고 해당 값을 **true**로 설정 하 고 범위를 Azure Stack 합니다.

    ![Azure 앱 배포에 변수 추가](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. 두 아티팩트의 **연속** 배포 트리거 아이콘을 선택 하 고 **계속 해 서** 배포 트리거를 사용 하도록 설정 합니다.

    ![연속 배포 트리거 선택](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. Azure Stack 환경에서 **배포 전** 조건 아이콘을 선택 하 고 트리거를 **릴리스 후로 설정 합니다.**

    ![배포 전 조건 선택](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. 모든 변경 내용을 저장합니다.

> [!Note]  
> 템플릿에서 릴리스 정의를 만들 때 작업에 대 한 일부 설정이 [환경 변수로](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) 자동으로 정의 될 수 있습니다. 이러한 설정은 작업 설정에서 수정할 수 없습니다. 대신 이러한 설정을 편집 하려면 부모 환경 항목을 선택 해야 합니다.

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a>Visual Studio를 통해 Azure Stack 허브에 게시

Azure DevOps Services 빌드는 끝점을 만들어 Azure Stack 허브에 Azure 서비스 앱을 배포할 수 있습니다. Azure Pipelines 빌드 에이전트에 연결 되 고 Azure Stack 허브에 연결 됩니다.

1. Azure DevOps Services에 로그인 하 고 앱 설정 페이지로 이동 합니다.

2. **설정**에서 **보안**을 선택합니다.

3. **VSTS 그룹**에서 **끝점 작성자**를 선택 합니다.

4. **구성원** 탭에서 **추가**를 선택합니다.

5. 사용자 **및 그룹 추가**에서 사용자 이름을 입력 하 고 사용자 목록에서 해당 사용자를 선택 합니다.

6. **변경 내용 저장**을 선택합니다.

7. **VSTS 그룹** 목록에서 **끝점 관리자**를 선택 합니다.

8. **구성원** 탭에서 **추가**를 선택합니다.

9. 사용자 **및 그룹 추가**에서 사용자 이름을 입력 하 고 사용자 목록에서 해당 사용자를 선택 합니다.

10. **변경 내용 저장**을 선택합니다.

이제 끝점 정보가 있으므로 Azure Stack Hub 연결에 대 한 Azure Pipelines를 사용할 준비가 되었습니다. Azure Stack Hub의 빌드 에이전트는 Azure Pipelines에서 명령을 가져온 다음 에이전트 Azure Stack 허브와의 통신을 위해 끝점 정보를 전달 합니다.

## <a name="develop-the-app-build"></a>앱 빌드 개발

> [!Note]  
> 적절 한 이미지 게시를 실행 하는 Azure Stack 허브 (Windows Server 및 SQL) 및 App Service 배포가 필요 합니다. 자세한 내용은 [Azure Stack 허브에 App Service를 배포 하기 위한 필수 조건](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md)을 참조 하세요.

Azure Repos의 웹 앱 코드와 같은 [Azure Resource Manager 템플릿을](https://azure.microsoft.com/resources/templates/) 사용 하 여 두 클라우드 모두에 배포 합니다.

### <a name="add-code-to-an-azure-repos-project"></a>Azure Repos 프로젝트에 코드 추가

1. Azure Stack Hub에 대 한 프로젝트 만들기 권한이 있는 계정으로 Azure Repos에 로그인 합니다.

2. 기본 웹 앱을 만들고 열어 **리포지토리를 복제** 합니다.

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>두 클라우드에서 App Services에 대 한 자체 포함 웹 앱 배포 만들기

1. **WebApplication .csproj** 파일을 편집 합니다 .를 선택한 `Runtimeidentifier` 다음 추가를 클릭 `win10-x64` 합니다. 자세한 내용은 [자체 포함 된 배포](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) 설명서를 참조 하세요.

2. 팀 탐색기를 사용 하 여 Azure Repos 코드를 확인 합니다.

3. 앱 코드가 Azure Repos 체크 인 되었는지 확인 합니다.

### <a name="create-the-build-definition"></a>빌드 정의 만들기

1. 빌드 정의를 만들 수 있는 계정으로 Azure Pipelines에 로그인 합니다.

2. 프로젝트에 대 한 **웹 응용 프로그램 빌드** 페이지로 이동 합니다.

3. **인수**에서 **-r win10-x64** 코드를 추가 합니다. 이러한 추가는 .NET Core를 사용 하 여 자체 포함 된 배포를 트리거하는 데 필요 합니다.

4. 빌드를 실행 합니다. [자체 포함 배포 빌드](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) 프로세스는 Azure 및 Azure Stack 허브에서 실행할 수 있는 아티팩트를 게시 합니다.

#### <a name="use-an-azure-hosted-build-agent"></a>Azure 호스트 된 빌드 에이전트 사용

Azure Pipelines에서 호스팅된 빌드 에이전트를 사용 하는 것은 웹 앱을 빌드 및 배포 하는 편리한 옵션입니다. 유지 관리 및 업그레이드는 Microsoft Azure에 의해 자동으로 수행 되어 지속적이 고 중단 되지 않는 개발 주기를 가능 하 게 합니다.

### <a name="configure-the-continuous-deployment-cd-process"></a>CD (연속 배포) 프로세스 구성

Azure Pipelines 및 Azure DevOps Services는 개발, 스테이징, QA (품질 보증) 및 프로덕션과 같은 여러 환경에 릴리스를 위한 매우 구성 가능 하 고 관리 하기 쉬운 파이프라인을 제공 합니다. 이 프로세스에는 앱 수명 주기의 특정 단계에서 승인이 필요한 것으로 포함 될 수 있습니다.

#### <a name="create-release-definition"></a>릴리스 정의 만들기

릴리스 정의 만들기는 앱 빌드 프로세스의 마지막 단계입니다. 이 릴리스 정의는 릴리스를 만들고 빌드를 배포 하는 데 사용 됩니다.

1. Azure Pipelines에 로그인 하 고 프로젝트에 대 한 **빌드 및 릴리스** 로 이동 합니다.

2. **릴리스** 탭에서 **[+]** 를 선택한 다음 **릴리스 정의 만들기**를 선택 합니다.

3. **템플릿 선택**에서 **Azure App Service 배포**를 선택 하 고 **적용**을 선택 합니다.

4. **아티팩트 추가**의 **원본 (빌드 정의)** 에서 Azure 클라우드 빌드 앱을 선택 합니다.

5. **파이프라인** 탭에서 **환경 작업 보기**에 대 한 **1 단계**, **1 작업** 링크를 선택 합니다.

6. **작업** 탭에서 **환경 이름** 으로 azure를 입력 하 고 **Azure 구독** 목록에서 azurecloud Traders-웹 EP를 선택 합니다.

7. 다음 화면 캡처에 있는 **Azure app service 이름을**입력 합니다 `northwindtraders` .

8. 에이전트 단계의 경우 **에이전트 큐** 목록에서 **Hosted VS2017** 를 선택 합니다.

9. **Azure App Service 배포**에서 환경에 대 한 올바른 **패키지 또는 폴더** 를 선택 합니다.

10. **파일 또는 폴더 선택**에서 확인을 클릭 하 여 **위치**를 **확인** 합니다.

11. 모든 변경 내용을 저장 하 고 **파이프라인**으로 돌아갑니다.

12. **파이프라인** 탭에서 **아티팩트 추가**를 선택 하 고 **원본 (빌드 정의)** 목록에서 **NorthwindCloud 상인-용기** 를 선택 합니다.

13. **템플릿 선택**에서 다른 환경을 추가 합니다. **Azure App Service 배포** 를 선택 하 고 **적용**을 선택 합니다.

14. `Azure Stack Hub` **환경 이름**으로를 입력 합니다.

15. **작업** 탭에서 Azure Stack 허브를 찾아 선택 합니다.

16. **Azure 구독** 목록에서 Azure Stack 허브 끝점에 대해 **azurestack TRADERS-용기 EP** 를 선택 합니다.

17. Azure Stack 허브 웹 앱 이름을 **app service 이름**으로 입력 합니다.

18. **에이전트 선택**의 **에이전트 큐** 목록에서 **Azurestack-b Douglas 전나무** 를 선택 합니다.

19. **배포 Azure App Service**에 대해 환경에 유효한 **패키지 또는 폴더** 를 선택 합니다. **파일 또는 폴더 선택**에서 폴더 **위치**에 대해 **확인** 을 선택 합니다.

20. **변수** 탭에서 라는 변수를 찾습니다 `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` . 변수 값을 **true**로 설정 하 고 해당 범위를 **Azure Stack 허브**로 설정 합니다.

21. **파이프라인** 탭에서 NorthwindCloud 상인-웹 아티팩트의 **연속 배포 트리거** 아이콘을 선택 하 고 **연속 배포 트리거** 를 **사용**으로 설정 합니다. **NorthwindCloud 상인-용기** 아티팩트에 대해 동일한 작업을 수행 합니다.

22. Azure Stack 허브 환경의 경우 **배포 전 조건** 아이콘을 선택 하 여 트리거를 **릴리스 후**로 설정 합니다.

23. 모든 변경 내용을 저장합니다.

> [!Note]  
> 릴리스 작업에 대 한 일부 설정은 템플릿에서 릴리스 정의를 만들 때 [환경 변수로](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) 자동으로 정의 됩니다. 이러한 설정은 작업 설정에서 수정할 수 없지만 부모 환경 항목에서는 수정할 수 있습니다.

## <a name="create-a-release"></a>릴리스 만들기

1. **파이프라인** 탭에서 **릴리스** 목록을 열고 **릴리스 만들기**를 선택 합니다.

2. 릴리스에 대 한 설명을 입력 하 고, 올바른 아티팩트가 선택 되었는지 확인 한 다음, **만들기**를 선택 합니다. 잠시 후 새 릴리스가 만들어지고 릴리스 이름이 링크로 표시 됨을 나타내는 배너가 표시 됩니다. 링크를 선택 하 여 릴리스 요약 페이지를 표시 합니다.

3. 릴리스 요약 페이지에는 릴리스에 대 한 세부 정보가 표시 됩니다. "릴리스-2"에 대 한 다음 화면 캡처에서 **환경** 섹션에는 Azure에 대 한 **배포 상태가** "진행 중"으로 표시 되 고 Azure Stack 허브의 상태가 "성공"으로 표시 됩니다. Azure 환경의 배포 상태가 "성공"으로 변경 되 면 릴리스가 승인 될 준비가 되었음을 나타내는 배너가 표시 됩니다. 배포가 보류 중이거나 실패 한 경우 파란색 **(i)** 정보 아이콘이 표시 됩니다. 아이콘을 마우스로 가리키면 지연 또는 실패의 이유가 포함 된 팝업이 표시 됩니다.

4. 릴리스 목록과 같은 다른 보기에는 승인이 보류 중임을 나타내는 아이콘도 표시 됩니다. 이 아이콘의 팝업은 환경 이름 및 배포와 관련 된 자세한 정보를 표시 합니다. 관리자는 전반적인 릴리스 진행률을 확인 하 고 승인 대기 중인 릴리스를 확인 하는 것이 쉽습니다.

## <a name="monitor-and-track-deployments"></a>배포 모니터링 및 추적

1. **릴리스 2** 요약 페이지에서 **로그**를 선택 합니다. 배포 하는 동안이 페이지에는 에이전트의 실시간 로그가 표시 됩니다. 왼쪽 창에는 각 환경에 대 한 배포의 각 작업 상태가 표시 됩니다.

2. 배포 전 또는 배포 후 승인의 **작업** 열에서 사용자 아이콘을 선택 하 여 배포와 해당 사용자가 제공한 메시지를 승인 하거나 거부 한 사용자를 확인 합니다.

3. 배포가 완료 되 면 전체 로그 파일이 오른쪽 창에 표시 됩니다. 왼쪽 창에서 **단계** 를 선택 하 여 **작업 초기화**와 같은 한 단계에 대 한 로그 파일을 확인 합니다. 개별 로그를 확인 하는 기능을 사용 하면 전체 배포의 일부를 쉽게 추적 하 고 디버그할 수 있습니다. 단계에 대 한 로그 파일을 **저장** 하거나 **모든 로그를 zip으로 다운로드**합니다.

4. **요약** 탭을 열어 릴리스에 대 한 일반 정보를 확인 합니다. 이 보기에는 빌드, 배포 된 환경, 배포 상태 및 릴리스에 대 한 기타 정보에 대 한 세부 정보가 표시 됩니다.

5. 특정 환경에 대 한 기존 배포 및 보류 중인 배포에 대 한 정보를 보려면 환경 링크 (**Azure** 또는 **Azure Stack 허브**)를 선택 합니다. 이러한 뷰를 사용 하 여 동일한 빌드가 두 환경에 배포 되었는지 확인 하는 빠른 방법입니다.

6. 배포 된 **프로덕션 앱** 을 브라우저에서 엽니다. 예를 들어 Azure 앱 Services 웹 사이트의 경우 URL을 엽니다 `https://[your-app-name\].azurewebsites.net` .

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a>Azure와 Azure Stack Hub의 통합은 확장 가능한 클라우드 간 솔루션을 제공 합니다.

유연 하 고 강력한 다중 클라우드 서비스는 데이터 보안, 백업 및 중복성, 일관 되 고 빠른 가용성, 확장 가능한 저장소 및 배포, 지리적 규격 라우팅을 제공 합니다. 수동으로 트리거된이 프로세스를 통해 호스팅된 웹 앱과 중요 한 데이터의 즉각적인 가용성 간에 안정적이 고 효율적인 부하를 전환할 수 있습니다.

## <a name="next-steps"></a>다음 단계

- Azure Cloud 패턴에 대해 자세히 알아보려면 [클라우드 디자인 패턴](https://docs.microsoft.com/azure/architecture/patterns)을 참조 하세요.
