---
title: Azure 및 Azure Stack Hub에서 클라우드 간 크기를 조정 하는 앱 배포
description: Azure 및 Azure Stack Hub에서 클라우드 간 크기를 조정 하는 앱을 배포 하는 방법에 대해 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 740a8c0ec904fe8eb3f9744626bc9dd6655bdb52
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911526"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a><span data-ttu-id="b5d60-103">Azure 및 Azure Stack Hub를 사용 하 여 클라우드 간 크기를 조정 하는 앱 배포</span><span class="sxs-lookup"><span data-stu-id="b5d60-103">Deploy an app that scales cross-cloud using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="b5d60-104">Traffic manager를 통해 자동 크기 조정을 사용 하 여 Azure Stack Hub 호스팅 웹 앱에서 Azure 호스트 된 웹 앱으로 전환 하기 위한 수동으로 트리거된 프로세스를 제공 하는 클라우드 간 솔루션을 만드는 방법을 알아봅니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-104">Learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app with autoscaling via traffic manager.</span></span> <span data-ttu-id="b5d60-105">이 프로세스를 통해 부하 상태에서 유연 하 고 확장 가능한 클라우드 유틸리티를 사용할 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-105">This process ensures flexible and scalable cloud utility when under load.</span></span>

<span data-ttu-id="b5d60-106">이 패턴을 사용 하면 테 넌 트가 공용 클라우드에서 앱을 실행할 준비가 되지 않았을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-106">With this pattern, your tenant may not be ready to run your app in the public cloud.</span></span> <span data-ttu-id="b5d60-107">그러나 앱에 대 한 급증 하는 요청을 처리 하기 위해 온-프레미스 환경에 필요한 용량을 유지 하는 것이 경제적으로 수 있는 것은 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-107">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="b5d60-108">테 넌 트는 온-프레미스 솔루션으로 공용 클라우드의 탄력성을 활용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-108">Your tenant can make use of the elasticity of the public cloud with their on-premises solution.</span></span>

<span data-ttu-id="b5d60-109">이 솔루션에서는 다음을 수행 하는 샘플 환경을 빌드합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="b5d60-110">다중 노드 웹 앱을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-110">Create a multi-node web app.</span></span>
> - <span data-ttu-id="b5d60-111">CD (연속 배포) 프로세스를 구성 하 고 관리 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-111">Configure and manage the Continuous Deployment (CD) process.</span></span>
> - <span data-ttu-id="b5d60-112">Azure Stack 허브에 웹 앱을 게시 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-112">Publish the web app to Azure Stack Hub.</span></span>
> - <span data-ttu-id="b5d60-113">릴리스를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-113">Create a release.</span></span>
> - <span data-ttu-id="b5d60-114">배포를 모니터링 하 고 추적 하는 방법을 알아봅니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-114">Learn to monitor and track your deployments.</span></span>

