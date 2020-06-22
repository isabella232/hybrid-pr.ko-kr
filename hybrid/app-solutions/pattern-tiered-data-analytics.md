---
title: Azure 및 Azure Stack Hub를 사용 하 여 분석 패턴에 대 한 계층화 된 데이터
description: Azure 및 Azure Stack Hub를 사용 하 여 하이브리드 클라우드를 통해 계층화 된 데이터 솔루션을 구현 하는 방법을 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: b671fa9f47fa51ab6e40633c04964957d613fec2
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911246"
---
# <a name="tiered-data-for-analytics-pattern"></a>분석 패턴에 대 한 계층화 된 데이터

이 패턴은 Azure Stack 허브와 Azure를 사용 하 여 여러 온-프레미스 및 클라우드 위치에서 데이터를 준비, 분석, 처리, 정리 및 저장 하는 방법을 보여 줍니다.

## <a name="context-and-problem"></a>컨텍스트 및 문제점

최신 기술 환경에서 엔터프라이즈 조직이 직면 하는 문제 중 하나는 데이터 저장, 처리 및 분석에 중요 한 문제입니다. 고려해 야 할 사항은 다음과 같습니다.

- 데이터 콘텐츠
- 위치
- 보안 및 개인 정보 요구 사항
- 액세스 권한
- 유지 관리
- 저장소 웨어하우징

Azure는 Azure Stack Hub와 함께 데이터 문제를 해결 하 고 저렴 한 솔루션을 제공 합니다. 이 솔루션은 분산 제조 또는 물류 회사를 통해 가장 잘 표현 됩니다.

이 솔루션은 다음 시나리오를 기반으로 합니다.

- 대규모 다중 분기 제조 조직.
- 글로벌 원격 위치와 중앙 본사 간의 빠르고 안전한 데이터 저장, 처리 및 배포가 필요 합니다.
- 직원 및 기계 활동, 시설 정보 및 비즈니스 보고 데이터를 안전 하 게 유지 해야 합니다. 데이터를 적절 하 게 배포 하 고 지역 규정 준수 정책 및 산업 규정을 충족 해야 합니다.

## <a name="solution"></a>해결 방법

온-프레미스 및 공용 클라우드 환경을 모두 사용 하는 것은 다중 시설 기업의 요구를 충족 합니다. Azure Stack 허브는 로컬 및 원격 데이터를 수집, 처리, 저장 및 배포 하기 위한 빠르고 안전 하 고 유연한 솔루션을 제공 합니다. 이 패턴은 위치와 사용자 간에 보안, 기밀성, 회사 정책 및 규정 요구 사항이 다를 수 있는 경우에 특히 유용 합니다.

![분석 솔루션 아키텍처에 대 한 계층화 된 데이터 패턴](media/pattern-tiered-data-analytics/solution-architecture.png)

## <a name="components"></a>구성 요소

이 패턴은 다음 구성 요소를 사용 합니다.

