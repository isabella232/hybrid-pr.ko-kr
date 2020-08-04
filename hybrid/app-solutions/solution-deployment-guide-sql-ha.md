---
title: Azure 및 Azure Stack Hub에 SQL Server 2016 가용성 그룹 배포
description: Azure 및 Azure Stack Hub에 SQL Server 2016 가용성 그룹을 배포하는 방법에 대해 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 85b859457b9b54a973c5fc23329b927212b60a07
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477085"
---
# <a name="deploy-a-sql-server-2016-availability-group-to-azure-and-azure-stack-hub"></a><span data-ttu-id="c981d-103">Azure 및 Azure Stack Hub에 SQL Server 2016 가용성 그룹 배포</span><span class="sxs-lookup"><span data-stu-id="c981d-103">Deploy a SQL Server 2016 availability group to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="c981d-104">이 문서에서는 두 개의 Azure Stack Hub 환경에서 비동기 DR(재해 복구) 사이트를 사용하여 기본 HA(고가용성) SQL Server 2016 Enterprise 클러스터를 자동으로 배포하는 과정을 안내합니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-104">This article will step you through an automated deployment of a basic highly available (HA) SQL Server 2016 Enterprise cluster with an asynchronous disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="c981d-105">SQL Server 2016 및 고가용성에 대한 자세한 내용은 [Always On 가용성 그룹: 고가용성 및 재해 복구 솔루션](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="c981d-105">To learn more about SQL Server 2016 and high availability, see [Always On availability groups: a high-availability and disaster-recovery solution](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span></span>

<span data-ttu-id="c981d-106">이 솔루션에서는 다음을 수행하는 샘플 환경을 빌드합니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="c981d-107">두 Azure Stack Hub에서 배포를 오케스트레이션합니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="c981d-108">Docker를 사용하여 Azure API 프로필의 종속성 문제를 최소화합니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="c981d-109">재해 복구 사이트를 사용하여 기본 고가용성 SQL Server 2016 Enterprise 클러스터를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-109">Deploy a basic highly available SQL Server 2016 Enterprise cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="c981d-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="c981d-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="c981d-111">Microsoft Azure Stack Hub는 Azure의 확장입니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="c981d-112">Azure Stack Hub는 온-프레미스 환경에 클라우드 컴퓨팅의 민첩성과 혁신을 제공하여 어디서나 하이브리드 앱을 빌드하고 배포할 수 있는 유일한 하이브리드 클라우드를 사용하도록 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="c981d-113">[하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 문서는 하이브리드 앱 디자인, 배포 및 운영에 대한 소프트웨어 품질(배치, 확장성, 가용성, 복원력, 관리 효율성 및 보안)의 핵심 요소를 검토합니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="c981d-114">디자인 고려 사항은 하이브리드 앱 디자인을 최적화하고 프로덕션 환경에서 문제를 최소화하는 데 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-sql-server-2016"></a><span data-ttu-id="c981d-115">SQL Server 2016에 대한 아키텍처</span><span class="sxs-lookup"><span data-stu-id="c981d-115">Architecture for SQL Server 2016</span></span>