> [!Tip]  
> <span data-ttu-id="b5d60-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="b5d60-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="b5d60-116">Microsoft Azure Stack 허브는 Azure의 확장입니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-116">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="b5d60-117">Azure Stack 허브는 온-프레미스 환경에 클라우드 컴퓨팅의 민첩성과 혁신을 제공 하 여 어디서 나 하이브리드 앱을 빌드하고 배포할 수 있는 유일한 하이브리드 클라우드를 사용 하도록 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-117">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="b5d60-118">[하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 문서에서는 하이브리드 앱을 디자인, 배포 및 운영 하기 위한 소프트웨어 품질 (배치, 확장성, 가용성, 복원 력, 관리 효율성 및 보안)의 핵심 요소을 검토 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-118">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="b5d60-119">디자인 고려 사항은 하이브리드 앱 디자인을 최적화 하 고 프로덕션 환경에서 문제를 최소화 하는 데 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-119">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="b5d60-120">필수 구성 요소</span><span class="sxs-lookup"><span data-stu-id="b5d60-120">Prerequisites</span></span>

- <span data-ttu-id="b5d60-121">동작합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-121">Azure subscription.</span></span> <span data-ttu-id="b5d60-122">필요한 경우 시작 하기 전에 [무료 계정](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-122">If needed, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before beginning.</span></span>
- <span data-ttu-id="b5d60-123">Azure Stack 허브 통합 시스템 또는 ASDK (Azure Stack Development Kit 배포).</span><span class="sxs-lookup"><span data-stu-id="b5d60-123">An Azure Stack Hub integrated system or deployment of Azure Stack Development Kit (ASDK).</span></span>
  - <span data-ttu-id="b5d60-124">Azure Stack 허브를 설치 하는 방법에 대 한 지침은 [ASDK 설치](/azure-stack/asdk/asdk-install.md)를 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="b5d60-124">For instructions on installing Azure Stack Hub, see [Install the ASDK](/azure-stack/asdk/asdk-install.md).</span></span>
  - <span data-ttu-id="b5d60-125">ASDK 배포 후 자동화 스크립트의 경우 다음으로 이동 합니다.[https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span><span class="sxs-lookup"><span data-stu-id="b5d60-125">For an ASDK post-deployment automation script, go to: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span></span>
  - <span data-ttu-id="b5d60-126">이 설치를 완료 하는 데 몇 시간 정도 걸릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-126">This installation may require a few hours to complete.</span></span>
- <span data-ttu-id="b5d60-127">Azure Stack 허브에 [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS 서비스를 배포 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-127">Deploy [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS services to Azure Stack Hub.</span></span>
- <span data-ttu-id="b5d60-128">Azure Stack 허브 환경 내에서 [계획/제품을 만듭니다](/azure-stack/operator/service-plan-offer-subscription-overview.md) .</span><span class="sxs-lookup"><span data-stu-id="b5d60-128">[Create plans/offers](/azure-stack/operator/service-plan-offer-subscription-overview.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="b5d60-129">Azure Stack 허브 환경 내에서 [테 넌 트 구독을 만듭니다](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) .</span><span class="sxs-lookup"><span data-stu-id="b5d60-129">[Create tenant subscription](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="b5d60-130">테 넌 트 구독 내에서 웹 앱을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-130">Create a web app within the tenant subscription.</span></span> <span data-ttu-id="b5d60-131">나중에 사용 하기 위해 새 웹 앱 URL을 기록해 둡니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-131">Make note of the new web app URL for later use.</span></span>
- <span data-ttu-id="b5d60-132">테 넌 트 구독 내에서 VM (가상 머신) Azure Pipelines 배포</span><span class="sxs-lookup"><span data-stu-id="b5d60-132">Deploy Azure Pipelines virtual machine (VM) within the tenant subscription.</span></span>
- <span data-ttu-id="b5d60-133">.NET 3.5이 있는 Windows Server 2016 VM이 필요 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-133">Windows Server 2016 VM with .NET 3.5 is required.</span></span> <span data-ttu-id="b5d60-134">이 VM은 Azure Stack 허브의 테 넌 트 구독에 전용 빌드 에이전트로 빌드됩니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-134">This VM will be built in the tenant subscription on Azure Stack Hub as the private build agent.</span></span>
- <span data-ttu-id="b5d60-135">[SQL 2017 VM 이미지를 포함 하는 Windows Server 2016](/azure-stack/operator/azure-stack-add-vm-image.md) 는 Azure Stack Hub Marketplace에서 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-135">[Windows Server 2016 with SQL 2017 VM image](/azure-stack/operator/azure-stack-add-vm-image.md) is available in the Azure Stack Hub Marketplace.</span></span> <span data-ttu-id="b5d60-136">이 이미지를 사용할 수 없는 경우 Azure Stack 허브 운영자에 게 작업 하 여 환경에 추가 되었는지 확인 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-136">If this image isn't available, work with an Azure Stack Hub Operator to ensure it's added to the environment.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="b5d60-137">문제 및 고려 사항</span><span class="sxs-lookup"><span data-stu-id="b5d60-137">Issues and considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="b5d60-138">확장성</span><span class="sxs-lookup"><span data-stu-id="b5d60-138">Scalability</span></span>

<span data-ttu-id="b5d60-139">클라우드 간 크기 조정의 핵심 구성 요소는 공용 및 온-프레미스 클라우드 인프라 간에 즉각적인 주문형 및 주문형 크기 조정을 제공 하 여 일관적이 고 안정적인 서비스를 제공 하는 기능입니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-139">The key component of cross-cloud scaling is the ability to deliver immediate and on-demand scaling between public and on-premises cloud infrastructure, providing consistent and reliable service.</span></span>

### <a name="availability"></a><span data-ttu-id="b5d60-140">가용성</span><span class="sxs-lookup"><span data-stu-id="b5d60-140">Availability</span></span>

<span data-ttu-id="b5d60-141">로컬로 배포 된 앱이 온-프레미스 하드웨어 구성 및 소프트웨어 배포를 통해 고가용성으로 구성 되어 있는지 확인 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-141">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="b5d60-142">관리 효율</span><span class="sxs-lookup"><span data-stu-id="b5d60-142">Manageability</span></span>

<span data-ttu-id="b5d60-143">클라우드 간 솔루션은 환경 간에 원활한 관리 및 친숙 한 인터페이스를 보장 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-143">The cross-cloud solution ensures seamless management and familiar interface between environments.</span></span> <span data-ttu-id="b5d60-144">PowerShell은 플랫폼 간 관리에 권장 됩니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-144">PowerShell is recommended for cross-platform management.</span></span>

## <a name="cross-cloud-scaling"></a><span data-ttu-id="b5d60-145">클라우드 간 크기 조정</span><span class="sxs-lookup"><span data-stu-id="b5d60-145">Cross-cloud scaling</span></span>

### <a name="get-a-custom-domain-and-configure-dns"></a><span data-ttu-id="b5d60-146">사용자 지정 도메인 가져오기 및 DNS 구성</span><span class="sxs-lookup"><span data-stu-id="b5d60-146">Get a custom domain and configure DNS</span></span>

<span data-ttu-id="b5d60-147">도메인에 대 한 DNS 영역 파일을 업데이트 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-147">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="b5d60-148">Azure AD에서 사용자 지정 도메인 이름의 소유권을 확인 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-148">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="b5d60-149">Azure 내에서 Azure/Office 365/외부 DNS 레코드에 대 한 [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) 를 사용 하거나 [다른 DNS 등록자](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/)에 dns 항목을 추가 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-149">Use [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) for Azure/Office 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).</span></span>

1. <span data-ttu-id="b5d60-150">공용 등록자에 게 사용자 지정 도메인을 등록 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-150">Register a custom domain with a public registrar.</span></span>
2. <span data-ttu-id="b5d60-151">도메인에 대한 도메인 이름 등록 기관에 로그인합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-151">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="b5d60-152">승인 된 관리자는 DNS를 업데이트 해야 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-152">An approved admin may be required to make DNS updates.</span></span>
3. <span data-ttu-id="b5d60-153">Azure AD에서 제공 하는 DNS 항목을 추가 하 여 도메인에 대 한 DNS 영역 파일을 업데이트 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-153">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="b5d60-154">DNS 항목은 메일 라우팅 또는 웹 호스팅 동작에 영향을 주지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-154">(The DNS entry won't affect email routing or web hosting behaviors.)</span></span>

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a><span data-ttu-id="b5d60-155">Azure Stack 허브에서 기본 다중 노드 웹 앱 만들기</span><span class="sxs-lookup"><span data-stu-id="b5d60-155">Create a default multi-node web app in Azure Stack Hub</span></span>

<span data-ttu-id="b5d60-156">웹 앱을 Azure Stack Azure에 배포 하는 CI/CD (연속 통합 및 지속적인 배포)를 설정 하 고 두 클라우드 모두에 변경 내용을 autopush.</span><span class="sxs-lookup"><span data-stu-id="b5d60-156">Set up hybrid continuous integration and continuous deployment (CI/CD) to deploy web apps to Azure and Azure Stack Hub and to autopush changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="b5d60-157">적절 한 이미지 게시를 실행 하는 Azure Stack 허브 (Windows Server 및 SQL) 및 App Service 배포가 필요 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-157">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="b5d60-158">자세한 내용은 [Azure Stack 허브에 App Service 배포에 대 한 App Service 설명서 필수 조건](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md)을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="b5d60-158">For more information, review the App Service documentation [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

### <a name="add-code-to-azure-repos"></a><span data-ttu-id="b5d60-159">Azure Repos에 코드 추가</span><span class="sxs-lookup"><span data-stu-id="b5d60-159">Add Code to Azure Repos</span></span>

<span data-ttu-id="b5d60-160">Azure Repos</span><span class="sxs-lookup"><span data-stu-id="b5d60-160">Azure Repos</span></span>

1. <span data-ttu-id="b5d60-161">Azure Repos에 대 한 프로젝트 만들기 권한이 있는 계정으로 Azure Repos에 로그인 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-161">Sign in to Azure Repos with an account that has project creation rights on Azure Repos.</span></span>

    <span data-ttu-id="b5d60-162">하이브리드 CI/CD는 앱 코드와 인프라 코드 모두에 적용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-162">Hybrid CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="b5d60-163">사설 및 호스 티 드 클라우드 개발 모두에 [Azure Resource Manager 템플릿을](https://azure.microsoft.com/resources/templates/) 사용 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-163">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Azure Repos에서 프로젝트에 연결](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. <span data-ttu-id="b5d60-165">기본 웹 앱을 만들고 열어 **리포지토리를 복제** 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-165">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Azure 웹 앱에서 리포지토리 복제](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="b5d60-167">두 클라우드에서 App Services에 대 한 자체 포함 웹 앱 배포 만들기</span><span class="sxs-lookup"><span data-stu-id="b5d60-167">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="b5d60-168">**WebApplication .csproj** 파일을 편집 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-168">Edit the **WebApplication.csproj** file.</span></span> <span data-ttu-id="b5d60-169">`Runtimeidentifier`을 선택 하 고 추가 `win10-x64` 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-169">Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="b5d60-170">[자체 포함 된 배포](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) 설명서를 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="b5d60-170">(See [Self-contained deployment](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![웹 앱 프로젝트 파일 편집](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. <span data-ttu-id="b5d60-172">팀 탐색기를 사용 하 여 Azure Repos 코드를 체크 인 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-172">Check in the code to Azure Repos using Team Explorer.</span></span>

3. <span data-ttu-id="b5d60-173">앱 코드가 Azure Repos 체크 인 되었는지 확인 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-173">Confirm that the app code has been checked into Azure Repos.</span></span>

## <a name="create-the-build-definition"></a><span data-ttu-id="b5d60-174">빌드 정의 만들기</span><span class="sxs-lookup"><span data-stu-id="b5d60-174">Create the build definition</span></span>

1. <span data-ttu-id="b5d60-175">Azure Pipelines에 로그인 하 여 빌드 정의를 만들 수 있는지 확인 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-175">Sign in to Azure Pipelines to confirm the ability to create build definitions.</span></span>

2. <span data-ttu-id="b5d60-176">**-R win10-x64** 코드를 추가 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-176">Add **-r win10-x64** code.</span></span> <span data-ttu-id="b5d60-177">이러한 추가는 .NET Core를 사용 하 여 자체 포함 된 배포를 트리거하기 위해 필요 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-177">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![웹 앱에 코드 추가](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. <span data-ttu-id="b5d60-179">빌드를 실행 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-179">Run the build.</span></span> <span data-ttu-id="b5d60-180">[자체 포함 배포 빌드](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) 프로세스는 Azure 및 Azure Stack Hub에서 실행 되는 아티팩트를 게시 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-180">The [self-contained deployment build](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that run on Azure and Azure Stack Hub.</span></span>

## <a name="use-an-azure-hosted-agent"></a><span data-ttu-id="b5d60-181">Azure 호스트 된 에이전트 사용</span><span class="sxs-lookup"><span data-stu-id="b5d60-181">Use an Azure hosted agent</span></span>

<span data-ttu-id="b5d60-182">Azure Pipelines에서 호스팅된 빌드 에이전트를 사용 하는 것은 웹 앱을 빌드 및 배포 하는 편리한 옵션입니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-182">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="b5d60-183">유지 관리 및 업그레이드는 Microsoft Azure에 의해 자동으로 수행 되어 지속적이 고 중단 되지 않는 개발 주기를 가능 하 게 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-183">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="b5d60-184">CD 프로세스 관리 및 구성</span><span class="sxs-lookup"><span data-stu-id="b5d60-184">Manage and configure the CD process</span></span>

<span data-ttu-id="b5d60-185">Azure Pipelines 및 Azure DevOps Services는 개발, 스테이징, QA 및 프로덕션 환경과 같은 여러 환경에 릴리스를 위한 매우 구성 가능 하 고 관리 하기 쉬운 파이프라인을 제공 합니다. 특정 단계에서 승인 요구를 포함 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-185">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="b5d60-186">릴리스 정의 만들기</span><span class="sxs-lookup"><span data-stu-id="b5d60-186">Create release definition</span></span>

1. <span data-ttu-id="b5d60-187">**더하기** 단추를 선택 **하 여 Azure DevOps Services** 의 **빌드 및 릴리스** 섹션에서 릴리스 탭 아래에 새 릴리스를 추가 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-187">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![릴리스 정의 만들기](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. <span data-ttu-id="b5d60-189">Azure App Service 배포 템플릿을 적용 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-189">Apply the Azure App Service Deployment template.</span></span>

   ![Azure App Service 배포 템플릿 적용](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. <span data-ttu-id="b5d60-191">**아티팩트 추가**에서 Azure 클라우드 빌드 앱에 대 한 아티팩트를 추가 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-191">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Azure 클라우드 빌드에 아티팩트 추가](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. <span data-ttu-id="b5d60-193">파이프라인 탭에서 환경의 **단계, 작업** 링크를 선택 하 고 Azure 클라우드 환경 값을 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-193">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Azure 클라우드 환경 값 설정](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. <span data-ttu-id="b5d60-195">**환경 이름을** 설정 하 고 azure 클라우드 끝점에 대 한 **azure 구독** 을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-195">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Azure Cloud 끝점에 대 한 Azure 구독 선택](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. <span data-ttu-id="b5d60-197">**App service name**에서 필요한 Azure App service 이름을 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-197">Under **App service name**, set the required Azure app service name.</span></span>

      ![Azure app service 이름 설정](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. <span data-ttu-id="b5d60-199">Azure 클라우드 호스팅 환경에 대 한 **에이전트 큐** 아래에 "Hosted VS2017"를 입력 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-199">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Azure 클라우드 호스팅 환경에 대 한 에이전트 큐 설정](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. <span data-ttu-id="b5d60-201">배포 Azure App Service 메뉴에서 환경의 올바른 **패키지 또는 폴더** 를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-201">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="b5d60-202">**폴더 위치**에 **확인** 을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-202">Select **OK** to **folder location**.</span></span>
  
      ![Azure App Service 환경을 위한 패키지 또는 폴더 선택](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![Azure App Service 환경을 위한 패키지 또는 폴더 선택](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. <span data-ttu-id="b5d60-205">모든 변경 내용을 저장 하 고 **릴리스 파이프라인**으로 돌아갑니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-205">Save all changes and go back to **release pipeline**.</span></span>

    ![릴리스 파이프라인의 변경 내용 저장](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. <span data-ttu-id="b5d60-207">Azure Stack 허브 앱에 대 한 빌드를 선택 하는 새 아티팩트를 추가 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-207">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Azure Stack Hub 앱에 대 한 새 아티팩트 추가](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. <span data-ttu-id="b5d60-209">Azure App Service 배포를 적용 하 여 환경을 하나 더 추가 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-209">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Azure App Service 배포에 환경 추가](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. <span data-ttu-id="b5d60-211">새 환경의 이름을 "Azure Stack"로 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-211">Name the new environment "Azure Stack".</span></span>

    ![Azure App Service 배포의 이름 환경](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. <span data-ttu-id="b5d60-213">**작업** 탭에서 Azure Stack 환경을 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-213">Find the Azure Stack environment under **Task** tab.</span></span>

    ![Azure Stack 환경](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. <span data-ttu-id="b5d60-215">Azure Stack 끝점에 대 한 구독을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-215">Select the subscription for the Azure Stack endpoint.</span></span>

    ![Azure Stack 끝점에 대 한 구독을 선택 합니다.](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. <span data-ttu-id="b5d60-217">Azure Stack 웹 앱 이름을 App service 이름으로 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-217">Set the Azure Stack web app name as the App service name.</span></span>
    <span data-ttu-id="b5d60-218">![Azure Stack 웹 앱 이름 설정](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span><span class="sxs-lookup"><span data-stu-id="b5d60-218">![Set Azure Stack web app name](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span></span>

16. <span data-ttu-id="b5d60-219">Azure Stack 에이전트를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-219">Select the Azure Stack agent.</span></span>

    ![Azure Stack 에이전트를 선택 합니다.](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. <span data-ttu-id="b5d60-221">배포 Azure App Service 섹션에서 환경의 올바른 **패키지 또는 폴더** 를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-221">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="b5d60-222">폴더 위치에 **확인** 을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-222">Select **OK** to folder location.</span></span>

    ![Azure App Service 배포를 위한 폴더 선택](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![Azure App Service 배포를 위한 폴더 선택](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. <span data-ttu-id="b5d60-225">변수 탭에서 라는 변수를 추가 하 `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` 고 해당 값을 **true**로 설정 하 고 범위를 Azure Stack 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-225">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack.</span></span>

    ![Azure 앱 배포에 변수 추가](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. <span data-ttu-id="b5d60-227">두 아티팩트의 **연속** 배포 트리거 아이콘을 선택 하 고 **계속 해 서** 배포 트리거를 사용 하도록 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-227">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![연속 배포 트리거 선택](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. <span data-ttu-id="b5d60-229">Azure Stack 환경에서 **배포 전** 조건 아이콘을 선택 하 고 트리거를 **릴리스 후로 설정 합니다.**</span><span class="sxs-lookup"><span data-stu-id="b5d60-229">Select the **Pre-deployment** conditions icon in the Azure Stack environment and set the trigger to **After release.**</span></span>

    ![배포 전 조건 선택](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. <span data-ttu-id="b5d60-231">모든 변경 내용을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-231">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="b5d60-232">템플릿에서 릴리스 정의를 만들 때 작업에 대 한 일부 설정이 [환경 변수로](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) 자동으로 정의 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-232">Some settings for the tasks may have been automatically defined as [environment variables](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="b5d60-233">이러한 설정은 작업 설정에서 수정할 수 없습니다. 대신 이러한 설정을 편집 하려면 부모 환경 항목을 선택 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-233">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a><span data-ttu-id="b5d60-234">Visual Studio를 통해 Azure Stack 허브에 게시</span><span class="sxs-lookup"><span data-stu-id="b5d60-234">Publish to Azure Stack Hub via Visual Studio</span></span>

<span data-ttu-id="b5d60-235">Azure DevOps Services 빌드는 끝점을 만들어 Azure Stack 허브에 Azure 서비스 앱을 배포할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-235">By creating endpoints, an Azure DevOps Services build can deploy Azure Service apps to Azure Stack Hub.</span></span> <span data-ttu-id="b5d60-236">Azure Pipelines 빌드 에이전트에 연결 되 고 Azure Stack 허브에 연결 됩니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-236">Azure Pipelines connects to the build agent, which connects to Azure Stack Hub.</span></span>

1. <span data-ttu-id="b5d60-237">Azure DevOps Services에 로그인 하 고 앱 설정 페이지로 이동 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-237">Sign in to Azure DevOps Services and go to the app settings page.</span></span>

2. <span data-ttu-id="b5d60-238">**설정**에서 **보안**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-238">On **Settings**, select **Security**.</span></span>

3. <span data-ttu-id="b5d60-239">**VSTS 그룹**에서 **끝점 작성자**를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-239">In **VSTS Groups**, select **Endpoint Creators**.</span></span>

4. <span data-ttu-id="b5d60-240">**구성원** 탭에서 **추가**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-240">On the **Members** tab, select **Add**.</span></span>

5. <span data-ttu-id="b5d60-241">사용자 **및 그룹 추가**에서 사용자 이름을 입력 하 고 사용자 목록에서 해당 사용자를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-241">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

6. <span data-ttu-id="b5d60-242">**변경 내용 저장**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-242">Select **Save changes**.</span></span>

7. <span data-ttu-id="b5d60-243">**VSTS 그룹** 목록에서 **끝점 관리자**를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-243">In the **VSTS Groups** list, select **Endpoint Administrators**.</span></span>

8. <span data-ttu-id="b5d60-244">**구성원** 탭에서 **추가**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-244">On the **Members** tab, select **Add**.</span></span>

9. <span data-ttu-id="b5d60-245">사용자 **및 그룹 추가**에서 사용자 이름을 입력 하 고 사용자 목록에서 해당 사용자를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-245">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

10. <span data-ttu-id="b5d60-246">**변경 내용 저장**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-246">Select **Save changes**.</span></span>

<span data-ttu-id="b5d60-247">이제 끝점 정보가 있으므로 Azure Stack Hub 연결에 대 한 Azure Pipelines를 사용할 준비가 되었습니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-247">Now that the endpoint information exists, the Azure Pipelines to Azure Stack Hub connection is ready to use.</span></span> <span data-ttu-id="b5d60-248">Azure Stack Hub의 빌드 에이전트는 Azure Pipelines에서 명령을 가져온 다음 에이전트 Azure Stack 허브와의 통신을 위해 끝점 정보를 전달 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-248">The build agent in Azure Stack Hub gets instructions from Azure Pipelines and then the agent conveys endpoint information for communication with Azure Stack Hub.</span></span>

## <a name="develop-the-app-build"></a><span data-ttu-id="b5d60-249">앱 빌드 개발</span><span class="sxs-lookup"><span data-stu-id="b5d60-249">Develop the app build</span></span>

> [!Note]  
> <span data-ttu-id="b5d60-250">적절 한 이미지 게시를 실행 하는 Azure Stack 허브 (Windows Server 및 SQL) 및 App Service 배포가 필요 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-250">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="b5d60-251">자세한 내용은 [Azure Stack 허브에 App Service를 배포 하기 위한 필수 조건](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md)을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="b5d60-251">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

<span data-ttu-id="b5d60-252">Azure Repos의 웹 앱 코드와 같은 [Azure Resource Manager 템플릿을](https://azure.microsoft.com/resources/templates/) 사용 하 여 두 클라우드 모두에 배포 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-252">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) like web app code from Azure Repos to deploy to both clouds.</span></span>

### <a name="add-code-to-an-azure-repos-project"></a><span data-ttu-id="b5d60-253">Azure Repos 프로젝트에 코드 추가</span><span class="sxs-lookup"><span data-stu-id="b5d60-253">Add code to an Azure Repos project</span></span>

1. <span data-ttu-id="b5d60-254">Azure Stack Hub에 대 한 프로젝트 만들기 권한이 있는 계정으로 Azure Repos에 로그인 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-254">Sign in to Azure Repos with an account that has project creation rights on Azure Stack Hub.</span></span>

2. <span data-ttu-id="b5d60-255">기본 웹 앱을 만들고 열어 **리포지토리를 복제** 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-255">**Clone the repository** by creating and opening the default web app.</span></span>

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="b5d60-256">두 클라우드에서 App Services에 대 한 자체 포함 웹 앱 배포 만들기</span><span class="sxs-lookup"><span data-stu-id="b5d60-256">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="b5d60-257">**WebApplication .csproj** 파일을 편집 합니다 .를 선택한 `Runtimeidentifier` 다음 추가를 클릭 `win10-x64` 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-257">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and then add `win10-x64`.</span></span> <span data-ttu-id="b5d60-258">자세한 내용은 [자체 포함 된 배포](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) 설명서를 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="b5d60-258">For more information, see [Self-contained deployment](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.</span></span>

2. <span data-ttu-id="b5d60-259">팀 탐색기를 사용 하 여 Azure Repos 코드를 확인 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-259">Use Team Explorer to check the code into Azure Repos.</span></span>

3. <span data-ttu-id="b5d60-260">앱 코드가 Azure Repos 체크 인 되었는지 확인 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-260">Confirm that the app code was checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="b5d60-261">빌드 정의 만들기</span><span class="sxs-lookup"><span data-stu-id="b5d60-261">Create the build definition</span></span>

1. <span data-ttu-id="b5d60-262">빌드 정의를 만들 수 있는 계정으로 Azure Pipelines에 로그인 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-262">Sign in to Azure Pipelines with an account that can create a build definition.</span></span>

2. <span data-ttu-id="b5d60-263">프로젝트에 대 한 **웹 응용 프로그램 빌드** 페이지로 이동 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-263">Go to the **Build Web Application** page for the project.</span></span>

3. <span data-ttu-id="b5d60-264">**인수**에서 **-r win10-x64** 코드를 추가 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-264">In **Arguments**, add **-r win10-x64** code.</span></span> <span data-ttu-id="b5d60-265">이러한 추가는 .NET Core를 사용 하 여 자체 포함 된 배포를 트리거하는 데 필요 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-265">This addition is required to trigger a self-contained deployment with .NET Core.</span></span>

4. <span data-ttu-id="b5d60-266">빌드를 실행 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-266">Run the build.</span></span> <span data-ttu-id="b5d60-267">[자체 포함 배포 빌드](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) 프로세스는 Azure 및 Azure Stack 허브에서 실행할 수 있는 아티팩트를 게시 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-267">The [self-contained deployment build](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="use-an-azure-hosted-build-agent"></a><span data-ttu-id="b5d60-268">Azure 호스트 된 빌드 에이전트 사용</span><span class="sxs-lookup"><span data-stu-id="b5d60-268">Use an Azure hosted build agent</span></span>

<span data-ttu-id="b5d60-269">Azure Pipelines에서 호스팅된 빌드 에이전트를 사용 하는 것은 웹 앱을 빌드 및 배포 하는 편리한 옵션입니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-269">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="b5d60-270">유지 관리 및 업그레이드는 Microsoft Azure에 의해 자동으로 수행 되어 지속적이 고 중단 되지 않는 개발 주기를 가능 하 게 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-270">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="configure-the-continuous-deployment-cd-process"></a><span data-ttu-id="b5d60-271">CD (연속 배포) 프로세스 구성</span><span class="sxs-lookup"><span data-stu-id="b5d60-271">Configure the continuous deployment (CD) process</span></span>

<span data-ttu-id="b5d60-272">Azure Pipelines 및 Azure DevOps Services는 개발, 스테이징, QA (품질 보증) 및 프로덕션과 같은 여러 환경에 릴리스를 위한 매우 구성 가능 하 고 관리 하기 쉬운 파이프라인을 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-272">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, quality assurance (QA), and production.</span></span> <span data-ttu-id="b5d60-273">이 프로세스에는 앱 수명 주기의 특정 단계에서 승인이 필요한 것으로 포함 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-273">This process can include requiring approvals at specific stages of the app life cycle.</span></span>

#### <a name="create-release-definition"></a><span data-ttu-id="b5d60-274">릴리스 정의 만들기</span><span class="sxs-lookup"><span data-stu-id="b5d60-274">Create release definition</span></span>

<span data-ttu-id="b5d60-275">릴리스 정의 만들기는 앱 빌드 프로세스의 마지막 단계입니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-275">Creating a release definition is the final step in the app build process.</span></span> <span data-ttu-id="b5d60-276">이 릴리스 정의는 릴리스를 만들고 빌드를 배포 하는 데 사용 됩니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-276">This release definition is used to create a release and deploy a build.</span></span>

1. <span data-ttu-id="b5d60-277">Azure Pipelines에 로그인 하 고 프로젝트에 대 한 **빌드 및 릴리스** 로 이동 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-277">Sign in to Azure Pipelines and go to **Build and Release** for the project.</span></span>

2. <span data-ttu-id="b5d60-278">**릴리스** 탭에서 **[+]** 를 선택한 다음 **릴리스 정의 만들기**를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-278">On the **Releases** tab, select **[ + ]** and then pick **Create release definition**.</span></span>

3. <span data-ttu-id="b5d60-279">**템플릿 선택**에서 **Azure App Service 배포**를 선택 하 고 **적용**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-279">On **Select a Template**, choose **Azure App Service Deployment**, and then select **Apply**.</span></span>

4. <span data-ttu-id="b5d60-280">**아티팩트 추가**의 **원본 (빌드 정의)** 에서 Azure 클라우드 빌드 앱을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-280">On **Add artifact**, from the **Source (Build definition)**, select the Azure Cloud build app.</span></span>

5. <span data-ttu-id="b5d60-281">**파이프라인** 탭에서 **환경 작업 보기**에 대 한 **1 단계**, **1 작업** 링크를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-281">On the **Pipeline** tab, select the **1 Phase**, **1 Task** link to **View environment tasks**.</span></span>

6. <span data-ttu-id="b5d60-282">**작업** 탭에서 **환경 이름** 으로 azure를 입력 하 고 **Azure 구독** 목록에서 azurecloud Traders-웹 EP를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-282">On the **Tasks** tab, enter Azure as the **Environment name** and select the AzureCloud Traders-Web EP from the **Azure subscription** list.</span></span>

7. <span data-ttu-id="b5d60-283">다음 화면 캡처에 있는 **Azure app service 이름을**입력 합니다 `northwindtraders` .</span><span class="sxs-lookup"><span data-stu-id="b5d60-283">Enter the **Azure app service name**, which is `northwindtraders` in the next screen capture.</span></span>

8. <span data-ttu-id="b5d60-284">에이전트 단계의 경우 **에이전트 큐** 목록에서 **Hosted VS2017** 를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-284">For the Agent phase, select **Hosted VS2017** from the **Agent queue** list.</span></span>

9. <span data-ttu-id="b5d60-285">**Azure App Service 배포**에서 환경에 대 한 올바른 **패키지 또는 폴더** 를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-285">In **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span>

10. <span data-ttu-id="b5d60-286">**파일 또는 폴더 선택**에서 확인을 클릭 하 여 **위치**를 **확인** 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-286">In **Select File or Folder**, select **OK** to **Location**.</span></span>

11. <span data-ttu-id="b5d60-287">모든 변경 내용을 저장 하 고 **파이프라인**으로 돌아갑니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-287">Save all changes and go back to **Pipeline**.</span></span>

12. <span data-ttu-id="b5d60-288">**파이프라인** 탭에서 **아티팩트 추가**를 선택 하 고 **원본 (빌드 정의)** 목록에서 **NorthwindCloud 상인-용기** 를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-288">On the **Pipeline** tab, select **Add artifact**, and choose the **NorthwindCloud Traders-Vessel** from the **Source (Build Definition)** list.</span></span>

13. <span data-ttu-id="b5d60-289">**템플릿 선택**에서 다른 환경을 추가 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-289">On **Select a Template**, add another environment.</span></span> <span data-ttu-id="b5d60-290">**Azure App Service 배포** 를 선택 하 고 **적용**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-290">Pick **Azure App Service Deployment** and then select **Apply**.</span></span>

14. <span data-ttu-id="b5d60-291">`Azure Stack Hub` **환경 이름**으로를 입력 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-291">Enter `Azure Stack Hub` as the **Environment name**.</span></span>

15. <span data-ttu-id="b5d60-292">**작업** 탭에서 Azure Stack 허브를 찾아 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-292">On the **Tasks** tab, find and select Azure Stack Hub.</span></span>

16. <span data-ttu-id="b5d60-293">**Azure 구독** 목록에서 Azure Stack 허브 끝점에 대해 **azurestack TRADERS-용기 EP** 를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-293">From the **Azure subscription** list, select **AzureStack Traders-Vessel EP** for the Azure Stack Hub endpoint.</span></span>

17. <span data-ttu-id="b5d60-294">Azure Stack 허브 웹 앱 이름을 **app service 이름**으로 입력 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-294">Enter the Azure Stack Hub web app name as the **App service name**.</span></span>

18. <span data-ttu-id="b5d60-295">**에이전트 선택**의 **에이전트 큐** 목록에서 **Azurestack-b Douglas 전나무** 를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-295">Under **Agent selection**, pick **AzureStack -b Douglas Fir** from the **Agent queue** list.</span></span>

19. <span data-ttu-id="b5d60-296">**배포 Azure App Service**에 대해 환경에 유효한 **패키지 또는 폴더** 를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-296">For **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span> <span data-ttu-id="b5d60-297">**파일 또는 폴더 선택**에서 폴더 **위치**에 대해 **확인** 을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-297">On **Select File Or Folder**, select **OK** for the folder **Location**.</span></span>

20. <span data-ttu-id="b5d60-298">**변수** 탭에서 라는 변수를 찾습니다 `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` .</span><span class="sxs-lookup"><span data-stu-id="b5d60-298">On the **Variable** tab, find the variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`.</span></span> <span data-ttu-id="b5d60-299">변수 값을 **true**로 설정 하 고 해당 범위를 **Azure Stack 허브**로 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-299">Set the variable value to **true**, and set its scope to **Azure Stack Hub**.</span></span>

21. <span data-ttu-id="b5d60-300">**파이프라인** 탭에서 NorthwindCloud 상인-웹 아티팩트의 **연속 배포 트리거** 아이콘을 선택 하 고 **연속 배포 트리거** 를 **사용**으로 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-300">On the **Pipeline** tab, select the **Continuous deployment trigger** icon for the NorthwindCloud Traders-Web artifact and set the **Continuous deployment trigger** to **Enabled**.</span></span> <span data-ttu-id="b5d60-301">**NorthwindCloud 상인-용기** 아티팩트에 대해 동일한 작업을 수행 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-301">Do the same thing for the **NorthwindCloud Traders-Vessel** artifact.</span></span>

22. <span data-ttu-id="b5d60-302">Azure Stack 허브 환경의 경우 **배포 전 조건** 아이콘을 선택 하 여 트리거를 **릴리스 후**로 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-302">For the Azure Stack Hub environment, select the **Pre-deployment conditions** icon set the trigger to **After release**.</span></span>

23. <span data-ttu-id="b5d60-303">모든 변경 내용을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-303">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="b5d60-304">릴리스 작업에 대 한 일부 설정은 템플릿에서 릴리스 정의를 만들 때 [환경 변수로](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) 자동으로 정의 됩니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-304">Some settings for release tasks are automatically defined as [environment variables](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="b5d60-305">이러한 설정은 작업 설정에서 수정할 수 없지만 부모 환경 항목에서는 수정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-305">These settings can't be modified in the task settings but can be modified in the parent environment items.</span></span>

## <a name="create-a-release"></a><span data-ttu-id="b5d60-306">릴리스 만들기</span><span class="sxs-lookup"><span data-stu-id="b5d60-306">Create a release</span></span>

1. <span data-ttu-id="b5d60-307">**파이프라인** 탭에서 **릴리스** 목록을 열고 **릴리스 만들기**를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-307">On the **Pipeline** tab, open the **Release** list and select **Create release**.</span></span>

2. <span data-ttu-id="b5d60-308">릴리스에 대 한 설명을 입력 하 고, 올바른 아티팩트가 선택 되었는지 확인 한 다음, **만들기**를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-308">Enter a description for the release, check to see that the correct artifacts are selected, and then select **Create**.</span></span> <span data-ttu-id="b5d60-309">잠시 후 새 릴리스가 만들어지고 릴리스 이름이 링크로 표시 됨을 나타내는 배너가 표시 됩니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-309">After a few moments, a banner appears indicating that the new release was created and the release name is displayed as a link.</span></span> <span data-ttu-id="b5d60-310">링크를 선택 하 여 릴리스 요약 페이지를 표시 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-310">Select the link to see the release summary page.</span></span>

3. <span data-ttu-id="b5d60-311">릴리스 요약 페이지에는 릴리스에 대 한 세부 정보가 표시 됩니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-311">The release summary page shows details about the release.</span></span> <span data-ttu-id="b5d60-312">"릴리스-2"에 대 한 다음 화면 캡처에서 **환경** 섹션에는 Azure에 대 한 **배포 상태가** "진행 중"으로 표시 되 고 Azure Stack 허브의 상태가 "성공"으로 표시 됩니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-312">In the following screen capture for "Release-2", the **Environments** section shows the **Deployment status** for Azure as "IN PROGRESS", and the status for Azure Stack Hub is "SUCCEEDED".</span></span> <span data-ttu-id="b5d60-313">Azure 환경의 배포 상태가 "성공"으로 변경 되 면 릴리스가 승인 될 준비가 되었음을 나타내는 배너가 표시 됩니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-313">When the deployment status for the Azure environment changes to "SUCCEEDED", a banner appears indicating that the release is ready for approval.</span></span> <span data-ttu-id="b5d60-314">배포가 보류 중이거나 실패 한 경우 파란색 **(i)** 정보 아이콘이 표시 됩니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-314">When a deployment is pending or has failed, a blue **(i)** information icon is shown.</span></span> <span data-ttu-id="b5d60-315">아이콘을 마우스로 가리키면 지연 또는 실패의 이유가 포함 된 팝업이 표시 됩니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-315">Hover over the icon to see a pop-up that contains the reason for delay or failure.</span></span>

4. <span data-ttu-id="b5d60-316">릴리스 목록과 같은 다른 보기에는 승인이 보류 중임을 나타내는 아이콘도 표시 됩니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-316">Other views, like the list of releases, also display an icon that indicates approval is pending.</span></span> <span data-ttu-id="b5d60-317">이 아이콘의 팝업은 환경 이름 및 배포와 관련 된 자세한 정보를 표시 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-317">The pop-up for this icon shows the environment name and more details related to the deployment.</span></span> <span data-ttu-id="b5d60-318">관리자는 전반적인 릴리스 진행률을 확인 하 고 승인 대기 중인 릴리스를 확인 하는 것이 쉽습니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-318">It's easy for an admin see the overall progress of releases and see which releases are waiting for approval.</span></span>

## <a name="monitor-and-track-deployments"></a><span data-ttu-id="b5d60-319">배포 모니터링 및 추적</span><span class="sxs-lookup"><span data-stu-id="b5d60-319">Monitor and track deployments</span></span>

1. <span data-ttu-id="b5d60-320">**릴리스 2** 요약 페이지에서 **로그**를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-320">On the **Release-2** summary page, select **Logs**.</span></span> <span data-ttu-id="b5d60-321">배포 하는 동안이 페이지에는 에이전트의 실시간 로그가 표시 됩니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-321">During a deployment, this page shows the live log from the agent.</span></span> <span data-ttu-id="b5d60-322">왼쪽 창에는 각 환경에 대 한 배포의 각 작업 상태가 표시 됩니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-322">The left pane shows the status of each operation in the deployment for each environment.</span></span>

2. <span data-ttu-id="b5d60-323">배포 전 또는 배포 후 승인의 **작업** 열에서 사용자 아이콘을 선택 하 여 배포와 해당 사용자가 제공한 메시지를 승인 하거나 거부 한 사용자를 확인 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-323">Select the person icon in the **Action** column for a pre-deployment or post-deployment approval to see who approved (or rejected) the deployment and the message they provided.</span></span>

3. <span data-ttu-id="b5d60-324">배포가 완료 되 면 전체 로그 파일이 오른쪽 창에 표시 됩니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-324">After the deployment finishes, the entire log file is displayed in the right pane.</span></span> <span data-ttu-id="b5d60-325">왼쪽 창에서 **단계** 를 선택 하 여 **작업 초기화**와 같은 한 단계에 대 한 로그 파일을 확인 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-325">Select any **Step** in the left pane to see the log file for a single step, like **Initialize Job**.</span></span> <span data-ttu-id="b5d60-326">개별 로그를 확인 하는 기능을 사용 하면 전체 배포의 일부를 쉽게 추적 하 고 디버그할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-326">The ability to see individual logs makes it easier to trace and debug parts of the overall deployment.</span></span> <span data-ttu-id="b5d60-327">단계에 대 한 로그 파일을 **저장** 하거나 **모든 로그를 zip으로 다운로드**합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-327">**Save** the log file for a step or **Download all logs as zip**.</span></span>

4. <span data-ttu-id="b5d60-328">**요약** 탭을 열어 릴리스에 대 한 일반 정보를 확인 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-328">Open the **Summary** tab to see general information about the release.</span></span> <span data-ttu-id="b5d60-329">이 보기에는 빌드, 배포 된 환경, 배포 상태 및 릴리스에 대 한 기타 정보에 대 한 세부 정보가 표시 됩니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-329">This view shows details about the build, the environments it was deployed to, deployment status, and other information about the release.</span></span>

5. <span data-ttu-id="b5d60-330">특정 환경에 대 한 기존 배포 및 보류 중인 배포에 대 한 정보를 보려면 환경 링크 (**Azure** 또는 **Azure Stack 허브**)를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-330">Select an environment link (**Azure** or **Azure Stack Hub**) to see information about existing and pending deployments to a specific environment.</span></span> <span data-ttu-id="b5d60-331">이러한 뷰를 사용 하 여 동일한 빌드가 두 환경에 배포 되었는지 확인 하는 빠른 방법입니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-331">Use these views as a quick way to check that the same build was deployed to both environments.</span></span>

6. <span data-ttu-id="b5d60-332">배포 된 **프로덕션 앱** 을 브라우저에서 엽니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-332">Open the **deployed production app** in a browser.</span></span> <span data-ttu-id="b5d60-333">예를 들어 Azure 앱 Services 웹 사이트의 경우 URL을 엽니다 `https://[your-app-name\].azurewebsites.net` .</span><span class="sxs-lookup"><span data-stu-id="b5d60-333">For example, for the Azure App Services website, open the URL `https://[your-app-name\].azurewebsites.net`.</span></span>

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a><span data-ttu-id="b5d60-334">Azure와 Azure Stack Hub의 통합은 확장 가능한 클라우드 간 솔루션을 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-334">Integration of Azure and Azure Stack Hub provides a scalable cross-cloud solution</span></span>

<span data-ttu-id="b5d60-335">유연 하 고 강력한 다중 클라우드 서비스는 데이터 보안, 백업 및 중복성, 일관 되 고 빠른 가용성, 확장 가능한 저장소 및 배포, 지리적 규격 라우팅을 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-335">A flexible and robust multi-cloud service provides data security, back up and redundancy, consistent and rapid availability, scalable storage and distribution, and geo-compliant routing.</span></span> <span data-ttu-id="b5d60-336">수동으로 트리거된이 프로세스를 통해 호스팅된 웹 앱과 중요 한 데이터의 즉각적인 가용성 간에 안정적이 고 효율적인 부하를 전환할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b5d60-336">This manually triggered process ensures reliable and efficient load switching between hosted web apps and immediate availability of crucial data.</span></span>

## <a name="next-steps"></a><span data-ttu-id="b5d60-337">다음 단계</span><span class="sxs-lookup"><span data-stu-id="b5d60-337">Next steps</span></span>

- <span data-ttu-id="b5d60-338">Azure Cloud 패턴에 대해 자세히 알아보려면 [클라우드 디자인 패턴](https://docs.microsoft.com/azure/architecture/patterns)을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="b5d60-338">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](https://docs.microsoft.com/azure/architecture/patterns).</span></span>
