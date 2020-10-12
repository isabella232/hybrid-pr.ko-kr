---
title: Azure 및 Azure Stack Hub에 고가용성 MongoDB 솔루션 배포
description: Azure 및 Azure Stack Hub에 고가용성 MongoDB 솔루션을 배포하는 방법에 대해 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: def9abaa2a7231648f11453f66119399be015a4d
ms.sourcegitcommit: 485a1f97fa1579364e2be1755cadfc5ea89db50e
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/08/2020
ms.locfileid: "91852510"
---
# <a name="deploy-a-highly-available-mongodb-solution-to-azure-and-azure-stack-hub"></a><span data-ttu-id="7ae7a-103">Azure 및 Azure Stack Hub에 고가용성 MongoDB 솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="7ae7a-103">Deploy a highly available MongoDB solution to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="7ae7a-104">이 문서에서는 두 개의 Azure Stack Hub 환경에서 DR(재해 복구) 사이트를 사용하여 기본 HA(고가용성) MongoDB 클러스터를 자동으로 배포하는 과정을 안내합니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-104">This article will step you through an automated deployment of a basic highly available (HA) MongoDB cluster with a disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="7ae7a-105">MongoDB 및 고가용성에 대한 자세한 내용은 [복제본 세트 멤버](https://docs.mongodb.com/manual/core/replica-set-members/)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-105">To learn more about MongoDB and high availability, see [Replica Set Members](https://docs.mongodb.com/manual/core/replica-set-members/).</span></span>

