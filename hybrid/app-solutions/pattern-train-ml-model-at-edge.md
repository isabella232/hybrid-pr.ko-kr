---
title: Edge 패턴에서 기계 학습 모델 학습
description: Azure 및 Azure Stack Hub를 사용 하 여에 지에서 기계 학습 모델 학습을 수행 하는 방법에 대해 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: da1012cc8847f221de6bb540ba4e191e43920f41
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911099"
---
# <a name="train-machine-learning-model-at-the-edge-pattern"></a>Edge 패턴에서 기계 학습 모델 학습

온-프레미스에만 존재 하는 데이터에서 ML (이식 가능한 기계 학습) 모델을 생성 합니다.

## <a name="context-and-problem"></a>컨텍스트 및 문제점

대부분의 조직에서는 데이터 과학자 이해 하는 도구를 사용 하 여 온-프레미스 또는 레거시 데이터에서 통찰력을 잠금 해제 하려고 합니다. [Azure Machine Learning](/azure/machine-learning/) 는 ML 및 심층 학습 모델을 학습, 조정 및 배포 하기 위한 클라우드 기본 도구를 제공 합니다.  

그러나 일부 데이터는 클라우드로의 송신이 너무 크거나 규정 상의 이유로 클라우드로 전송할 수 없습니다. 이 패턴을 사용 하 여 데이터 과학자는 온-프레미스 데이터 및 계산을 사용 하 여 모델을 학습 하는 Azure Machine Learning 사용할 수 있습니다.

## <a name="solution"></a>해결 방법

Edge 패턴의 교육은 Azure Stack 허브에서 실행 되는 VM (가상 머신)을 사용 합니다. VM은 Azure ML에서 계산 대상으로 등록 되므로 온-프레미스로만 데이터에 액세스할 수 있습니다. 이 경우 데이터는 Azure Stack 허브의 blob 저장소에 저장 됩니다.

모델을 학습 한 후에는 Azure ML, 컨테이너 화 된에 등록 되 고 배포를 위해 Azure Container Registry에 추가 됩니다. 이 패턴 반복의 경우 공용 인터넷을 통해 Azure Stack 허브 교육 VM에 연결할 수 있어야 합니다.

[![가장자리 아키텍처에서 ML 모델 학습](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)

패턴의 작동 방식은 다음과 같습니다.

1. Azure Stack 허브 VM은 Azure ML을 사용 하 여 계산 대상으로 배포 및 등록 됩니다.
2. Azure ML에서 계산 대상으로 Azure Stack 허브 VM을 사용 하는 실험을 만듭니다.
3. 모델을 학습 한 후에는 등록 되 고 컨테이너 화 된 됩니다.
4. 이제 온-프레미스 또는 클라우드의 위치에 모델을 배포할 수 있습니다.

## <a name="components"></a>구성 요소

이 솔루션은 다음 구성 요소를 사용 합니다.

| 계층 | 구성 요소 | Description |
|----------|-----------|-------------|
| Azure | Azure Machine Learning | 오케스트레이션는 ML 모델의 학습을 합니다. [Azure Machine Learning](/azure/machine-learning/) |
| | Azure Container Registry | Azure ML은 모델을 컨테이너에 패키지 하 고 배포를 위해 [Azure Container Registry](/azure/container-registry/) 에 저장 합니다.|
| Azure Stack 허브 | App Service | [App Service를 사용 하 Azure Stack 허브](/azure-stack/operator/azure-stack-app-service-overview) 는에 지의 구성 요소에 대 한 기반을 제공 합니다. |
| | Compute | Docker를 사용 하 여 Ubuntu를 실행 하는 Azure Stack 허브 VM은 ML 모델을 학습 하는 데 사용 됩니다. |
| | Storage | 개인 데이터는 Azure Stack Hub blob 저장소에서 호스팅될 수 있습니다. |

## <a name="issues-and-considerations"></a>문제 및 고려 사항

이 솔루션을 구현 하는 방법을 결정할 때 다음 사항을 고려 하십시오.

### <a name="scalability"></a>확장성

이 솔루션을 확장할 수 있도록 하려면 Azure Stack 허브에서 적절 한 크기의 VM을 만들어 학습 해야 합니다.

### <a name="availability"></a>가용성

학습 스크립트와 Azure Stack 허브 VM이 학습에 사용 되는 온-프레미스 데이터에 액세스할 수 있는지 확인 합니다.

### <a name="manageability"></a>관리 효율

모델을 배포 하는 동안 혼동을 피하기 위해 모델 및 실험을 적절 하 게 등록 하 고, 버전을 지정 하 고, 태그를 지정

### <a name="security"></a>보안

이 패턴을 통해 Azure ML은 가능한 중요 한 데이터를 온-프레미스에 액세스할 수 있습니다. Azure Stack 허브 VM에 대 한 SSH에 사용 되는 계정에 강력한 암호가 있고 학습 스크립트가 데이터를 클라우드에 유지 하거나 업로드 하지 않도록 합니다.

## <a name="next-steps"></a>다음 단계

이 문서에서 소개 하는 항목에 대 한 자세한 내용은 다음을 참조 하세요.

- ML 및 관련 항목의 개요는 [Azure Machine Learning 설명서](/azure/machine-learning) 를 참조 하세요.
- 컨테이너 배포를 위한 이미지를 작성, 저장 및 관리 하는 방법을 알아보려면 [Azure Container Registry](/azure/container-registry/) 를 참조 하세요.
- 리소스 공급자 및 배포 방법에 대 한 자세한 내용은 [Azure Stack Hub의 App Service](/azure-stack/operator/azure-stack-app-service-overview) 를 참조 하세요.
- 모범 사례에 대해 자세히 알아보고 추가 질문에 대 한 답변을 얻으려면 [하이브리드 응용 프로그램 디자인 고려 사항](overview-app-design-considerations.md) 을 참조 하세요.
- 제품 및 솔루션의 전체 포트폴리오에 대해 자세히 알아보려면 [Azure Stack 제품군 및 솔루션](/azure-stack) 을 참조 하세요.

솔루션 예제를 테스트할 준비가 되 면 [edge 배포 가이드에서 ML 모델 학습](https://aka.ms/edgetrainingdeploy)을 계속 진행 합니다. 배포 가이드에서는 구성 요소를 배포 및 테스트 하는 방법에 대 한 단계별 지침을 제공 합니다.
