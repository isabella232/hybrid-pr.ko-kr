---
title: Azure 및 Azure Stack Hub에 대 한 하이브리드 패턴 및 솔루션 예제
description: Azure 및 Azure Stack Hub에서 하이브리드 솔루션을 학습 하 고 빌드하기 위한 하이브리드 패턴 및 솔루션 예제에 대 한 개요입니다.
author: BryanLa
ms.topic: overview
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ab0eb885e7b0fefaca8991522712652f979d8712
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910917"
---
# <a name="hybrid-patterns-and-solution-examples-for-azure-and-azure-stack"></a><span data-ttu-id="b4e57-103">Azure 및 Azure Stack에 대 한 하이브리드 패턴 및 솔루션 예제</span><span class="sxs-lookup"><span data-stu-id="b4e57-103">Hybrid patterns and solution examples for Azure and Azure Stack</span></span>

<span data-ttu-id="b4e57-104">Microsoft는 Azure 및 Azure Stack 제품 및 솔루션을 일관 된 단일 Azure 에코 시스템으로 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="b4e57-104">Microsoft provides Azure and Azure Stack products and solutions as one consistent Azure ecosystem.</span></span> <span data-ttu-id="b4e57-105">Microsoft Azure Stack 제품군은 Azure의 확장입니다.</span><span class="sxs-lookup"><span data-stu-id="b4e57-105">The Microsoft Azure Stack family is an extension of Azure.</span></span>

## <a name="the-hybrid-cloud-and-hybrid-apps"></a><span data-ttu-id="b4e57-106">하이브리드 클라우드 및 하이브리드 앱</span><span class="sxs-lookup"><span data-stu-id="b4e57-106">The hybrid cloud and hybrid apps</span></span>

<span data-ttu-id="b4e57-107">Azure Stack은 *하이브리드 클라우드*를 사용 하도록 설정 하 여 온-프레미스 환경 및에 지에 대 한 클라우드 컴퓨팅 민첩성을 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="b4e57-107">Azure Stack brings the agility of cloud computing to your on-premises environment and the edge by enabling a *hybrid cloud*.</span></span> <span data-ttu-id="b4e57-108">Azure Stack Hub, Azure Stack HCI 및 Azure Stack Edge는 클라우드에서 Azure를 소 버린 데이터 센터, 지사, 현장 및 이상으로 확장 합니다.</span><span class="sxs-lookup"><span data-stu-id="b4e57-108">Azure Stack Hub, Azure Stack HCI, and Azure Stack Edge extend Azure from the cloud into your sovereign datacenters, branch offices, field, and beyond.</span></span> <span data-ttu-id="b4e57-109">다양 한 기능 집합을 사용 하 여 다음을 수행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b4e57-109">With this diverse set of capabilities, you can:</span></span>

- <span data-ttu-id="b4e57-110">Azure와 온-프레미스 환경에서 일관 되 게 코드를 다시 사용 하 고 클라우드 네이티브 앱을 실행 하세요.</span><span class="sxs-lookup"><span data-stu-id="b4e57-110">Reuse code and run cloud-native apps consistently across Azure and your on-premises environments.</span></span>
- <span data-ttu-id="b4e57-111">Azure 서비스에 대 한 선택적 연결로 기존의 가상화 된 워크 로드를 실행 합니다.</span><span class="sxs-lookup"><span data-stu-id="b4e57-111">Run traditional virtualized workloads with optional connections to Azure services.</span></span>
- <span data-ttu-id="b4e57-112">데이터를 클라우드로 전송 하거나 규정 준수를 유지 하기 위해 소 버린 데이터 센터에 보관 합니다.</span><span class="sxs-lookup"><span data-stu-id="b4e57-112">Transfer data to the cloud, or keep it in your sovereign datacenter to maintain compliance.</span></span>
- <span data-ttu-id="b4e57-113">지능형에 지에서 하드웨어 가속 기계 학습, 컨테이너 화 된 또는 가상화 된 워크 로드를 실행 합니다.</span><span class="sxs-lookup"><span data-stu-id="b4e57-113">Run hardware-accelerated machine-learning, containerized, or virtualized workloads, all at the intelligent edge.</span></span>

<span data-ttu-id="b4e57-114">클라우드를 확장 하는 앱을 *하이브리드 앱*이 라고도 합니다.</span><span class="sxs-lookup"><span data-stu-id="b4e57-114">Apps that span clouds are also referred to as *hybrid apps*.</span></span> <span data-ttu-id="b4e57-115">Azure에서 하이브리드 클라우드 앱을 빌드하고 어디에 있든 연결 된 데이터 센터 또는 연결 되지 않은 데이터 센터에 배포할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b4e57-115">You can build hybrid cloud apps in Azure and deploy them to your connected or disconnected datacenter located anywhere.</span></span>