![SQL Server 2016 SQL HA Azure Stack Hub](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a><span data-ttu-id="c981d-117">SQL Server 2016에 대한 필수 구성 요소</span><span class="sxs-lookup"><span data-stu-id="c981d-117">Prerequisites for SQL Server 2016</span></span>

- <span data-ttu-id="c981d-118">2개의 연결된 Azure Stack Hub 통합 시스템(Azure Stack Hub)</span><span class="sxs-lookup"><span data-stu-id="c981d-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="c981d-119">이 배포는 ASDK(Azure Stack Development Kit)에서 작동하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="c981d-120">Azure Stack Hub에 대한 자세한 내용은 [Azure Stack 개요](https://azure.microsoft.com/overview/azure-stack/)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="c981d-120">To learn more about Azure Stack Hub, see the [Azure Stack overview](https://azure.microsoft.com/overview/azure-stack/).</span></span>
- <span data-ttu-id="c981d-121">각 Azure Stack Hub에 대한 테넌트 구독</span><span class="sxs-lookup"><span data-stu-id="c981d-121">A tenant subscription on each Azure Stack Hub.</span></span>
  - <span data-ttu-id="c981d-122">**각 Azure Stack Hub에 대한 각 구독 ID와 Azure Resource Manager 엔드포인트를 기록해 둡니다.**</span><span class="sxs-lookup"><span data-stu-id="c981d-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="c981d-123">각 Azure Stack Hub에서 테넌트 구독에 대한 권한이 있는 Azure AD(Azure Active Directory) 서비스 사용자입니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="c981d-124">Azure Stack Azure Active Directory가 다른 Azure AD 테넌트에 대해 배포되는 경우 두 개의 서비스 사용자를 만들어야 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="c981d-125">Azure Stack Hub에 대한 서비스 사용자를 만드는 방법에 대한 자세한 내용은 [서비스 사용자를 만들어 앱에 Azure Stack Hub 리소스에 대한 액세스 권한 부여](/azure-stack/user/azure-stack-create-service-principals)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="c981d-125">To learn how to create a service principal for Azure Stack Hub, see [Create service principals to give apps access to Azure Stack Hub resources](/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="c981d-126">**각 서비스 사용자의 애플리케이션 ID, 클라이언트 암호 및 테넌트 이름(xxxxx.onmicrosoft.com)을 기록해 둡니다.**</span><span class="sxs-lookup"><span data-stu-id="c981d-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="c981d-127">각 Azure Stack Hub의 Marketplace에 배포된 SQL Server 2016 엔터프라이즈</span><span class="sxs-lookup"><span data-stu-id="c981d-127">SQL Server 2016 Enterprise syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="c981d-128">마켓플레이스 배포에 대한 자세한 내용은 [Marketplace 항목을 Azure Stack Hub에 다운로드](/azure-stack/operator/azure-stack-download-azure-marketplace-item)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="c981d-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
    <span data-ttu-id="c981d-129">**조직에 적절한 SQL 라이선스가 있는지 확인합니다.**</span><span class="sxs-lookup"><span data-stu-id="c981d-129">**Make sure that your organization has the appropriate SQL licenses.**</span></span>
- <span data-ttu-id="c981d-130">로컬 머신에 설치된 [Windows용 Docker](https://docs.docker.com/docker-for-windows/)</span><span class="sxs-lookup"><span data-stu-id="c981d-130">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="c981d-131">Docker 이미지 가져오기</span><span class="sxs-lookup"><span data-stu-id="c981d-131">Get the Docker image</span></span>

<span data-ttu-id="c981d-132">각 배포에 대한 Docker 이미지는 서로 다른 Azure PowerShell 버전 간의 종속성 문제를 제거합니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-132">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="c981d-133">Windows용 Docker에서 Windows 컨테이너를 사용하고 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-133">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="c981d-134">관리자 권한 명령 프롬프트에서 다음 스크립트를 실행하여 배포 스크립트를 통해 Docker 컨테이너를 가져옵니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-134">Run the following script in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a><span data-ttu-id="c981d-135">가용성 그룹 배포</span><span class="sxs-lookup"><span data-stu-id="c981d-135">Deploy the availability group</span></span>

1. <span data-ttu-id="c981d-136">컨테이너 이미지를 성공적으로 끌어온 후 이미지를 시작합니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-136">Once the container image has been successfully pulled, start the image.</span></span>

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. <span data-ttu-id="c981d-137">컨테이너가 시작되면 컨테이너에 관리자 권한 PowerShell 터미널이 제공됩니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-137">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="c981d-138">배포 스크립트로 가져올 디렉터리를 변경합니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-138">Change directories to get to the deployment script.</span></span>

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. <span data-ttu-id="c981d-139">배포를 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-139">Run the deployment.</span></span> <span data-ttu-id="c981d-140">필요한 경우 자격 증명과 리소스 이름을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-140">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="c981d-141">HA는 HA 클러스터가 배포될 Azure Stack Hub를 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-141">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="c981d-142">DR은 DR 클러스터가 배포될 Azure Stack Hub를 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-142">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

      ```powershell
      > .\Deploy-AzureResourceGroup.ps1 `
      -AzureStackApplicationId_HA "applicationIDforHAServicePrincipal" `
      -AzureStackApplicationSercet_HA "clientSecretforHAServicePrincipal" `
      -AADTenantName_HA "hatenantname.onmicrosoft.com" `
      -AzureStackResourceGroup_HA "haresourcegroupname" `
      -AzureStackArmEndpoint_HA "https://management.haazurestack.com" `
      -AzureStackSubscriptionId_HA "haSubscriptionId" `
      -AzureStackApplicationId_DR "applicationIDforDRServicePrincipal" `
      -AzureStackApplicationSercet_DR "ClientSecretforDRServicePrincipal" `
      -AADTenantName_DR "drtenantname.onmicrosoft.com" `
      -AzureStackResourceGroup_DR "drresourcegroupname" `
      -AzureStackArmEndpoint_DR "https://management.drazurestack.com" `
      -AzureStackSubscriptionId_DR "drSubscriptionId"
      ```

4. <span data-ttu-id="c981d-143">NuGet 공급자 설치를 허용하기 위해 `Y`를 입력합니다. 그러면 설치될 API 프로필 "2018-03-01-hybrid" 모듈이 시작됩니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-143">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="c981d-144">리소스 배포가 완료될 때까지 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-144">Wait for resource deployment to complete.</span></span>

6. <span data-ttu-id="c981d-145">DR 리소스 배포가 완료되면 컨테이너를 종료합니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-145">Once DR resource deployment has completed, exit the container.</span></span>

      ```powershell
      exit
      ```

7. <span data-ttu-id="c981d-146">각 Azure Stack Hub 포털의 리소스를 확인하여 배포를 검사합니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-146">Inspect the deployment by viewing the resources in each Azure Stack Hub's portal.</span></span> <span data-ttu-id="c981d-147">HA 환경에서 SQL 인스턴스 중 하나에 연결하고 SSMS(SQL Server Management Studio)를 통해 가용성 그룹을 검사합니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-147">Connect to one of the SQL instances on the HA environment and inspect the Availability Group through SQL Server Management Studio (SSMS).</span></span>

    ![SQL Server 2016 SQL HA](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a><span data-ttu-id="c981d-149">다음 단계</span><span class="sxs-lookup"><span data-stu-id="c981d-149">Next steps</span></span>

- <span data-ttu-id="c981d-150">SQL Server Management Studio를 사용하여 클러스터를 수동으로 장애 조치(failover)합니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-150">Use SQL Server Management Studio to manually fail over the cluster.</span></span> <span data-ttu-id="c981d-151">[Always On 가용성 그룹의 강제 수동 장애 조치(failover) 수행(SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="c981d-151">See [Perform a Forced Manual Failover of an Always On Availability Group (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span></span>
- <span data-ttu-id="c981d-152">하이브리드 클라우드 앱에 대해 자세히 알아봅니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-152">Learn more about hybrid cloud apps.</span></span> <span data-ttu-id="c981d-153">[하이브리드 클라우드 솔루션](https://aka.ms/azsdevtutorials)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="c981d-153">See [Hybrid Cloud Solutions.](https://aka.ms/azsdevtutorials)</span></span>
- <span data-ttu-id="c981d-154">[GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)에서 사용자 고유의 데이터를 사용하거나 코드를 이 샘플로 수정합니다.</span><span class="sxs-lookup"><span data-stu-id="c981d-154">Use your own data or modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>
