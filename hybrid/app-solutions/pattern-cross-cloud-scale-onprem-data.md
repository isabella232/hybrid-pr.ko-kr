---
title: Azure Stack Hub의 클라우드 간 크기 조정(온-프레미스 데이터) 패턴
description: Azure 및 Azure Stack Hub에서 온-프레미스 데이터를 사용하는 확장 가능한 클라우드 간 앱을 빌드하는 방법을 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 5c8e3adb621ae4322bf6d60792fc307dbb24ff90
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281247"
---
# <a name="cross-cloud-scaling-on-premises-data-pattern"></a>클라우드 간 크기 조정(온-프레미스 데이터) 패턴

Azure 및 Azure Stack Hub를 포함하는 하이브리드 앱을 빌드하는 방법을 알아봅니다. 이 패턴은 또한 규정 준수를 위해 단일 온-프레미스 데이터 원본을 사용하는 방법도 보여줍니다.

## <a name="context-and-problem"></a>컨텍스트 및 문제점

많은 조직에서 방대한 양의 민감한 고객 데이터를 수집하고 저장합니다. 기업 규정이나 정부 정책 때문에 중요한 데이터를 퍼블릭 클라우드에 저장하지 못하는 경우가 많습니다. 이러한 조직은 또한 퍼블릭 클라우드의 확장성을 활용하려고 합니다. 퍼블릭 클라우드는 계절에 따른 최대 트래픽을 처리할 수 있으므로 고객은 필요할 때 필요한 하드웨어에 대해 정확하게 비용을 지불할 수 있습니다.

## <a name="solution"></a>해결 방법

이 솔루션은 프라이빗 클라우드의 규정 준수 이점을 활용하고, 이를 퍼블릭 클라우드의 확장성과 결합합니다. Azure 및 Azure Stack Hub 하이브리드 클라우드는 개발자에게 일관된 환경을 제공합니다. 이러한 일관성을 통해 퍼블릭 클라우드와 온-프레미스 환경 모두에 기술을 적용할 수 있습니다.

솔루션 배포 가이드를 사용하면 퍼블릭 및 프라이빗 클라우드에 동일한 웹 애플리케이션을 배포할 수 있습니다. 또한 프라이빗 클라우드에서 호스트되는 인터넷 라우팅이 불가능한 네트워크에 액세스할 수 있습니다. 웹앱의 부하가 모니터링됩니다. 트래픽이 크게 증가하면 프로그램이 DNS 레코드를 조작하여 트래픽을 퍼블릭 클라우드로 리디렉션합니다. 트래픽이 더 이상 증가하지 않으면 DNS 레코드가 업데이트되어 트래픽을 다시 프라이빗 클라우드로 보냅니다.

