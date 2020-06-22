---
title: Azure 및 Azure Stack Hub를 사용 하는 Footfall 검색 패턴
description: Azure 및 Azure Stack Hub를 사용 하 여 소매 매장 트래픽을 분석 하는 AI 기반 footfall 검색 솔루션을 구현 하는 방법을 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 10/31/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 10/31/2019
ms.openlocfilehash: 0bf07bb38537f530a0adb3569c43d53af13b8d56
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911225"
---
# <a name="footfall-detection-pattern"></a>Footfall 감지 패턴

이 패턴은 소매 상점에서 방문자 트래픽을 분석 하기 위한 AI 기반 footfall 검색 솔루션을 구현 하는 데 대 한 개요를 제공 합니다. 이 솔루션은 Azure, Azure Stack Hub 및 Custom Vision AI Dev Kit를 사용 하 여 실제 작업에서 통찰력을 생성 합니다.

## <a name="context-and-problem"></a>컨텍스트 및 문제점

Contoso store는 고객이 현재 제품을 받는 방법에 대 한 정보를 상점 레이아웃에 대 한 정보를 얻고 싶습니다. 모든 섹션에 직원을 배치 하는 것은 할 수 없으며 분석가 팀이 전체 상점의 카메라 푸티지를 검토 하는 것은 비효율적입니다. 또한 분석을 위해 모든 카메라에서 클라우드로 비디오를 스트림 하는 데 충분 한 대역폭이 없는 저장소도 있습니다.

Contoso는 표시 및 제품을 저장 하는 고객의 인구 통계, 충성도 및 반응을 확인 하는 데 있어 사용자의 정보를 파악 하기 쉬운 방식으로 찾고 싶습니다.

## <a name="solution"></a>해결 방법

이 소매점 분석 패턴은 계층화 된 접근 방식을 사용 하 여에 지를 추론 합니다. Custom Vision AI Dev Kit를 사용 하 여 인간 얼굴의 이미지만 Azure Cognitive Services를 실행 하는 개인 Azure Stack 허브로 전송 됩니다. 익명화 집계 된 데이터는 Power BI의 모든 매장 및 시각화에서 집계를 위해 Azure에 전송 됩니다. Edge와 공용 클라우드를 결합 하면 Contoso에서 최신 AI 기술을 활용할 수 있을 뿐만 아니라 회사 정책에 부합 하 고 고객의 개인 정보를 존중 합니다.

