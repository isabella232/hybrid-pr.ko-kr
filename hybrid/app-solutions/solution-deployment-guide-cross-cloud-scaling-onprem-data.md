---
title: 클라우드 간 크기를 조정 하는 온-프레미스 데이터를 사용 하 여 하이브리드 앱 배포
description: Azure 및 Azure Stack Hub를 사용 하 여 온-프레미스 데이터를 사용 하는 앱을 배포 하 고 클라우드 간 크기를 조정 하는 방법을 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 75289eae902c5363862e345bdedb97cbcee0476e
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910889"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a>클라우드 간 크기를 조정 하는 온-프레미스 데이터를 사용 하 여 하이브리드 앱 배포

이 솔루션 가이드에서는 Azure와 Azure Stack Hub 모두에 걸친 하이브리드 앱을 배포 하 고 단일 온-프레미스 데이터 원본을 사용 하는 방법을 보여 줍니다.

하이브리드 클라우드 솔루션을 사용 하 여 사설 클라우드의 규정 준수 이점을 공용 클라우드의 확장성과 결합할 수 있습니다. 개발자는 Microsoft 개발자 에코 시스템을 활용 하 여 클라우드 및 온-프레미스 환경에 기술을 적용할 수도 있습니다.

## <a name="overview-and-assumptions"></a>개요 및 가정

이 자습서에 따라 개발자가 동일한 웹 앱을 공용 클라우드와 사설 클라우드에 배포할 수 있는 워크플로를 설정 합니다. 이 앱은 사설 클라우드에서 호스트 되는 인터넷 라우팅할 수 없는 네트워크에 액세스할 수 있습니다. 이러한 웹 앱은 모니터링 되 고 트래픽이 급증 하는 경우 프로그램은 트래픽을 공용 클라우드로 리디렉션하도록 DNS 레코드를 수정 합니다. 급증 이전 수준으로 트래픽이 떨어지면 트래픽이 사설 클라우드로 다시 라우팅됩니다.

이 자습서에서 다루는 작업은 다음과 같습니다.

