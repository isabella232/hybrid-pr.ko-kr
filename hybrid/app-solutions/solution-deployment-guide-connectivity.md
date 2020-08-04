---
title: Azure 및 Azure Stack Hub에서 하이브리드 클라우드 연결 구성
description: Azure 및 Azure Stack Hub를 사용하여 하이브리드 클라우드 연결을 구성하는 방법을 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 16c5d7820e8c865a9f88cb00da5cc7c854379414
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477289"
---
# <a name="configure-hybrid-cloud-connectivity-using-azure-and-azure-stack-hub"></a>Azure 및 Azure Stack Hub를 사용하여 하이브리드 클라우드 연결 구성

하이브리드 연결 패턴을 사용하면 글로벌 Azure 및 Azure Stack Hub에서 보안을 통해 리소스에 액세스할 수 있습니다.

이 솔루션에서는 다음을 수행하는 샘플 환경을 빌드합니다.

> [!div class="checklist"]
> - 개인 정보 보호 또는 규정 요구 사항을 충족하기 위해 온-프레미스에 데이터를 유지하지만 글로벌 Azure 리소스에 대한 액세스는 유지합니다.
> - 글로벌 Azure에서 클라우드 규모의 앱 배포 및 리소스를 사용하면서 레거시 시스템을 유지 관리합니다.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub는 Azure의 확장입니다. Azure Stack Hub는 온-프레미스 환경에 클라우드 컴퓨팅의 민첩성과 혁신을 제공하여 어디서나 하이브리드 앱을 빌드하고 배포할 수 있는 유일한 하이브리드 클라우드를 사용하도록 설정합니다.  
> 
> [하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 문서는 하이브리드 앱 디자인, 배포 및 운영에 대한 소프트웨어 품질(배치, 확장성, 가용성, 복원력, 관리 효율성 및 보안)의 핵심 요소를 검토합니다. 디자인 고려 사항은 하이브리드 앱 디자인을 최적화하고 프로덕션 환경에서 문제를 최소화하는 데 도움이 됩니다.

## <a name="prerequisites"></a>필수 구성 요소

하이브리드 연결 배포를 빌드하려면 몇 가지 구성 요소가 필요합니다. 이러한 구성 요소 중 일부는 준비하는 데 시간이 걸리므로 적절하게 계획하세요.

### <a name="azure"></a>Azure

- Azure 구독이 아직 없는 경우 시작하기 전에 [체험 계정](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)을 만듭니다.
- Azure에서 [웹앱](/vsts/build-release/apps/cd/azure/aspnet-core-to-azure-webapp?tabs=vsts&view=vsts)을 만듭니다. 웹앱 URL은 적어두세요. 솔루션에서 필요합니다.

### <a name="azure-stack-hub"></a>Azure Stack Hub

Azure OEM/하드웨어 파트너는 프로덕션 Azure Stack Hub를 배포할 수 있으며, 모든 사용자는 ASDK(Azure Stack Development Kit)를 배포할 수 있습니다.

- 프로덕션 Azure Stack Hub를 사용하거나 ASDK를 배포합니다.
   >[!Note]
   >ASDK 배포에는 최대 7시간이 걸릴 수 있으므로 적절하게 계획하세요.

- [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS 서비스를 Azure Stack 허브에 배포합니다.
- Azure Stack Hub 환경에서 [계획 및 제안을 만듭니다](/azure-stack/operator/service-plan-offer-subscription-overview.md).
- Azure Stack Hub 환경 내에 [테넌트 구독을 만듭니다](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md).

### <a name="azure-stack-hub-components"></a>Azure Stack Hub 구성 요소

Azure Stack Hub 운영자는 App Service를 배포하고 계획 및 제안을 만든 다음, 테넌트 구독을 만들고 Windows Server 2016 이미지를 추가해야 합니다. 이러한 구성 요소가 이미 있는 경우 이 솔루션을 시작하기 전에 해당 구성 요소가 요구 사항을 충족하는지 확인합니다.

이 솔루션 예제에서는 Azure 및 Azure Stack Hub에 대한 기본적인 지식이 있다고 가정합니다. 솔루션을 시작하기 전에 자세히 알아보려면 다음 문서를 참조하세요.

- [Azure 소개](https://azure.microsoft.com/overview/what-is-azure/)
- [Azure Stack Hub 주요 개념](/azure-stack/operator/azure-stack-overview.md)

### <a name="before-you-begin"></a>시작하기 전에

하이브리드 클라우드 연결 구성을 시작하기 전에 다음 기준을 충족하는지 확인합니다.

- VPN 디바이스에 대한 외부 연결 공용 IPv4 주소가 필요합니다. 이 IP 주소는 NAT(Network Address Translation) 뒤에 있을 수 없습니다.
- 모든 리소스는 동일한 지역/위치에 배포됩니다.

#### <a name="solution-example-values"></a>솔루션 예제 값

이 솔루션의 예제에서는 다음 값을 사용합니다. 이러한 값을 사용하여 테스트 환경을 만들거나 이 값을 참조하여 예제를 보다 정확하게 이해할 수 있습니다. VPN 게이트웨이 설정에 대한 자세한 내용은 [VPN Gateway 설정 정보](/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings)를 참조하세요.

연결 사양:

- **VPN 유형**: 경로 기반
- **연결 형식:** 사이트 간(IPsec)
- **게이트웨이 유형**: VPN
- **Azure 연결 이름**: Azure-Gateway-AzureStack-S2SGateway(포털에서 이 값이 자동으로 채워짐)
- **Azure Stack Hub 연결 이름**: AzureStack-Gateway-Azure-S2SGateway(포털에서 이 값이 자동으로 채워짐)
- **공유 키**: 연결의 양쪽에서 일치하는 값을 사용하는 VPN 하드웨어와 호환 가능
- **구독**: 선호하는 구독
- **리소스 그룹**: Test-Infra

네트워크 및 서브넷 IP 주소:

| Azure/Azure Stack Hub 연결 | Name | 서브넷 | IP 주소 |
|---|---|---|---|
| Azure vNet | ApplicationvNet<br>10.100.102.9/23 | ApplicationSubnet<br>10.100.102.0/24 |  |
|  |  | GatewaySubnet<br>10.100.103.0/24 |  |
| Azure Stack Hub vNet | ApplicationvNet<br>10.100.100.0/23 | ApplicationSubnet <br>10.100.100.0/24 |  |
|  |  | GatewaySubnet <br>10.100101.0/24 |  |
| Azure Virtual Network 게이트웨이 | Azure-Gateway |  |  |
| Azure Stack Hub Virtual Network 게이트웨이 | AzureStack-Gateway |  |  |
| Azure 공용 IP | Azure-GatewayPublicIP |  | 생성 시 결정됨 |
| Azure Stack Hub 공용 IP | AzureStack-GatewayPublicIP |  | 생성 시 결정됨 |
| Azure 로컬 네트워크 게이트웨이 | AzureStack-S2SGateway<br>   10.100.100.0/23 |  | Azure Stack Hub 공용 IP 값 |
| Azure Stack Hub 로컬 네트워크 게이트웨이 | Azure-S2SGateway<br>10.100.102.0/23 |  | Azure 공용 IP 값 |

## <a name="create-a-virtual-network-in-global-azure-and-azure-stack-hub"></a>글로벌 Azure 및 Azure Stack Hub에서 가상 네트워크 만들기

다음 단계를 사용하여 포털에서 가상 네트워크를 만듭니다. 이 문서를 솔루션으로만 사용하는 경우 [예제 값](/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal#values)을 사용할 수 있습니다. 이 문서를 사용하여 프로덕션 환경을 구성하는 경우에는 예제 설정을 자체 값으로 바꿉니다.

> [!IMPORTANT]
> Azure 또는 Azure Stack Hub vNet 주소 공간에 IP 주소가 겹치지 않도록 해야 합니다.

Azure에서 vNet을 만들려면 다음을 수행합니다.

1. 브라우저를 사용하여 [Azure Portal](https://portal.azure.com/)에 연결하고 Azure 계정을 사용하여 로그인합니다.
2. **리소스 만들기**를 선택합니다. **마켓플레이스 검색** 필드에 ‘virtual network’를 입력합니다. 결과에서 **Virtual network**를 선택합니다.
3. **배포 모델 선택** 목록에서 **Resource Manager**를 선택한 다음, **만들기**를 선택합니다.
4. **가상 네트워크 만들기**에서 VNet 설정을 구성합니다. 필수 필드 이름 앞에는 빨간색 별표가 붙습니다.  유효한 값을 입력하면 별표가 녹색 확인 표시로 바뀝니다.

Azure Stack Hub에서 vNet을 만들려면 다음을 수행합니다.

1. Azure Stack Hub **테넌트 포털**을 사용하여 위의(1-4) 단계를 반복합니다.

## <a name="add-a-gateway-subnet"></a>게이트웨이 서브넷 추가

가상 네트워크를 게이트웨이에 연결하기 전에 먼저 연결하려는 가상 네트워크에 대한 게이트웨이 서브넷을 만들어야 합니다. 게이트웨이 서비스는 게이트웨이 서브넷에 지정한 IP 주소를 사용합니다.

[Azure Portal](https://portal.azure.com/)에서 가상 네트워크 게이트웨이를 만들려는 Resource Manager 가상 네트워크로 이동합니다.

1. vNet을 선택하여 **가상 네트워크** 페이지를 엽니다.
2. **설정**에서 **서브넷**을 선택합니다.
3. **서브넷** 페이지에서 **+게이트웨이 서브넷**을 선택하여 **서브넷 추가** 페이지를 엽니다.

    ![게이트웨이 서브넷 추가](media/solution-deployment-guide-connectivity/image4.png)

4. 서브넷의 **이름**에 'GatewaySubnet' 값이 자동으로 채워집니다. Azure가 서브넷을 게이트웨이 서브넷으로 인식하기 위해 이 값이 필요합니다.
5. 구성 요구 사항과 일치하도록 제공된 **주소 범위** 값을 변경한 다음, **확인**을 선택합니다.

## <a name="create-a-virtual-network-gateway-in-azure-and-azure-stack"></a>Azure 및 Azure Stack에서 가상 네트워크 게이트웨이 만들기

다음 단계를 사용하여 Azure에서 가상 네트워크 게이트웨이를 만듭니다.

1. 포털 페이지의 왼쪽에서 **+** 를 선택하고 검색 필드에 '가상 네트워크 게이트웨이'를 입력합니다.
2. **결과**에서 **가상 네트워크 게이트웨이**를 선택합니다.
3. **가상 네트워크 게이트웨이**에서 **만들기**를 선택하여 **가상 네트워크 게이트웨이 만들기** 페이지를 엽니다.
4. **가상 네트워크 게이트웨이 만들기**에서 **자습서 예제 값**을 사용하여 네트워크 게이트웨이의 값을 지정합니다. 다음과 같은 추가 값을 포함합니다.

   - **SKU**: 기본
   - **Virtual Network**: 이전에 만든 가상 네트워크를 선택합니다. 만든 게이트웨이 서브넷이 자동으로 선택됩니다.
   - **첫 IP 구성**:  게이트웨이의 공용 IP입니다.
     - **게이트웨이 IP 구성 생성**을 선택하면 **공용 IP 주소 선택** 페이지로 이동합니다.
     - **+새로 만들기**를 선택하여 **공용 IP 주소 만들기** 페이지를 엽니다.
     - 공용 IP 주소의 **이름**을 입력합니다. SKU를 **기본**으로 유지한 다음, **확인**을 선택하여 변경 내용을 저장합니다.

       > [!Note]
       > 현재 VPN Gateway는 동적 공용 IP 주소 할당만 지원합니다. 하지만 IP 주소가 VPN 게이트웨이에 할당된 후에 변경되는 것은 아닙니다. 게이트웨이를 삭제하고 다시 만드는 경우에만 공용 IP 주소가 변경됩니다. VPN 게이트웨이에 대해 크기 조정, 재설정 또는 기타 내부 유지 관리/업그레이드를 수행해도 IP 주소는 변경되지 않습니다.

5. 게이트웨이 설정을 확인합니다.
6. **만들기**를 선택하여 VPN 게이트웨이를 만듭니다. 게이트웨이 설정의 유효성이 검사되고 "가상 네트워크 게이트웨이 배포" 타일이 대시보드에 표시됩니다.

   >[!Note]
   >하나의 게이트웨이를 만드는 데 최대 45분이 걸릴 수 있습니다. 완료 상태를 확인하기 위해 포털 페이지를 새로 고쳐야 할 수 있습니다.

    게이트웨이가 생성된 후 포털에서 가상 네트워크를 살펴보면 게이트웨이에 할당된 IP 주소를 볼 수 있습니다. 게이트웨이가 연결된 디바이스로 표시됩니다. 게이트웨이에 대한 자세한 정보를 보려면 디바이스를 선택합니다.

7. Azure Stack Hub 배포에서 이전 단계(1-5)를 반복합니다.

## <a name="create-the-local-network-gateway-in-azure-and-azure-stack-hub"></a>Azure 및 Azure Stack Hub에서 로컬 네트워크 게이트웨이 만들기

로컬 네트워크 게이트웨이는 일반적으로 온-프레미스 위치를 가리킵니다. Azure 또는 Azure Stack 허브가 참조할 수 있는 이름을 사이트에 지정하고 다음을 지정합니다.

- 연결하려는 온-프레미스 VPN 디바이스의 IP 주소
- VPN 게이트웨이를 통해 VPN 디바이스로 라우팅될 IP 주소 접두사. 사용자가 지정하는 주소 접두사는 온-프레미스 네트워크에 있는 접두사입니다.

  >[!Note]
  >온-프레미스 네트워크가 변경되거나 VPN 디바이스의 공용 IP 주소를 변경해야 하는 경우 나중에 값을 업데이트할 수 있습니다.

1. 포털에서 **+리소스 만들기**를 선택합니다.
2. 검색 상자에 **로컬 네트워크 게이트웨이**를 입력한 다음, **Enter**를 선택하여 검색합니다. 결과 목록이 표시됩니다.
3. **로컬 네트워크 게이트웨이**를 선택한 다음, **만들기**를 선택하여 **로컬 네트워크 게이트웨이 만들기** 페이지를 엽니다.
4. **로컬 네트워크 게이트웨이 만들기**에서 **자습서 예제 값**을 사용하여 로컬 네트워크 게이트웨이의 값을 지정합니다. 다음과 같은 추가 값을 포함합니다.

    - **IP 주소**: Azure 또는 Azure Stack Hub에 연결하려는 VPN 디바이스의 공용 IP 주소입니다. NAT 뒤에 있지 않은 유효한 공용 IP 주소를 지정합니다. 그래야 Azure가 해당 주소에 도달할 수 있습니다. 지금 IP 주소가 없으면 예제의 값을 자리 표시자로 사용할 수 있습니다. 뒤로 돌아가서 자리 표시자를 VPN 디바이스의 공용 IP 주소로 바꿔야 합니다. 유효한 주소를 제공할 때까지는 Azure를 디바이스에 연결할 수 없습니다.
    - **주소 공간**: 이 로컬 네트워크가 나타내는 네트워크의 주소 범위입니다. 주소 공간 범위를 여러 개 추가할 수 있습니다. 지정한 범위가 연결하려는 다른 네트워크의 범위와 겹치지 않아야 합니다. Azure는 지정한 주소 범위를 온-프레미스 VPN 디바이스 IP 주소에 라우팅합니다. 온-프레미스 사이트에 연결하려면 예제 값이 아닌 자체 값을 사용하십시오.
    - **BGP 설정 구성**: BGP를 구성할 경우에만 사용합니다. 그렇지 않으면 이 옵션을 선택하지 마십시오.
    - **구독**: 올바른 구독이 표시되는지 확인합니다.
    - **리소스 그룹**: 사용하려는 리소스 그룹을 선택합니다. 새 리소스 그룹을 만들거나 이미 만든 리소스 그룹을 선택할 수 있습니다.
    - **위치**: 이 개체를 만들 위치를 선택합니다. VNet이 있는 동일한 위치를 선택하는 것이 좋지만 반드시 그럴 필요는 없습니다.
5. 필요한 값을 모두 지정했으면 **만들기**를 선택하여 로컬 네트워크 게이트웨이를 만듭니다.
6. Azure Stack Hub 배포에서 해당 단계(1-5)를 반복합니다.

## <a name="configure-your-connection"></a>연결 구성

온-프레미스 네트워크에 대한 사이트 간 연결에는 VPN 디바이스가 필요합니다. 구성하는 VPN 디바이스를 연결이라고 합니다. 연결을 구성하려면 다음이 필요합니다.

- 공유 키 - 이 키는 사이트 간 VPN 연결을 만들 때 지정하는 것과 동일한 공유 키입니다. 이 예제에서는 기본적인 공유 키를 사용합니다. 실제로 사용할 키는 좀 더 복잡하게 생성하는 것이 좋습니다.
- 가상 네트워크 게이트웨이의 공용 IP 주소입니다. Azure Portal, PowerShell 또는 CLI를 사용하여 공용 IP 주소를 볼 수 있습니다. Azure Portal을 사용하여 VPN 게이트웨이의 공용 IP 주소를 찾으려면 가상 네트워크 게이트웨이로 이동한 다음, 게이트웨이의 이름을 선택합니다.

다음 단계를 사용하여 가상 네트워크 게이트웨이와 온-프레미스 VPN 디바이스 사이에 사이트 간 VPN 연결을 만듭니다.

1. Azure Portal에서 **+리소스 만들기**를 선택합니다.
2. **연결**을 검색합니다.
3. **결과**에서 **연결**을 선택합니다.
4. **연결**에서 **만들기**를 선택합니다.
5. **연결 만들기**에서 다음 설정을 구성합니다.

    - **연결 형식**: 사이트 간(IPSec)을 선택합니다.
    - **리소스 그룹**: 테스트 리소스 그룹을 선택합니다.
    - **가상 네트워크 게이트웨이**: 직접 만든 가상 네트워크 게이트웨이를 선택합니다.
    - **로컬 네트워크 게이트웨이**: 직접 만든 로컬 네트워크 게이트웨이를 선택합니다.
    - **연결 이름**: 이 이름은 두 게이트웨이의 값을 사용하여 자동으로 채워집니다.
    - **공유 키**: 이 값은 로컬 온-프레미스 VPN 디바이스에 사용하는 값과 일치해야 합니다. 자습서 예제에서는 'abc123'을 사용하지만 더 복잡한 값을 사용해야 합니다. 중요한 점은 이 값이 VPN 디바이스를 구성할 때 지정한 값과 *동일해야 한다*는 것입니다.
    - **구독**, **리소스 그룹** 및 **위치**에 대한 값이 고정됩니다.

6. **확인**을 선택하여 연결을 만듭니다.

가상 네트워크 게이트웨이의 **연결** 페이지에서 연결을 볼 수 있습니다. 상태는 알 수 없음에서 연결 중으로 바뀌고 성공으로 바뀝니다.  

## <a name="next-steps"></a>다음 단계

- Azure Cloud 패턴에 대해 자세히 알아보려면 [클라우드 디자인 패턴](/azure/architecture/patterns)을 참조하세요.
