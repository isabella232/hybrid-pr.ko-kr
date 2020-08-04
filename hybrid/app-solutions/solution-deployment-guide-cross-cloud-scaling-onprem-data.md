---
title: 클라우드 간 크기를 조정하는 온-프레미스 데이터를 사용하는 하이브리드 앱 배포
description: 온-프레미스 데이터를 사용하고 Azure 및 Azure Stack Hub를 사용하여 클라우드 간 크기를 조정하는 앱을 배포하는 방법을 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 6de35cb55c4c35a2a9927f9ffc2516ccb00cd89f
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477323"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a>클라우드 간 크기를 조정하는 온-프레미스 데이터를 사용하는 하이브리드 앱 배포

이 솔루션 가이드는 Azure와 Azure Stack Hub 모두에 걸쳐 있고 단일 온-프레미스 데이터 원본을 사용하는 하이브리드 앱을 배포하는 방법을 보여줍니다.

하이브리드 클라우드 솔루션을 사용하여 프라이빗 클라우드의 규정 준수 이점을 퍼블릭 클라우드의 확장성과 결합할 수 있습니다. 개발자는 Microsoft 개발자 에코 시스템을 활용하여 클라우드 및 온-프레미스 환경에 기술을 적용할 수도 있습니다.

## <a name="overview-and-assumptions"></a>개요 및 가정

이 자습서에 따라 개발자가 퍼블릭 클라우드와 프라이빗 클라우드에 동일한 웹앱을 배포할 수 있는 워크플로를 설정합니다. 이 앱은 프라이빗 클라우드에서 호스트되는 비인터넷 라우팅 가능 네트워크에 액세스할 수 있습니다. 이러한 웹앱은 모니터링되며 트래픽이 급증하는 경우에는 트래픽이 퍼블릭 클라우드로 리디렉션되도록 프로그램에서 DNS 레코드가 수정됩니다. 트래픽이 급증 이전 수준으로 떨어지면 다시 프라이빗 클라우드로 트래픽이 라우팅됩니다.

이 자습서에서 다루는 작업은 다음과 같습니다.

