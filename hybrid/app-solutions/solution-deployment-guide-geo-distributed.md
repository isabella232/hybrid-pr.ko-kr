---
title: Azure 및 Azure Stack Hub를 사용 하 여 지리적으로 분산 된 앱으로 트래픽 직접 전송
description: Azure 및 Azure Stack Hub를 사용 하 여 지리적으로 분산 된 앱 솔루션으로 트래픽을 특정 끝점으로 전달 하는 방법을 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 8f2b7e48a62896acfce7293dcd4f18d5a43add01
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911323"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a><span data-ttu-id="ce9f3-103">Azure 및 Azure Stack Hub를 사용 하 여 지리적으로 분산 된 앱으로 트래픽 직접 전송</span><span class="sxs-lookup"><span data-stu-id="ce9f3-103">Direct traffic with a geo-distributed app using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="ce9f3-104">지리적으로 분산 된 앱 패턴을 사용 하 여 다양 한 메트릭에 따라 특정 끝점으로 트래픽을 전송 하는 방법을 알아봅니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-104">Learn how to direct traffic to specific endpoints based on various metrics using the geo-distributed apps pattern.</span></span> <span data-ttu-id="ce9f3-105">지리적 기반 라우팅 및 끝점 구성을 사용 하 여 Traffic Manager 프로필을 만들면 정보가 지역 요구 사항, 회사 및 국제 규정 및 데이터 요구 사항에 따라 끝점으로 라우팅됩니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-105">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>

<span data-ttu-id="ce9f3-106">이 솔루션에서는 다음을 수행 하는 샘플 환경을 빌드합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="ce9f3-107">지리적으로 분산 된 앱을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-107">Create a geo-distributed app.</span></span>
> - <span data-ttu-id="ce9f3-108">Traffic Manager를 사용 하 여 앱을 대상으로 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-108">Use Traffic Manager to target your app.</span></span>

## <a name="use-the-geo-distributed-apps-pattern"></a><span data-ttu-id="ce9f3-109">지리적으로 분산 된 앱 패턴 사용</span><span class="sxs-lookup"><span data-stu-id="ce9f3-109">Use the geo-distributed apps pattern</span></span>

<span data-ttu-id="ce9f3-110">지리적으로 분산 된 패턴을 사용 하 여 앱은 지역에 걸쳐 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-110">With the geo-distributed pattern, your app spans regions.</span></span> <span data-ttu-id="ce9f3-111">공용 클라우드를 기본적으로 사용할 수 있지만 일부 사용자가 해당 데이터를 해당 지역에 유지 해야 할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-111">You can default to the public cloud, but some of your users may require that their data remain in their region.</span></span> <span data-ttu-id="ce9f3-112">사용자를 요구 사항에 따라 가장 적합 한 클라우드로 안내할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-112">You can direct users to the most suitable cloud based on their requirements.</span></span>

### <a name="issues-and-considerations"></a><span data-ttu-id="ce9f3-113">문제 및 고려 사항</span><span class="sxs-lookup"><span data-stu-id="ce9f3-113">Issues and considerations</span></span>

#### <a name="scalability-considerations"></a><span data-ttu-id="ce9f3-114">확장성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="ce9f3-114">Scalability considerations</span></span>

<span data-ttu-id="ce9f3-115">이 문서를 사용 하 여 빌드할 솔루션은 확장성을 고려 하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-115">The solution you'll build with this article isn't to accommodate scalability.</span></span> <span data-ttu-id="ce9f3-116">그러나 다른 Azure 및 온-프레미스 솔루션과 함께 사용 하는 경우 확장성 요구 사항을 수용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-116">However, if used in combination with other Azure and on-premises solutions, you can accommodate scalability requirements.</span></span> <span data-ttu-id="ce9f3-117">Traffic manager를 통해 자동 크기 조정을 사용 하는 하이브리드 솔루션을 만드는 방법에 대 한 자세한 내용은 [Azure를 사용 하 여 클라우드 간 확장 솔루션 만들기](solution-deployment-guide-cross-cloud-scaling.md)를 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-117">For information on creating a hybrid solution with autoscaling via traffic manager, see [Create cross-cloud scaling solutions with Azure](solution-deployment-guide-cross-cloud-scaling.md).</span></span>

#### <a name="availability-considerations"></a><span data-ttu-id="ce9f3-118">가용성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="ce9f3-118">Availability considerations</span></span>

<span data-ttu-id="ce9f3-119">확장성을 고려 하는 경우에도이 솔루션은 가용성을 직접 해결 하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-119">As is the case with scalability considerations, this solution doesn't directly address availability.</span></span> <span data-ttu-id="ce9f3-120">그러나이 솔루션 내에서 Azure 및 온-프레미스 솔루션을 구현 하 여 관련 된 모든 구성 요소에 대 한 고가용성을 보장할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-120">However, Azure and on-premises solutions can be implemented within this solution to ensure high availability for all components involved.</span></span>

### <a name="when-to-use-this-pattern"></a><span data-ttu-id="ce9f3-121">이 패턴을 사용해야 하는 경우</span><span class="sxs-lookup"><span data-stu-id="ce9f3-121">When to use this pattern</span></span>

- <span data-ttu-id="ce9f3-122">조직에 사용자 지정 지역 보안 및 배포 정책을 요구 하는 국제 분기가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-122">Your organization has international branches requiring custom regional security and distribution policies.</span></span>

- <span data-ttu-id="ce9f3-123">각 조직 사무소는 직원, 비즈니스 및 시설 데이터를 가져오며,이를 통해 지역 규정 및 표준 시간대 별로 보고 작업을 수행 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-123">Each of your organization's offices pulls employee, business, and facility data, which requires reporting activity per local regulations and time zones.</span></span>

- <span data-ttu-id="ce9f3-124">단일 지역 내에서 여러 앱 배포를 사용 하 여 응용 프로그램을 수평으로 확장 하 고 많은 지역에 걸쳐 응용 프로그램을 수평으로 확장 하 여 매우 많은 요구 사항을 충족할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-124">High-scale requirements are met by horizontally scaling out apps with multiple app deployments within a single region and across regions to handle extreme load requirements.</span></span>

### <a name="planning-the-topology"></a><span data-ttu-id="ce9f3-125">토폴로지 계획</span><span class="sxs-lookup"><span data-stu-id="ce9f3-125">Planning the topology</span></span>

<span data-ttu-id="ce9f3-126">분산 된 앱 공간을 빌드하기 전에 다음 사항을 파악 하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-126">Before building out a distributed app footprint, it helps to know the following things:</span></span>

- <span data-ttu-id="ce9f3-127">**앱에 대 한 사용자 지정 도메인:** 고객이 앱에 액세스 하는 데 사용할 사용자 지정 도메인 이름은 무엇 인가요?</span><span class="sxs-lookup"><span data-stu-id="ce9f3-127">**Custom domain for the app:** What's the custom domain name that customers will use to access the app?</span></span> <span data-ttu-id="ce9f3-128">샘플 앱의 경우 사용자 지정 도메인 이름은 *www \. scalableasedemo.com입니다.*</span><span class="sxs-lookup"><span data-stu-id="ce9f3-128">For the sample app, the custom domain name is *www\.scalableasedemo.com.*</span></span>

- <span data-ttu-id="ce9f3-129">**도메인 Traffic Manager:** [Azure Traffic Manager 프로필](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-manage-profiles)을 만들 때 도메인 이름이 선택 됩니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-129">**Traffic Manager domain:** A domain name is chosen when creating an [Azure Traffic Manager profile](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-manage-profiles).</span></span> <span data-ttu-id="ce9f3-130">이 이름은 Traffic Manager로 관리 되는 도메인 항목을 등록 하기 위해 *trafficmanager.net* 접미사와 결합 됩니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-130">This name is combined with the *trafficmanager.net* suffix to register a domain entry that's managed by Traffic Manager.</span></span> <span data-ttu-id="ce9f3-131">샘플 앱의 경우 선택한 이름은 *scalable-ase-demo*입니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-131">For the sample app, the name chosen is *scalable-ase-demo*.</span></span> <span data-ttu-id="ce9f3-132">따라서 Traffic Manager에서 관리 하는 전체 도메인 이름은 *scalable-ase-demo.trafficmanager.net*입니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-132">As a result, the full domain name that's managed by Traffic Manager is *scalable-ase-demo.trafficmanager.net*.</span></span>

- <span data-ttu-id="ce9f3-133">**앱 사용 공간의 크기를 조정 하는 전략:** 앱 공간을 단일 지역 또는 여러 지역에 있는 여러 App Service 환경에 배포할 것인지 아니면 두 방법의 조합으로 배포할 것인지 결정 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-133">**Strategy for scaling the app footprint:** Decide whether the app footprint will be distributed across multiple App Service environments in a single region, multiple regions, or a mix of both approaches.</span></span> <span data-ttu-id="ce9f3-134">결정은 고객 트래픽이 발생 하는 위치와 앱이 지 원하는 백 엔드 인프라의 나머지 부분이 얼마나 잘 확장 될 수 있는지에 따라 결정 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-134">The decision should be based on expectations of where customer traffic will originate and how well the rest of an app's supporting back-end infrastructure can scale.</span></span> <span data-ttu-id="ce9f3-135">예를 들어 100% 상태 비저장 앱을 사용 하는 경우 Azure 지역 마다 여러 App Service 환경의 조합을 사용 하 여 앱을 대규모 확장할 수 있으며 여러 Azure 지역에 배포 된 App Service 환경에 곱합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-135">For example, with a 100% stateless app, an app can be massively scaled using a combination of multiple App Service environments per Azure region, multiplied by App Service environments deployed across multiple Azure regions.</span></span> <span data-ttu-id="ce9f3-136">Azure에서 15 개 이상의 글로벌 Azure 지역을 선택할 수 있으므로, 고객은 세계 전체의 하이퍼 규모 앱 사용 공간을 실제로 구축할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-136">With 15+ global Azure regions available to choose from, customers can truly build a world-wide hyper-scale app footprint.</span></span> <span data-ttu-id="ce9f3-137">여기에 사용 된 샘플 앱의 경우 단일 Azure 지역 (미국 중부)에서 세 개의 App Service 환경을 만들었습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-137">For the sample app used here, three App Service environments were created in a single Azure region (South Central US).</span></span>

