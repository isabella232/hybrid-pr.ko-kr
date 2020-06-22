---
title: 클라우드 간 크기를 조정 하는 온-프레미스 데이터를 사용 하 여 하이브리드 앱 배포
description: Azure 및 Azure Stack Hub를 사용 하 여 온-프레미스 데이터를 사용 하는 앱을 배포 하 고 클라우드 간 크기를 조정 하는 방법을 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 75289eae902c5363862e345bdedb97cbcee0476e
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910889"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a><span data-ttu-id="87cc6-103">클라우드 간 크기를 조정 하는 온-프레미스 데이터를 사용 하 여 하이브리드 앱 배포</span><span class="sxs-lookup"><span data-stu-id="87cc6-103">Deploy hybrid app with on-premises data that scales cross-cloud</span></span>

<span data-ttu-id="87cc6-104">이 솔루션 가이드에서는 Azure와 Azure Stack Hub 모두에 걸친 하이브리드 앱을 배포 하 고 단일 온-프레미스 데이터 원본을 사용 하는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-104">This solution guide shows you how to deploy a hybrid app that spans both Azure and Azure Stack Hub and uses a single on-premises data source.</span></span>

<span data-ttu-id="87cc6-105">하이브리드 클라우드 솔루션을 사용 하 여 사설 클라우드의 규정 준수 이점을 공용 클라우드의 확장성과 결합할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-105">By using a hybrid cloud solution, you can combine the compliance benefits of a private cloud with the scalability of the public cloud.</span></span> <span data-ttu-id="87cc6-106">개발자는 Microsoft 개발자 에코 시스템을 활용 하 여 클라우드 및 온-프레미스 환경에 기술을 적용할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-106">Your developers can also take advantage of the Microsoft developer ecosystem and apply their skills to the cloud and on-premises environments.</span></span>

## <a name="overview-and-assumptions"></a><span data-ttu-id="87cc6-107">개요 및 가정</span><span class="sxs-lookup"><span data-stu-id="87cc6-107">Overview and assumptions</span></span>

<span data-ttu-id="87cc6-108">이 자습서에 따라 개발자가 동일한 웹 앱을 공용 클라우드와 사설 클라우드에 배포할 수 있는 워크플로를 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-108">Follow this tutorial to set up a workflow that lets developers deploy an identical web app to a public cloud and a private cloud.</span></span> <span data-ttu-id="87cc6-109">이 앱은 사설 클라우드에서 호스트 되는 인터넷 라우팅할 수 없는 네트워크에 액세스할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-109">This app can access a non-internet routable network hosted on the private cloud.</span></span> <span data-ttu-id="87cc6-110">이러한 웹 앱은 모니터링 되 고 트래픽이 급증 하는 경우 프로그램은 트래픽을 공용 클라우드로 리디렉션하도록 DNS 레코드를 수정 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-110">These web apps are monitored and when there's a spike in traffic, a program modifies the DNS records to redirect traffic to the public cloud.</span></span> <span data-ttu-id="87cc6-111">급증 이전 수준으로 트래픽이 떨어지면 트래픽이 사설 클라우드로 다시 라우팅됩니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-111">When traffic drops to the level before the spike, traffic is routed back to the private cloud.</span></span>

<span data-ttu-id="87cc6-112">이 자습서에서 다루는 작업은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-112">This tutorial covers the following tasks:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="87cc6-113">하이브리드 연결 SQL Server 데이터베이스 서버를 배포 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-113">Deploy a hybrid-connected SQL Server database server.</span></span>
> - <span data-ttu-id="87cc6-114">글로벌 Azure의 웹 앱을 하이브리드 네트워크에 연결 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-114">Connect a web app in global Azure to a hybrid network.</span></span>
> - <span data-ttu-id="87cc6-115">클라우드 간 크기 조정에 대해 DNS를 구성 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-115">Configure DNS for cross-cloud scaling.</span></span>
> - <span data-ttu-id="87cc6-116">클라우드 간 크기 조정에 대 한 SSL 인증서를 구성 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-116">Configure SSL certificates for cross-cloud scaling.</span></span>
> - <span data-ttu-id="87cc6-117">웹 앱을 구성 하 고 배포 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-117">Configure and deploy the web app.</span></span>
> - <span data-ttu-id="87cc6-118">Traffic Manager 프로필을 만들고 클라우드 간 크기 조정을 위해 구성 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-118">Create a Traffic Manager profile and configure it for cross-cloud scaling.</span></span>
> - <span data-ttu-id="87cc6-119">증가 하는 트래픽에 대 한 모니터링 및 경고를 Application Insights 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-119">Set up Application Insights monitoring and alerting for increased traffic.</span></span>
> - <span data-ttu-id="87cc6-120">글로벌 Azure와 Azure Stack Hub 간에 자동 트래픽 전환을 구성 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-120">Configure automatic traffic switching between global Azure and Azure Stack Hub.</span></span>

