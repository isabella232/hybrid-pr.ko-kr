---
title: Azure 및 Azure Stack Hub의 하이브리드 릴레이 패턴
description: Azure의 하이브리드 릴레이 패턴과 Azure Stack 허브를 사용 하 여 방화벽으로 보호 되는에 지 리소스에 연결 합니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 03b20a20a04f620c977fb20e1ea26f5982e42721
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911148"
---
# <a name="hybrid-relay-pattern"></a>하이브리드 릴레이 패턴

하이브리드 릴레이 패턴 및 Azure Relay를 사용 하 여 방화벽으로 보호 되는에 지 리소스 또는 장치에 연결 하는 방법을 알아봅니다.

## <a name="context-and-problem"></a>컨텍스트 및 문제점

Edge 장치는 종종 회사 방화벽이 나 NAT 장치 뒤에 있습니다. 보안은 안전 하지만 다른 회사 네트워크의 공용 클라우드 또는에 지 장치와 통신 하지 못할 수 있습니다. 공용 클라우드의 사용자에 게 특정 포트 및 기능을 안전 하 게 노출 하는 것이 필요할 수 있습니다.

## <a name="solution"></a>해결 방법

하이브리드 릴레이 패턴은 Azure Relay를 사용 하 여 직접 통신할 수 없는 두 끝점 간에 Websocket 터널을 설정 합니다. 온-프레미스에는 없지만 온-프레미스 끝점에 연결 해야 하는 장치는 공용 클라우드의 끝점에 연결 됩니다. 이 끝점은 보안 채널을 통해 미리 정의 된 경로에서 트래픽을 리디렉션합니다. 온-프레미스 환경 내의 끝점은 트래픽을 수신 하 고 올바른 대상으로 라우팅합니다.

![하이브리드 릴레이 패턴 솔루션 아키텍처](media/pattern-hybrid-relay/solution-architecture.png)

하이브리드 릴레이 패턴의 작동 원리는 다음과 같습니다.

1. 장치는 미리 정의 된 포트에서 Azure의 VM (가상 머신)에 연결 됩니다.
2. 트래픽이 Azure의 Azure Relay 전달 됩니다.
3. Azure Relay에 대 한 장기 연결을 이미 설정한 Azure Stack 허브의 VM은 트래픽을 수신 하 고 대상에 전달 합니다.
4. 온-프레미스 서비스 또는 끝점에서 요청을 처리 합니다.

## <a name="components"></a>구성 요소

이 솔루션은 다음 구성 요소를 사용 합니다.

| 계층 | 구성 요소 | Description |
|----------|-----------|-------------|
| Azure | Azure VM | Azure VM은 온-프레미스 리소스에 대해 공개적으로 액세스할 수 있는 끝점을 제공 합니다. |
| | Azure Relay | [Azure Relay](/azure/azure-relay/) 은 Azure vm 및 AZURE STACK 허브 vm 간의 터널 및 연결을 유지 관리 하기 위한 인프라를 제공 합니다.|
| Azure Stack 허브 | Compute | Azure Stack 허브 VM은 하이브리드 릴레이 터널의 서버 쪽을 제공 합니다. |
| | Storage | Azure Stack 허브에 배포 된 AKS 엔진 클러스터는 Face API 컨테이너를 실행할 수 있는 확장 가능한 복원 엔진을 제공 합니다.|

## <a name="issues-and-considerations"></a>문제 및 고려 사항

이 솔루션을 구현 하는 방법을 결정할 때 다음 사항을 고려 하십시오.

### <a name="scalability"></a>확장성

이 패턴은 클라이언트와 서버에서 1:1 포트 매핑만 허용 합니다. 예를 들어, 포트 80가 Azure 끝점의 한 서비스에 대해 터널링 되는 경우 다른 서비스에 사용할 수 없습니다. 포트 매핑을 적절 하 게 계획 해야 합니다. 트래픽을 처리 하려면 Azure Relay 및 Vm의 크기를 적절히 조정 해야 합니다.

### <a name="availability"></a>가용성

이러한 터널 및 연결은 중복 되지 않습니다. 고가용성을 보장 하기 위해 오류 검사 코드를 구현할 수 있습니다. 또 다른 옵션은 부하 분산 장치 뒤에 Azure Relay 연결 된 Vm의 풀을 포함 하는 것입니다.

### <a name="manageability"></a>관리 효율

이 솔루션은 많은 장치 및 위치에 걸쳐 있을 수 있으므로 다루기가 어려울 수 있습니다. Azure의 IoT 서비스는 자동으로 새 위치 및 장치를 온라인 상태로 전환 하 고 최신 상태로 유지할 수 있습니다.

### <a name="security"></a>보안

표시 된 것 처럼이 패턴을 사용 하면에 지에서 내부 장치의 포트에 무제한적인 액세스할 수 있습니다. 내부 장치 또는 하이브리드 릴레이 끝점 앞에 있는 서비스에 인증 메커니즘을 추가 하는 것이 좋습니다.

## <a name="next-steps"></a>다음 단계

이 문서에서 소개 하는 항목에 대 한 자세한 내용은 다음을 참조 하세요.

- 이 패턴은 Azure Relay를 사용 합니다. 자세한 내용은 [Azure Relay 설명서](/azure/azure-relay/)를 참조 하세요.
- 모범 사례에 대 한 자세한 내용을 알아보고 추가 질문에 대 한 답변을 얻으려면 [하이브리드 응용 프로그램 디자인 고려 사항](overview-app-design-considerations.md) 을 참조 하세요.
- 제품 및 솔루션의 전체 포트폴리오에 대해 자세히 알아보려면 [Azure Stack 제품군 및 솔루션](/azure-stack) 을 참조 하세요.

솔루션 예제를 테스트할 준비가 되 면 [하이브리드 릴레이 솔루션 배포 가이드](https://aka.ms/hybridrelaydeployment)를 계속 진행 합니다. 배포 가이드에서는 구성 요소를 배포 및 테스트 하는 방법에 대 한 단계별 지침을 제공 합니다.