[![온-프레미스 데이터 패턴을 사용하여 클라우드 간 크기 조정](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)

## <a name="components"></a>구성 요소

이 솔루션은 다음과 같은 구성 요소를 사용합니다.

| 계층 | 구성 요소 | 설명 |
|----------|-----------|-------------|
| Azure | Azure App Service | [Azure App Service](/azure/app-service/)를 사용하여 웹앱, RESTful API 앱 및 Azure Functions를 빌드하고 호스트할 수 있습니다. 인프라 관리 없이 선택한 프로그래밍 언어로 모든 것이 가능합니다. |
| | Azure Virtual Network| [Azure Virtual Network(VNet)](/azure/virtual-network/virtual-networks-overview)는 Azure에서 개인 네트워크의 기본 구성 요소입니다. VNet을 사용하면 VM(가상 머신)과 같은 여러 Azure 리소스 종류가 서로 인터넷 및 온-프레미스 네트워크와 안전하게 통신할 수 있습니다. 이 솔루션은 또한 다음과 같은 추가 네트워킹 구성 요소의 사용을 보여줍니다.<br>- 앱 및 게이트웨이 서브넷.<br>- 로컬 온-프레미스 네트워크 게이트웨이.<br>- 사이트 간 VPN 게이트웨이 연결의 역할을 하는 가상 네트워크 게이트웨이.<br>- 공용 IP 주소.<br>- 지점 및 사이트 간 VPN 연결.<br>- DNS 도메인을 호스트하고 이름 확인을 제공하는 Azure DNS. |
| | Azure Traffic Manager | [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview)는 DNS 기반 트래픽 부하 분산 장치입니다. 이를 사용하면 다양한 데이터 센터에서 서비스 엔드포인트에 대한 사용자 트래픽의 배포를 제어할 수 있습니다. |
| | Azure Application Insights | [Application Insights](/azure/azure-monitor/app/app-insights-overview)는 여러 플랫폼에서 앱을 빌드하고 관리하는 웹 개발자를 위한 확장 가능한 Application Performance Management 서비스입니다.|
| | Azure Functions | [Azure Functions](/azure/azure-functions/)를 사용하면 먼저 VM을 만들거나 웹앱을 게시하지 않고도 서버리스 환경에서 코드를 실행할 수 있습니다. |
| | Azure 자동 크기 조정 | [자동 크기 조정](/azure/azure-monitor/platform/autoscale-overview)은 Cloud Services, VM 및 웹앱의 기본 제공 기능입니다. 이 기능을 사용하면 수요가 변할 때 앱이 최상의 성능을 발휘할 수 있습니다. 앱은 트래픽 급증을 조정하고, 메트릭이 변경될 때 이를 알려주고 필요에 따라 조정합니다. |
| Azure Stack Hub | IaaS Compute | Azure Stack Hub를 사용하면 Azure에서 사용하는 것과 동일한 앱 모델, 셀프 서비스 포털 및 API를 사용할 수 있습니다. Azure Stack Hub IaaS는 일관된 하이브리드 클라우드 배포를 위한 광범위한 오픈 소스 기술을 지원합니다. 예를 들어, 이 솔루션 예제에서는 Windows Server VM을 SQL Server로 사용합니다.|
| | Azure App Service | Azure 웹앱과 마찬가지로 이 솔루션은 [Azure Stack Hub의 Azure App Service](/azure-stack/operator/azure-stack-app-service-overview)를 사용하여 웹앱을 호스트합니다. |
| | 네트워킹 | Azure Stack Hub Virtual Network는 Azure Virtual Network와 동일하게 작동합니다. 사용자 지정 호스트 이름을 포함하여 많은 동일한 네트워킹 구성 요소를 사용합니다.
| Azure DevOps Services | 등록 | 빌드, 테스트 및 배포를 위한 연속 통합을 빠르게 설정합니다. 자세한 내용은 [Azure DevOps에 등록, 로그인](/azure/devops/user-guide/sign-up-invite-teammates?view=azure-devops)을 참조하세요. |
| | Azure Pipelines | 연속 통합/지속적인 업데이트를 위해 [Azure Pipelines](/azure/devops/pipelines/agents/agents?view=azure-devops)를 사용합니다. Azure Pipelines를 사용하면 호스트된 빌드 및 릴리스 에이전트와 정의를 관리할 수 있습니다. |
| | 코드 리포지토리 | 여러 코드 리포지토리를 활용하여 개발 파이프라인을 간소화합니다. GitHub, Bitbucket, Dropbox, OneDrive 및 Azure Repos에서 기존 코드 리포지토리를 사용합니다. |

## <a name="issues-and-considerations"></a>문제 및 고려 사항

이 솔루션을 구현할 방법을 결정할 때 다음 사항을 고려하세요.

### <a name="scalability"></a>확장성

Azure 및 Azure Stack Hub는 오늘날 전역적으로 분산된 비즈니스의 요구 사항을 지원하는 데 적합합니다.

#### <a name="hybrid-cloud-without-the-hassle"></a>번거로움 없는 하이브리드 클라우드

Microsoft는 단일 통합 솔루션에서 Azure Stack Hub 및 Azure와 온-프레미스 자산의 타의 추종을 불허하는 통합을 제공합니다. 이러한 통합을 통해 여러 포인트 솔루션을 관리하고 여러 클라우드 공급자를 관리해야 하는 번거로움을 없앨 수 있습니다. 클라우드 간 크기 조정을 사용하면 몇 번의 클릭만으로 Azure의 강력한 기능을 사용할 수 있습니다. 클라우드 버스팅을 사용하여 Azure Stack Hub를 Azure에 연결하기만 하면 필요할 때 Azure에서 데이터와 앱을 사용할 수 있습니다.

- 보조 DR 사이트를 구축하고 유지 관리할 필요가 없습니다.
- 테이프 백업을 제거하고 Azure에 최대 99년 치의 백업 데이터를 보관하여 시간과 비용을 절약하세요.
- 실행 중인 Hyper-V, 물리적(미리 보기) 및 VMware(미리 보기) 워크로드를 Azure로 쉽게 마이그레이션하여 클라우드의 경제성과 탄력성을 활용합니다.
- 프로덕션 워크로드에 영향을 주지 않고 Azure에서 복제된 온-프레미스 자산 복사본에 대해 컴퓨팅 집약적인 보고서 또는 분석을 실행합니다.
- 필요할 때 더 큰 컴퓨팅 템플릿을 사용하여 Azure에서 클라우드로 버스트하고 온-프레미스 워크로드를 실행합니다. 하이브리드는 필요할 때 필요한 기능을 제공합니다.
- 몇 번의 클릭으로 Azure에서 다중 계층 개발 환경을 만들고 라이브 프로덕션 데이터를 개발/테스트 환경에 복제하여 거의 실시간으로 동기화할 수 있습니다.

#### <a name="economy-of-cross-cloud-scaling-with-azure-stack-hub"></a>Azure Stack Hub를 사용한 클라우드 간 확장의 경제성

클라우드 버스팅의 주요 이점은 경제적인 비용 절감입니다. 해당 리소스에 대한 수요가 있을 때만 추가 리소스에 대해 비용을 지불합니다. 더 이상 불필요한 추가 용량에 지출하거나 수요 피크 및 변동을 예측하려고 할 필요가 없습니다.

#### <a name="reduce-high-demand-loads-into-the-cloud"></a>클라우드로의 높은 수요 부하 감소

클라우드 간 크기 조정을 사용하여 처리 부담을 짊어질 수 있습니다. 기본 앱을 퍼블릭 클라우드로 이동하여 부하를 분산하여 중요 비즈니스용 앱을 위한 로컬 리소스를 확보합니다. 앱을 프라이빗 클라우드에 적용한 다음, 수요를 충족하기 위해 필요할 때만 퍼블릭 클라우드로 버스트할 수 있습니다.

### <a name="availability"></a>가용성

글로벌 배포에는 가변 연결 및 지역별 정부 규정의 차이와 같은 고유한 문제가 있습니다. 개발자는 하나의 앱만 개발한 다음, 다양한 요구 사항을 가진 여러 가지 이유에 따라 배포할 수 있습니다. Azure 퍼블릭 클라우드에 앱을 배포한 다음, 추가 인스턴스 또는 구성 요소를 로컬로 배포합니다. Azure를 사용하여 모든 인스턴스 간의 트래픽을 관리할 수 있습니다.

### <a name="manageability"></a>관리 효율

#### <a name="a-single-consistent-development-approach"></a>일관된 단일 개발 접근 방식

Azure 및 Azure Stack Hub를 사용하면 조직 전체에서 일관된 개발 도구 집합을 사용할 수 있습니다. 이러한 일관성을 통해 CI/CD(연속 통합 및 지속적인 개발) 방식을 더 쉽게 구현할 수 있습니다. Azure 또는 Azure Stack Hub에 배포된 많은 앱과 서비스는 서로 교환이 가능하며 어느 위치에서든 원활하게 실행할 수 있습니다.

하이브리드 CI/CD 파이프라인을 통해 다음을 수행할 수 있습니다.

- 단일 리포지토리에 대한 코드 커밋을 기반으로 새 빌드를 시작합니다.
- 사용자 승인 테스트를 위해 새로 빌드된 코드를 Azure에 자동으로 배포합니다.
- 코드가 테스트를 통과하면 Azure Stack Hub에 자동으로 배포합니다.

### <a name="a-single-consistent-identity-management-solution"></a>일관된 단일 ID 관리 솔루션

Azure Stack Hub는 Azure AD(Azure Active Directory) 및 ADFS(Active Directory Federation Services)와 함께 작동합니다. Azure Stack Hub는 연결된 시나리오에서 Azure AD와 함께 작동합니다. 연결성이 없는 환경의 경우 ADFS를 연결이 끊긴 솔루션으로 사용할 수 있습니다. 서비스 주체는 앱에 대한 액세스 권한을 부여하여 Azure Resource Manager를 통해 리소스를 배포하거나 구성할 수 있도록 합니다.

### <a name="security"></a>보안

#### <a name="ensure-compliance-and-data-sovereignty"></a>규정 준수 및 데이터 주권 보장

Azure Stack Hub를 사용하면 퍼블릭 클라우드를 사용하는 것처럼 여러 국가에서 동일한 서비스를 실행할 수 있습니다. 각 국가의 데이터 센터에 동일한 앱을 배포하면 데이터 주권 요구 사항을 충족할 수 있습니다. 이 기능은 개인 데이터가 각 국가의 국경 내에 유지되도록 합니다.

#### <a name="azure-stack-hub---security-posture"></a>Azure Stack Hub - 보안 태세

견고하고 지속적인 서비스 프로세스가 없으면 보안 태세가 유지되지 않습니다. 이러한 이유로 Microsoft는 전체 인프라에 패치와 업데이트를 원활하게 적용하는 오케스트레이션 엔진에 투자했습니다.

Microsoft는 Azure Stack Hub OEM 파트너와의 파트너 관계 덕분에 하드웨어 수명 주기 호스트와 그 위에서 실행되는 소프트웨어와 같은 OEM별 구성 요소에도 동일한 보안 태세를 확장합니다. 이 파트너 관계를 통해 Azure Stack Hub는 전체 인프라에서 균일하고 견고한 보안 태세를 유지할 수 있습니다. 결과적으로 고객은 앱 워크로드를 빌드하고 보호할 수 있습니다.

#### <a name="use-of-service-principals-via-powershell-cli-and-azure-portal"></a>PowerShell, CLI 및 Azure Portal을 통해 서비스 주체 사용

스크립트 또는 앱에 대한 리소스 액세스 권한을 부여하려면 앱의 ID를 설정하고 자체 자격 증명으로 앱을 인증합니다. 이 ID를 서비스 주체라고 하며 다음을 수행할 수 있습니다.

- 사용자 고유의 권한과 다르고 앱의 요구 사항에 맞게 제한되는 앱 ID에 사용 권한을 할당합니다.
- 무인 스크립트를 실행할 때 인증을 위해 인증서를 사용합니다.

서비스 주체 만들기 및 자격 증명에 인증서 사용에 대한 자세한 내용은 [앱 ID를 사용하여 리소스 액세스](/azure-stack/operator/azure-stack-create-service-principals)를 참조하세요.

## <a name="when-to-use-this-pattern"></a>이 패턴을 사용해야 하는 경우

- 우리 조직에서 DevOps 접근 방식을 사용하고 있거나 가까운 장래에 계획하고 있습니다.
- Azure Stack Hub 구현과 퍼블릭 클라우드에서 CI/CD 방식을 구현하려고 합니다.
- 클라우드 및 온-프레미스 환경에서 CI/CD 파이프라인을 통합하려고 합니다.
- 클라우드 또는 온-프레미스 서비스를 사용하여 원활하게 앱을 개발하는 기능을 원합니다.
- 클라우드 및 온-프레미스 앱에서 일관된 개발자 기술을 활용하려고 합니다.
- Azure를 사용하지만 온-프레미스 Azure Stack Hub 클라우드에서 작업하는 개발자가 있습니다.
- 내 온-프레미스 앱은 계절적, 주기적 또는 예측할 수 없는 변동 시 수요가 급증합니다.
- 온-프레미스 구성 요소가 있고 클라우드를 사용하여 원활하게 크기를 조정하려고 합니다.
- 클라우드 확장성을 원하지만 내 앱을 가능한 한 온-프레미스에서 실행하려고 합니다.

## <a name="next-steps"></a>다음 단계

이 문서에서 소개하는 항목에 대한 자세한 내용은 다음을 참조하세요.

- 이 패턴이 사용되는 방법에 대한 개요는 [데이터 센터와 퍼블릭 클라우드 간에 앱을 동적으로 확장](https://www.youtube.com/watch?v=2lw8zOpJTn0)을 참조하세요.
- 모범 사례에 대해 자세히 알아보고 추가 질문에 답변하려면 [하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md)을 참조하세요.
- 이 패턴은 Azure Stack Hub를 포함하여 Azure Stack 제품군을 사용합니다. 제품 및 솔루션의 전체 포트폴리오에 대해 자세히 알아보려면 [Azure Stack 제품군 및 솔루션](/azure-stack)을 참조하세요.

솔루션 예제를 테스트할 준비가 되면 [클라우드 간 크기 조정(온-프레미스 데이터) 솔루션 배포 가이드](/azure/architecture/hybrid/deployments/solution-deployment-guide-cross-cloud-scaling-onprem-data)를 계속 진행하세요. 배포 가이드에서는 해당 구성 요소를 배포 및 테스트하는 방법에 대한 단계별 지침을 제공합니다.