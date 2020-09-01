---
title: Azure 및 Azure Stack Hub에서 클라우드 간에 스케일링되는 앱 배포
description: Azure 및 Azure Stack Hub에서 클라우드 간에 스케일링되는 앱을 배포하는 방법을 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 5ae6c4323324fa104cd0e5c7b5198492be14b8eb
ms.sourcegitcommit: 56980e3c118ca0a672974ee3835b18f6e81b6f43
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/26/2020
ms.locfileid: "88886818"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a><span data-ttu-id="070fc-103">Azure 및 Azure Stack Hub를 사용하여 클라우드 간에 스케일링되는 앱 배포</span><span class="sxs-lookup"><span data-stu-id="070fc-103">Deploy an app that scales cross-cloud using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="070fc-104">트래픽 관리자를 통해 자동 스케일링을 사용하여 Azure Stack Hub 호스팅 웹앱에서 Azure 호스팅 웹앱으로 전환하는 수동으로 트리거되는 프로세스를 제공하는 클라우드 간 솔루션을 만드는 방법을 알아봅니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-104">Learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app with autoscaling via traffic manager.</span></span> <span data-ttu-id="070fc-105">이 프로세스를 사용하면 부하를 받을 때 유연하게 스케일링 가능한 클라우드 유틸리티를 만들 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-105">This process ensures flexible and scalable cloud utility when under load.</span></span>

<span data-ttu-id="070fc-106">이 패턴을 사용하면 테넌트가 앱을 퍼블릭 클라우드에서 즉시 실행하지 못할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-106">With this pattern, your tenant may not be ready to run your app in the public cloud.</span></span> <span data-ttu-id="070fc-107">그러나 앱의 급증하는 수요를 처리하기 위해 온-프레미스 환경에 필요한 용량을 유지하기가 경제적으로 어려운 기업도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-107">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="070fc-108">테넌트에서 탄력적인 퍼블릭 클라우드를 온-프레미스 솔루션에 활용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-108">Your tenant can make use of the elasticity of the public cloud with their on-premises solution.</span></span>

<span data-ttu-id="070fc-109">이 솔루션에서는 다음을 수행하는 샘플 환경을 빌드합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="070fc-110">다중 노드 웹앱을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-110">Create a multi-node web app.</span></span>
> - <span data-ttu-id="070fc-111">CD(지속적인 배포) 프로세스를 구성하고 관리합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-111">Configure and manage the Continuous Deployment (CD) process.</span></span>
> - <span data-ttu-id="070fc-112">웹앱을 Azure Stack Hub에 게시합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-112">Publish the web app to Azure Stack Hub.</span></span>
> - <span data-ttu-id="070fc-113">릴리스를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-113">Create a release.</span></span>
> - <span data-ttu-id="070fc-114">배포를 모니터링하고 추적하는 방법을 알아봅니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-114">Learn to monitor and track your deployments.</span></span>

