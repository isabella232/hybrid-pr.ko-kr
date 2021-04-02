---
title: Azure 및 Azure Stack Hub를 사용하는 고객 수 검색 패턴
description: Azure 및 Azure Stack Hub를 사용하여 소매점 트래픽을 분석하는 AI 기반 고객 수 검색 솔루션을 구현하는 방법을 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 10/31/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 10/31/2019
ms.openlocfilehash: 866557ec3af2337e9f034da84cf417675508563b
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895332"
---
# <a name="footfall-detection-pattern"></a>고객 수 검색 패턴

이 패턴은 소매점에서 방문자 트래픽을 분석하기 위한 AI 기반 고객 수 검색 솔루션을 구현하는 방법에 대한 개요를 제공합니다. 이 솔루션은 Azure, Azure Stack Hub 및 Custom Vision AI Dev Kit를 사용하여 실제 작업에서 인사이트를 생성합니다.

## <a name="context-and-problem"></a>컨텍스트 및 문제점

Contoso 매장에서는 고객이 매장 레이아웃과 관련하여 현재 제품을 어떻게 받아들이는지에 대한 인사이트를 얻으려고 합니다. 모든 섹션에 직원을 배치할 수는 없으며, 분석가 팀이 전체 매장의 카메라 화면을 검토하는 것은 비효율적입니다. 또한 모든 매장에서 분석을 위해 모든 카메라의 영상을 클라우드로 스트리밍하는 데 필요한 대역폭이 부족합니다.

Contoso에서는 개인 정보를 존중하면서 지나치게 간섭하지 않는 방식으로 고객의 인구 통계, 충성도, 매장 진열 및 제품에 대한 반응을 확인하고자 합니다.

## <a name="solution"></a>솔루션

이 소매 분석 패턴은 에지에 계층화된 추론 방식을 사용합니다. Custom Vision AI Dev Kit를 사용하면 사람의 얼굴이 찍힌 이미지만 Azure Cognitive Services를 실행하는 개인 Azure Stack Hub로 전송됩니다. 모든 매장을 집계하고 Power BI에 시각화하기 위해 익명화되어 집계된 데이터가 Azure에 전송됩니다. Edge와 퍼블릭 클라우드를 결합하면 Contoso에서 최신 AI 기술을 활용할 수 있을 뿐만 아니라 회사 정책을 준수하고 고객의 개인 정보를 존중할 수 있습니다.

