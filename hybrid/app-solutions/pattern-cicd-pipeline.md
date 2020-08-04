---
title: Azure Stack Hub의 DevOps 패턴
description: Azure 및 Azure Stack Hub의 배포 간에 일관성을 유지할 수 있도록 DevOps 패턴에 대해 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: e26056a9507a7467473b009725d4f210d9d59ec8
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477238"
---
# <a name="devops-pattern"></a><span data-ttu-id="8dd1e-103">DevOps 패턴</span><span class="sxs-lookup"><span data-stu-id="8dd1e-103">DevOps pattern</span></span>

<span data-ttu-id="8dd1e-104">단일 위치에서 코딩하고, 로컬 데이터 센터, 프라이빗 클라우드 또는 퍼블릭 클라우드에 있을 수 있는 개발, 테스트 및 프로덕션 환경에서 여러 대상에 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-104">Code from a single location and deploy to multiple targets in development, test, and production environments that may be in your local datacenter, private clouds, or the public cloud.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="8dd1e-105">컨텍스트 및 문제점</span><span class="sxs-lookup"><span data-stu-id="8dd1e-105">Context and problem</span></span>

<span data-ttu-id="8dd1e-106">애플리케이션 배포 연속성, 보안 및 안정성은 조직에 필수적이며 개발 팀에 중요합니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-106">Application deployment continuity, security, and reliability are essential to organizations and critical to development teams.</span></span>

<span data-ttu-id="8dd1e-107">앱은 일반적으로 각 대상 환경에서 실행하기 위해 코드를 리팩터링해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-107">Apps often require refactored code to run in each target environment.</span></span> <span data-ttu-id="8dd1e-108">즉, 앱이 완전히 이식할 가능성이 없습니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-108">This means that an app isn't completely portable.</span></span> <span data-ttu-id="8dd1e-109">각 환경을 통해 이동하므로 업데이트하고, 테스트하고, 유효성을 검사해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-109">It must be updated, tested, and validated as it moves through each environment.</span></span> <span data-ttu-id="8dd1e-110">예를 들어 개발 환경에서 작성된 코드는 테스트 환경에서 작동하고 최종적으로 프로덕션 환경에 있는 경우 다시 작성하도록 다시 작성해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-110">For example, code written in a development environment must then be rewritten to work in a test environment and rewritten when it finally lands in a production environment.</span></span> <span data-ttu-id="8dd1e-111">또한 이 코드는 구체적으로 호스트에 연결됩니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-111">Furthermore, this code is specifically tied to the host.</span></span> <span data-ttu-id="8dd1e-112">이렇게 하면 앱을 유지 관리하는 비용과 복잡성이 증가합니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-112">This increases the cost and complexity of maintaining your app.</span></span> <span data-ttu-id="8dd1e-113">각 앱 버전은 각 환경에 연결됩니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-113">Each version of the app is tied to each environment.</span></span> <span data-ttu-id="8dd1e-114">복잡성 및 중복은 보안과 코드 품질의 위험을 증가시킵니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-114">The increased complexity and duplication increase the risk of security and code quality.</span></span> <span data-ttu-id="8dd1e-115">또한 실패한 호스트 복원을 제거하거나 추가 호스트를 배포하여 수요를 늘릴 때 코드를 즉시 재배포할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-115">In addition, the code can't be readily redeployed when you remove restore failed hosts or deploy additional hosts to handle increases in demand.</span></span>

## <a name="solution"></a><span data-ttu-id="8dd1e-116">해결 방법</span><span class="sxs-lookup"><span data-stu-id="8dd1e-116">Solution</span></span>

<span data-ttu-id="8dd1e-117">DevOps 패턴을 사용하면 여러 클라우드에서 실행되는 앱을 빌드, 테스트 및 배포할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-117">The DevOps Pattern enables you to build, test, and deploy an app that runs on multiple clouds.</span></span> <span data-ttu-id="8dd1e-118">이 패턴은 연속 통합 및 지속적인 업데이트의 결합을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-118">This pattern unites the practice of continuous integration and continuous delivery.</span></span> <span data-ttu-id="8dd1e-119">연속 통합을 사용하면 팀 멤버가 버전 제어 변경 내용을 커밋할 때마다 코드가 빌드되고 테스트됩니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-119">With continuous integration, code is built and tested every time a team member commits a change to version control.</span></span> <span data-ttu-id="8dd1e-120">지속적인 업데이트는 빌드에서 프로덕션 환경으로 각 단계를 자동화합니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-120">Continuous delivery automates each step from a build to a production environment.</span></span> <span data-ttu-id="8dd1e-121">이러한 프로세스는 함께 다양한 환경에서 배포를 지원하는 릴리스 프로세스를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-121">Together, these processes create a release process that supports deployment across diverse environments.</span></span> <span data-ttu-id="8dd1e-122">이 패턴을 사용하여 코드를 초안한 다음, 동일한 코드를 온-프레미스 환경, 다른 프라이빗 클라우드 및 퍼블릭 클라우드에 배포할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-122">With this pattern, you can draft your code and then deploy the same code to a premise environment, different private clouds, and the public clouds.</span></span> <span data-ttu-id="8dd1e-123">환경의 차이에는 코드를 변경하는 대신 구성 파일을 변경해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-123">Differences in environment require a change to a configuration file rather than changes to the code.</span></span>

