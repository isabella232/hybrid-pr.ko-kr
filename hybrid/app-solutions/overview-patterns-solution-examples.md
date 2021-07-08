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
# <a name="hybrid-solution-patterns-and-examples-for-azure-and-azure-stack"></a><span data-ttu-id="c608c-103">Azure 및 Azure Stack의 하이브리드 솔루션 패턴 및 예제</span><span class="sxs-lookup"><span data-stu-id="c608c-103">Hybrid solution patterns and examples for Azure and Azure Stack</span></span>

<span data-ttu-id="c608c-104">Microsoft는 Azure 및 Azure Stack 제품과 솔루션을 하나의 일관된 Azure 에코시스템으로 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="c608c-104">Microsoft provides Azure and Azure Stack products and solutions as one consistent Azure ecosystem.</span></span> <span data-ttu-id="c608c-105">Microsoft Azure Stack 제품군은 Azure의 확장입니다.</span><span class="sxs-lookup"><span data-stu-id="c608c-105">The Microsoft Azure Stack family is an extension of Azure.</span></span>

## <a name="the-hybrid-cloud-and-hybrid-apps"></a><span data-ttu-id="c608c-106">하이브리드 클라우드 및 하이브리드 앱</span><span class="sxs-lookup"><span data-stu-id="c608c-106">The hybrid cloud and hybrid apps</span></span>

<span data-ttu-id="c608c-107">Azure Stack은 *하이브리드 클라우드* 를 사용하도록 설정하여 온-프레미스 환경 및 에지에 클라우드 컴퓨팅의 민첩성을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="c608c-107">Azure Stack brings the agility of cloud computing to your on-premises environment and the edge by enabling a *hybrid cloud*.</span></span> <span data-ttu-id="c608c-108">Azure Stack Hub, Azure Stack HCI 및 Azure Stack Edge는 Azure를 클라우드에서 소버린 데이터 센터, 지점, 현장 및 그 이상으로 확장합니다.</span><span class="sxs-lookup"><span data-stu-id="c608c-108">Azure Stack Hub, Azure Stack HCI, and Azure Stack Edge extend Azure from the cloud into your sovereign datacenters, branch offices, field, and beyond.</span></span> <span data-ttu-id="c608c-109">이러한 다양한 기능 집합을 사용하여 다음을 수행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c608c-109">With this diverse set of capabilities, you can:</span></span>

- <span data-ttu-id="c608c-110">Azure와 온-프레미스 환경에서 일관되게 코드를 다시 사용하고 클라우드 네이티브 앱 실행</span><span class="sxs-lookup"><span data-stu-id="c608c-110">Reuse code and run cloud-native apps consistently across Azure and your on-premises environments.</span></span>
- <span data-ttu-id="c608c-111">Azure 서비스에 대한 선택적 연결로 기존의 가상화된 워크로드 실행</span><span class="sxs-lookup"><span data-stu-id="c608c-111">Run traditional virtualized workloads with optional connections to Azure services.</span></span>
- <span data-ttu-id="c608c-112">데이터를 클라우드로 전송하거나 소버린 데이터 센터에 보관하여 규정 준수 유지</span><span class="sxs-lookup"><span data-stu-id="c608c-112">Transfer data to the cloud, or keep it in your sovereign datacenter to maintain compliance.</span></span>
- <span data-ttu-id="c608c-113">하드웨어 가속 기계 학습, 컨테이너화된 또는 가상화된 워크로드 모두를 인텔리전트 에지에서 실행</span><span class="sxs-lookup"><span data-stu-id="c608c-113">Run hardware-accelerated machine-learning, containerized, or virtualized workloads, all at the intelligent edge.</span></span>

<span data-ttu-id="c608c-114">클라우드를 포괄하는 앱을 *하이브리드 앱* 이라고도 합니다.</span><span class="sxs-lookup"><span data-stu-id="c608c-114">Apps that span clouds are also referred to as *hybrid apps*.</span></span> <span data-ttu-id="c608c-115">Azure에서 하이브리드 클라우드 앱을 빌드하고, 위치에 관계없이 연결된 데이터 센터 또는 연결되지 않은 데이터 센터에 배포할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c608c-115">You can build hybrid cloud apps in Azure and deploy them to your connected or disconnected datacenter located anywhere.</span></span>

<span data-ttu-id="c608c-116">하이브리드 앱 시나리오는 개발에 사용할 수 있는 리소스에 따라 크게 달라집니다.</span><span class="sxs-lookup"><span data-stu-id="c608c-116">Hybrid app scenarios vary greatly with the resources that are available for development.</span></span> <span data-ttu-id="c608c-117">또한 지리, 보안, 인터넷 액세스 등과 같은 고려 사항도 적용됩니다.</span><span class="sxs-lookup"><span data-stu-id="c608c-117">They also span considerations such as geography, security, internet access, and others.</span></span> <span data-ttu-id="c608c-118">여기에 설명된 솔루션 패턴과 예제가 모든 요구 사항을 해결해 주는 것은 아니지만, 하이브리드 솔루션을 구현하는 동안 살펴보고 다시 사용할 수 있는 지침과 예제를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="c608c-118">Although the solution patterns and examples described here may not address all requirements, they provide guidelines and examples to explore and reuse while implementing hybrid solutions.</span></span>