> [!div class="checklist"]
> - 하이브리드 연결 SQL Server 데이터베이스 서버를 배포합니다.
> - 글로벌 Azure의 웹앱을 하이브리드 네트워크에 연결합니다.
> - 클라우드 간 크기 조정을 위해 DNS 구성
> - 클라우드 간 크기 조정을 위해 SSL 인증서를 구성합니다.
> - 웹앱을 구성하고 배포합니다.
> - Traffic Manager 프로필을 만들고 클라우드 간 크기 조정에 맞게 구성합니다.
> - Application Insights 모니터링 및 트래픽 증가에 대한 경고를 설정합니다.
> - 글로벌 Azure와 Azure Stack Hub 간에 자동 트래픽 전환을 구성합니다.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub는 Azure의 확장입니다. Azure Stack Hub는 온-프레미스 환경에 클라우드 컴퓨팅의 민첩성과 혁신을 제공하여 어디서나 하이브리드 앱을 빌드하고 배포할 수 있는 유일한 하이브리드 클라우드를 사용하도록 설정합니다.  
> 
> [하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 문서는 하이브리드 앱 디자인, 배포 및 운영에 대한 소프트웨어 품질(배치, 확장성, 가용성, 복원력, 관리 효율성 및 보안)의 핵심 요소를 검토합니다. 디자인 고려 사항은 하이브리드 앱 디자인을 최적화하고 프로덕션 환경에서 문제를 최소화하는 데 도움이 됩니다.

### <a name="assumptions"></a>가정

이 자습서에서는 글로벌 Azure 및 Azure Stack Hub에 대한 기본 지식이 있다고 가정합니다. 자습서를 시작하기 전에 자세히 알아보려면 다음 문서를 검토하세요.

- [Azure 소개](https://azure.microsoft.com/overview/what-is-azure/)
- [Azure Stack Hub 주요 개념](/azure-stack/operator/azure-stack-overview.md)

이 자습서에서는 Azure 구독이 있다고 가정합니다. 구독이 없는 경우 시작하기 전에 [체험 계정](https://azure.microsoft.com/free/)을 만드세요.

## <a name="prerequisites"></a>필수 구성 요소

이 솔루션을 시작하기 전에 다음 요구 사항을 충족하는지 확인하십시오.

- ASDK(Azure Stack Development Kit) 또는 Azure Stack Hub 통합 시스템 구독 ASDK을 배포하려면 [설치 프로그램을 사용하여 ASDK 배포](/azure-stack/asdk/asdk-install.md)의 지침을 참조하세요.
- Azure Stack Hub 설치에는 다음이 설치되어 있어야 합니다.
  - Azure App Service. Azure Stack Hub 운영자와 협력하여 Azure App Service를 환경에 배포하고 구성하세요. 이 자습서에서는 App Service에 사용 가능한 전용 작업자 역할이 하나(1) 이상 있어야 합니다.
  - Windows Server 2016 이미지.
  - Microsoft SQL Server 이미지를 포함하는 Windows Server 2016.
  - 적절한 계획 및 제안.
  - 웹앱의 도메인 이름. 도메인 이름이 없는 경우 GoDaddy, Bluehost 및 InMotion과 같은 도메인 공급자로부터 구입할 수 있습니다.
- 신뢰할 수 있는 인증 기관(예: LetsEncrypt)의 도메인에 대한 SSL 인증서
- SQL Server 데이터베이스와 통신하고 Application Insights를 지원하는 웹앱. GitHub에서 [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) 샘플 앱을 다운로드할 수 있습니다.
- Azure 가상 네트워크와 Azure Stack Hub 가상 네트워크 간의 하이브리드 네트워크. 자세한 지침은 [Azure 및 Azure Stack Hub를 사용하여 하이브리드 클라우드 연결 구성](solution-deployment-guide-connectivity.md)을 참조하세요.

- Azure Stack Hub에 프라이빗 빌드 에이전트가 있는 하이브리드 CI/CD(연속 통합/지속적인 배포) 파이프라인. 자세한 지침은 [Azure 및 Azure Stack Hub 앱을 사용하여 하이브리드 클라우드 ID 구성](solution-deployment-guide-identity.md)을 참조하세요.

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a>하이브리드 연결 SQL Server 데이터베이스 서버 배포

1. Azure Stack Hub 사용자 포털에 로그인합니다.

2. **대시보드**에서 **Marketplace**를 선택합니다.

    ![Azure Stack Hub Marketplace](media/solution-deployment-guide-hybrid/image1.png)

3. **Marketplace**에서 **Compute**를 선택한 다음, **추가**를 선택합니다. **기타**에서 **무료 SQL Server 라이선스: Windows Server의 SQL Server 2017 Developer** 이미지를 선택합니다.

    ![Azure Stack Hub 사용자 포털에서 가상 머신 이미지를 선택합니다.](media/solution-deployment-guide-hybrid/image2.png)

4. **체험 SQL Server 라이선스: Windows Server의 SQL Server 2017 Developer**에서 **만들기**를 선택합니다.

5. **기본 > 기본 설정 구성**에서 VM(가상 머신)의 **이름**, SQL Server SA의 **사용자 이름**, SA의 **암호**를 입력합니다.  **구독** 드롭다운 목록에서 배포하려는 구독을 선택합니다. **리소스 그룹**의 경우 **기존 항목 선택**을 사용하고 Azure Stack Hub 웹앱과 동일한 리소스 그룹에 VM을 배치합니다.

    ![Azure Stack Hub 사용자 포털에서 VM에 대한 기본 설정 구성](media/solution-deployment-guide-hybrid/image3.png)

6. **크기**에서 VM의 크기를 선택합니다. 이 자습서에서는 A2_Standard 또는 DS2_V2_Standard를 선택하는 것이 좋습니다.

7. **설정 > 옵션 기능 구성**에서 다음 설정을 구성합니다.

   - **스토리지 계정**: 필요한 경우 새 계정을 만듭니다.
   - **가상 네트워크**:

     > [!Important]  
     > VPN 게이트웨이와 동일한 가상 네트워크에 SQL Server VM이 배포되어 있는지 확인합니다.

   - **공용 IP 주소**: 기본 설정을 사용합니다.
   - **네트워크 보안 그룹**: (NSG). 새 NSG를 만듭니다.
   - **확장 및 모니터링**: 기본 설정을 유지합니다.
   - **진단 스토리지 계정**: 필요한 경우 새 계정을 만듭니다.
   - **확인**을 선택하여 구성을 저장합니다.

     ![Azure Stack Hub 사용자 포털에서 선택적 VM 기능 구성](media/solution-deployment-guide-hybrid/image4.png)

8. **SQL Server 설정**에서 다음 설정을 구성합니다.

   - **SQL 연결**에는 **공용(인터넷)** 을 선택합니다.
   - **포트**의 경우 기본값인 **1433**을 유지합니다.
   - **SQL 인증**에는 **사용**을 선택합니다.

     > [!Note]  
     > SQL 인증을 사용하도록 설정하면 **기본**에서 구성한 "SQLAdmin" 정보가 자동으로 채워집니다.

   - 나머지 설정에 대해서는 기본값을 유지합니다. **확인**을 선택합니다.

     ![Azure Stack Hub 사용자 포털에서 SQL Server 설정 구성](media/solution-deployment-guide-hybrid/image5.png)

9. **요약**에서 VM 구성을 검토한 다음, **확인**을 선택하여 배포를 시작합니다.

    ![Azure Stack Hub 사용자 포털의 구성 요약](media/solution-deployment-guide-hybrid/image6.png)

10. 새 VM을 만드는 데 다소 시간이 걸립니다. **가상 머신**에서 VM의 상태를 볼 수 있습니다.

    ![Azure Stack Hub 사용자 포털의 가상 머신 상태](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a>Azure 및 Azure Stack Hub에서 웹앱 만들기

Azure App Service는 웹앱 실행 및 관리를 간소화합니다. Azure Stack Hub는 Azure와 일치하기 때문에 App Service를 두 환경 모두에서 실행할 수 있습니다. App Service를 사용하여 앱을 호스트합니다.

### <a name="create-web-apps"></a>웹앱 만들기

1. [Azure에서 App Service 계획 관리](/azure/app-service/app-service-plan-manage#create-an-app-service-plan)의 지침에 따라 Azure에서 웹앱을 만듭니다. 웹앱을 하이브리드 네트워크와 동일한 구독 및 리소스 그룹에 배치해야 합니다.

2. Azure Stack Hub에서 이전 단계(1)를 반복합니다.

### <a name="add-route-for-azure-stack-hub"></a>Azure Stack Hub에 대한 경로 추가

사용자가 앱에 액세스할 수 있도록 Azure Stack Hub의 App Service는 공용 인터넷에서 라우팅할 수 있어야 합니다. Azure Stack Hub를 인터넷에서 액세스할 수 있는 경우 Azure Stack Hub 웹앱의 공용 IP 주소 또는 URL을 적어둡니다.

ASDK를 사용하는 경우 가상 환경 외부에 App Service가 노출되도록 [정적 NAT 매핑을 구성](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal)할 수 있습니다.

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a>Azure의 웹앱을 하이브리드 네트워크에 연결

Azure의 웹 프런트 엔드와 Azure Stack Hub의 SQL Server 데이터베이스 간에 연결을 제공하려면 Azure와 Azure Stack Hub 사이의 하이브리드 네트워크에 웹앱이 연결되어 있어야 합니다. 연결이 가능하도록 설정하려면 다음을 수행해야 합니다.

- 지점 및 사이트 간 연결 구성
- 웹앱 구성
- Azure Stack Hub에서 로컬 네트워크 게이트웨이 수정

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a>지점 및 사이트 간 연결을 위해 Azure 가상 네트워크 구성

하이브리드 네트워크의 Azure 쪽에 있는 가상 네트워크 게이트웨이는 지점 및 사이트 간 연결이 Azure App Service와 통합되도록 허용해야 합니다.

1. Azure에서 가상 네트워크 게이트웨이 페이지로 이동합니다. **설정**에서 **지점 및 사이트 간 구성**을 선택합니다.

    ![Azure 가상 네트워크 게이트웨이의 지점 및 사이트 간 옵션](media/solution-deployment-guide-hybrid/image8.png)

2. **지금 구성**을 선택하여 지점 및 사이트 간 구성을 수행합니다.

    ![Azure 가상 네트워크 게이트웨이에서 지점 및 사이트 간 구성 시작](media/solution-deployment-guide-hybrid/image9.png)

3. **지점 및 사이트 간** 구성 페이지에서 **주소 풀**에 사용하려는 개인 IP 주소 범위를 입력합니다.

   > [!Note]  
   > 지정한 범위는 하이브리드 네트워크의 글로벌 Azure 또는 Azure Stack Hub 구성 요소의 서브넷에 이미 사용된 주소 범위와 겹치지 않아야 합니다.

   **터널 종류**에서 **IKEv2 VPN**을 선택 취소합니다. **저장**을 선택하여 지점 및 사이트 간 구성을 마칩니다.

   ![Azure 가상 네트워크 게이트웨이의 지점 및 사이트 간 설정](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a>하이브리드 네트워크와 Azure App Service 앱 통합

1. 앱을 Azure VNet에 연결하려면 [게이트웨이에 VNet 통합 필요](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration)의 지침을 따르세요.

2. 웹앱을 호스트하는 App Service 계획에 대한 **설정**으로 이동합니다. **설정**에서 **네트워킹**을 선택합니다.

    ![App Service 계획에 대한 네트워킹 구성](media/solution-deployment-guide-hybrid/image11.png)

3. **VNet 통합**에서 **관리하려면 여기를 클릭**을 선택하세요.

    ![App Service 계획에 대한 VNet 통합 관리](media/solution-deployment-guide-hybrid/image12.png)

4. 구성하려는 VNet을 선택합니다. **VNet으로 라우팅되는 IP 주소** 아래에 Azure VNet, Azure Stack Hub VNet, 지점 및 사이트 간 주소 공간의 IP 주소 범위를 입력합니다. **저장**을 선택하여 설정을 확인하고 저장합니다.

    ![Virtual Network 통합에서 라우팅할 IP 주소 범위](media/solution-deployment-guide-hybrid/image13.png)

App Service가 Azure VNet과 통합되는 방법에 대해 자세히 알아보려면 [Azure Virtual Network에 앱 통합](/azure/app-service/web-sites-integrate-with-vnet)을 참조하세요.

### <a name="configure-the-azure-stack-hub-virtual-network"></a>Azure Stack Hub 가상 네트워크 구성

Azure Stack Hub 가상 네트워크의 로컬 네트워크 게이트웨이는 App Service 지점 및 사이트 간 주소 범위에서 트래픽을 라우팅하도록 구성해야 합니다.

1. Azure Stack Hub에서 **로컬 네트워크 게이트웨이**로 이동합니다. **설정**에서 **구성**을 선택합니다.

    ![Azure Stack Hub 로컬 네트워크 게이트웨이의 게이트웨이 구성 옵션](media/solution-deployment-guide-hybrid/image14.png)

2. **주소 공간**에 Azure의 가상 네트워크 게이트웨이의 지점 및 사이트 간 주소 범위를 입력합니다.

    ![Azure Stack Hub 로컬 네트워크 게이트웨이의 지점 및 사이트 간 주소 공간](media/solution-deployment-guide-hybrid/image15.png)

3. **저장**을 선택하여 구성의 유효성을 검사하고 저장합니다.

## <a name="configure-dns-for-cross-cloud-scaling"></a>클라우드 간 크기 조정을 위해 DNS 구성

클라우드 간 앱에 대해 DNS를 올바르게 구성하면, 웹앱의 글로벌 Azure 및 Azure Stack Hub 인스턴스에 사용자가 액세스할 수 있습니다. 이 자습서의 DNS 구성을 사용하면 부하가 늘어나거나 감소하는 경우 Azure Traffic Manager가 트래픽을 라우팅할 수도 있습니다.

이 자습서에서는 App Service 도메인이 작동하지 않기 때문에 Azure DNS를 사용하여 DNS를 관리합니다.

### <a name="create-subdomains"></a>하위 도메인 만들기

Traffic Manager는 DNS CNAME을 사용하기 때문에 트래픽을 엔드포인트로 적절하게 라우팅하려면 하위 도메인이 필요합니다. DNS 레코드 및 도메인 매핑에 대한 자세한 내용은 [Traffic Manager를 사용하여 도메인 매핑](/azure/app-service/web-sites-traffic-manager-custom-domain-name)을 참조하세요.

Azure 엔드포인트의 경우 사용자가 웹앱에 액세스하는 데 사용할 수 있는 하위 도메인을 만듭니다. 이 자습서에서는 **app.northwind.com**을 사용할 수 있지만, 자체 도메인을 기반으로 이 값을 사용자 지정해야 합니다.

또한 Azure Stack Hub 엔드포인트에 대한 A 레코드로 하위 도메인을 만들어야 합니다. **azurestack.northwind.com**을 사용할 수 있습니다.

### <a name="configure-a-custom-domain-in-azure"></a>Azure에서 사용자 지정 도메인 구성

1. [CNAME을 Azure App Service에 매핑](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record)하여 **app.northwind.com** 호스트 이름을 Azure 웹앱에 추가합니다.

### <a name="configure-custom-domains-in-azure-stack-hub"></a>Azure Stack Hub에서 사용자 지정 도메인 구성

1. [A 레코드를 Azure App Service에 매핑](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record)하여 **azurestack.northwind.com** 호스트 이름을 Azure Stack Hub 웹앱에 추가합니다. App Service 앱에 인터넷 라우팅 가능 IP 주소를 사용합니다.

2. [CNAME을 Azure App Service에 매핑](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record)하여 **app.northwind.com** 호스트 이름을 Azure Stack Hub 웹앱에 추가합니다. 이전 단계(1)에서 구성한 호스트 이름을 CNAME의 대상으로 사용합니다.

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a>클라우드 간 크기 조정을 위해 SSL 인증서 구성

웹앱에서 수집한 중요한 데이터가 SQL 데이터베이스에 저장되거나 전송될 때 보안을 유지하는 것이 중요합니다.

들어오는 모든 트래픽에 대해 SSL 인증서를 사용하도록 Azure 및 Azure Stack Hub 웹앱을 구성합니다.

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a>Azure 및 Azure Stack Hub에 SSL 추가

Azure에 SSL을 추가하려면 다음을 수행합니다.

1. 확보한 SSL 인증서가 생성한 하위 도메인에 유효한지 확인합니다. (와일드카드 인증서를 사용해도 됩니다.)

2. Azure의 [기존 사용자 지정 SSL 인증서를 Azure Web Apps에 바인딩](/azure/app-service/app-service-web-tutorial-custom-ssl) 문서에서 **SSL 인증서 바인딩** 및 **웹앱 준비** 섹션의 지침을 따릅니다. **SSL 형식**으로 **SNI 기반 SSL**을 선택합니다.

3. 모든 트래픽을 HTTPS 포트로 리디렉션합니다. [기존 사용자 지정 SSL 인증서를 Azure Web Apps에 바인딩](/azure/app-service/app-service-web-tutorial-custom-ssl) 문서에서 **HTTPS 적용** 섹션의 지침을 따릅니다.

Azure Stack Hub에 SSL을 추가하려면 다음을 수행합니다.

1. Azure에 사용한 1-3 단계를 반복합니다.

## <a name="configure-and-deploy-the-web-app"></a>웹앱 구성 및 배포

원격 분석을 올바른 Application Insights 인스턴스에 보고하고 올바른 연결 문자열로 웹앱을 구성하도록 앱 코드를 구성합니다. Application Insights에 대해 자세히 알아보려면 [Application Insights란?](/azure/application-insights/app-insights-overview)을 참조하세요.

### <a name="add-application-insights"></a>Application Insights 추가

1. Microsoft Visual Studio에서 웹앱을 엽니다.

2. 웹 트래픽이 늘어나거나 감소하는 경우 Application Insights가 경고를 생성하는 데 사용하는 원격 분석을 전송하도록 [Application Insights를 프로젝트에 추가](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications)합니다.

### <a name="configure-dynamic-connection-strings"></a>동적 연결 문자열 구성

웹앱의 각 인스턴스는 다른 방법을 사용하여 SQL 데이터베이스에 연결합니다. Azure의 앱은 SQL Server VM의 개인 IP 주소를 사용하고 Azure Stack Hub의 앱은 SQL Server VM의 공용 IP 주소를 사용합니다.

> [!Note]  
> Azure Stack Hub 통합 시스템에서 공용 IP 주소는 인터넷 라우팅이 가능하지 않아야 합니다. ASDK에서는 공용 IP 주소는 ASDK 외부로 라우팅할 수 없습니다.

App Service 환경 변수를 사용하여 앱의 각 인스턴스에 다른 연결 문자열을 전달할 수 있습니다.

1. Visual Studio에서 앱을 엽니다.

2. Startup.cs를 열고 다음 코드 블록을 찾습니다.

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. 이전 코드 블록을 다음 코드로 바꿉니다. 이 코드는 *appsettings* 파일에 정의된 연결 문자열을 사용합니다.

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a>App Service 앱 설정 구성

1. Azure 및 Azure Stack Hub에 대한 연결 문자열을 만듭니다. 이 문자열은 사용되는 IP 주소를 제외하고 동일해야 합니다.

2. Azure 및 Azure Stack Hub에서 웹앱의 [앱 설정으로](/azure/app-service/web-sites-configure) 적절한 연결 문자열을 추가합니다(`SQLCONNSTR\_`를 이름의 접두사로 사용).

3. 웹앱 설정을 **저장**하고 앱을 다시 시작합니다.

## <a name="enable-automatic-scaling-in-global-azure"></a>글로벌 Azure에서 자동 크기 조정 사용

App Service Environment에서 웹앱을 만드는 경우 하나의 인스턴스로 시작합니다. 자동으로 규모를 확장하여 인스턴스를 추가하고 앱에 컴퓨팅 리소스를 더 많이 제공할 수 있습니다. 마찬가지로, 자동으로 규모를 감축하고 앱에 필요한 인스턴스 수를 줄일 수 있습니다.

> [!Note]  
> 규모 확장 및 규모 감축을 구성하려면 App Service 계획이 있어야 합니다. 계획이 없으면 다음 단계를 시작하기 전에 계획을 만드십시오.

### <a name="enable-automatic-scale-out"></a>자동 규모 확장 사용

1. Azure에서 규모를 확장하려는 사이트에 대한 App Service 계획을 찾은 다음, **규모 확장(App Service 계획)** 을 선택합니다.

    ![Azure App Service 규모 확장](media/solution-deployment-guide-hybrid/image16.png)

2. **자동 크기 조정 사용**을 선택합니다.

    ![Azure App Service에서 자동 크기 조정 사용](media/solution-deployment-guide-hybrid/image17.png)

3. **자동 크기 조정 설정 이름**에 이름을 입력합니다. **기본** 자동 크기 조정 규칙의 경우 **메트릭 기준 크기 조정**을 선택합니다. **인스턴스 제한**을 **최소: 1**, **최대: 10**, **기본값: 1**로 설정합니다.

    ![Azure App Service에서 자동 크기 조정 구성](media/solution-deployment-guide-hybrid/image18.png)

4. **+ 규칙 추가**를 선택합니다.

5. **메트릭 원본**에서 **현재 리소스**를 선택합니다. 규칙에 대해 다음 조건 및 동작을 사용합니다.

#### <a name="criteria"></a>조건

1. **시간 집계**에서 **평균**을 선택합니다.

2. **메트릭 이름**에서 **CPU 백분율**을 선택합니다.

3. **연산자**에서 **보다 큼**을 선택합니다.

   - **임계값**을 **50**으로 설정합니다.
   - **기간**을 **10**으로 설정합니다.

#### <a name="action"></a>작업

1. **작업**에서 **다음을 기준으로 개수 늘이기**를 선택합니다.

2. **인스턴스 수**를 **2**로 설정합니다.

3. **정지 시간**을 **5**로 설정합니다.

4. **추가**를 선택합니다.

5. **+ 규칙 추가**를 선택합니다.

6. **메트릭 원본**에서 **현재 리소스**를 선택합니다.

   > [!Note]  
   > 현재 리소스에 App Service 계획의 이름/GUID가 포함되며 **리소스 유형** 및 **리소스** 드롭다운 목록은 사용할 수 없습니다.

### <a name="enable-automatic-scale-in"></a>자동 규모 감축 사용

트래픽이 감소하면 Azure 웹앱이 활성 인스턴스 수를 자동으로 줄여서 비용을 절감할 수 있습니다. 이 작업은 규모 확장보다 덜 적극적이며 앱 사용자에게 미치는 영향을 최소화합니다.

1. **기본** 규모 확장 조건으로 이동한 다음, **+ 규칙 추가**를 선택합니다. 규칙에 대해 다음 조건 및 동작을 사용합니다.

#### <a name="criteria"></a>조건

1. **시간 집계**에서 **평균**을 선택합니다.

2. **메트릭 이름**에서 **CPU 백분율**을 선택합니다.

3. **연산자**에서 **보다 작음**을 선택합니다.

   - **임계값**을 **30**으로 설정합니다.
   - **기간**을 **10**으로 설정합니다.

#### <a name="action"></a>작업

1. **작업**에서 **다음을 기준으로 개수 줄이기**를 선택합니다.

   - **인스턴스 수**를 **1**로 설정합니다.
   - **정지 시간**을 **5**로 설정합니다.

2. **추가**를 선택합니다.

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a>Traffic Manager 프로필을 만들고 클라우드 간 크기 조정에 맞게 구성

Azure에서 Traffic Manager 프로필을 만든 다음, 클라우드 간 크기 조정이 가능하도록 엔드포인트를 설정합니다.

### <a name="create-traffic-manager-profile"></a>Traffic Manager 프로필 만들기

1. **리소스 만들기**를 선택합니다.
2. **네트워킹**을 선택합니다.
3. **Traffic Manager 프로필**을 선택하고 다음 설정을 구성합니다.

   - **이름**에 사용자의 프로필에 사용할 이름을 입력합니다. 이 이름은 trafficmanager.net 영역에서 **고유해야 하며**, 새 DNS 이름(예: northwindstore.trafficmanager.net)을 만드는 데 사용됩니다.
   - **라우팅 방법**으로 **가중치**를 선택합니다.
   - **구독**에는 이 프로필을 만들 구독을 선택합니다.
   - **리소스 그룹**에서 이 프로필의 새 리소스 그룹을 만듭니다.
   - **리소스 그룹 위치**에서 리소스 그룹의 위치를 선택합니다. 이 설정은 리소스 그룹의 위치를 나타내며 전역적으로 배포되는 Traffic Manager 프로필에는 영향을 미치지 않습니다.

4. **만들기**를 선택합니다.

    ![Traffic Manager 프로필 만들기](media/solution-deployment-guide-hybrid/image19.png)

   Traffic Manager 프로필의 전역 배포가 완료되면 프로필을 만든 리소스 그룹의 리소스 목록에 해당 프로필이 표시됩니다.

### <a name="add-traffic-manager-endpoints"></a>Traffic Manager 엔드포인트 추가

1. 만든 Traffic Manager 프로필을 검색합니다. 프로필의 리소스 그룹으로 이동한 경우 프로필을 선택합니다.

2. **Traffic Manager 프로필**의 **설정**에서 **엔드포인트**를 선택합니다.

3. **추가**를 선택합니다.

4. **엔드포인트 추가**에서 Azure Stack Hub에 대해 다음 설정을 사용합니다.

   - **형식**의 경우 **외부 엔드포인트**를 선택합니다.
   - 엔드포인트에 사용할 **이름**을 입력합니다.
   - **FQDN(정규화된 도메인 이름) 또는 IP**에는 Azure Stack Hub 웹앱의 외부 URL을 입력합니다.
   - **가중치**의 경우 기본값 **1**을 유지합니다. 가중치를 이렇게 설정하면 모든 트래픽이 정상일 경우 이 엔드포인트로 이동합니다.
   - **사용 안 함으로 추가**는 선택하지 않은 상태로 유지합니다.

5. **확인**을 선택하여 Azure Stack Hub 엔드포인트를 저장합니다.

다음으로 Azure 엔드포인트를 구성합니다.

1. **Traffic Manager 프로필**에서 **엔드포인트**를 선택합니다.
2. **+추가**를 선택합니다.
3. **엔드포인트 추가**에서 Azure에 대해 다음 설정을 사용합니다.

   - **형식**에는 **Azure 엔드포인트**를 선택합니다.
   - 엔드포인트에 사용할 **이름**을 입력합니다.
   - **대상 리소스 형식**에는 **App Service**를 선택합니다.
   - **대상 리소스**의 경우 동일한 구독의 Web Apps 목록을 볼 수 있도록 **앱 서비스 선택**을 선택합니다.
   - **리소스**에서 첫 번째 엔드포인트로 추가할 앱 서비스를 선택합니다.
   - **가중치**로 **2**를 선택합니다. 이렇게 설정하면 기본 엔드포인트가 비정상이거나 트리거되면 트래픽을 리디렉션하는 규칙/경고가 있는 경우 모든 트래픽이 이 엔드포인트로 이동합니다.
   - **사용 안 함으로 추가**는 선택하지 않은 상태로 유지합니다.

4. **확인**을 선택하여 Azure 엔드포인트를 저장합니다.

두 엔드포인트를 모두 구성한 후 **엔드포인트**를 선택하면 **Traffic Manager 프로필**에 나열됩니다. 다음 화면 캡처에 있는 예에는 각각 상태 및 구성 정보가 있는 엔드포인트가 두 개 있습니다.

![Traffic Manager 프로필의 엔드포인트](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting"></a>Application Insights 모니터링 및 경고 설정

Azure Application Insights를 사용하면 구성한 조건에 따라 앱을 모니터링하고 경고를 보낼 수 있습니다. 앱을 사용할 수 없음, 앱에 오류가 발생함, 앱에 성능 문제가 있음 등이 그 예입니다.

Application Insights 메트릭을 사용하여 경고를 만듭니다. 이러한 경고가 트리거되면, 웹앱의 인스턴스가 규모 확장을 위해 Azure Stack Hub에서 Azure로 자동 전환된 다음, 규모 감축을 위해 Azure Stack Hub로 다시 전환됩니다.

### <a name="create-an-alert-from-metrics"></a>메트릭에서 경고 만들기

이 자습서의 리소스 그룹으로 이동한 다음, Application Insights 인스턴스를 선택하여 **Application Insights**를 엽니다.

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

이 보기를 사용하여 규모 확장 경고와 규모 감축 경고를 만듭니다.

### <a name="create-the-scale-out-alert"></a>규모 확장 경고 만들기

1. **구성**에서 **경고(클래식)** 를 선택합니다.
2. **메트릭 경고 추가(클래식)** 를 선택합니다.
3. **규칙 추가**에서 다음 설정을 구성합니다.

   - **이름**에 **Burst into Azure Cloud**를 입력합니다.
   - **설명**은 선택 사항입니다.
   - **원본** > **경고 대상**에서 **메트릭**을 선택합니다.
   - **조건**에서 구독, Traffic Manager 프로필의 리소스 그룹, 리소스의 Traffic Manager 프로필 이름을 선택합니다.

4. **메트릭**에 대해 **요청 빈도**를 선택합니다.
5. **조건**에는 **보다 큼**을 선택합니다.
6. **임계값**에 **2**를 입력합니다.
7. **기간**에는 **지난 5분**을 선택합니다.
8. **다음을 통해 알림**에서:
   - **이메일 소유자, 기여자 및 구독자** 확인란을 선택합니다.
   - **추가 관리자 이메일**에 사용할 이메일 주소를 입력합니다.

9. 메뉴 모음에서 **저장**을 선택합니다.

### <a name="create-the-scale-in-alert"></a>규모 감축 경고 만들기

1. **구성**에서 **경고(클래식)** 를 선택합니다.
2. **메트릭 경고 추가(클래식)** 를 선택합니다.
3. **규칙 추가**에서 다음 설정을 구성합니다.

   - **이름**에 **Scale back into Azure Stack Hub**를 입력합니다.
   - **설명**은 선택 사항입니다.
   - **원본** > **경고 대상**에서 **메트릭**을 선택합니다.
   - **조건**에서 구독, Traffic Manager 프로필의 리소스 그룹, 리소스의 Traffic Manager 프로필 이름을 선택합니다.

4. **메트릭**에 대해 **요청 빈도**를 선택합니다.
5. **조건**에는 **보다 작음**을 선택합니다.
6. **임계값**에 **2**를 입력합니다.
7. **기간**에는 **지난 5분**을 선택합니다.
8. **다음을 통해 알림**에서:
   - **이메일 소유자, 기여자 및 구독자** 확인란을 선택합니다.
   - **추가 관리자 이메일**에 사용할 이메일 주소를 입력합니다.

9. 메뉴 모음에서 **저장**을 선택합니다.

다음 스크린샷은 규모 확장 및 규모 감축에 대한 경고를 보여줍니다.

   ![Application Insights 경고(클래식)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a>Azure와 Azure Stack Hub 간에 트래픽 리디렉션

Azure와 Azure Stack Hub 간에 웹앱 트래픽 수동 또는 자동 전환을 구성할 수 있습니다.

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a>Azure와 Azure Stack Hub 간의 수동 전환 구성

웹 사이트가 구성한 임계값에 도달하면 경고를 받게 됩니다. 다음 단계를 사용하여 트래픽을 Azure로 수동 리디렉션합니다.

1. Azure Portal에서 Traffic Manager 프로필을 선택합니다.

    ![Azure Portal의 Traffic Manager 엔드포인트](media/solution-deployment-guide-hybrid/image20.png)

2. **엔드포인트**를 선택합니다.
3. **Azure 엔드포인트**를 선택합니다.
4. **상태**에서 **사용**을 선택한 다음, **저장**을 선택합니다.

    ![Azure Portal에서 Azure 엔드포인트 사용](media/solution-deployment-guide-hybrid/image23.png)

5. Traffic Manager 프로필의 **엔드포인트**에서 **외부 엔드포인트**를 선택합니다.
6. **상태**에서 **사용 안 함**을 선택한 다음, **저장**을 선택합니다.

    ![Azure Portal에서 Azure Stack Hub 엔드포인트 사용 안 함](media/solution-deployment-guide-hybrid/image24.png)

엔드포인트가 구성된 후에는 앱 트래픽이 Azure Stack Hub 웹앱 대신 Azure 규모 확장 웹앱으로 이동합니다.

 ![Azure 웹앱 트래픽에서 엔드포인트가 변경됨](media/solution-deployment-guide-hybrid/image25.png)

흐름을 Azure Stack Hub로 되돌리려면 이전 단계를 사용하여 다음을 수행합니다.

- Azure Stack Hub 엔드포인트를 사용하도록 설정합니다.
- Azure 엔드포인트를 사용하지 않도록 설정합니다.

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a>Azure와 Azure Stack Hub 간의 자동 전환 구성

앱이 Azure Functions에서 제공하는 [서버리스](https://azure.microsoft.com/overview/serverless-computing/) 환경에서 실행되는 경우 Application Insights 모니터링을 사용할 수도 있습니다.

이 시나리오에서는 함수 앱을 호출하는 웹후크를 사용하도록 Application Insights를 구성할 수 있습니다. 이 앱은 경고에 대한 응답으로 엔드포인트를 사용하거나 사용하지 않도록 자동으로 설정합니다.

다음 단계를 지침으로 사용하여 자동 트래픽 전환을 구성합니다.

1. Azure 함수 앱을 만듭니다.
2. HTTP 트리거 함수를 만듭니다.
3. Resource Manager, Web Apps 및 Traffic Manager용 Azure SDK를 가져옵니다.
4. 다음을 수행하는 코드를 개발합니다.

   - Azure 구독을 인증합니다.
   - Traffic Manager 엔드포인트를 전환하는 매개 변수를 사용하여 Azure 또는 Azure Stack Hub로 트래픽을 전달합니다.

5. 코드를 저장하고 Application Insights 경고 규칙 설정의 **웹후크** 섹션에 적절한 매개 변수가 있는 함수 앱의 URL을 추가합니다.
6. Application Insights 경고가 발생하면 트래픽이 자동으로 리디렉션됩니다.

## <a name="next-steps"></a>다음 단계

- Azure Cloud 패턴에 대해 자세히 알아보려면 [클라우드 디자인 패턴](/azure/architecture/patterns)을 참조하세요.