- <span data-ttu-id="ce9f3-138">**App Service 환경의 명명 규칙:** 각 App Service 환경에는 고유한 이름이 필요 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-138">**Naming convention for the App Service environments:** Each App Service environment requires a unique name.</span></span> <span data-ttu-id="ce9f3-139">하나 또는 두 개의 App Service 환경 이외에 각 App Service 환경을 식별 하는 데 도움이 되는 명명 규칙을 포함 하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-139">Beyond one or two App Service environments, it's helpful to have a naming convention to help identify each App Service environment.</span></span> <span data-ttu-id="ce9f3-140">여기에 사용 된 샘플 앱의 경우 간단한 명명 규칙을 사용 했습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-140">For the sample app used here, a simple naming convention was used.</span></span> <span data-ttu-id="ce9f3-141">세 가지 App Service 환경의 이름은 *fe1ase*, *fe2ase*및 *fe3ase*입니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-141">The names of the three App Service environments are *fe1ase*, *fe2ase*, and *fe3ase*.</span></span>

- <span data-ttu-id="ce9f3-142">**앱에 대 한 명명 규칙:** 앱의 여러 인스턴스가 배포 되므로 배포 된 앱의 각 인스턴스에 대해 이름이 필요 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-142">**Naming convention for the apps:** Since multiple instances of the app will be deployed, a name is needed for each instance of the deployed app.</span></span> <span data-ttu-id="ce9f3-143">Power Apps에 대 한 App Service Environment를 사용 하면 동일한 앱 이름을 여러 환경에서 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-143">With App Service Environment for Power Apps, the same app name can be used across multiple environments.</span></span> <span data-ttu-id="ce9f3-144">각 App Service 환경에 고유한 도메인 접미사가 있으므로 개발자는 각 환경에서 정확히 동일한 앱 이름을 다시 사용 하도록 선택할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-144">Since each App Service environment has a unique domain suffix, developers can choose to reuse the exact same app name in each environment.</span></span> <span data-ttu-id="ce9f3-145">예를 들어 개발자에 게 이름이 *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net*등 인 앱이 있을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-145">For example, a developer could have apps named as follows: *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net*, and so on.</span></span> <span data-ttu-id="ce9f3-146">여기에서 사용 되는 앱의 경우 각 앱 인스턴스에 고유한 이름이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-146">For the app used here, each app instance has a unique name.</span></span> <span data-ttu-id="ce9f3-147">앱 인스턴스에 사용되는 이름은 *webfrontend1*, *webfrontend2* 및 *webfrontend3*입니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-147">The app instance names used are *webfrontend1*, *webfrontend2*, and *webfrontend3*.</span></span>

