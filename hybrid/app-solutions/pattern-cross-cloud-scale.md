---
title: Azure Stack 허브의 교차 클라우드 크기 조정 패턴
description: Azure에서 확장 가능한 크로스 클라우드 앱을 빌드하는 방법 및 Azure Stack 허브에 대해 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: a830f96e97c347cbbcc09a1b17f4836ecb6eb3e6
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911134"
---
# <a name="cross-cloud-scaling-pattern"></a>클라우드 간 배율 패턴

부하 증가를 수용 하기 위해 기존 앱에 리소스를 자동으로 추가 합니다.

## <a name="context-and-problem"></a>컨텍스트 및 문제점

앱은 예기치 않은 수요 증가를 충족 하기 위해 용량을 늘릴 수 없습니다. 이러한 확장성이 부족 하면 사용량이 가장 많은 시간 동안 사용자가 앱에 도달 하지 않습니다. 앱은 고정 된 수의 사용자를 서비스 할 수 있습니다.

글로벌 기업은 안전 하 고 신뢰할 수 있으며 사용 가능한 클라우드 기반 앱이 필요 합니다. 요청에 대 한 모임이 늘어나고 적절 한 인프라를 사용 하 여 수요를 지원할 수 있습니다. 비즈니스는 비즈니스 데이터 보안, 저장소 및 실시간 가용성을 사용 하 여 비용 및 유지 관리를 분산 하는 데 어려움을 갖고 있습니다.

공용 클라우드에서 앱을 실행할 수 없습니다. 그러나 앱에 대 한 급증 하는 요청을 처리 하기 위해 온-프레미스 환경에 필요한 용량을 유지 하는 것이 경제적으로 수 있는 것은 아닙니다. 이 패턴을 사용 하면 온-프레미스 솔루션에서 공용 클라우드의 탄력성을 사용할 수 있습니다.

## <a name="solution"></a>해결 방법

클라우드 간 크기 조정 패턴은 공용 클라우드 리소스를 사용 하 여 로컬 클라우드에 있는 앱을 확장 합니다. 패턴은 요청 시 증가 또는 감소에 의해 트리거되고, 각각 클라우드에서 리소스를 추가 하거나 제거 합니다. 이러한 리소스는 중복성, 신속한 가용성 및 지리적 규격 라우팅을 제공 합니다.

