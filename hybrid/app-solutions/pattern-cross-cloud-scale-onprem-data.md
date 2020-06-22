---
title: Azure Stack 허브의 클라우드 간 크기 조정 (온-프레미스 데이터) 패턴
description: Azure 및 Azure Stack Hub에서 온-프레미스 데이터를 사용 하는 확장 가능한 클라우드 간 앱을 빌드하는 방법에 대해 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: edbb608fbf8e5288f29572bfe4cca98ffb3cb8fc
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911127"
---
# <a name="cross-cloud-scaling-on-premises-data-pattern"></a>클라우드 간 크기 조정 (온-프레미스 데이터) 패턴

Azure 및 Azure Stack Hub를 포괄 하는 하이브리드 앱을 빌드하는 방법에 대해 알아봅니다. 또한이 패턴에서는 단일 온-프레미스 데이터 원본을 규정 준수에 사용 하는 방법을 보여 줍니다.

## <a name="context-and-problem"></a>컨텍스트 및 문제점

많은 조직에서 대량의 중요 한 고객 데이터를 수집 하 고 저장 합니다. 회사 규정 또는 정부 정책으로 인해 공용 클라우드의 중요 한 데이터를 저장 하지 못하는 경우가 종종 있습니다. 또한 이러한 조직은 공용 클라우드의 확장성을 활용 하려고 합니다. 공용 클라우드는 트래픽에서 계절 최대 사용량을 처리할 수 있으므로 고객이 필요할 때 필요한 하드웨어에 대 한 비용을 지불할 수 있습니다.

## <a name="solution"></a>해결 방법

이 솔루션은 사설 클라우드의 규정 준수 이점을 활용 하 여 공용 클라우드의 확장성과 결합 합니다. Azure 및 Azure Stack Hub 하이브리드 클라우드는 개발자에 게 일관 된 환경을 제공 합니다. 이러한 일관성을 통해 공용 클라우드와 온-프레미스 환경 모두에 기술을 적용할 수 있습니다.

솔루션 배포 가이드를 사용 하 여 공용 및 사설 클라우드에 동일한 웹 앱을 배포할 수 있습니다. 사설 클라우드에서 호스트 되는 인터넷 라우팅할 수 없는 네트워크에도 액세스할 수 있습니다. 웹 앱이 부하에 대해 모니터링 됩니다. 트래픽이 크게 증가할 때 프로그램은 DNS 레코드를 조작 하 여 트래픽을 공용 클라우드로 리디렉션합니다. 트래픽이 더 이상 중요 하지 않은 경우 DNS 레코드는 트래픽을 사설 클라우드로 다시 보내도록 업데이트 됩니다.

