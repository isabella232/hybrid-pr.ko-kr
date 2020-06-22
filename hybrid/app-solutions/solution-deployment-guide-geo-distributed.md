---
title: Azure 및 Azure Stack Hub를 사용 하 여 지리적으로 분산 된 앱으로 트래픽 직접 전송
description: Azure 및 Azure Stack Hub를 사용 하 여 지리적으로 분산 된 앱 솔루션으로 트래픽을 특정 끝점으로 전달 하는 방법을 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 8f2b7e48a62896acfce7293dcd4f18d5a43add01
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911323"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a>Azure 및 Azure Stack Hub를 사용 하 여 지리적으로 분산 된 앱으로 트래픽 직접 전송

지리적으로 분산 된 앱 패턴을 사용 하 여 다양 한 메트릭에 따라 특정 끝점으로 트래픽을 전송 하는 방법을 알아봅니다. 지리적 기반 라우팅 및 끝점 구성을 사용 하 여 Traffic Manager 프로필을 만들면 정보가 지역 요구 사항, 회사 및 국제 규정 및 데이터 요구 사항에 따라 끝점으로 라우팅됩니다.

이 솔루션에서는 다음을 수행 하는 샘플 환경을 빌드합니다.

> [!div class="checklist"]
> - 지리적으로 분산 된 앱을 만듭니다.
> - Traffic Manager를 사용 하 여 앱을 대상으로 합니다.

## <a name="use-the-geo-distributed-apps-pattern"></a>지리적으로 분산 된 앱 패턴 사용

지리적으로 분산 된 패턴을 사용 하 여 앱은 지역에 걸쳐 있습니다. 공용 클라우드를 기본적으로 사용할 수 있지만 일부 사용자가 해당 데이터를 해당 지역에 유지 해야 할 수도 있습니다. 사용자를 요구 사항에 따라 가장 적합 한 클라우드로 안내할 수 있습니다.

### <a name="issues-and-considerations"></a>문제 및 고려 사항

#### <a name="scalability-considerations"></a>확장성 고려 사항

이 문서를 사용 하 여 빌드할 솔루션은 확장성을 고려 하지 않습니다. 그러나 다른 Azure 및 온-프레미스 솔루션과 함께 사용 하는 경우 확장성 요구 사항을 수용할 수 있습니다. Traffic manager를 통해 자동 크기 조정을 사용 하는 하이브리드 솔루션을 만드는 방법에 대 한 자세한 내용은 [Azure를 사용 하 여 클라우드 간 확장 솔루션 만들기](solution-deployment-guide-cross-cloud-scaling.md)를 참조 하세요.

#### <a name="availability-considerations"></a>가용성 고려 사항

확장성을 고려 하는 경우에도이 솔루션은 가용성을 직접 해결 하지 않습니다. 그러나이 솔루션 내에서 Azure 및 온-프레미스 솔루션을 구현 하 여 관련 된 모든 구성 요소에 대 한 고가용성을 보장할 수 있습니다.

### <a name="when-to-use-this-pattern"></a>이 패턴을 사용해야 하는 경우

- 조직에 사용자 지정 지역 보안 및 배포 정책을 요구 하는 국제 분기가 있습니다.

- 각 조직 사무소는 직원, 비즈니스 및 시설 데이터를 가져오며,이를 통해 지역 규정 및 표준 시간대 별로 보고 작업을 수행 해야 합니다.

- 단일 지역 내에서 여러 앱 배포를 사용 하 여 응용 프로그램을 수평으로 확장 하 고 많은 지역에 걸쳐 응용 프로그램을 수평으로 확장 하 여 매우 많은 요구 사항을 충족할 수 있습니다.

### <a name="planning-the-topology"></a>토폴로지 계획

분산 된 앱 공간을 빌드하기 전에 다음 사항을 파악 하는 것이 좋습니다.

- **앱에 대 한 사용자 지정 도메인:** 고객이 앱에 액세스 하는 데 사용할 사용자 지정 도메인 이름은 무엇 인가요? 샘플 앱의 경우 사용자 지정 도메인 이름은 *www \. scalableasedemo.com입니다.*

- **도메인 Traffic Manager:** [Azure Traffic Manager 프로필](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-manage-profiles)을 만들 때 도메인 이름이 선택 됩니다. 이 이름은 Traffic Manager로 관리 되는 도메인 항목을 등록 하기 위해 *trafficmanager.net* 접미사와 결합 됩니다. 샘플 앱의 경우 선택한 이름은 *scalable-ase-demo*입니다. 따라서 Traffic Manager에서 관리 하는 전체 도메인 이름은 *scalable-ase-demo.trafficmanager.net*입니다.

- **앱 사용 공간의 크기를 조정 하는 전략:** 앱 공간을 단일 지역 또는 여러 지역에 있는 여러 App Service 환경에 배포할 것인지 아니면 두 방법의 조합으로 배포할 것인지 결정 합니다. 결정은 고객 트래픽이 발생 하는 위치와 앱이 지 원하는 백 엔드 인프라의 나머지 부분이 얼마나 잘 확장 될 수 있는지에 따라 결정 해야 합니다. 예를 들어 100% 상태 비저장 앱을 사용 하는 경우 Azure 지역 마다 여러 App Service 환경의 조합을 사용 하 여 앱을 대규모 확장할 수 있으며 여러 Azure 지역에 배포 된 App Service 환경에 곱합니다. Azure에서 15 개 이상의 글로벌 Azure 지역을 선택할 수 있으므로, 고객은 세계 전체의 하이퍼 규모 앱 사용 공간을 실제로 구축할 수 있습니다. 여기에 사용 된 샘플 앱의 경우 단일 Azure 지역 (미국 중부)에서 세 개의 App Service 환경을 만들었습니다.