## <a name="solution-patterns"></a><span data-ttu-id="c608c-119">솔루션 패턴</span><span class="sxs-lookup"><span data-stu-id="c608c-119">Solution patterns</span></span>

<span data-ttu-id="c608c-120">솔루션 패턴은 실제 고객 시나리오 및 환경에서 일반화된 반복 가능한 디자인 지침을 추려냅니다.</span><span class="sxs-lookup"><span data-stu-id="c608c-120">Solution patterns cull generalized repeatable design guidance, from real world customer scenarios and experiences.</span></span> <span data-ttu-id="c608c-121">패턴은 추상적이며 다양한 유형의 시나리오 또는 업종에 적용될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c608c-121">A pattern is abstract, allowing it to be applicable to different types of scenarios or vertical industries.</span></span> <span data-ttu-id="c608c-122">각 패턴은 컨텍스트와 문제를 문서화하고 솔루션 예제에 대한 개요를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="c608c-122">Each pattern documents the context and problem, and provides an overview of a solution example.</span></span> <span data-ttu-id="c608c-123">솔루션 예제는 패턴의 가능한 구현을 의미합니다.</span><span class="sxs-lookup"><span data-stu-id="c608c-123">The solution example is meant as a possible implementation of the pattern.</span></span>

<span data-ttu-id="c608c-124">두 가지 유형의 패턴 문서가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c608c-124">There are two types of pattern articles:</span></span>

- <span data-ttu-id="c608c-125">단일 패턴: 단일 범용 시나리오에 대한 디자인 지침을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="c608c-125">Single pattern: provides design guidance for a single general-purpose scenario.</span></span>
- <span data-ttu-id="c608c-126">다중 패턴: 여러 패턴의 애플리케이션을 사용하는 디자인 지침을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="c608c-126">Multi-pattern: provides design guidance where the application of multiple patterns is used.</span></span> <span data-ttu-id="c608c-127">이 패턴은 더 복잡한 시나리오 또는 업계별 문제를 해결하는 데 자주 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="c608c-127">This pattern is frequently required for solving more complex scenarios or industry-specific problems.</span></span>

## <a name="solution-deployment-guides"></a><span data-ttu-id="c608c-128">솔루션 배포 가이드</span><span class="sxs-lookup"><span data-stu-id="c608c-128">Solution deployment guides</span></span>

<span data-ttu-id="c608c-129">단계별 배포 가이드는 솔루션 예제를 배포하는 데 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="c608c-129">Step-by-step deployment guides assist in deploying a solution example.</span></span> <span data-ttu-id="c608c-130">이 가이드는 GitHub [솔루션 샘플 리포지토리](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)에 저장된 도우미 코드 샘플을 참조할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c608c-130">The guide may also refer to a companion code sample, stored in the GitHub [solutions sample repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>

## <a name="next-steps"></a><span data-ttu-id="c608c-131">다음 단계</span><span class="sxs-lookup"><span data-stu-id="c608c-131">Next steps</span></span>

- <span data-ttu-id="c608c-132">제품 및 솔루션의 전체 포트폴리오에 대해 자세히 알아보려면 [Azure Stack 제품군 및 솔루션](/azure-stack)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="c608c-132">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>
- <span data-ttu-id="c608c-133">각각에 대해 자세히 알아보려면 TOC의 "패턴" 및 "솔루션 배포 가이드" 섹션을 살펴보세요.</span><span class="sxs-lookup"><span data-stu-id="c608c-133">Explore the "Patterns" and the "Solution deployment guides" sections of the TOC to learn more about each.</span></span>
- <span data-ttu-id="c608c-134">하이브리드 앱 디자인, 배포 및 운영을 위한 소프트웨어 품질의 핵심 요소를 검토하려면 [하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="c608c-134">Read about [Hybrid app design considerations](overview-app-design-considerations.md) to review pillars of software quality for designing, deploying, and operating hybrid apps.</span></span>
- <span data-ttu-id="c608c-135">[Azure Stack에서 개발 환경을 설정](/azure-stack/user/azure-stack-dev-start)하고 Azure Stack에 [첫 번째 앱을 배포](/azure-stack/user/azure-stack-dev-start-deploy-app)합니다.</span><span class="sxs-lookup"><span data-stu-id="c608c-135">[Set up a development environment on Azure Stack](/azure-stack/user/azure-stack-dev-start) and [deploy your first app](/azure-stack/user/azure-stack-dev-start-deploy-app) on Azure Stack.</span></span>