![클라우드 간 배율 패턴](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> 이 패턴은 응용 프로그램의 상태 비저장 구성 요소에만 적용 됩니다.

## <a name="components"></a>구성 요소

클라우드 간 배율 패턴은 다음 구성 요소로 구성 됩니다.

### <a name="outside-the-cloud"></a>클라우드 외부

#### <a name="traffic-manager"></a>Traffic Manager

다이어그램에서이는 공용 클라우드 그룹의 외부에 있지만 로컬 데이터 센터와 공용 클라우드 모두에서 트래픽을 조정할 수 있어야 합니다. 분산 장치는 끝점을 모니터링 하 고 필요한 경우 장애 조치 (failover) 재배포를 제공 하 여 앱의 고가용성을 제공 합니다.

#### <a name="domain-name-system-dns"></a>DNS(Domain Name System)

Domain Name System, 즉 DNS는 웹 사이트 또는 서비스 이름을 해당 IP 주소로 변환(또는 확인)합니다.

### <a name="cloud"></a>클라우드

#### <a name="hosted-build-server"></a>호스팅된 빌드 서버

빌드 파이프라인을 호스트 하는 환경입니다.

#### <a name="app-resources"></a>앱 리소스

앱 리소스는 가상 머신 확장 집합 및 컨테이너와 같이 규모를 확장 하 고 확장할 수 있어야 합니다.

#### <a name="custom-domain-name"></a>사용자 지정 도메인 이름

Glob 라우팅 요청에 사용자 지정 도메인 이름을 사용 합니다.

#### <a name="public-ip-addresses"></a>공용 IP 주소

공용 IP 주소는 traffic manager를 통해 들어오는 트래픽을 공용 클라우드 앱 리소스 끝점으로 라우팅하는 데 사용 됩니다.  

### <a name="local-cloud"></a>로컬 클라우드

#### <a name="hosted-build-server"></a>호스팅된 빌드 서버

빌드 파이프라인을 호스트 하는 환경입니다.

#### <a name="app-resources"></a>앱 리소스

앱 리소스에는 가상 머신 확장 집합 및 컨테이너와 같은 규모를 확장 하 고 확장 하는 기능이 필요 합니다.

#### <a name="custom-domain-name"></a>사용자 지정 도메인 이름

Glob 라우팅 요청에 사용자 지정 도메인 이름을 사용 합니다.

#### <a name="public-ip-addresses"></a>공용 IP 주소

공용 IP 주소는 traffic manager를 통해 들어오는 트래픽을 공용 클라우드 앱 리소스 끝점으로 라우팅하는 데 사용 됩니다.

## <a name="issues-and-considerations"></a>문제 및 고려 사항

이 패턴을 구현할 방법을 결정할 때 다음 사항을 고려하세요.

### <a name="scalability"></a>확장성

클라우드 간 크기 조정의 주요 구성 요소는 주문형 크기 조정을 제공 하는 기능입니다. 확장은 공용 및 로컬 클라우드 인프라 사이에서 이루어져야 하며, 수요에 따라 일관적이 고 신뢰할 수 있는 서비스를 제공 해야 합니다.

### <a name="availability"></a>가용성

로컬로 배포 된 앱이 온-프레미스 하드웨어 구성 및 소프트웨어 배포를 통해 고가용성으로 구성 되어 있는지 확인 합니다.

### <a name="manageability"></a>관리 효율

클라우드 간 패턴은 환경 간에 원활한 관리 및 친숙 한 인터페이스를 보장 합니다.

## <a name="when-to-use-this-pattern"></a>이 패턴을 사용해야 하는 경우

이 패턴을 사용합니다.

- 예기치 않은 요청 또는 요청 시 정기적 수요로 앱 용량을 늘려야 하는 경우
- 최대 사용 기간 동안에만 사용 되는 리소스에 투자 하지 않으려는 경우 사용한 양만큼 요금을 지불 합니다.

이 패턴은 다음과 같은 경우에 권장 되지 않습니다.

- 사용자의 솔루션을 사용 하려면 인터넷을 통해 연결 하는 사용자가 필요 합니다.
- 비즈니스에는 온사이트 호출에서 들어오는 연결을 요구 하는 로컬 규정이 있습니다.
- 네트워크에는 크기 조정의 성능을 제한 하는 정기적인 병목 현상이 발생 합니다.
- 사용자 환경이 인터넷에서 연결이 끊어져 공용 클라우드에 연결할 수 없습니다.

## <a name="next-steps"></a>다음 단계

이 문서에서 소개 하는 항목에 대 한 자세한 내용은 다음을 참조 하세요.

- 이 DNS 기반 트래픽 부하 분산 장치 작동 방법에 대해 자세히 알아보려면 [Azure Traffic Manager 개요](/azure/traffic-manager/traffic-manager-overview) 를 참조 하세요.
- 모범 사례에 대 한 자세한 내용을 알아보고 추가 질문에 대 한 답변을 얻으려면 [하이브리드 응용 프로그램 디자인 고려 사항](overview-app-design-considerations.md) 을 참조 하세요.
- 제품 및 솔루션의 전체 포트폴리오에 대해 자세히 알아보려면 [Azure Stack 제품군 및 솔루션](/azure-stack) 을 참조 하세요.

솔루션 예제를 테스트할 준비가 되 면 [클라우드 간 확장 솔루션 배포 가이드](solution-deployment-guide-cross-cloud-scaling.md)를 계속 진행 합니다. 배포 가이드에서는 구성 요소를 배포 및 테스트 하는 방법에 대 한 단계별 지침을 제공 합니다. Azure Stack Hub 호스팅 웹 앱에서 Azure 호스트 된 웹 앱으로 전환 하기 위한 수동으로 트리거된 프로세스를 제공 하는 클라우드 간 솔루션을 만드는 방법을 알아봅니다. 또한 부하 상태에서 유연 하 고 확장 가능한 클라우드 유틸리티를 사용 하 여 traffic manager를 통해 자동 크기 조정을 사용 하는 방법을 알아봅니다.
