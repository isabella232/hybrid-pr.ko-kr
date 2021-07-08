---
title: Azure 및 Azure Stack Hub의 하이브리드 패턴 및 솔루션 예제
description: Azure 및 Azure Stack Hub에서 하이브리드 솔루션을 학습하고 빌드하기 위한 하이브리드 패턴 및 솔루션 예제를 소개합니다.
author: BryanLa
ms.topic: overview
ms.date: 05/24/2021
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 05/24/2021
ms.openlocfilehash: 9f3f13c23bec31c5132c7e90294356b9463fd72b
ms.sourcegitcommit: cf2c4033d1b169f5b63980ce1865281366905e2e
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/25/2021
ms.locfileid: "110343861"
---
# <a name="hybrid-solution-patterns-and-examples-for-azure-and-azure-stack"></a>Azure 및 Azure Stack의 하이브리드 솔루션 패턴 및 예제

Microsoft는 Azure 및 Azure Stack 제품과 솔루션을 하나의 일관된 Azure 에코시스템으로 제공합니다. Microsoft Azure Stack 제품군은 Azure의 확장입니다.

## <a name="the-hybrid-cloud-and-hybrid-apps"></a>하이브리드 클라우드 및 하이브리드 앱

Azure Stack은 *하이브리드 클라우드* 를 사용하도록 설정하여 온-프레미스 환경 및 에지에 클라우드 컴퓨팅의 민첩성을 제공합니다. Azure Stack Hub, Azure Stack HCI 및 Azure Stack Edge는 Azure를 클라우드에서 소버린 데이터 센터, 지점, 현장 및 그 이상으로 확장합니다. 이러한 다양한 기능 집합을 사용하여 다음을 수행할 수 있습니다.

- Azure와 온-프레미스 환경에서 일관되게 코드를 다시 사용하고 클라우드 네이티브 앱 실행
- Azure 서비스에 대한 선택적 연결로 기존의 가상화된 워크로드 실행
- 데이터를 클라우드로 전송하거나 소버린 데이터 센터에 보관하여 규정 준수 유지
- 하드웨어 가속 기계 학습, 컨테이너화된 또는 가상화된 워크로드 모두를 인텔리전트 에지에서 실행

클라우드를 포괄하는 앱을 *하이브리드 앱* 이라고도 합니다. Azure에서 하이브리드 클라우드 앱을 빌드하고, 위치에 관계없이 연결된 데이터 센터 또는 연결되지 않은 데이터 센터에 배포할 수 있습니다.

하이브리드 앱 시나리오는 개발에 사용할 수 있는 리소스에 따라 크게 달라집니다. 또한 지리, 보안, 인터넷 액세스 등과 같은 고려 사항도 적용됩니다. 여기에 설명된 솔루션 패턴과 예제가 모든 요구 사항을 해결해 주는 것은 아니지만, 하이브리드 솔루션을 구현하는 동안 살펴보고 다시 사용할 수 있는 지침과 예제를 제공합니다.

## <a name="solution-patterns"></a>솔루션 패턴

솔루션 패턴은 실제 고객 시나리오 및 환경에서 일반화된 반복 가능한 디자인 지침을 추려냅니다. 패턴은 추상적이며 다양한 유형의 시나리오 또는 업종에 적용될 수 있습니다. 각 패턴은 컨텍스트와 문제를 문서화하고 솔루션 예제에 대한 개요를 제공합니다. 솔루션 예제는 패턴의 가능한 구현을 의미합니다.

두 가지 유형의 패턴 문서가 있습니다.

- 단일 패턴: 단일 범용 시나리오에 대한 디자인 지침을 제공합니다.
- 다중 패턴: 여러 패턴의 애플리케이션을 사용하는 디자인 지침을 제공합니다. 이 패턴은 더 복잡한 시나리오 또는 업계별 문제를 해결하는 데 자주 필요합니다.

## <a name="solution-deployment-guides"></a>솔루션 배포 가이드

단계별 배포 가이드는 솔루션 예제를 배포하는 데 도움이 됩니다. 이 가이드는 GitHub [솔루션 샘플 리포지토리](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)에 저장된 도우미 코드 샘플을 참조할 수도 있습니다.

## <a name="next-steps"></a>다음 단계

- 제품 및 솔루션의 전체 포트폴리오에 대해 자세히 알아보려면 [Azure Stack 제품군 및 솔루션](/azure-stack)을 참조하세요.
- 각각에 대해 자세히 알아보려면 TOC의 "패턴" 및 "솔루션 배포 가이드" 섹션을 살펴보세요.
- 하이브리드 앱 디자인, 배포 및 운영을 위한 소프트웨어 품질의 핵심 요소를 검토하려면 [하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md)을 참조하세요.
- [Azure Stack에서 개발 환경을 설정](/azure-stack/user/azure-stack-dev-start)하고 Azure Stack에 [첫 번째 앱을 배포](/azure-stack/user/azure-stack-dev-start-deploy-app)합니다.
