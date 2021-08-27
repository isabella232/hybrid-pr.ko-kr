---
title: Azure Stack Hub의 클라우드 간 크기 조정 패턴
description: Azure 및 Azure Stack Hub에서 확장 가능한 클라우드 간 앱을 빌드하는 방법을 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 90e0c177b5eaee4d223b4613e0b2ddf385fa799c
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281264"
---
# <a name="cross-cloud-scaling-pattern"></a>클라우드 간 크기 조정 패턴

부하 증가를 수용하기 위해 기존 앱에 리소스를 자동으로 추가합니다.

## <a name="context-and-problem"></a>컨텍스트 및 문제점

앱은 예기치 않은 수요 증가를 충족하기 위해 용량을 늘릴 수 없습니다. 이렇게 확장성이 부족하면 사용자는 사용량이 가장 많은 시간에 앱에 연결할 수 없습니다. 앱은 정해진 수의 사용자에게만 서비스할 수 있습니다.

글로벌 기업에는 안전하고 안정적이며 사용 가능한 클라우드 기반 앱이 필요합니다. 수요 증가를 충족하고 적절한 인프라를 사용하여 해당 수요를 지원하는 것이 중요합니다. 기업은 비즈니스 데이터 보안, 스토리지 및 실시간 가용성과 비용 및 유지 관리의 균형을 맞추기 위해 노력하고 있습니다.

퍼블릭 클라우드에서는 앱을 실행하지 못할 수도 있습니다. 그러나 앱의 급증하는 수요를 처리하기 위해 온-프레미스 환경에 필요한 용량을 유지하기가 경제적으로 어려운 기업도 있습니다. 이 패턴을 사용하면 온-프레미스 솔루션에서 퍼블릭 클라우드의 탄력성을 사용할 수 있습니다.

## <a name="solution"></a>해결 방법

클라우드 간 크기 조정 패턴은 퍼블릭 클라우드 리소스가 있는 로컬 클라우드에 있는 앱을 확장합니다. 이 패턴은 수요 증가 또는 감소에 의해 트리거되며 클라우드에서 리소스를 각각 추가하거나 제거합니다. 이러한 리소스는 중복성, 신속한 가용성 및 지역 규격 라우팅을 제공합니다.