<span data-ttu-id="b4e57-116">하이브리드 앱 시나리오는 개발에 사용할 수 있는 리소스에 따라 크게 달라 집니다.</span><span class="sxs-lookup"><span data-stu-id="b4e57-116">Hybrid app scenarios vary greatly with the resources that are available for development.</span></span> <span data-ttu-id="b4e57-117">또한 지리, 보안, 인터넷 액세스 등과 같은 고려 사항을 포괄 합니다.</span><span class="sxs-lookup"><span data-stu-id="b4e57-117">They also span considerations such as geography, security, internet access, and others.</span></span> <span data-ttu-id="b4e57-118">여기에 설명 된 패턴과 솔루션은 모든 요구 사항을 해결할 수는 없지만 하이브리드 솔루션을 구현 하는 동안 탐색 하 고 다시 사용 하기 위한 지침과 예제를 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="b4e57-118">Although the patterns and solutions described here may not address all requirements, they provide guidelines and examples to explore and reuse while implementing hybrid solutions.</span></span>

## <a name="design-patterns"></a><span data-ttu-id="b4e57-119">디자인 패턴</span><span class="sxs-lookup"><span data-stu-id="b4e57-119">Design patterns</span></span>

<span data-ttu-id="b4e57-120">실제 고객 시나리오 및 환경에서 일반화 된 반복 가능한 디자인 지침을 추려내 디자인 패턴입니다.</span><span class="sxs-lookup"><span data-stu-id="b4e57-120">Design patterns cull generalized repeatable design guidance, from real world customer scenarios and experiences.</span></span> <span data-ttu-id="b4e57-121">패턴은 추상적 이며 다양 한 유형의 시나리오 또는 수직선에 적용 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b4e57-121">A pattern is abstract, allowing it to be applicable to different types of scenarios or vertical industries.</span></span> <span data-ttu-id="b4e57-122">각 패턴은 컨텍스트와 문제를 문서화 하 고 솔루션 예제에 대 한 개요를 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="b4e57-122">Each pattern documents the context and problem, and provides an overview of a solution example.</span></span> <span data-ttu-id="b4e57-123">솔루션 예제는 패턴의 가능한 구현입니다.</span><span class="sxs-lookup"><span data-stu-id="b4e57-123">The solution example is meant as a possible implementation of the pattern.</span></span>

<span data-ttu-id="b4e57-124">패턴 문서에는 다음과 같은 두 가지 유형이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b4e57-124">There are two types of pattern articles:</span></span>

- <span data-ttu-id="b4e57-125">단일 패턴: 단일 범용 시나리오에 대 한 디자인 지침을 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="b4e57-125">Single pattern: provides design guidance for a single general-purpose scenario.</span></span>
- <span data-ttu-id="b4e57-126">다중 패턴: 여러 패턴의 응용 프로그램을 사용 하는 디자인 지침을 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="b4e57-126">Multi-pattern: provides design guidance where the application of multiple patterns is used.</span></span> <span data-ttu-id="b4e57-127">이 패턴은 더 복잡 한 시나리오 또는 업계 특정 문제를 해결 하는 데 자주 필요 합니다.</span><span class="sxs-lookup"><span data-stu-id="b4e57-127">This pattern is frequently required for solving more complex scenarios or industry-specific problems.</span></span>

## <a name="solution-deployment-guides"></a><span data-ttu-id="b4e57-128">솔루션 배포 가이드</span><span class="sxs-lookup"><span data-stu-id="b4e57-128">Solution deployment guides</span></span>

<span data-ttu-id="b4e57-129">단계별 배포 가이드는 솔루션 예제를 배포 하는 데 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="b4e57-129">Step-by-step deployment guides assist in deploying a solution example.</span></span> <span data-ttu-id="b4e57-130">이 가이드는 GitHub [솔루션 샘플](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)리포지토리에 저장 된 도우미 코드 샘플을 참조할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b4e57-130">The guide may also refer to a companion code sample, stored in the GitHub [solutions sample repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>

## <a name="next-steps"></a><span data-ttu-id="b4e57-131">다음 단계</span><span class="sxs-lookup"><span data-stu-id="b4e57-131">Next steps</span></span>

- <span data-ttu-id="b4e57-132">제품 및 솔루션의 전체 포트폴리오에 대해 자세히 알아보려면 [Azure Stack 제품군 및 솔루션](/azure-stack) 을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="b4e57-132">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>
- <span data-ttu-id="b4e57-133">각각에 대해 자세히 알아보려면 TOC의 "패턴" 및 "솔루션 배포 가이드" 섹션을 살펴보세요.</span><span class="sxs-lookup"><span data-stu-id="b4e57-133">Explore the "Patterns" and the "Solution deployment guides" sections of the TOC to learn more about each.</span></span>
- <span data-ttu-id="b4e57-134">하이브리드 앱 디자인, 배포 및 운영을 위한 핵심 요소 소프트웨어 품질을 검토 하려면 [하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="b4e57-134">Read about [Hybrid app design considerations](overview-app-design-considerations.md) to review pillars of software quality for designing, deploying, and operating hybrid apps.</span></span>
- <span data-ttu-id="b4e57-135">[Azure Stack에서 개발 환경을 설정](/azure-stack/user/azure-stack-dev-start.md) 하 고 Azure Stack에 [첫 번째 앱을 배포](/azure-stack/user/azure-stack-dev-start-deploy-app.md) 합니다.</span><span class="sxs-lookup"><span data-stu-id="b4e57-135">[Set up a development environment on Azure Stack](/azure-stack/user/azure-stack-dev-start.md) and [deploy your first app](/azure-stack/user/azure-stack-dev-start-deploy-app.md) on Azure Stack.</span></span>