[![Footfall 검색 패턴 솔루션](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)

다음은 솔루션의 작동 방식에 대 한 요약입니다.

1. Custom Vision AI Dev Kit는 IoT Edge 런타임과 ML 모델을 설치 하는 IoT Hub에서 구성을 가져옵니다.
2. 모델에서 사람을 볼 경우 그림을 사용 하 여 Azure Stack Hub blob 저장소에 업로드 합니다.
3. Blob service는 Azure Stack 허브에서 Azure 함수를 트리거합니다.
4. Azure 함수는 Face API 있는 컨테이너를 호출 하 여 이미지에서 인구 통계 및 emotion 데이터를 가져옵니다.
5. 데이터는 익명화 Azure Event Hubs 클러스터로 전송 됩니다.
6. Event Hubs 클러스터는 Stream Analytics 데이터를 푸시합니다.
7. Stream Analytics는 데이터를 집계 하 여 Power BI에 푸시합니다.

## <a name="components"></a>구성 요소

이 솔루션은 다음 구성 요소를 사용 합니다.

| 계층 | 구성 요소 | Description |
|----------|-----------|-------------|
| 매장 내 하드웨어 | [Custom Vision AI Dev Kit](https://azure.github.io/Vision-AI-DevKit-Pages/) | 는 분석을 위해 사람들의 이미지를 캡처하는 로컬 ML 모델을 사용 하 여 매장 내 필터링을 제공 합니다. IoT Hub를 통해 안전 하 게 프로 비전 되 고 업데이트 됩니다.<br><br>|
| Azure | [Azure Event Hubs](/azure/event-hubs/) | Azure Event Hubs는 Azure Stream Analytics와 깔끔하게 통합 되는 수집 익명화 데이터를 위한 확장 가능한 플랫폼을 제공 합니다. |
|  | [Azure Stream Analytics](/azure/stream-analytics/) | Azure Stream Analytics 작업은 익명화 데이터를 집계 하 고 시각화를 위해 15 초 windows로 그룹화 합니다. |
|  | [Microsoft Power BI](https://powerbi.microsoft.com/) | Power BI Azure Stream Analytics의 출력을 볼 수 있는 사용 하기 쉬운 대시보드 인터페이스를 제공 합니다. |
| Azure Stack 허브 | [App Service](/azure-stack/operator/azure-stack-app-service-overview.md) | RP (App Service 리소스 공급자)는 웹 앱/a p i 및 함수에 대 한 호스팅 및 관리 기능을 포함 하 여에 지 구성 요소에 대 한 기반을 제공 합니다. |
| | AKS (Azure Kubernetes Service [) 엔진](https://github.com/Azure/aks-engine) 클러스터 | Azure Stack Hub에 배포 된 AKS RP는 Face API 컨테이너를 실행할 수 있는 확장 가능한 복원 력 엔진을 제공 합니다. |
| | Azure Cognitive Services [Face API 컨테이너](/azure/cognitive-services/face/face-how-to-install-containers)| Face API 컨테이너가 있는 Azure Cognitive Services RP는 Contoso의 개인 네트워크에 대 한 인구 통계, emotion 및 고유 방문자 검색을 제공 합니다. |
| | Blob Storage | AI Dev Kit에서 캡처된 이미지는 Azure Stack 허브의 blob 저장소에 업로드 됩니다. |
| | Azure 기능 | Azure Stack Hub에서 실행 되는 Azure 함수는 blob storage에서 입력을 받고 Face API와의 상호 작용을 관리 합니다. Azure에 있는 Event Hubs 클러스터로 익명화 데이터를 내보냅니다.<br><br>|

## <a name="issues-and-considerations"></a>문제 및 고려 사항

이 솔루션을 구현 하는 방법을 결정할 때 다음 사항을 고려 하십시오.

### <a name="scalability"></a>확장성

이 솔루션을 여러 카메라 및 위치에서 확장할 수 있도록 하려면 모든 구성 요소에서 증가 된 부하를 처리할 수 있는지 확인 해야 합니다. 다음과 같은 작업을 수행 해야 할 수 있습니다.

- Stream Analytics 스트리밍 단위의 수를 늘립니다.
- Face API 배포를 확장 합니다.
- Event Hubs 클러스터 처리량을 늘립니다.
- 극단적인 경우에는 Azure Functions에서 가상 머신으로 마이그레이션해야 할 수 있습니다.

### <a name="availability"></a>가용성

이 솔루션은 계층화 되므로 네트워킹 또는 전원 오류를 처리 하는 방법을 고려 하는 것이 중요 합니다. 비즈니스 요구 사항에 따라 이미지를 로컬로 캐시 한 다음 연결이 반환 될 때 Azure Stack 허브로 전달 하는 메커니즘을 구현할 수 있습니다. 위치가 충분히 크면 Face API 컨테이너를 사용 하 여 Data Box Edge를 해당 위치에 배포 하는 것이 더 나을 수 있습니다.

### <a name="manageability"></a>관리 효율

이 솔루션은 많은 장치 및 위치에 걸쳐 있을 수 있으므로 다루기가 어려울 수 있습니다. [Azure의 IoT 서비스](/azure/iot-fundamentals/) 를 사용 하 여 자동으로 새 위치 및 장치를 온라인 상태로 전환 하 고 최신 상태로 유지할 수 있습니다.

### <a name="security"></a>보안

이 솔루션은 고객 이미지를 캡처하여 보안을 가장 중요 하 게 고려 합니다. 모든 저장소 계정에 적절 한 액세스 정책을 사용 하 여 보안을 설정 하 고 키를 정기적으로 회전 해야 합니다. 저장소 계정 및 Event Hubs 회사 및 정부 개인 정보 규정을 충족 하는 보존 정책을 보유 하 고 있는지 확인 하세요. 또한 사용자 액세스 수준도 계층으로 지정 해야 합니다. 계층화를 사용 하면 사용자가 자신의 역할에 필요한 데이터에만 액세스할 수 있습니다.

## <a name="next-steps"></a>다음 단계

이 문서에서 소개 하는 항목에 대 한 자세한 내용은 다음을 참조 하세요.

- Footfall 검색 패턴에서 활용 하는 [계층화 된 데이터 패턴](https://aka.ms/tiereddatadeploy)을 참조 하세요.
- 사용자 지정 비전을 사용 하는 방법에 대해 자세히 알아보려면 [CUSTOM VISION AI Dev Kit](https://azure.github.io/Vision-AI-DevKit-Pages/) 를 참조 하세요. 

솔루션 예제를 테스트할 준비가 되 면 [Footfall 검색 배포 가이드](solution-deployment-guide-retail-footfall-detection.md)를 계속 진행 합니다. 배포 가이드에서는 구성 요소를 배포 및 테스트 하는 방법에 대 한 단계별 지침을 제공 합니다.
