---
title: Azure 및 Azure Stack Edge를 사용 하 여 품절 검색 부족
description: Azure 및 Azure Stack Edge 서비스를 사용 하 여 품절 검색을 구현 하는 방법을 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 865f63bc4234e50ed169aa29cefdb1886750594c
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911190"
---
# <a name="out-of-stock-detection-at-the-edge-pattern"></a>Edge 패턴에서 품절 검색

이 패턴은 Azure Stack Edge 나 Azure IoT Edge 장치 및 네트워크 카메라를 사용 하 여 선반에 재고 항목이 있는지 여부를 확인 하는 방법을 보여 줍니다.

## <a name="context-and-problem"></a>컨텍스트 및 문제점

고객이 항목을 찾을 때 선반에 표시 되지 않기 때문에 실제 소매점 매장에는 판매량이 손실 됩니다. 그러나 항목은 저장소의 뒤에 있을 수 있으며 restocked 되지 않을 수 있습니다. 매장에서 직원 들을 보다 효율적으로 사용 하 고 항목에 restocking 필요 하면 자동으로 통보를 받고 싶습니다.

## <a name="solution"></a>해결 방법

솔루션 예제는 저장소의 카메라에서 데이터를 효율적으로 처리 하는 각 상점의 Azure Stack Edge와 같은에 지 장치를 사용 합니다. 이러한 최적화 된 설계를 통해 관련 이벤트와 이미지를 클라우드로 보낼 수 있습니다. 디자인은 대역폭, 저장소 공간을 절약 하 고 고객의 개인 정보 보호를 보장 합니다. 각 카메라에서 프레임을 읽을 때 ML 모델은 이미지를 처리 하 고 품절 영역을 모두 반환 합니다. 로컬 웹 앱에 이미지 및 재고 영역이 표시 되지 않습니다. 이 데이터를 시계열 정보 환경으로 전송 하 여 Power BI에 대 한 정보를 표시할 수 있습니다.

![Edge 솔루션 아키텍처의 품절](media/pattern-out-of-stock-at-edge/solution-architecture.png)

솔루션의 작동 원리는 다음과 같습니다.

1. 이미지는 HTTP 또는 RTSP를 통해 네트워크 카메라에서 캡처됩니다.
2. 이미지의 크기를 조정 하 고, ML 모델과 통신 하 여 재고량 이미지가 있는지 확인 하는 유추 드라이버에 전송 됩니다.
3. ML 모델은 모든 재고 영역을 반환 합니다.
4. 추론 드라이버는 원시 이미지를 blob에 업로드 하 고 (지정 된 경우), 모델의 결과를 Azure IoT Hub으로 보내고, 경계 상자 프로세서를 장치에 보냅니다.
5. 경계 상자 프로세서는 이미지에 경계 상자를 추가 하 고 메모리 내 데이터베이스에서 이미지 경로를 캐시 합니다.
6. 웹 앱은 이미지를 쿼리하고 받은 순서 대로 표시 합니다.
7. IoT Hub의 메시지는 Time Series Insights 집계 됩니다.
8. Power BI Time Series Insights의 데이터를 사용 하 여 시간별 재고 항목에 대 한 대화형 보고서를 표시 합니다.


## <a name="components"></a>구성 요소

이 솔루션은 다음 구성 요소를 사용 합니다.