[![온-프레미스 데이터 패턴을 사용 하 여 클라우드 간 크기 조정](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)

## <a name="components"></a>구성 요소

이 솔루션은 다음 구성 요소를 사용 합니다.

| 계층 | 구성 요소 | Description |
|----------|-----------|-------------|
| Azure | Azure App Service | [Azure App Service](/azure/app-service/) 를 사용 하 여 웹 앱, RESTful API apps 및 Azure Functions를 빌드하고 호스트할 수 있습니다. 인프라를 관리 하지 않고 선택한 프로그래밍 언어로 모두 사용 합니다. |
| | Azure Virtual Network| [VNet (azure Virtual Network)](/azure/virtual-network/virtual-networks-overview) 은 azure의 개인 네트워크에 대 한 기본 구성 요소입니다. VNet을 사용 하면 VM (가상 머신)과 같은 여러 Azure 리소스 유형을 사용 하 여 서로 안전 하 게 통신할 수 있습니다. 또한이 솔루션은 추가 네트워킹 구성 요소를 사용 하는 방법을 보여 줍니다.<br>-앱 및 게이트웨이 서브넷입니다.<br>-로컬 온-프레미스 네트워크 게이트웨이입니다.<br>-사이트 간 VPN gateway 연결의 역할을 하는 가상 네트워크 게이트웨이.<br>-공용 IP 주소입니다.<br>-지점 및 사이트 간 VPN 연결입니다.<br>-DNS 도메인을 호스트 하 고 이름 확인을 제공 하는 Azure DNS. |
| | Azure Traffic Manager | [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) 는 DNS 기반 트래픽 부하 분산 장치입니다. 이를 통해 다양 한 데이터 센터의 서비스 끝점에 대 한 사용자 트래픽 배포를 제어할 수 있습니다. |
| | Azure Application Insights | [Application Insights](/azure/azure-monitor/app/app-insights-overview) 는 여러 플랫폼에서 앱을 빌드하고 관리 하는 웹 개발자를 위한 확장 가능한 응용 프로그램 성능 관리 서비스입니다.|
| | Azure 기능 | [Azure Functions](/azure/azure-functions/) 를 사용 하면 먼저 VM을 만들거나 웹 앱을 게시 하지 않고도 서버를 사용 하지 않는 환경에서 코드를 실행할 수 있습니다. |
| | Azure 자동 크기 조정 | [자동 크기 조정은](/azure/azure-monitor/platform/autoscale-overview) Cloud Services, vm 및 웹 앱의 기본 제공 기능입니다. 이 기능을 사용 하면 요구 사항이 변경 될 때 앱에서 가장 적합 한 기능을 수행할 수 있습니다. 앱은 트래픽 급증에 맞게 조정 하 여 필요에 따라 메트릭을 변경 하 고 크기를 조정 하는 경우 사용자에 게 알립니다. |
| Azure Stack 허브 | IaaS 계산 | Azure Stack 허브를 사용 하면 Azure에서 사용 하는 것과 동일한 앱 모델, 셀프 서비스 포털 및 Api를 사용할 수 있습니다. Azure Stack 허브 IaaS는 일관 된 하이브리드 클라우드 배포를 위해 광범위 한 오픈 소스 기술을 허용 합니다. 예를 들어 솔루션 예제에서는 Windows Server VM을 사용 하 여 SQL Server 합니다.|
| | Azure App Service | Azure 웹 앱과 마찬가지로 솔루션은 [Azure Stack Hub에서 Azure App Service](/azure-stack/operator/azure-stack-app-service-overview) 를 사용 하 여 웹 앱을 호스팅합니다. |
| | 네트워킹 | Azure Stack 허브 Virtual Network Azure Virtual Network와 동일 하 게 작동 합니다. 사용자 지정 호스트 이름을 포함 하 여 여러 개의 동일한 네트워킹 구성 요소를 사용 합니다.
| Azure DevOps Services | 등록 | 빌드, 테스트 및 배포에 대 한 지속적인 통합을 신속 하 게 설정 합니다. 자세한 내용은 [등록, Azure DevOps에 로그인](/azure/devops/user-guide/sign-up-invite-teammates?view=azure-devops)을 참조 하세요. |
| | Azure Pipelines | 지속적인 통합/지속적인 업데이트를 위해 [Azure Pipelines](/azure/devops/pipelines/agents/agents?view=azure-devops) 를 사용 합니다. Azure Pipelines를 사용 하 여 호스팅된 빌드 및 릴리스 에이전트와 정의를 관리할 수 있습니다. |
| | 코드 리포지토리 | 여러 코드 리포지토리를 활용 하 여 개발 파이프라인을 간소화 합니다. GitHub, Bitbucket, Dropbox, OneDrive 및 Azure Repos에서 기존 코드 리포지토리를 사용 합니다. |

## <a name="issues-and-considerations"></a>문제 및 고려 사항

이 솔루션을 구현 하는 방법을 결정할 때 다음 사항을 고려 하십시오.

### <a name="scalability"></a>확장성

Azure 및 Azure Stack 허브는 오늘날 전 세계에 분산 된 비즈니스의 요구를 지원 하기 위해 고유 하 게 적합 합니다.

#### <a name="hybrid-cloud-without-the-hassle"></a>하이브리드 클라우드

Microsoft는 단일 통합 솔루션에서 Azure Stack 허브 및 Azure와 온-프레미스 자산의 통합을 제공 합니다. 이러한 통합을 통해 여러 지점 솔루션과 클라우드 공급자를 함께 관리할 필요가 없습니다. 클라우드 간 확장을 사용 하면 Azure의 강력한 기능을 몇 번만 클릭 하면 됩니다. 클라우드 버스트를 사용 하 여 Azure Stack 허브를 Azure에 연결 하기만 하면 필요한 경우 Azure에서 데이터와 앱을 사용할 수 있습니다.

- 보조 DR 사이트를 구축 하 고 유지 관리할 필요가 없습니다.
- 테이프 백업을 제거 하 고 Azure에서 최대 99 년 분량의 백업 데이터를 보관 하 여 시간과 비용을 절감 하세요.
- 실행 중인 Hyper-v, 물리적 (미리 보기) 및 VMware (미리 보기 상태) 워크 로드를 Azure로 쉽게 마이그레이션하여 클라우드의 경제성과 탄력성을 활용할 수 있습니다.
- 프로덕션 워크 로드에 영향을 주지 않고 Azure에서 온-프레미스 자산의 복제 된 복사본에 대해 계산 집약적 보고서 또는 분석을 실행 합니다.
- 클라우드로 버스트 하 고 필요한 경우 더 큰 계산 템플릿을 사용 하 여 Azure에서 온-프레미스 워크 로드를 실행 합니다. 하이브리드는 필요할 때 필요한 기능을 제공 합니다.
- 몇 번의 클릭으로 Azure에서 다중 계층 개발 환경을 만들 수 있습니다. 실시간 프로덕션 데이터를 개발/테스트 환경으로 복제 하 여 거의 실시간 동기화로 유지 합니다.

#### <a name="economy-of-cross-cloud-scaling-with-azure-stack-hub"></a>Azure Stack 허브를 사용한 클라우드 간 확장의 경제

클라우드 버스트의 주요 이점은 경제적으로 절감 됩니다. 해당 리소스에 대 한 요구가 있으면 추가 리소스에 대해서만 비용을 지불 합니다. 불필요 한 추가 용량에 대 한 지출 또는 수요 증가 및 변동 예측을 시도 하지 않습니다.

#### <a name="reduce-high-demand-loads-into-the-cloud"></a>대량 수요 부하를 클라우드로 줄이기

클라우드 간 크기 조정을 사용 하 여 부담 처리를 수행할 수 있습니다. 부하는 기본 앱을 공용 클라우드로 이동 하 여 비즈니스에 중요 한 앱에 대 한 로컬 리소스를 확보 하 여 배포 됩니다. 앱을 사설 클라우드에 적용 한 다음 요청을 충족 하는 데 필요한 경우에만 공용 클라우드로 버스트 할 수 있습니다.

### <a name="availability"></a>가용성

글로벌 배포에는 가변 연결 및 지역별 정부 규제와 같은 고유한 과제가 있습니다. 개발자는 앱을 하나만 개발한 다음 요구 사항이 서로 다른 여러 가지 이유로 배포할 수 있습니다. 앱을 Azure 공용 클라우드에 배포한 다음 추가 인스턴스 또는 구성 요소를 로컬로 배포 합니다. Azure를 사용 하 여 모든 인스턴스 간의 트래픽을 관리할 수 있습니다.

### <a name="manageability"></a>관리 효율

#### <a name="a-single-consistent-development-approach"></a>일관 된 단일 개발 방법

Azure 및 Azure Stack 허브를 사용 하면 조직 전체에서 일관 된 개발 도구 집합을 사용할 수 있습니다. 이러한 일관성을 통해 지속적인 통합 및 지속적인 개발 (CI/CD)의 사례를 보다 쉽게 구현할 수 있습니다. Azure 또는 Azure Stack 허브에 배포 된 많은 앱과 서비스는 서로 교환할 수 있으며 어느 위치에서 나 원활 하 게 실행할 수 있습니다.

하이브리드 CI/CD 파이프라인을 통해 다음과 같은 작업을 수행할 수 있습니다.

- 코드 리포지토리에 코드 커밋을 기반으로 하는 새 빌드를 시작 합니다.
- 사용자 승인 테스트를 위해 새로 빌드된 코드를 Azure에 자동으로 배포 합니다.
- 코드가 테스트에 통과 하면 Azure Stack Hub에 자동으로 배포 됩니다.

### <a name="a-single-consistent-identity-management-solution"></a>단일 일관 된 id 관리 솔루션

Azure Stack 허브는 Azure Active Directory (Azure AD) 및 Active Directory Federation Services (ADFS) 둘 다에서 작동 합니다. Azure Stack 허브는 연결 된 시나리오에서 Azure AD와 함께 작동 합니다. 연결이 없는 환경의 경우 연결이 끊어진 솔루션으로 ADFS를 사용할 수 있습니다. 서비스 사용자는 앱에 대 한 액세스 권한을 부여 하 여 Azure Resource Manager를 통해 리소스를 배포 하거나 구성할 수 있도록 하는 데 사용 됩니다.

### <a name="security"></a>보안

#### <a name="ensure-compliance-and-data-sovereignty"></a>준수 및 데이터 주권 보장

Azure Stack 허브를 사용 하면 공용 클라우드를 사용 하는 경우 처럼 여러 국가에서 동일한 서비스를 실행할 수 있습니다. 각 국가의 데이터 센터에 동일한 앱을 배포 하면 데이터 주권 요구 사항을 충족할 수 있습니다. 이 기능을 사용 하면 개인 데이터가 각 국가의 테두리 내에 유지 됩니다.

#### <a name="azure-stack-hub---security-posture"></a>Azure Stack 허브-보안 상태

지속적인 연속 서비스 프로세스가 없는 보안 상태는 없습니다. 따라서 Microsoft는 전체 인프라에서 패치 및 업데이트를 원활 하 게 적용 하는 오케스트레이션 엔진에 투자 했습니다.

Azure Stack Hub OEM 파트너와의 파트너 관계 덕분에 Microsoft는 하드웨어 수명 주기 호스트 및이를 기반으로 하는 소프트웨어와 같은 OEM 관련 구성 요소와 동일한 보안 환경을 확장 합니다. 이 파트너 관계를 통해 Azure Stack 허브는 전체 인프라에서 일관 되 고 견고한 보안 상태를 유지할 것입니다. 그러면 고객은 자신의 앱 워크 로드를 빌드하고 보호할 수 있습니다.

#### <a name="use-of-service-principals-via-powershell-cli-and-azure-portal"></a>PowerShell, CLI 및 Azure Portal를 통해 서비스 주체 사용

스크립트나 앱에 대 한 리소스 액세스 권한을 부여 하려면 앱에 대 한 id를 설정 하 고 자체 자격 증명을 사용 하 여 앱을 인증 합니다. 이 id를 서비스 주체 라고 하며 다음을 수행할 수 있습니다.

- 사용자 고유의 권한과 다르며 앱의 요구 사항에 따라 정확 하 게 제한 되는 앱 id에 사용 권한을 할당 합니다.
- 무인 스크립트를 실행할 때 인증을 위해 인증서를 사용합니다.

서비스 사용자 만들기 및 자격 증명에 대 한 인증서 사용에 대 한 자세한 내용은 [앱 id를 사용 하 여 리소스에 액세스](/azure-stack/operator/azure-stack-create-service-principals)를 참조 하세요.

## <a name="when-to-use-this-pattern"></a>이 패턴을 사용해야 하는 경우

- 조직에서 DevOps 방법을 사용 하 고 있거나 가까운 장래에 계획 된 방법이 있습니다.
- Azure Stack 허브 구현 및 공용 클라우드에서 CI/CD 사례를 구현 하려고 합니다.
- 클라우드 및 온-프레미스 환경에서 CI/CD 파이프라인을 통합 하려고 합니다.
- 클라우드 또는 온-프레미스 서비스를 사용 하 여 앱을 원활 하 게 개발할 수 있습니다.
- 클라우드 및 온-프레미스 앱에서 일관 된 개발자 기술을 활용 하려고 합니다.
- Azure를 사용 중이지만 온-프레미스 Azure Stack 허브 클라우드에서 작업 하는 개발자가 있습니다.
- 내 온-프레미스 앱은 계절, 순환 또는 예측할 수 없는 변동 중에 수요 급증을 경험 합니다.
- 온-프레미스 구성 요소가 있고 클라우드를 사용 하 여 원활 하 게 크기를 조정 하려고 합니다.
- 클라우드 확장성을 원합니다. 하지만 앱을 최대한 많이 온-프레미스에서 실행 하려고 합니다.

## <a name="next-steps"></a>다음 단계

이 문서에서 소개 하는 항목에 대 한 자세한 내용은 다음을 참조 하세요.

- 이 패턴이 사용 되는 방법에 대 한 개요는 [데이터 센터와 공용 클라우드 간에 동적으로 앱 크기 조정](https://www.youtube.com/watch?v=2lw8zOpJTn0) 을 시청 하세요.
- 모범 사례에 대 한 자세한 내용과 추가 질문에 답변 하려면 [하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 을 참조 하세요.
- 이 패턴은 Azure Stack 허브를 포함 하 여 제품의 Azure Stack 제품군을 사용 합니다. 제품 및 솔루션의 전체 포트폴리오에 대해 자세히 알아보려면 [Azure Stack 제품군 및 솔루션](/azure-stack) 을 참조 하세요.

솔루션 예제를 테스트할 준비가 되 면 [클라우드 간 확장 (온-프레미스 데이터) 솔루션 배포 가이드](solution-deployment-guide-cross-cloud-scaling-onprem-data.md)를 계속 진행 합니다. 배포 가이드에서는 구성 요소를 배포 및 테스트 하는 방법에 대 한 단계별 지침을 제공 합니다.
