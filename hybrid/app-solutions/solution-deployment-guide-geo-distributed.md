---
title: 지리적으로 분산된 앱에서 Azure 및 Azure Stack Hub를 사용하여 트래픽 전달
description: 지리적으로 분산된 앱 솔루션에서 Azure 및 Azure Stack Hub를 사용하여 트래픽을 특정 엔드포인트에 전달하는 방법을 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 9fa2c351d2c13d85fe1adb17a35e165de96ea2a2
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895434"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a>지리적으로 분산된 앱에서 Azure 및 Azure Stack Hub를 사용하여 트래픽 전달

지리적으로 분산된 앱 패턴을 사용하여 다양한 메트릭에 따라 특정 엔드포인트로 트래픽을 전달하는 방법을 알아봅니다. 지리 기반 라우팅 및 엔드포인트 구성을 사용하여 Traffic Manager 프로필을 만들면 지역별 요구 사항, 회사 및 국제 규정, 데이터 요구 사항에 따라 정보가 엔드포인트로 라우팅됩니다.

이 솔루션에서는 다음을 수행하는 샘플 환경을 빌드합니다.

> [!div class="checklist"]
> - 지리적으로 분산된 앱을 만듭니다.
> - Traffic Manager를 사용하여 앱을 대상으로 지정합니다.

## <a name="use-the-geo-distributed-apps-pattern"></a>지리적으로 분산된 앱 패턴 사용

지리적으로 분산된 패턴을 사용하면 앱이 여러 지역에 걸쳐 있습니다. 퍼블릭 클라우드를 기본 클라우드로 사용할 수 있지만, 일부 사용자는 현재 있는 지역에 데이터를 유지해야 할 수도 있습니다. 사용자의 요구 사항에 따라 사용자를 가장 적합한 클라우드로 안내할 수 있습니다.

### <a name="issues-and-considerations"></a>문제 및 고려 사항

#### <a name="scalability-considerations"></a>확장성 고려 사항

이 문서에서 빌드할 솔루션은 확장성을 고려하지 않습니다. 그러나 다른 Azure 및 온-프레미스 솔루션과 함께 사용하는 경우 확장성 요구 사항을 수용할 수 있습니다. Traffic Manager를 통해 자동 스케일링되는 하이브리드 솔루션을 만드는 방법에 대한 자세한 내용은 [Azure를 사용하여 클라우드 간에 스케일링되는 솔루션 만들기](solution-deployment-guide-cross-cloud-scaling.md)를 참조하세요.

#### <a name="availability-considerations"></a>가용성 고려 사항

확장성을 고려하는 경우에도 이 솔루션은 가용성을 직접 해결하지 않습니다. 그러나 이 솔루션 내에서 Azure 및 온-프레미스 솔루션을 구현하여 관련된 모든 구성 요소의 고가용성을 확보할 수 있습니다.

### <a name="when-to-use-this-pattern"></a>이 패턴을 사용해야 하는 경우

- 조직의 해외 지사는 맞춤형 현지 보안 및 배포 정책이 필요합니다.

- 조직의 각 사무소는 직원, 비즈니스 및 시설 데이터를 수집하며, 현지 규정 및 표준 시간대에 따라 보고 활동을 수행해야 합니다.

- 매우 높은 부하 요구 사항을 처리할 수 있도록 단일 지역과 여러 지역에 여러 앱을 배포하는 수평적 앱 스케일 아웃을 통해 대규모 요구 사항을 충족합니다.

### <a name="planning-the-topology"></a>토폴로지 계획

분산된 앱 공간을 빌드하기 전에 다음 사항을 파악하는 것이 좋습니다.

- **앱의 사용자 지정 도메인:** 고객이 앱에 액세스하는 데 사용할 사용자 지정 도메인 이름은 무엇인가요? 샘플 앱의 경우 사용자 지정 도메인 이름은 *www\.scalableasedemo.com* 입니다.

- **트래픽 관리자 도메인:** 도메인 이름은 [Azure Traffic Manager 프로필](/azure/traffic-manager/traffic-manager-manage-profiles)을 만들 때 선택합니다. 이 이름은 *trafficmanager.net* 접미사와 결합되어 Traffic Manager에서 관리되는 도메인 항목을 등록합니다. 샘플 앱의 경우 선택한 이름은 *scalable-ase-demo* 입니다. 결과적으로 Traffic Manager에서 관리되는 전체 도메인 이름은 *scalable-ase-demo.trafficmanager.net* 입니다.

- **앱 공간을 크기 조정하는 전략:** 애플리케이션 공간을 단일 지역, 여러 지역 또는 두 가지를 혼합한 App Service 환경 중 어디에 분산할 것인지 결정합니다. 고객 트래픽이 생성될 것으로 예상되는 위치와 백 엔드 인프라를 지원하는 앱의 나머지 부분이 스케일링되어야 하는 범위를 감안하여 결정해야 합니다. 예를 들어 100% 상태 비저장 앱의 경우 Azure 지역마다 여러 App Service 환경을 조합하여 앱을 대규모로 스케일링할 수 있으며, 여러 Azure 지역에 배포된 App Service 환경을 통해 더욱 크게 스케일링할 수 있습니다. 고객은 15개 이상의 글로벌 Azure 지역 중에 선택하여 진정한 글로벌 하이퍼스케일 앱 공간을 구축할 수 있습니다. 여기서 사용되는 샘플 앱의 경우 세 가지 App Service 환경을 단일 Azure 지역(미국 중남부)에 만들었습니다.

