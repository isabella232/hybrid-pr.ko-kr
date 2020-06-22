---
title: Azure Stack 허브의 지리적으로 분산 되는 앱 패턴
description: Azure 및 Azure Stack Hub를 사용 하는 지능형에 지에 대 한 지리적으로 분산 된 앱 패턴에 대해 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 1f6243927390c7a520c2607c722664b2d31fc07f
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910882"
---
# <a name="geo-distributed-app-pattern"></a>지리적으로 분산 되는 앱 패턴

여러 지역에 걸쳐 앱 끝점을 제공 하 고 위치 및 규정 준수 요구 사항에 따라 사용자 트래픽을 라우팅하는 방법을 알아봅니다.

## <a name="context-and-problem"></a>컨텍스트 및 문제점

광범위 한 지역이 있는 조직에서는 데이터에 대 한 액세스를 안전 하 고 정확 하 게 배포 하 고 사용 하도록 설정 하는 동시에 사용자, 위치 및 장치 마다 사용자, 위치 및 장치에 대해 필요한 수준의 보안, 규정 준수 및 성능을

## <a name="solution"></a>해결 방법

Azure Stack 허브 지리적 트래픽 라우팅 패턴 또는 지리적으로 분산 된 앱을 사용 하면 다양 한 메트릭에 따라 트래픽을 특정 끝점으로 보낼 수 있습니다. 지리적 기반 라우팅 및 끝점 구성을 사용 하 여 Traffic Manager를 만들면 지역 요구 사항, 회사 및 국제 규정 및 데이터 요구 사항에 따라 끝점으로 트래픽을 라우팅합니다.

![지리적으로 분산 되는 패턴](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a>구성 요소

### <a name="outside-the-cloud"></a>클라우드 외부

#### <a name="traffic-manager"></a>Traffic Manager

다이어그램에서 Traffic Manager 공용 클라우드 외부에 위치 하지만 로컬 데이터 센터와 공용 클라우드 모두에서 트래픽을 조정할 수 있어야 합니다. 부하 분산 장치는 트래픽을 지리적 위치로 라우팅합니다.

#### <a name="domain-name-system-dns"></a>DNS(Domain Name System)

Domain Name System, 즉 DNS는 웹 사이트 또는 서비스 이름을 해당 IP 주소로 변환(또는 확인)합니다.

### <a name="public-cloud"></a>퍼블릭 클라우드

#### <a name="cloud-endpoint"></a>클라우드 엔드포인트

공용 IP 주소는 traffic manager를 통해 들어오는 트래픽을 공용 클라우드 앱 리소스 끝점으로 라우팅하는 데 사용 됩니다.  

### <a name="local-clouds"></a>로컬 클라우드

#### <a name="local-endpoint"></a>로컬 끝점

공용 IP 주소는 traffic manager를 통해 들어오는 트래픽을 공용 클라우드 앱 리소스 끝점으로 라우팅하는 데 사용 됩니다.

## <a name="issues-and-considerations"></a>문제 및 고려 사항

이 패턴을 구현할 방법을 결정할 때 다음 사항을 고려하세요.

### <a name="scalability"></a>확장성

패턴은 트래픽 증가량을 충족 하도록 크기를 조정 하는 대신 지리 트래픽 라우팅을 처리 합니다. 그러나이 패턴을 다른 Azure 및 온-프레미스 솔루션과 결합할 수 있습니다. 예를 들어이 패턴은 클라우드 간 배율 패턴과 함께 사용할 수 있습니다.

### <a name="availability"></a>가용성

로컬로 배포 된 앱이 온-프레미스 하드웨어 구성 및 소프트웨어 배포를 통해 고가용성으로 구성 되어 있는지 확인 합니다.

### <a name="manageability"></a>관리 효율

패턴은 환경 간에 원활한 관리 및 친숙 한 인터페이스를 보장 합니다.

## <a name="when-to-use-this-pattern"></a>이 패턴을 사용해야 하는 경우

- 조직에 사용자 지정 지역 보안 및 배포 정책을 요구 하는 국제 분기가 있습니다.
- 각 조직 사무소는 직원, 비즈니스 및 시설 데이터를 가져와서 로컬 규정과 표준 시간대에 대 한 보고 작업을 요구 합니다.
- 단일 지역 내에서 여러 개의 앱 배포를 수행 하 고 여러 지역에서 부하 요구 사항을 처리 하기 위해 여러 앱 배포를 사용 하 여 앱을 수평 확장 하 여 대규모 요구 사항을 충족할 수 있습니다.
- 단일 지역 중단이 발생 하더라도 앱은 항상 사용 가능 하 고 클라이언트 요청에 응답 해야 합니다.

## <a name="next-steps"></a>다음 단계

이 문서에서 소개 하는 항목에 대 한 자세한 내용은 다음을 참조 하세요.

- 이 DNS 기반 트래픽 부하 분산 장치 작동 방법에 대해 자세히 알아보려면 [Azure Traffic Manager 개요](/azure/traffic-manager/traffic-manager-overview) 를 참조 하세요.
- 모범 사례에 대 한 자세한 내용을 알아보고 추가 질문에 대 한 답변을 얻으려면 [하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 을 참조 하세요.
- 제품 및 솔루션의 전체 포트폴리오에 대해 자세히 알아보려면 [Azure Stack 제품군 및 솔루션](/azure-stack) 을 참조 하세요.

솔루션 예제를 테스트할 준비가 되 면 [지리적으로 분산 된 앱 솔루션 배포 가이드](solution-deployment-guide-geo-distributed.md)를 계속 진행 합니다. 배포 가이드에서는 구성 요소를 배포 및 테스트 하는 방법에 대 한 단계별 지침을 제공 합니다. 지리적으로 분산 된 응용 프로그램 패턴을 사용 하 여 다양 한 메트릭에 따라 특정 끝점으로 트래픽을 전송 하는 방법을 알아봅니다. 지리적 기반 라우팅 및 끝점 구성을 사용 하 여 Traffic Manager 프로필을 만들면 정보가 지역 요구 사항, 회사 및 국제 규정 및 데이터 요구 사항에 따라 끝점으로 라우팅됩니다.