| 계층 | 구성 요소 | Description |
|----------|-----------|-------------|
| Azure | Storage | [Azure Storage](/azure/storage/) 계정은 sterile 데이터 소비 끝점을 제공 합니다. Azure Storage는 최신 데이터 스토리지 시나리오를 위한 Microsoft의 클라우드 스토리지 솔루션입니다. Azure Storage은 데이터 개체에 대 한 대규모 확장 가능한 개체 저장소와 클라우드의 파일 시스템 서비스를 제공 합니다. 또한 신뢰할 수 있는 메시징 및 NoSQL 저장소에 대 한 메시징 저장소를 제공 합니다. |
| Azure Stack 허브 | Storage | [Azure Stack 허브 저장소](/azure-stack/user/azure-stack-storage-overview) 계정은 여러 서비스에 사용 됩니다.<br><br>- 원시 데이터 저장소에 대 한 **Blob 저장소** 입니다. Blob storage는 문서, 미디어 파일 또는 앱 설치 관리자와 같은 모든 유형의 텍스트 또는 이진 데이터를 보유할 수 있습니다. 모든 blob은 컨테이너 아래에 구성 됩니다. 컨테이너는 개체 그룹에 보안 정책을 할당 하는 유용한 방법을 제공 합니다. 저장소 계정에는 원하는 수의 컨테이너가 포함 될 수 있으며, 컨테이너에는 저장소 계정의 최대 500의 용량 제한까지 임의의 blob이 포함 될 수 있습니다.<br>- 데이터 보관을 위한 **Blob 저장소** 입니다. 쿨 데이터 보관을 위해 저렴 한 저장소에 대 한 이점이 있습니다. 쿨 데이터의 예로는 백업, 미디어 콘텐츠, 과학 데이터, 규정 준수 및 보관 데이터 등이 있습니다. 일반적으로 자주 액세스 되는 데이터는 쿨 저장소로 간주 됩니다. 액세스 빈도 및 보존 기간과 같은 특성에 따라 데이터를 계층화 합니다. 고객 데이터는 자주 액세스 되지 않지만 핫 데이터에 대 한 비슷한 대기 시간 및 성능이 필요 합니다.<br>- 처리 된 데이터 저장소에 대 한 **큐 저장소** 입니다. Queue storage는 응용 프로그램 구성 요소 간에 클라우드 메시징을 제공 합니다. 규모에 맞게 앱을 디자인할 때 앱 구성 요소는 독립적으로 확장할 수 있도록 분리 되는 경우가 많습니다. Queue storage는 클라우드, 데스크톱, 온-프레미스 서버 또는 모바일 장치에서 실행 되는 응용 프로그램 구성 요소 간의 통신을 위한 비동기 메시징을 제공 합니다. Queue Storage는 또한 비동기 작업 관리와 프로세스 워크플로 작성을 지원합니다. |
| | Azure 기능 | [Azure Functions](/azure/azure-functions/) 서비스는 Azure Stack 허브 리소스 공급자 [에서 Azure App Service](/azure-stack/operator/azure-stack-app-service-overview) 제공 합니다. Azure Functions를 사용 하면 다양 한 이벤트에 대 한 응답으로 간단한 서버를 사용 하지 않는 환경에서 코드를 실행할 수 있습니다. 사용자가 선택한 프로그래밍 언어를 사용 하 여 VM을 만들거나 웹 앱을 게시 하지 않고도 수요를 충족 하는 Azure Functions 확장할 수 있습니다. 함수는 다음에 대 한 솔루션에서 사용 됩니다.<br><br>- **데이터 흡입구**<br>- **데이터 sterilization.** 수동으로 트리거된 함수는 예약 된 데이터 처리, 정리 및 보관을 수행할 수 있습니다. 예를 들어 야간 고객 목록 삭제 및 월간 보고서 처리를 포함할 수 있습니다.|

## <a name="issues-and-considerations"></a>문제 및 고려 사항

이 솔루션을 구현 하는 방법을 결정할 때 다음 사항을 고려 하십시오.

### <a name="scalability"></a>확장성

Azure Functions 및 저장소 솔루션은 데이터 볼륨 및 처리 요구에 맞게 확장 됩니다. Azure 확장성 정보 및 대상에 대해서는 [Azure Storage 확장성 설명서](/azure/storage/common/storage-scalability-targets)를 참조 하세요.

### <a name="availability"></a>가용성

저장소는이 패턴에 대 한 주요 가용성 고려 사항입니다. 대량 데이터 볼륨 처리 및 배포에는 빠른 링크를 통한 연결이 필요 합니다.

### <a name="manageability"></a>관리 효율

이 솔루션의 관리 용이성은 원본 제어를 사용 하 고 참여 하는 제작 도구에 따라 달라 집니다.

## <a name="next-steps"></a>다음 단계

이 문서에서 소개 하는 항목에 대 한 자세한 내용은 다음을 참조 하세요.

- [Azure Storage](/azure/storage/) 및 [Azure Functions](/azure/azure-functions/) 설명서를 참조 하세요. 이 패턴은 Azure 및 Azure Stack 허브 모두에서 Azure Storage 계정과 Azure Functions을 많이 사용 합니다.
- 모범 사례에 대 한 자세한 내용을 알아보고 추가 질문에 대 한 답변을 얻으려면 [하이브리드 응용 프로그램 디자인 고려 사항](overview-app-design-considerations.md) 을 참조 하세요.
- 제품 및 솔루션의 전체 포트폴리오에 대해 자세히 알아보려면 [Azure Stack 제품군 및 솔루션](/azure-stack) 을 참조 하세요.

솔루션 예제를 테스트할 준비가 되 면 [분석 솔루션에 대 한 계층화 된 데이터 배포 가이드](https://aka.ms/tiereddatadeploy)를 계속 진행 합니다. 배포 가이드에서는 구성 요소를 배포 및 테스트 하는 방법에 대 한 단계별 지침을 제공 합니다.