- **App Service 환경의 명명 규칙:** App Service 환경마다 고유한 이름이 필요합니다. 하나 또는 두 개의 App Service 환경 외에도, 각 App Service 환경을 식별하는 데 도움이 되는 명명 규칙이 있는 것이 좋습니다. 여기서 사용하는 샘플 앱의 경우 간단한 명명 규칙을 사용했습니다. 세 가지 App Service 환경 이름은 *fe1ase*, *fe2ase* 및 *fe3ase* 입니다.

- **앱에 대한 명명 규칙:** 앱의 여러 인스턴스가 배포되기 때문에 배포된 앱의 인스턴스마다 이름이 필요합니다. Power Apps용 App Service Environment에서는 동일한 앱 이름을 여러 환경에 사용할 수 있습니다. App Service 환경마다 고유한 도메인 접미사가 있으므로 개발자는 각 환경에서 정확히 동일한 앱 이름을 다시 사용하도록 선택할 수 있습니다. 예를 들어 개발자가 앱 이름을 *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net* 등으로 지정할 수 있습니다. 여기에서 사용되는 앱의 경우 각 앱 인스턴스에 고유한 이름이 있습니다. 앱 인스턴스에 사용되는 이름은 *webfrontend1*, *webfrontend2* 및 *webfrontend3* 입니다.