> [!Tip]  
> <span data-ttu-id="87cc6-121">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="87cc6-121">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="87cc6-122">Microsoft Azure Stack 허브는 Azure의 확장입니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-122">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="87cc6-123">Azure Stack 허브는 온-프레미스 환경에 클라우드 컴퓨팅의 민첩성과 혁신을 제공 하 여 어디서 나 하이브리드 앱을 빌드하고 배포할 수 있는 유일한 하이브리드 클라우드를 사용 하도록 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-123">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="87cc6-124">[하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 문서에서는 하이브리드 앱을 디자인, 배포 및 운영 하기 위한 소프트웨어 품질 (배치, 확장성, 가용성, 복원 력, 관리 효율성 및 보안)의 핵심 요소을 검토 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-124">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="87cc6-125">디자인 고려 사항은 하이브리드 앱 디자인을 최적화 하 고 프로덕션 환경에서 문제를 최소화 하는 데 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-125">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

### <a name="assumptions"></a><span data-ttu-id="87cc6-126">가정</span><span class="sxs-lookup"><span data-stu-id="87cc6-126">Assumptions</span></span>

<span data-ttu-id="87cc6-127">이 자습서에서는 사용자에 게 글로벌 Azure 및 Azure Stack Hub에 대 한 기본 지식이 있다고 가정 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-127">This tutorial assumes that you have a basic knowledge of global Azure and Azure Stack Hub.</span></span> <span data-ttu-id="87cc6-128">자습서를 시작 하기 전에 자세히 알아보려면 다음 문서를 검토 하세요.</span><span class="sxs-lookup"><span data-stu-id="87cc6-128">If you want to learn more before starting the tutorial, review these articles:</span></span>

- [<span data-ttu-id="87cc6-129">Azure 소개</span><span class="sxs-lookup"><span data-stu-id="87cc6-129">Introduction to Azure</span></span>](https://azure.microsoft.com/overview/what-is-azure/)
- [<span data-ttu-id="87cc6-130">Azure Stack 허브 주요 개념</span><span class="sxs-lookup"><span data-stu-id="87cc6-130">Azure Stack Hub Key Concepts</span></span>](/azure-stack/operator/azure-stack-overview.md)

<span data-ttu-id="87cc6-131">또한이 자습서에서는 Azure 구독이 있다고 가정 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-131">This tutorial also assumes that you have an Azure subscription.</span></span> <span data-ttu-id="87cc6-132">구독이 없는 경우 시작 하기 전에 [무료 계정을 만듭니다](https://azure.microsoft.com/free/) .</span><span class="sxs-lookup"><span data-stu-id="87cc6-132">If you don't have a subscription, [create a free account](https://azure.microsoft.com/free/) before you begin.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="87cc6-133">필수 조건</span><span class="sxs-lookup"><span data-stu-id="87cc6-133">Prerequisites</span></span>

<span data-ttu-id="87cc6-134">이 솔루션을 시작 하기 전에 다음 요구 사항을 충족 하는지 확인 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-134">Before you start this solution, make sure you meet the following requirements:</span></span>

- <span data-ttu-id="87cc6-135">Azure Stack Development Kit (ASDK) 또는 Azure Stack 허브 통합 시스템의 구독입니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-135">An Azure Stack Development Kit (ASDK) or a subscription on an Azure Stack Hub Integrated System.</span></span> <span data-ttu-id="87cc6-136">ASDK을 배포 하려면 [설치 관리자를 사용 하 여 Asdk 배포](/azure-stack/asdk/asdk-install.md)의 지침을 따르세요.</span><span class="sxs-lookup"><span data-stu-id="87cc6-136">To deploy the ASDK, follow the instructions in [Deploy the ASDK using the installer](/azure-stack/asdk/asdk-install.md).</span></span>
- <span data-ttu-id="87cc6-137">Azure Stack Hub 설치에는 다음이 설치 되어 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-137">Your Azure Stack Hub installation should have the following installed:</span></span>
  - <span data-ttu-id="87cc6-138">Azure App Service입니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-138">The Azure App Service.</span></span> <span data-ttu-id="87cc6-139">Azure Stack 허브 운영자에 게 작업 하 여 환경에서 Azure App Service를 배포 하 고 구성 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-139">Work with your Azure Stack Hub Operator to deploy and configure the Azure App Service on your environment.</span></span> <span data-ttu-id="87cc6-140">이 자습서에서는 App Service에 사용 가능한 전용 작업자 역할이 하나 이상 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-140">This tutorial requires the App Service to have at least one (1) available dedicated worker role.</span></span>
  - <span data-ttu-id="87cc6-141">Windows Server 2016 이미지입니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-141">A Windows Server 2016 image.</span></span>
  - <span data-ttu-id="87cc6-142">Microsoft SQL Server 이미지를 포함 하는 Windows Server 2016</span><span class="sxs-lookup"><span data-stu-id="87cc6-142">A Windows Server 2016 with a Microsoft SQL Server image.</span></span>
  - <span data-ttu-id="87cc6-143">적절 한 계획 및 제안.</span><span class="sxs-lookup"><span data-stu-id="87cc6-143">The appropriate plans and offers.</span></span>
  - <span data-ttu-id="87cc6-144">웹 앱에 대 한 도메인 이름입니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-144">A domain name for your web app.</span></span> <span data-ttu-id="87cc6-145">도메인 이름이 없는 경우 GoDaddy, Bluehost 및 InMotion 같은 도메인 공급자에서 구매할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-145">If you don't have a domain name, you can buy one from a domain provider like GoDaddy, Bluehost, and InMotion.</span></span>
- <span data-ttu-id="87cc6-146">도메인에 대 한 SSL 인증서는 도메인에 대 한 인증서입니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-146">An SSL certificate for your domain from a trusted certificate authority like LetsEncrypt.</span></span>
- <span data-ttu-id="87cc6-147">SQL Server 데이터베이스와 통신 하 고 Application Insights을 지 원하는 웹 앱입니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-147">A web app that communicates with a SQL Server database and supports Application Insights.</span></span> <span data-ttu-id="87cc6-148">GitHub에서 [dotnetcore-sqldb 자습서](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) 샘플 앱을 다운로드할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-148">You can download the [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) sample app from GitHub.</span></span>
- <span data-ttu-id="87cc6-149">Azure 가상 네트워크와 Azure Stack 허브 가상 네트워크 간의 하이브리드 네트워크</span><span class="sxs-lookup"><span data-stu-id="87cc6-149">A hybrid network between an Azure virtual network and Azure Stack Hub virtual network.</span></span> <span data-ttu-id="87cc6-150">자세한 지침은 [Azure 및 Azure Stack Hub를 사용 하 여 하이브리드 클라우드 연결 구성](solution-deployment-guide-connectivity.md)을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="87cc6-150">For detailed instructions, see [Configure hybrid cloud connectivity with Azure and Azure Stack Hub](solution-deployment-guide-connectivity.md).</span></span>

- <span data-ttu-id="87cc6-151">Azure Stack 허브에 개인 빌드 에이전트가 있는 CI/CD (하이브리드 연속 통합/연속 배포) 파이프라인</span><span class="sxs-lookup"><span data-stu-id="87cc6-151">A hybrid continuous integration/continuous deployment (CI/CD) pipeline with a private build agent on Azure Stack Hub.</span></span> <span data-ttu-id="87cc6-152">자세한 지침은 [Azure 및 Azure Stack Hub 앱을 사용 하 여 하이브리드 클라우드 Id 구성](solution-deployment-guide-identity.md)을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="87cc6-152">For detailed instructions, see [Configure hybrid cloud identity with Azure and Azure Stack Hub apps](solution-deployment-guide-identity.md).</span></span>

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a><span data-ttu-id="87cc6-153">하이브리드 연결 SQL Server 데이터베이스 서버 배포</span><span class="sxs-lookup"><span data-stu-id="87cc6-153">Deploy a hybrid-connected SQL Server database server</span></span>

1. <span data-ttu-id="87cc6-154">Azure Stack Hub 사용자 포털에 로그인 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-154">Sign to the Azure Stack Hub user portal.</span></span>

2. <span data-ttu-id="87cc6-155">**대시보드에서** **Marketplace**를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-155">On the **Dashboard**, select **Marketplace**.</span></span>

    ![Azure Stack 허브 마켓플레이스](media/solution-deployment-guide-hybrid/image1.png)

3. <span data-ttu-id="87cc6-157">**Marketplace**에서 **Compute**를 선택 하 고 **자세히**를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-157">In **Marketplace**, select **Compute**, and then choose **More**.</span></span> <span data-ttu-id="87cc6-158">**자세히**아래에서 **무료 SQL Server 라이선스: SQL Server 2017 Developer on Windows Server** 이미지를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-158">Under **More**, select the **Free SQL Server License: SQL Server 2017 Developer on Windows Server** image.</span></span>

    ![Azure Stack Hub 사용자 포털에서 가상 머신 이미지를 선택 합니다.](media/solution-deployment-guide-hybrid/image2.png)

4. <span data-ttu-id="87cc6-160">**무료 SQL Server 라이선스: SQL Server 2017 Developer On Windows Server**에서 **만들기**를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-160">On **Free SQL Server License: SQL Server 2017 Developer on Windows Server**, select **Create**.</span></span>

5. <span data-ttu-id="87cc6-161">**기본 설정 > 기본 설정 구성**에서 VM (가상 머신) **이름** , SQL Server sa의 **사용자 이름** 및 sa **암호** 를 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-161">On **Basics > Configure basic settings**, provide a **Name** for the virtual machine (VM), a **User name** for the SQL Server SA, and a **Password** for the SA.</span></span>  <span data-ttu-id="87cc6-162">**구독** 드롭다운 목록에서 배포 하려는 구독을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-162">From the **Subscription** drop-down list, select the subscription that you're deploying to.</span></span> <span data-ttu-id="87cc6-163">**리소스 그룹**의 경우 **기존 선택** 을 사용 하 여 Azure Stack 허브 웹 앱과 동일한 리소스 그룹에 VM을 배치 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-163">For **Resource group**, use **Choose existing** and put the VM in the same resource group as your Azure Stack Hub web app.</span></span>

    ![Azure Stack Hub 사용자 포털에서 VM에 대 한 기본 설정 구성](media/solution-deployment-guide-hybrid/image3.png)

6. <span data-ttu-id="87cc6-165">**크기**에서 VM의 크기를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-165">Under **Size**, pick a size for your VM.</span></span> <span data-ttu-id="87cc6-166">이 자습서에서는 A2_Standard 또는 DS2_V2_Standard를 권장 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-166">For this tutorial, we recommend A2_Standard or a DS2_V2_Standard.</span></span>

7. <span data-ttu-id="87cc6-167">**설정 > 옵션 기능 구성**에서 다음 설정을 구성 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-167">Under **Settings > Configure optional features**, configure the following settings:</span></span>

   - <span data-ttu-id="87cc6-168">**저장소 계정**: 필요한 경우 새 계정을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-168">**Storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="87cc6-169">**가상 네트워크**:</span><span class="sxs-lookup"><span data-stu-id="87cc6-169">**Virtual network**:</span></span>

     > [!Important]  
     > <span data-ttu-id="87cc6-170">SQL Server VM VPN 게이트웨이와 동일한 가상 네트워크에 배포 되었는지 확인 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-170">Make sure your SQL Server VM is deployed on the same  virtual network as the VPN gateways.</span></span>

   - <span data-ttu-id="87cc6-171">**공용 IP 주소**: 기본 설정을 사용 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-171">**Public IP address**: Use the default settings.</span></span>
   - <span data-ttu-id="87cc6-172">**네트워크 보안 그룹**: (nsg).</span><span class="sxs-lookup"><span data-stu-id="87cc6-172">**Network security group**: (NSG).</span></span> <span data-ttu-id="87cc6-173">새 NSG를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-173">Create a new NSG.</span></span>
   - <span data-ttu-id="87cc6-174">**확장 및 모니터링**: 기본 설정을 유지 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-174">**Extensions and Monitoring**: Keep the default settings.</span></span>
   - <span data-ttu-id="87cc6-175">**진단 저장소 계정**: 필요한 경우 새 계정을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-175">**Diagnostics storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="87cc6-176">**확인** 을 선택 하 여 구성을 저장 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-176">Select **OK** to save your configuration.</span></span>

     ![Azure Stack Hub 사용자 포털에서 선택적 VM 기능 구성](media/solution-deployment-guide-hybrid/image4.png)

8. <span data-ttu-id="87cc6-178">**SQL Server 설정**에서 다음 설정을 구성 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-178">Under **SQL Server settings**, configure the following settings:</span></span>

   - <span data-ttu-id="87cc6-179">**SQL 연결**의 경우 **공용 (인터넷)** 을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-179">For **SQL connectivity**, select **Public (Internet)**.</span></span>
   - <span data-ttu-id="87cc6-180">**포트**의 경우 기본값 **1433**을 유지 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-180">For **Port**, keep the default, **1433**.</span></span>
   - <span data-ttu-id="87cc6-181">**SQL 인증**에 대해 **사용**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-181">For **SQL authentication**, select **Enable**.</span></span>

     > [!Note]  
     > <span data-ttu-id="87cc6-182">SQL 인증을 사용 하도록 설정 하면 **기본**설정에서 구성한 "sqladmin" 정보로 자동으로 채워집니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-182">When you enable SQL authentication, it should auto-populate with the "SQLAdmin" information that you configured in **Basics**.</span></span>

   - <span data-ttu-id="87cc6-183">나머지 설정에 대해서는 기본값을 유지 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-183">For the rest of the settings, keep the defaults.</span></span> <span data-ttu-id="87cc6-184">**확인**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-184">Select **OK**.</span></span>

     ![Azure Stack 허브 사용자 포털에서 SQL Server 설정 구성](media/solution-deployment-guide-hybrid/image5.png)

9. <span data-ttu-id="87cc6-186">**요약**에서 VM 구성을 검토 한 다음 **확인** 을 선택 하 여 배포를 시작 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-186">On **Summary**, review the VM configuration and then select **OK** to start the deployment.</span></span>

    ![Azure Stack 허브 사용자 포털의 구성 요약](media/solution-deployment-guide-hybrid/image6.png)

10. <span data-ttu-id="87cc6-188">새 VM을 만드는 데 다소 시간이 걸립니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-188">It takes some time to create the new VM.</span></span> <span data-ttu-id="87cc6-189">**가상 컴퓨터**에서 VM의 상태를 볼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-189">You can view the STATUS of your VMs in **Virtual machines**.</span></span>

    ![Azure Stack 허브 사용자 포털의 가상 컴퓨터 상태](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a><span data-ttu-id="87cc6-191">Azure에서 웹 앱 만들기 및 Azure Stack 허브</span><span class="sxs-lookup"><span data-stu-id="87cc6-191">Create web apps in Azure and Azure Stack Hub</span></span>

<span data-ttu-id="87cc6-192">Azure App Service는 웹 앱의 실행 및 관리를 간소화 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-192">The Azure App Service simplifies running and managing a web app.</span></span> <span data-ttu-id="87cc6-193">Azure Stack 허브가 Azure와 일치 하기 때문에 App Service를 두 환경에서 모두 실행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-193">Because Azure Stack Hub is consistent with Azure,  the App Service can run in both environments.</span></span> <span data-ttu-id="87cc6-194">App Service를 사용 하 여 앱을 호스팅합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-194">You'll use the App Service to host your app.</span></span>

### <a name="create-web-apps"></a><span data-ttu-id="87cc6-195">웹앱 만들기</span><span class="sxs-lookup"><span data-stu-id="87cc6-195">Create web apps</span></span>

1. <span data-ttu-id="87cc6-196">Azure에서 [App Service 계획 관리](https://docs.microsoft.com/azure/app-service/app-service-plan-manage#create-an-app-service-plan)의 지침에 따라 azure에서 웹 앱을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-196">Create a web app in Azure by following the instructions in [Manage an App Service plan in Azure](https://docs.microsoft.com/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span></span> <span data-ttu-id="87cc6-197">웹 앱을 하이브리드 네트워크와 동일한 구독 및 리소스 그룹에 배치 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-197">Make sure you put the web app in the same subscription and resource group as your hybrid network.</span></span>

2. <span data-ttu-id="87cc6-198">Azure Stack 허브에서 이전 단계 (1)를 반복 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-198">Repeat the previous step (1) in Azure Stack Hub.</span></span>

### <a name="add-route-for-azure-stack-hub"></a><span data-ttu-id="87cc6-199">Azure Stack 허브에 대 한 경로 추가</span><span class="sxs-lookup"><span data-stu-id="87cc6-199">Add route for Azure Stack Hub</span></span>

<span data-ttu-id="87cc6-200">사용자가 앱에 액세스할 수 있도록 Azure Stack 허브의 App Service는 공용 인터넷에서 라우팅할 수 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-200">The App Service on Azure Stack Hub must be routable from the public internet to let users access your app.</span></span> <span data-ttu-id="87cc6-201">Azure Stack 허브가 인터넷에서 액세스할 수 있는 경우 Azure Stack 허브 웹 앱에 대 한 공용 IP 주소 또는 URL을 기록해 둡니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-201">If your Azure Stack Hub is accessible from the internet, make a note of the public-facing IP address or URL for the Azure Stack Hub web app.</span></span>

<span data-ttu-id="87cc6-202">ASDK를 사용 하는 경우 가상 환경 외부 App Service를 노출 하도록 [정적 NAT 매핑을 구성할](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-202">If you're using an ASDK, you can [configure a static NAT mapping](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) to expose App Service outside the virtual environment.</span></span>

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a><span data-ttu-id="87cc6-203">Azure의 웹 앱을 하이브리드 네트워크에 연결</span><span class="sxs-lookup"><span data-stu-id="87cc6-203">Connect a web app in Azure to a hybrid network</span></span>

<span data-ttu-id="87cc6-204">Azure의 웹 프런트 엔드와 Azure Stack Hub의 SQL Server 데이터베이스 간에 연결을 제공 하려면 웹 앱이 Azure와 Azure Stack 허브 간에 하이브리드 네트워크에 연결 되어 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-204">To provide connectivity between the web front end in Azure and the SQL Server database in Azure Stack Hub, the web app must be connected to the hybrid network between Azure and Azure Stack Hub.</span></span> <span data-ttu-id="87cc6-205">연결을 사용 하도록 설정 하려면 다음을 수행 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-205">To enable connectivity, you'll have to:</span></span>

- <span data-ttu-id="87cc6-206">지점 및 사이트 간 연결을 구성 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-206">Configure point-to-site connectivity.</span></span>
- <span data-ttu-id="87cc6-207">웹 앱을 구성 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-207">Configure the web app.</span></span>
- <span data-ttu-id="87cc6-208">Azure Stack 허브에서 로컬 네트워크 게이트웨이를 수정 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-208">Modify the local network gateway in Azure Stack Hub.</span></span>

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a><span data-ttu-id="87cc6-209">지점 및 사이트 간 연결에 대 한 Azure 가상 네트워크 구성</span><span class="sxs-lookup"><span data-stu-id="87cc6-209">Configure the Azure virtual network for point-to-site connectivity</span></span>

<span data-ttu-id="87cc6-210">하이브리드 네트워크의 Azure 쪽에 있는 가상 네트워크 게이트웨이는 지점 및 사이트 간 연결이 Azure App Service와 통합 될 수 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-210">The virtual network gateway in the Azure side of the hybrid network must allow point-to-site connections to integrate with Azure App Service.</span></span>

1. <span data-ttu-id="87cc6-211">Azure에서 가상 네트워크 게이트웨이 페이지로 이동 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-211">In Azure, go to the virtual network gateway page.</span></span> <span data-ttu-id="87cc6-212">**설정**아래에서 **지점 및 사이트 간 구성을**선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-212">Under **Settings**, select **Point-to-site configuration**.</span></span>

    ![Azure virtual network 게이트웨이의 지점 및 사이트 간 옵션](media/solution-deployment-guide-hybrid/image8.png)

2. <span data-ttu-id="87cc6-214">지점 및 사이트 간을 구성 하려면 **지금 구성** 을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-214">Select **Configure now** to configure point-to-site.</span></span>

    ![Azure virtual network 게이트웨이에서 시작 지점 및 사이트 간 구성](media/solution-deployment-guide-hybrid/image9.png)

3. <span data-ttu-id="87cc6-216">지점 및 **사이트 간** 구성 페이지에서 **주소 풀**에 사용 하려는 개인 IP 주소 범위를 입력 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-216">On the **Point-to-site** configuration page, enter the private IP address range that you want to use in **Address pool**.</span></span>

   > [!Note]  
   > <span data-ttu-id="87cc6-217">지정한 범위가 글로벌 Azure의 서브넷에서 이미 사용 하는 주소 범위 또는 하이브리드 네트워크의 Azure Stack 허브 구성 요소와 겹치지 않는지 확인 하세요.</span><span class="sxs-lookup"><span data-stu-id="87cc6-217">Make sure that the range you specify doesn't overlap with any of the address ranges already used by subnets in the global Azure or Azure Stack Hub components of the hybrid network.</span></span>

   <span data-ttu-id="87cc6-218">**터널 유형**에서 **IKEv2 VPN**을 선택 취소 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-218">Under **Tunnel Type**, uncheck the **IKEv2 VPN**.</span></span> <span data-ttu-id="87cc6-219">**저장** 을 선택 하 여 지점 및 사이트 간 구성을 완료 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-219">Select **Save** to finish configuring point-to-site.</span></span>

   ![Azure virtual network 게이트웨이의 지점 및 사이트 간 설정](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a><span data-ttu-id="87cc6-221">하이브리드 네트워크와 Azure App Service 앱 통합</span><span class="sxs-lookup"><span data-stu-id="87cc6-221">Integrate the Azure App Service app with the hybrid network</span></span>

1. <span data-ttu-id="87cc6-222">앱을 Azure VNet에 연결 하려면 [게이트웨이 필수 VNet 통합](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration)의 지침을 따르세요.</span><span class="sxs-lookup"><span data-stu-id="87cc6-222">To connect the app to the Azure VNet, follow the instructions in [Gateway required VNet integration](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span></span>

2. <span data-ttu-id="87cc6-223">웹 앱을 호스트 하는 App Service 계획에 대 한 **설정** 으로 이동 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-223">Go to **Settings** for the App Service plan hosting the web app.</span></span> <span data-ttu-id="87cc6-224">**설정**에서 **네트워킹**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-224">In **Settings**, select **Networking**.</span></span>

    ![App Service 계획에 대 한 네트워킹 구성](media/solution-deployment-guide-hybrid/image11.png)

3. <span data-ttu-id="87cc6-226">**VNET 통합**에서 **관리 하려면 여기를 클릭**하세요 .를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-226">In **VNET Integration**, select **Click here to manage**.</span></span>

    ![App Service 계획에 대 한 VNET 통합 관리](media/solution-deployment-guide-hybrid/image12.png)

4. <span data-ttu-id="87cc6-228">구성 하려는 VNET을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-228">Select the VNET that you want to configure.</span></span> <span data-ttu-id="87cc6-229">**VNET에 라우팅되는 IP 주소**에서 Azure vnet, Azure Stack 허브 VNET 및 지점 및 사이트 간 주소 공간에 대 한 ip 주소 범위를 입력 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-229">Under **IP ADDRESSES ROUTED TO VNET**, enter the IP address range for the Azure VNet, the Azure Stack Hub VNet, and the point-to-site address spaces.</span></span> <span data-ttu-id="87cc6-230">**저장** 을 선택 하 여 이러한 설정을 확인 하 고 저장 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-230">Select **Save** to validate and save these settings.</span></span>

    ![Virtual Network 통합에서 라우팅할 IP 주소 범위](media/solution-deployment-guide-hybrid/image13.png)

<span data-ttu-id="87cc6-232">App Service Azure Vnet와 통합 하는 방법에 대 한 자세한 내용은 [azure Virtual Network와 앱 통합](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet)을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="87cc6-232">To learn more about how App Service integrates with Azure VNets, see [Integrate your app with an Azure Virtual Network](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet).</span></span>

### <a name="configure-the-azure-stack-hub-virtual-network"></a><span data-ttu-id="87cc6-233">Azure Stack 허브 가상 네트워크 구성</span><span class="sxs-lookup"><span data-stu-id="87cc6-233">Configure the Azure Stack Hub virtual network</span></span>

<span data-ttu-id="87cc6-234">App Service 지점 및 사이트 간 주소 범위에서 트래픽을 라우팅하도록 Azure Stack 허브 가상 네트워크의 로컬 네트워크 게이트웨이를 구성 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-234">The local network gateway in the Azure Stack Hub virtual network needs to be configured to route traffic from the App Service point-to-site address range.</span></span>

1. <span data-ttu-id="87cc6-235">Azure Stack 허브에서 **로컬 네트워크 게이트웨이로**이동 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-235">In Azure Stack Hub, go to **Local network gateway**.</span></span> <span data-ttu-id="87cc6-236">**설정**에서 **구성**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-236">Under **Settings**, select **Configuration**.</span></span>

    ![Azure Stack 허브 로컬 네트워크 게이트웨이의 게이트웨이 구성 옵션](media/solution-deployment-guide-hybrid/image14.png)

2. <span data-ttu-id="87cc6-238">**주소 공간**에 Azure의 가상 네트워크 게이트웨이에 대 한 지점 및 사이트 간 주소 범위를 입력 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-238">In **Address space**, enter the point-to-site address range for the virtual network gateway in Azure.</span></span>

    ![Azure Stack 허브 로컬 네트워크 게이트웨이의 지점 및 사이트 간 주소 공간](media/solution-deployment-guide-hybrid/image15.png)

3. <span data-ttu-id="87cc6-240">**저장** 을 선택 하 여 구성을 확인 하 고 저장 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-240">Select **Save** to validate and save the configuration.</span></span>

## <a name="configure-dns-for-cross-cloud-scaling"></a><span data-ttu-id="87cc6-241">클라우드 간 확장을 위한 DNS 구성</span><span class="sxs-lookup"><span data-stu-id="87cc6-241">Configure DNS for cross-cloud scaling</span></span>

<span data-ttu-id="87cc6-242">사용자는 클라우드 간 앱에 대 한 DNS를 적절히 구성 하 여 웹 앱의 글로벌 Azure 및 Azure Stack Hub 인스턴스에 액세스할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-242">By properly configuring DNS for cross-cloud apps, users can access the global Azure and Azure Stack Hub instances of your web app.</span></span> <span data-ttu-id="87cc6-243">이 자습서의 DNS 구성에서는 부하가 늘어나거나 감소할 때 Azure Traffic Manager 트래픽을 라우팅할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-243">The DNS configuration for this tutorial also lets Azure Traffic Manager route traffic when the load increases or decreases.</span></span>

<span data-ttu-id="87cc6-244">이 자습서에서는 Azure DNS 사용 하 여 App Service 도메인이 작동 하지 않기 때문에 DNS를 관리 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-244">This tutorial uses Azure DNS to manage the DNS because App Service domains won't work.</span></span>

### <a name="create-subdomains"></a><span data-ttu-id="87cc6-245">하위 도메인 만들기</span><span class="sxs-lookup"><span data-stu-id="87cc6-245">Create subdomains</span></span>

<span data-ttu-id="87cc6-246">Traffic Manager는 DNS CNAMEs를 사용 하기 때문에 트래픽을 끝점으로 적절 하 게 라우팅하도록 하위 도메인이 필요 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-246">Because Traffic Manager relies on DNS CNAMEs, a subdomain is needed to properly route traffic to endpoints.</span></span> <span data-ttu-id="87cc6-247">DNS 레코드 및 도메인 매핑에 대 한 자세한 내용은 [Traffic Manager를 사용 하 여 도메인 매핑](https://docs.microsoft.com/azure/app-service/web-sites-traffic-manager-custom-domain-name)을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="87cc6-247">For more information about DNS records and domain mapping, see [map domains with Traffic Manager](https://docs.microsoft.com/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span></span>

<span data-ttu-id="87cc6-248">Azure 끝점의 경우 사용자가 웹 앱에 액세스 하는 데 사용할 수 있는 하위 도메인을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-248">For the Azure endpoint, you'll create a subdomain that users can use to access your web app.</span></span> <span data-ttu-id="87cc6-249">이 자습서에서는 **app.northwind.com**를 사용할 수 있지만 사용자 고유의 도메인에 따라이 값을 사용자 지정 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-249">For this tutorial, can use **app.northwind.com**, but you should customize this value based on your own domain.</span></span>

<span data-ttu-id="87cc6-250">또한 Azure Stack 허브 끝점에 대 한 A 레코드를 사용 하 여 하위 도메인을 만들어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-250">You'll also need to create a subdomain with an A record for the Azure Stack Hub endpoint.</span></span> <span data-ttu-id="87cc6-251">**Azurestack.northwind.com**를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-251">You can use **azurestack.northwind.com**.</span></span>

### <a name="configure-a-custom-domain-in-azure"></a><span data-ttu-id="87cc6-252">Azure에서 사용자 지정 도메인 구성</span><span class="sxs-lookup"><span data-stu-id="87cc6-252">Configure a custom domain in Azure</span></span>

1. <span data-ttu-id="87cc6-253">[Azure App Service에 CNAME을 매핑하여](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record)Azure 웹 앱에 **app.northwind.com** 호스트 이름을 추가 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-253">Add the **app.northwind.com** hostname to the Azure web app by [mapping a CNAME to Azure App Service](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span>

### <a name="configure-custom-domains-in-azure-stack-hub"></a><span data-ttu-id="87cc6-254">Azure Stack 허브에서 사용자 지정 도메인 구성</span><span class="sxs-lookup"><span data-stu-id="87cc6-254">Configure custom domains in Azure Stack Hub</span></span>

1. <span data-ttu-id="87cc6-255">[A 레코드를 Azure App Service 매핑하여](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record)Azure Stack 허브 웹 앱에 **azurestack.northwind.com** 호스트 이름을 추가 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-255">Add the **azurestack.northwind.com** hostname to the Azure Stack Hub web app by [mapping an A record to Azure App Service](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span></span> <span data-ttu-id="87cc6-256">App Service 앱에 인터넷 라우팅 가능한 IP 주소를 사용 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-256">Use the internet-routable IP address for the App Service app.</span></span>

2. <span data-ttu-id="87cc6-257">[CNAME을 Azure App Service 매핑하여](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record)Azure Stack 허브 웹 앱에 **app.northwind.com** 호스트 이름을 추가 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-257">Add the **app.northwind.com** hostname to the Azure Stack Hub web app by [mapping a CNAME to Azure App Service](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span> <span data-ttu-id="87cc6-258">이전 단계에서 구성한 호스트 이름 (1)을 CNAME의 대상으로 사용 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-258">Use the hostname you configured in the previous step (1) as the target for the CNAME.</span></span>

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a><span data-ttu-id="87cc6-259">클라우드 간 크기 조정에 대 한 SSL 인증서 구성</span><span class="sxs-lookup"><span data-stu-id="87cc6-259">Configure SSL certificates for cross-cloud scaling</span></span>

<span data-ttu-id="87cc6-260">웹 앱에 의해 수집 된 중요 한 데이터를 SQL 데이터베이스에 저장 하는 경우 안전 하 게 전송할 수 있는지 확인 하는 것이 중요 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-260">It's important to ensure sensitive data collected by your web app is secure in transit to and when stored on the SQL database.</span></span>

<span data-ttu-id="87cc6-261">들어오는 모든 트래픽에 대해 SSL 인증서를 사용 하도록 Azure 및 Azure Stack Hub 웹 앱을 구성 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-261">You'll configure your Azure and Azure Stack Hub web apps to use SSL certificates for all incoming traffic.</span></span>

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a><span data-ttu-id="87cc6-262">Azure 및 Azure Stack Hub에 SSL 추가</span><span class="sxs-lookup"><span data-stu-id="87cc6-262">Add SSL to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="87cc6-263">Azure에 SSL을 추가 하려면:</span><span class="sxs-lookup"><span data-stu-id="87cc6-263">To add SSL to Azure:</span></span>

1. <span data-ttu-id="87cc6-264">사용자가 만든 하위 도메인에 대 한 SSL 인증서가 유효한 지 확인 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-264">Make sure that the SSL certificate you get is valid for the subdomain you created.</span></span> <span data-ttu-id="87cc6-265">와일드 카드 인증서를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-265">(It's okay to use wildcard certificates.)</span></span>

2. <span data-ttu-id="87cc6-266">Azure에서 [azure Web Apps에 기존 사용자 지정 ssl 인증서 바인딩](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl) 문서에 있는 **웹 앱 준비** 및 **SSL 인증서 바인딩** 섹션의 지침을 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-266">In Azure, follow the instructions in the **Prepare your web app** and **Bind your SSL certificate** sections of the [Bind an existing custom SSL certificate to Azure Web Apps](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span> <span data-ttu-id="87cc6-267">**SNI 기반 ssl** 을 **ssl 유형**으로 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-267">Select **SNI-based SSL** as the **SSL Type**.</span></span>

3. <span data-ttu-id="87cc6-268">모든 트래픽을 HTTPS 포트로 리디렉션합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-268">Redirect all traffic to the HTTPS port.</span></span> <span data-ttu-id="87cc6-269">[Azure Web Apps에 기존 사용자 지정 SSL 인증서 바인딩](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl) 문서에서 **HTTPS 적용** 섹션의 지침을 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-269">Follow the instructions in the   **Enforce HTTPS** section of the [Bind an existing custom SSL certificate to Azure Web Apps](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span>

<span data-ttu-id="87cc6-270">Azure Stack 허브에 SSL을 추가 하려면 다음을 수행 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-270">To add SSL to Azure Stack Hub:</span></span>

1. <span data-ttu-id="87cc6-271">Azure에 사용한 1-3 단계를 반복 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-271">Repeat steps 1-3 that you used for Azure.</span></span>

## <a name="configure-and-deploy-the-web-app"></a><span data-ttu-id="87cc6-272">웹 앱 구성 및 배포</span><span class="sxs-lookup"><span data-stu-id="87cc6-272">Configure and deploy the web app</span></span>

<span data-ttu-id="87cc6-273">올바른 Application Insights 인스턴스에 대 한 원격 분석을 보고 하 고 올바른 연결 문자열을 사용 하 여 웹 앱을 구성 하도록 앱 코드를 구성 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-273">You'll configure the app code to report telemetry to the correct Application Insights instance and configure the web apps with the right connection strings.</span></span> <span data-ttu-id="87cc6-274">Application Insights에 대 한 자세한 내용은 [Application Insights 항목](https://docs.microsoft.com/azure/application-insights/app-insights-overview) 을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="87cc6-274">To learn more about Application Insights, see [What is Application Insights?](https://docs.microsoft.com/azure/application-insights/app-insights-overview)</span></span>

### <a name="add-application-insights"></a><span data-ttu-id="87cc6-275">Application Insights 추가</span><span class="sxs-lookup"><span data-stu-id="87cc6-275">Add Application Insights</span></span>

1. <span data-ttu-id="87cc6-276">Microsoft Visual Studio에서 웹 앱을 엽니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-276">Open your web app in Microsoft Visual Studio.</span></span>

2. <span data-ttu-id="87cc6-277">프로젝트에 [Application Insights를 추가](https://docs.microsoft.com/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) 하 Application Insights에서 웹 트래픽이 늘어나거나 감소할 때 경고를 생성 하는 데 사용 하는 원격 분석을 전송 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-277">[Add Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) to your project to transmit the telemetry that Application Insights uses to create alerts when web traffic increases or decreases.</span></span>

### <a name="configure-dynamic-connection-strings"></a><span data-ttu-id="87cc6-278">동적 연결 문자열 구성</span><span class="sxs-lookup"><span data-stu-id="87cc6-278">Configure dynamic connection strings</span></span>

<span data-ttu-id="87cc6-279">웹 앱의 각 인스턴스는 다른 방법을 사용 하 여 SQL database에 연결 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-279">Each instance of the web app will use a different method to connect to the SQL database.</span></span> <span data-ttu-id="87cc6-280">Azure의 앱은 SQL Server VM의 개인 IP 주소를 사용 하 고 Azure Stack 허브의 앱은 SQL Server VM의 공용 IP 주소를 사용 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-280">The app in Azure uses the private IP address of the SQL Server VM and the app in Azure Stack Hub uses the public IP address of the SQL Server VM.</span></span>

> [!Note]  
> <span data-ttu-id="87cc6-281">Azure Stack 허브 통합 시스템에서 공용 IP 주소는 인터넷 라우팅할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-281">On an Azure Stack Hub integrated system, the public IP address shouldn't be internet-routable.</span></span> <span data-ttu-id="87cc6-282">ASDK에서는 공용 IP 주소를 ASDK 외부에서 라우팅할 경로를 설정 하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-282">On an ASDK, the public IP address isn't routable outside the ASDK.</span></span>

<span data-ttu-id="87cc6-283">App Service 환경 변수를 사용 하 여 앱의 각 인스턴스에 다른 연결 문자열을 전달할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-283">You can use App Service environment variables to pass a different connection string to each instance of the app.</span></span>

1. <span data-ttu-id="87cc6-284">Visual Studio에서 앱을 엽니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-284">Open the app in Visual Studio.</span></span>

2. <span data-ttu-id="87cc6-285">Startup.cs를 열고 다음 코드 블록을 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-285">Open Startup.cs and find the following code block:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. <span data-ttu-id="87cc6-286">위의 코드 블록을 파일 *의appsettings.js* 에 정의 된 연결 문자열을 사용 하는 다음 코드로 바꿉니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-286">Replace the previous code block with the following code, which uses a connection string defined in the *appsettings.json* file:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a><span data-ttu-id="87cc6-287">앱 설정 App Service 구성</span><span class="sxs-lookup"><span data-stu-id="87cc6-287">Configure App Service app settings</span></span>

1. <span data-ttu-id="87cc6-288">Azure 및 Azure Stack Hub에 대 한 연결 문자열을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-288">Create connection strings for Azure and Azure Stack Hub.</span></span> <span data-ttu-id="87cc6-289">사용 되는 IP 주소를 제외 하 고 문자열은 동일 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-289">The strings should be the same, except for the IP addresses that are used.</span></span>

2. <span data-ttu-id="87cc6-290">Azure 및 Azure Stack 허브에서 이름에 접두사로를 사용 하 여 웹 앱의 [앱 설정으로](https://docs.microsoft.com/azure/app-service/web-sites-configure) 적절 한 연결 문자열을 추가 합니다 `SQLCONNSTR\_` .</span><span class="sxs-lookup"><span data-stu-id="87cc6-290">In Azure and Azure Stack Hub, add the appropriate connection string [as an app setting](https://docs.microsoft.com/azure/app-service/web-sites-configure) in the web app, using `SQLCONNSTR\_` as a prefix in the name.</span></span>

3. <span data-ttu-id="87cc6-291">웹 앱 설정을 **저장** 하 고 앱을 다시 시작 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-291">**Save** the web app settings and restart the app.</span></span>

## <a name="enable-automatic-scaling-in-global-azure"></a><span data-ttu-id="87cc6-292">글로벌 Azure에서 자동 크기 조정 사용</span><span class="sxs-lookup"><span data-stu-id="87cc6-292">Enable automatic scaling in global Azure</span></span>

<span data-ttu-id="87cc6-293">App Service 환경에서 웹 앱을 만들 때 하나의 인스턴스로 시작 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-293">When you create your web app in an App Service environment, it starts with one instance.</span></span> <span data-ttu-id="87cc6-294">앱에 더 많은 계산 리소스를 제공 하기 위해 인스턴스를 추가 하도록 자동으로 확장할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-294">You can automatically scale out to add instances to provide more compute resources for your app.</span></span> <span data-ttu-id="87cc6-295">마찬가지로, 자동으로 규모를 확장 하 고 앱에 필요한 인스턴스 수를 줄일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-295">Similarly, you can automatically scale in and reduce the number of instances your app needs.</span></span>

> [!Note]  
> <span data-ttu-id="87cc6-296">규모 확장 및 규모 확장을 구성 하려면 App Service 계획이 필요 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-296">You need to have an App Service plan to configure scale out and scale in.</span></span> <span data-ttu-id="87cc6-297">계획이 없는 경우 다음 단계를 시작 하기 전에 계획을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-297">If you don't have a plan, create one before starting the next steps.</span></span>

### <a name="enable-automatic-scale-out"></a><span data-ttu-id="87cc6-298">자동 확장 사용</span><span class="sxs-lookup"><span data-stu-id="87cc6-298">Enable automatic scale-out</span></span>

1. <span data-ttu-id="87cc6-299">Azure에서 규모 확장 하려는 사이트에 대 한 App Service 계획을 찾은 다음 **확장 (App Service 계획)** 을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-299">In Azure, find the App Service plan for the sites you want to scale out, and then select **Scale-out (App Service plan)**.</span></span>

    ![규모 확장 Azure App Service](media/solution-deployment-guide-hybrid/image16.png)

2. <span data-ttu-id="87cc6-301">**자동 크기 조정 사용**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-301">Select **Enable autoscale**.</span></span>

    ![Azure App Service에서 자동 크기 조정 사용](media/solution-deployment-guide-hybrid/image17.png)

3. <span data-ttu-id="87cc6-303">**자동 크기 조정 설정 이름**에 이름을 입력 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-303">Enter a name for **Autoscale Setting Name**.</span></span> <span data-ttu-id="87cc6-304">**기본** 자동 크기 조정 규칙의 경우 **메트릭에 따라 크기 조정**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-304">For the **Default** auto scale rule, select **Scale based on a metric**.</span></span> <span data-ttu-id="87cc6-305">**인스턴스 제한을** **최소: 1**, **최대값: 10**및 **기본값: 1**로 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-305">Set the **Instance limits** to **Minimum: 1**, **Maximum: 10**, and **Default: 1**.</span></span>

    ![Azure App Service에서 자동 크기 조정 구성](media/solution-deployment-guide-hybrid/image18.png)

4. <span data-ttu-id="87cc6-307">**+ 규칙 추가를**선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-307">Select **+Add a rule**.</span></span>

5. <span data-ttu-id="87cc6-308">**메트릭 원본**에서 **현재 리소스**를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-308">In **Metric Source**, select **Current Resource**.</span></span> <span data-ttu-id="87cc6-309">규칙에 대해 다음 조건 및 동작을 사용 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-309">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="87cc6-310">조건</span><span class="sxs-lookup"><span data-stu-id="87cc6-310">Criteria</span></span>

1. <span data-ttu-id="87cc6-311">**시간 집계** 에서 **평균**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-311">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="87cc6-312">**메트릭 이름**아래에서 **CPU 비율**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-312">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="87cc6-313">**연산자**아래에서 **보다 큼**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-313">Under **Operator**, select **Greater than**.</span></span>

   - <span data-ttu-id="87cc6-314">**임계값** 을 **50**으로 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-314">Set the **Threshold** to **50**.</span></span>
   - <span data-ttu-id="87cc6-315">**기간** 을 **10**으로 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-315">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="87cc6-316">작업</span><span class="sxs-lookup"><span data-stu-id="87cc6-316">Action</span></span>

1. <span data-ttu-id="87cc6-317">**작업**아래에서 **개수 증가를**선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-317">Under **Operation**, select **Increase Count by**.</span></span>

2. <span data-ttu-id="87cc6-318">**인스턴스 수** 를 **2**로 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-318">Set the **Instance Count** to **2**.</span></span>

3. <span data-ttu-id="87cc6-319">**쿨** 를 **5**로 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-319">Set the **Cool down** to **5**.</span></span>

4. <span data-ttu-id="87cc6-320">**추가**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-320">Select **Add**.</span></span>

5. <span data-ttu-id="87cc6-321">**+ 규칙 추가**를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-321">Select the **+ Add a rule**.</span></span>

6. <span data-ttu-id="87cc6-322">**메트릭 원본**에서 **현재 리소스를 선택 합니다.**</span><span class="sxs-lookup"><span data-stu-id="87cc6-322">In **Metric Source**, select **Current Resource.**</span></span>

   > [!Note]  
   > <span data-ttu-id="87cc6-323">현재 리소스에 App Service 계획의 이름/GUID가 포함 되 고 **리소스 종류** 및 **리소스** 드롭다운 목록을 사용할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-323">The current resource will contain your App Service plan's name/GUID and the **Resource Type** and **Resource** drop-down lists will be unavailable.</span></span>

### <a name="enable-automatic-scale-in"></a><span data-ttu-id="87cc6-324">자동 크기 조정 사용</span><span class="sxs-lookup"><span data-stu-id="87cc6-324">Enable automatic scale in</span></span>

<span data-ttu-id="87cc6-325">트래픽이 줄어들면 Azure 웹 앱에서 자동으로 활성 인스턴스 수를 줄여 비용을 절감할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-325">When traffic decreases, the Azure web app can automatically reduce the number of active instances to reduce costs.</span></span> <span data-ttu-id="87cc6-326">이 작업은 확장 보다 낮은 수준 이며 앱 사용자에 대 한 영향을 최소화 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-326">This action is less aggressive than scale-out and minimizes the impact on app users.</span></span>

1. <span data-ttu-id="87cc6-327">**기본** scale out 조건으로 이동한 다음 **+ 규칙 추가**를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-327">Go to the **Default** scale out condition, then select **+ Add a rule**.</span></span> <span data-ttu-id="87cc6-328">규칙에 대해 다음 조건 및 동작을 사용 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-328">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="87cc6-329">조건</span><span class="sxs-lookup"><span data-stu-id="87cc6-329">Criteria</span></span>

1. <span data-ttu-id="87cc6-330">**시간 집계** 에서 **평균**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-330">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="87cc6-331">**메트릭 이름**아래에서 **CPU 비율**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-331">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="87cc6-332">**연산자**아래에서 **보다 작음**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-332">Under **Operator**, select **Less than**.</span></span>

   - <span data-ttu-id="87cc6-333">**임계값** 을 **30**으로 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-333">Set the **Threshold** to **30**.</span></span>
   - <span data-ttu-id="87cc6-334">**기간** 을 **10**으로 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-334">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="87cc6-335">작업</span><span class="sxs-lookup"><span data-stu-id="87cc6-335">Action</span></span>

1. <span data-ttu-id="87cc6-336">**작업**아래에서 **개수 줄이기를**선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-336">Under **Operation**, select **Decrease Count by**.</span></span>

   - <span data-ttu-id="87cc6-337">**인스턴스 수** 를 **1**로 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-337">Set the **Instance Count** to **1**.</span></span>
   - <span data-ttu-id="87cc6-338">**쿨** 를 **5**로 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-338">Set the **Cool down** to **5**.</span></span>

2. <span data-ttu-id="87cc6-339">**추가**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-339">Select **Add**.</span></span>

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a><span data-ttu-id="87cc6-340">Traffic Manager 프로필 만들기 및 클라우드 간 배율 구성</span><span class="sxs-lookup"><span data-stu-id="87cc6-340">Create a Traffic Manager profile and configure cross-cloud scaling</span></span>

<span data-ttu-id="87cc6-341">Azure에서 Traffic Manager 프로필을 만든 다음 클라우드를 구성 하 여 클라우드 간 크기 조정을 사용 하도록 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-341">Create a Traffic Manager profile in Azure and then configure endpoints to enable cross-cloud scaling.</span></span>

### <a name="create-traffic-manager-profile"></a><span data-ttu-id="87cc6-342">Traffic Manager 프로필 만들기</span><span class="sxs-lookup"><span data-stu-id="87cc6-342">Create Traffic Manager profile</span></span>

1. <span data-ttu-id="87cc6-343">**리소스 만들기**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-343">Select **Create a resource**.</span></span>
2. <span data-ttu-id="87cc6-344">**네트워킹**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-344">Select **Networking**.</span></span>
3. <span data-ttu-id="87cc6-345">**Traffic Manager 프로필** 을 선택 하 고 다음 설정을 구성 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-345">Select **Traffic Manager profile** and configure the following settings:</span></span>

   - <span data-ttu-id="87cc6-346">**이름**에 프로필의 이름을 입력 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-346">In **Name**, enter a name for your profile.</span></span> <span data-ttu-id="87cc6-347">이 이름은 trafficmanager.net 영역에서 고유 **해야** 하며 새 DNS 이름 (예: northwindstore.trafficmanager.net)을 만드는 데 사용 됩니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-347">This name **must** be unique in the trafficmanager.net zone and is used to create a new DNS name (for example, northwindstore.trafficmanager.net).</span></span>
   - <span data-ttu-id="87cc6-348">**라우팅 방법**의 경우 **가중치**를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-348">For **Routing method**, select the **Weighted**.</span></span>
   - <span data-ttu-id="87cc6-349">**구독**에 대해이 프로필을 만들려는 구독을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-349">For **Subscription**, select the subscription you want to create  this profile in.</span></span>
   - <span data-ttu-id="87cc6-350">**리소스 그룹**에서이 프로필에 대 한 새 리소스 그룹을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-350">In **Resource Group**, create a new resource group for this profile.</span></span>
   - <span data-ttu-id="87cc6-351">**리소스 그룹 위치**에서 리소스 그룹의 위치를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-351">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="87cc6-352">이 설정은 리소스 그룹의 위치를 나타내며 전역적으로 배포 된 Traffic Manager 프로필에는 영향을 주지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-352">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile that's deployed globally.</span></span>

4. <span data-ttu-id="87cc6-353">**만들기**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-353">Select **Create**.</span></span>

    ![Traffic Manager 프로필 만들기](media/solution-deployment-guide-hybrid/image19.png)

   <span data-ttu-id="87cc6-355">Traffic Manager 프로필의 전역 배포가 완료 되 면 해당 프로필을 만든 리소스 그룹의 리소스 목록에 표시 됩니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-355">When the global deployment of your Traffic Manager profile is complete, it's shown in the list of resources for the resource group you created it under.</span></span>

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="87cc6-356">Traffic Manager 엔드포인트 추가</span><span class="sxs-lookup"><span data-stu-id="87cc6-356">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="87cc6-357">만든 Traffic Manager 프로필을 검색 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-357">Search for the Traffic Manager profile you created.</span></span> <span data-ttu-id="87cc6-358">프로필에 대 한 리소스 그룹을 탐색 한 경우 프로필을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-358">If you navigated to the resource group for the profile, select the profile.</span></span>

2. <span data-ttu-id="87cc6-359">**Traffic Manager 프로필**의 **설정**에서 **끝점**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-359">In **Traffic Manager profile**, under **SETTINGS**, select **Endpoints**.</span></span>

3. <span data-ttu-id="87cc6-360">**추가**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-360">Select **Add**.</span></span>

4. <span data-ttu-id="87cc6-361">**끝점 추가**에서 Azure Stack 허브에 대해 다음 설정을 사용 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-361">In **Add endpoint**, use the following settings for Azure Stack Hub:</span></span>

   - <span data-ttu-id="87cc6-362">**형식**에서 **외부 끝점**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-362">For **Type**, select **External endpoint**.</span></span>
   - <span data-ttu-id="87cc6-363">끝점의 **이름을** 입력 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-363">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="87cc6-364">**FQDN (정규화 된 도메인 이름) 또는 IP**의 경우 Azure Stack 허브 웹 앱에 대 한 외부 URL을 입력 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-364">For **Fully qualified domain name (FQDN) or IP**, enter the external URL for your Azure Stack Hub web app.</span></span>
   - <span data-ttu-id="87cc6-365">**가중치**의 경우 기본값 **1**을 유지 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-365">For **Weight**, keep the default, **1**.</span></span> <span data-ttu-id="87cc6-366">이 가중치를 통해 정상 상태 이면 모든 트래픽이이 끝점으로 이동 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-366">This weight results in all traffic going to this endpoint if it's healthy.</span></span>
   - <span data-ttu-id="87cc6-367">**추가 사용 안 함을** 선택 하지 않은 상태로 둡니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-367">Leave **Add as disabled** unchecked.</span></span>

5. <span data-ttu-id="87cc6-368">**확인** 을 선택 하 여 Azure Stack 허브 끝점을 저장 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-368">Select **OK** to save the Azure Stack Hub endpoint.</span></span>

<span data-ttu-id="87cc6-369">다음으로 Azure 엔드포인트를 구성 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-369">You'll configure the Azure endpoint next.</span></span>

1. <span data-ttu-id="87cc6-370">**Traffic Manager 프로필**에서 **끝점**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-370">On **Traffic Manager profile**, select **Endpoints**.</span></span>
2. <span data-ttu-id="87cc6-371">**+ 추가**를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-371">Select **+Add**.</span></span>
3. <span data-ttu-id="87cc6-372">**끝점 추가**에서 Azure에 대해 다음 설정을 사용 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-372">On **Add endpoint**, use the following settings for Azure:</span></span>

   - <span data-ttu-id="87cc6-373">**유형**에 대해 **Azure 엔드포인트**를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-373">For **Type**, select **Azure endpoint**.</span></span>
   - <span data-ttu-id="87cc6-374">끝점의 **이름을** 입력 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-374">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="87cc6-375">**대상 리소스 종류**에 대해 **App Service**를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-375">For **Target resource type**, select **App Service**.</span></span>
   - <span data-ttu-id="87cc6-376">**대상 리소스**에 대해 **앱 서비스 선택** 을 선택 하 여 동일한 구독의 Web Apps 목록을 표시 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-376">For **Target resource**, select **Choose an app service** to see a list of Web Apps in the same subscription.</span></span>
   - <span data-ttu-id="87cc6-377">**리소스**에서 첫 번째 엔드포인트로 추가할 앱 서비스를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-377">In **Resource**, pick the App service that you want to add as the first endpoint.</span></span>
   - <span data-ttu-id="87cc6-378">**가중치**에 대해 **2**를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-378">For **Weight**, select **2**.</span></span> <span data-ttu-id="87cc6-379">이렇게 설정 하면 기본 끝점이 비정상 이거나 트리거된 경우 트래픽을 리디렉션하는 규칙/경고가 있는 경우 모든 트래픽이이 끝점으로 이동 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-379">This setting results in all traffic going to this endpoint if the primary endpoint is unhealthy, or if you have a rule/alert that redirects traffic when triggered.</span></span>
   - <span data-ttu-id="87cc6-380">**추가 사용 안 함을** 선택 하지 않은 상태로 둡니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-380">Leave **Add as disabled** unchecked.</span></span>

4. <span data-ttu-id="87cc6-381">**확인** 을 선택 하 여 Azure 끝점을 저장 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-381">Select **OK** to save the Azure endpoint.</span></span>

<span data-ttu-id="87cc6-382">두 끝점을 모두 구성한 후에는 **끝점**을 선택할 때 **Traffic Manager 프로필** 에 나열 됩니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-382">After both endpoints are configured, they're listed in **Traffic Manager profile** when you select **Endpoints**.</span></span> <span data-ttu-id="87cc6-383">다음 화면 캡처의 예제에서는 각각에 대 한 상태 및 구성 정보를 포함 하는 두 개의 끝점을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-383">The example in the following screen capture shows two endpoints, with status and configuration information for each one.</span></span>

![Traffic Manager 프로필의 끝점](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting"></a><span data-ttu-id="87cc6-385">Application Insights 모니터링 및 경고 설정</span><span class="sxs-lookup"><span data-stu-id="87cc6-385">Set up Application Insights monitoring and alerting</span></span>

<span data-ttu-id="87cc6-386">Azure 애플리케이션 Insights를 사용 하 여 앱을 모니터링 하 고 구성 하는 조건에 따라 경고를 보낼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-386">Azure Application Insights lets you monitor your app and send alerts based on conditions you configure.</span></span> <span data-ttu-id="87cc6-387">응용 프로그램을 사용할 수 없거나 오류가 발생 하거나 성능 문제를 보여 주는 몇 가지 예가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-387">Some examples are: the app is unavailable, is experiencing failures, or is showing performance issues.</span></span>

<span data-ttu-id="87cc6-388">Application Insights 메트릭을 사용 하 여 경고를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-388">You'll use Application Insights metrics to create alerts.</span></span> <span data-ttu-id="87cc6-389">이러한 경고가 트리거될 때 웹 앱의 인스턴스는 자동으로 Azure Stack 허브에서 Azure로 전환 하 여 규모를 확장 한 후 규모 확장을 위해 Azure Stack 허브로 돌아갑니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-389">When these alerts trigger, your web app's instance will automatically switch from Azure Stack Hub to Azure to scale out, and then back to Azure Stack Hub to scale in.</span></span>

### <a name="create-an-alert-from-metrics"></a><span data-ttu-id="87cc6-390">메트릭에서 경고 만들기</span><span class="sxs-lookup"><span data-stu-id="87cc6-390">Create an alert from metrics</span></span>

<span data-ttu-id="87cc6-391">이 자습서의 리소스 그룹으로 이동한 다음 Application Insights 인스턴스를 선택 하 여 **Application Insights**를 엽니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-391">Go to the resource group for this tutorial and then select the Application Insights instance to open **Application Insights**.</span></span>

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

<span data-ttu-id="87cc6-393">이 보기를 사용 하 여 스케일 아웃 경고 및 규모 확장 경고를 만들 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-393">You'll use this view to create a scale-out alert and a scale-in alert.</span></span>

### <a name="create-the-scale-out-alert"></a><span data-ttu-id="87cc6-394">스케일 아웃 경고 만들기</span><span class="sxs-lookup"><span data-stu-id="87cc6-394">Create the scale-out alert</span></span>

1. <span data-ttu-id="87cc6-395">**구성**아래에서 **경고 (클래식)** 를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-395">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="87cc6-396">**메트릭 경고 추가(클래식)** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-396">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="87cc6-397">**규칙 추가**에서 다음 설정을 구성 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-397">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="87cc6-398">**이름**에 대해 **Azure 클라우드에 버스트**를 입력 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-398">For **Name**, enter **Burst into Azure Cloud**.</span></span>
   - <span data-ttu-id="87cc6-399">**설명은** 선택 사항입니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-399">A **Description** is optional.</span></span>
   - <span data-ttu-id="87cc6-400">**원본**  >  **경고 켜기**에서 **메트릭**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-400">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="87cc6-401">**조건**에서 구독, Traffic Manager 프로필에 대 한 리소스 그룹 및 리소스에 대 한 Traffic Manager 프로필의 이름을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-401">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="87cc6-402">**메트릭에**대해 **요청 빈도**를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-402">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="87cc6-403">**조건**에 대해 **보다 큼**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-403">For **Condition**, select **Greater than**.</span></span>
6. <span data-ttu-id="87cc6-404">**임계값**에 **2**를 입력 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-404">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="87cc6-405">**기간**에서 **지난 5 분**동안을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-405">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="87cc6-406">다음을 **통해 알림**:</span><span class="sxs-lookup"><span data-stu-id="87cc6-406">Under **Notify via**:</span></span>
   - <span data-ttu-id="87cc6-407">**전자 메일 소유자, 참가자 및 읽기 권한자**의 확인란을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-407">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="87cc6-408">**추가 관리자 전자 메일**에 대 한 전자 메일 주소를 입력 하세요.</span><span class="sxs-lookup"><span data-stu-id="87cc6-408">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="87cc6-409">메뉴 모음에서 **저장**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-409">On the menu bar, select **Save**.</span></span>

### <a name="create-the-scale-in-alert"></a><span data-ttu-id="87cc6-410">규모 확장 경고 만들기</span><span class="sxs-lookup"><span data-stu-id="87cc6-410">Create the scale-in alert</span></span>

1. <span data-ttu-id="87cc6-411">**구성**아래에서 **경고 (클래식)** 를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-411">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="87cc6-412">**메트릭 경고 추가(클래식)** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-412">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="87cc6-413">**규칙 추가**에서 다음 설정을 구성 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-413">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="87cc6-414">**이름**에 **Azure Stack 허브로 크기를 다시**입력 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-414">For **Name**, enter **Scale back into Azure Stack Hub**.</span></span>
   - <span data-ttu-id="87cc6-415">**설명은** 선택 사항입니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-415">A **Description** is optional.</span></span>
   - <span data-ttu-id="87cc6-416">**원본**  >  **경고 켜기**에서 **메트릭**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-416">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="87cc6-417">**조건**에서 구독, Traffic Manager 프로필에 대 한 리소스 그룹 및 리소스에 대 한 Traffic Manager 프로필의 이름을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-417">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="87cc6-418">**메트릭에**대해 **요청 빈도**를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-418">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="87cc6-419">**조건**에 대해 **미만**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-419">For **Condition**, select **Less than**.</span></span>
6. <span data-ttu-id="87cc6-420">**임계값**에 **2**를 입력 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-420">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="87cc6-421">**기간**에서 **지난 5 분**동안을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-421">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="87cc6-422">다음을 **통해 알림**:</span><span class="sxs-lookup"><span data-stu-id="87cc6-422">Under **Notify via**:</span></span>
   - <span data-ttu-id="87cc6-423">**전자 메일 소유자, 참가자 및 읽기 권한자**의 확인란을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-423">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="87cc6-424">**추가 관리자 전자 메일**에 대 한 전자 메일 주소를 입력 하세요.</span><span class="sxs-lookup"><span data-stu-id="87cc6-424">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="87cc6-425">메뉴 모음에서 **저장**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-425">On the menu bar, select **Save**.</span></span>

<span data-ttu-id="87cc6-426">다음 스크린샷에서는 규모 확장 및 규모 확장에 대 한 경고를 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-426">The following screenshot shows the alerts for scale-out and scale-in.</span></span>

   ![Application Insights 경고 (클래식)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a><span data-ttu-id="87cc6-428">Azure와 Azure Stack 허브 간에 트래픽 리디렉션</span><span class="sxs-lookup"><span data-stu-id="87cc6-428">Redirect traffic between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="87cc6-429">Azure와 Azure Stack Hub 간의 웹 앱 트래픽 수동 또는 자동 전환을 구성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-429">You can configure manual or automatic switching of your web app traffic between Azure and Azure Stack Hub.</span></span>

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="87cc6-430">Azure와 Azure Stack Hub 간의 수동 전환 구성</span><span class="sxs-lookup"><span data-stu-id="87cc6-430">Configure manual switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="87cc6-431">웹 사이트가 구성 된 임계값에 도달 하면 경고를 받게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-431">When your web site reaches the thresholds that you configure, you'll receive an alert.</span></span> <span data-ttu-id="87cc6-432">다음 단계를 사용 하 여 트래픽을 Azure로 수동으로 리디렉션합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-432">Use the following steps to manually redirect traffic to Azure.</span></span>

1. <span data-ttu-id="87cc6-433">Azure Portal에서 Traffic Manager 프로필을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-433">In the Azure portal, select your Traffic Manager profile.</span></span>

    ![Azure Portal의 Traffic Manager 끝점](media/solution-deployment-guide-hybrid/image20.png)

2. <span data-ttu-id="87cc6-435">**엔드포인트**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-435">Select **Endpoints**.</span></span>
3. <span data-ttu-id="87cc6-436">**Azure 끝점**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-436">Select the **Azure endpoint**.</span></span>
4. <span data-ttu-id="87cc6-437">**상태**에서 **사용**을 선택한 다음 **저장**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-437">Under **Status**, select **Enabled**, and then select **Save**.</span></span>

    ![Azure Portal에서 Azure 끝점 사용](media/solution-deployment-guide-hybrid/image23.png)

5. <span data-ttu-id="87cc6-439">Traffic Manager 프로필에 대 한 **끝점** 에서 **외부 끝점**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-439">On **Endpoints** for the Traffic Manager profile, select **External endpoint**.</span></span>
6. <span data-ttu-id="87cc6-440">**상태**에서 **사용 안 함**을 선택한 다음, **저장**을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-440">Under **Status**, select **Disabled**, and then select **Save**.</span></span>

    ![Azure Portal에서 Azure Stack 허브 끝점 사용 안 함](media/solution-deployment-guide-hybrid/image24.png)

<span data-ttu-id="87cc6-442">끝점이 구성 되 면 앱 트래픽이 Azure Stack 허브 웹 앱 대신 Azure 스케일 아웃 웹 앱으로 이동 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-442">After the endpoints are configured, app traffic goes to your Azure scale-out web app instead of the Azure Stack Hub web app.</span></span>

 ![Azure 웹 앱 트래픽에서 변경 된 끝점](media/solution-deployment-guide-hybrid/image25.png)

<span data-ttu-id="87cc6-444">흐름을 Azure Stack 허브로 되돌리려면 이전 단계를 사용 하 여 다음을 수행 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-444">To reverse the flow back to Azure Stack Hub, use the previous steps to:</span></span>

- <span data-ttu-id="87cc6-445">Azure Stack 허브 끝점을 사용 하도록 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-445">Enable the Azure Stack Hub endpoint.</span></span>
- <span data-ttu-id="87cc6-446">Azure 끝점을 사용 하지 않도록 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-446">Disable the Azure endpoint.</span></span>

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="87cc6-447">Azure와 Azure Stack Hub 간의 자동 전환 구성</span><span class="sxs-lookup"><span data-stu-id="87cc6-447">Configure automatic switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="87cc6-448">Azure Functions에서 제공 하는 [서버](https://azure.microsoft.com/overview/serverless-computing/) 를 사용 하지 않는 환경에서 앱을 실행 하는 경우에도 Application Insights 모니터링을 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-448">You can also use Application Insights monitoring if your app runs in a [serverless](https://azure.microsoft.com/overview/serverless-computing/) environment provided by Azure Functions.</span></span>

<span data-ttu-id="87cc6-449">이 시나리오에서는 함수 앱을 호출 하는 webhook를 사용 하도록 Application Insights를 구성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-449">In this scenario, you can configure Application Insights to use a webhook that calls a function app.</span></span> <span data-ttu-id="87cc6-450">이 앱은 경고에 대 한 응답으로 끝점을 자동으로 사용 하거나 사용 하지 않도록 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-450">This app automatically enables or disables an endpoint in response to an alert.</span></span>

<span data-ttu-id="87cc6-451">자동 트래픽 전환을 구성 하려면 다음 단계를 지침으로 사용 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-451">Use the following steps as a guide to configure automatic traffic switching.</span></span>

1. <span data-ttu-id="87cc6-452">Azure 함수 앱을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-452">Create an Azure Function app.</span></span>
2. <span data-ttu-id="87cc6-453">HTTP 트리거 함수를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-453">Create an HTTP-triggered function.</span></span>
3. <span data-ttu-id="87cc6-454">리소스 관리자, Web Apps 및 Traffic Manager에 대 한 Azure Sdk를 가져옵니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-454">Import the Azure SDKs for Resource Manager, Web Apps, and Traffic Manager.</span></span>
4. <span data-ttu-id="87cc6-455">다음에 대 한 코드를 개발 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-455">Develop code to:</span></span>

   - <span data-ttu-id="87cc6-456">Azure 구독에 인증 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-456">Authenticate to your Azure subscription.</span></span>
   - <span data-ttu-id="87cc6-457">Traffic Manager 끝점을 전환 하는 매개 변수를 사용 하 여 트래픽을 Azure 또는 Azure Stack 허브로 직접 보냅니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-457">Use a parameter that toggles the Traffic Manager endpoints to direct traffic to Azure or Azure Stack Hub.</span></span>

5. <span data-ttu-id="87cc6-458">코드를 저장 하 고 적절 한 매개 변수를 사용 하 여 함수 앱의 URL을 Application Insights 경고 규칙 설정의 **Webhook** 섹션에 추가 합니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-458">Save your code and add the function app's URL with the appropriate parameters to the **Webhook** section of the Application Insights alert rule settings.</span></span>
6. <span data-ttu-id="87cc6-459">Application Insights 경고가 발생 하면 트래픽이 자동으로 리디렉션됩니다.</span><span class="sxs-lookup"><span data-stu-id="87cc6-459">Traffic is automatically redirected when an Application Insights alert fires.</span></span>

## <a name="next-steps"></a><span data-ttu-id="87cc6-460">다음 단계</span><span class="sxs-lookup"><span data-stu-id="87cc6-460">Next steps</span></span>

- <span data-ttu-id="87cc6-461">Azure Cloud 패턴에 대해 자세히 알아보려면 [클라우드 디자인 패턴](https://docs.microsoft.com/azure/architecture/patterns)을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="87cc6-461">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](https://docs.microsoft.com/azure/architecture/patterns).</span></span>
