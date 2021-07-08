---
title: Azure 및 Azure Stack Edge를 사용하여 품절 검색
description: Azure 및 Azure Stack Edge 서비스를 사용하여 품절 검색을 구현하는 방법을 알아보세요.
author: BryanLa
ms.topic: article
ms.date: 05/24/2021
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 05/24/2021
ms.openlocfilehash: b25a6391c4e64fa7018031bac4fb7d098c56b529
ms.sourcegitcommit: cf2c4033d1b169f5b63980ce1865281366905e2e
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/25/2021
ms.locfileid: "110343878"
---
# <a name="out-of-stock-detection-at-the-edge-pattern"></a>에지 패턴에서 품절 검색

이 패턴은 Azure Stack Edge 또는 Azure IoT Edge 디바이스 및 네트워크 카메라를 사용하여 진열대에 품절된 품목이 있는지 확인하는 방법을 보여줍니다.

## <a name="context-and-problem"></a>컨텍스트 및 문제점

실제 소매 매장에서는 고객이 찾는 품목이 진열대에 없기 때문에 매출이 감소합니다. 그러나 해당 품목은 매장 뒤에 있으며 재입고되지 않았을 수 있습니다. 매장은 직원을 보다 효율적으로 활용하고 품목을 재입고해야 할 때 자동으로 알림을 받고자 합니다.

## <a name="solution"></a>솔루션

솔루션 예제는 각 매장에서 Azure Stack Edge와 같은 에지 디바이스를 사용하여 매장의 카메라의 데이터를 효율적으로 처리합니다. 이러한 최적화된 디자인을 통해 관련 이벤트 및 이미지만 클라우드로 보냅니다. 이 디자인은 대역폭, 스토리지 공간을 절약하고 고객의 개인 정보 보호를 보장합니다. 각 카메라에서 프레임을 읽을 때 ML 모델은 이미지를 처리하고 품절 영역을 반환합니다. 이미지 및 품절 영역은 로컬 웹 앱에 표시됩니다. 이 데이터는 Power BI에서 인사이트를 표시하기 위해 Time Series Insight 환경으로 보낼 수 있습니다.

![에지 솔루션 아키텍처의 품절](media/pattern-out-of-stock-at-edge/solution-architecture.png)

솔루션의 작동 방식은 다음과 같습니다.

1. HTTP 또는 RTSP를 통해 네트워크 카메라에서 이미지를 캡처합니다.
2. 이미지 크기가 조정되고 추론 드라이버로 보내지며, 추론 드라이버는 ML 모델과 통신하여 품절 이미지가 있는지 확인합니다.
3. ML 모델은 모든 품절 영역을 반환합니다.
4. 추론 드라이버는 원시 이미지를 Blob(지정된 경우)에 업로드하고 모델의 결과를 디바이스의 Azure IoT Hub 및 경계 상자 프로세서로 보냅니다.
5. 경계 상자 프로세서는 이미지에 경계 상자를 추가하고 메모리 내 데이터베이스에서 이미지 경로를 캐시합니다.
6. 웹앱은 이미지를 쿼리하고 받은 순서대로 표시합니다.
7. IoT Hub의 메시지는 Time Series Insights에서 집계됩니다.
8. Power BI는 Time Series Insights의 데이터를 사용하여 시간이 지남에 따라 품절된 품목에 대한 대화형 보고서를 표시합니다.


## <a name="components"></a>구성 요소

이 솔루션은 다음과 같은 구성 요소를 사용합니다.