> [!Tip]  
> <span data-ttu-id="ce9f3-148">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="ce9f3-148">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="ce9f3-149">Microsoft Azure Stack 허브는 Azure의 확장입니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-149">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="ce9f3-150">Azure Stack 허브는 온-프레미스 환경에 클라우드 컴퓨팅의 민첩성과 혁신을 제공 하 여 어디서 나 하이브리드 앱을 빌드하고 배포할 수 있는 유일한 하이브리드 클라우드를 사용 하도록 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-150">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="ce9f3-151">[하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 문서에서는 하이브리드 앱을 디자인, 배포 및 운영 하기 위한 소프트웨어 품질 (배치, 확장성, 가용성, 복원 력, 관리 효율성 및 보안)의 핵심 요소을 검토 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-151">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="ce9f3-152">디자인 고려 사항은 하이브리드 앱 디자인을 최적화 하 고 프로덕션 환경에서 문제를 최소화 하는 데 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-152">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="part-1-create-a-geo-distributed-app"></a><span data-ttu-id="ce9f3-153">1 부: 지리적으로 분산 된 앱 만들기</span><span class="sxs-lookup"><span data-stu-id="ce9f3-153">Part 1: Create a geo-distributed app</span></span>

<span data-ttu-id="ce9f3-154">이 부분에서는 웹 앱을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-154">In this part, you'll create a web app.</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="ce9f3-155">웹 앱을 만들고 게시 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-155">Create web apps and publish.</span></span>
> - <span data-ttu-id="ce9f3-156">Azure Repos에 코드를 추가 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-156">Add code to Azure Repos.</span></span>
> - <span data-ttu-id="ce9f3-157">앱이 여러 클라우드 대상에 대해 빌드를 가리킵니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-157">Point the app build to multiple cloud targets.</span></span>
> - <span data-ttu-id="ce9f3-158">CD 프로세스를 관리 하 고 구성 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-158">Manage and configure the CD process.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="ce9f3-159">필수 조건</span><span class="sxs-lookup"><span data-stu-id="ce9f3-159">Prerequisites</span></span>

<span data-ttu-id="ce9f3-160">Azure 구독 및 Azure Stack 허브 설치가 필요 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-160">An Azure subscription and Azure Stack Hub installation are required.</span></span>

### <a name="geo-distributed-app-steps"></a><span data-ttu-id="ce9f3-161">지리적으로 분산 되는 앱 단계</span><span class="sxs-lookup"><span data-stu-id="ce9f3-161">Geo-distributed app steps</span></span>

### <a name="obtain-a-custom-domain-and-configure-dns"></a><span data-ttu-id="ce9f3-162">사용자 지정 도메인 가져오기 및 DNS 구성</span><span class="sxs-lookup"><span data-stu-id="ce9f3-162">Obtain a custom domain and configure DNS</span></span>

<span data-ttu-id="ce9f3-163">도메인에 대 한 DNS 영역 파일을 업데이트 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-163">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="ce9f3-164">그러면 Azure AD에서 사용자 지정 도메인 이름의 소유권을 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-164">Azure AD can then verify ownership of the custom domain name.</span></span> <span data-ttu-id="ce9f3-165">Azure 내에서 Azure/Office 365/외부 DNS 레코드에 대 한 [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) 를 사용 하거나 [다른 DNS 등록자](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/)에 dns 항목을 추가 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-165">Use [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) for Azure/Office 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).</span></span>

1. <span data-ttu-id="ce9f3-166">공용 등록자에 게 사용자 지정 도메인을 등록 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-166">Register a custom domain with a public registrar.</span></span>

2. <span data-ttu-id="ce9f3-167">도메인에 대한 도메인 이름 등록 기관에 로그인합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-167">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="ce9f3-168">승인 된 관리자가 DNS 업데이트를 수행 해야 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-168">An approved admin may be required to make the DNS updates.</span></span>

3. <span data-ttu-id="ce9f3-169">Azure AD에서 제공 하는 DNS 항목을 추가 하 여 도메인에 대 한 DNS 영역 파일을 업데이트 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-169">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="ce9f3-170">DNS 항목은 메일 라우팅 또는 웹 호스팅 등의 동작을 변경 하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-170">The DNS entry doesn't change behaviors such as mail routing or web hosting.</span></span>

### <a name="create-web-apps-and-publish"></a><span data-ttu-id="ce9f3-171">웹 앱 만들기 및 게시</span><span class="sxs-lookup"><span data-stu-id="ce9f3-171">Create web apps and publish</span></span>

<span data-ttu-id="ce9f3-172">웹 앱을 Azure Stack Azure에 배포 하는 CI/CD (연속 통합/지속적인 업데이트)를 설정 하 고, 두 클라우드 모두에 변경 내용을 자동으로 푸시합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-172">Set up Hybrid Continuous Integration/Continuous Delivery (CI/CD) to deploy Web App to Azure and Azure Stack Hub, and auto push changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="ce9f3-173">적절 한 이미지 게시를 실행 하는 Azure Stack 허브 (Windows Server 및 SQL) 및 App Service 배포가 필요 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-173">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="ce9f3-174">자세한 내용은 [Azure Stack 허브에 App Service를 배포 하기 위한 필수 조건](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md)을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-174">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

#### <a name="add-code-to-azure-repos"></a><span data-ttu-id="ce9f3-175">Azure Repos에 코드 추가</span><span class="sxs-lookup"><span data-stu-id="ce9f3-175">Add Code to Azure Repos</span></span>

1. <span data-ttu-id="ce9f3-176">Azure Repos에 대 한 **프로젝트 만들기 권한이 있는 계정** 으로 Visual Studio에 로그인 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-176">Sign in to Visual Studio with an **account that has project creation rights** on Azure Repos.</span></span>

    <span data-ttu-id="ce9f3-177">CI/CD는 앱 코드와 인프라 코드 모두에 적용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-177">CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="ce9f3-178">사설 및 호스 티 드 클라우드 개발 모두에 [Azure Resource Manager 템플릿을](https://azure.microsoft.com/resources/templates/) 사용 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-178">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Visual Studio에서 프로젝트에 연결](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. <span data-ttu-id="ce9f3-180">기본 웹 앱을 만들고 열어 **리포지토리를 복제** 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-180">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Visual Studio에서 리포지토리 복제](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a><span data-ttu-id="ce9f3-182">두 클라우드에서 웹 앱 배포 만들기</span><span class="sxs-lookup"><span data-stu-id="ce9f3-182">Create web app deployment in both clouds</span></span>

1. <span data-ttu-id="ce9f3-183">**WebApplication .csproj** 파일 (선택 `Runtimeidentifier` 및 추가)을 편집 `win10-x64` 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-183">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="ce9f3-184">[자체 포함 된 배포](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) 설명서를 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-184">(See [Self-contained Deployment](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Visual Studio에서 웹 앱 프로젝트 파일 편집](media/solution-deployment-guide-geo-distributed/image3.png)

2. <span data-ttu-id="ce9f3-186">팀 탐색기를 사용 하 여 **Azure Repos 코드를 체크 인** 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-186">**Check in the code to Azure Repos** using Team Explorer.</span></span>

3. <span data-ttu-id="ce9f3-187">**응용 프로그램 코드가** Azure Repos 체크 인 되었는지 확인 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-187">Confirm that the **application code** has been checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="ce9f3-188">빌드 정의 만들기</span><span class="sxs-lookup"><span data-stu-id="ce9f3-188">Create the build definition</span></span>

1. <span data-ttu-id="ce9f3-189">**Azure Pipelines에 로그인** 하 여 빌드 정의를 만들 수 있는지 확인 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-189">**Sign in to Azure Pipelines** to confirm ability to create build definitions.</span></span>

2. <span data-ttu-id="ce9f3-190">`-r win10-x64`코드를 추가 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-190">Add `-r win10-x64` code.</span></span> <span data-ttu-id="ce9f3-191">이러한 추가는 .NET Core를 사용 하 여 자체 포함 된 배포를 트리거하기 위해 필요 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-191">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Azure Pipelines의 빌드 정의에 코드를 추가 합니다.](media/solution-deployment-guide-geo-distributed/image4.png)

3. <span data-ttu-id="ce9f3-193">**빌드를 실행**합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-193">**Run the build**.</span></span> <span data-ttu-id="ce9f3-194">[자체 포함 배포 빌드](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) 프로세스는 Azure 및 Azure Stack 허브에서 실행할 수 있는 아티팩트를 게시 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-194">The [self-contained deployment build](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="using-an-azure-hosted-agent"></a><span data-ttu-id="ce9f3-195">Azure 호스트 된 에이전트 사용</span><span class="sxs-lookup"><span data-stu-id="ce9f3-195">Using an Azure Hosted Agent</span></span>

<span data-ttu-id="ce9f3-196">Azure Pipelines에서 호스팅된 에이전트를 사용 하는 것은 웹 앱을 빌드하고 배포 하는 편리한 옵션입니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-196">Using a hosted agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="ce9f3-197">유지 관리 및 업그레이드는 중단 없이 개발, 테스트 및 배포를 가능 하 게 하는 Microsoft Azure에 의해 자동으로 수행 됩니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-197">Maintenance and upgrades are automatically performed by Microsoft Azure, which enables uninterrupted development, testing, and deployment.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="ce9f3-198">CD 프로세스 관리 및 구성</span><span class="sxs-lookup"><span data-stu-id="ce9f3-198">Manage and configure the CD process</span></span>

<span data-ttu-id="ce9f3-199">개발, 스테이징, QA 및 프로덕션 환경과 같은 여러 환경에 릴리스를 위한 매우 구성 가능 하 고 관리 하기 쉬운 파이프라인을 제공 Azure DevOps Services 합니다. 특정 단계에서 승인 요구를 포함 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-199">Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments such as development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="ce9f3-200">릴리스 정의 만들기</span><span class="sxs-lookup"><span data-stu-id="ce9f3-200">Create release definition</span></span>

1. <span data-ttu-id="ce9f3-201">**더하기** 단추를 선택 **하 여 Azure DevOps Services** 의 **빌드 및 릴리스** 섹션에서 릴리스 탭 아래에 새 릴리스를 추가 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-201">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Azure DevOps Services에서 릴리스 정의 만들기](media/solution-deployment-guide-geo-distributed/image5.png)

2. <span data-ttu-id="ce9f3-203">Azure App Service 배포 템플릿을 적용 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-203">Apply the Azure App Service Deployment template.</span></span>

   ![Azure DevOps Services에서 Azure App Service 배포 템플릿 적용](media/solution-deployment-guide-geo-distributed/image6.png)

3. <span data-ttu-id="ce9f3-205">**아티팩트 추가**에서 Azure 클라우드 빌드 앱에 대 한 아티팩트를 추가 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-205">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Azure DevOps Services에서 Azure 클라우드 빌드에 아티팩트 추가](media/solution-deployment-guide-geo-distributed/image7.png)

4. <span data-ttu-id="ce9f3-207">파이프라인 탭에서 환경의 **단계, 작업** 링크를 선택 하 고 Azure 클라우드 환경 값을 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-207">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Azure DevOps Services에서 Azure 클라우드 환경 값 설정](media/solution-deployment-guide-geo-distributed/image8.png)

5. <span data-ttu-id="ce9f3-209">**환경 이름을** 설정 하 고 azure 클라우드 끝점에 대 한 **azure 구독** 을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-209">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Azure DevOps Services에서 Azure 클라우드 끝점에 대 한 Azure 구독을 선택 합니다.](media/solution-deployment-guide-geo-distributed/image9.png)

6. <span data-ttu-id="ce9f3-211">**App service name**에서 필요한 Azure App service 이름을 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-211">Under **App service name**, set the required Azure app service name.</span></span>

      ![Azure DevOps Services에서 Azure app service 이름 설정](media/solution-deployment-guide-geo-distributed/image10.png)

7. <span data-ttu-id="ce9f3-213">Azure 클라우드 호스팅 환경에 대 한 **에이전트 큐** 아래에 "Hosted VS2017"를 입력 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-213">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Azure DevOps Services에서 Azure 클라우드 호스팅 환경에 대 한 에이전트 큐 설정](media/solution-deployment-guide-geo-distributed/image11.png)

8. <span data-ttu-id="ce9f3-215">배포 Azure App Service 메뉴에서 환경의 올바른 **패키지 또는 폴더** 를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-215">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="ce9f3-216">**폴더 위치**에 **확인** 을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-216">Select **OK** to **folder location**.</span></span>
  
      ![Azure DevOps Services에서 Azure App Service 환경을 위한 패키지 또는 폴더를 선택 합니다.](media/solution-deployment-guide-geo-distributed/image12.png)

      ![Azure DevOps Services에서 Azure App Service 환경을 위한 패키지 또는 폴더를 선택 합니다.](media/solution-deployment-guide-geo-distributed/image13.png)

9. <span data-ttu-id="ce9f3-219">모든 변경 내용을 저장 하 고 **릴리스 파이프라인**으로 돌아갑니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-219">Save all changes and go back to **release pipeline**.</span></span>

    ![Azure DevOps Services의 릴리스 파이프라인 변경 내용 저장](media/solution-deployment-guide-geo-distributed/image14.png)

10. <span data-ttu-id="ce9f3-221">Azure Stack 허브 앱에 대 한 빌드를 선택 하는 새 아티팩트를 추가 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-221">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Azure DevOps Services에서 Azure Stack 허브 앱에 대 한 새 아티팩트 추가](media/solution-deployment-guide-geo-distributed/image15.png)


11. <span data-ttu-id="ce9f3-223">Azure App Service 배포를 적용 하 여 환경을 하나 더 추가 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-223">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Azure DevOps Services에서 Azure App Service 배포에 환경 추가](media/solution-deployment-guide-geo-distributed/image16.png)

12. <span data-ttu-id="ce9f3-225">새 환경의 이름을 Hub Azure Stack 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-225">Name the new environment Azure Stack Hub.</span></span>

    ![Azure DevOps Services에서 Azure App Service 배포의 이름 환경](media/solution-deployment-guide-geo-distributed/image17.png)

13. <span data-ttu-id="ce9f3-227">**작업** 탭에서 Azure Stack 허브 환경을 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-227">Find the Azure Stack Hub environment under **Task** tab.</span></span>

    ![Azure DevOps Services의 Azure DevOps Services에 Azure Stack 허브 환경](media/solution-deployment-guide-geo-distributed/image18.png)

14. <span data-ttu-id="ce9f3-229">Azure Stack 허브 끝점에 대 한 구독을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-229">Select the subscription for the Azure Stack Hub endpoint.</span></span>

    ![Azure DevOps Services에서 Azure Stack 허브 끝점에 대 한 구독을 선택 합니다.](media/solution-deployment-guide-geo-distributed/image19.png)

15. <span data-ttu-id="ce9f3-231">Azure Stack 허브 웹 앱 이름을 App service 이름으로 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-231">Set the Azure Stack Hub web app name as the App service name.</span></span>

    ![Azure DevOps Services에서 Azure Stack 허브 웹 앱 이름 설정](media/solution-deployment-guide-geo-distributed/image20.png)

16. <span data-ttu-id="ce9f3-233">Azure Stack 허브 에이전트를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-233">Select the Azure Stack Hub agent.</span></span>

    ![Azure DevOps Services에서 Azure Stack 허브 에이전트를 선택 합니다.](media/solution-deployment-guide-geo-distributed/image21.png)

17. <span data-ttu-id="ce9f3-235">배포 Azure App Service 섹션에서 환경의 올바른 **패키지 또는 폴더** 를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-235">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="ce9f3-236">폴더 위치에 **확인** 을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-236">Select **OK** to folder location.</span></span>

    ![Azure DevOps Services에서 Azure App Service 배포를 위한 폴더를 선택 하십시오.](media/solution-deployment-guide-geo-distributed/image22.png)

    ![Azure DevOps Services에서 Azure App Service 배포를 위한 폴더를 선택 하십시오.](media/solution-deployment-guide-geo-distributed/image23.png)

18. <span data-ttu-id="ce9f3-239">변수 탭에서 라는 변수를 추가 하 `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` 고 해당 값을 **true**로, 범위를 Azure Stack 허브로 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-239">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack Hub.</span></span>

    ![Azure DevOps Services에서 Azure 앱 배포에 변수 추가](media/solution-deployment-guide-geo-distributed/image24.png)

19. <span data-ttu-id="ce9f3-241">두 아티팩트의 **연속** 배포 트리거 아이콘을 선택 하 고 **계속 해 서** 배포 트리거를 사용 하도록 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-241">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Azure DevOps Services에서 연속 배포 트리거를 선택 합니다.](media/solution-deployment-guide-geo-distributed/image25.png)

20. <span data-ttu-id="ce9f3-243">Azure Stack 허브 환경에서 **배포 전** 조건 아이콘을 선택 하 고 트리거를 **릴리스 후로 설정 합니다.**</span><span class="sxs-lookup"><span data-stu-id="ce9f3-243">Select the **Pre-deployment** conditions icon in the Azure Stack Hub environment and set the trigger to **After release.**</span></span>

    ![Azure DevOps Services에서 배포 전 조건 선택](media/solution-deployment-guide-geo-distributed/image26.png)

21. <span data-ttu-id="ce9f3-245">모든 변경 내용을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-245">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="ce9f3-246">템플릿에서 릴리스 정의를 만들 때 작업에 대 한 일부 설정이 [환경 변수로](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) 자동으로 정의 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-246">Some settings for the tasks may have been automatically defined as [environment variables](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="ce9f3-247">이러한 설정은 작업 설정에서 수정할 수 없습니다. 대신 이러한 설정을 편집 하려면 부모 환경 항목을 선택 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-247">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="part-2-update-web-app-options"></a><span data-ttu-id="ce9f3-248">2 부: 웹 앱 옵션 업데이트</span><span class="sxs-lookup"><span data-stu-id="ce9f3-248">Part 2: Update web app options</span></span>

<span data-ttu-id="ce9f3-249">[Azure App Service](https://docs.microsoft.com/azure/app-service/overview)는 확장성 높은 자체 패치 웹 호스팅 서비스를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-249">[Azure App Service](https://docs.microsoft.com/azure/app-service/overview) provides a highly scalable, self-patching web hosting service.</span></span>

![Azure App Service](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - <span data-ttu-id="ce9f3-251">기존 사용자 지정 DNS 이름을 Azure Web Apps에 매핑합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-251">Map an existing custom DNS name to Azure Web Apps.</span></span>
> - <span data-ttu-id="ce9f3-252">**CNAME 레코드** 와 **a 레코드** 를 사용 하 여 사용자 지정 DNS 이름을 App Service에 매핑합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-252">Use a **CNAME record** and an **A record** to map a custom DNS name to App Service.</span></span>

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a><span data-ttu-id="ce9f3-253">Azure Web Apps에 기존 사용자 지정 DNS 이름 매핑</span><span class="sxs-lookup"><span data-stu-id="ce9f3-253">Map an existing custom DNS name to Azure Web Apps</span></span>

> [!Note]  
> <span data-ttu-id="ce9f3-254">루트 도메인 (예: northwind.com)을 제외한 모든 사용자 지정 DNS 이름에 대해 CNAME을 사용 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-254">Use a CNAME for all custom DNS names except a root domain (for example, northwind.com).</span></span>

<span data-ttu-id="ce9f3-255">라이브 사이트 및 해당 DNS 도메인 이름을 App Service로 마이그레이션하려면 [활성 DNS 이름을 Azure App Service로 마이그레이션](https://docs.microsoft.com/azure/app-service/manage-custom-dns-migrate-domain)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-255">To migrate a live site and its DNS domain name to App Service, see [Migrate an active DNS name to Azure App Service](https://docs.microsoft.com/azure/app-service/manage-custom-dns-migrate-domain).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="ce9f3-256">필수 조건</span><span class="sxs-lookup"><span data-stu-id="ce9f3-256">Prerequisites</span></span>

<span data-ttu-id="ce9f3-257">이 솔루션을 완료 하려면:</span><span class="sxs-lookup"><span data-stu-id="ce9f3-257">To complete this solution:</span></span>

- <span data-ttu-id="ce9f3-258">[App Service 앱을 만들거나](https://docs.microsoft.com/azure/app-service/)다른 솔루션에 대해 만든 앱을 사용 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-258">[Create an App Service app](https://docs.microsoft.com/azure/app-service/), or use an app created for another  solution.</span></span>

- <span data-ttu-id="ce9f3-259">도메인 이름을 구매 하 고 도메인 공급자의 DNS 레지스트리에 대 한 액세스를 확인 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-259">Purchase a domain name and ensure access to the DNS registry for the domain provider.</span></span>

<span data-ttu-id="ce9f3-260">도메인에 대 한 DNS 영역 파일을 업데이트 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-260">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="ce9f3-261">Azure AD에서 사용자 지정 도메인 이름의 소유권을 확인 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-261">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="ce9f3-262">Azure 내에서 Azure/Office 365/외부 DNS 레코드에 대 한 [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) 를 사용 하거나 [다른 DNS 등록자](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/)에 dns 항목을 추가 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-262">Use [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) for Azure/Office 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).</span></span>

- <span data-ttu-id="ce9f3-263">공용 등록자에 게 사용자 지정 도메인을 등록 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-263">Register a custom domain with a public registrar.</span></span>

- <span data-ttu-id="ce9f3-264">도메인에 대한 도메인 이름 등록 기관에 로그인합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-264">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="ce9f3-265">(승인 된 관리자는 DNS를 업데이트 해야 할 수도 있습니다.)</span><span class="sxs-lookup"><span data-stu-id="ce9f3-265">(An approved admin may be required to make DNS updates.)</span></span>

- <span data-ttu-id="ce9f3-266">Azure AD에서 제공 하는 DNS 항목을 추가 하 여 도메인에 대 한 DNS 영역 파일을 업데이트 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-266">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span>

<span data-ttu-id="ce9f3-267">예를 들어 northwindcloud.com 및 www northwindcloud.com에 대 한 DNS 항목을 추가 하려면 \. northwindcloud.com 루트 도메인에 대 한 dns 설정을 구성 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-267">For example, to add DNS entries for northwindcloud.com and www\.northwindcloud.com, configure DNS settings for the northwindcloud.com root domain.</span></span>

> [!Note]  
> <span data-ttu-id="ce9f3-268">[Azure Portal](https://docs.microsoft.com/azure/app-service/manage-custom-dns-buy-domain)를 사용 하 여 도메인 이름을 구입할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-268">A domain name may be purchased using the [Azure portal](https://docs.microsoft.com/azure/app-service/manage-custom-dns-buy-domain).</span></span> <span data-ttu-id="ce9f3-269">사용자 지정 DNS 이름을 웹앱에 매핑하려면 [App Service 계획](https://azure.microsoft.com/pricing/details/app-service/)이 유료 계층(**공유**, **기본**, **표준** 또는 **프리미엄**)이어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-269">To map a custom DNS name to a web app, the web app's [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be a paid tier (**Shared**, **Basic**, **Standard**, or **Premium**).</span></span>

### <a name="create-and-map-cname-and-a-records"></a><span data-ttu-id="ce9f3-270">CNAME 및 A 레코드 만들기 및 매핑</span><span class="sxs-lookup"><span data-stu-id="ce9f3-270">Create and map CNAME and A records</span></span>

#### <a name="access-dns-records-with-domain-provider"></a><span data-ttu-id="ce9f3-271">도메인 공급자로 DNS 레코드 액세스</span><span class="sxs-lookup"><span data-stu-id="ce9f3-271">Access DNS records with domain provider</span></span>

> [!Note]  
>  <span data-ttu-id="ce9f3-272">Azure DNS를 사용 하 여 Azure Web Apps에 대 한 사용자 지정 DNS 이름을 구성 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-272">Use Azure DNS to configure a custom DNS name for Azure Web Apps.</span></span> <span data-ttu-id="ce9f3-273">자세한 내용은 [Azure DNS를 사용하여 Azure 서비스에 대해 사용자 지정 도메인 설정 제공](https://docs.microsoft.com/azure/dns/dns-custom-domain)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-273">For more information, see [Use Azure DNS to provide custom domain settings for an Azure service](https://docs.microsoft.com/azure/dns/dns-custom-domain).</span></span>

1. <span data-ttu-id="ce9f3-274">주 공급자의 웹 사이트에 로그인 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-274">Sign in to the website of the main provider.</span></span>

2. <span data-ttu-id="ce9f3-275">DNS 레코드를 관리하기 위한 페이지를 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-275">Find the page for managing DNS records.</span></span> <span data-ttu-id="ce9f3-276">모든 도메인 공급자에는 자체 DNS 레코드 인터페이스가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-276">Every domain provider has its own DNS records interface.</span></span> <span data-ttu-id="ce9f3-277">**도메인 이름**, **DNS** 또는 **이름 서버 관리**라는 레이블이 지정된 사이트 영역을 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-277">Look for areas of the site labeled **Domain Name**, **DNS**, or **Name Server Management**.</span></span>

<span data-ttu-id="ce9f3-278">DNS 레코드 페이지는 **내 도메인**에서 볼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-278">DNS records page can be viewed in **My domains**.</span></span> <span data-ttu-id="ce9f3-279">**영역 파일**, **DNS 레코드**또는 **고급 구성**이라는 링크를 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-279">Find the link named **Zone file**, **DNS Records**, or **Advanced configuration**.</span></span>

<span data-ttu-id="ce9f3-280">다음 스크린샷은 DNS 레코드 페이지의 예입니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-280">The following screenshot is an example of a DNS records page:</span></span>

![DNS 레코드 페이지 예](media/solution-deployment-guide-geo-distributed/image28.png)

1. <span data-ttu-id="ce9f3-282">도메인 이름 등록자에서 **추가 또는 만들기** 를 선택 하 여 레코드를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-282">In Domain Name Registrar, select **Add or Create** to create a record.</span></span> <span data-ttu-id="ce9f3-283">일부 공급자에는 다른 레코드 형식을 추가하는 다양한 링크가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-283">Some providers have different links to add different record types.</span></span> <span data-ttu-id="ce9f3-284">공급자 설명서를 참조 하십시오.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-284">Consult the provider's documentation.</span></span>

2. <span data-ttu-id="ce9f3-285">CNAME 레코드를 추가 하 여 하위 도메인을 앱의 기본 호스트 이름에 매핑합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-285">Add a CNAME record to map a subdomain to the app's default hostname.</span></span>

   <span data-ttu-id="ce9f3-286">Www \. northwindcloud.com 도메인 예의 경우 이름을에 매핑하는 CNAME 레코드를 추가 `<app_name>.azurewebsites.net` 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-286">For the www\.northwindcloud.com domain example, add a CNAME record that maps the name to `<app_name>.azurewebsites.net`.</span></span>

<span data-ttu-id="ce9f3-287">CNAME을 추가 하면 DNS 레코드 페이지가 다음 예제와 같습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-287">After adding the CNAME, the DNS records page looks like the following example:</span></span>

![Azure 앱에 대한 포털 탐색](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a><span data-ttu-id="ce9f3-289">Azure에서 CNAME 레코드 매핑 사용</span><span class="sxs-lookup"><span data-stu-id="ce9f3-289">Enable the CNAME record mapping in Azure</span></span>

1. <span data-ttu-id="ce9f3-290">새 탭에서 Azure Portal에 로그인 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-290">In a new tab, sign in to the Azure portal.</span></span>

2. <span data-ttu-id="ce9f3-291">App Services로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-291">Go to App Services.</span></span>

3. <span data-ttu-id="ce9f3-292">웹 앱을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-292">Select web app.</span></span>

4. <span data-ttu-id="ce9f3-293">Azure Portal의 앱 페이지 왼쪽 탐색 영역에서 **사용자 지정 도메인**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-293">In the left navigation of the app page in the Azure portal, select **Custom domains**.</span></span>

5. <span data-ttu-id="ce9f3-294">**+** **호스트 이름 추가**옆에 있는 아이콘을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-294">Select the **+** icon next to **Add hostname**.</span></span>

6. <span data-ttu-id="ce9f3-295">정규화 된 도메인 이름 (예:)을 입력 합니다 `www.northwindcloud.com` .</span><span class="sxs-lookup"><span data-stu-id="ce9f3-295">Type the fully qualified domain name, like `www.northwindcloud.com`.</span></span>

7. <span data-ttu-id="ce9f3-296">**유효성 검사**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-296">Select **Validate**.</span></span>

8. <span data-ttu-id="ce9f3-297">표시 되는 경우 다른 형식 (또는)의 다른 레코드 `A` `TXT` 를 도메인 이름 등록 기관 DNS 레코드에 추가 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-297">If indicated, add additional records of other types (`A` or `TXT`) to the domain name registrars DNS records.</span></span> <span data-ttu-id="ce9f3-298">Azure는 이러한 레코드의 값과 유형을 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-298">Azure will provide the values and types of these records:</span></span>

   <span data-ttu-id="ce9f3-299">a.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-299">a.</span></span>  <span data-ttu-id="ce9f3-300">앱의 IP 주소에 매핑할 **A** 레코드</span><span class="sxs-lookup"><span data-stu-id="ce9f3-300">An **A** record to map to the app's IP address.</span></span>

   <span data-ttu-id="ce9f3-301">b.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-301">b.</span></span>  <span data-ttu-id="ce9f3-302">앱의 기본 호스트 이름(`<app_name>.azurewebsites.net`)에 매핑할 **TXT** 레코드 -</span><span class="sxs-lookup"><span data-stu-id="ce9f3-302">A **TXT** record to map to the app's default hostname `<app_name>.azurewebsites.net`.</span></span> <span data-ttu-id="ce9f3-303">App Service는 구성 시에만이 레코드를 사용 하 여 사용자 지정 도메인 소유권을 확인 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-303">App Service uses this record only at configuration time to verify custom domain ownership.</span></span> <span data-ttu-id="ce9f3-304">확인 후 TXT 레코드를 삭제 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-304">After verification, delete the TXT record.</span></span>

9. <span data-ttu-id="ce9f3-305">도메인 등록자 탭에서이 작업을 완료 하 고 **호스트 이름 추가** 단추가 활성화 될 때까지 유효성을 다시 검사 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-305">Complete this task in the domain registrar tab and revalidate until the **Add hostname** button is activated.</span></span>

10. <span data-ttu-id="ce9f3-306">**호스트 이름 레코드 형식이** **CNAME** (www.example.com 또는 하위 도메인)으로 설정 되어 있는지 확인 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-306">Make sure that **Hostname record type** is set to **CNAME** (www.example.com or any subdomain).</span></span>

11. <span data-ttu-id="ce9f3-307">**호스트 이름 추가**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-307">Select **Add hostname**.</span></span>

12. <span data-ttu-id="ce9f3-308">정규화 된 도메인 이름 (예:)을 입력 합니다 `northwindcloud.com` .</span><span class="sxs-lookup"><span data-stu-id="ce9f3-308">Type the fully qualified domain name, like `northwindcloud.com`.</span></span>

13. <span data-ttu-id="ce9f3-309">**유효성 검사**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-309">Select **Validate**.</span></span> <span data-ttu-id="ce9f3-310">**추가** 가 활성화 됩니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-310">The **Add** is activated.</span></span>

14. <span data-ttu-id="ce9f3-311">**호스트 이름 레코드 형식이** **레코드** (example.com)로 설정 되어 있는지 확인 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-311">Make sure that **Hostname record type** is set to **A record** (example.com).</span></span>

15. <span data-ttu-id="ce9f3-312">**호스트 이름을 추가**합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-312">**Add hostname**.</span></span>

    <span data-ttu-id="ce9f3-313">새 호스트 이름이 앱의 **사용자 지정 도메인** 페이지에 반영 되는 데 약간의 시간이 걸릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-313">It might take some time for the new hostnames to be reflected in the app's **Custom domains** page.</span></span> <span data-ttu-id="ce9f3-314">데이터를 업데이트하려면 브라우저를 새로 고쳐 보세요.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-314">Try refreshing the browser to update the data.</span></span>
  
    ![사용자 지정 도메인](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    <span data-ttu-id="ce9f3-316">오류가 발생 하면 페이지 아래쪽에 확인 오류 알림이 표시 됩니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-316">If there's an error, a verification error notification will appear at the bottom of the page.</span></span> ![도메인 확인 오류](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  <span data-ttu-id="ce9f3-318">위의 단계를 반복 하 여 와일드 카드 도메인 ( \* . northwindcloud.com)을 매핑할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-318">The above steps may be repeated to map a wildcard domain (\*.northwindcloud.com).</span></span> <span data-ttu-id="ce9f3-319">이렇게 하면 각 항목에 대 한 별도의 CNAME 레코드를 만들지 않고도이 app service에 추가 하위 도메인을 추가할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-319">This allows the addition of any additional subdomains to this app service without having to create a separate CNAME record for each one.</span></span> <span data-ttu-id="ce9f3-320">등록자 지침에 따라이 설정을 구성 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-320">Follow the registrar instructions to configure this setting.</span></span>

#### <a name="test-in-a-browser"></a><span data-ttu-id="ce9f3-321">브라우저에서 테스트</span><span class="sxs-lookup"><span data-stu-id="ce9f3-321">Test in a browser</span></span>

<span data-ttu-id="ce9f3-322">이전에 구성 된 DNS 이름 (예: `northwindcloud.com` 또는)으로 이동 `www.northwindcloud.com` 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-322">Browse to the DNS name(s) configured earlier (for example, `northwindcloud.com` or `www.northwindcloud.com`).</span></span>

## <a name="part-3-bind-a-custom-ssl-cert"></a><span data-ttu-id="ce9f3-323">3 부: 사용자 지정 SSL 인증서 바인딩</span><span class="sxs-lookup"><span data-stu-id="ce9f3-323">Part 3: Bind a custom SSL cert</span></span>

<span data-ttu-id="ce9f3-324">이 부분에서는 다음을 수행 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-324">In this part, we will:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="ce9f3-325">App Service에 사용자 지정 SSL 인증서를 바인딩합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-325">Bind the custom SSL certificate to App Service.</span></span>
> - <span data-ttu-id="ce9f3-326">앱에 HTTPS를 적용 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-326">Enforce HTTPS for the app.</span></span>
> - <span data-ttu-id="ce9f3-327">스크립트를 사용 하 여 SSL 인증서 바인딩을 자동화 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-327">Automate SSL certificate binding with scripts.</span></span>

> [!Note]  
> <span data-ttu-id="ce9f3-328">필요한 경우 Azure Portal에서 고객 SSL 인증서를 가져와 웹 앱에 바인딩합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-328">If needed, obtain a customer SSL certificate in the Azure portal and bind it to the web app.</span></span> <span data-ttu-id="ce9f3-329">자세한 내용은 [App Service 인증서 자습서](https://docs.microsoft.com/azure/app-service/web-sites-purchase-ssl-web-site)를 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-329">For more information, see the [App Service Certificates tutorial](https://docs.microsoft.com/azure/app-service/web-sites-purchase-ssl-web-site).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="ce9f3-330">필수 조건</span><span class="sxs-lookup"><span data-stu-id="ce9f3-330">Prerequisites</span></span>

<span data-ttu-id="ce9f3-331">이 솔루션을 완료 하려면:</span><span class="sxs-lookup"><span data-stu-id="ce9f3-331">To complete this  solution:</span></span>

- [<span data-ttu-id="ce9f3-332">App Service 앱을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-332">Create an App Service app.</span></span>](https://docs.microsoft.com/azure/app-service/)
- [<span data-ttu-id="ce9f3-333">사용자 지정 DNS 이름을 웹 앱에 매핑합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-333">Map a custom DNS name to your web app.</span></span>](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain)
- <span data-ttu-id="ce9f3-334">신뢰할 수 있는 인증 기관에서 SSL 인증서를 획득 하 고 키를 사용 하 여 요청에 서명 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-334">Acquire an SSL certificate from a trusted certificate authority and use the key to sign the request.</span></span>

### <a name="requirements-for-your-ssl-certificate"></a><span data-ttu-id="ce9f3-335">SSL 인증서에 대한 요구 사항</span><span class="sxs-lookup"><span data-stu-id="ce9f3-335">Requirements for your SSL certificate</span></span>

<span data-ttu-id="ce9f3-336">App Service에서 인증서를 사용하려면 인증서가 다음 요구 사항을 모두 충족해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-336">To use a certificate in App Service, the certificate must meet all the following requirements:</span></span>

- <span data-ttu-id="ce9f3-337">신뢰할 수 있는 인증 기관에서 서명 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-337">Signed by a trusted certificate authority.</span></span>

- <span data-ttu-id="ce9f3-338">암호로 보호 된 PFX 파일로 내보냅니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-338">Exported as a password-protected PFX file.</span></span>

- <span data-ttu-id="ce9f3-339">최소 2048 비트 길이의 개인 키를 포함 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-339">Contains private key at least 2048 bits long.</span></span>

- <span data-ttu-id="ce9f3-340">인증서 체인의 모든 중간 인증서를 포함 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-340">Contains all intermediate certificates in the certificate chain.</span></span>

> [!Note]  
> <span data-ttu-id="ce9f3-341">**ECC (타원 Curve Cryptography) 인증서** App Service에서 작동 하지만이 가이드에는 포함 되어 있지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-341">**Elliptic Curve Cryptography (ECC) certificates** work with App Service but aren't included in this guide.</span></span> <span data-ttu-id="ce9f3-342">ECC 인증서를 만드는 방법에 대 한 자세한 내용은 인증 기관을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-342">Consult a certificate authority for assistance in creating ECC certificates.</span></span>

#### <a name="prepare-the-web-app"></a><span data-ttu-id="ce9f3-343">웹 앱 준비</span><span class="sxs-lookup"><span data-stu-id="ce9f3-343">Prepare the web app</span></span>

<span data-ttu-id="ce9f3-344">웹 앱에 사용자 지정 SSL 인증서를 바인딩하려면 [App Service 계획이](https://azure.microsoft.com/pricing/details/app-service/) **기본**, **표준**또는 **프리미엄** 계층에 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-344">To bind a custom SSL certificate to the web app, the [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be in the **Basic**, **Standard**, or **Premium** tier.</span></span>

#### <a name="sign-in-to-azure"></a><span data-ttu-id="ce9f3-345">Azure에 로그인</span><span class="sxs-lookup"><span data-stu-id="ce9f3-345">Sign in to Azure</span></span>

1. <span data-ttu-id="ce9f3-346">[Azure Portal](https://portal.azure.com/) 열고 웹 앱으로 이동 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-346">Open the [Azure portal](https://portal.azure.com/) and go to the web app.</span></span>

2. <span data-ttu-id="ce9f3-347">왼쪽 메뉴에서 **App Services**를 선택 하 고 웹 앱 이름을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-347">From the left menu, select **App Services**, and then select the web app name.</span></span>

![Azure Portal에서 웹 앱을 선택 합니다.](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a><span data-ttu-id="ce9f3-349">가격 책정 계층 확인</span><span class="sxs-lookup"><span data-stu-id="ce9f3-349">Check the pricing tier</span></span>

1. <span data-ttu-id="ce9f3-350">웹 앱 페이지의 왼쪽 탐색 영역에서 **설정** 섹션으로 스크롤하고 강화 **(App Service 계획)** 를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-350">In the left-hand navigation of the web app page, scroll to the **Settings** section and select **Scale up (App Service plan)**.</span></span>

    ![웹 앱의 확장 메뉴](media/solution-deployment-guide-geo-distributed/image34.png)

1. <span data-ttu-id="ce9f3-352">웹 앱이 **무료** 또는 **공유** 계층에 있지 않은지 확인 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-352">Ensure the web app isn't in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="ce9f3-353">웹 앱의 현재 계층이 진한 파란색 상자로 강조 표시 됩니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-353">The web app's current tier is highlighted in a dark blue box.</span></span>

    ![웹 앱의 가격 책정 계층 확인](media/solution-deployment-guide-geo-distributed/image35.png)

<span data-ttu-id="ce9f3-355">사용자 지정 SSL은 **무료** 또는 **공유** 계층에서 지원 되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-355">Custom SSL isn't supported in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="ce9f3-356">Upscale 하려면 다음 섹션 또는 **가격 책정 계층 선택** 페이지의 단계를 수행 하 고 [SSL 인증서 업로드 및 바인딩](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl)을 건너뜁니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-356">To upscale, follow the steps in the next section or the **Choose your pricing tier** page and skip to [Upload and bind your SSL certificate](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

#### <a name="scale-up-your-app-service-plan"></a><span data-ttu-id="ce9f3-357">App Service 계획 강화</span><span class="sxs-lookup"><span data-stu-id="ce9f3-357">Scale up your App Service plan</span></span>

1. <span data-ttu-id="ce9f3-358">**기본**, **표준** 또는 **프리미엄** 계층 중 하나를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-358">Select one of the **Basic**, **Standard**, or **Premium** tiers.</span></span>

2. <span data-ttu-id="ce9f3-359">**선택**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-359">Select **Select**.</span></span>

![웹 앱에 대 한 가격 책정 계층 선택](media/solution-deployment-guide-geo-distributed/image36.png)

<span data-ttu-id="ce9f3-361">알림이 표시 되 면 크기 조정 작업이 완료 됩니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-361">The scale operation is complete when notification is displayed.</span></span>

![강화 알림](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a><span data-ttu-id="ce9f3-363">SSL 인증서 바인딩 및 중간 인증서 병합</span><span class="sxs-lookup"><span data-stu-id="ce9f3-363">Bind your SSL certificate and merge intermediate certificates</span></span>

<span data-ttu-id="ce9f3-364">체인의 여러 인증서를 병합 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-364">Merge multiple certificates in the chain.</span></span>

1. <span data-ttu-id="ce9f3-365">텍스트 편집기에서 받은 **각 인증서를 엽니다** .</span><span class="sxs-lookup"><span data-stu-id="ce9f3-365">**Open each certificate** you received in a text editor.</span></span>

2. <span data-ttu-id="ce9f3-366">*Mergedcertificate*라는 병합 된 인증서에 대 한 파일을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-366">Create a file for the merged certificate called *mergedcertificate.crt*.</span></span> <span data-ttu-id="ce9f3-367">텍스트 편집기에서 각 인증서의 내용을 이 파일에 복사합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-367">In a text editor, copy the content of each certificate into this file.</span></span> <span data-ttu-id="ce9f3-368">사용자 인증서의 순서는 사용자의 인증서로 시작하고 루트 인증서로 끝나는 인증서 체인의 순서에 따라야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-368">The order of your certificates should follow the order in the certificate chain, beginning with your certificate and ending with the root certificate.</span></span> <span data-ttu-id="ce9f3-369">다음 예제와 유사합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-369">It looks like the following example:</span></span>

    ```Text

    -----BEGIN CERTIFICATE-----

    <your entire Base64 encoded SSL certificate>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 1>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 2>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded root certificate>

    -----END CERTIFICATE-----
    ```

#### <a name="export-certificate-to-pfx"></a><span data-ttu-id="ce9f3-370">PFX로 인증서 내보내기</span><span class="sxs-lookup"><span data-stu-id="ce9f3-370">Export certificate to PFX</span></span>

<span data-ttu-id="ce9f3-371">인증서로 생성 된 개인 키를 사용 하 여 병합 된 SSL 인증서를 내보냅니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-371">Export the merged SSL certificate with the private key generated by the certificate.</span></span>

<span data-ttu-id="ce9f3-372">개인 키 파일은 OpenSSL를 통해 생성 됩니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-372">A private key file is created via OpenSSL.</span></span> <span data-ttu-id="ce9f3-373">인증서를 PFX로 내보내려면 다음 명령을 실행 하 고 자리 표시자를 `<private-key-file>` `<merged-certificate-file>` 개인 키 경로 및 병합 된 인증서 파일로 바꿉니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-373">To export the certificate to PFX, run the following command and replace the placeholders `<private-key-file>` and `<merged-certificate-file>` with the private key path and the merged certificate file:</span></span>

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

<span data-ttu-id="ce9f3-374">메시지가 표시 되 면 나중에 App Service 위해 SSL 인증서를 업로드 하기 위한 내보내기 암호를 정의 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-374">When prompted, define an export password for uploading your SSL certificate to App Service later.</span></span>

<span data-ttu-id="ce9f3-375">인증서 요청을 생성 하는 데 IIS 또는 **Certreq.exe** 를 사용 하는 경우 인증서를 로컬 컴퓨터에 설치한 다음 [인증서를 PFX로 내보냅니다](https://technet.microsoft.com/library/cc754329(v=ws.11).aspx).</span><span class="sxs-lookup"><span data-stu-id="ce9f3-375">When IIS or **Certreq.exe** are used to generate the certificate request, install the certificate to a local machine and then [export the certificate to PFX](https://technet.microsoft.com/library/cc754329(v=ws.11).aspx).</span></span>

#### <a name="upload-the-ssl-certificate"></a><span data-ttu-id="ce9f3-376">SSL 인증서 업로드</span><span class="sxs-lookup"><span data-stu-id="ce9f3-376">Upload the SSL certificate</span></span>

1. <span data-ttu-id="ce9f3-377">웹 앱의 왼쪽 탐색 영역에서 **SSL 설정** 을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-377">Select **SSL settings** in the left navigation of the web app.</span></span>

2. <span data-ttu-id="ce9f3-378">**인증서 업로드**를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-378">Select **Upload Certificate**.</span></span>

3. <span data-ttu-id="ce9f3-379">**Pfx 인증서 파일**에서 pfx 파일을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-379">In **PFX Certificate File**, select PFX file.</span></span>

4. <span data-ttu-id="ce9f3-380">**인증서 암호**에 PFX 파일을 내보낼 때 만든 암호를 입력 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-380">In **Certificate password**, type the password created when exporting the PFX file.</span></span>

5. <span data-ttu-id="ce9f3-381">**업로드**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-381">Select **Upload**.</span></span>

    ![SSL 인증서 업로드](media/solution-deployment-guide-geo-distributed/image38.png)

<span data-ttu-id="ce9f3-383">App Service에서 인증서 업로드가 완료 되 면 **SSL 설정** 페이지에 표시 됩니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-383">When App Service finishes uploading the certificate, it appears in the **SSL settings** page.</span></span>

![SSL 설정](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a><span data-ttu-id="ce9f3-385">SSL 인증서 바인딩</span><span class="sxs-lookup"><span data-stu-id="ce9f3-385">Bind your SSL certificate</span></span>

1. <span data-ttu-id="ce9f3-386">**SSL 바인딩** 섹션에서 **바인딩 추가**를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-386">In the **SSL bindings** section, select **Add binding**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="ce9f3-387">인증서가 업로드 되었지만 **호스트** 이름 드롭다운에서 도메인 이름에 표시 되지 않는 경우 브라우저 페이지를 새로 고쳐 보세요.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-387">If the certificate has been uploaded, but doesn't appear in domain name(s) in the **Hostname** dropdown, try refreshing the browser page.</span></span>

2. <span data-ttu-id="ce9f3-388">**SSL 바인딩 추가** 페이지에서 드롭다운을 사용 하 여 보호할 도메인 이름과 사용할 인증서를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-388">In the **Add SSL Binding** page, use the drop downs to select the domain name to secure and the certificate to use.</span></span>

3. <span data-ttu-id="ce9f3-389">**SSL 유형**에서 [**SNI(서버 이름 표시)**](https://en.wikipedia.org/wiki/Server_Name_Indication) 또는 IP 기반 SSL을 사용할지 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-389">In **SSL Type**, select whether to use [**Server Name Indication (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) or IP-based SSL.</span></span>

    - <span data-ttu-id="ce9f3-390">**Sni 기반 ssl**: SNI 기반 ssl 바인딩을 여러 개 추가할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-390">**SNI-based SSL**: Multiple SNI-based SSL bindings may be added.</span></span> <span data-ttu-id="ce9f3-391">이 옵션을 사용하면 여러 SSL 인증서로 같은 IP 주소의 여러 도메인을 보호할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-391">This option allows multiple SSL certificates to secure multiple domains on the same IP address.</span></span> <span data-ttu-id="ce9f3-392">대부분의 최신 브라우저(Internet Explorer, Chrome, Firefox 및 Opera 포함)는 SNI를 지원합니다. [Server Name Indication](https://wikipedia.org/wiki/Server_Name_Indication)(서버 이름 표시)에서 더 포괄적인 브라우저 지원 정보를 찾을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-392">Most modern browsers (including Internet Explorer, Chrome, Firefox, and Opera) support SNI (find more comprehensive browser support information at [Server Name Indication](https://wikipedia.org/wiki/Server_Name_Indication)).</span></span>

    - <span data-ttu-id="ce9f3-393">**Ip 기반 ssl**: IP 기반 ssl 바인딩을 하나만 추가할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-393">**IP-based SSL**: Only one IP-based SSL binding may be added.</span></span> <span data-ttu-id="ce9f3-394">이 옵션을 사용하면 전용 공용 IP 주소를 보호하는 데 하나의 SSL 인증서만 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-394">This option allows only one SSL certificate to secure a dedicated public IP address.</span></span> <span data-ttu-id="ce9f3-395">여러 도메인을 보호 하려면 동일한 SSL 인증서를 사용 하 여 보안을 유지 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-395">To secure multiple domains, secure them all using the same SSL certificate.</span></span> <span data-ttu-id="ce9f3-396">IP 기반 SSL은 SSL 바인딩에 대 한 기존 옵션입니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-396">IP-based SSL is the traditional option for SSL binding.</span></span>

4. <span data-ttu-id="ce9f3-397">**바인딩 추가**를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-397">Select **Add Binding**.</span></span>

    ![SSL 바인딩 추가](media/solution-deployment-guide-geo-distributed/image40.png)

<span data-ttu-id="ce9f3-399">App Service에서 인증서 업로드가 완료 되 면 **SSL 바인딩** 섹션에 표시 됩니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-399">When App Service finishes uploading the certificate, it appears in the **SSL bindings** sections.</span></span>

![SSL 바인딩 업로드 완료](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a><span data-ttu-id="ce9f3-401">IP SSL에 대 한 A 레코드 다시 매핑</span><span class="sxs-lookup"><span data-stu-id="ce9f3-401">Remap the A record for IP SSL</span></span>

<span data-ttu-id="ce9f3-402">웹 앱에서 IP 기반 SSL을 사용 하지 않는 경우 [사용자 지정 도메인에 대해 HTTPS 테스트](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl)로 건너뜁니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-402">If IP-based SSL isn't used in the web app, skip to [Test HTTPS for your custom domain](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

<span data-ttu-id="ce9f3-403">기본적으로 웹 앱은 공유 공용 IP 주소를 사용 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-403">By default, the web app uses a shared public IP address.</span></span> <span data-ttu-id="ce9f3-404">인증서가 IP 기반 SSL에 바인딩된 경우 App Service는 웹 앱에 대 한 새 전용 IP 주소를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-404">When the certificate is bound with IP-based SSL, App Service creates a new and dedicated IP address for the web app.</span></span>

<span data-ttu-id="ce9f3-405">A 레코드가 웹 앱에 매핑되면 도메인 레지스트리는 전용 IP 주소를 사용 하 여 업데이트 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-405">When an A record is mapped to the web app, the domain registry must be updated with the dedicated IP address.</span></span>

<span data-ttu-id="ce9f3-406">**사용자 지정 도메인** 페이지는 새로운 전용 IP 주소로 업데이트 됩니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-406">The **Custom domain** page is updated with the new, dedicated IP address.</span></span> <span data-ttu-id="ce9f3-407">이 [ip 주소](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain)를 복사한 다음이 새 ip 주소에 [A 레코드](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain) 를 다시 매핑합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-407">Copy this [IP address](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain), then remap the [A record](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain) to this new IP address.</span></span>

#### <a name="test-https"></a><span data-ttu-id="ce9f3-408">HTTPS 테스트</span><span class="sxs-lookup"><span data-stu-id="ce9f3-408">Test HTTPS</span></span>

<span data-ttu-id="ce9f3-409">다른 브라우저에서로 이동 `https://<your.custom.domain>` 하 여 웹 앱이 제공 되는지 확인 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-409">In different browsers, go to `https://<your.custom.domain>` to ensure the web app is served.</span></span>

![웹 앱으로 이동](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> <span data-ttu-id="ce9f3-411">인증서 유효성 검사 오류가 발생 하는 경우에는 자체 서명 된 인증서가 원인일 수 있으며, 그렇지 않은 경우에는 PFX 파일로 내보낼 때 중간 인증서가 꺼진 것일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-411">If certificate validation errors occur, a self-signed certificate may be the cause, or intermediate certificates may have been left off when exporting to the PFX file.</span></span>

#### <a name="enforce-https"></a><span data-ttu-id="ce9f3-412">HTTPS 적용</span><span class="sxs-lookup"><span data-stu-id="ce9f3-412">Enforce HTTPS</span></span>

<span data-ttu-id="ce9f3-413">기본적으로 모든 사용자는 HTTP를 사용 하 여 웹 앱에 액세스할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-413">By default, anyone can access the web app using HTTP.</span></span> <span data-ttu-id="ce9f3-414">HTTPS 포트에 대 한 모든 HTTP 요청이 리디렉션될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-414">All HTTP requests to the HTTPS port may be redirected.</span></span>

<span data-ttu-id="ce9f3-415">웹 앱 페이지에서 **SL 설정**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-415">In the web app page, select **SL settings**.</span></span> <span data-ttu-id="ce9f3-416">그런 다음 **HTTPS에만 해당**에서 **켜기**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-416">Then, in **HTTPS Only**, select **On**.</span></span>

![HTTPS 적용](media/solution-deployment-guide-geo-distributed/image43.png)

<span data-ttu-id="ce9f3-418">작업이 완료 되 면 앱을 가리키는 HTTP Url로 이동 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-418">When the operation is complete, go to any of the HTTP URLs that point to the app.</span></span> <span data-ttu-id="ce9f3-419">예를 들면 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-419">For example:</span></span>

- <span data-ttu-id="ce9f3-420">https://<app_name>. azurewebsites.net</span><span class="sxs-lookup"><span data-stu-id="ce9f3-420">https://<app_name>.azurewebsites.net</span></span>
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a><span data-ttu-id="ce9f3-421">TLS 1.1/1.2 적용</span><span class="sxs-lookup"><span data-stu-id="ce9f3-421">Enforce TLS 1.1/1.2</span></span>

<span data-ttu-id="ce9f3-422">앱은 기본적으로 [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0을 허용 하며,이는 더 이상 업계 표준 (예: [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard))으로 안전 하 게 고려 되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-422">The app allows [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0 by default, which is no longer considered secure by industry standards (like [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)).</span></span> <span data-ttu-id="ce9f3-423">더 높은 TLS 버전을 적용하려면 다음이 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-423">To enforce higher TLS versions, follow these steps:</span></span>

1. <span data-ttu-id="ce9f3-424">웹 앱 페이지의 왼쪽 탐색 영역에서 **SSL 설정**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-424">In the web app page, in the left navigation, select **SSL settings**.</span></span>

2. <span data-ttu-id="ce9f3-425">**Tls 버전**에서 최소 tls 버전을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-425">In **TLS version**, select the minimum TLS version.</span></span>

    ![TLS 1.1 또는 1.2 적용](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a><span data-ttu-id="ce9f3-427">Traffic Manager 프로필 만들기</span><span class="sxs-lookup"><span data-stu-id="ce9f3-427">Create a Traffic Manager profile</span></span>

1. <span data-ttu-id="ce9f3-428">**리소스 만들기**  >  **네트워킹**  >  **Traffic Manager 프로필**  >  **만들기**를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-428">Select **Create a resource** > **Networking** > **Traffic Manager profile** > **Create**.</span></span>

2. <span data-ttu-id="ce9f3-429">**Traffic Manager 프로필 만들기**에 다음과 같이 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-429">In the **Create Traffic Manager profile**, complete as follows:</span></span>

    1. <span data-ttu-id="ce9f3-430">**이름**에 프로필의 이름을 입력 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-430">In **Name**, provide a name for the profile.</span></span> <span data-ttu-id="ce9f3-431">이 이름은 traffic manager.net 영역 내에서 고유 해야 하며, Traffic Manager 프로필에 액세스 하는 데 사용 되는 DNS 이름 trafficmanager.net이 생성 됩니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-431">This name needs to be unique within the traffic manager.net zone and results in the DNS name, trafficmanager.net, which is used to access the Traffic Manager profile.</span></span>

    2. <span data-ttu-id="ce9f3-432">**라우팅 방법**에서 **지리적 라우팅 방법을**선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-432">In **Routing method**, select the **Geographic routing method**.</span></span>

    3. <span data-ttu-id="ce9f3-433">**구독**에서이 프로필을 만들 구독을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-433">In **Subscription**, select the subscription under which to create this profile.</span></span>

    4. <span data-ttu-id="ce9f3-434">**리소스 그룹**에서 이 프로필을 배치할 새 리소스 그룹을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-434">In **Resource Group**, create a new resource group to place this profile under.</span></span>

    5. <span data-ttu-id="ce9f3-435">**리소스 그룹 위치**에서 리소스 그룹의 위치를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-435">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="ce9f3-436">이 설정은 리소스 그룹의 위치를 나타내며 전역적으로 배포 된 Traffic Manager 프로필에는 영향을 주지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-436">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile deployed globally.</span></span>

    6. <span data-ttu-id="ce9f3-437">**만들기**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-437">Select **Create**.</span></span>

    7. <span data-ttu-id="ce9f3-438">Traffic Manager 프로필의 전역 배포가 완료 되 면 해당 리소스 그룹에 리소스 중 하나로 나열 됩니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-438">When the global deployment of the Traffic Manager profile is complete, it's listed in the respective resource group as one of the resources.</span></span>

        ![Traffic Manager 프로필 만들기의 리소스 그룹](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="ce9f3-440">Traffic Manager 엔드포인트 추가</span><span class="sxs-lookup"><span data-stu-id="ce9f3-440">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="ce9f3-441">포털 검색 창에서 이전 섹션에서 만든 **Traffic Manager 프로필** 이름을 검색 하 고 표시 된 결과에서 Traffic Manager 프로필을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-441">In the portal search bar, search for the **Traffic Manager profile** name created in the preceding section and select the traffic manager profile in the displayed results.</span></span>

2. <span data-ttu-id="ce9f3-442">**Traffic Manager 프로필**의 **설정** 섹션에서 **끝점**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-442">In **Traffic Manager profile**, in the **Settings** section, select **Endpoints**.</span></span>

3. <span data-ttu-id="ce9f3-443">**추가**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-443">Select **Add**.</span></span>

4. <span data-ttu-id="ce9f3-444">Azure Stack 허브 끝점을 추가 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-444">Adding the Azure Stack Hub Endpoint.</span></span>

5. <span data-ttu-id="ce9f3-445">**형식**에서 **외부 끝점**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-445">For **Type**, select **External endpoint**.</span></span>

6. <span data-ttu-id="ce9f3-446">이 끝점의 **이름을** 제공 합니다. 이상적으로 Azure Stack 허브의 이름입니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-446">Provide a **Name** for this endpoint, ideally the name of the Azure Stack Hub.</span></span>

7. <span data-ttu-id="ce9f3-447">**FQDN**(정규화 된 도메인 이름)의 경우 Azure Stack 허브 웹 앱에 대 한 외부 URL을 사용 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-447">For fully qualified domain name (**FQDN**), use the external URL for the Azure Stack Hub Web App.</span></span>

8. <span data-ttu-id="ce9f3-448">지리적 매핑에서 리소스가 있는 지역/대륙을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-448">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="ce9f3-449">예: **유럽.**</span><span class="sxs-lookup"><span data-stu-id="ce9f3-449">For example, **Europe.**</span></span>

9. <span data-ttu-id="ce9f3-450">표시 되는 국가/지역 드롭다운에서이 끝점에 적용 되는 국가를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-450">Under the Country/Region drop-down that appears, select the country that applies to this endpoint.</span></span> <span data-ttu-id="ce9f3-451">예: **독일**.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-451">For example, **Germany**.</span></span>

10. <span data-ttu-id="ce9f3-452">**사용 안 함으로 추가**를 선택 취소 상태로 유지합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-452">Keep **Add as disabled** unchecked.</span></span>

11. <span data-ttu-id="ce9f3-453">**확인**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-453">Select **OK**.</span></span>

12. <span data-ttu-id="ce9f3-454">Azure 엔드포인트 추가:</span><span class="sxs-lookup"><span data-stu-id="ce9f3-454">Adding the Azure Endpoint:</span></span>

    1. <span data-ttu-id="ce9f3-455">**유형**에 대해 **Azure 엔드포인트**를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-455">For **Type**, select **Azure endpoint**.</span></span>

    2. <span data-ttu-id="ce9f3-456">끝점의 **이름을** 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-456">Provide a **Name** for the endpoint.</span></span>

    3. <span data-ttu-id="ce9f3-457">**대상 리소스 종류**에 대해 **App Service**를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-457">For **Target resource type**, select **App Service**.</span></span>

    4. <span data-ttu-id="ce9f3-458">**대상 리소스**에 대해 **앱 서비스 선택** 을 선택 하 여 동일한 구독에서 Web Apps 목록을 표시 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-458">For **Target resource**, select **Choose an app service** to show the listing of the Web Apps under the same subscription.</span></span> <span data-ttu-id="ce9f3-459">**리소스**에서 첫 번째 끝점으로 사용 되는 App service를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-459">In **Resource**, pick the App service used as the first endpoint.</span></span>

13. <span data-ttu-id="ce9f3-460">지리적 매핑에서 리소스가 있는 지역/대륙을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-460">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="ce9f3-461">예를 들어 **북아메리카/중앙 아메리카/카리브 합니다.**</span><span class="sxs-lookup"><span data-stu-id="ce9f3-461">For example, **North America/Central America/Caribbean.**</span></span>

14. <span data-ttu-id="ce9f3-462">표시 되는 국가/지역 드롭다운에서이 상자를 비워 두고 위의 모든 지역 그룹화를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-462">Under the Country/Region drop-down that appears, leave this spot blank to select all of the above regional grouping.</span></span>

15. <span data-ttu-id="ce9f3-463">**사용 안 함으로 추가**를 선택 취소 상태로 유지합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-463">Keep **Add as disabled** unchecked.</span></span>

16. <span data-ttu-id="ce9f3-464">**확인**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-464">Select **OK**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="ce9f3-465">리소스의 기본 끝점으로 사용할 지리적 범위가 모든 (세계) 인 끝점을 하나 이상 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-465">Create at least one endpoint with a geographic scope of All (World) to serve as the default endpoint for the resource.</span></span>

17. <span data-ttu-id="ce9f3-466">두 끝점의 추가가 완료 되 면 해당 모니터링 상태와 함께 **Traffic Manager 프로필** 에 **온라인**으로 표시 됩니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-466">When the addition of both endpoints is complete, they're displayed in **Traffic Manager profile** along with their monitoring status as **Online**.</span></span>

    ![Traffic Manager 프로필 끝점 상태](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a><span data-ttu-id="ce9f3-468">글로벌 기업은 Azure 지역 배포 기능에 의존 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-468">Global Enterprise relies on Azure geo-distribution capabilities</span></span>

<span data-ttu-id="ce9f3-469">Azure Traffic Manager 및 지리 관련 끝점을 통해 데이터 트래픽을 전송 하면 글로벌 기업은 지역 규정을 준수 하 고 데이터를 준수 하 고 안전 하 게 유지할 수 있습니다 .이는 로컬 및 원격 비즈니스 위치의 성공에 매우 중요 합니다.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-469">Directing data traffic via Azure Traffic Manager and geography-specific endpoints enables global enterprises to adhere to regional regulations and keep data compliant and secure, which is crucial to the success of local and remote business locations.</span></span>

## <a name="next-steps"></a><span data-ttu-id="ce9f3-470">다음 단계</span><span class="sxs-lookup"><span data-stu-id="ce9f3-470">Next steps</span></span>

- <span data-ttu-id="ce9f3-471">Azure Cloud 패턴에 대해 자세히 알아보려면 [클라우드 디자인 패턴](https://docs.microsoft.com/azure/architecture/patterns)을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="ce9f3-471">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](https://docs.microsoft.com/azure/architecture/patterns).</span></span>