- **App Service 환경의 명명 규칙:** 각 App Service 환경에는 고유한 이름이 필요 합니다. 하나 또는 두 개의 App Service 환경 이외에 각 App Service 환경을 식별 하는 데 도움이 되는 명명 규칙을 포함 하는 것이 좋습니다. 여기에 사용 된 샘플 앱의 경우 간단한 명명 규칙을 사용 했습니다. 세 가지 App Service 환경의 이름은 *fe1ase*, *fe2ase*및 *fe3ase*입니다.

- **앱에 대 한 명명 규칙:** 앱의 여러 인스턴스가 배포 되므로 배포 된 앱의 각 인스턴스에 대해 이름이 필요 합니다. Power Apps에 대 한 App Service Environment를 사용 하면 동일한 앱 이름을 여러 환경에서 사용할 수 있습니다. 각 App Service 환경에 고유한 도메인 접미사가 있으므로 개발자는 각 환경에서 정확히 동일한 앱 이름을 다시 사용 하도록 선택할 수 있습니다. 예를 들어 개발자에 게 이름이 *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net*등 인 앱이 있을 수 있습니다. 여기에서 사용 되는 앱의 경우 각 앱 인스턴스에 고유한 이름이 있습니다. 앱 인스턴스에 사용되는 이름은 *webfrontend1*, *webfrontend2* 및 *webfrontend3*입니다.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack 허브는 Azure의 확장입니다. Azure Stack 허브는 온-프레미스 환경에 클라우드 컴퓨팅의 민첩성과 혁신을 제공 하 여 어디서 나 하이브리드 앱을 빌드하고 배포할 수 있는 유일한 하이브리드 클라우드를 사용 하도록 설정 합니다.  
> 
> [하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 문서에서는 하이브리드 앱을 디자인, 배포 및 운영 하기 위한 소프트웨어 품질 (배치, 확장성, 가용성, 복원 력, 관리 효율성 및 보안)의 핵심 요소을 검토 합니다. 디자인 고려 사항은 하이브리드 앱 디자인을 최적화 하 고 프로덕션 환경에서 문제를 최소화 하는 데 도움이 됩니다.

## <a name="part-1-create-a-geo-distributed-app"></a>1 부: 지리적으로 분산 된 앱 만들기

이 부분에서는 웹 앱을 만듭니다.

> [!div class="checklist"]
> - 웹 앱을 만들고 게시 합니다.
> - Azure Repos에 코드를 추가 합니다.
> - 앱이 여러 클라우드 대상에 대해 빌드를 가리킵니다.
> - CD 프로세스를 관리 하 고 구성 합니다.

### <a name="prerequisites"></a>필수 조건

Azure 구독 및 Azure Stack 허브 설치가 필요 합니다.

### <a name="geo-distributed-app-steps"></a>지리적으로 분산 되는 앱 단계

### <a name="obtain-a-custom-domain-and-configure-dns"></a>사용자 지정 도메인 가져오기 및 DNS 구성

도메인에 대 한 DNS 영역 파일을 업데이트 합니다. 그러면 Azure AD에서 사용자 지정 도메인 이름의 소유권을 확인할 수 있습니다. Azure 내에서 Azure/Office 365/외부 DNS 레코드에 대 한 [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) 를 사용 하거나 [다른 DNS 등록자](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/)에 dns 항목을 추가 합니다.

1. 공용 등록자에 게 사용자 지정 도메인을 등록 합니다.

2. 도메인에 대한 도메인 이름 등록 기관에 로그인합니다. 승인 된 관리자가 DNS 업데이트를 수행 해야 할 수 있습니다.

3. Azure AD에서 제공 하는 DNS 항목을 추가 하 여 도메인에 대 한 DNS 영역 파일을 업데이트 합니다. DNS 항목은 메일 라우팅 또는 웹 호스팅 등의 동작을 변경 하지 않습니다.

### <a name="create-web-apps-and-publish"></a>웹 앱 만들기 및 게시

웹 앱을 Azure Stack Azure에 배포 하는 CI/CD (연속 통합/지속적인 업데이트)를 설정 하 고, 두 클라우드 모두에 변경 내용을 자동으로 푸시합니다.

> [!Note]  
> 적절 한 이미지 게시를 실행 하는 Azure Stack 허브 (Windows Server 및 SQL) 및 App Service 배포가 필요 합니다. 자세한 내용은 [Azure Stack 허브에 App Service를 배포 하기 위한 필수 조건](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md)을 참조 하세요.

#### <a name="add-code-to-azure-repos"></a>Azure Repos에 코드 추가

1. Azure Repos에 대 한 **프로젝트 만들기 권한이 있는 계정** 으로 Visual Studio에 로그인 합니다.

    CI/CD는 앱 코드와 인프라 코드 모두에 적용할 수 있습니다. 사설 및 호스 티 드 클라우드 개발 모두에 [Azure Resource Manager 템플릿을](https://azure.microsoft.com/resources/templates/) 사용 합니다.

    ![Visual Studio에서 프로젝트에 연결](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. 기본 웹 앱을 만들고 열어 **리포지토리를 복제** 합니다.

    ![Visual Studio에서 리포지토리 복제](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a>두 클라우드에서 웹 앱 배포 만들기

1. **WebApplication .csproj** 파일 (선택 `Runtimeidentifier` 및 추가)을 편집 `win10-x64` 합니다. [자체 포함 된 배포](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) 설명서를 참조 하세요.

    ![Visual Studio에서 웹 앱 프로젝트 파일 편집](media/solution-deployment-guide-geo-distributed/image3.png)

2. 팀 탐색기를 사용 하 여 **Azure Repos 코드를 체크 인** 합니다.

3. **응용 프로그램 코드가** Azure Repos 체크 인 되었는지 확인 합니다.

### <a name="create-the-build-definition"></a>빌드 정의 만들기

1. **Azure Pipelines에 로그인** 하 여 빌드 정의를 만들 수 있는지 확인 합니다.

2. `-r win10-x64`코드를 추가 합니다. 이러한 추가는 .NET Core를 사용 하 여 자체 포함 된 배포를 트리거하기 위해 필요 합니다.

    ![Azure Pipelines의 빌드 정의에 코드를 추가 합니다.](media/solution-deployment-guide-geo-distributed/image4.png)

3. **빌드를 실행**합니다. [자체 포함 배포 빌드](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) 프로세스는 Azure 및 Azure Stack 허브에서 실행할 수 있는 아티팩트를 게시 합니다.

#### <a name="using-an-azure-hosted-agent"></a>Azure 호스트 된 에이전트 사용

Azure Pipelines에서 호스팅된 에이전트를 사용 하는 것은 웹 앱을 빌드하고 배포 하는 편리한 옵션입니다. 유지 관리 및 업그레이드는 중단 없이 개발, 테스트 및 배포를 가능 하 게 하는 Microsoft Azure에 의해 자동으로 수행 됩니다.

### <a name="manage-and-configure-the-cd-process"></a>CD 프로세스 관리 및 구성

개발, 스테이징, QA 및 프로덕션 환경과 같은 여러 환경에 릴리스를 위한 매우 구성 가능 하 고 관리 하기 쉬운 파이프라인을 제공 Azure DevOps Services 합니다. 특정 단계에서 승인 요구를 포함 합니다.

## <a name="create-release-definition"></a>릴리스 정의 만들기

1. **더하기** 단추를 선택 **하 여 Azure DevOps Services** 의 **빌드 및 릴리스** 섹션에서 릴리스 탭 아래에 새 릴리스를 추가 합니다.

    ![Azure DevOps Services에서 릴리스 정의 만들기](media/solution-deployment-guide-geo-distributed/image5.png)

2. Azure App Service 배포 템플릿을 적용 합니다.

   ![Azure DevOps Services에서 Azure App Service 배포 템플릿 적용](media/solution-deployment-guide-geo-distributed/image6.png)

3. **아티팩트 추가**에서 Azure 클라우드 빌드 앱에 대 한 아티팩트를 추가 합니다.

   ![Azure DevOps Services에서 Azure 클라우드 빌드에 아티팩트 추가](media/solution-deployment-guide-geo-distributed/image7.png)

4. 파이프라인 탭에서 환경의 **단계, 작업** 링크를 선택 하 고 Azure 클라우드 환경 값을 설정 합니다.

   ![Azure DevOps Services에서 Azure 클라우드 환경 값 설정](media/solution-deployment-guide-geo-distributed/image8.png)

5. **환경 이름을** 설정 하 고 azure 클라우드 끝점에 대 한 **azure 구독** 을 선택 합니다.

      ![Azure DevOps Services에서 Azure 클라우드 끝점에 대 한 Azure 구독을 선택 합니다.](media/solution-deployment-guide-geo-distributed/image9.png)

6. **App service name**에서 필요한 Azure App service 이름을 설정 합니다.

      ![Azure DevOps Services에서 Azure app service 이름 설정](media/solution-deployment-guide-geo-distributed/image10.png)

7. Azure 클라우드 호스팅 환경에 대 한 **에이전트 큐** 아래에 "Hosted VS2017"를 입력 합니다.

      ![Azure DevOps Services에서 Azure 클라우드 호스팅 환경에 대 한 에이전트 큐 설정](media/solution-deployment-guide-geo-distributed/image11.png)

8. 배포 Azure App Service 메뉴에서 환경의 올바른 **패키지 또는 폴더** 를 선택 합니다. **폴더 위치**에 **확인** 을 선택 합니다.
  
      ![Azure DevOps Services에서 Azure App Service 환경을 위한 패키지 또는 폴더를 선택 합니다.](media/solution-deployment-guide-geo-distributed/image12.png)

      ![Azure DevOps Services에서 Azure App Service 환경을 위한 패키지 또는 폴더를 선택 합니다.](media/solution-deployment-guide-geo-distributed/image13.png)

9. 모든 변경 내용을 저장 하 고 **릴리스 파이프라인**으로 돌아갑니다.

    ![Azure DevOps Services의 릴리스 파이프라인 변경 내용 저장](media/solution-deployment-guide-geo-distributed/image14.png)

10. Azure Stack 허브 앱에 대 한 빌드를 선택 하는 새 아티팩트를 추가 합니다.

    ![Azure DevOps Services에서 Azure Stack 허브 앱에 대 한 새 아티팩트 추가](media/solution-deployment-guide-geo-distributed/image15.png)


11. Azure App Service 배포를 적용 하 여 환경을 하나 더 추가 합니다.

    ![Azure DevOps Services에서 Azure App Service 배포에 환경 추가](media/solution-deployment-guide-geo-distributed/image16.png)

12. 새 환경의 이름을 Hub Azure Stack 합니다.

    ![Azure DevOps Services에서 Azure App Service 배포의 이름 환경](media/solution-deployment-guide-geo-distributed/image17.png)

13. **작업** 탭에서 Azure Stack 허브 환경을 찾습니다.

    ![Azure DevOps Services의 Azure DevOps Services에 Azure Stack 허브 환경](media/solution-deployment-guide-geo-distributed/image18.png)

14. Azure Stack 허브 끝점에 대 한 구독을 선택 합니다.

    ![Azure DevOps Services에서 Azure Stack 허브 끝점에 대 한 구독을 선택 합니다.](media/solution-deployment-guide-geo-distributed/image19.png)

15. Azure Stack 허브 웹 앱 이름을 App service 이름으로 설정 합니다.

    ![Azure DevOps Services에서 Azure Stack 허브 웹 앱 이름 설정](media/solution-deployment-guide-geo-distributed/image20.png)

16. Azure Stack 허브 에이전트를 선택 합니다.

    ![Azure DevOps Services에서 Azure Stack 허브 에이전트를 선택 합니다.](media/solution-deployment-guide-geo-distributed/image21.png)

17. 배포 Azure App Service 섹션에서 환경의 올바른 **패키지 또는 폴더** 를 선택 합니다. 폴더 위치에 **확인** 을 선택 합니다.

    ![Azure DevOps Services에서 Azure App Service 배포를 위한 폴더를 선택 하십시오.](media/solution-deployment-guide-geo-distributed/image22.png)

    ![Azure DevOps Services에서 Azure App Service 배포를 위한 폴더를 선택 하십시오.](media/solution-deployment-guide-geo-distributed/image23.png)

18. 변수 탭에서 라는 변수를 추가 하 `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` 고 해당 값을 **true**로, 범위를 Azure Stack 허브로 설정 합니다.

    ![Azure DevOps Services에서 Azure 앱 배포에 변수 추가](media/solution-deployment-guide-geo-distributed/image24.png)

19. 두 아티팩트의 **연속** 배포 트리거 아이콘을 선택 하 고 **계속 해 서** 배포 트리거를 사용 하도록 설정 합니다.

    ![Azure DevOps Services에서 연속 배포 트리거를 선택 합니다.](media/solution-deployment-guide-geo-distributed/image25.png)

20. Azure Stack 허브 환경에서 **배포 전** 조건 아이콘을 선택 하 고 트리거를 **릴리스 후로 설정 합니다.**

    ![Azure DevOps Services에서 배포 전 조건 선택](media/solution-deployment-guide-geo-distributed/image26.png)

21. 모든 변경 내용을 저장합니다.

> [!Note]  
> 템플릿에서 릴리스 정의를 만들 때 작업에 대 한 일부 설정이 [환경 변수로](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) 자동으로 정의 될 수 있습니다. 이러한 설정은 작업 설정에서 수정할 수 없습니다. 대신 이러한 설정을 편집 하려면 부모 환경 항목을 선택 해야 합니다.

## <a name="part-2-update-web-app-options"></a>2 부: 웹 앱 옵션 업데이트

[Azure App Service](https://docs.microsoft.com/azure/app-service/overview)는 확장성 높은 자체 패치 웹 호스팅 서비스를 제공합니다.

![Azure App Service](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - 기존 사용자 지정 DNS 이름을 Azure Web Apps에 매핑합니다.
> - **CNAME 레코드** 와 **a 레코드** 를 사용 하 여 사용자 지정 DNS 이름을 App Service에 매핑합니다.

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a>Azure Web Apps에 기존 사용자 지정 DNS 이름 매핑

> [!Note]  
> 루트 도메인 (예: northwind.com)을 제외한 모든 사용자 지정 DNS 이름에 대해 CNAME을 사용 합니다.

라이브 사이트 및 해당 DNS 도메인 이름을 App Service로 마이그레이션하려면 [활성 DNS 이름을 Azure App Service로 마이그레이션](https://docs.microsoft.com/azure/app-service/manage-custom-dns-migrate-domain)을 참조하세요.

### <a name="prerequisites"></a>필수 조건

이 솔루션을 완료 하려면:

- [App Service 앱을 만들거나](https://docs.microsoft.com/azure/app-service/)다른 솔루션에 대해 만든 앱을 사용 합니다.

- 도메인 이름을 구매 하 고 도메인 공급자의 DNS 레지스트리에 대 한 액세스를 확인 합니다.

도메인에 대 한 DNS 영역 파일을 업데이트 합니다. Azure AD에서 사용자 지정 도메인 이름의 소유권을 확인 합니다. Azure 내에서 Azure/Office 365/외부 DNS 레코드에 대 한 [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) 를 사용 하거나 [다른 DNS 등록자](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/)에 dns 항목을 추가 합니다.

- 공용 등록자에 게 사용자 지정 도메인을 등록 합니다.

- 도메인에 대한 도메인 이름 등록 기관에 로그인합니다. (승인 된 관리자는 DNS를 업데이트 해야 할 수도 있습니다.)

- Azure AD에서 제공 하는 DNS 항목을 추가 하 여 도메인에 대 한 DNS 영역 파일을 업데이트 합니다.

예를 들어 northwindcloud.com 및 www northwindcloud.com에 대 한 DNS 항목을 추가 하려면 \. northwindcloud.com 루트 도메인에 대 한 dns 설정을 구성 합니다.

> [!Note]  
> [Azure Portal](https://docs.microsoft.com/azure/app-service/manage-custom-dns-buy-domain)를 사용 하 여 도메인 이름을 구입할 수 있습니다. 사용자 지정 DNS 이름을 웹앱에 매핑하려면 [App Service 계획](https://azure.microsoft.com/pricing/details/app-service/)이 유료 계층(**공유**, **기본**, **표준** 또는 **프리미엄**)이어야 합니다.

### <a name="create-and-map-cname-and-a-records"></a>CNAME 및 A 레코드 만들기 및 매핑

#### <a name="access-dns-records-with-domain-provider"></a>도메인 공급자로 DNS 레코드 액세스

> [!Note]  
>  Azure DNS를 사용 하 여 Azure Web Apps에 대 한 사용자 지정 DNS 이름을 구성 합니다. 자세한 내용은 [Azure DNS를 사용하여 Azure 서비스에 대해 사용자 지정 도메인 설정 제공](https://docs.microsoft.com/azure/dns/dns-custom-domain)을 참조하세요.

1. 주 공급자의 웹 사이트에 로그인 합니다.

2. DNS 레코드를 관리하기 위한 페이지를 찾습니다. 모든 도메인 공급자에는 자체 DNS 레코드 인터페이스가 있습니다. **도메인 이름**, **DNS** 또는 **이름 서버 관리**라는 레이블이 지정된 사이트 영역을 찾습니다.

DNS 레코드 페이지는 **내 도메인**에서 볼 수 있습니다. **영역 파일**, **DNS 레코드**또는 **고급 구성**이라는 링크를 찾습니다.

다음 스크린샷은 DNS 레코드 페이지의 예입니다.

![DNS 레코드 페이지 예](media/solution-deployment-guide-geo-distributed/image28.png)

1. 도메인 이름 등록자에서 **추가 또는 만들기** 를 선택 하 여 레코드를 만듭니다. 일부 공급자에는 다른 레코드 형식을 추가하는 다양한 링크가 있습니다. 공급자 설명서를 참조 하십시오.

2. CNAME 레코드를 추가 하 여 하위 도메인을 앱의 기본 호스트 이름에 매핑합니다.

   Www \. northwindcloud.com 도메인 예의 경우 이름을에 매핑하는 CNAME 레코드를 추가 `<app_name>.azurewebsites.net` 합니다.

CNAME을 추가 하면 DNS 레코드 페이지가 다음 예제와 같습니다.

![Azure 앱에 대한 포털 탐색](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a>Azure에서 CNAME 레코드 매핑 사용

1. 새 탭에서 Azure Portal에 로그인 합니다.

2. App Services로 이동합니다.

3. 웹 앱을 선택 합니다.

4. Azure Portal의 앱 페이지 왼쪽 탐색 영역에서 **사용자 지정 도메인**을 선택합니다.

5. **+** **호스트 이름 추가**옆에 있는 아이콘을 선택 합니다.

6. 정규화 된 도메인 이름 (예:)을 입력 합니다 `www.northwindcloud.com` .

7. **유효성 검사**를 선택합니다.

8. 표시 되는 경우 다른 형식 (또는)의 다른 레코드 `A` `TXT` 를 도메인 이름 등록 기관 DNS 레코드에 추가 합니다. Azure는 이러한 레코드의 값과 유형을 제공 합니다.

   a.  앱의 IP 주소에 매핑할 **A** 레코드

   b.  앱의 기본 호스트 이름(`<app_name>.azurewebsites.net`)에 매핑할 **TXT** 레코드 - App Service는 구성 시에만이 레코드를 사용 하 여 사용자 지정 도메인 소유권을 확인 합니다. 확인 후 TXT 레코드를 삭제 합니다.

9. 도메인 등록자 탭에서이 작업을 완료 하 고 **호스트 이름 추가** 단추가 활성화 될 때까지 유효성을 다시 검사 합니다.

10. **호스트 이름 레코드 형식이** **CNAME** (www.example.com 또는 하위 도메인)으로 설정 되어 있는지 확인 합니다.

11. **호스트 이름 추가**를 선택합니다.

12. 정규화 된 도메인 이름 (예:)을 입력 합니다 `northwindcloud.com` .

13. **유효성 검사**를 선택합니다. **추가** 가 활성화 됩니다.

14. **호스트 이름 레코드 형식이** **레코드** (example.com)로 설정 되어 있는지 확인 합니다.

15. **호스트 이름을 추가**합니다.

    새 호스트 이름이 앱의 **사용자 지정 도메인** 페이지에 반영 되는 데 약간의 시간이 걸릴 수 있습니다. 데이터를 업데이트하려면 브라우저를 새로 고쳐 보세요.
  
    ![사용자 지정 도메인](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    오류가 발생 하면 페이지 아래쪽에 확인 오류 알림이 표시 됩니다. ![도메인 확인 오류](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  위의 단계를 반복 하 여 와일드 카드 도메인 ( \* . northwindcloud.com)을 매핑할 수 있습니다. 이렇게 하면 각 항목에 대 한 별도의 CNAME 레코드를 만들지 않고도이 app service에 추가 하위 도메인을 추가할 수 있습니다. 등록자 지침에 따라이 설정을 구성 합니다.

#### <a name="test-in-a-browser"></a>브라우저에서 테스트

이전에 구성 된 DNS 이름 (예: `northwindcloud.com` 또는)으로 이동 `www.northwindcloud.com` 합니다.

## <a name="part-3-bind-a-custom-ssl-cert"></a>3 부: 사용자 지정 SSL 인증서 바인딩

이 부분에서는 다음을 수행 합니다.

> [!div class="checklist"]
> - App Service에 사용자 지정 SSL 인증서를 바인딩합니다.
> - 앱에 HTTPS를 적용 합니다.
> - 스크립트를 사용 하 여 SSL 인증서 바인딩을 자동화 합니다.

> [!Note]  
> 필요한 경우 Azure Portal에서 고객 SSL 인증서를 가져와 웹 앱에 바인딩합니다. 자세한 내용은 [App Service 인증서 자습서](https://docs.microsoft.com/azure/app-service/web-sites-purchase-ssl-web-site)를 참조 하세요.

### <a name="prerequisites"></a>필수 조건

이 솔루션을 완료 하려면:

- [App Service 앱을 만듭니다.](https://docs.microsoft.com/azure/app-service/)
- [사용자 지정 DNS 이름을 웹 앱에 매핑합니다.](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain)
- 신뢰할 수 있는 인증 기관에서 SSL 인증서를 획득 하 고 키를 사용 하 여 요청에 서명 합니다.

### <a name="requirements-for-your-ssl-certificate"></a>SSL 인증서에 대한 요구 사항

App Service에서 인증서를 사용하려면 인증서가 다음 요구 사항을 모두 충족해야 합니다.

- 신뢰할 수 있는 인증 기관에서 서명 합니다.

- 암호로 보호 된 PFX 파일로 내보냅니다.

- 최소 2048 비트 길이의 개인 키를 포함 합니다.

- 인증서 체인의 모든 중간 인증서를 포함 합니다.

> [!Note]  
> **ECC (타원 Curve Cryptography) 인증서** App Service에서 작동 하지만이 가이드에는 포함 되어 있지 않습니다. ECC 인증서를 만드는 방법에 대 한 자세한 내용은 인증 기관을 참조 하세요.

#### <a name="prepare-the-web-app"></a>웹 앱 준비

웹 앱에 사용자 지정 SSL 인증서를 바인딩하려면 [App Service 계획이](https://azure.microsoft.com/pricing/details/app-service/) **기본**, **표준**또는 **프리미엄** 계층에 있어야 합니다.

#### <a name="sign-in-to-azure"></a>Azure에 로그인

1. [Azure Portal](https://portal.azure.com/) 열고 웹 앱으로 이동 합니다.

2. 왼쪽 메뉴에서 **App Services**를 선택 하 고 웹 앱 이름을 선택 합니다.

![Azure Portal에서 웹 앱을 선택 합니다.](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a>가격 책정 계층 확인

1. 웹 앱 페이지의 왼쪽 탐색 영역에서 **설정** 섹션으로 스크롤하고 강화 **(App Service 계획)** 를 선택 합니다.

    ![웹 앱의 확장 메뉴](media/solution-deployment-guide-geo-distributed/image34.png)

1. 웹 앱이 **무료** 또는 **공유** 계층에 있지 않은지 확인 합니다. 웹 앱의 현재 계층이 진한 파란색 상자로 강조 표시 됩니다.

    ![웹 앱의 가격 책정 계층 확인](media/solution-deployment-guide-geo-distributed/image35.png)

사용자 지정 SSL은 **무료** 또는 **공유** 계층에서 지원 되지 않습니다. Upscale 하려면 다음 섹션 또는 **가격 책정 계층 선택** 페이지의 단계를 수행 하 고 [SSL 인증서 업로드 및 바인딩](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl)을 건너뜁니다.

#### <a name="scale-up-your-app-service-plan"></a>App Service 계획 강화

1. **기본**, **표준** 또는 **프리미엄** 계층 중 하나를 선택합니다.

2. **선택**을 선택합니다.

![웹 앱에 대 한 가격 책정 계층 선택](media/solution-deployment-guide-geo-distributed/image36.png)

알림이 표시 되 면 크기 조정 작업이 완료 됩니다.

![강화 알림](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a>SSL 인증서 바인딩 및 중간 인증서 병합

체인의 여러 인증서를 병합 합니다.

1. 텍스트 편집기에서 받은 **각 인증서를 엽니다** .

2. *Mergedcertificate*라는 병합 된 인증서에 대 한 파일을 만듭니다. 텍스트 편집기에서 각 인증서의 내용을 이 파일에 복사합니다. 사용자 인증서의 순서는 사용자의 인증서로 시작하고 루트 인증서로 끝나는 인증서 체인의 순서에 따라야 합니다. 다음 예제와 유사합니다.

    ```Text

    -----BEGIN CERTIFICATE-----

    <your entire Base64 encoded SSL certificate>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 1>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 2>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded root certificate>

    -----END CERTIFICATE-----
    ```

#### <a name="export-certificate-to-pfx"></a>PFX로 인증서 내보내기

인증서로 생성 된 개인 키를 사용 하 여 병합 된 SSL 인증서를 내보냅니다.

개인 키 파일은 OpenSSL를 통해 생성 됩니다. 인증서를 PFX로 내보내려면 다음 명령을 실행 하 고 자리 표시자를 `<private-key-file>` `<merged-certificate-file>` 개인 키 경로 및 병합 된 인증서 파일로 바꿉니다.

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

메시지가 표시 되 면 나중에 App Service 위해 SSL 인증서를 업로드 하기 위한 내보내기 암호를 정의 합니다.

인증서 요청을 생성 하는 데 IIS 또는 **Certreq.exe** 를 사용 하는 경우 인증서를 로컬 컴퓨터에 설치한 다음 [인증서를 PFX로 내보냅니다](https://technet.microsoft.com/library/cc754329(v=ws.11).aspx).

#### <a name="upload-the-ssl-certificate"></a>SSL 인증서 업로드

1. 웹 앱의 왼쪽 탐색 영역에서 **SSL 설정** 을 선택 합니다.

2. **인증서 업로드**를 선택 합니다.

3. **Pfx 인증서 파일**에서 pfx 파일을 선택 합니다.

4. **인증서 암호**에 PFX 파일을 내보낼 때 만든 암호를 입력 합니다.

5. **업로드**를 선택합니다.

    ![SSL 인증서 업로드](media/solution-deployment-guide-geo-distributed/image38.png)

App Service에서 인증서 업로드가 완료 되 면 **SSL 설정** 페이지에 표시 됩니다.

![SSL 설정](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a>SSL 인증서 바인딩

1. **SSL 바인딩** 섹션에서 **바인딩 추가**를 선택 합니다.

    > [!Note]  
    >  인증서가 업로드 되었지만 **호스트** 이름 드롭다운에서 도메인 이름에 표시 되지 않는 경우 브라우저 페이지를 새로 고쳐 보세요.

2. **SSL 바인딩 추가** 페이지에서 드롭다운을 사용 하 여 보호할 도메인 이름과 사용할 인증서를 선택 합니다.

3. **SSL 유형**에서 [**SNI(서버 이름 표시)**](https://en.wikipedia.org/wiki/Server_Name_Indication) 또는 IP 기반 SSL을 사용할지 선택합니다.

    - **Sni 기반 ssl**: SNI 기반 ssl 바인딩을 여러 개 추가할 수 있습니다. 이 옵션을 사용하면 여러 SSL 인증서로 같은 IP 주소의 여러 도메인을 보호할 수 있습니다. 대부분의 최신 브라우저(Internet Explorer, Chrome, Firefox 및 Opera 포함)는 SNI를 지원합니다. [Server Name Indication](https://wikipedia.org/wiki/Server_Name_Indication)(서버 이름 표시)에서 더 포괄적인 브라우저 지원 정보를 찾을 수 있습니다.

    - **Ip 기반 ssl**: IP 기반 ssl 바인딩을 하나만 추가할 수 있습니다. 이 옵션을 사용하면 전용 공용 IP 주소를 보호하는 데 하나의 SSL 인증서만 사용할 수 있습니다. 여러 도메인을 보호 하려면 동일한 SSL 인증서를 사용 하 여 보안을 유지 합니다. IP 기반 SSL은 SSL 바인딩에 대 한 기존 옵션입니다.

4. **바인딩 추가**를 선택 합니다.

    ![SSL 바인딩 추가](media/solution-deployment-guide-geo-distributed/image40.png)

App Service에서 인증서 업로드가 완료 되 면 **SSL 바인딩** 섹션에 표시 됩니다.

![SSL 바인딩 업로드 완료](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a>IP SSL에 대 한 A 레코드 다시 매핑

웹 앱에서 IP 기반 SSL을 사용 하지 않는 경우 [사용자 지정 도메인에 대해 HTTPS 테스트](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl)로 건너뜁니다.

기본적으로 웹 앱은 공유 공용 IP 주소를 사용 합니다. 인증서가 IP 기반 SSL에 바인딩된 경우 App Service는 웹 앱에 대 한 새 전용 IP 주소를 만듭니다.

A 레코드가 웹 앱에 매핑되면 도메인 레지스트리는 전용 IP 주소를 사용 하 여 업데이트 해야 합니다.

**사용자 지정 도메인** 페이지는 새로운 전용 IP 주소로 업데이트 됩니다. 이 [ip 주소](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain)를 복사한 다음이 새 ip 주소에 [A 레코드](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain) 를 다시 매핑합니다.

#### <a name="test-https"></a>HTTPS 테스트

다른 브라우저에서로 이동 `https://<your.custom.domain>` 하 여 웹 앱이 제공 되는지 확인 합니다.

![웹 앱으로 이동](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> 인증서 유효성 검사 오류가 발생 하는 경우에는 자체 서명 된 인증서가 원인일 수 있으며, 그렇지 않은 경우에는 PFX 파일로 내보낼 때 중간 인증서가 꺼진 것일 수 있습니다.

#### <a name="enforce-https"></a>HTTPS 적용

기본적으로 모든 사용자는 HTTP를 사용 하 여 웹 앱에 액세스할 수 있습니다. HTTPS 포트에 대 한 모든 HTTP 요청이 리디렉션될 수 있습니다.

웹 앱 페이지에서 **SL 설정**을 선택 합니다. 그런 다음 **HTTPS에만 해당**에서 **켜기**를 선택합니다.

![HTTPS 적용](media/solution-deployment-guide-geo-distributed/image43.png)

작업이 완료 되 면 앱을 가리키는 HTTP Url로 이동 합니다. 예를 들면 다음과 같습니다.

- https://<app_name>. azurewebsites.net
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a>TLS 1.1/1.2 적용

앱은 기본적으로 [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0을 허용 하며,이는 더 이상 업계 표준 (예: [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard))으로 안전 하 게 고려 되지 않습니다. 더 높은 TLS 버전을 적용하려면 다음이 단계를 수행합니다.

1. 웹 앱 페이지의 왼쪽 탐색 영역에서 **SSL 설정**을 선택 합니다.

2. **Tls 버전**에서 최소 tls 버전을 선택 합니다.

    ![TLS 1.1 또는 1.2 적용](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a>Traffic Manager 프로필 만들기

1. **리소스 만들기**  >  **네트워킹**  >  **Traffic Manager 프로필**  >  **만들기**를 선택 합니다.

2. **Traffic Manager 프로필 만들기**에 다음과 같이 입력합니다.

    1. **이름**에 프로필의 이름을 입력 합니다. 이 이름은 traffic manager.net 영역 내에서 고유 해야 하며, Traffic Manager 프로필에 액세스 하는 데 사용 되는 DNS 이름 trafficmanager.net이 생성 됩니다.

    2. **라우팅 방법**에서 **지리적 라우팅 방법을**선택 합니다.

    3. **구독**에서이 프로필을 만들 구독을 선택 합니다.

    4. **리소스 그룹**에서 이 프로필을 배치할 새 리소스 그룹을 만듭니다.

    5. **리소스 그룹 위치**에서 리소스 그룹의 위치를 선택합니다. 이 설정은 리소스 그룹의 위치를 나타내며 전역적으로 배포 된 Traffic Manager 프로필에는 영향을 주지 않습니다.

    6. **만들기**를 선택합니다.

    7. Traffic Manager 프로필의 전역 배포가 완료 되 면 해당 리소스 그룹에 리소스 중 하나로 나열 됩니다.

        ![Traffic Manager 프로필 만들기의 리소스 그룹](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a>Traffic Manager 엔드포인트 추가

1. 포털 검색 창에서 이전 섹션에서 만든 **Traffic Manager 프로필** 이름을 검색 하 고 표시 된 결과에서 Traffic Manager 프로필을 선택 합니다.

2. **Traffic Manager 프로필**의 **설정** 섹션에서 **끝점**을 선택 합니다.

3. **추가**를 선택합니다.

4. Azure Stack 허브 끝점을 추가 합니다.

5. **형식**에서 **외부 끝점**을 선택 합니다.

6. 이 끝점의 **이름을** 제공 합니다. 이상적으로 Azure Stack 허브의 이름입니다.

7. **FQDN**(정규화 된 도메인 이름)의 경우 Azure Stack 허브 웹 앱에 대 한 외부 URL을 사용 합니다.

8. 지리적 매핑에서 리소스가 있는 지역/대륙을 선택 합니다. 예: **유럽.**

9. 표시 되는 국가/지역 드롭다운에서이 끝점에 적용 되는 국가를 선택 합니다. 예: **독일**.

10. **사용 안 함으로 추가**를 선택 취소 상태로 유지합니다.

11. **확인**을 선택합니다.

12. Azure 엔드포인트 추가:

    1. **유형**에 대해 **Azure 엔드포인트**를 선택 합니다.

    2. 끝점의 **이름을** 제공 합니다.

    3. **대상 리소스 종류**에 대해 **App Service**를 선택 합니다.

    4. **대상 리소스**에 대해 **앱 서비스 선택** 을 선택 하 여 동일한 구독에서 Web Apps 목록을 표시 합니다. **리소스**에서 첫 번째 끝점으로 사용 되는 App service를 선택 합니다.

13. 지리적 매핑에서 리소스가 있는 지역/대륙을 선택 합니다. 예를 들어 **북아메리카/중앙 아메리카/카리브 합니다.**

14. 표시 되는 국가/지역 드롭다운에서이 상자를 비워 두고 위의 모든 지역 그룹화를 선택 합니다.

15. **사용 안 함으로 추가**를 선택 취소 상태로 유지합니다.

16. **확인**을 선택합니다.

    > [!Note]  
    >  리소스의 기본 끝점으로 사용할 지리적 범위가 모든 (세계) 인 끝점을 하나 이상 만듭니다.

17. 두 끝점의 추가가 완료 되 면 해당 모니터링 상태와 함께 **Traffic Manager 프로필** 에 **온라인**으로 표시 됩니다.

    ![Traffic Manager 프로필 끝점 상태](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a>글로벌 기업은 Azure 지역 배포 기능에 의존 합니다.

Azure Traffic Manager 및 지리 관련 끝점을 통해 데이터 트래픽을 전송 하면 글로벌 기업은 지역 규정을 준수 하 고 데이터를 준수 하 고 안전 하 게 유지할 수 있습니다 .이는 로컬 및 원격 비즈니스 위치의 성공에 매우 중요 합니다.

## <a name="next-steps"></a>다음 단계

- Azure Cloud 패턴에 대해 자세히 알아보려면 [클라우드 디자인 패턴](https://docs.microsoft.com/azure/architecture/patterns)을 참조 하세요.