<span data-ttu-id="7ae7a-106">이 솔루션에서는 다음을 수행하는 샘플 환경을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-106">In this solution, you'll create a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="7ae7a-107">두 Azure Stack Hub에서 배포를 오케스트레이션합니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="7ae7a-108">Docker를 사용하여 Azure API 프로필의 종속성 문제를 최소화합니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="7ae7a-109">재해 복구 사이트를 사용하여 기본 고가용성 MongoDB 클러스터를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-109">Deploy a basic highly available MongoDB cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="7ae7a-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="7ae7a-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="7ae7a-111">Microsoft Azure Stack Hub는 Azure의 확장입니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="7ae7a-112">Azure Stack Hub는 온-프레미스 환경에 클라우드 컴퓨팅의 민첩성과 혁신을 제공하여 어디서나 하이브리드 앱을 빌드하고 배포할 수 있는 유일한 하이브리드 클라우드를 사용하도록 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="7ae7a-113">[하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 문서는 하이브리드 앱 디자인, 배포 및 운영에 대한 소프트웨어 품질(배치, 확장성, 가용성, 복원력, 관리 효율성 및 보안)의 핵심 요소를 검토합니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="7ae7a-114">디자인 고려 사항은 하이브리드 앱 디자인을 최적화하고 프로덕션 환경에서 문제를 최소화하는 데 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="7ae7a-115">Azure Stack Hub가 있는 MongoDB 아키텍처</span><span class="sxs-lookup"><span data-stu-id="7ae7a-115">Architecture for MongoDB with Azure Stack Hub</span></span>

![Azure Stack Hub의 고가용성 MongoDB 아키텍처](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="7ae7a-117">Azure Stack Hub가 있는 MongoDB에 대한 필수 구성 요소</span><span class="sxs-lookup"><span data-stu-id="7ae7a-117">Prerequisites for MongoDB with Azure Stack Hub</span></span>

- <span data-ttu-id="7ae7a-118">2개의 연결된 Azure Stack Hub 통합 시스템(Azure Stack Hub)</span><span class="sxs-lookup"><span data-stu-id="7ae7a-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="7ae7a-119">이 배포는 ASDK(Azure Stack Development Kit)에서 작동하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="7ae7a-120">Azure Stack Hub에 대한 자세한 내용은 [Azure Stack Hub란?](https://azure.microsoft.com/products/azure-stack/hub/)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-120">To learn more about Azure Stack Hub, see [What is Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)</span></span>
  - <span data-ttu-id="7ae7a-121">각 Azure Stack Hub에 대한 테넌트 구독</span><span class="sxs-lookup"><span data-stu-id="7ae7a-121">A tenant subscription on each Azure Stack Hub.</span></span> 
  - <span data-ttu-id="7ae7a-122">**각 Azure Stack Hub에 대한 각 구독 ID와 Azure Resource Manager 엔드포인트를 기록해 둡니다.**</span><span class="sxs-lookup"><span data-stu-id="7ae7a-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="7ae7a-123">각 Azure Stack Hub에서 테넌트 구독에 대한 권한이 있는 Azure AD(Azure Active Directory) 서비스 사용자입니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="7ae7a-124">Azure Stack Azure Active Directory가 다른 Azure AD 테넌트에 대해 배포되는 경우 두 개의 서비스 사용자를 만들어야 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="7ae7a-125">Azure Stack Hub에 대한 서비스 주체를 만드는 방법에 대한 자세한 내용은 [앱 ID를 사용하여 Azure Stack Hub 리소스에 액세스](/azure-stack/user/azure-stack-create-service-principals)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-125">To learn how to create a service principal for Azure Stack Hub, see [Use an app identity to access Azure Stack Hub resources](/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="7ae7a-126">**각 서비스 사용자의 애플리케이션 ID, 클라이언트 암호 및 테넌트 이름(xxxxx.onmicrosoft.com)을 기록해 둡니다.**</span><span class="sxs-lookup"><span data-stu-id="7ae7a-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="7ae7a-127">각 Azure Stack Hub의 Marketplace에 배포된 Ubuntu 16.04</span><span class="sxs-lookup"><span data-stu-id="7ae7a-127">Ubuntu 16.04 syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="7ae7a-128">마켓플레이스 배포에 대한 자세한 내용은 [Marketplace 항목을 Azure Stack Hub에 다운로드](/azure-stack/operator/azure-stack-download-azure-marketplace-item)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
- <span data-ttu-id="7ae7a-129">로컬 머신에 설치된 [Windows용 Docker](https://docs.docker.com/docker-for-windows/)</span><span class="sxs-lookup"><span data-stu-id="7ae7a-129">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="7ae7a-130">Docker 이미지 가져오기</span><span class="sxs-lookup"><span data-stu-id="7ae7a-130">Get the Docker image</span></span>

<span data-ttu-id="7ae7a-131">각 배포에 대한 Docker 이미지는 서로 다른 Azure PowerShell 버전 간의 종속성 문제를 제거합니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-131">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="7ae7a-132">Windows용 Docker에서 Windows 컨테이너를 사용하고 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-132">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="7ae7a-133">관리자 권한 명령 프롬프트에서 다음 명령을 실행하여 배포 스크립트를 통해 Docker 컨테이너를 가져옵니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-133">Run the following command in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a><span data-ttu-id="7ae7a-134">클러스터 배포</span><span class="sxs-lookup"><span data-stu-id="7ae7a-134">Deploy the clusters</span></span>

1. <span data-ttu-id="7ae7a-135">컨테이너 이미지를 성공적으로 끌어온 후 이미지를 시작합니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-135">Once the container image has been successfully pulled, start the image.</span></span>

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. <span data-ttu-id="7ae7a-136">컨테이너가 시작되면 컨테이너에 관리자 권한 PowerShell 터미널이 제공됩니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-136">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="7ae7a-137">배포 스크립트로 가져올 디렉터리를 변경합니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-137">Change directories to get to the deployment script.</span></span>

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. <span data-ttu-id="7ae7a-138">배포를 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-138">Run the deployment.</span></span> <span data-ttu-id="7ae7a-139">필요한 경우 자격 증명과 리소스 이름을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-139">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="7ae7a-140">HA는 HA 클러스터가 배포될 Azure Stack Hub를 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-140">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="7ae7a-141">DR은 DR 클러스터가 배포될 Azure Stack Hub를 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-141">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

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

4. <span data-ttu-id="7ae7a-142">NuGet 공급자 설치를 허용하기 위해 `Y`를 입력합니다. 그러면 설치될 API 프로필 "2018-03-01-hybrid" 모듈이 시작됩니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-142">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="7ae7a-143">HA 리소스가 먼저 배포됩니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-143">The HA resources will deploy first.</span></span> <span data-ttu-id="7ae7a-144">배포를 모니터링하고 배포가 끝나기를 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-144">Monitor the deployment and wait for it to finish.</span></span> <span data-ttu-id="7ae7a-145">HA 배포가 완료되었다는 메시지가 표시되면 HA Azure Stack Hub의 포털에서 배포된 리소스를 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-145">Once you have the message stating that the HA deployment is finished, you can check the HA Azure Stack Hub's portal to see the resources deployed.</span></span>

6. <span data-ttu-id="7ae7a-146">DR 리소스의 배포를 계속 진행하고 DR Azure Stack Hub의 점프 상자를 사용하여 클러스터와 상호 작용하도록 할지 결정합니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-146">Continue with the deployment of DR resources and decide if you'd like to enable a jump box on the DR Azure Stack Hub to interact with the cluster.</span></span>

7. <span data-ttu-id="7ae7a-147">DR 리소스 배포가 완료될 때까지 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-147">Wait for DR resource deployment to finish.</span></span>

8. <span data-ttu-id="7ae7a-148">DR 리소스 배포가 완료되면 컨테이너를 종료합니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-148">Once DR resource deployment has finished, exit the container.</span></span>

  ```powershell
  exit
  ```

## <a name="next-steps"></a><span data-ttu-id="7ae7a-149">다음 단계</span><span class="sxs-lookup"><span data-stu-id="7ae7a-149">Next steps</span></span>

- <span data-ttu-id="7ae7a-150">DR Azure Stack Hub에서 점프 상자 VM을 사용하도록 설정한 경우 SSH를 통해 연결하고 mongo CLI를 설치하여 MongoDB 클러스터와 상호 작용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-150">If you enabled the jump box VM on the DR Azure Stack Hub, you can connect via SSH and interact with the MongoDB cluster by installing the mongo CLI.</span></span> <span data-ttu-id="7ae7a-151">MongoDB와 상호 작용하는 방법에 대한 자세한 내용은 [mongo Shell](https://docs.mongodb.com/manual/mongo/)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-151">To learn more about interacting with MongoDB, see [The mongo Shell](https://docs.mongodb.com/manual/mongo/).</span></span>
- <span data-ttu-id="7ae7a-152">하이브리드 클라우드 앱에 대해 자세히 알아보려면 [하이브리드 클라우드 솔루션](/azure-stack/user/)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-152">To learn more about hybrid cloud apps, see [Hybrid Cloud Solutions.](/azure-stack/user/)</span></span>
- <span data-ttu-id="7ae7a-153">[GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)에서 코드를 이 샘플로 수정합니다.</span><span class="sxs-lookup"><span data-stu-id="7ae7a-153">Modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>