> [!Tip]  
> ![하이브리드 핵심 요소 다이어그램](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub는 Azure의 확장입니다. Azure Stack Hub는 온-프레미스 환경에 클라우드 컴퓨팅의 민첩성과 혁신을 제공하여 어디서나 하이브리드 앱을 빌드하고 배포할 수 있는 유일한 하이브리드 클라우드를 사용하도록 설정합니다.  
> 
> [하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 문서는 하이브리드 앱 디자인, 배포 및 운영에 대한 소프트웨어 품질(배치, 확장성, 가용성, 복원력, 관리 효율성 및 보안)의 핵심 요소를 검토합니다. 디자인 고려 사항은 하이브리드 앱 디자인을 최적화하고 프로덕션 환경에서 문제를 최소화하는 데 도움이 됩니다.

## <a name="part-1-create-a-geo-distributed-app"></a>1부: 지리적으로 분산된 앱 만들기

여기서는 웹앱을 만듭니다.

> [!div class="checklist"]
> - 웹앱을 만들고 게시합니다.
> - Azure Repos에 코드를 추가합니다.
> - 앱 빌드의 대상으로 여러 클라우드를 지정합니다.
> - CD 프로세스를 관리하고 구성합니다.

### <a name="prerequisites"></a>필수 구성 요소

Azure 구독 및 Azure Stack Hub가 설치되어 있어야 합니다.

### <a name="geo-distributed-app-steps"></a>지리적으로 분산된 앱 단계

### <a name="obtain-a-custom-domain-and-configure-dns"></a>사용자 지정 도메인 획득 및 DNS 구성

도메인에 대한 DNS 영역 파일을 업데이트합니다. 그런 다음, Azure AD가 사용자 지정 도메인 이름의 소유권을 확인할 수 있습니다. Azure 내에서 Azure/Microsoft 365/외부 DNS 레코드에 대해 [Azure DNS](/azure/dns/dns-getstarted-portal)를 사용하거나 [서로 다른 DNS 등록 기관](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider)에서 DNS 항목을 추가합니다.

1. 사용자 지정 도메인을 퍼블릭 등록자에 등록합니다.

2. 도메인에 대한 도메인 이름 등록 기관에 로그인합니다. 승인된 관리자가 DNS를 업데이트해야 할 수 있습니다.

3. Azure AD에서 제공하는 DNS 항목을 추가하여 도메인에 대한 DNS 영역 파일을 업데이트합니다. DNS 항목은 메일 라우팅이나 웹 호스팅 같은 동작을 변경하지 않습니다.

### <a name="create-web-apps-and-publish"></a>웹앱 만들기 및 게시

웹앱을 Azure 및 Azure Stack Hub에 배포하고 두 클라우드의 변경 내용을 자동으로 푸시하는 하이브리드 CI/CD(연속 통합 및 지속적인 업데이트)를 설정합니다.

> [!Note]  
> 실행을 위한 적절한 이미지(Windows Server 및 SQL)가 신디케이트된 Azure Stack Hub 및 App Service를 배포해야 합니다. 자세한 내용은 [Azure Stack Hub에 App Service를 배포하기 위한 필수 조건](/azure-stack/operator/azure-stack-app-service-before-you-get-started)을 참조하세요.

#### <a name="add-code-to-azure-repos"></a>Azure Repos에 코드 추가

1. Azure Repos에 프로젝트를 만드는 **권한** 이 있는 계정으로 Visual Studio에 로그인합니다.

    CI/CD는 앱 코드와 인프라 코드 모두에 적용할 수 있습니다. 프라이빗 클라우드와 호스트된 클라우드 개발 모두에 [Azure Resource Manager 템플릿](https://azure.microsoft.com/resources/templates/)을 사용합니다.

    ![Visual Studio에서 프로젝트에 연결](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. 기본 웹앱을 만들고 열어 **리포지토리를 복제** 합니다.

    ![Visual Studio에서 리포지토리 복제](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a>두 클라우드에서 웹앱 배포 만들기

1. 다음과 같이 **WebApplication.csproj** 파일을 편집합니다. `Runtimeidentifier`를 선택하고 `win10-x64`를 추가합니다. ([자체 포함 배포](/dotnet/core/deploying/deploy-with-vs#simpleSelf) 설명서를 참조하세요.)

    ![Visual Studio에서 웹앱 프로젝트 파일 편집](media/solution-deployment-guide-geo-distributed/image3.png)

2. 팀 탐색기를 사용하여 **코드를 Azure Repos에 체크 인** 합니다.

3. **애플리케이션 코드** 가 Azure Repos에 체크 인되었는지 확인합니다.

### <a name="create-the-build-definition"></a>빌드 정의 만들기

1. **Azure Pipelines에 로그인** 하여 빌드 정의를 만드는 기능을 확인합니다.

2. `-r win10-x64` 코드를 추가합니다. .NET Core를 사용하여 자체 포함 배포를 트리거하려면 이 코드를 추가해야 합니다.

    ![Azure Pipelines의 빌드 정의에 코드를 추가합니다.](media/solution-deployment-guide-geo-distributed/image4.png)

3. **빌드를 실행** 합니다. [자체 포함 배포 빌드](/dotnet/core/deploying/deploy-with-vs#simpleSelf) 프로세스는 Azure 및 Azure Stack Hub에서 실행 가능한 아티팩트를 게시합니다.

#### <a name="using-an-azure-hosted-agent"></a>Azure 호스트된 에이전트 사용

Azure Pipelines에서 호스트된 에이전트를 사용하면 간편하게 웹앱을 빌드하고 배포할 수 있습니다. Microsoft Azure에서 유지 관리 및 업그레이드를 자동으로 수행하므로 중단 없는 개발, 테스트 및 배포가 가능합니다.

### <a name="manage-and-configure-the-cd-process"></a>CD 프로세스 관리 및 구성

Azure DevOps Services는 개발, 준비, QA, 프로덕션 환경 등의 여러 환경에 릴리스할 수 있도록 매우 유연하게 구성하고 쉽게 관리할 수 있는 파이프라인을 제공하며, 특정 단계에서 승인을 요구합니다.

## <a name="create-release-definition"></a>릴리스 정의 만들기

1. Azure DevOps Services의 **빌드 및 릴리스** 섹션에 있는 **릴리스** 탭 아래에서 **+** 단추를 선택하여 새 릴리스를 추가합니다.

    ![Azure DevOps Services에서 릴리스 정의 만들기](media/solution-deployment-guide-geo-distributed/image5.png)

2. Azure App Service 배포 템플릿을 적용합니다.

   ![Azure DevOps Services에서 Azure App Service 배포 템플릿 적용](media/solution-deployment-guide-geo-distributed/image6.png)

3. **아티팩트 추가** 에서 Azure 클라우드 빌드 앱에 대한 아티팩트를 추가합니다.

   ![Azure DevOps Services에서 Azure 클라우드 빌드에 아티팩트 추가](media/solution-deployment-guide-geo-distributed/image7.png)

4. [파이프라인] 탭에서 환경의 **단계, 작업** 링크를 선택하고 Azure 클라우드 환경 값을 설정합니다.

   ![Azure DevOps Services에서 Azure 클라우드 환경 값 설정](media/solution-deployment-guide-geo-distributed/image8.png)

5. **환경 이름** 을 설정하고 Azure 클라우드 엔드포인트에 대한 **Azure 구독** 을 선택합니다.

      ![Azure DevOps Services에서 Azure 클라우드 엔드포인트에 대한 Azure 구독 선택](media/solution-deployment-guide-geo-distributed/image9.png)

6. **앱 서비스 이름** 에서 필요한 Azure 앱 서비스 이름을 설정합니다.

      ![Azure DevOps Services에서 Azure 앱 서비스 이름 설정](media/solution-deployment-guide-geo-distributed/image10.png)

7. **에이전트 큐** 에서 Azure 클라우드 호스팅 환경으로 "호스트된 VS2017"을 입력합니다.

      ![Azure DevOps Services에서 Azure 클라우드 호스팅 환경의 에이전트 큐 설정](media/solution-deployment-guide-geo-distributed/image11.png)

8. [Azure App Service 배포] 메뉴에서 해당 환경의 유효한 **패키지 또는 폴더** 를 선택합니다. **폴더 위치** 에 대해 **확인** 을 선택합니다.
  
      ![Azure DevOps Services에서 Azure App Service 환경에 사용할 패키지 또는 폴더 선택](media/solution-deployment-guide-geo-distributed/image12.png)

      ![폴더 선택기 대화 상자 1](media/solution-deployment-guide-geo-distributed/image13.png)

9. 모든 변경 내용을 저장하고 **릴리스 파이프라인** 으로 돌아갑니다.

    ![Azure DevOps Services에서 릴리스 파이프라인의 변경 내용 저장](media/solution-deployment-guide-geo-distributed/image14.png)

10. 새 아티팩트를 추가하고 Azure Stack Hub 앱에 대한 빌드를 선택합니다.

    ![Azure DevOps Services에서 Azure Stack Hub 앱에 대한 새 아티팩트 추가](media/solution-deployment-guide-geo-distributed/image15.png)


11. Azure App Service 배포를 적용하여 환경을 하나 더 추가합니다.

    ![Azure DevOps Services에서 Azure App Service 배포에 환경 추가](media/solution-deployment-guide-geo-distributed/image16.png)

12. 새 환경 Azure Stack Hub의 이름을 지정합니다.

    ![Azure DevOps Services에서 Azure App Service 배포의 환경 이름 지정](media/solution-deployment-guide-geo-distributed/image17.png)

13. **작업** 탭에서 Azure Stack Hub 환경을 찾습니다.

    ![Azure DevOps Services에 포함된 Azure DevOps Services의 Azure Stack Hub 환경](media/solution-deployment-guide-geo-distributed/image18.png)

14. Azure Stack Hub 엔드포인트의 구독을 선택합니다.

    ![Azure DevOps Services에서 Azure Stack Hub 엔드포인트의 구독 선택](media/solution-deployment-guide-geo-distributed/image19.png)

15. 앱 서비스 이름으로 Azure Stack Hub 웹앱 이름을 설정합니다.

    ![Azure DevOps Services에서 Azure Stack Hub 웹앱 이름 설정](media/solution-deployment-guide-geo-distributed/image20.png)

16. Azure Stack Hub 에이전트를 선택합니다.

    ![Azure DevOps Services에서 Azure Stack Hub 에이전트 선택](media/solution-deployment-guide-geo-distributed/image21.png)

17. [Azure App Service 배포] 섹션에서 해당 환경의 유효한 **패키지 또는 폴더** 를 선택합니다. 폴더 위치에 대해 **확인** 을 선택합니다.

    ![Azure DevOps Services에서 Azure App Service 배포에 사용할 폴더 선택](media/solution-deployment-guide-geo-distributed/image22.png)

    ![폴더 선택기 대화 상자 2](media/solution-deployment-guide-geo-distributed/image23.png)

18. [변수] 탭에서 `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`라는 변수를 추가하고 해당 값을 **true**, 범위를 Azure Stack Hub로 설정합니다.

    ![Azure DevOps Services에서 Azure 앱 배포에 변수 추가](media/solution-deployment-guide-geo-distributed/image24.png)

19. 두 아티팩트 모두에서 **지속적인** 배포 트리거 아이콘을 선택하고 **지속적인** 배포 트리거를 사용하도록 설정합니다.

    ![Azure DevOps Services에서 지속적인 배포 트리거 선택](media/solution-deployment-guide-geo-distributed/image25.png)

20. Azure Stack Hub 환경에서 **배포 전** 조건 아이콘을 선택하고 트리거를 **릴리스 후** 로 설정합니다.

    ![Azure DevOps Services에서 배포 전 조건 선택](media/solution-deployment-guide-geo-distributed/image26.png)

21. 모든 변경 내용을 저장합니다.

> [!Note]  
> 템플릿에서 릴리스 정의를 만들 때 작업에 대한 일부 설정이 자동으로 [환경 변수](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables)로 정의되었을 수 있습니다. 이러한 설정은 작업 설정에서 수정할 수 없습니다. 이러한 설정을 편집하려면 부모 환경 항목을 선택해야 합니다.

## <a name="part-2-update-web-app-options"></a>2부: 웹앱 옵션 업데이트

[Azure App Service](/azure/app-service/overview)는 확장성 높은 자체 패치 웹 호스팅 서비스를 제공합니다.

![Azure App Service](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - Azure Web Apps에 기존 사용자 지정 DNS 이름 매핑
> - **CNAME 레코드** 및 **A 레코드** 를 사용하여 사용자 지정 DNS 이름을 App Service에 매핑합니다.

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a>Azure Web Apps에 기존 사용자 지정 DNS 이름 매핑

> [!Note]  
> 루트 도메인(예: northwind.com)을 제외한 모든 사용자 지정 DNS 이름에 CNAME을 사용합니다.

라이브 사이트 및 해당 DNS 도메인 이름을 App Service로 마이그레이션하려면 [활성 DNS 이름을 Azure App Service로 마이그레이션](/azure/app-service/manage-custom-dns-migrate-domain)을 참조하세요.

### <a name="prerequisites"></a>필수 구성 요소

이 시나리오를 완료하려면 다음이 필요합니다.

- [App Service 앱을 만들거나](/azure/app-service/) 다른 솔루션에서 만든 앱을 사용합니다.

- 도메인 이름을 구매하고 도메인 공급자의 DNS 레지스트리에 대한 액세스 권한을 확보합니다.

도메인에 대한 DNS 영역 파일을 업데이트합니다. Azure AD는 사용자 지정 도메인 이름의 소유권을 확인합니다. Azure 내에서 Azure/Microsoft 365/외부 DNS 레코드에 대해 [Azure DNS](/azure/dns/dns-getstarted-portal)를 사용하거나 [서로 다른 DNS 등록 기관](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider)에서 DNS 항목을 추가합니다.

- 사용자 지정 도메인을 퍼블릭 등록자에 등록합니다.

- 도메인에 대한 도메인 이름 등록 기관에 로그인합니다. (승인된 관리자가 DNS를 업데이트해야 할 수 있습니다.)

- Azure AD에서 제공하는 DNS 항목을 추가하여 도메인에 대한 DNS 영역 파일을 업데이트합니다.

예를 들어 northwindcloud.com 및 www\.northwindcloud.com에 대한 DNS 항목을 추가하려면 northwindcloud.com 루트 도메인에 대한 DNS 설정을 구성합니다.

> [!Note]  
> 도메인 이름은 [Azure Portal](/azure/app-service/manage-custom-dns-buy-domain)을 사용하여 구매할 수 있습니다. 사용자 지정 DNS 이름을 웹앱에 매핑하려면 [App Service 계획](https://azure.microsoft.com/pricing/details/app-service/)이 유료 계층(**공유**, **기본**, **표준** 또는 **프리미엄**)이어야 합니다.

### <a name="create-and-map-cname-and-a-records"></a>CNAME 및 A 레코드 만들기 및 매핑

#### <a name="access-dns-records-with-domain-provider"></a>도메인 공급자로 DNS 레코드 액세스

> [!Note]  
>  Azure DNS를 사용하여 Azure Web Apps에 대한 사용자 지정 DNS 이름을 구성합니다. 자세한 내용은 [Azure DNS를 사용하여 Azure 서비스에 대해 사용자 지정 도메인 설정 제공](/azure/dns/dns-custom-domain)을 참조하세요.

1. 기본 공급자의 웹 사이트에 로그인합니다.

2. DNS 레코드를 관리하기 위한 페이지를 찾습니다. 모든 도메인 공급자는 자체 DNS 레코드 인터페이스를 갖고 있습니다. **도메인 이름**, **DNS** 또는 **이름 서버 관리** 라는 레이블이 지정된 사이트 영역을 찾습니다.

DNS 레코드 페이지는 **내 도메인** 에서 볼 수 있습니다. **영역 파일**, **DNS 레코드** 또는 **고급 구성** 이라는 링크를 찾습니다.

다음 스크린샷은 DNS 레코드 페이지의 예입니다.

![DNS 레코드 페이지 예](media/solution-deployment-guide-geo-distributed/image28.png)

1. [도메인 이름 등록자]에서 **추가 또는 만들기** 를 선택하여 레코드를 만듭니다. 일부 공급자에는 다른 레코드 형식을 추가하는 다양한 링크가 있습니다. 공급자의 설명서를 참조하세요.

2. CNAME 레코드를 추가하여 하위 도메인을 앱의 기본 호스트 이름에 매핑합니다.

   www\.northwindcloud.com 도메인 예제의 경우 이름을 `<app_name>.azurewebsites.net`에 매핑하는 CNAME 레코드를 추가합니다.

CNAME을 추가하면 DNS 레코드 페이지가 다음 예제와 비슷합니다.

![Azure 앱에 대한 포털 탐색](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a>Azure에서 CNAME 레코드 매핑 사용

1. 새 탭에서 Azure Portal에 로그인합니다.

2. App Services로 이동합니다.

3. 웹앱을 선택합니다.

4. Azure Portal의 앱 페이지 왼쪽 탐색 영역에서 **사용자 지정 도메인** 을 선택합니다.

5. **호스트 이름 추가** 옆에 있는 **+** 아이콘을 선택합니다.

6. 정규화된 도메인 이름(예: `www.northwindcloud.com`)을 입력합니다.

7. **유효성 검사** 를 선택합니다.

8. 레코드 추가 메시지가 표시되면 다른 형식의 추가 레코드(`A` 또는 `TXT`)를 도메인 이름 등록자 DNS 레코드에 추가합니다. Azure는 다음과 같은 레코드의 값과 유형을 제공합니다.

   a.  앱의 IP 주소에 매핑할 **A** 레코드

   b.  앱의 기본 호스트 이름(`<app_name>.azurewebsites.net`)에 매핑할 **TXT** 레코드 - App Service는 구성할 때만 이 레코드를 사용하여 사용자 지정 도메인 소유권을 확인합니다. 확인 후 TXT 레코드를 삭제합니다.

9. 도메인 등록자 탭에서 이 작업을 완료하고 **호스트 이름 추가** 단추가 활성화될 때까지 유효성을 다시 검사합니다.

10. **호스트 이름 레코드 형식** 이 **CNAME**(www.example.com 또는 하위 도메인)으로 설정되어 있는지 확인합니다.

11. **호스트 이름 추가** 를 선택합니다.

12. 정규화된 도메인 이름(예: `northwindcloud.com`)을 입력합니다.

13. **유효성 검사** 를 선택합니다. **추가** 단추가 활성화됩니다.

14. **호스트 이름 레코드 형식** 이 **A 레코드(example.com)** 로 설정되어 있는지 확인합니다.

15. **호스트 이름을 추가합니다**.

    새 호스트 이름이 앱의 **사용자 지정 도메인** 페이지에 반영될 때까지 시간이 약간 걸릴 수 있습니다. 데이터를 업데이트하려면 브라우저를 새로 고쳐 보세요.
  
    ![사용자 지정 도메인](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    오류가 발생하면 페이지 아래쪽에 확인 오류 알림이 표시됩니다. ![도메인 확인 오류](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  위의 단계를 반복하여 와일드카드 도메인(\*.northwindcloud.com)을 매핑할 수 있습니다. 이렇게 하면 항목마다 별도의 CNAME 레코드를 만들지 않고도 이 앱 서비스에 하위 도메인을 추가할 수 있습니다. 등록자 지침에 따라 이 설정을 구성합니다.

#### <a name="test-in-a-browser"></a>브라우저에서 테스트

이전에 구성한 DNS 이름(`northwindcloud.com` 또는 `www.northwindcloud.com`)을 찾습니다.

## <a name="part-3-bind-a-custom-ssl-cert"></a>3부: 사용자 지정 SSL 인증서 바인딩

여기서는 다음 작업을 수행합니다.

> [!div class="checklist"]
> - App Service에 사용자 지정 SSL 인증서를 바인딩합니다.
> - 앱에 HTTPS를 적용합니다.
> - 스크립트를 사용하여 SSL 인증서 바인딩을 자동화합니다.

> [!Note]  
> 필요한 경우 Azure Portal에서 고객 SSL 인증서를 가져와 웹앱에 바인딩합니다. 자세한 내용은 [App Service 인증서 자습서](/azure/app-service/web-sites-purchase-ssl-web-site)를 참조하세요.

### <a name="prerequisites"></a>필수 구성 요소

이 솔루션을 완료하려면 다음이 필요합니다.

- [App Service 앱을 만듭니다](/azure/app-service/).
- [웹앱에 사용자 지정 DNS 이름을 매핑합니다.](/azure/app-service/app-service-web-tutorial-custom-domain)
- 신뢰할 수 있는 인증 기관에서 SSL 인증서를 획득하고 키를 사용하여 요청에 서명합니다.

### <a name="requirements-for-your-ssl-certificate"></a>SSL 인증서에 대한 요구 사항

App Service에서 인증서를 사용하려면 인증서가 다음 요구 사항을 모두 충족해야 합니다.

- 신뢰할 수 있는 인증 기관에서 서명됨

- 암호로 보호된 PFX 파일로 내보냄

- 길이가 2048비트 이상인 프라이빗 키 포함

- 인증서 체인의 모든 중간 인증서를 포함함

> [!Note]  
> **ECC(타원 곡선 암호화) 인증서** 는 App Service에서 작동하지만 이 가이드에서는 다루지 않습니다. ECC 인증서 만들기와 관련하여 도움이 필요하면 인증 기관에 문의하세요.

#### <a name="prepare-the-web-app"></a>웹앱 준비

사용자 지정 SSL 인증서를 웹앱에 바인딩하려면 [App Service 플랜](https://azure.microsoft.com/pricing/details/app-service/)이 **기본**, **표준** 또는 **프리미엄** 계층에 있어야 합니다.

#### <a name="sign-in-to-azure"></a>Azure에 로그인

1. [Azure Portal](https://portal.azure.com/)을 열고 웹앱으로 이동합니다.

2. 왼쪽 메뉴에서 **App Services** 를 선택한 다음, 웹앱 이름을 선택합니다.

![Azure Portal에서 웹앱 선택](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a>가격 책정 계층 확인

1. 웹앱 페이지의 왼쪽 탐색 영역에서 **설정** 섹션으로 스크롤하고 **강화(App Service 계획)** 를 선택합니다.

    ![웹앱의 스케일 업 메뉴](media/solution-deployment-guide-geo-distributed/image34.png)

1. 웹앱이 **체험** 또는 **공유** 계층에 있지 않은지 확인합니다. 웹앱의 현재 계층은 진한 파란색 상자로 강조 표시됩니다.

    ![웹앱에서 가격 책정 계층 확인](media/solution-deployment-guide-geo-distributed/image35.png)

사용자 지정 SSL은 **무료** 또는 **공유** 계층에서 지원되지 않습니다. 업스케일하려면 다음 섹션 또는 **가격 책정 계층 선택** 페이지의 단계에 따라 [SSL 인증서 업로드 및 바인딩](/azure/app-service/app-service-web-tutorial-custom-ssl)으로 건너뜁니다.

#### <a name="scale-up-your-app-service-plan"></a>App Service 계획 강화

1. **기본**, **표준** 또는 **프리미엄** 계층 중 하나를 선택합니다.

2. **선택** 을 선택합니다.

![웹앱의 가격 책정 계층 선택](media/solution-deployment-guide-geo-distributed/image36.png)

알림이 표시되면 스케일링 작업이 완료된 것입니다.

![강화 알림](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a>SSL 인증서 바인딩 및 중간 인증서 병합

체인의 여러 인증서를 병합합니다.

1. 받은 **각 인증서를 텍스트 편집기에서 엽니다**.

2. 병합된 인증서에 대한 *mergedcertificate.crt* 라는 파일을 만듭니다. 텍스트 편집기에서 각 인증서의 내용을 이 파일에 복사합니다. 사용자 인증서의 순서는 사용자의 인증서로 시작하고 루트 인증서로 끝나는 인증서 체인의 순서에 따라야 합니다. 다음 예제와 유사합니다.

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

인증서에서 생성한 프라이빗 키를 사용하여 병합된 SSL 인증서를 내보냅니다.

프라이빗 키 파일은 OpenSSL을 통해 생성됩니다. 인증서를 PFX로 내보내려면 다음 명령을 실행하고 `<private-key-file>` 및 `<merged-certificate-file>` 자리 표시자를 각각 프라이빗 키 경로와 병합된 인증서 파일로 바꿉니다.

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

메시지가 표시되면 나중에 SSL 인증서를 App Service에 업로드하기 위한 내보내기 암호를 정의합니다.

IIS 또는 **Certreq.exe** 파일은 인증서 요청을 생성하고 로컬 머신에 인증서를 설치한 다음, [인증서를 PFX로 내보내는](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)) 데 사용됩니다.

#### <a name="upload-the-ssl-certificate"></a>SSL 인증서 업로드

1. 웹앱의 왼쪽 탐색 메뉴에서 **SSL 설정** 을 선택합니다.

2. **인증서 업로드** 를 선택합니다.

3. **PFX 인증서 파일** 에서 PFX 파일을 선택합니다.

4. **인증서 암호** 에서 PFX 파일을 내보낼 때 만든 암호를 입력합니다.

5. **업로드** 를 선택합니다.

    ![SSL 인증서 업로드](media/solution-deployment-guide-geo-distributed/image38.png)

App Service에서 인증서 업로드가 완료되면 **SSL 설정** 페이지에 업로드된 인증서가 표시됩니다.

![SSL 설정](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a>SSL 인증서 바인딩

1. **SSL 바인딩** 섹션에서 **바인딩 추가** 를 선택합니다.

    > [!Note]  
    >  인증서를 업로드했지만 **호스트 이름** 드롭다운의 도메인 이름에 표시되지 않으면 브라우저 페이지를 새로 고칩니다.

2. **SSL 바인딩 추가** 페이지에서 드롭다운을 사용하여 보호할 도메인 이름과 사용할 인증서를 선택합니다.

3. **SSL 유형** 에서 [**SNI(서버 이름 표시)**](https://en.wikipedia.org/wiki/Server_Name_Indication) 또는 IP 기반 SSL을 사용할지 선택합니다.

    - **SNI 기반 SSL**: 여러 개의 SNI 기반 SSL 바인딩을 추가할 수 있습니다. 이 옵션을 사용하면 여러 SSL 인증서로 같은 IP 주소의 여러 도메인을 보호할 수 있습니다. 대부분의 최신 브라우저(Internet Explorer, Chrome, Firefox 및 Opera 포함)는 SNI를 지원합니다. [Server Name Indication](https://wikipedia.org/wiki/Server_Name_Indication)(서버 이름 표시)에서 더 포괄적인 브라우저 지원 정보를 찾을 수 있습니다.

    - **IP 기반 SSL**: IP 기반 SSL 바인딩은 하나만 추가할 수 있습니다. 이 옵션을 사용하면 전용 공용 IP 주소를 보호하는 데 하나의 SSL 인증서만 사용할 수 있습니다. 여러 도메인을 보호하려면 동일한 SSL 인증서를 사용하여 모두 보호합니다. IP 기반 SSL은 전통적인 SSL 바인딩 옵션입니다.

4. **바인딩 추가** 를 선택합니다.

    ![SSL 바인딩 추가](media/solution-deployment-guide-geo-distributed/image40.png)

App Service에서 인증서 업로드를 완료하면 **SSL 바인딩** 섹션에 업로드된 인증서가 표시됩니다.

![SSL 바인딩 업로드 완료](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a>IP SSL에 대한 A 레코드 다시 매핑

웹앱에서 IP 기반 SSL을 사용하지 않을 경우 [사용자 지정 도메인에 대한 HTTPS 테스트](/azure/app-service/app-service-web-tutorial-custom-ssl)로 건너뜁니다.

기본적으로 웹앱에서는 공유 공용 IP 주소를 사용합니다. IP 기반 SSL을 사용하여 인증서를 바인딩하면 App Service에서 웹앱에 대한 새로운 전용 IP 주소를 만듭니다.

A 레코드가 웹앱에 매핑되면 도메인 레지스트리를 전용 IP 주소로 업데이트해야 합니다.

**사용자 지정 도메인** 페이지가 새로운 전용 IP 주소로 업데이트됩니다. [이 IP 주소를 복사](/azure/app-service/app-service-web-tutorial-custom-domain)하고 이 새로운 IP 주소에 [A 레코드를 다시 매핑](/azure/app-service/app-service-web-tutorial-custom-domain)합니다.

#### <a name="test-https"></a>HTTPS 테스트

다른 브라우저에서 `https://<your.custom.domain>`으로 이동하여 웹앱이 제공되는지 확인합니다.

![웹앱으로 이동](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> 인증서 유효성 검사 오류가 발생하는 경우 자체 서명된 인증서가 원인이거나, PFX 파일로 내보낼 때 중간 인증서가 남겨진 것일 수 있습니다.

#### <a name="enforce-https"></a>HTTPS 적용

기본적으로 누구나 HTTP를 사용하여 웹앱에 액세스할 수 있습니다. HTTPS 포트에 대한 모든 HTTP 요청을 리디렉션할 수 있습니다.

웹앱 페이지에서 **SL 설정** 을 선택합니다. 그런 다음 **HTTPS에만 해당** 에서 **켜기** 를 선택합니다.

![HTTPS 적용](media/solution-deployment-guide-geo-distributed/image43.png)

작업이 완료되면 앱을 가리키는 HTTP URL 중 하나로 이동합니다. 예를 들면 다음과 같습니다.

- https://<app_name>.azurewebsites.net
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a>TLS 1.1/1.2 적용

[TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0은 기본적으로 앱에서 허용되며, 더 이상 [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard) 같은 산업 표준에서 안전한 것으로 간주되지 않습니다. 더 높은 TLS 버전을 적용하려면 다음이 단계를 수행합니다.

1. 웹앱 페이지의 왼쪽 탐색 영역에서 **SSL 설정** 을 선택합니다.

2. **TLS 버전** 에서 원하는 최소 TLS 버전을 선택합니다.

    ![TLS 1.1 또는 1.2 적용](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a>Traffic Manager 프로필 만들기

1. **리소스 만들기** > **네트워킹** > **Traffic Manager 프로필** > **만들기** 를 선택합니다.

2. **Traffic Manager 프로필 만들기** 에 다음과 같이 입력합니다.

    1. **이름** 에 프로필 이름을 입력합니다. 이 이름은 trafficmanager.net 영역 내에서 고유해야 하며, Traffic Manager 프로필에 액세스하는 데 사용되는 DNS 이름인 trafficmanager.net이 됩니다.

    2. **라우팅 방법** 에서 **지리적 라우팅 방법** 을 선택합니다.

    3. **구독** 에서 이 프로필을 만들 구독을 선택합니다.

    4. **리소스 그룹** 에서 이 프로필을 배치할 새 리소스 그룹을 만듭니다.

    5. **리소스 그룹 위치** 에서 리소스 그룹의 위치를 선택합니다. 이 설정은 리소스 그룹의 위치를 나타내며 전역적으로 배포되는 Traffic Manager 프로필에는 영향을 미치지 않습니다.

    6. **만들기** 를 선택합니다.

    7. Traffic Manager 프로필의 글로벌 배포가 완료되면 각 리소스 그룹의 리소스 중 하나로 나열됩니다.

        ![Traffic Manager 프로필 만들기의 리소스 그룹](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a>Traffic Manager 엔드포인트 추가

1. 포털의 검색 창에서 이전 섹션에서 만든 **Traffic Manager 프로필** 이름을 검색하고, 표시되는 결과에서 해당 Traffic Manager 프로필을 선택합니다.

2. **Traffic Manager 프로필** 의 **설정** 섹션에서 **엔드포인트** 를 선택합니다.

3. **추가** 를 선택합니다.

4. Azure Stack Hub 엔드포인트를 추가합니다.

5. **형식** 의 경우 **외부 엔드포인트** 를 선택합니다.

6. 이 엔드포인트의 **이름** 를 입력합니다. Azure Stack Hub의 이름을 사용하는 것이 가장 좋습니다.

7. **FQDN**(정규화된 도메인 이름)에는 Azure Stack Hub 웹앱의 외부 URL을 사용합니다.

8. 지리적 매핑에서 리소스가 있는 지역/대륙을 선택합니다. 예를 들어 **유럽** 을 선택합니다.

9. 표시되는 국가/지역 드롭다운에서 이 엔드포인트에 적용되는 국가를 선택합니다. 예를 들어 **독일** 을 선택합니다.

10. **사용 안 함으로 추가** 를 선택 취소 상태로 유지합니다.

11. **확인** 을 선택합니다.

12. Azure 엔드포인트를 추가합니다.

    1. **형식** 에는 **Azure 엔드포인트** 를 선택합니다.

    2. 엔드포인트에 사용할 **이름** 을 입력합니다.

    3. **대상 리소스 종류** 에는 **App Service** 를 선택합니다.

    4. **대상 리소스** 의 경우 **앱 서비스 선택** 을 선택하여 동일한 구독에 속한 웹앱 목록을 표시합니다. **리소스** 에서 첫 번째 엔드포인트로 사용할 앱 서비스를 선택합니다.

13. 지리적 매핑에서 리소스가 있는 지역/대륙을 선택합니다. 예를 들어 **북아메리카/중앙 아메리카/카리브 해** 를 선택합니다.

14. 표시되는 국가/지역 드롭다운에서 이 상자를 비워 두어 위의 모든 지역 그룹화를 선택합니다.

15. **사용 안 함으로 추가** 를 선택 취소 상태로 유지합니다.

16. **확인** 을 선택합니다.

    > [!Note]  
    >  리소스의 기본 엔드포인트로 사용할, 지리적 범위가 모두(세계)인 엔드포인트를 하나 이상 만듭니다.

17. 두 엔드포인트가 추가되면 두 엔드포인트가 **Traffic Manager 프로필** 에 표시되고 모니터링 상태는 **온라인** 으로 나타납니다.

    ![Traffic Manager 프로필 엔드포인트 상태](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a>글로벌 기업은 Azure 지역 분산 기능을 사용합니다.

Azure Traffic Manager 및 지리적 엔드포인트를 통해 데이터 트래픽을 전송하면 글로벌 기업은 현지의 데이터 관련 규정을 준수하고 데이터를 안전하게 유지할 수 있으며, 이는 로컬 및 원격 비즈니스 위치의 성공에 매우 중요합니다.

## <a name="next-steps"></a>다음 단계

- Azure Cloud 패턴에 대해 자세히 알아보려면 [클라우드 디자인 패턴](/azure/architecture/patterns)을 참조하세요.
