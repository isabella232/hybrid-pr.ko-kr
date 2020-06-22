---
title: Azure 및 Azure Stack Hub에 항상 사용 가능한 MongoDB 솔루션 배포
description: Azure 및 Azure Stack Hub에 항상 사용 가능한 MongoDB 솔루션을 배포 하는 방법을 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: b34ba7c10ff5f658d645923ae8b6de2fb2607ccb
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911449"
---
# <a name="deploy-a-highly-available-mongodb-solution-to-azure-and-azure-stack-hub"></a><span data-ttu-id="c91ad-103">Azure 및 Azure Stack Hub에 항상 사용 가능한 MongoDB 솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="c91ad-103">Deploy a highly available MongoDB solution to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="c91ad-104">이 문서에서는 두 개의 Azure Stack 허브 환경에서 DR (재해 복구) 사이트를 사용 하 여 기본 HA (고가용성) MongoDB 클러스터를 자동으로 배포 하는 과정을 안내 합니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-104">This article will step you through an automated deployment of a basic highly available (HA) MongoDB cluster with a disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="c91ad-105">MongoDB 및 고가용성에 대 한 자세한 내용은 [복제본 집합 구성원](https://docs.mongodb.com/manual/core/replica-set-members/)을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="c91ad-105">To learn more about MongoDB and high availability, see [Replica Set Members](https://docs.mongodb.com/manual/core/replica-set-members/).</span></span>