![DevOps 패턴](media/pattern-cicd-pipeline/hybrid-ci-cd.png)

<span data-ttu-id="8dd1e-125">온-프레미스, 프라이빗 클라우드 및 퍼블릭 클라우드 환경에서 일관된 개발 도구 세트를 사용하여 연속 통합 및 지속적인 업데이트를 구현할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-125">With a consistent set of development tools across on-premises, private cloud, and public cloud environments, you can implement a practice of continuous integration and continuous delivery.</span></span> <span data-ttu-id="8dd1e-126">DevOps 패턴을 사용하여 배포된 앱과 서비스는 교환할 수 있으며, 온-프레미스 및 퍼블릭 클라우드 기능과 기능을 활용하여 이러한 위치에서 실행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-126">Apps and services deployed using the DevOps Pattern are interchangeable and can run in any of these locations, taking advantage of on-premises and public cloud features and capabilities.</span></span>

<span data-ttu-id="8dd1e-127">DevOps 릴리스 파이프라인을 사용하면 다음과 같은 작업을 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-127">Using a DevOps release pipeline helps you:</span></span>

- <span data-ttu-id="8dd1e-128">단일 리포지토리에 대한 코드 커밋을 기반으로 새 빌드를 시작합니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-128">Initiate a new build based on code commits to a single repository.</span></span>
- <span data-ttu-id="8dd1e-129">사용자 승인 테스트를 위해 새로 빌드된 코드를 퍼블릭 클라우드에 자동으로 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-129">Automatically deploy your newly built code to the public cloud for user acceptance testing.</span></span>
- <span data-ttu-id="8dd1e-130">코드에서 테스트가 통과되면 프라이빗 클라우드에 자동으로 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-130">Automatically deploy to a private cloud once your code has passed testing.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="8dd1e-131">문제 및 고려 사항</span><span class="sxs-lookup"><span data-stu-id="8dd1e-131">Issues and considerations</span></span>

<span data-ttu-id="8dd1e-132">DevOps 패턴은 대상 환경에 관계 없이 배포 간에 일관성을 유지하기 위한 것입니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-132">The DevOps Pattern is intended to ensure consistency across deployments regardless of the target environment.</span></span> <span data-ttu-id="8dd1e-133">그러나 기능은 클라우드 및 온-프레미스 환경에 따라 다릅니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-133">However, capabilities vary across cloud and on-premises environments.</span></span> <span data-ttu-id="8dd1e-134">다음 사항을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-134">Consider the following points:</span></span>

- <span data-ttu-id="8dd1e-135">배포의 함수, 엔드포인트, 서비스 및 기타 리소스를 대상 배포 위치에서 사용할 수 있나요?</span><span class="sxs-lookup"><span data-stu-id="8dd1e-135">Are the functions, endpoints, services, and other resources in your deployment available in the target deployment locations?</span></span>
- <span data-ttu-id="8dd1e-136">구성 아티팩트가 여러 클라우드에서 액세스할 수 있는 위치에 저장되나요?</span><span class="sxs-lookup"><span data-stu-id="8dd1e-136">Are configuration artifacts stored in locations that are accessible across clouds?</span></span>
- <span data-ttu-id="8dd1e-137">배포 매개 변수는 모든 대상 환경에서 작동하나요?</span><span class="sxs-lookup"><span data-stu-id="8dd1e-137">Will deployment parameters work in all the target environments?</span></span>
- <span data-ttu-id="8dd1e-138">모든 대상 클라우드에서 리소스 특정 속성을 사용할 수 있나요?</span><span class="sxs-lookup"><span data-stu-id="8dd1e-138">Are resource-specific properties available in all target clouds?</span></span>

<span data-ttu-id="8dd1e-139">자세한 내용은 [클라우드 일관성을 위한 Azure Resource Manager 템플릿 개발](/azure/azure-resource-manager/templates-cloud-consistency)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-139">For more information, see [Develop Azure Resource Manager templates for cloud consistency](/azure/azure-resource-manager/templates-cloud-consistency).</span></span>

<span data-ttu-id="8dd1e-140">또한 이 패턴을 구현할 방법을 결정할 때 다음 사항을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-140">In addition, consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="8dd1e-141">확장성</span><span class="sxs-lookup"><span data-stu-id="8dd1e-141">Scalability</span></span>

