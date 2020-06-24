---
title: Azure 및 Azure Stack Hub에서 하이브리드 클라우드 연결 구성
description: Azure 및 Azure Stack Hub를 사용 하 여 하이브리드 클라우드 연결을 구성 하는 방법을 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 0e1a0fc4fb4110fdb406d4b4b2e72abb8f5412c9
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910868"
---
# <a name="configure-hybrid-cloud-connectivity-using-azure-and-azure-stack-hub"></a>Azure 및 Azure Stack Hub를 사용 하 여 하이브리드 클라우드 연결 구성

하이브리드 연결 패턴을 사용 하 여 글로벌 Azure 및 Azure Stack 허브의 보안을 사용 하 여 리소스에 액세스할 수 있습니다.

이 솔루션에서는 다음을 수행 하는 샘플 환경을 빌드합니다.

> [!div class="checklist"]
> - 데이터를 온-프레미스로 유지 하 여 개인 정보나 규정 요구 사항을 충족 하지만 글로벌 Azure 리소스에 대 한 액세스를 유지 합니다.
> - 글로벌 Azure에서 클라우드 크기 조정 된 앱 배포 및 리소스를 사용 하는 동안 레거시 시스템을 유지 관리 합니다.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack 허브는 Azure의 확장입니다. Azure Stack 허브는 온-프레미스 환경에 클라우드 컴퓨팅의 민첩성과 혁신을 제공 하 여 어디서 나 하이브리드 앱을 빌드하고 배포할 수 있는 유일한 하이브리드 클라우드를 사용 하도록 설정 합니다.  
> 
> [하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 문서에서는 하이브리드 앱을 디자인, 배포 및 운영 하기 위한 소프트웨어 품질 (배치, 확장성, 가용성, 복원 력, 관리 효율성 및 보안)의 핵심 요소을 검토 합니다. 디자인 고려 사항은 하이브리드 앱 디자인을 최적화 하 고 프로덕션 환경에서 문제를 최소화 하는 데 도움이 됩니다.

## <a name="prerequisites"></a>전제 조건

하이브리드 연결 배포를 빌드하려면 몇 가지 구성 요소가 필요 합니다. 이러한 구성 요소 중 일부는 준비 하는 데 시간이 걸리므로 적절 하 게 계획 합니다.

### <a name="azure"></a>Azure

- Azure 구독이 아직 없는 경우 시작하기 전에 [체험 계정](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)을 만듭니다.
- Azure에서 [웹 앱](https://docs.microsoft.com/vsts/build-release/apps/cd/azure/aspnet-core-to-azure-webapp?view=vsts&tabs=vsts) 을 만듭니다. 솔루션에 필요 하므로 웹 앱 URL을 기록해 둡니다.

### <a name="azure-stack-hub"></a>Azure Stack Hub

Azure OEM/하드웨어 파트너는 프로덕션 Azure Stack 허브를 배포할 수 있으며, 모든 사용자가 ASDK (Azure Stack Development Kit)를 배포할 수 있습니다.

- 프로덕션 Azure Stack 허브를 사용 하거나 ASDK을 배포 합니다.
   >[!Note]
   >ASDK 배포에는 최대 7 시간이 걸릴 수 있으므로 적절 하 게 계획 합니다.

- Azure Stack 허브에 [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS 서비스를 배포 합니다.
- Azure Stack 허브 환경에서 [계획 및 제품을 만듭니다](/azure-stack/operator/service-plan-offer-subscription-overview.md) .
- Azure Stack 허브 환경 내에서 [테 넌 트 구독을 만듭니다](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) .

### <a name="azure-stack-hub-components"></a>Azure Stack 허브 구성 요소

Azure Stack 허브 운영자는 App Service를 배포 하 고 계획 및 제안을 만든 다음 테 넌 트 구독을 만들고 Windows Server 2016 이미지를 추가 해야 합니다. 이러한 구성 요소가 이미 있는 경우이 솔루션을 시작 하기 전에 요구 사항을 충족 하는지 확인 합니다.

이 솔루션 예제에서는 Azure 및 Azure Stack Hub에 대 한 기본적인 지식이 있다고 가정 합니다. 솔루션을 시작 하기 전에 자세히 알아보려면 다음 문서를 참조 하세요.

- [Azure 소개](https://azure.microsoft.com/overview/what-is-azure/)
- [Azure Stack 허브 주요 개념](/azure-stack/operator/azure-stack-overview.md)

### <a name="before-you-begin"></a>시작하기 전에

하이브리드 클라우드 연결 구성을 시작 하기 전에 다음 기준을 충족 하는지 확인 합니다.

- VPN 장치에 대 한 외부 연결 공용 IPv4 주소가 필요 합니다. 이 IP 주소는 NAT 뒤에 있을 수 없습니다 (네트워크 주소 변환).
- 모든 리소스는 동일한 지역/위치에 배포 됩니다.

#### <a name="solution-example-values"></a>솔루션 예제 값

이 솔루션의 예제에서는 다음 값을 사용 합니다. 이러한 값을 사용 하 여 테스트 환경을 만들거나이를 참조 하 여 예제를 보다 잘 이해할 수 있습니다. VPN gateway 설정에 대 한 자세한 내용은 [VPN Gateway 설정 정보](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings)를 참조 하세요.

연결 사양:

- **VPN 유형**: 경로 기반
- **연결 형식**: 사이트 간 (IPsec)
- **게이트웨이 유형**: VPN
- **Azure 연결 이름**: Azure-S2SGateway (포털에서이 값을 자동으로 채우기)
- **Azure Stack 허브 연결 이름**: Azurestack-S2SGateway (포털에서이 값을 자동으로 채우기)
- **공유 키**: 연결의 양쪽에서 일치 하는 값을 사용 하 여 VPN 하드웨어와 호환 가능
- **구독**: 모든 기본 설정 구독
- **리소스 그룹**: 인프라

네트워크 및 서브넷 IP 주소:

| Azure/Azure Stack 허브 연결 | Name | 서브넷 | IP 주소 |
|---|---|---|---|
| Azure vNet | ApplicationvNet<br>10.100.102.9/23 | ApplicationSubnet<br>10.100.102.0/24 |  |
|  |  | GatewaySubnet<br>10.100.103.0/24 |  |
| Azure Stack 허브 vNet | ApplicationvNet<br>10.100.100.0/23 | ApplicationSubnet <br>10.100.100.0/24 |  |
|  |  | GatewaySubnet <br>10.100101.0/24 |  |
| Azure Virtual Network 게이트웨이 | Azure-게이트웨이 |  |  |
| Azure Stack 허브 Virtual Network 게이트웨이 | AzureStack-게이트웨이 |  |  |
| Azure 공용 IP | Azure-GatewayPublicIP |  | 생성 시 결정 됨 |
| Azure Stack 허브 공용 IP | AzureStack-GatewayPublicIP |  | 생성 시 결정 됨 |
| Azure 로컬 네트워크 게이트웨이 | AzureStack-S2SGateway<br>   10.100.100.0/23 |  | Azure Stack 허브 공용 IP 값 |
| Azure Stack 허브 로컬 네트워크 게이트웨이 | Azure-S2SGateway<br>10.100.102.0/23 |  | Azure 공용 IP 값 |

## <a name="create-a-virtual-network-in-global-azure-and-azure-stack-hub"></a>글로벌 Azure 및 Azure Stack 허브에서 가상 네트워크 만들기

포털을 사용 하 여 가상 네트워크를 만들려면 다음 단계를 수행 합니다. 이 문서를 솔루션 으로만 사용 하는 경우 다음 [예제 값](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal#values) 을 사용할 수 있습니다. 이 문서를 사용 하 여 프로덕션 환경을 구성 하는 경우 예제 설정을 사용자 고유의 값으로 바꿉니다.

> [!IMPORTANT]
> Azure의 IP 주소 또는 Azure Stack 허브 vNet 주소 공간이 중복 되지 않았는지 확인 해야 합니다.

Azure에서 vNet을 만들려면 다음을 수행 합니다.

1. 브라우저를 사용 하 여 [Azure Portal](https://portal.azure.com/) 에 연결 하 고 Azure 계정으로 로그인 합니다.
2. **리소스 만들기**를 선택합니다. **Marketplace 검색** 필드에 ' 가상 네트워크 '를 입력 합니다. 결과에서 **가상 네트워크** 를 선택 합니다.
3. **배포 모델 선택** 목록에서 **리소스 관리자**을 선택 하 고 **만들기**를 선택 합니다.
4. **가상 네트워크 만들기**에서 VNet 설정을 구성 합니다. 필수 필드 이름에는 빨간색 별표 접두사가 붙습니다.  유효한 값을 입력 하면 별표가 녹색 확인 표시로 바뀝니다.

Azure Stack 허브에서 vNet을 만들려면 다음을 수행 합니다.

1. Azure Stack Hub **테 넌 트 포털**을 사용 하 여 위의 단계 (1-4)를 반복 합니다.

## <a name="add-a-gateway-subnet"></a>게이트웨이 서브넷 추가

가상 네트워크를 게이트웨이에 연결 하기 전에 연결 하려는 가상 네트워크에 대 한 게이트웨이 서브넷을 만들어야 합니다. 게이트웨이 서비스는 게이트웨이 서브넷에서 지정 하는 IP 주소를 사용 합니다.

[Azure Portal](https://portal.azure.com/)에서 가상 네트워크 게이트웨이를 만들려는 리소스 관리자 가상 네트워크로 이동 합니다.

1. VNet을 선택 하 여 **가상 네트워크** 페이지를 엽니다.
2. **설정**에서 **서브넷**을 선택 합니다.
3. **서브넷** 페이지에서 **+ 게이트웨이 서브넷** 을 선택 하 여 **서브넷 추가** 페이지를 엽니다.

    ![게이트웨이 서브넷 추가](media/solution-deployment-guide-connectivity/image4.png)

4. 서브넷의 **이름** 에는 ' gsubnet ' 값이 자동으로 입력 됩니다. Azure가 서브넷을 게이트웨이 서브넷으로 인식하기 위해 이 값이 필요합니다.
5. 구성 요구 사항과 일치 하도록 제공 된 **주소 범위** 값을 변경 하 고 **확인**을 선택 합니다.

## <a name="create-a-virtual-network-gateway-in-azure-and-azure-stack"></a>Azure에서 Virtual Network 게이트웨이 만들기 및 Azure Stack

Azure에서 가상 네트워크 게이트웨이를 만들려면 다음 단계를 사용 합니다.

1. 포털 페이지의 왼쪽에서를 선택 **+** 하 고 검색 필드에 ' 가상 네트워크 게이트웨이 '를 입력 합니다.
2. **결과**에서 **가상 네트워크 게이트웨이**를 선택 합니다.
3. **가상 네트워크 게이트웨이**에서 **만들기** 를 선택 하 여 **가상 네트워크 게이트웨이 만들기** 페이지를 엽니다.
4. **가상 네트워크 게이트웨이 만들기**에서 **자습서 예제 값**을 사용 하 여 네트워크 게이트웨이의 값을 지정 합니다. 다음과 같은 추가 값을 포함 합니다.

   - **SKU**: 기본
   - **Virtual Network**: 이전에 만든 가상 네트워크를 선택 합니다. 만든 게이트웨이 서브넷이 자동으로 선택 됩니다.
   - **첫 번째 Ip 구성**: 게이트웨이의 공용 ip입니다.
     - **게이트웨이 ip 구성 만들기**를 선택 하 여 **공용 ip 주소 선택** 페이지로 이동 합니다.
     - **+ 새로 만들기** 를 선택 하 여 **공용 IP 주소 만들기** 페이지를 엽니다.
     - 공용 IP 주소의 **이름을** 입력 합니다. SKU를 **기본**으로 그대로 두고 **확인** 을 선택 하 여 변경 내용을 저장 합니다.

       > [!Note]
       > 현재 VPN Gateway는 동적 공용 IP 주소 할당만 지원 합니다. 그러나 VPN gateway에 할당 된 후에는 IP 주소가 변경 되는 것을 의미 하지 않습니다. 게이트웨이를 삭제하고 다시 만드는 경우에만 공용 IP 주소가 변경됩니다. VPN gateway에 대 한 크기 조정, 다시 설정 또는 기타 내부 유지 관리/업그레이드는 IP 주소를 변경 하지 않습니다.

5. 게이트웨이 설정을 확인 합니다.
6. **만들기** 를 선택 하 여 VPN gateway를 만듭니다. 게이트웨이 설정의 유효성이 검사 되 고 "가상 네트워크 게이트웨이 배포" 타일이 대시보드에 표시 됩니다.

   >[!Note]
   >하나의 게이트웨이를 만드는 데 최대 45분이 걸릴 수 있습니다. 완료 상태를 확인하기 위해 포털 페이지를 새로 고쳐야 할 수 있습니다.

    게이트웨이를 만든 후 포털에서 가상 네트워크를 살펴보면 할당 된 IP 주소를 볼 수 있습니다. 게이트웨이가 연결된 디바이스로 표시됩니다. 게이트웨이에 대 한 자세한 정보를 보려면 장치를 선택 합니다.

7. Azure Stack Hub 배포에서 이전 단계 (1-5)를 반복 합니다.

## <a name="create-the-local-network-gateway-in-azure-and-azure-stack-hub"></a>Azure에서 로컬 네트워크 게이트웨이 만들기 및 Azure Stack 허브

로컬 네트워크 게이트웨이는 일반적으로 온-프레미스 위치를 가리킵니다. Azure 또는 Azure Stack 허브가 참조할 수 있는 이름을 사이트에 지정 하 고 다음을 지정 합니다.

- 연결을 만드는 온-프레미스 VPN 장치의 IP 주소입니다.
- Vpn gateway를 통해 VPN 장치로 라우팅되는 IP 주소 접두사입니다. 사용자가 지정하는 주소 접두사는 온-프레미스 네트워크에 있는 접두사입니다.

  >[!Note]
  >온-프레미스 네트워크가 변경 되거나 VPN 장치에 대 한 공용 IP 주소를 변경 해야 하는 경우 나중에 이러한 값을 업데이트할 수 있습니다.

1. 포털에서 **+ 리소스 만들기**를 선택 합니다.
2. 검색 상자에서 **로컬 네트워크 게이트웨이**를 입력 한 다음 **enter 키** 를 선택 하 여 검색 합니다. 결과 목록이 표시 됩니다.
3. **로컬 네트워크 게이트웨이**를 선택 하 고 **만들기** 를 선택 하 여 **로컬 네트워크 게이트웨이 만들기** 페이지를 엽니다.
4. **로컬 네트워크 게이트웨이 만들기**에서 **자습서 예제 값**을 사용 하 여 로컬 네트워크 게이트웨이의 값을 지정 합니다. 다음과 같은 추가 값을 포함 합니다.

    - **IP 주소**: Azure 또는 Azure Stack 허브에 연결 하려는 VPN 장치의 공용 IP 주소입니다. Azure가 주소에 도달할 수 있도록 NAT 뒤에 있지 않은 유효한 공용 IP 주소를 지정 합니다. 현재 IP 주소가 없으면 예제의 값을 자리 표시자로 사용할 수 있습니다. 이전 단계로 이동 하 여 자리 표시자를 VPN 장치의 공용 IP 주소로 바꾸어야 합니다. 올바른 주소를 제공할 때까지 Azure에서 장치에 연결할 수 없습니다.
    - **주소 공간**:이 로컬 네트워크가 나타내는 네트워크의 주소 범위입니다. 주소 공간 범위를 여러 개 추가할 수 있습니다. 지정 하는 범위가 연결 하려는 다른 네트워크의 범위와 겹치지 않는지 확인 합니다. Azure는 지정한 주소 범위를 온-프레미스 VPN 디바이스 IP 주소에 라우팅합니다. 예 값을 사용 하지 않고 온-프레미스 사이트에 연결 하려는 경우 고유한 값을 사용 합니다.
    - **Bgp 설정 구성**: bgp를 구성 하는 경우에만 사용 합니다. 그렇지 않으면이 옵션을 선택 하지 마세요.
    - **구독**: 올바른 구독이 표시 되는지 확인 합니다.
    - **리소스 그룹**: 사용 하려는 리소스 그룹을 선택 합니다. 새 리소스 그룹을 만들거나 이미 만든 리소스 그룹을 선택할 수 있습니다.
    - **위치**:이 개체가 생성 될 위치를 선택 합니다. VNet이 있는 위치와 동일한 위치를 선택할 수 있지만 반드시 그럴 필요는 없습니다.
5. 필요한 값의 지정을 마치면 **만들기** 를 선택 하 여 로컬 네트워크 게이트웨이를 만듭니다.
6. Azure Stack Hub 배포에서이 단계 (1-5)를 반복 합니다.

## <a name="configure-your-connection"></a>연결 구성

온-프레미스 네트워크에 대 한 사이트 간 연결에는 VPN 장치가 필요 합니다. 구성 하는 VPN 장치를 연결 이라고 합니다. 연결을 구성 하려면 다음이 필요 합니다.

- 공유 키 - 이 키는 사이트 간 VPN 연결을 만들 때 지정 하는 것과 동일한 공유 키입니다. 이 예제에서는 기본적인 공유 키를 사용합니다. 실제로 사용할 키는 좀 더 복잡하게 생성하는 것이 좋습니다.
- 가상 네트워크 게이트웨이의 공용 IP 주소입니다. Azure Portal, PowerShell 또는 CLI를 사용하여 공용 IP 주소를 볼 수 있습니다. Azure Portal를 사용 하 여 VPN 게이트웨이의 공용 IP 주소를 찾으려면 가상 네트워크 게이트웨이로 이동한 다음 게이트웨이의 이름을 선택 합니다.

다음 단계를 사용 하 여 가상 네트워크 게이트웨이와 온-프레미스 VPN 장치 간에 사이트 간 VPN 연결을 만듭니다.

1. Azure Portal에서 **+ 리소스 만들기**를 선택 합니다.
2. **연결**을 검색 합니다.
3. **결과**에서 **연결**을 선택 합니다.
4. **연결**에서 **만들기**를 선택 합니다.
5. **연결 만들기**에서 다음 설정을 구성 합니다.

    - **연결 형식**: 사이트 간 (IPSec)을 선택 합니다.
    - **리소스 그룹**: 테스트 리소스 그룹을 선택 합니다.
    - **Virtual Network Gateway**: 만든 가상 네트워크 게이트웨이를 선택 합니다.
    - **로컬 네트워크 게이트웨이**: 만든 로컬 네트워크 게이트웨이를 선택 합니다.
    - **연결 이름**:이 이름은 두 게이트웨이의 값을 사용 하 여 채워집니다 됩니다.
    - **공유 키**:이 값은 로컬 온-프레미스 VPN 장치에 사용 하는 값과 일치 해야 합니다. 자습서 예제에서는 ' abc123 '를 사용 하지만 좀 더 복잡 한 항목을 사용 해야 합니다. 중요 한 것은이 값이 VPN 장치를 구성할 때 지정 하는 값과 동일 *해야* 한다는 점입니다.
    - **구독**, **리소스 그룹** 및 **위치**에 대한 값이 고정됩니다.

6. **확인** 을 선택 하 여 연결을 만듭니다.

가상 네트워크 게이트웨이의 **연결 페이지에서 연결을** 볼 수 있습니다. 상태는 *알 수 없음* 에서 *연결 중*으로 전환 된 다음 *성공*으로 전환 됩니다.

## <a name="next-steps"></a>다음 단계

- Azure Cloud 패턴에 대해 자세히 알아보려면 [클라우드 디자인 패턴](https://docs.microsoft.com/azure/architecture/patterns)을 참조 하세요.