> [!Tip]  
> <span data-ttu-id="070fc-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="070fc-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="070fc-116">Microsoft Azure Stack Hub는 Azure의 확장입니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-116">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="070fc-117">Azure Stack Hub는 온-프레미스 환경에 클라우드 컴퓨팅의 민첩성과 혁신을 제공하여 어디서나 하이브리드 앱을 빌드하고 배포할 수 있는 유일한 하이브리드 클라우드를 사용하도록 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-117">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="070fc-118">[하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 문서는 하이브리드 앱 디자인, 배포 및 운영에 대한 소프트웨어 품질(배치, 확장성, 가용성, 복원력, 관리 효율성 및 보안)의 핵심 요소를 검토합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-118">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="070fc-119">디자인 고려 사항은 하이브리드 앱 디자인을 최적화하고 프로덕션 환경에서 문제를 최소화하는 데 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-119">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="070fc-120">사전 요구 사항</span><span class="sxs-lookup"><span data-stu-id="070fc-120">Prerequisites</span></span>

- <span data-ttu-id="070fc-121">동작합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-121">Azure subscription.</span></span> <span data-ttu-id="070fc-122">필요한 경우 시작하기 전에 [체험 계정](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-122">If needed, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before beginning.</span></span>
- <span data-ttu-id="070fc-123">Azure Stack Hub 통합 시스템 또는 ASDK(Azure Stack Development Kit) 배포</span><span class="sxs-lookup"><span data-stu-id="070fc-123">An Azure Stack Hub integrated system or deployment of Azure Stack Development Kit (ASDK).</span></span>
  - <span data-ttu-id="070fc-124">Azure Stack Hub를 설치하는 방법에 대한 지침은 [ASDK 설치](/azure-stack/asdk/asdk-install.md)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="070fc-124">For instructions on installing Azure Stack Hub, see [Install the ASDK](/azure-stack/asdk/asdk-install.md).</span></span>
  - <span data-ttu-id="070fc-125">ASDK 배포 후 자동화 스크립트는 [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)에서 찾을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-125">For an ASDK post-deployment automation script, go to: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span></span>
  - <span data-ttu-id="070fc-126">이 설치를 완료하는 데 몇 시간 정도 걸릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-126">This installation may require a few hours to complete.</span></span>
- <span data-ttu-id="070fc-127">[App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS 서비스를 Azure Stack Hub에 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-127">Deploy [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS services to Azure Stack Hub.</span></span>
- <span data-ttu-id="070fc-128">Azure Stack Hub 환경 내에서 [계획/제안을 만듭니다](/azure-stack/operator/service-plan-offer-subscription-overview.md).</span><span class="sxs-lookup"><span data-stu-id="070fc-128">[Create plans/offers](/azure-stack/operator/service-plan-offer-subscription-overview.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="070fc-129">Azure Stack Hub 환경 내에서 [테넌트 구독을 만듭니다](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md).</span><span class="sxs-lookup"><span data-stu-id="070fc-129">[Create tenant subscription](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="070fc-130">테넌트 구독 내에서 웹앱을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-130">Create a web app within the tenant subscription.</span></span> <span data-ttu-id="070fc-131">나중에 사용할 수 있도록 새 웹앱 URL을 기록해 둡니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-131">Make note of the new web app URL for later use.</span></span>
- <span data-ttu-id="070fc-132">테넌트 구독 내에서 Azure Pipelines VM(가상 머신)을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-132">Deploy Azure Pipelines virtual machine (VM) within the tenant subscription.</span></span>
- <span data-ttu-id="070fc-133">.NET 3.5가 설치된 Windows Server 2016 VM이 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-133">Windows Server 2016 VM with .NET 3.5 is required.</span></span> <span data-ttu-id="070fc-134">이 VM은 Azure Stack Hub의 테넌트 구독에서 프라이빗 빌드 에이전트로 빌드됩니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-134">This VM will be built in the tenant subscription on Azure Stack Hub as the private build agent.</span></span>
- <span data-ttu-id="070fc-135">[SQL 2017 VM 이미지가 있는 Windows Server 2016](/azure-stack/operator/azure-stack-add-vm-image.md)은 Azure Stack Hub Marketplace에서 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-135">[Windows Server 2016 with SQL 2017 VM image](/azure-stack/operator/azure-stack-add-vm-image.md) is available in the Azure Stack Hub Marketplace.</span></span> <span data-ttu-id="070fc-136">이 이미지를 사용할 수 없는 경우 Azure Stack Hub 운영자에게 문의하여 환경에 추가해 달라고 요청합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-136">If this image isn't available, work with an Azure Stack Hub Operator to ensure it's added to the environment.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="070fc-137">문제 및 고려 사항</span><span class="sxs-lookup"><span data-stu-id="070fc-137">Issues and considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="070fc-138">확장성</span><span class="sxs-lookup"><span data-stu-id="070fc-138">Scalability</span></span>

<span data-ttu-id="070fc-139">클라우드 간 스케일링의 핵심 구성 요소는 퍼블릭 클라우드와 온-프레미스 클라우드 인프라 간에 즉시 스케일링 및 주문형 스케일링을 제공하여 일관적이고 안정적인 서비스를 제공하는 기능입니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-139">The key component of cross-cloud scaling is the ability to deliver immediate and on-demand scaling between public and on-premises cloud infrastructure, providing consistent and reliable service.</span></span>

### <a name="availability"></a><span data-ttu-id="070fc-140">가용성</span><span class="sxs-lookup"><span data-stu-id="070fc-140">Availability</span></span>

<span data-ttu-id="070fc-141">로컬로 배포된 앱이 온-프레미스 하드웨어 구성 및 소프트웨어 배포를 통해 고가용성으로 구성되도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-141">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="070fc-142">관리 효율</span><span class="sxs-lookup"><span data-stu-id="070fc-142">Manageability</span></span>

<span data-ttu-id="070fc-143">클라우드 간 솔루션은 환경 간에 원활한 관리 및 친숙한 인터페이스를 보장합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-143">The cross-cloud solution ensures seamless management and familiar interface between environments.</span></span> <span data-ttu-id="070fc-144">PowerShell은 플랫폼 간 관리에 권장되는 도구입니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-144">PowerShell is recommended for cross-platform management.</span></span>

## <a name="cross-cloud-scaling"></a><span data-ttu-id="070fc-145">클라우드 간 크기 조정</span><span class="sxs-lookup"><span data-stu-id="070fc-145">Cross-cloud scaling</span></span>

### <a name="get-a-custom-domain-and-configure-dns"></a><span data-ttu-id="070fc-146">사용자 지정 도메인 가져오기 및 DNS 구성</span><span class="sxs-lookup"><span data-stu-id="070fc-146">Get a custom domain and configure DNS</span></span>

<span data-ttu-id="070fc-147">도메인에 대한 DNS 영역 파일을 업데이트합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-147">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="070fc-148">Azure AD는 사용자 지정 도메인 이름의 소유권을 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-148">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="070fc-149">Azure 내에서 Azure/Microsoft 365/외부 DNS 레코드에 대해 [Azure DNS](/azure/dns/dns-getstarted-portal)를 사용하거나 [서로 다른 DNS 등록 기관](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider)에서 DNS 항목을 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-149">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

1. <span data-ttu-id="070fc-150">사용자 지정 도메인을 퍼블릭 등록자에 등록합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-150">Register a custom domain with a public registrar.</span></span>
2. <span data-ttu-id="070fc-151">도메인에 대한 도메인 이름 등록 기관에 로그인합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-151">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="070fc-152">승인된 관리자가 DNS를 업데이트해야 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-152">An approved admin may be required to make DNS updates.</span></span>
3. <span data-ttu-id="070fc-153">Azure AD에서 제공하는 DNS 항목을 추가하여 도메인에 대한 DNS 영역 파일을 업데이트합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-153">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="070fc-154">(DNS 항목은 이메일 라우팅 또는 웹 호스팅 동작에 영향을 주지 않습니다.)</span><span class="sxs-lookup"><span data-stu-id="070fc-154">(The DNS entry won't affect email routing or web hosting behaviors.)</span></span>

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a><span data-ttu-id="070fc-155">Azure Stack Hub에서 기본 다중 노드 웹앱 만들기</span><span class="sxs-lookup"><span data-stu-id="070fc-155">Create a default multi-node web app in Azure Stack Hub</span></span>

<span data-ttu-id="070fc-156">웹앱을 Azure 및 Azure Stack Hub에 배포하고 두 클라우드의 변경 내용을 자동으로 푸시하는 하이브리드 CI/CD(연속 통합 및 지속적인 배포)를 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-156">Set up hybrid continuous integration and continuous deployment (CI/CD) to deploy web apps to Azure and Azure Stack Hub and to autopush changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="070fc-157">실행을 위한 적절한 이미지(Windows Server 및 SQL)가 신디케이트된 Azure Stack Hub 및 App Service를 배포해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-157">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="070fc-158">자세한 내용은 App Service 설명서 [Azure Stack Hub에 App Service를 배포하기 위한 필수 조건](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md)을 검토하세요.</span><span class="sxs-lookup"><span data-stu-id="070fc-158">For more information, review the App Service documentation [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

### <a name="add-code-to-azure-repos"></a><span data-ttu-id="070fc-159">Azure Repos에 코드 추가</span><span class="sxs-lookup"><span data-stu-id="070fc-159">Add Code to Azure Repos</span></span>

<span data-ttu-id="070fc-160">Azure Repos</span><span class="sxs-lookup"><span data-stu-id="070fc-160">Azure Repos</span></span>

1. <span data-ttu-id="070fc-161">Azure Repos에 프로젝트를 만드는 권한이 있는 계정으로 Azure Repos에 로그인합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-161">Sign in to Azure Repos with an account that has project creation rights on Azure Repos.</span></span>

    <span data-ttu-id="070fc-162">하이브리드 CI/CD는 앱 코드와 인프라 코드 모두에 적용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-162">Hybrid CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="070fc-163">프라이빗 클라우드와 호스트된 클라우드 개발 모두에 [Azure Resource Manager 템플릿](https://azure.microsoft.com/resources/templates/)을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-163">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Azure Repos에서 프로젝트에 연결](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. <span data-ttu-id="070fc-165">기본 웹앱을 만들고 열어 **리포지토리를 복제**합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-165">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Azure 웹앱에서 리포지토리 복제](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="070fc-167">두 클라우드의 App Services에 대한 자체 포함 웹앱 배포 만들기</span><span class="sxs-lookup"><span data-stu-id="070fc-167">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="070fc-168">**WebApplication.csproj** 파일을 편집합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-168">Edit the **WebApplication.csproj** file.</span></span> <span data-ttu-id="070fc-169">`Runtimeidentifier`를 선택하고 `win10-x64`를 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-169">Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="070fc-170">([자체 포함 배포](/dotnet/core/deploying/deploy-with-vs#simpleSelf) 설명서를 참조하세요.)</span><span class="sxs-lookup"><span data-stu-id="070fc-170">(See [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![웹앱 프로젝트 파일 편집](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. <span data-ttu-id="070fc-172">팀 탐색기를 사용하여 코드를 Azure Repos에 체크 인합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-172">Check in the code to Azure Repos using Team Explorer.</span></span>

3. <span data-ttu-id="070fc-173">앱 코드가 Azure Repos에 체크 인되었는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-173">Confirm that the app code has been checked into Azure Repos.</span></span>

## <a name="create-the-build-definition"></a><span data-ttu-id="070fc-174">빌드 정의 만들기</span><span class="sxs-lookup"><span data-stu-id="070fc-174">Create the build definition</span></span>

1. <span data-ttu-id="070fc-175">Azure Pipelines에 로그인하여 빌드 정의를 만드는 기능을 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-175">Sign in to Azure Pipelines to confirm the ability to create build definitions.</span></span>

2. <span data-ttu-id="070fc-176">**-r win10-x64** 코드를 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-176">Add **-r win10-x64** code.</span></span> <span data-ttu-id="070fc-177">.NET Core를 사용하여 자체 포함 배포를 트리거하려면 이 코드를 추가해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-177">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![웹앱에 코드 추가](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. <span data-ttu-id="070fc-179">빌드를 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-179">Run the build.</span></span> <span data-ttu-id="070fc-180">[자체 포함 배포 빌드](/dotnet/core/deploying/deploy-with-vs#simpleSelf) 프로세스는 Azure 및 Azure Stack Hub에서 실행되는 아티팩트를 게시합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-180">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that run on Azure and Azure Stack Hub.</span></span>

## <a name="use-an-azure-hosted-agent"></a><span data-ttu-id="070fc-181">Azure 호스트된 에이전트 사용</span><span class="sxs-lookup"><span data-stu-id="070fc-181">Use an Azure hosted agent</span></span>

<span data-ttu-id="070fc-182">Azure Pipelines에서 호스트된 빌드 에이전트를 사용하면 간편하게 웹앱을 빌드하고 배포할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-182">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="070fc-183">Microsoft Azure에서 유지 관리 및 업그레이드를 자동으로 수행하므로 지속적이고 중단 없는 개발 주기가 가능합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-183">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="070fc-184">CD 프로세스 관리 및 구성</span><span class="sxs-lookup"><span data-stu-id="070fc-184">Manage and configure the CD process</span></span>

<span data-ttu-id="070fc-185">Azure Pipelines 및 Azure DevOps Services는 개발, 준비, QA, 프로덕션 환경 등의 여러 환경에 릴리스할 수 있도록 매우 유연하게 구성하고 쉽게 관리할 수 있는 파이프라인을 제공하며, 특정 단계에서 승인을 요구합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-185">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="070fc-186">릴리스 정의 만들기</span><span class="sxs-lookup"><span data-stu-id="070fc-186">Create release definition</span></span>

1. <span data-ttu-id="070fc-187">Azure DevOps Services의 **빌드 및 릴리스** 섹션에 있는 **릴리스** 탭 아래에서 **+** 단추를 선택하여 새 릴리스를 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-187">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![릴리스 정의 만들기](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. <span data-ttu-id="070fc-189">Azure App Service 배포 템플릿을 적용합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-189">Apply the Azure App Service Deployment template.</span></span>

   ![Azure App Service 배포 템플릿 적용](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. <span data-ttu-id="070fc-191">**아티팩트 추가**에서 Azure 클라우드 빌드 앱에 대한 아티팩트를 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-191">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Azure 클라우드 빌드에 아티팩트 추가](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. <span data-ttu-id="070fc-193">[파이프라인] 탭에서 환경의 **단계, 작업** 링크를 선택하고 Azure 클라우드 환경 값을 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-193">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Azure 클라우드 환경 값 설정](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. <span data-ttu-id="070fc-195">**환경 이름**을 설정하고 Azure 클라우드 엔드포인트에 대한 **Azure 구독**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-195">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Azure 클라우드 엔드포인트에 대한 Azure 구독 선택](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. <span data-ttu-id="070fc-197">**앱 서비스 이름**에서 필요한 Azure 앱 서비스 이름을 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-197">Under **App service name**, set the required Azure app service name.</span></span>

      ![Azure 앱 서비스 이름 설정](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. <span data-ttu-id="070fc-199">**에이전트 큐**에서 Azure 클라우드 호스팅 환경으로 "호스트된 VS2017"을 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-199">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Azure 클라우드 호스팅 환경의 에이전트 큐 설정](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. <span data-ttu-id="070fc-201">[Azure App Service 배포] 메뉴에서 해당 환경의 유효한 **패키지 또는 폴더**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-201">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="070fc-202">**폴더 위치**에 대해 **확인**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-202">Select **OK** to **folder location**.</span></span>
  
      ![Azure App Service 환경에 사용할 패키지 또는 폴더 선택](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![Azure App Service 환경에 사용할 패키지 또는 폴더 선택](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. <span data-ttu-id="070fc-205">모든 변경 내용을 저장하고 **릴리스 파이프라인**으로 돌아갑니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-205">Save all changes and go back to **release pipeline**.</span></span>

    ![릴리스 파이프라인의 변경 내용 저장](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. <span data-ttu-id="070fc-207">새 아티팩트를 추가하고 Azure Stack Hub 앱에 대한 빌드를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-207">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Azure Stack Hub 앱에 대한 새 아티팩트 추가](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. <span data-ttu-id="070fc-209">Azure App Service 배포를 적용하여 환경을 하나 더 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-209">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Azure App Service 배포에 환경 추가](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. <span data-ttu-id="070fc-211">새 환경의 이름을 "Azure Stack"으로 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-211">Name the new environment "Azure Stack".</span></span>

    ![Azure App Service 배포에서 환경의 이름 지정](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. <span data-ttu-id="070fc-213">**작업** 탭에서 Azure Stack 환경을 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-213">Find the Azure Stack environment under **Task** tab.</span></span>

    ![Azure Stack 환경](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. <span data-ttu-id="070fc-215">Azure Stack 엔드포인트의 구독을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-215">Select the subscription for the Azure Stack endpoint.</span></span>

    ![Azure Stack 엔드포인트의 구독 선택](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. <span data-ttu-id="070fc-217">앱 서비스 이름으로 Azure Stack 웹앱 이름을 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-217">Set the Azure Stack web app name as the App service name.</span></span>
    <span data-ttu-id="070fc-218">![Azure Stack 웹앱 이름 설정](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span><span class="sxs-lookup"><span data-stu-id="070fc-218">![Set Azure Stack web app name](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span></span>

16. <span data-ttu-id="070fc-219">Azure Stack 에이전트를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-219">Select the Azure Stack agent.</span></span>

    ![Azure Stack 에이전트 선택](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. <span data-ttu-id="070fc-221">[Azure App Service 배포] 섹션에서 해당 환경의 유효한 **패키지 또는 폴더**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-221">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="070fc-222">폴더 위치에 대해 **확인**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-222">Select **OK** to folder location.</span></span>

    ![Azure App Service 배포에 사용할 폴더 선택](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![Azure App Service 배포에 사용할 폴더 선택](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. <span data-ttu-id="070fc-225">[변수] 탭에서 `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`라는 변수를 추가하고 해당 값을 **true**, 범위를 Azure Stack으로 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-225">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack.</span></span>

    ![Azure 앱 배포에 변수 추가](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. <span data-ttu-id="070fc-227">두 아티팩트 모두에서 **지속적인** 배포 트리거 아이콘을 선택하고 **지속적인** 배포 트리거를 사용하도록 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-227">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![지속적인 배포 트리거 선택](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. <span data-ttu-id="070fc-229">Azure Stack 환경에서 **배포 전** 조건 아이콘을 선택하고 트리거를 **릴리스 후**로 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-229">Select the **Pre-deployment** conditions icon in the Azure Stack environment and set the trigger to **After release.**</span></span>

    ![배포 전 조건 선택](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. <span data-ttu-id="070fc-231">모든 변경 내용을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-231">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="070fc-232">템플릿에서 릴리스 정의를 만들 때 작업에 대한 일부 설정이 자동으로 [환경 변수](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables)로 정의되었을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-232">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="070fc-233">이러한 설정은 작업 설정에서 수정할 수 없습니다. 이러한 설정을 편집하려면 부모 환경 항목을 선택해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-233">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a><span data-ttu-id="070fc-234">Visual Studio를 통해 Azure Stack Hub에 게시</span><span class="sxs-lookup"><span data-stu-id="070fc-234">Publish to Azure Stack Hub via Visual Studio</span></span>

<span data-ttu-id="070fc-235">Azure DevOps Services 빌드는 엔드포인트를 만들어서 Azure Stack Hub에 Azure 서비스 앱을 배포할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-235">By creating endpoints, an Azure DevOps Services build can deploy Azure Service apps to Azure Stack Hub.</span></span> <span data-ttu-id="070fc-236">Azure Pipelines는 빌드 에이전트에 연결하고, 빌드 에이전트는 Azure Stack Hub에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-236">Azure Pipelines connects to the build agent, which connects to Azure Stack Hub.</span></span>

1. <span data-ttu-id="070fc-237">Azure DevOps Services에 로그인하고 앱 설정 페이지로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-237">Sign in to Azure DevOps Services and go to the app settings page.</span></span>

2. <span data-ttu-id="070fc-238">**설정**에서 **보안**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-238">On **Settings**, select **Security**.</span></span>

3. <span data-ttu-id="070fc-239">**VSTS 그룹**에서 **엔드포인트 작성자**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-239">In **VSTS Groups**, select **Endpoint Creators**.</span></span>

4. <span data-ttu-id="070fc-240">**구성원** 탭에서 **추가**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-240">On the **Members** tab, select **Add**.</span></span>

5. <span data-ttu-id="070fc-241">**사용자 및 그룹 추가**에서 사용자 이름을 입력하고 사용자 목록에서 해당 사용자를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-241">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

6. <span data-ttu-id="070fc-242">**변경 내용 저장**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-242">Select **Save changes**.</span></span>

7. <span data-ttu-id="070fc-243">**VSTS 그룹** 목록에서 **엔드포인트 관리자**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-243">In the **VSTS Groups** list, select **Endpoint Administrators**.</span></span>

8. <span data-ttu-id="070fc-244">**구성원** 탭에서 **추가**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-244">On the **Members** tab, select **Add**.</span></span>

9. <span data-ttu-id="070fc-245">**사용자 및 그룹 추가**에서 사용자 이름을 입력하고 사용자 목록에서 해당 사용자를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-245">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

10. <span data-ttu-id="070fc-246">**변경 내용 저장**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-246">Select **Save changes**.</span></span>

<span data-ttu-id="070fc-247">이제 엔드포인트 정보가 있으므로 Azure Pipelines에서 Azure Stack Hub에 연결할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-247">Now that the endpoint information exists, the Azure Pipelines to Azure Stack Hub connection is ready to use.</span></span> <span data-ttu-id="070fc-248">Azure Stack Hub의 빌드 에이전트는 Azure Pipelines에서 명령을 가져온 다음, Azure Stack Hub와 통신할 수 있도록 엔드포인트 정보를 전달합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-248">The build agent in Azure Stack Hub gets instructions from Azure Pipelines and then the agent conveys endpoint information for communication with Azure Stack Hub.</span></span>

## <a name="develop-the-app-build"></a><span data-ttu-id="070fc-249">앱 빌드 개발</span><span class="sxs-lookup"><span data-stu-id="070fc-249">Develop the app build</span></span>

> [!Note]  
> <span data-ttu-id="070fc-250">실행을 위한 적절한 이미지(Windows Server 및 SQL)가 신디케이트된 Azure Stack Hub 및 App Service를 배포해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-250">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="070fc-251">자세한 내용은 [Azure Stack Hub에 App Service를 배포하기 위한 필수 조건](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="070fc-251">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

<span data-ttu-id="070fc-252">Azure Repos의 웹앱 코드와 같은 [Azure Resource Manager 템플릿](https://azure.microsoft.com/resources/templates/)을 사용하여 두 클라우드를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-252">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) like web app code from Azure Repos to deploy to both clouds.</span></span>

### <a name="add-code-to-an-azure-repos-project"></a><span data-ttu-id="070fc-253">Azure Repos 프로젝트에 코드 추가</span><span class="sxs-lookup"><span data-stu-id="070fc-253">Add code to an Azure Repos project</span></span>

1. <span data-ttu-id="070fc-254">Azure Stack Hub에 프로젝트를 만드는 권한이 있는 계정으로 Azure Repos에 로그인합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-254">Sign in to Azure Repos with an account that has project creation rights on Azure Stack Hub.</span></span>

2. <span data-ttu-id="070fc-255">기본 웹앱을 만들고 열어 **리포지토리를 복제**합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-255">**Clone the repository** by creating and opening the default web app.</span></span>

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="070fc-256">두 클라우드의 App Services에 대한 자체 포함 웹앱 배포 만들기</span><span class="sxs-lookup"><span data-stu-id="070fc-256">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="070fc-257">다음과 같이 **WebApplication.csproj** 파일을 편집합니다. `Runtimeidentifier`를 선택한 다음, `win10-x64`를 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-257">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and then add `win10-x64`.</span></span> <span data-ttu-id="070fc-258">자세한 내용은 [자체 포함 배포](/dotnet/core/deploying/deploy-with-vs#simpleSelf) 설명서를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="070fc-258">For more information, see [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.</span></span>

2. <span data-ttu-id="070fc-259">팀 탐색기를 사용하여 코드를 Azure Repos에 체크 인합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-259">Use Team Explorer to check the code into Azure Repos.</span></span>

3. <span data-ttu-id="070fc-260">앱 코드가 Azure Repos에 체크 인되었는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-260">Confirm that the app code was checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="070fc-261">빌드 정의 만들기</span><span class="sxs-lookup"><span data-stu-id="070fc-261">Create the build definition</span></span>

1. <span data-ttu-id="070fc-262">빌드 정의를 만들 수 있는 계정으로 Azure Pipelines에 로그인합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-262">Sign in to Azure Pipelines with an account that can create a build definition.</span></span>

2. <span data-ttu-id="070fc-263">프로젝트에 대한 **웹 애플리케이션 빌드** 페이지로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-263">Go to the **Build Web Application** page for the project.</span></span>

3. <span data-ttu-id="070fc-264">**인수**에서 **-r win10-x64** 코드를 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-264">In **Arguments**, add **-r win10-x64** code.</span></span> <span data-ttu-id="070fc-265">.NET Core를 사용하여 자체 포함 배포를 트리거하려면 이 코드를 추가해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-265">This addition is required to trigger a self-contained deployment with .NET Core.</span></span>

4. <span data-ttu-id="070fc-266">빌드를 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-266">Run the build.</span></span> <span data-ttu-id="070fc-267">[자체 포함 배포 빌드](/dotnet/core/deploying/deploy-with-vs#simpleSelf) 프로세스는 Azure 및 Azure Stack Hub에서 실행 가능한 아티팩트를 게시합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-267">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="use-an-azure-hosted-build-agent"></a><span data-ttu-id="070fc-268">Azure 호스트된 빌드 에이전트 사용</span><span class="sxs-lookup"><span data-stu-id="070fc-268">Use an Azure hosted build agent</span></span>

<span data-ttu-id="070fc-269">Azure Pipelines에서 호스트된 빌드 에이전트를 사용하면 간편하게 웹앱을 빌드하고 배포할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-269">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="070fc-270">Microsoft Azure에서 유지 관리 및 업그레이드를 자동으로 수행하므로 지속적이고 중단 없는 개발 주기가 가능합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-270">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="configure-the-continuous-deployment-cd-process"></a><span data-ttu-id="070fc-271">CD(지속적인 배포) 프로세스를 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-271">Configure the continuous deployment (CD) process</span></span>

<span data-ttu-id="070fc-272">Azure Pipelines 및 Azure DevOps Services는 개발, 준비, QA, 프로덕션 환경 등의 여러 환경에 릴리스할 수 있도록 매우 유연하게 구성하고 쉽게 관리할 수 있는 파이프라인을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-272">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, quality assurance (QA), and production.</span></span> <span data-ttu-id="070fc-273">이 프로세스는 앱 수명 주기의 특정 단계에서 승인이 필요할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-273">This process can include requiring approvals at specific stages of the app life cycle.</span></span>

#### <a name="create-release-definition"></a><span data-ttu-id="070fc-274">릴리스 정의 만들기</span><span class="sxs-lookup"><span data-stu-id="070fc-274">Create release definition</span></span>

<span data-ttu-id="070fc-275">앱 빌드 프로세스의 마지막 단계는 릴리스 정의 만들기입니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-275">Creating a release definition is the final step in the app build process.</span></span> <span data-ttu-id="070fc-276">이 릴리스 정의는 릴리스를 만들고 빌드를 배포하는 데 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-276">This release definition is used to create a release and deploy a build.</span></span>

1. <span data-ttu-id="070fc-277">Azure Pipelines에 로그인하고 프로젝트에 대한 **빌드 및 릴리스**로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-277">Sign in to Azure Pipelines and go to **Build and Release** for the project.</span></span>

2. <span data-ttu-id="070fc-278">**릴리스** 탭에서 **[+]** 기호를 선택한 다음, **릴리스 정의 만들기**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-278">On the **Releases** tab, select **[ + ]** and then pick **Create release definition**.</span></span>

3. <span data-ttu-id="070fc-279">**템플릿 선택**에서 **Azure App Service 배포**를 선택한 다음, **적용**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-279">On **Select a Template**, choose **Azure App Service Deployment**, and then select **Apply**.</span></span>

4. <span data-ttu-id="070fc-280">**아티팩트 추가**의 **원본(빌드 정의)** 에서 Azure 클라우드 빌드 앱을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-280">On **Add artifact**, from the **Source (Build definition)**, select the Azure Cloud build app.</span></span>

5. <span data-ttu-id="070fc-281">**파이프라인** 탭에서 **단계 1**, **작업 1** 링크를 선택하여 **환경 작업을 살펴봅니다**.</span><span class="sxs-lookup"><span data-stu-id="070fc-281">On the **Pipeline** tab, select the **1 Phase**, **1 Task** link to **View environment tasks**.</span></span>

6. <span data-ttu-id="070fc-282">**작업** 탭에서 **환경 이름**으로 Azure를 입력하고, **Azure 구독** 목록에서 AzureCloud Traders-Web EP를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-282">On the **Tasks** tab, enter Azure as the **Environment name** and select the AzureCloud Traders-Web EP from the **Azure subscription** list.</span></span>

7. <span data-ttu-id="070fc-283">**Azure 앱 서비스 이름**을 입력합니다. 다음 화면 캡처에서는 `northwindtraders`입니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-283">Enter the **Azure app service name**, which is `northwindtraders` in the next screen capture.</span></span>

8. <span data-ttu-id="070fc-284">에이전트 단계의 경우 **에이전트 큐** 목록에서 **호스트된 VS2017**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-284">For the Agent phase, select **Hosted VS2017** from the **Agent queue** list.</span></span>

9. <span data-ttu-id="070fc-285">**Azure App Service 배포**에서 해당 환경의 유효한 **패키지 또는 폴더**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-285">In **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span>

10. <span data-ttu-id="070fc-286">**파일 또는 폴더**에서 **위치**에 대해 **확인**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-286">In **Select File or Folder**, select **OK** to **Location**.</span></span>

11. <span data-ttu-id="070fc-287">모든 변경 내용을 저장하고 **파이프라인**으로 돌아갑니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-287">Save all changes and go back to **Pipeline**.</span></span>

12. <span data-ttu-id="070fc-288">**파이프라인** 탭에서 **아티팩트 추가**를 선택하고, **원본(빌드 정의)** 목록에서 **NorthwindCloud Traders-Vessel**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-288">On the **Pipeline** tab, select **Add artifact**, and choose the **NorthwindCloud Traders-Vessel** from the **Source (Build Definition)** list.</span></span>

13. <span data-ttu-id="070fc-289">**템플릿 선택**에서 또 다른 환경을 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-289">On **Select a Template**, add another environment.</span></span> <span data-ttu-id="070fc-290">**Azure App Service 배포**를 선택한 다음, **적용**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-290">Pick **Azure App Service Deployment** and then select **Apply**.</span></span>

14. <span data-ttu-id="070fc-291">**환경 이름**으로 `Azure Stack Hub`를 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-291">Enter `Azure Stack Hub` as the **Environment name**.</span></span>

15. <span data-ttu-id="070fc-292">**작업** 탭에서 Azure Stack Hub를 찾아 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-292">On the **Tasks** tab, find and select Azure Stack Hub.</span></span>

16. <span data-ttu-id="070fc-293">**Azure 구독** 목록에서 Azure Stack Hub 엔드포인트로 **AzureStack Traders-Vessel EP**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-293">From the **Azure subscription** list, select **AzureStack Traders-Vessel EP** for the Azure Stack Hub endpoint.</span></span>

17. <span data-ttu-id="070fc-294">**앱 서비스 이름**으로 Azure Stack Hub 웹앱 이름을 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-294">Enter the Azure Stack Hub web app name as the **App service name**.</span></span>

18. <span data-ttu-id="070fc-295">**에이전트 선택**의 **에이전트 큐** 목록에서 **AzureStack -b Douglas Fir**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-295">Under **Agent selection**, pick **AzureStack -b Douglas Fir** from the **Agent queue** list.</span></span>

19. <span data-ttu-id="070fc-296">**Azure App Service 배포**에서 해당 환경의 유효한 **패키지 또는 폴더**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-296">For **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span> <span data-ttu-id="070fc-297">**파일 또는 폴더 선택**에서 폴더 **위치**에 대해 **확인**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-297">On **Select File Or Folder**, select **OK** for the folder **Location**.</span></span>

20. <span data-ttu-id="070fc-298">**변수** 탭에서 `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`라는 변수를 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-298">On the **Variable** tab, find the variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`.</span></span> <span data-ttu-id="070fc-299">변수 값을 **true**로 설정하고, 범위를 **Azure Stack Hub**로 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-299">Set the variable value to **true**, and set its scope to **Azure Stack Hub**.</span></span>

21. <span data-ttu-id="070fc-300">**파이프라인** 탭에서 NorthwindCloud Traders-Web 아티팩트에 대한 **지속적인 배포 트리거** 아이콘을 선택하고, **지속적인 배포 트리거**를 **사용**으로 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-300">On the **Pipeline** tab, select the **Continuous deployment trigger** icon for the NorthwindCloud Traders-Web artifact and set the **Continuous deployment trigger** to **Enabled**.</span></span> <span data-ttu-id="070fc-301">**NorthwindCloud Traders-Vessel** 아티팩트에 대해 동일한 작업을 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-301">Do the same thing for the **NorthwindCloud Traders-Vessel** artifact.</span></span>

22. <span data-ttu-id="070fc-302">Azure Stack Hub 환경에서 **배포 전 조건** 아이콘을 선택하고, 트리거를 **릴리스 후**로 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-302">For the Azure Stack Hub environment, select the **Pre-deployment conditions** icon set the trigger to **After release**.</span></span>

23. <span data-ttu-id="070fc-303">모든 변경 내용을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-303">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="070fc-304">템플릿에서 릴리스 정의를 만들 때 작업에 대한 일부 설정이 자동으로 [환경 변수](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables)로 정의됩니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-304">Some settings for release tasks are automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="070fc-305">이러한 설정은 작업 설정에서 수정할 수 없고, 부모 환경 항목에서 수정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-305">These settings can't be modified in the task settings but can be modified in the parent environment items.</span></span>

## <a name="create-a-release"></a><span data-ttu-id="070fc-306">릴리스 만들기</span><span class="sxs-lookup"><span data-stu-id="070fc-306">Create a release</span></span>

1. <span data-ttu-id="070fc-307">**파이프라인** 탭에서 **릴리스** 목록을 열고 **릴리스 만들기**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-307">On the **Pipeline** tab, open the **Release** list and select **Create release**.</span></span>

2. <span data-ttu-id="070fc-308">릴리스에 대한 설명을 입력하고 올바른 아티팩트가 선택되었는지 확인한 다음, **만들기**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-308">Enter a description for the release, check to see that the correct artifacts are selected, and then select **Create**.</span></span> <span data-ttu-id="070fc-309">잠시 후 새 릴리스가 생성되었다는 배너가 나타나고 릴리스 이름이 링크로 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-309">After a few moments, a banner appears indicating that the new release was created and the release name is displayed as a link.</span></span> <span data-ttu-id="070fc-310">이 링크를 선택하여 릴리스 요약 페이지를 살펴봅니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-310">Select the link to see the release summary page.</span></span>

3. <span data-ttu-id="070fc-311">릴리스 요약 페이지에는 릴리스에 대한 세부 정보가 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-311">The release summary page shows details about the release.</span></span> <span data-ttu-id="070fc-312">"릴리스-2"의 다음 화면 캡처에서 **환경** 섹션에는 Azure **배포 상태**가 "진행 중"으로 표시되고, Azure Stack Hub의 상태는 "성공"으로 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-312">In the following screen capture for "Release-2", the **Environments** section shows the **Deployment status** for Azure as "IN PROGRESS", and the status for Azure Stack Hub is "SUCCEEDED".</span></span> <span data-ttu-id="070fc-313">Azure 환경의 배포 상태가 "성공"으로 변경되면 릴리스를 승인할 준비가 되었다는 배너가 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-313">When the deployment status for the Azure environment changes to "SUCCEEDED", a banner appears indicating that the release is ready for approval.</span></span> <span data-ttu-id="070fc-314">배포가 보류 중이거나 실패한 경우 파란색 **(i)** 정보 아이콘이 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-314">When a deployment is pending or has failed, a blue **(i)** information icon is shown.</span></span> <span data-ttu-id="070fc-315">아이콘을 마우스로 가리키면 지연 또는 실패 이유를 설명하는 팝업이 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-315">Hover over the icon to see a pop-up that contains the reason for delay or failure.</span></span>

4. <span data-ttu-id="070fc-316">릴리스 목록과 같은 기타 보기에도 승인이 보류 중이라는 아이콘이 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-316">Other views, like the list of releases, also display an icon that indicates approval is pending.</span></span> <span data-ttu-id="070fc-317">이 아이콘의 팝업은 환경 이름 및 배포와 관련된 자세한 정보를 표시합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-317">The pop-up for this icon shows the environment name and more details related to the deployment.</span></span> <span data-ttu-id="070fc-318">관리자는 손쉽게 전반적인 릴리스 진행률을 확인하고 승인 대기 중인 릴리스를 볼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-318">It's easy for an admin see the overall progress of releases and see which releases are waiting for approval.</span></span>

## <a name="monitor-and-track-deployments"></a><span data-ttu-id="070fc-319">배포 모니터링 및 추적</span><span class="sxs-lookup"><span data-stu-id="070fc-319">Monitor and track deployments</span></span>

1. <span data-ttu-id="070fc-320">**릴리스-2** 요약 페이지에서 **로그**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-320">On the **Release-2** summary page, select **Logs**.</span></span> <span data-ttu-id="070fc-321">배포하는 동안 이 페이지에는 에이전트의 실시간 로그가 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-321">During a deployment, this page shows the live log from the agent.</span></span> <span data-ttu-id="070fc-322">왼쪽 창에는 각 환경에 대한 각 배포 작업의 상태가 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-322">The left pane shows the status of each operation in the deployment for each environment.</span></span>

2. <span data-ttu-id="070fc-323">배포 전 또는 배포 후 승인에 대한 **작업** 열에서 사람 아이콘을 선택하여 배포를 승인(또는 거부)한 사람과 그 사람이 제공한 메시지를 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-323">Select the person icon in the **Action** column for a pre-deployment or post-deployment approval to see who approved (or rejected) the deployment and the message they provided.</span></span>

3. <span data-ttu-id="070fc-324">배포가 완료되면 전체 로그 파일이 오른쪽 창에 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-324">After the deployment finishes, the entire log file is displayed in the right pane.</span></span> <span data-ttu-id="070fc-325">왼쪽 창에서 원하는 **단계**를 선택하여 **작업 초기화** 같은 단일 단계에 대한 로그 파일을 살펴봅니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-325">Select any **Step** in the left pane to see the log file for a single step, like **Initialize Job**.</span></span> <span data-ttu-id="070fc-326">개별 로그를 확인하는 기능을 사용하면 전체 배포의 일부를 쉽게 추적하고 디버그할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-326">The ability to see individual logs makes it easier to trace and debug parts of the overall deployment.</span></span> <span data-ttu-id="070fc-327">단계에 대한 로그 파일을 **저장**하거나 **모든 로그를 zip으로 다운로드**합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-327">**Save** the log file for a step or **Download all logs as zip**.</span></span>

4. <span data-ttu-id="070fc-328">**요약** 탭을 열고 릴리스에 대한 일반 정보를 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-328">Open the **Summary** tab to see general information about the release.</span></span> <span data-ttu-id="070fc-329">이 보기에는 빌드, 빌드가 배포된 환경, 배포 상태에 대한 자세한 정보와 릴리스에 대한 기타 정보가 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-329">This view shows details about the build, the environments it was deployed to, deployment status, and other information about the release.</span></span>

5. <span data-ttu-id="070fc-330">환경 링크(**Azure** 또는 **Azure Stack Hub**)를 선택하여 특정 환경의 기존 배포 및 보류 중인 배포를 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-330">Select an environment link (**Azure** or **Azure Stack Hub**) to see information about existing and pending deployments to a specific environment.</span></span> <span data-ttu-id="070fc-331">이러한 보기를 사용하여 동일한 빌드가 두 환경에 배포되었는지 신속하게 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-331">Use these views as a quick way to check that the same build was deployed to both environments.</span></span>

6. <span data-ttu-id="070fc-332">브라우저에서 **배포된 프로덕션 앱**을 엽니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-332">Open the **deployed production app** in a browser.</span></span> <span data-ttu-id="070fc-333">예를 들어 Azure App Service 웹 사이트의 경우 URL `https://[your-app-name\].azurewebsites.net`을 엽니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-333">For example, for the Azure App Services website, open the URL `https://[your-app-name\].azurewebsites.net`.</span></span>

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a><span data-ttu-id="070fc-334">Azure와 Azure Stack Hub를 통합하여 확장성 있는 클라우드 간 솔루션 제공</span><span class="sxs-lookup"><span data-stu-id="070fc-334">Integration of Azure and Azure Stack Hub provides a scalable cross-cloud solution</span></span>

<span data-ttu-id="070fc-335">유연하고 강력한 다중 클라우드 서비스는 데이터 보안, 백업 및 중복성, 일관적이고 빠른 가용성, 확장성 있는 스토리지 및 배포, 지리적 규격 라우팅을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-335">A flexible and robust multi-cloud service provides data security, back up and redundancy, consistent and rapid availability, scalable storage and distribution, and geo-compliant routing.</span></span> <span data-ttu-id="070fc-336">수동으로 트리거되는 이 프로세스를 통해 호스트된 웹앱 간에 안정적이고 효율적으로 부하를 전환하고 중요한 데이터의 가용성을 즉시 확보할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="070fc-336">This manually triggered process ensures reliable and efficient load switching between hosted web apps and immediate availability of crucial data.</span></span>

## <a name="next-steps"></a><span data-ttu-id="070fc-337">다음 단계</span><span class="sxs-lookup"><span data-stu-id="070fc-337">Next steps</span></span>

- <span data-ttu-id="070fc-338">Azure Cloud 패턴에 대해 자세히 알아보려면 [클라우드 디자인 패턴](/azure/architecture/patterns)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="070fc-338">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