<span data-ttu-id="8dd1e-142">배포 자동화 시스템은 DevOps 패턴의 핵심 제어 지점입니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-142">Deployment automation systems are the key control point in the DevOps Patterns.</span></span> <span data-ttu-id="8dd1e-143">구현은 다를 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-143">Implementations can vary.</span></span> <span data-ttu-id="8dd1e-144">올바른 서버 크기를 선택하는 것은 예상된 워크로드 크기에 따라 달라집니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-144">The selection of the correct server size depends on the size of the expected workload.</span></span> <span data-ttu-id="8dd1e-145">VM은 컨테이너보다 크기를 조정하는 데 더 많은 비용이 듭니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-145">VMs cost more to scale than containers.</span></span> <span data-ttu-id="8dd1e-146">그러나 크기 조정을 위해 컨테이너를 사용하려면 컨테이너와 함께 빌드 프로세스를 실행해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-146">To use containers for scaling, however, your build process must run with containers.</span></span>

### <a name="availability"></a><span data-ttu-id="8dd1e-147">가용성</span><span class="sxs-lookup"><span data-stu-id="8dd1e-147">Availability</span></span>

<span data-ttu-id="8dd1e-148">DevPattern의 컨텍스트에서 가용성이란 테스트 결과, 코드 종속성 또는 기타 아티팩트 등의 워크플로와 연결된 모든 상태 정보를 복원할 수 있다는 뜻입니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-148">Availability in the context of the DevPattern means being able to recover any state information associated with your workflow, such as test results, code dependencies, or other artifacts.</span></span> <span data-ttu-id="8dd1e-149">가용성 요구 사항을 평가하려면 두 가지 공통 메트릭을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-149">To assess your availability requirements, consider two common metrics:</span></span>

- <span data-ttu-id="8dd1e-150">RTO(복구 시간 목표)는 시스템 없이 얼마나 오래 갈 수 있는지 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-150">Recovery Time Objective (RTO) specifies how long you can go without a system.</span></span>

- <span data-ttu-id="8dd1e-151">RPO(복구 지점 목표)는 서비스 중단으로 인해 시스템에 영향을 줄 경우 잃을 수 있는 데이터 양을 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-151">Recovery Point Objective (RPO) indicates how much data you can afford to lose if a disruption in service affects the system.</span></span>

<span data-ttu-id="8dd1e-152">실제로 RTO와 RPO는 중복성과 백업을 의미합니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-152">In practice, RTO, and RPO imply redundancy and backup.</span></span> <span data-ttu-id="8dd1e-153">글로벌 Azure 클라우드에서 가용성은 Azure의 일부인 하드웨어 복구의 문제가 아니라 DevOps 시스템의 상태 유지를 보장하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-153">On the global Azure cloud, availability isn't a question of hardware recovery—that's part of Azure—but rather ensuring you maintain the state of your DevOps systems.</span></span> <span data-ttu-id="8dd1e-154">Azure Stack Hub에서 하드웨어 복구를 고려할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-154">On Azure Stack Hub, hardware recovery may be a consideration.</span></span>

<span data-ttu-id="8dd1e-155">배포 자동화에 사용되는 시스템을 디자인할 때 고려해야 할 또 다른 주요 사항은 액세스 제어 및 클라우드 환경에 서비스를 배포하는 데 필요한 권한을 적절하게 관리하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-155">Another major consideration when designing the system used for deployment automation is access control and the proper management of the rights needed to deploy services to cloud environments.</span></span> <span data-ttu-id="8dd1e-156">배포를 생성, 삭제 또는 수정하는 데 필요한 권한은 무엇인가요?</span><span class="sxs-lookup"><span data-stu-id="8dd1e-156">What rights are needed to create, delete, or modify deployments?</span></span> <span data-ttu-id="8dd1e-157">예를 들어 Azure에서 리소스 그룹을 만들고 리소스 그룹에 서비스를 배포하기 위해 일반적으로 하나의 권한 세트가 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-157">For example, one set of rights is typically required to create a resource group in Azure and another to deploy services in the resource group.</span></span>

### <a name="manageability"></a><span data-ttu-id="8dd1e-158">관리 효율</span><span class="sxs-lookup"><span data-stu-id="8dd1e-158">Manageability</span></span>

<span data-ttu-id="8dd1e-159">DevOps 패턴을 기반으로 하는 시스템의 디자인은 포트폴리오에서 각 서비스에 대한 자동화, 로깅 및 경고를 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-159">The design of any system based on the DevOps pattern must consider automation, logging, and alerting for each service across the portfolio.</span></span> <span data-ttu-id="8dd1e-160">공유 서비스, 애플리케이션 팀 또는 둘 다를 사용하고 보안 정책 및 관리도 추적합니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-160">Use shared services, an application team, or both, and track security policies and governance as well.</span></span>