<span data-ttu-id="c91ad-106">이 솔루션에서는 다음을 수행 하는 샘플 환경을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-106">In this solution, you'll create a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="c91ad-107">두 Azure Stack 허브에서 배포를 오케스트레이션 합니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="c91ad-108">Docker를 사용 하 여 Azure API 프로필의 종속성 문제를 최소화 합니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="c91ad-109">재해 복구 사이트를 사용 하 여 기본 고가용성 MongoDB 클러스터를 배포 합니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-109">Deploy a basic highly available MongoDB cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="c91ad-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="c91ad-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="c91ad-111">Microsoft Azure Stack 허브는 Azure의 확장입니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="c91ad-112">Azure Stack 허브는 온-프레미스 환경에 클라우드 컴퓨팅의 민첩성과 혁신을 제공 하 여 어디서 나 하이브리드 앱을 빌드하고 배포할 수 있는 유일한 하이브리드 클라우드를 사용 하도록 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="c91ad-113">[하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 문서에서는 하이브리드 앱을 디자인, 배포 및 운영 하기 위한 소프트웨어 품질 (배치, 확장성, 가용성, 복원 력, 관리 효율성 및 보안)의 핵심 요소을 검토 합니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="c91ad-114">디자인 고려 사항은 하이브리드 앱 디자인을 최적화 하 고 프로덕션 환경에서 문제를 최소화 하는 데 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="c91ad-115">Azure Stack 허브가 있는 MongoDB 아키텍처</span><span class="sxs-lookup"><span data-stu-id="c91ad-115">Architecture for MongoDB with Azure Stack Hub</span></span>

![Azure Stack Hub의 항상 사용 가능한 MongoDB 아키텍처](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="c91ad-117">Azure Stack 허브가 있는 MongoDB에 대 한 필수 구성 요소</span><span class="sxs-lookup"><span data-stu-id="c91ad-117">Prerequisites for MongoDB with Azure Stack Hub</span></span>

- <span data-ttu-id="c91ad-118">2 개의 연결 된 Azure Stack 허브 통합 시스템 (Azure Stack 허브).</span><span class="sxs-lookup"><span data-stu-id="c91ad-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="c91ad-119">이 배포는 Azure Stack Development Kit (ASDK)에서 작동 하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="c91ad-120">Azure Stack Hub에 대해 자세히 알아보려면 [Azure Stack 허브 란?](https://azure.microsoft.com/products/azure-stack/hub/) 을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="c91ad-120">To learn more about Azure Stack Hub, see [What is Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)</span></span>
  - <span data-ttu-id="c91ad-121">각 Azure Stack 허브에 대 한 테 넌 트 구독</span><span class="sxs-lookup"><span data-stu-id="c91ad-121">A tenant subscription on each Azure Stack Hub.</span></span> 
  - <span data-ttu-id="c91ad-122">**각 Azure Stack 허브에 대 한 각 구독 ID와 Azure Resource Manager 끝점을 기록해 둡니다.**</span><span class="sxs-lookup"><span data-stu-id="c91ad-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="c91ad-123">각 Azure Stack 허브에서 테 넌 트 구독에 대 한 권한이 있는 Azure Active Directory (Azure AD) 서비스 사용자입니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="c91ad-124">Azure Stack 허브가 다른 Azure AD 테 넌 트에 대해 배포 되는 경우 두 개의 서비스 주체를 만들어야 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="c91ad-125">Azure Stack Hub에 대 한 서비스 주체를 만드는 방법을 알아보려면 [앱 id를 사용 하 여 Azure Stack 허브 리소스에 액세스](https://docs.microsoft.com/azure-stack/user/azure-stack-create-service-principals)를 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="c91ad-125">To learn how to create a service principal for Azure Stack Hub, see [Use an app identity to access Azure Stack Hub resources](https://docs.microsoft.com/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="c91ad-126">**각 서비스 주체의 응용 프로그램 ID, 클라이언트 암호 및 테 넌 트 이름 (xxxxx.onmicrosoft.com)을 기록해 둡니다.**</span><span class="sxs-lookup"><span data-stu-id="c91ad-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="c91ad-127">Ubuntu 16.04는 각 Azure Stack 허브의 Marketplace에 게시 됩니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-127">Ubuntu 16.04 syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="c91ad-128">Marketplace 배포에 대 한 자세한 내용은 [Azure Stack 허브에 marketplace 항목 다운로드](https://docs.microsoft.com/azure-stack/operator/azure-stack-download-azure-marketplace-item)를 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="c91ad-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](https://docs.microsoft.com/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
- <span data-ttu-id="c91ad-129">로컬 컴퓨터에 설치 된 [Windows용 Docker](https://docs.docker.com/docker-for-windows/)</span><span class="sxs-lookup"><span data-stu-id="c91ad-129">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="c91ad-130">Docker 이미지 가져오기</span><span class="sxs-lookup"><span data-stu-id="c91ad-130">Get the Docker image</span></span>

<span data-ttu-id="c91ad-131">각 배포에 대 한 Docker 이미지는 서로 다른 Azure PowerShell 버전 간의 종속성 문제를 제거 합니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-131">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="c91ad-132">Windows용 Docker Windows 컨테이너를 사용 하 고 있는지 확인 합니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-132">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="c91ad-133">관리자 권한 명령 프롬프트에서 다음 명령을 실행 하 여 배포 스크립트를 사용 하 여 Docker 컨테이너를 가져옵니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-133">Run the following command in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a><span data-ttu-id="c91ad-134">클러스터 배포</span><span class="sxs-lookup"><span data-stu-id="c91ad-134">Deploy the clusters</span></span>

1. <span data-ttu-id="c91ad-135">컨테이너 이미지를 성공적으로 끌어온 후 이미지를 시작 합니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-135">Once the container image has been successfully pulled, start the image.</span></span>

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. <span data-ttu-id="c91ad-136">컨테이너가 시작 되 면 컨테이너에 관리자 권한 PowerShell 터미널이 제공 됩니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-136">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="c91ad-137">배포 스크립트로 가져올 디렉터리를 변경 합니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-137">Change directories to get to the deployment script.</span></span>

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. <span data-ttu-id="c91ad-138">배포를 실행 합니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-138">Run the deployment.</span></span> <span data-ttu-id="c91ad-139">필요한 경우 자격 증명과 리소스 이름을 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-139">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="c91ad-140">HA는 HA 클러스터가 배포 될 Azure Stack 허브를 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-140">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="c91ad-141">DR은 DR 클러스터가 배포 될 Azure Stack 허브를 참조 합니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-141">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

    ```powershell
    .\Deploy-AzureResourceGroup.ps1 `
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

4. <span data-ttu-id="c91ad-142">`Y`NuGet 공급자 설치를 허용 하려면를 입력 합니다. 그러면 설치 될 API 프로필 "2018-03-01-하이브리드" 모듈이 시작 됩니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-142">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="c91ad-143">HA 리소스가 먼저 배포 됩니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-143">The HA resources will deploy first.</span></span> <span data-ttu-id="c91ad-144">배포를 모니터링 하 고 완료 될 때까지 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-144">Monitor the deployment and wait for it to finish.</span></span> <span data-ttu-id="c91ad-145">HA 배포가 완료 되었다는 메시지가 표시 되 면 HA Azure Stack 허브의 포털에서 배포 된 리소스를 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-145">Once you have the message stating that the HA deployment is finished, you can check the HA Azure Stack Hub's portal to see the resources deployed.</span></span>

6. <span data-ttu-id="c91ad-146">DR 리소스의 배포를 계속 진행 하 고 DR Azure Stack 허브의 점프 상자를 사용 하 여 클러스터와 상호 작용 하도록 할지 결정 합니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-146">Continue with the deployment of DR resources and decide if you'd like to enable a jump box on the DR Azure Stack Hub to interact with the cluster.</span></span>

7. <span data-ttu-id="c91ad-147">DR 리소스 배포가 완료 될 때까지 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-147">Wait for DR resource deployment to finish.</span></span>

8. <span data-ttu-id="c91ad-148">DR 리소스 배포가 완료 되 면 컨테이너를 종료 합니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-148">Once DR resource deployment has finished, exit the container.</span></span>

  ```powershell
  exit
  ```

## <a name="next-steps"></a><span data-ttu-id="c91ad-149">다음 단계</span><span class="sxs-lookup"><span data-stu-id="c91ad-149">Next steps</span></span>

- <span data-ttu-id="c91ad-150">DR Azure Stack 허브에서 점프 상자 VM을 사용 하도록 설정한 경우, SSH를 통해 연결 하 고 mongo CLI를 설치 하 여 MongoDB 클러스터와 상호 작용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-150">If you enabled the jump box VM on the DR Azure Stack Hub, you can connect via SSH and interact with the MongoDB cluster by installing the mongo CLI.</span></span> <span data-ttu-id="c91ad-151">MongoDB와 상호 작용 하는 방법에 대해 자세히 알아보려면 [mongo Shell](https://docs.mongodb.com/manual/mongo/)을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="c91ad-151">To learn more about interacting with MongoDB, see [The mongo Shell](https://docs.mongodb.com/manual/mongo/).</span></span>
- <span data-ttu-id="c91ad-152">하이브리드 클라우드 앱에 대해 자세히 알아보려면 [하이브리드 클라우드 솔루션](https://aka.ms/azsdevtutorials) 을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="c91ad-152">To learn more about hybrid cloud apps, see [Hybrid Cloud Solutions.](https://aka.ms/azsdevtutorials)</span></span>
- <span data-ttu-id="c91ad-153">[GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)에서 코드를이 샘플로 수정 합니다.</span><span class="sxs-lookup"><span data-stu-id="c91ad-153">Modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>