[![고객 수 검색 패턴 솔루션](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)

이 솔루션의 작동 방식을 요약하면 다음과 같습니다.

1. Custom Vision AI Dev Kit가 IoT Hub에서 구성을 가져오고 IoT Edge 런타임과 ML 모델을 설치합니다.
2. 이 모델에서 사람을 감지할 경우 사진을 찍어 Azure Stack Hub Blob Storage에 업로드합니다.
3. Blob service가 Azure Stack Hub에서 Azure 함수를 트리거합니다.
4. Azure 함수가 Face API로 컨테이너를 호출하여 이미지에서 인구 통계 및 감정 데이터를 가져옵니다.
5. 데이터가 익명화되고 Azure Event Hubs 클러스터로 전송됩니다.
6. Event Hubs 클러스터가 Stream Analytics에 데이터를 푸시합니다.
7. Stream Analytics가 데이터를 집계하여 Power BI에 푸시합니다.

## <a name="components"></a>구성 요소

이 솔루션은 다음과 같은 구성 요소를 사용합니다.

| 계층 | 구성 요소 | 설명 |
|----------|-----------|-------------|
| 매장 내 하드웨어 | [Custom Vision AI Dev Kit](https://azure.github.io/Vision-AI-DevKit-Pages/) | 분석을 위해 사람들의 영상만 캡처하는 로컬 ML 모델을 사용하여 매장 내 필터링을 제공합니다. IoT Hub를 통해 안전하게 프로비저닝되고 업데이트됩니다.<br><br>|
| Azure | [Azure Event Hubs](/azure/event-hubs/) | Azure Event Hubs는 익명화된 데이터를 수집하기 위한 확장 가능한 플랫폼을 제공하며, 이는 Azure Stream Analytics와 깔끔하게 통합됩니다. |
|  | [Azure Stream Analytics](/azure/stream-analytics/) | Azure Stream Analytics 작업은 익명화된 데이터를 집계하고 시각화를 위해 15초 간격으로 그룹화합니다. |
|  | [Microsoft Power BI](https://powerbi.microsoft.com/) | Power BI는 Azure Stream Analytics의 출력을 볼 수 있는 간편한 대시보드 인터페이스를 제공합니다. |
| Azure Stack Hub | [App Service](/azure-stack/operator/azure-stack-app-service-overview) | App Service 리소스 공급자(RP)는 웹앱/API 및 Functions에 대한 호스팅 및 관리 기능을 포함하여 에지 구성 요소에 대한 기반을 제공합니다. |
| | Azure Kubernetes Service [(AKS) 엔진](https://github.com/Azure/aks-engine) 클러스터 | AKS-Engine 클러스터가 Azure Stack Hub에 배포된 AKS RP는 Face API 컨테이너를 실행하기 위해 확장성과 복원력이 있는 엔진을 제공합니다. |
| | Azure Cognitive Services [Face API 컨테이너](/azure/cognitive-services/face/face-how-to-install-containers)| Face API 컨테이너가 있는 Azure Cognitive Services RP는 Contoso의 개인 네트워크에 대한 인구 통계, 감정 및 고유한 방문자 검색을 제공합니다. |
| | Blob Storage | AI Dev Kit에서 캡처된 이미지는 Azure Stack Hub의 Blob Storage에 업로드됩니다. |
| | Azure Functions | Azure Stack Hub에서 실행되는 Azure 함수가 Blob Storage에서 입력을 받고 Face API와의 상호 작용을 관리합니다. Azure에 있는 Event Hubs 클러스터로 익명화된 데이터를 내보냅니다.<br><br>|

## <a name="issues-and-considerations"></a>문제 및 고려 사항

이 솔루션을 구현할 방법을 결정할 때 다음 사항을 고려하세요.

### <a name="scalability"></a>확장성

이 솔루션을 여러 카메라 및 위치에서 확장할 수 있도록 하려면 모든 구성 요소에서 증가된 부하를 처리할 수 있는지 확인해야 합니다. 다음과 같은 작업을 수행해야 할 수 있습니다.

- Stream Analytics 스트리밍 단위의 수를 늘립니다.
- Face API 배포를 스케일 아웃합니다.
- Event Hubs 클러스터 처리량을 늘립니다.
- 극단적인 경우에는 Azure Functions에서 가상 머신으로 마이그레이션해야 할 수 있습니다.

### <a name="availability"></a>가용성

이 솔루션은 계층화되어 있으므로 네트워킹 또는 전원 오류를 처리하는 방법을 고려해야 합니다. 비즈니스 요구 사항에 따라 이미지를 로컬로 캐시한 다음, 다시 연결될 때 Azure Stack Hub로 전달하는 메커니즘을 구현하는 것이 좋습니다. 위치가 충분히 크면 Face API 컨테이너가 포함된 Data Box Edge를 해당 위치에 배포하는 것이 더 나을 수 있습니다.

### <a name="manageability"></a>관리 효율

이 솔루션은 여러 디바이스 및 위치에 걸쳐 있을 수 있으므로 다루기가 어려울 수 있습니다. [Azure의 IoT 서비스](/azure/iot-fundamentals/)를 사용하여 자동으로 새 위치 및 디바이스를 온라인 상태로 전환하고 최신 상태로 유지할 수 있습니다.

### <a name="security"></a>보안

이 솔루션은 고객 영상을 캡처하므로 보안을 최우선으로 고려합니다. 모든 스토리지 계정을 적절한 액세스 정책으로 보호하고 키를 정기적으로 순환시켜야 합니다. 스토리지 계정 및 Event Hubs에 회사 및 정부의 개인 정보 규정을 충족하는 보존 정책이 있는지 확인하세요. 또한 사용자 액세스 수준도 계층화해야 합니다. 계층화를 사용하면 사용자가 자신의 역할을 수행하는 데 필요한 데이터에만 액세스할 수 있습니다.

## <a name="next-steps"></a>다음 단계

이 문서에서 소개하는 항목에 대해 자세히 알아보려면 다음을 수행합니다.

- 고객 수 검색 패턴에서 활용하는 [계층화된 데이터 패턴](https://aka.ms/tiereddatadeploy)을 참조합니다.
- [Custom Vision AI Dev Kit](https://azure.github.io/Vision-AI-DevKit-Pages/)를 참조하여 사용자 지정 비전을 사용하는 방법에 대해 자세히 알아봅니다. 

솔루션 예제를 테스트할 준비가 되면 [고객 수 검색 배포 가이드](solution-deployment-guide-retail-footfall-detection.md)를 계속 진행하세요. 배포 가이드에서는 해당 구성 요소를 배포 및 테스트하는 방법에 대한 단계별 지침을 제공합니다.