<span data-ttu-id="8dd1e-161">프로덕션 환경 및 개발/테스트 환경을 Azure 또는 Azure Stack Hub의 별도 리소스 그룹에 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-161">Deploy production environments and development/test environments in separate resource groups on Azure or Azure Stack Hub.</span></span> <span data-ttu-id="8dd1e-162">그런 다음, 각 환경의 리소스를 모니터링하고 리소스 그룹별 청구 비용을 롤업할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-162">Then you can monitor each environment's resources and roll up billing costs by resource group.</span></span> <span data-ttu-id="8dd1e-163">리소스를 하나의 집합으로 삭제할 수도 있습니다. 이러한 기능은 테스트 배포에서 유용합니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-163">You can also delete resources as a set, which is useful for test deployments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="8dd1e-164">이 패턴을 사용해야 하는 경우</span><span class="sxs-lookup"><span data-stu-id="8dd1e-164">When to use this pattern</span></span>

<span data-ttu-id="8dd1e-165">다음 경우에 이 패턴을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-165">Use this pattern if:</span></span>

- <span data-ttu-id="8dd1e-166">개발자의 요구 사항을 충족하는 한 환경에서 코드를 개발하고, 새 코드를 개발하기 어려울 수 있는 솔루션 관련 환경에 배포할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-166">You can develop code in one environment that meets the needs of your developers, and deploy to an environment specific to your solution where it may be difficult to develop new code.</span></span>
- <span data-ttu-id="8dd1e-167">DevOps 패턴에서 연속 통합 및 지속적인 업데이트 프로세스를 수행할 수 있다면 개발자가 원하는 코드 및 도구를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-167">You can use the code and tools your developers would like, as long as they're able to follow the continuous integration and continuous delivery process in the DevOps Pattern.</span></span>

<span data-ttu-id="8dd1e-168">이 패턴이 권장되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-168">This pattern isn't recommended:</span></span>

- <span data-ttu-id="8dd1e-169">리소스, 구성, ID 및 보안 작업을 프로비저닝하는 인프라를 자동화할 수 없는 경우</span><span class="sxs-lookup"><span data-stu-id="8dd1e-169">If you can't automate infrastructure, provisioning resources, configuration, identity, and security tasks.</span></span>
- <span data-ttu-id="8dd1e-170">팀이 CI/CD(연속 통합/지속적인 개발) 접근 방식을 구현하는 하이브리드 클라우드 리소스에 액세스할 수 없는 경우</span><span class="sxs-lookup"><span data-stu-id="8dd1e-170">If teams don't have access to hybrid cloud resources to implement a Continuous Integration/Continuous Development (CI/CD) approach.</span></span>

## <a name="next-steps"></a><span data-ttu-id="8dd1e-171">다음 단계</span><span class="sxs-lookup"><span data-stu-id="8dd1e-171">Next steps</span></span>

<span data-ttu-id="8dd1e-172">이 문서에서 소개하는 항목에 대한 자세한 내용은 다음을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-172">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="8dd1e-173">Azure DevOps 및 Azure Repos와 Azure Pipelines를 비롯한 관련 도구에 대해 자세히 알아보려면 [Azure DevOps 설명서](/azure/devops)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-173">See the [Azure DevOps documentation](/azure/devops) to learn more about Azure DevOps and related tools, including Azure Repos, and Azure Pipelines.</span></span>
- <span data-ttu-id="8dd1e-174">제품 및 솔루션의 전체 포트폴리오에 대해 자세히 알아보려면 [Azure Stack 제품군 및 솔루션](/azure-stack)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-174">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="8dd1e-175">솔루션 예제를 테스트할 준비가 되면 [DevOps 하이브리드 CI/CD 솔루션 배포 가이드](https://aka.ms/hybriddevopsdeploy)를 계속 진행하세요.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-175">When you're ready to test the solution example, continue with the [DevOps hybrid CI/CD solution deployment guide](https://aka.ms/hybriddevopsdeploy).</span></span> <span data-ttu-id="8dd1e-176">배포 가이드에서는 해당 구성 요소를 배포 및 테스트하는 방법에 대한 단계별 지침을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-176">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="8dd1e-177">하이브리드 CI/CD(연속 통합/지속적인 업데이트) 파이프라인을 사용하여 Azure 및 Azure Stack Hub에 앱을 배포하는 방법을 알아봅니다.</span><span class="sxs-lookup"><span data-stu-id="8dd1e-177">You learn how to deploy an app to Azure and Azure Stack Hub using a hybrid continuous integration/continuous delivery (CI/CD) pipeline.</span></span>
