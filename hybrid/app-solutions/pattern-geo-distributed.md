---
title: Azure Stack Hub의 지역 분산 앱 패턴
description: Azure 및 Azure Stack Hub를 사용하는 인텔리전트 에지를 위한 지역 분산 앱 패턴에 대해 알아보세요.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 3c839d9bf3b6c3e1ff50cc695fd5f1a1127793d2
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281230"
---
# <a name="geo-distributed-app-pattern"></a>지역 분산 앱 패턴

여러 지역에 앱 엔드포인트를 제공하고 위치 및 규정 준수 요구 사항에 따라 사용자 트래픽을 라우팅하는 방법을 알아봅니다.

## <a name="context-and-problem"></a>컨텍스트 및 문제점

광범위한 지역에 걸쳐 있는 조직은 국경을 넘어 사용자, 위치 및 디바이스에 필요한 수준의 보안, 규정 준수 및 성능을 보장하면서 데이터를 안전하고 정확하게 배포하고 액세스할 수 있도록 노력하고 있습니다.

## <a name="solution"></a>해결 방법

Azure Stack Hub 지리적 트래픽 라우팅 패턴 또는 지역 분산 앱을 사용하면 다양한 메트릭에 따라 트래픽을 특정 엔드포인트로 보낼 수 있습니다. 지리적 기반 라우팅 및 엔드포인트 구성을 사용하여 Traffic Manager를 만들면 지역 요구 사항, 기업 및 국제 규정, 데이터 요구 사항에 따라 트래픽을 엔드포인트로 라우팅합니다.

![지역 분산 패턴](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a>구성 요소

### <a name="outside-the-cloud"></a>클러스터 외부

#### <a name="traffic-manager"></a>Traffic Manager

다이어그램에서 Traffic Manager는 퍼블릭 클라우드 외부에 있지만, 로컬 데이터 센터와 퍼블릭 클라우드 모두에서 트래픽을 조정할 수 있어야 합니다. 분산 장치는 트래픽을 지리적 위치로 라우팅합니다.

#### <a name="domain-name-system-dns"></a>DNS(Domain Name System)

Domain Name System, 즉 DNS는 웹 사이트 또는 서비스 이름을 해당 IP 주소로 변환(또는 확인)합니다.

### <a name="public-cloud"></a>퍼블릭 클라우드

#### <a name="cloud-endpoint"></a>클라우드 엔드포인트

공용 IP 주소는 트래픽 관리자를 통해 들어오는 트래픽을 퍼블릭 클라우드 앱 리소스 엔드포인트로 라우팅하는 데 사용됩니다.  

### <a name="local-clouds"></a>로컬 클라우드

#### <a name="local-endpoint"></a>로컬 엔드포인트

공용 IP 주소는 트래픽 관리자를 통해 들어오는 트래픽을 퍼블릭 클라우드 앱 리소스 엔드포인트로 라우팅하는 데 사용됩니다.

## <a name="issues-and-considerations"></a>문제 및 고려 사항

이 패턴을 구현할 방법을 결정할 때 다음 사항을 고려하세요.

### <a name="scalability"></a>확장성

패턴은 트래픽 증가량을 충족하도록 크기를 조정하는 대신 지리적 트래픽 라우팅을 처리합니다. 그러나 이 패턴을 다른 Azure 및 온-프레미스 솔루션과 결합할 수 있습니다. 예를 들어 이 패턴은 클라우드 간 크기 조정 패턴과 함께 사용할 수 있습니다.

### <a name="availability"></a>가용성

로컬로 배포된 앱이 온-프레미스 하드웨어 구성 및 소프트웨어 배포를 통해 고가용성으로 구성되도록 합니다.

### <a name="manageability"></a>관리 효율

패턴은 환경 간에 원활한 관리 및 친숙한 인터페이스를 보장합니다.

## <a name="when-to-use-this-pattern"></a>이 패턴을 사용해야 하는 경우

- 내 조직의 해외 지사는 맞춤형 현지 보안 및 배포 정책이 필요합니다.
- 내 조직의 각 사무소는 직원, 비즈니스 및 시설 데이터를 수집하며, 현지 규정 및 표준 시간대에 따라 보고 활동을 수행해야 합니다.
- 매우 높은 부하 요구 사항을 처리할 수 있도록 단일 지역과 여러 지역에 여러 앱을 배포하는 수평적 앱 스케일 아웃을 통해 대규모 요구 사항을 충족합니다.
- 단일 지역 운영 중단 시에도 앱은 클라이언트 요청에 대한 가용성과 응답성이 높아야 합니다.

## <a name="next-steps"></a>다음 단계

이 문서에서 소개하는 항목에 대한 자세한 내용은 다음을 참조하세요.

- 이 DNS 기반 트래픽 부하 분산 장치의 작동 방식에 대해 자세히 알아보려면 [Azure Traffic Manager 개요](/azure/traffic-manager/traffic-manager-overview)를 참조하세요.
- 모범 사례에 대해 자세히 알아보고 추가 질문에 대한 답변을 얻으려면 [하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md)을 참조하세요.
- 제품 및 솔루션의 전체 포트폴리오에 대해 자세히 알아보려면 [Azure Stack 제품군 및 솔루션](/azure-stack)을 참조하세요.

솔루션 예제를 테스트할 준비가 되면 [지역 분산 앱 솔루션 배포 가이드](/azure/architecture/hybrid/deployments/solution-deployment-guide-geo-distributed)를 계속 진행하세요. 배포 가이드에서는 해당 구성 요소를 배포 및 테스트하는 방법에 대한 단계별 지침을 제공합니다. 지역 분산 앱 패턴을 사용하여 다양한 메트릭에 따라 특정 엔드포인트로 트래픽을 전달하는 방법을 알아봅니다. 지리 기반 라우팅 및 엔드포인트 구성을 사용하여 Traffic Manager 프로필을 만들면 지역별 요구 사항, 회사 및 국제 규정, 데이터 요구 사항에 따라 정보가 엔드포인트로 라우팅됩니다.