![클라우드 간 크기 조정 패턴](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> 이 패턴은 앱의 상태 비저장 구성 요소에만 적용됩니다.

## <a name="components"></a>구성 요소

클라우드 간 크기 조정 패턴은 다음 구성 요소로 구성됩니다.

### <a name="outside-the-cloud"></a>클러스터 외부

#### <a name="traffic-manager"></a>Traffic Manager

다이어그램에서 이는 퍼블릭 클라우드 그룹 외부에 있지만, 로컬 데이터 센터와 퍼블릭 클라우드 모두에서 트래픽을 조정할 수 있어야 합니다. 분산 장치는 엔드포인트를 모니터링하고 필요할 때 장애 조치(failover) 재배포를 제공하여 앱에 대한 고가용성을 제공합니다.

#### <a name="domain-name-system-dns"></a>DNS(Domain Name System)

Domain Name System, 즉 DNS는 웹 사이트 또는 서비스 이름을 해당 IP 주소로 변환(또는 확인)합니다.

### <a name="cloud"></a>클라우드

#### <a name="hosted-build-server"></a>호스트된 빌드 서버

빌드 파이프라인을 호스트하기 위한 환경입니다.

#### <a name="app-resources"></a>앱 리소스

앱 리소스는 가상 머신 확장 집합 및 컨테이너와 같이 스케일 인 및 스케일 아웃할 수 있어야 합니다.

#### <a name="custom-domain-name"></a>사용자 지정 도메인 이름

라우팅 요청 GLOB에 사용자 지정 도메인 이름을 사용합니다.

#### <a name="public-ip-addresses"></a>공용 IP 주소

공용 IP 주소는 트래픽 관리자를 통해 들어오는 트래픽을 퍼블릭 클라우드 앱 리소스 엔드포인트로 라우팅하는 데 사용됩니다.  

### <a name="local-cloud"></a>로컬 클라우드

#### <a name="hosted-build-server"></a>호스트된 빌드 서버

빌드 파이프라인을 호스트하기 위한 환경입니다.

#### <a name="app-resources"></a>앱 리소스

앱 리소스는 가상 머신 확장 집합 및 컨테이너와 같이 스케일 인 및 스케일 아웃할 수 있는 기능이 필요합니다.

#### <a name="custom-domain-name"></a>사용자 지정 도메인 이름

라우팅 요청 GLOB에 사용자 지정 도메인 이름을 사용합니다.

#### <a name="public-ip-addresses"></a>공용 IP 주소

공용 IP 주소는 트래픽 관리자를 통해 들어오는 트래픽을 퍼블릭 클라우드 앱 리소스 엔드포인트로 라우팅하는 데 사용됩니다.

## <a name="issues-and-considerations"></a>문제 및 고려 사항

이 패턴을 구현할 방법을 결정할 때 다음 사항을 고려하세요.

### <a name="scalability"></a>확장성

클라우드 간 확장의 핵심 구성 요소는 주문형 크기 조정을 제공하는 기능입니다. 확장은 퍼블릭 및 로컬 클라우드 인프라 간에 발생해야 하며 수요에 따라 일관되고 안정적인 서비스를 제공해야 합니다.

### <a name="availability"></a>가용성

로컬로 배포된 앱이 온-프레미스 하드웨어 구성 및 소프트웨어 배포를 통해 고가용성으로 구성되도록 합니다.

### <a name="manageability"></a>관리 효율

클라우드 간 패턴은 환경 간에 원활한 관리 및 친숙한 인터페이스를 보장합니다.

## <a name="when-to-use-this-pattern"></a>이 패턴을 사용해야 하는 경우

다음과 같은 경우에 이 패턴을 사용합니다.

- 예상치 못한 수요나 주기적 수요로 앱 용량을 늘려야 하는 경우.
- 최대 사용 기간 동안에만 사용되는 리소스에 투자하지 않으려는 경우. 사용한 만큼 요금을 지불하려는 경우.

다음과 같은 경우 이 패턴이 권장되지 않습니다.

- 사용자가 인터넷에 연결하여 솔루션을 사용하는 경우.
- 회사에 현장 호출로부터 발생한 원래 연결을 요구하는 현지 규정이 있는 경우.
- 네트워크에서 크기 조정 성능을 제한하는 정기적인 병목 상태가 발생하는 경우.
- 사용자 환경에서 인터넷 연결이 끊어지고 퍼블릭 클라우드에 연결할 수 없는 경우.

## <a name="next-steps"></a>다음 단계

이 문서에서 소개하는 항목에 대한 자세한 내용은 다음을 참조하세요.

- 이 DNS 기반 트래픽 부하 분산 장치의 작동 방식에 대해 자세히 알아보려면 [Azure Traffic Manager 개요](/azure/traffic-manager/traffic-manager-overview)를 참조하세요.
- 모범 사례에 대해 자세히 알아보고 추가 질문에 대한 답변을 얻으려면 [하이브리드 애플리케이션 디자인 고려 사항](overview-app-design-considerations.md)을 참조하세요.
- 제품 및 솔루션의 전체 포트폴리오에 대해 자세히 알아보려면 [Azure Stack 제품군 및 솔루션](/azure-stack)을 참조하세요.

솔루션 예제를 테스트할 준비가 되면 [클라우드 간 크기 조정 솔루션 배포 가이드](/azure/architecture/hybrid/deployments/solution-deployment-guide-cross-cloud-scaling)를 계속 진행하세요. 배포 가이드에서는 해당 구성 요소를 배포 및 테스트하는 방법에 대한 단계별 지침을 제공합니다. Azure Stack Hub 호스트 웹앱에서 Azure 호스트 웹앱으로 전환하기 위해 수동으로 트리거되는 프로세스를 제공하는 클라우드 간 솔루션을 만드는 방법을 알아봅니다. 또한 트래픽 관리자를 통해 자동 크기 조정을 사용하여 부하가 있을 때 유연하고 확장 가능한 클라우드 유틸리티를 보장하는 방법을 알아봅니다.