| 계층 | 구성 요소 | Description |
|----------|-----------|-------------|
| 온-프레미스 하드웨어 | 네트워크 카메라 | 유추를 위한 이미지를 제공 하는 HTTP 또는 RTSP 피드가 포함 된 네트워크 카메라가 필요 합니다. |
| Azure | Azure IoT Hub | [Azure IoT Hub](/azure/iot-hub/) 는에 지 장치에 대 한 장치 프로 비전 및 메시징을 처리 합니다. |
|  | Azure Time Series Insights | [Azure Time Series Insights](/azure/time-series-insights/) 은 시각화를 위해 IoT Hub 메시지를 저장 합니다. |
|  | Power BI | [Microsoft Power BI](https://powerbi.microsoft.com/) 는 품절 이벤트의 비즈니스 중심 보고서를 제공 합니다. Power BI Azure Stream Analytics의 출력을 볼 수 있는 사용 하기 쉬운 대시보드 인터페이스를 제공 합니다. |
| Azure Stack Edge 또는<br>Azure IoT Edge 장치 | Azure IoT Edge | [Azure IoT Edge](/azure/iot-edge/) 온-프레미스 컨테이너에 대 한 런타임을 오케스트레이션 하 고 장치 관리 및 업데이트를 처리 합니다.|
| | Azure 프로젝트 brainwave | Azure Stack Edge 장치에서 [Project Brainwave](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) 는 추론 (FPGAs) 필드를 사용 하 여 ML를 가속화 합니다.|

## <a name="issues-and-considerations"></a>문제 및 고려 사항

이 솔루션을 구현 하는 방법을 결정할 때 다음 사항을 고려 하십시오.

### <a name="scalability"></a>확장성

대부분의 기계 학습 모델은 제공 된 하드웨어에 따라 초당 특정 개수의 프레임 에서만 실행할 수 있습니다. ML 파이프라인이 백업 하지 않도록 카메라에서 최적의 샘플 속도를 결정 합니다. 다른 종류의 하드웨어는 서로 다른 카메라 및 프레임 속도를 처리 합니다.

### <a name="availability"></a>가용성

에 지 장치가 연결을 해제 하는 경우 발생할 수 있는 상황을 고려 하는 것이 중요 합니다. Time Series Insights 및 Power BI 대시보드에서 손실 될 수 있는 데이터를 고려 합니다. 제공 된 예제 솔루션은 항상 사용 가능 하도록 설계 되지 않았습니다.

### <a name="manageability"></a>관리 효율

이 솔루션은 많은 장치 및 위치에 걸쳐 있을 수 있으므로 다루기가 어려울 수 있습니다. Azure의 IoT 서비스는 자동으로 새 위치 및 장치를 온라인 상태로 전환 하 고 최신 상태로 유지할 수 있습니다. 적절 한 데이터 거 버 넌 스 절차를 따라야 합니다.

### <a name="security"></a>보안

이 패턴은 잠재적으로 중요 한 데이터를 처리 합니다. 키를 정기적으로 회전 하 고 Azure Storage 계정 및 로컬 공유에 대 한 사용 권한이 올바르게 설정 되어 있는지 확인 합니다.

## <a name="next-steps"></a>다음 단계

이 문서에서 소개 하는 항목에 대 한 자세한 내용은 다음을 참조 하세요.
- [Azure IoT Edge](/azure/iot-edge/), [Azure IoT Hub](/azure/iot-hub/)및 [Azure Time Series Insights](/azure/time-series-insights/)를 포함 하 여이 패턴에서 여러 IoT 관련 서비스를 사용 합니다.
- Microsoft Project Brainwave에 대 한 자세한 내용은 [블로그 공지](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) 를 참조 하 고 [Project Brainwave 비디오를 사용 하 여 Azure 가속화 된 Machine Learning](https://www.youtube.com/watch?v=DJfMobMjCX0)를 체크 아웃 하세요.
- 모범 사례에 대 한 자세한 내용을 알아보고 추가 질문에 대 한 답변을 얻으려면 [하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 을 참조 하세요.
- 제품 및 솔루션의 전체 포트폴리오에 대해 자세히 알아보려면 [Azure Stack 제품군 및 솔루션](/azure-stack) 을 참조 하세요.

솔루션 예제를 테스트할 준비가 되 면 [분석 솔루션에 대 한 계층화 된 데이터 배포 가이드](https://aka.ms/edgeinferencingdeploy)를 계속 진행 합니다. 배포 가이드에서는 구성 요소를 배포 및 테스트 하는 방법에 대 한 단계별 지침을 제공 합니다.