> [!div class="checklist"]
> - 하이브리드 연결 SQL Server 데이터베이스 서버를 배포 합니다.
> - 글로벌 Azure의 웹 앱을 하이브리드 네트워크에 연결 합니다.
> - 클라우드 간 크기 조정에 대해 DNS를 구성 합니다.
> - 클라우드 간 크기 조정에 대 한 SSL 인증서를 구성 합니다.
> - 웹 앱을 구성 하 고 배포 합니다.
> - Traffic Manager 프로필을 만들고 클라우드 간 크기 조정을 위해 구성 합니다.
> - 증가 하는 트래픽에 대 한 모니터링 및 경고를 Application Insights 설정 합니다.
> - 글로벌 Azure와 Azure Stack Hub 간에 자동 트래픽 전환을 구성 합니다.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack 허브는 Azure의 확장입니다. Azure Stack 허브는 온-프레미스 환경에 클라우드 컴퓨팅의 민첩성과 혁신을 제공 하 여 어디서 나 하이브리드 앱을 빌드하고 배포할 수 있는 유일한 하이브리드 클라우드를 사용 하도록 설정 합니다.  
> 
> [하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 문서에서는 하이브리드 앱을 디자인, 배포 및 운영 하기 위한 소프트웨어 품질 (배치, 확장성, 가용성, 복원 력, 관리 효율성 및 보안)의 핵심 요소을 검토 합니다. 디자인 고려 사항은 하이브리드 앱 디자인을 최적화 하 고 프로덕션 환경에서 문제를 최소화 하는 데 도움이 됩니다.

### <a name="assumptions"></a>가정

이 자습서에서는 사용자에 게 글로벌 Azure 및 Azure Stack Hub에 대 한 기본 지식이 있다고 가정 합니다. 자습서를 시작 하기 전에 자세히 알아보려면 다음 문서를 검토 하세요.

- [Azure 소개](https://azure.microsoft.com/overview/what-is-azure/)
- [Azure Stack 허브 주요 개념](/azure-stack/operator/azure-stack-overview.md)

또한이 자습서에서는 Azure 구독이 있다고 가정 합니다. 구독이 없는 경우 시작 하기 전에 [무료 계정을 만듭니다](https://azure.microsoft.com/free/) .

## <a name="prerequisites"></a>필수 조건

이 솔루션을 시작 하기 전에 다음 요구 사항을 충족 하는지 확인 합니다.

- Azure Stack Development Kit (ASDK) 또는 Azure Stack 허브 통합 시스템의 구독입니다. ASDK을 배포 하려면 [설치 관리자를 사용 하 여 Asdk 배포](/azure-stack/asdk/asdk-install.md)의 지침을 따르세요.
- Azure Stack Hub 설치에는 다음이 설치 되어 있어야 합니다.
  - Azure App Service입니다. Azure Stack 허브 운영자에 게 작업 하 여 환경에서 Azure App Service를 배포 하 고 구성 합니다. 이 자습서에서는 App Service에 사용 가능한 전용 작업자 역할이 하나 이상 있어야 합니다.
  - Windows Server 2016 이미지입니다.
  - Microsoft SQL Server 이미지를 포함 하는 Windows Server 2016
  - 적절 한 계획 및 제안.
  - 웹 앱에 대 한 도메인 이름입니다. 도메인 이름이 없는 경우 GoDaddy, Bluehost 및 InMotion 같은 도메인 공급자에서 구매할 수 있습니다.
- 도메인에 대 한 SSL 인증서는 도메인에 대 한 인증서입니다.
- SQL Server 데이터베이스와 통신 하 고 Application Insights을 지 원하는 웹 앱입니다. GitHub에서 [dotnetcore-sqldb 자습서](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) 샘플 앱을 다운로드할 수 있습니다.
- Azure 가상 네트워크와 Azure Stack 허브 가상 네트워크 간의 하이브리드 네트워크 자세한 지침은 [Azure 및 Azure Stack Hub를 사용 하 여 하이브리드 클라우드 연결 구성](solution-deployment-guide-connectivity.md)을 참조 하세요.

- Azure Stack 허브에 개인 빌드 에이전트가 있는 CI/CD (하이브리드 연속 통합/연속 배포) 파이프라인 자세한 지침은 [Azure 및 Azure Stack Hub 앱을 사용 하 여 하이브리드 클라우드 Id 구성](solution-deployment-guide-identity.md)을 참조 하세요.

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a>하이브리드 연결 SQL Server 데이터베이스 서버 배포

1. Azure Stack Hub 사용자 포털에 로그인 합니다.

2. **대시보드에서** **Marketplace**를 선택 합니다.

    ![Azure Stack 허브 마켓플레이스](media/solution-deployment-guide-hybrid/image1.png)

3. **Marketplace**에서 **Compute**를 선택 하 고 **자세히**를 선택 합니다. **자세히**아래에서 **무료 SQL Server 라이선스: SQL Server 2017 Developer on Windows Server** 이미지를 선택 합니다.

    ![Azure Stack Hub 사용자 포털에서 가상 머신 이미지를 선택 합니다.](media/solution-deployment-guide-hybrid/image2.png)

4. **무료 SQL Server 라이선스: SQL Server 2017 Developer On Windows Server**에서 **만들기**를 선택 합니다.

5. **기본 설정 > 기본 설정 구성**에서 VM (가상 머신) **이름** , SQL Server sa의 **사용자 이름** 및 sa **암호** 를 제공 합니다.  **구독** 드롭다운 목록에서 배포 하려는 구독을 선택 합니다. **리소스 그룹**의 경우 **기존 선택** 을 사용 하 여 Azure Stack 허브 웹 앱과 동일한 리소스 그룹에 VM을 배치 합니다.

    ![Azure Stack Hub 사용자 포털에서 VM에 대 한 기본 설정 구성](media/solution-deployment-guide-hybrid/image3.png)

6. **크기**에서 VM의 크기를 선택 합니다. 이 자습서에서는 A2_Standard 또는 DS2_V2_Standard를 권장 합니다.

7. **설정 > 옵션 기능 구성**에서 다음 설정을 구성 합니다.

   - **저장소 계정**: 필요한 경우 새 계정을 만듭니다.
   - **가상 네트워크**:

     > [!Important]  
     > SQL Server VM VPN 게이트웨이와 동일한 가상 네트워크에 배포 되었는지 확인 합니다.

   - **공용 IP 주소**: 기본 설정을 사용 합니다.
   - **네트워크 보안 그룹**: (nsg). 새 NSG를 만듭니다.
   - **확장 및 모니터링**: 기본 설정을 유지 합니다.
   - **진단 저장소 계정**: 필요한 경우 새 계정을 만듭니다.
   - **확인** 을 선택 하 여 구성을 저장 합니다.

     ![Azure Stack Hub 사용자 포털에서 선택적 VM 기능 구성](media/solution-deployment-guide-hybrid/image4.png)

8. **SQL Server 설정**에서 다음 설정을 구성 합니다.

   - **SQL 연결**의 경우 **공용 (인터넷)** 을 선택 합니다.
   - **포트**의 경우 기본값 **1433**을 유지 합니다.
   - **SQL 인증**에 대해 **사용**을 선택 합니다.

     > [!Note]  
     > SQL 인증을 사용 하도록 설정 하면 **기본**설정에서 구성한 "sqladmin" 정보로 자동으로 채워집니다.

   - 나머지 설정에 대해서는 기본값을 유지 합니다. **확인**을 선택합니다.

     ![Azure Stack 허브 사용자 포털에서 SQL Server 설정 구성](media/solution-deployment-guide-hybrid/image5.png)

9. **요약**에서 VM 구성을 검토 한 다음 **확인** 을 선택 하 여 배포를 시작 합니다.

    ![Azure Stack 허브 사용자 포털의 구성 요약](media/solution-deployment-guide-hybrid/image6.png)

10. 새 VM을 만드는 데 다소 시간이 걸립니다. **가상 컴퓨터**에서 VM의 상태를 볼 수 있습니다.

    ![Azure Stack 허브 사용자 포털의 가상 컴퓨터 상태](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a>Azure에서 웹 앱 만들기 및 Azure Stack 허브

Azure App Service는 웹 앱의 실행 및 관리를 간소화 합니다. Azure Stack 허브가 Azure와 일치 하기 때문에 App Service를 두 환경에서 모두 실행할 수 있습니다. App Service를 사용 하 여 앱을 호스팅합니다.

### <a name="create-web-apps"></a>웹앱 만들기

1. Azure에서 [App Service 계획 관리](https://docs.microsoft.com/azure/app-service/app-service-plan-manage#create-an-app-service-plan)의 지침에 따라 azure에서 웹 앱을 만듭니다. 웹 앱을 하이브리드 네트워크와 동일한 구독 및 리소스 그룹에 배치 해야 합니다.

2. Azure Stack 허브에서 이전 단계 (1)를 반복 합니다.

### <a name="add-route-for-azure-stack-hub"></a>Azure Stack 허브에 대 한 경로 추가

사용자가 앱에 액세스할 수 있도록 Azure Stack 허브의 App Service는 공용 인터넷에서 라우팅할 수 있어야 합니다. Azure Stack 허브가 인터넷에서 액세스할 수 있는 경우 Azure Stack 허브 웹 앱에 대 한 공용 IP 주소 또는 URL을 기록해 둡니다.

ASDK를 사용 하는 경우 가상 환경 외부 App Service를 노출 하도록 [정적 NAT 매핑을 구성할](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) 수 있습니다.

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a>Azure의 웹 앱을 하이브리드 네트워크에 연결

Azure의 웹 프런트 엔드와 Azure Stack Hub의 SQL Server 데이터베이스 간에 연결을 제공 하려면 웹 앱이 Azure와 Azure Stack 허브 간에 하이브리드 네트워크에 연결 되어 있어야 합니다. 연결을 사용 하도록 설정 하려면 다음을 수행 해야 합니다.

- 지점 및 사이트 간 연결을 구성 합니다.
- 웹 앱을 구성 합니다.
- Azure Stack 허브에서 로컬 네트워크 게이트웨이를 수정 합니다.

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a>지점 및 사이트 간 연결에 대 한 Azure 가상 네트워크 구성

하이브리드 네트워크의 Azure 쪽에 있는 가상 네트워크 게이트웨이는 지점 및 사이트 간 연결이 Azure App Service와 통합 될 수 있어야 합니다.

1. Azure에서 가상 네트워크 게이트웨이 페이지로 이동 합니다. **설정**아래에서 **지점 및 사이트 간 구성을**선택 합니다.

    ![Azure virtual network 게이트웨이의 지점 및 사이트 간 옵션](media/solution-deployment-guide-hybrid/image8.png)

2. 지점 및 사이트 간을 구성 하려면 **지금 구성** 을 선택 합니다.

    ![Azure virtual network 게이트웨이에서 시작 지점 및 사이트 간 구성](media/solution-deployment-guide-hybrid/image9.png)

3. 지점 및 **사이트 간** 구성 페이지에서 **주소 풀**에 사용 하려는 개인 IP 주소 범위를 입력 합니다.

   > [!Note]  
   > 지정한 범위가 글로벌 Azure의 서브넷에서 이미 사용 하는 주소 범위 또는 하이브리드 네트워크의 Azure Stack 허브 구성 요소와 겹치지 않는지 확인 하세요.

   **터널 유형**에서 **IKEv2 VPN**을 선택 취소 합니다. **저장** 을 선택 하 여 지점 및 사이트 간 구성을 완료 합니다.

   ![Azure virtual network 게이트웨이의 지점 및 사이트 간 설정](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a>하이브리드 네트워크와 Azure App Service 앱 통합

1. 앱을 Azure VNet에 연결 하려면 [게이트웨이 필수 VNet 통합](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration)의 지침을 따르세요.

2. 웹 앱을 호스트 하는 App Service 계획에 대 한 **설정** 으로 이동 합니다. **설정**에서 **네트워킹**을 선택합니다.

    ![App Service 계획에 대 한 네트워킹 구성](media/solution-deployment-guide-hybrid/image11.png)

3. **VNET 통합**에서 **관리 하려면 여기를 클릭**하세요 .를 선택 합니다.

    ![App Service 계획에 대 한 VNET 통합 관리](media/solution-deployment-guide-hybrid/image12.png)

4. 구성 하려는 VNET을 선택 합니다. **VNET에 라우팅되는 IP 주소**에서 Azure vnet, Azure Stack 허브 VNET 및 지점 및 사이트 간 주소 공간에 대 한 ip 주소 범위를 입력 합니다. **저장** 을 선택 하 여 이러한 설정을 확인 하 고 저장 합니다.

    ![Virtual Network 통합에서 라우팅할 IP 주소 범위](media/solution-deployment-guide-hybrid/image13.png)

App Service Azure Vnet와 통합 하는 방법에 대 한 자세한 내용은 [azure Virtual Network와 앱 통합](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet)을 참조 하세요.

### <a name="configure-the-azure-stack-hub-virtual-network"></a>Azure Stack 허브 가상 네트워크 구성

App Service 지점 및 사이트 간 주소 범위에서 트래픽을 라우팅하도록 Azure Stack 허브 가상 네트워크의 로컬 네트워크 게이트웨이를 구성 해야 합니다.

1. Azure Stack 허브에서 **로컬 네트워크 게이트웨이로**이동 합니다. **설정**에서 **구성**을 선택합니다.

    ![Azure Stack 허브 로컬 네트워크 게이트웨이의 게이트웨이 구성 옵션](media/solution-deployment-guide-hybrid/image14.png)

2. **주소 공간**에 Azure의 가상 네트워크 게이트웨이에 대 한 지점 및 사이트 간 주소 범위를 입력 합니다.

    ![Azure Stack 허브 로컬 네트워크 게이트웨이의 지점 및 사이트 간 주소 공간](media/solution-deployment-guide-hybrid/image15.png)

3. **저장** 을 선택 하 여 구성을 확인 하 고 저장 합니다.

## <a name="configure-dns-for-cross-cloud-scaling"></a>클라우드 간 확장을 위한 DNS 구성

사용자는 클라우드 간 앱에 대 한 DNS를 적절히 구성 하 여 웹 앱의 글로벌 Azure 및 Azure Stack Hub 인스턴스에 액세스할 수 있습니다. 이 자습서의 DNS 구성에서는 부하가 늘어나거나 감소할 때 Azure Traffic Manager 트래픽을 라우팅할 수도 있습니다.

이 자습서에서는 Azure DNS 사용 하 여 App Service 도메인이 작동 하지 않기 때문에 DNS를 관리 합니다.

### <a name="create-subdomains"></a>하위 도메인 만들기

Traffic Manager는 DNS CNAMEs를 사용 하기 때문에 트래픽을 끝점으로 적절 하 게 라우팅하도록 하위 도메인이 필요 합니다. DNS 레코드 및 도메인 매핑에 대 한 자세한 내용은 [Traffic Manager를 사용 하 여 도메인 매핑](https://docs.microsoft.com/azure/app-service/web-sites-traffic-manager-custom-domain-name)을 참조 하세요.

Azure 끝점의 경우 사용자가 웹 앱에 액세스 하는 데 사용할 수 있는 하위 도메인을 만듭니다. 이 자습서에서는 **app.northwind.com**를 사용할 수 있지만 사용자 고유의 도메인에 따라이 값을 사용자 지정 해야 합니다.

또한 Azure Stack 허브 끝점에 대 한 A 레코드를 사용 하 여 하위 도메인을 만들어야 합니다. **Azurestack.northwind.com**를 사용할 수 있습니다.

### <a name="configure-a-custom-domain-in-azure"></a>Azure에서 사용자 지정 도메인 구성

1. [Azure App Service에 CNAME을 매핑하여](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record)Azure 웹 앱에 **app.northwind.com** 호스트 이름을 추가 합니다.

### <a name="configure-custom-domains-in-azure-stack-hub"></a>Azure Stack 허브에서 사용자 지정 도메인 구성

1. [A 레코드를 Azure App Service 매핑하여](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record)Azure Stack 허브 웹 앱에 **azurestack.northwind.com** 호스트 이름을 추가 합니다. App Service 앱에 인터넷 라우팅 가능한 IP 주소를 사용 합니다.

2. [CNAME을 Azure App Service 매핑하여](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record)Azure Stack 허브 웹 앱에 **app.northwind.com** 호스트 이름을 추가 합니다. 이전 단계에서 구성한 호스트 이름 (1)을 CNAME의 대상으로 사용 합니다.

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a>클라우드 간 크기 조정에 대 한 SSL 인증서 구성

웹 앱에 의해 수집 된 중요 한 데이터를 SQL 데이터베이스에 저장 하는 경우 안전 하 게 전송할 수 있는지 확인 하는 것이 중요 합니다.

들어오는 모든 트래픽에 대해 SSL 인증서를 사용 하도록 Azure 및 Azure Stack Hub 웹 앱을 구성 합니다.

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a>Azure 및 Azure Stack Hub에 SSL 추가

Azure에 SSL을 추가 하려면:

1. 사용자가 만든 하위 도메인에 대 한 SSL 인증서가 유효한 지 확인 합니다. 와일드 카드 인증서를 사용할 수 있습니다.

2. Azure에서 [azure Web Apps에 기존 사용자 지정 ssl 인증서 바인딩](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl) 문서에 있는 **웹 앱 준비** 및 **SSL 인증서 바인딩** 섹션의 지침을 따릅니다. **SNI 기반 ssl** 을 **ssl 유형**으로 선택 합니다.

3. 모든 트래픽을 HTTPS 포트로 리디렉션합니다. [Azure Web Apps에 기존 사용자 지정 SSL 인증서 바인딩](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl) 문서에서 **HTTPS 적용** 섹션의 지침을 따릅니다.

Azure Stack 허브에 SSL을 추가 하려면 다음을 수행 합니다.

1. Azure에 사용한 1-3 단계를 반복 합니다.

## <a name="configure-and-deploy-the-web-app"></a>웹 앱 구성 및 배포

올바른 Application Insights 인스턴스에 대 한 원격 분석을 보고 하 고 올바른 연결 문자열을 사용 하 여 웹 앱을 구성 하도록 앱 코드를 구성 합니다. Application Insights에 대 한 자세한 내용은 [Application Insights 항목](https://docs.microsoft.com/azure/application-insights/app-insights-overview) 을 참조 하세요.

### <a name="add-application-insights"></a>Application Insights 추가

1. Microsoft Visual Studio에서 웹 앱을 엽니다.

2. 프로젝트에 [Application Insights를 추가](https://docs.microsoft.com/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) 하 Application Insights에서 웹 트래픽이 늘어나거나 감소할 때 경고를 생성 하는 데 사용 하는 원격 분석을 전송 합니다.

### <a name="configure-dynamic-connection-strings"></a>동적 연결 문자열 구성

웹 앱의 각 인스턴스는 다른 방법을 사용 하 여 SQL database에 연결 합니다. Azure의 앱은 SQL Server VM의 개인 IP 주소를 사용 하 고 Azure Stack 허브의 앱은 SQL Server VM의 공용 IP 주소를 사용 합니다.

> [!Note]  
> Azure Stack 허브 통합 시스템에서 공용 IP 주소는 인터넷 라우팅할 수 없습니다. ASDK에서는 공용 IP 주소를 ASDK 외부에서 라우팅할 경로를 설정 하지 않습니다.

App Service 환경 변수를 사용 하 여 앱의 각 인스턴스에 다른 연결 문자열을 전달할 수 있습니다.

1. Visual Studio에서 앱을 엽니다.

2. Startup.cs를 열고 다음 코드 블록을 찾습니다.

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. 위의 코드 블록을 파일 *의appsettings.js* 에 정의 된 연결 문자열을 사용 하는 다음 코드로 바꿉니다.

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a>앱 설정 App Service 구성

1. Azure 및 Azure Stack Hub에 대 한 연결 문자열을 만듭니다. 사용 되는 IP 주소를 제외 하 고 문자열은 동일 해야 합니다.

2. Azure 및 Azure Stack 허브에서 이름에 접두사로를 사용 하 여 웹 앱의 [앱 설정으로](https://docs.microsoft.com/azure/app-service/web-sites-configure) 적절 한 연결 문자열을 추가 합니다 `SQLCONNSTR\_` .

3. 웹 앱 설정을 **저장** 하 고 앱을 다시 시작 합니다.

## <a name="enable-automatic-scaling-in-global-azure"></a>글로벌 Azure에서 자동 크기 조정 사용

App Service 환경에서 웹 앱을 만들 때 하나의 인스턴스로 시작 합니다. 앱에 더 많은 계산 리소스를 제공 하기 위해 인스턴스를 추가 하도록 자동으로 확장할 수 있습니다. 마찬가지로, 자동으로 규모를 확장 하 고 앱에 필요한 인스턴스 수를 줄일 수 있습니다.

> [!Note]  
> 규모 확장 및 규모 확장을 구성 하려면 App Service 계획이 필요 합니다. 계획이 없는 경우 다음 단계를 시작 하기 전에 계획을 만듭니다.

### <a name="enable-automatic-scale-out"></a>자동 확장 사용

1. Azure에서 규모 확장 하려는 사이트에 대 한 App Service 계획을 찾은 다음 **확장 (App Service 계획)** 을 선택 합니다.

    ![규모 확장 Azure App Service](media/solution-deployment-guide-hybrid/image16.png)

2. **자동 크기 조정 사용**을 선택 합니다.

    ![Azure App Service에서 자동 크기 조정 사용](media/solution-deployment-guide-hybrid/image17.png)

3. **자동 크기 조정 설정 이름**에 이름을 입력 합니다. **기본** 자동 크기 조정 규칙의 경우 **메트릭에 따라 크기 조정**을 선택 합니다. **인스턴스 제한을** **최소: 1**, **최대값: 10**및 **기본값: 1**로 설정 합니다.

    ![Azure App Service에서 자동 크기 조정 구성](media/solution-deployment-guide-hybrid/image18.png)

4. **+ 규칙 추가를**선택 합니다.

5. **메트릭 원본**에서 **현재 리소스**를 선택 합니다. 규칙에 대해 다음 조건 및 동작을 사용 합니다.

#### <a name="criteria"></a>조건

1. **시간 집계** 에서 **평균**을 선택 합니다.

2. **메트릭 이름**아래에서 **CPU 비율**을 선택 합니다.

3. **연산자**아래에서 **보다 큼**을 선택 합니다.

   - **임계값** 을 **50**으로 설정 합니다.
   - **기간** 을 **10**으로 설정 합니다.

#### <a name="action"></a>작업

1. **작업**아래에서 **개수 증가를**선택 합니다.

2. **인스턴스 수** 를 **2**로 설정 합니다.

3. **쿨** 를 **5**로 설정 합니다.

4. **추가**를 선택합니다.

5. **+ 규칙 추가**를 선택 합니다.

6. **메트릭 원본**에서 **현재 리소스를 선택 합니다.**

   > [!Note]  
   > 현재 리소스에 App Service 계획의 이름/GUID가 포함 되 고 **리소스 종류** 및 **리소스** 드롭다운 목록을 사용할 수 없습니다.

### <a name="enable-automatic-scale-in"></a>자동 크기 조정 사용

트래픽이 줄어들면 Azure 웹 앱에서 자동으로 활성 인스턴스 수를 줄여 비용을 절감할 수 있습니다. 이 작업은 확장 보다 낮은 수준 이며 앱 사용자에 대 한 영향을 최소화 합니다.

1. **기본** scale out 조건으로 이동한 다음 **+ 규칙 추가**를 선택 합니다. 규칙에 대해 다음 조건 및 동작을 사용 합니다.

#### <a name="criteria"></a>조건

1. **시간 집계** 에서 **평균**을 선택 합니다.

2. **메트릭 이름**아래에서 **CPU 비율**을 선택 합니다.

3. **연산자**아래에서 **보다 작음**을 선택 합니다.

   - **임계값** 을 **30**으로 설정 합니다.
   - **기간** 을 **10**으로 설정 합니다.

#### <a name="action"></a>작업

1. **작업**아래에서 **개수 줄이기를**선택 합니다.

   - **인스턴스 수** 를 **1**로 설정 합니다.
   - **쿨** 를 **5**로 설정 합니다.

2. **추가**를 선택합니다.

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a>Traffic Manager 프로필 만들기 및 클라우드 간 배율 구성

Azure에서 Traffic Manager 프로필을 만든 다음 클라우드를 구성 하 여 클라우드 간 크기 조정을 사용 하도록 설정 합니다.

### <a name="create-traffic-manager-profile"></a>Traffic Manager 프로필 만들기

1. **리소스 만들기**를 선택합니다.
2. **네트워킹**을 선택 합니다.
3. **Traffic Manager 프로필** 을 선택 하 고 다음 설정을 구성 합니다.

   - **이름**에 프로필의 이름을 입력 합니다. 이 이름은 trafficmanager.net 영역에서 고유 **해야** 하며 새 DNS 이름 (예: northwindstore.trafficmanager.net)을 만드는 데 사용 됩니다.
   - **라우팅 방법**의 경우 **가중치**를 선택 합니다.
   - **구독**에 대해이 프로필을 만들려는 구독을 선택 합니다.
   - **리소스 그룹**에서이 프로필에 대 한 새 리소스 그룹을 만듭니다.
   - **리소스 그룹 위치**에서 리소스 그룹의 위치를 선택합니다. 이 설정은 리소스 그룹의 위치를 나타내며 전역적으로 배포 된 Traffic Manager 프로필에는 영향을 주지 않습니다.

4. **만들기**를 선택합니다.

    ![Traffic Manager 프로필 만들기](media/solution-deployment-guide-hybrid/image19.png)

   Traffic Manager 프로필의 전역 배포가 완료 되 면 해당 프로필을 만든 리소스 그룹의 리소스 목록에 표시 됩니다.

### <a name="add-traffic-manager-endpoints"></a>Traffic Manager 엔드포인트 추가

1. 만든 Traffic Manager 프로필을 검색 합니다. 프로필에 대 한 리소스 그룹을 탐색 한 경우 프로필을 선택 합니다.

2. **Traffic Manager 프로필**의 **설정**에서 **끝점**을 선택 합니다.

3. **추가**를 선택합니다.

4. **끝점 추가**에서 Azure Stack 허브에 대해 다음 설정을 사용 합니다.

   - **형식**에서 **외부 끝점**을 선택 합니다.
   - 끝점의 **이름을** 입력 합니다.
   - **FQDN (정규화 된 도메인 이름) 또는 IP**의 경우 Azure Stack 허브 웹 앱에 대 한 외부 URL을 입력 합니다.
   - **가중치**의 경우 기본값 **1**을 유지 합니다. 이 가중치를 통해 정상 상태 이면 모든 트래픽이이 끝점으로 이동 합니다.
   - **추가 사용 안 함을** 선택 하지 않은 상태로 둡니다.

5. **확인** 을 선택 하 여 Azure Stack 허브 끝점을 저장 합니다.

다음으로 Azure 엔드포인트를 구성 합니다.

1. **Traffic Manager 프로필**에서 **끝점**을 선택 합니다.
2. **+ 추가**를 선택 합니다.
3. **끝점 추가**에서 Azure에 대해 다음 설정을 사용 합니다.

   - **유형**에 대해 **Azure 엔드포인트**를 선택 합니다.
   - 끝점의 **이름을** 입력 합니다.
   - **대상 리소스 종류**에 대해 **App Service**를 선택 합니다.
   - **대상 리소스**에 대해 **앱 서비스 선택** 을 선택 하 여 동일한 구독의 Web Apps 목록을 표시 합니다.
   - **리소스**에서 첫 번째 엔드포인트로 추가할 앱 서비스를 선택합니다.
   - **가중치**에 대해 **2**를 선택 합니다. 이렇게 설정 하면 기본 끝점이 비정상 이거나 트리거된 경우 트래픽을 리디렉션하는 규칙/경고가 있는 경우 모든 트래픽이이 끝점으로 이동 합니다.
   - **추가 사용 안 함을** 선택 하지 않은 상태로 둡니다.

4. **확인** 을 선택 하 여 Azure 끝점을 저장 합니다.

두 끝점을 모두 구성한 후에는 **끝점**을 선택할 때 **Traffic Manager 프로필** 에 나열 됩니다. 다음 화면 캡처의 예제에서는 각각에 대 한 상태 및 구성 정보를 포함 하는 두 개의 끝점을 보여 줍니다.

![Traffic Manager 프로필의 끝점](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting"></a>Application Insights 모니터링 및 경고 설정

Azure 애플리케이션 Insights를 사용 하 여 앱을 모니터링 하 고 구성 하는 조건에 따라 경고를 보낼 수 있습니다. 응용 프로그램을 사용할 수 없거나 오류가 발생 하거나 성능 문제를 보여 주는 몇 가지 예가 있습니다.

Application Insights 메트릭을 사용 하 여 경고를 만듭니다. 이러한 경고가 트리거될 때 웹 앱의 인스턴스는 자동으로 Azure Stack 허브에서 Azure로 전환 하 여 규모를 확장 한 후 규모 확장을 위해 Azure Stack 허브로 돌아갑니다.

### <a name="create-an-alert-from-metrics"></a>메트릭에서 경고 만들기

이 자습서의 리소스 그룹으로 이동한 다음 Application Insights 인스턴스를 선택 하 여 **Application Insights**를 엽니다.

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

이 보기를 사용 하 여 스케일 아웃 경고 및 규모 확장 경고를 만들 수 있습니다.

### <a name="create-the-scale-out-alert"></a>스케일 아웃 경고 만들기

1. **구성**아래에서 **경고 (클래식)** 를 선택 합니다.
2. **메트릭 경고 추가(클래식)** 를 선택합니다.
3. **규칙 추가**에서 다음 설정을 구성 합니다.

   - **이름**에 대해 **Azure 클라우드에 버스트**를 입력 합니다.
   - **설명은** 선택 사항입니다.
   - **원본**  >  **경고 켜기**에서 **메트릭**을 선택 합니다.
   - **조건**에서 구독, Traffic Manager 프로필에 대 한 리소스 그룹 및 리소스에 대 한 Traffic Manager 프로필의 이름을 선택 합니다.

4. **메트릭에**대해 **요청 빈도**를 선택 합니다.
5. **조건**에 대해 **보다 큼**을 선택 합니다.
6. **임계값**에 **2**를 입력 합니다.
7. **기간**에서 **지난 5 분**동안을 선택 합니다.
8. 다음을 **통해 알림**:
   - **전자 메일 소유자, 참가자 및 읽기 권한자**의 확인란을 선택 합니다.
   - **추가 관리자 전자 메일**에 대 한 전자 메일 주소를 입력 하세요.

9. 메뉴 모음에서 **저장**을 선택 합니다.

### <a name="create-the-scale-in-alert"></a>규모 확장 경고 만들기

1. **구성**아래에서 **경고 (클래식)** 를 선택 합니다.
2. **메트릭 경고 추가(클래식)** 를 선택합니다.
3. **규칙 추가**에서 다음 설정을 구성 합니다.

   - **이름**에 **Azure Stack 허브로 크기를 다시**입력 합니다.
   - **설명은** 선택 사항입니다.
   - **원본**  >  **경고 켜기**에서 **메트릭**을 선택 합니다.
   - **조건**에서 구독, Traffic Manager 프로필에 대 한 리소스 그룹 및 리소스에 대 한 Traffic Manager 프로필의 이름을 선택 합니다.

4. **메트릭에**대해 **요청 빈도**를 선택 합니다.
5. **조건**에 대해 **미만**을 선택 합니다.
6. **임계값**에 **2**를 입력 합니다.
7. **기간**에서 **지난 5 분**동안을 선택 합니다.
8. 다음을 **통해 알림**:
   - **전자 메일 소유자, 참가자 및 읽기 권한자**의 확인란을 선택 합니다.
   - **추가 관리자 전자 메일**에 대 한 전자 메일 주소를 입력 하세요.

9. 메뉴 모음에서 **저장**을 선택 합니다.

다음 스크린샷에서는 규모 확장 및 규모 확장에 대 한 경고를 보여 줍니다.

   ![Application Insights 경고 (클래식)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a>Azure와 Azure Stack 허브 간에 트래픽 리디렉션

Azure와 Azure Stack Hub 간의 웹 앱 트래픽 수동 또는 자동 전환을 구성할 수 있습니다.

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a>Azure와 Azure Stack Hub 간의 수동 전환 구성

웹 사이트가 구성 된 임계값에 도달 하면 경고를 받게 됩니다. 다음 단계를 사용 하 여 트래픽을 Azure로 수동으로 리디렉션합니다.

1. Azure Portal에서 Traffic Manager 프로필을 선택 합니다.

    ![Azure Portal의 Traffic Manager 끝점](media/solution-deployment-guide-hybrid/image20.png)

2. **엔드포인트**를 선택합니다.
3. **Azure 끝점**을 선택 합니다.
4. **상태**에서 **사용**을 선택한 다음 **저장**을 선택 합니다.

    ![Azure Portal에서 Azure 끝점 사용](media/solution-deployment-guide-hybrid/image23.png)

5. Traffic Manager 프로필에 대 한 **끝점** 에서 **외부 끝점**을 선택 합니다.
6. **상태**에서 **사용 안 함**을 선택한 다음, **저장**을 선택 합니다.

    ![Azure Portal에서 Azure Stack 허브 끝점 사용 안 함](media/solution-deployment-guide-hybrid/image24.png)

끝점이 구성 되 면 앱 트래픽이 Azure Stack 허브 웹 앱 대신 Azure 스케일 아웃 웹 앱으로 이동 합니다.

 ![Azure 웹 앱 트래픽에서 변경 된 끝점](media/solution-deployment-guide-hybrid/image25.png)

흐름을 Azure Stack 허브로 되돌리려면 이전 단계를 사용 하 여 다음을 수행 합니다.

- Azure Stack 허브 끝점을 사용 하도록 설정 합니다.
- Azure 끝점을 사용 하지 않도록 설정 합니다.

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a>Azure와 Azure Stack Hub 간의 자동 전환 구성

Azure Functions에서 제공 하는 [서버](https://azure.microsoft.com/overview/serverless-computing/) 를 사용 하지 않는 환경에서 앱을 실행 하는 경우에도 Application Insights 모니터링을 사용할 수 있습니다.

이 시나리오에서는 함수 앱을 호출 하는 webhook를 사용 하도록 Application Insights를 구성할 수 있습니다. 이 앱은 경고에 대 한 응답으로 끝점을 자동으로 사용 하거나 사용 하지 않도록 설정 합니다.

자동 트래픽 전환을 구성 하려면 다음 단계를 지침으로 사용 합니다.

1. Azure 함수 앱을 만듭니다.
2. HTTP 트리거 함수를 만듭니다.
3. 리소스 관리자, Web Apps 및 Traffic Manager에 대 한 Azure Sdk를 가져옵니다.
4. 다음에 대 한 코드를 개발 합니다.

   - Azure 구독에 인증 합니다.
   - Traffic Manager 끝점을 전환 하는 매개 변수를 사용 하 여 트래픽을 Azure 또는 Azure Stack 허브로 직접 보냅니다.

5. 코드를 저장 하 고 적절 한 매개 변수를 사용 하 여 함수 앱의 URL을 Application Insights 경고 규칙 설정의 **Webhook** 섹션에 추가 합니다.
6. Application Insights 경고가 발생 하면 트래픽이 자동으로 리디렉션됩니다.

## <a name="next-steps"></a>다음 단계

- Azure Cloud 패턴에 대해 자세히 알아보려면 [클라우드 디자인 패턴](https://docs.microsoft.com/azure/architecture/patterns)을 참조 하세요.