| 계층 | 구성 요소 | 설명 |
|----------|-----------|-------------|
| 온-프레미스 하드웨어 | 네트워크 카메라 | 추론을 위한 이미지를 제공하는 HTTP 또는 RTSP 피드가 있는 네트워크 카메라가 필요합니다. |
| Azure | Azure IoT Hub | [Azure IoT Hub](/azure/iot-hub/)는 에지 디바이스에 대한 디바이스 프로비저닝 및 메시지를 처리합니다. |
|  | Azure Time Series Insights | [Azure Time Series Insights](/azure/time-series-insights/)는 시각화를 위해 IoT Hub의 메시지를 저장합니다. |
|  | Power BI | [Microsoft Power BI](https://powerbi.microsoft.com/)는 품절 이벤트에 대한 비즈니스 중심 보고서를 제공합니다. Power BI는 Azure Stream Analytics의 출력을 볼 수 있는 간편한 대시보드 인터페이스를 제공합니다. |
| Azure Stack Edge 또는<br>Azure IoT Edge 디바이스 | Azure IoT Edge | [Azure IoT Edge](/azure/iot-edge/)는 온-프레미스 컨테이너의 런타임을 오케스트레이션하고 디바이스 관리 및 업데이트를 처리합니다.|
| | Azure Project Brainwave | Azure Stack Edge 디바이스에서 [Project Brainwave](https://blogs.microsoft.com/ai/build-2018-project-brainwave/)는 FPGA(Field-Programmable Gate Arrays)를 사용하여 ML 추론을 가속화합니다.|

## <a name="issues-and-considerations"></a>문제 및 고려 사항

이 솔루션을 구현할 방법을 결정할 때 다음 사항을 고려하세요.

### <a name="scalability"></a>확장성

대부분의 기계 학습 모델은 제공된 하드웨어에 따라 초당 특정 프레임 수에서만 실행할 수 있습니다. ML 파이프라인이 백업되지 않도록 카메라의 최적 샘플링 속도를 결정합니다. 하드웨어 유형에 따라 카메라 수와 프레임 속도가 달라집니다.

### <a name="availability"></a>가용성

에지 디바이스의 연결이 끊어진 경우 발생할 수 있는 상황을 고려하는 것이 중요합니다. Time Series Insights 및 Power BI 대시보드에서 손실될 수 있는 데이터를 고려합니다. 제공된 예제 솔루션은 고가용성으로 디자인되지 않았습니다.

### <a name="manageability"></a>관리 효율

이 솔루션은 여러 디바이스 및 위치에 걸쳐 있을 수 있으므로 다루기가 어려울 수 있습니다. Azure의 IoT 서비스는 자동으로 새 위치 및 디바이스를 온라인 상태로 전환하고 최신 상태로 유지할 수 있습니다. 또한 적절한 데이터 거버넌스 절차를 따라야 합니다.

### <a name="security"></a>보안

이 패턴은 잠재적으로 중요한 데이터를 처리합니다. 키가 정기적으로 순환되고 Azure Storage 계정 및 로컬 공유에 대한 권한이 올바르게 설정되었는지 확인합니다.

## <a name="next-steps"></a>다음 단계

이 문서에서 소개하는 항목에 대한 자세한 내용은 다음을 참조하세요.
- 이 패턴에는 [Azure IoT Edge](/azure/iot-edge/), [Azure IoT Hub](/azure/iot-hub/) 및 [Azure Time Series Insights](/azure/time-series-insights/)를 비롯한 여러 IoT 관련 서비스가 사용됩니다.
- Microsoft Project Brainwave에 대한 자세한 내용은 [블로그 공지](https://blogs.microsoft.com/ai/build-2018-project-brainwave/)를 참조하고 [Azure Accelerated Machine Learning with Project Brainwave 비디오](https://www.youtube.com/watch?v=DJfMobMjCX0)를 확인하세요.
- 모범 사례에 대해 자세히 알아보고 추가 질문에 대한 답변을 얻으려면 [하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md)을 참조하세요.
- 제품 및 솔루션의 전체 포트폴리오에 대해 자세히 알아보려면 [Azure Stack 제품군 및 솔루션](/azure-stack)을 참조하세요.

솔루션 예제를 테스트할 준비가 되면 [에지 ML 추론 솔루션 배포 가이드](https://aka.ms/edgeinferencingdeploy)를 계속 진행하세요. 배포 가이드에서는 해당 구성 요소를 배포 및 테스트하는 방법에 대한 단계별 지침을 제공합니다.
