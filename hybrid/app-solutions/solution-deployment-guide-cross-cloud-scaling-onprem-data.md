---
title: 클라우드 간 크기를 조정하는 온-프레미스 데이터를 사용하는 하이브리드 앱 배포
description: 온-프레미스 데이터를 사용하고 Azure 및 Azure Stack Hub를 사용하여 클라우드 간 크기를 조정하는 앱을 배포하는 방법을 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 0989859fd68847932d3e69defee59740a2bffd44
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895400"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a><span data-ttu-id="d1e60-103">클라우드 간 크기를 조정하는 온-프레미스 데이터를 사용하는 하이브리드 앱 배포</span><span class="sxs-lookup"><span data-stu-id="d1e60-103">Deploy hybrid app with on-premises data that scales cross-cloud</span></span>

<span data-ttu-id="d1e60-104">이 솔루션 가이드는 Azure와 Azure Stack Hub 모두에 걸쳐 있고 단일 온-프레미스 데이터 원본을 사용하는 하이브리드 앱을 배포하는 방법을 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-104">This solution guide shows you how to deploy a hybrid app that spans both Azure and Azure Stack Hub and uses a single on-premises data source.</span></span>

<span data-ttu-id="d1e60-105">하이브리드 클라우드 솔루션을 사용하여 프라이빗 클라우드의 규정 준수 이점을 퍼블릭 클라우드의 확장성과 결합할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-105">By using a hybrid cloud solution, you can combine the compliance benefits of a private cloud with the scalability of the public cloud.</span></span> <span data-ttu-id="d1e60-106">개발자는 Microsoft 개발자 에코 시스템을 활용하여 클라우드 및 온-프레미스 환경에 기술을 적용할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-106">Your developers can also take advantage of the Microsoft developer ecosystem and apply their skills to the cloud and on-premises environments.</span></span>

## <a name="overview-and-assumptions"></a><span data-ttu-id="d1e60-107">개요 및 가정</span><span class="sxs-lookup"><span data-stu-id="d1e60-107">Overview and assumptions</span></span>

<span data-ttu-id="d1e60-108">이 자습서에 따라 개발자가 퍼블릭 클라우드와 프라이빗 클라우드에 동일한 웹앱을 배포할 수 있는 워크플로를 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-108">Follow this tutorial to set up a workflow that lets developers deploy an identical web app to a public cloud and a private cloud.</span></span> <span data-ttu-id="d1e60-109">이 앱은 프라이빗 클라우드에서 호스트되는 비인터넷 라우팅 가능 네트워크에 액세스할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-109">This app can access a non-internet routable network hosted on the private cloud.</span></span> <span data-ttu-id="d1e60-110">이러한 웹앱은 모니터링되며 트래픽이 급증하는 경우에는 트래픽이 퍼블릭 클라우드로 리디렉션되도록 프로그램에서 DNS 레코드가 수정됩니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-110">These web apps are monitored and when there's a spike in traffic, a program modifies the DNS records to redirect traffic to the public cloud.</span></span> <span data-ttu-id="d1e60-111">트래픽이 급증 이전 수준으로 떨어지면 다시 프라이빗 클라우드로 트래픽이 라우팅됩니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-111">When traffic drops to the level before the spike, traffic is routed back to the private cloud.</span></span>

<span data-ttu-id="d1e60-112">이 자습서에서 다루는 작업은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-112">This tutorial covers the following tasks:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="d1e60-113">하이브리드 연결 SQL Server 데이터베이스 서버를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-113">Deploy a hybrid-connected SQL Server database server.</span></span>
> - <span data-ttu-id="d1e60-114">글로벌 Azure의 웹앱을 하이브리드 네트워크에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-114">Connect a web app in global Azure to a hybrid network.</span></span>
> - <span data-ttu-id="d1e60-115">클라우드 간 크기 조정을 위해 DNS 구성</span><span class="sxs-lookup"><span data-stu-id="d1e60-115">Configure DNS for cross-cloud scaling.</span></span>
> - <span data-ttu-id="d1e60-116">클라우드 간 크기 조정을 위해 SSL 인증서를 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-116">Configure SSL certificates for cross-cloud scaling.</span></span>
> - <span data-ttu-id="d1e60-117">웹앱을 구성하고 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-117">Configure and deploy the web app.</span></span>
> - <span data-ttu-id="d1e60-118">Traffic Manager 프로필을 만들고 클라우드 간 크기 조정에 맞게 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-118">Create a Traffic Manager profile and configure it for cross-cloud scaling.</span></span>
> - <span data-ttu-id="d1e60-119">Application Insights 모니터링 및 트래픽 증가에 대한 경고를 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-119">Set up Application Insights monitoring and alerting for increased traffic.</span></span>
> - <span data-ttu-id="d1e60-120">글로벌 Azure와 Azure Stack Hub 간에 자동 트래픽 전환을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-120">Configure automatic traffic switching between global Azure and Azure Stack Hub.</span></span>

> [!Tip]  
> <span data-ttu-id="d1e60-121">![하이브리드 핵심 요소 다이어그램](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="d1e60-121">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="d1e60-122">Microsoft Azure Stack Hub는 Azure의 확장입니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-122">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="d1e60-123">Azure Stack Hub는 온-프레미스 환경에 클라우드 컴퓨팅의 민첩성과 혁신을 제공하여 어디서나 하이브리드 앱을 빌드하고 배포할 수 있는 유일한 하이브리드 클라우드를 사용하도록 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-123">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="d1e60-124">[하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 문서는 하이브리드 앱 디자인, 배포 및 운영에 대한 소프트웨어 품질(배치, 확장성, 가용성, 복원력, 관리 효율성 및 보안)의 핵심 요소를 검토합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-124">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="d1e60-125">디자인 고려 사항은 하이브리드 앱 디자인을 최적화하고 프로덕션 환경에서 문제를 최소화하는 데 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-125">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

### <a name="assumptions"></a><span data-ttu-id="d1e60-126">가정</span><span class="sxs-lookup"><span data-stu-id="d1e60-126">Assumptions</span></span>

<span data-ttu-id="d1e60-127">이 자습서에서는 글로벌 Azure 및 Azure Stack Hub에 대한 기본 지식이 있다고 가정합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-127">This tutorial assumes that you have a basic knowledge of global Azure and Azure Stack Hub.</span></span> <span data-ttu-id="d1e60-128">자습서를 시작하기 전에 자세히 알아보려면 다음 문서를 검토하세요.</span><span class="sxs-lookup"><span data-stu-id="d1e60-128">If you want to learn more before starting the tutorial, review these articles:</span></span>

- [<span data-ttu-id="d1e60-129">Azure 소개</span><span class="sxs-lookup"><span data-stu-id="d1e60-129">Introduction to Azure</span></span>](https://azure.microsoft.com/overview/what-is-azure/)
- [<span data-ttu-id="d1e60-130">Azure Stack Hub 주요 개념</span><span class="sxs-lookup"><span data-stu-id="d1e60-130">Azure Stack Hub Key Concepts</span></span>](/azure-stack/operator/azure-stack-overview)

<span data-ttu-id="d1e60-131">이 자습서에서는 Azure 구독이 있다고 가정합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-131">This tutorial also assumes that you have an Azure subscription.</span></span> <span data-ttu-id="d1e60-132">구독이 없는 경우 시작하기 전에 [체험 계정](https://azure.microsoft.com/free/)을 만드세요.</span><span class="sxs-lookup"><span data-stu-id="d1e60-132">If you don't have a subscription, [create a free account](https://azure.microsoft.com/free/) before you begin.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="d1e60-133">필수 구성 요소</span><span class="sxs-lookup"><span data-stu-id="d1e60-133">Prerequisites</span></span>

<span data-ttu-id="d1e60-134">이 솔루션을 시작하기 전에 다음 요구 사항을 충족하는지 확인하십시오.</span><span class="sxs-lookup"><span data-stu-id="d1e60-134">Before you start this solution, make sure you meet the following requirements:</span></span>

- <span data-ttu-id="d1e60-135">ASDK(Azure Stack Development Kit) 또는 Azure Stack Hub 통합 시스템 구독</span><span class="sxs-lookup"><span data-stu-id="d1e60-135">An Azure Stack Development Kit (ASDK) or a subscription on an Azure Stack Hub Integrated System.</span></span> <span data-ttu-id="d1e60-136">ASDK을 배포하려면 [설치 프로그램을 사용하여 ASDK 배포](/azure-stack/asdk/asdk-install)의 지침을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="d1e60-136">To deploy the ASDK, follow the instructions in [Deploy the ASDK using the installer](/azure-stack/asdk/asdk-install).</span></span>
- <span data-ttu-id="d1e60-137">Azure Stack Hub 설치에는 다음이 설치되어 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-137">Your Azure Stack Hub installation should have the following installed:</span></span>
  - <span data-ttu-id="d1e60-138">Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="d1e60-138">The Azure App Service.</span></span> <span data-ttu-id="d1e60-139">Azure Stack Hub 운영자와 협력하여 Azure App Service를 환경에 배포하고 구성하세요.</span><span class="sxs-lookup"><span data-stu-id="d1e60-139">Work with your Azure Stack Hub Operator to deploy and configure the Azure App Service on your environment.</span></span> <span data-ttu-id="d1e60-140">이 자습서에서는 App Service에 사용 가능한 전용 작업자 역할이 하나(1) 이상 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-140">This tutorial requires the App Service to have at least one (1) available dedicated worker role.</span></span>
  - <span data-ttu-id="d1e60-141">Windows Server 2016 이미지.</span><span class="sxs-lookup"><span data-stu-id="d1e60-141">A Windows Server 2016 image.</span></span>
  - <span data-ttu-id="d1e60-142">Microsoft SQL Server 이미지를 포함하는 Windows Server 2016.</span><span class="sxs-lookup"><span data-stu-id="d1e60-142">A Windows Server 2016 with a Microsoft SQL Server image.</span></span>
  - <span data-ttu-id="d1e60-143">적절한 계획 및 제안.</span><span class="sxs-lookup"><span data-stu-id="d1e60-143">The appropriate plans and offers.</span></span>
  - <span data-ttu-id="d1e60-144">웹앱의 도메인 이름.</span><span class="sxs-lookup"><span data-stu-id="d1e60-144">A domain name for your web app.</span></span> <span data-ttu-id="d1e60-145">도메인 이름이 없는 경우 GoDaddy, Bluehost 및 InMotion과 같은 도메인 공급자로부터 구입할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-145">If you don't have a domain name, you can buy one from a domain provider like GoDaddy, Bluehost, and InMotion.</span></span>
- <span data-ttu-id="d1e60-146">신뢰할 수 있는 인증 기관(예: LetsEncrypt)의 도메인에 대한 SSL 인증서</span><span class="sxs-lookup"><span data-stu-id="d1e60-146">An SSL certificate for your domain from a trusted certificate authority like LetsEncrypt.</span></span>
- <span data-ttu-id="d1e60-147">SQL Server 데이터베이스와 통신하고 Application Insights를 지원하는 웹앱.</span><span class="sxs-lookup"><span data-stu-id="d1e60-147">A web app that communicates with a SQL Server database and supports Application Insights.</span></span> <span data-ttu-id="d1e60-148">GitHub에서 [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) 샘플 앱을 다운로드할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-148">You can download the [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) sample app from GitHub.</span></span>
- <span data-ttu-id="d1e60-149">Azure 가상 네트워크와 Azure Stack Hub 가상 네트워크 간의 하이브리드 네트워크.</span><span class="sxs-lookup"><span data-stu-id="d1e60-149">A hybrid network between an Azure virtual network and Azure Stack Hub virtual network.</span></span> <span data-ttu-id="d1e60-150">자세한 지침은 [Azure 및 Azure Stack Hub를 사용하여 하이브리드 클라우드 연결 구성](solution-deployment-guide-connectivity.md)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="d1e60-150">For detailed instructions, see [Configure hybrid cloud connectivity with Azure and Azure Stack Hub](solution-deployment-guide-connectivity.md).</span></span>

- <span data-ttu-id="d1e60-151">Azure Stack Hub에 프라이빗 빌드 에이전트가 있는 하이브리드 CI/CD(연속 통합/지속적인 배포) 파이프라인.</span><span class="sxs-lookup"><span data-stu-id="d1e60-151">A hybrid continuous integration/continuous deployment (CI/CD) pipeline with a private build agent on Azure Stack Hub.</span></span> <span data-ttu-id="d1e60-152">자세한 지침은 [Azure 및 Azure Stack Hub 앱을 사용하여 하이브리드 클라우드 ID 구성](solution-deployment-guide-identity.md)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="d1e60-152">For detailed instructions, see [Configure hybrid cloud identity with Azure and Azure Stack Hub apps](solution-deployment-guide-identity.md).</span></span>

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a><span data-ttu-id="d1e60-153">하이브리드 연결 SQL Server 데이터베이스 서버 배포</span><span class="sxs-lookup"><span data-stu-id="d1e60-153">Deploy a hybrid-connected SQL Server database server</span></span>

1. <span data-ttu-id="d1e60-154">Azure Stack Hub 사용자 포털에 로그인합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-154">Sign to the Azure Stack Hub user portal.</span></span>

2. <span data-ttu-id="d1e60-155">**대시보드** 에서 **Marketplace** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-155">On the **Dashboard**, select **Marketplace**.</span></span>

    ![Azure Stack Hub Marketplace](media/solution-deployment-guide-hybrid/image1.png)

3. <span data-ttu-id="d1e60-157">**Marketplace** 에서 **Compute** 를 선택한 다음, **추가** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-157">In **Marketplace**, select **Compute**, and then choose **More**.</span></span> <span data-ttu-id="d1e60-158">**기타** 에서 **무료 SQL Server 라이선스: Windows Server의 SQL Server 2017 Developer** 이미지를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-158">Under **More**, select the **Free SQL Server License: SQL Server 2017 Developer on Windows Server** image.</span></span>

    ![Azure Stack Hub 사용자 포털에서 가상 머신 이미지를 선택합니다.](media/solution-deployment-guide-hybrid/image2.png)

4. <span data-ttu-id="d1e60-160">**체험 SQL Server 라이선스: Windows Server의 SQL Server 2017 Developer** 에서 **만들기** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-160">On **Free SQL Server License: SQL Server 2017 Developer on Windows Server**, select **Create**.</span></span>

5. <span data-ttu-id="d1e60-161">**기본 > 기본 설정 구성** 에서 VM(가상 머신)의 **이름**, SQL Server SA의 **사용자 이름**, SA의 **암호** 를 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-161">On **Basics > Configure basic settings**, provide a **Name** for the virtual machine (VM), a **User name** for the SQL Server SA, and a **Password** for the SA.</span></span>  <span data-ttu-id="d1e60-162">**구독** 드롭다운 목록에서 배포하려는 구독을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-162">From the **Subscription** drop-down list, select the subscription that you're deploying to.</span></span> <span data-ttu-id="d1e60-163">**리소스 그룹** 의 경우 **기존 항목 선택** 을 사용하고 Azure Stack Hub 웹앱과 동일한 리소스 그룹에 VM을 배치합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-163">For **Resource group**, use **Choose existing** and put the VM in the same resource group as your Azure Stack Hub web app.</span></span>

    ![Azure Stack Hub 사용자 포털에서 VM에 대한 기본 설정 구성](media/solution-deployment-guide-hybrid/image3.png)

6. <span data-ttu-id="d1e60-165">**크기** 에서 VM의 크기를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-165">Under **Size**, pick a size for your VM.</span></span> <span data-ttu-id="d1e60-166">이 자습서에서는 A2_Standard 또는 DS2_V2_Standard를 선택하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-166">For this tutorial, we recommend A2_Standard or a DS2_V2_Standard.</span></span>

7. <span data-ttu-id="d1e60-167">**설정 > 옵션 기능 구성** 에서 다음 설정을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-167">Under **Settings > Configure optional features**, configure the following settings:</span></span>

   - <span data-ttu-id="d1e60-168">**스토리지 계정**: 필요한 경우 새 계정을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-168">**Storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="d1e60-169">**가상 네트워크**:</span><span class="sxs-lookup"><span data-stu-id="d1e60-169">**Virtual network**:</span></span>

     > [!Important]  
     > <span data-ttu-id="d1e60-170">VPN 게이트웨이와 동일한 가상 네트워크에 SQL Server VM이 배포되어 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-170">Make sure your SQL Server VM is deployed on the same  virtual network as the VPN gateways.</span></span>

   - <span data-ttu-id="d1e60-171">**공용 IP 주소**: 기본 설정을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-171">**Public IP address**: Use the default settings.</span></span>
   - <span data-ttu-id="d1e60-172">**네트워크 보안 그룹**: (NSG).</span><span class="sxs-lookup"><span data-stu-id="d1e60-172">**Network security group**: (NSG).</span></span> <span data-ttu-id="d1e60-173">새 NSG를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-173">Create a new NSG.</span></span>
   - <span data-ttu-id="d1e60-174">**확장 및 모니터링**: 기본 설정을 유지합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-174">**Extensions and Monitoring**: Keep the default settings.</span></span>
   - <span data-ttu-id="d1e60-175">**진단 스토리지 계정**: 필요한 경우 새 계정을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-175">**Diagnostics storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="d1e60-176">**확인** 을 선택하여 구성을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-176">Select **OK** to save your configuration.</span></span>

     ![Azure Stack Hub 사용자 포털에서 선택적 VM 기능 구성](media/solution-deployment-guide-hybrid/image4.png)

8. <span data-ttu-id="d1e60-178">**SQL Server 설정** 에서 다음 설정을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-178">Under **SQL Server settings**, configure the following settings:</span></span>

   - <span data-ttu-id="d1e60-179">**SQL 연결** 에는 **공용(인터넷)** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-179">For **SQL connectivity**, select **Public (Internet)**.</span></span>
   - <span data-ttu-id="d1e60-180">**포트** 의 경우 기본값인 **1433** 을 유지합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-180">For **Port**, keep the default, **1433**.</span></span>
   - <span data-ttu-id="d1e60-181">**SQL 인증** 에는 **사용** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-181">For **SQL authentication**, select **Enable**.</span></span>

     > [!Note]  
     > <span data-ttu-id="d1e60-182">SQL 인증을 사용하도록 설정하면 **기본** 에서 구성한 "SQLAdmin" 정보가 자동으로 채워집니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-182">When you enable SQL authentication, it should auto-populate with the "SQLAdmin" information that you configured in **Basics**.</span></span>

   - <span data-ttu-id="d1e60-183">나머지 설정에 대해서는 기본값을 유지합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-183">For the rest of the settings, keep the defaults.</span></span> <span data-ttu-id="d1e60-184">**확인** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-184">Select **OK**.</span></span>

     ![Azure Stack Hub 사용자 포털에서 SQL Server 설정 구성](media/solution-deployment-guide-hybrid/image5.png)

9. <span data-ttu-id="d1e60-186">**요약** 에서 VM 구성을 검토한 다음, **확인** 을 선택하여 배포를 시작합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-186">On **Summary**, review the VM configuration and then select **OK** to start the deployment.</span></span>

    ![Azure Stack Hub 사용자 포털의 구성 요약](media/solution-deployment-guide-hybrid/image6.png)

10. <span data-ttu-id="d1e60-188">새 VM을 만드는 데 다소 시간이 걸립니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-188">It takes some time to create the new VM.</span></span> <span data-ttu-id="d1e60-189">**가상 머신** 에서 VM의 상태를 볼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-189">You can view the STATUS of your VMs in **Virtual machines**.</span></span>

    ![Azure Stack Hub 사용자 포털의 가상 머신 상태](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a><span data-ttu-id="d1e60-191">Azure 및 Azure Stack Hub에서 웹앱 만들기</span><span class="sxs-lookup"><span data-stu-id="d1e60-191">Create web apps in Azure and Azure Stack Hub</span></span>

<span data-ttu-id="d1e60-192">Azure App Service는 웹앱 실행 및 관리를 간소화합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-192">The Azure App Service simplifies running and managing a web app.</span></span> <span data-ttu-id="d1e60-193">Azure Stack Hub는 Azure와 일치하기 때문에 App Service를 두 환경 모두에서 실행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-193">Because Azure Stack Hub is consistent with Azure,  the App Service can run in both environments.</span></span> <span data-ttu-id="d1e60-194">App Service를 사용하여 앱을 호스트합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-194">You'll use the App Service to host your app.</span></span>

### <a name="create-web-apps"></a><span data-ttu-id="d1e60-195">웹앱 만들기</span><span class="sxs-lookup"><span data-stu-id="d1e60-195">Create web apps</span></span>

1. <span data-ttu-id="d1e60-196">[Azure에서 App Service 계획 관리](/azure/app-service/app-service-plan-manage#create-an-app-service-plan)의 지침에 따라 Azure에서 웹앱을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-196">Create a web app in Azure by following the instructions in [Manage an App Service plan in Azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span></span> <span data-ttu-id="d1e60-197">웹앱을 하이브리드 네트워크와 동일한 구독 및 리소스 그룹에 배치해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-197">Make sure you put the web app in the same subscription and resource group as your hybrid network.</span></span>

2. <span data-ttu-id="d1e60-198">Azure Stack Hub에서 이전 단계(1)를 반복합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-198">Repeat the previous step (1) in Azure Stack Hub.</span></span>

### <a name="add-route-for-azure-stack-hub"></a><span data-ttu-id="d1e60-199">Azure Stack Hub에 대한 경로 추가</span><span class="sxs-lookup"><span data-stu-id="d1e60-199">Add route for Azure Stack Hub</span></span>

<span data-ttu-id="d1e60-200">사용자가 앱에 액세스할 수 있도록 Azure Stack Hub의 App Service는 공용 인터넷에서 라우팅할 수 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-200">The App Service on Azure Stack Hub must be routable from the public internet to let users access your app.</span></span> <span data-ttu-id="d1e60-201">Azure Stack Hub를 인터넷에서 액세스할 수 있는 경우 Azure Stack Hub 웹앱의 공용 IP 주소 또는 URL을 적어둡니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-201">If your Azure Stack Hub is accessible from the internet, make a note of the public-facing IP address or URL for the Azure Stack Hub web app.</span></span>

<span data-ttu-id="d1e60-202">ASDK를 사용하는 경우 가상 환경 외부에 App Service가 노출되도록 [정적 NAT 매핑을 구성](/azure-stack/operator/azure-stack-create-vpn-connection-one-node#configure-the-nat-vm-on-each-asdk-for-gateway-traversal)할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-202">If you're using an ASDK, you can [configure a static NAT mapping](/azure-stack/operator/azure-stack-create-vpn-connection-one-node#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) to expose App Service outside the virtual environment.</span></span>

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a><span data-ttu-id="d1e60-203">Azure의 웹앱을 하이브리드 네트워크에 연결</span><span class="sxs-lookup"><span data-stu-id="d1e60-203">Connect a web app in Azure to a hybrid network</span></span>

<span data-ttu-id="d1e60-204">Azure의 웹 프런트 엔드와 Azure Stack Hub의 SQL Server 데이터베이스 간에 연결을 제공하려면 Azure와 Azure Stack Hub 사이의 하이브리드 네트워크에 웹앱이 연결되어 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-204">To provide connectivity between the web front end in Azure and the SQL Server database in Azure Stack Hub, the web app must be connected to the hybrid network between Azure and Azure Stack Hub.</span></span> <span data-ttu-id="d1e60-205">연결이 가능하도록 설정하려면 다음을 수행해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-205">To enable connectivity, you'll have to:</span></span>

- <span data-ttu-id="d1e60-206">지점 및 사이트 간 연결 구성</span><span class="sxs-lookup"><span data-stu-id="d1e60-206">Configure point-to-site connectivity.</span></span>
- <span data-ttu-id="d1e60-207">웹앱 구성</span><span class="sxs-lookup"><span data-stu-id="d1e60-207">Configure the web app.</span></span>
- <span data-ttu-id="d1e60-208">Azure Stack Hub에서 로컬 네트워크 게이트웨이 수정</span><span class="sxs-lookup"><span data-stu-id="d1e60-208">Modify the local network gateway in Azure Stack Hub.</span></span>

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a><span data-ttu-id="d1e60-209">지점 및 사이트 간 연결을 위해 Azure 가상 네트워크 구성</span><span class="sxs-lookup"><span data-stu-id="d1e60-209">Configure the Azure virtual network for point-to-site connectivity</span></span>

<span data-ttu-id="d1e60-210">하이브리드 네트워크의 Azure 쪽에 있는 가상 네트워크 게이트웨이는 지점 및 사이트 간 연결이 Azure App Service와 통합되도록 허용해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-210">The virtual network gateway in the Azure side of the hybrid network must allow point-to-site connections to integrate with Azure App Service.</span></span>

1. <span data-ttu-id="d1e60-211">Azure Portal에서 가상 네트워크 게이트웨이 페이지로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-211">In the Azure portal, go to the virtual network gateway page.</span></span> <span data-ttu-id="d1e60-212">**설정** 에서 **지점 및 사이트 간 구성** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-212">Under **Settings**, select **Point-to-site configuration**.</span></span>

    ![Azure 가상 네트워크 게이트웨이의 지점 및 사이트 간 옵션](media/solution-deployment-guide-hybrid/image8.png)

2. <span data-ttu-id="d1e60-214">**지금 구성** 을 선택하여 지점 및 사이트 간 구성을 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-214">Select **Configure now** to configure point-to-site.</span></span>

    ![Azure 가상 네트워크 게이트웨이에서 지점 및 사이트 간 구성 시작](media/solution-deployment-guide-hybrid/image9.png)

3. <span data-ttu-id="d1e60-216">**지점 및 사이트 간** 구성 페이지에서 **주소 풀** 에 사용하려는 개인 IP 주소 범위를 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-216">On the **Point-to-site** configuration page, enter the private IP address range that you want to use in **Address pool**.</span></span>

   > [!Note]  
   > <span data-ttu-id="d1e60-217">지정한 범위는 하이브리드 네트워크의 글로벌 Azure 또는 Azure Stack Hub 구성 요소의 서브넷에 이미 사용된 주소 범위와 겹치지 않아야 합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-217">Make sure that the range you specify doesn't overlap with any of the address ranges already used by subnets in the global Azure or Azure Stack Hub components of the hybrid network.</span></span>

   <span data-ttu-id="d1e60-218">**터널 종류** 에서 **IKEv2 VPN** 을 선택 취소합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-218">Under **Tunnel Type**, uncheck the **IKEv2 VPN**.</span></span> <span data-ttu-id="d1e60-219">**저장** 을 선택하여 지점 및 사이트 간 구성을 마칩니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-219">Select **Save** to finish configuring point-to-site.</span></span>

   ![Azure 가상 네트워크 게이트웨이의 지점 및 사이트 간 설정](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a><span data-ttu-id="d1e60-221">하이브리드 네트워크와 Azure App Service 앱 통합</span><span class="sxs-lookup"><span data-stu-id="d1e60-221">Integrate the Azure App Service app with the hybrid network</span></span>

1. <span data-ttu-id="d1e60-222">앱을 Azure VNet에 연결하려면 [게이트웨이에 VNet 통합 필요](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration)의 지침을 따르세요.</span><span class="sxs-lookup"><span data-stu-id="d1e60-222">To connect the app to the Azure VNet, follow the instructions in [Gateway required VNet integration](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span></span>

2. <span data-ttu-id="d1e60-223">웹앱을 호스트하는 App Service 계획에 대한 **설정** 으로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-223">Go to **Settings** for the App Service plan hosting the web app.</span></span> <span data-ttu-id="d1e60-224">**설정** 에서 **네트워킹** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-224">In **Settings**, select **Networking**.</span></span>

    ![App Service 계획에 대한 네트워킹 구성](media/solution-deployment-guide-hybrid/image11.png)

3. <span data-ttu-id="d1e60-226">**VNet 통합** 에서 **관리하려면 여기를 클릭** 을 선택하세요.</span><span class="sxs-lookup"><span data-stu-id="d1e60-226">In **VNET Integration**, select **Click here to manage**.</span></span>

    ![App Service 계획에 대한 VNet 통합 관리](media/solution-deployment-guide-hybrid/image12.png)

4. <span data-ttu-id="d1e60-228">구성하려는 VNet을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-228">Select the VNET that you want to configure.</span></span> <span data-ttu-id="d1e60-229">**VNet으로 라우팅되는 IP 주소** 아래에 Azure VNet, Azure Stack Hub VNet, 지점 및 사이트 간 주소 공간의 IP 주소 범위를 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-229">Under **IP ADDRESSES ROUTED TO VNET**, enter the IP address range for the Azure VNet, the Azure Stack Hub VNet, and the point-to-site address spaces.</span></span> <span data-ttu-id="d1e60-230">**저장** 을 선택하여 설정을 확인하고 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-230">Select **Save** to validate and save these settings.</span></span>

    ![Virtual Network 통합에서 라우팅할 IP 주소 범위](media/solution-deployment-guide-hybrid/image13.png)

<span data-ttu-id="d1e60-232">App Service가 Azure VNet과 통합되는 방법에 대해 자세히 알아보려면 [Azure Virtual Network에 앱 통합](/azure/app-service/web-sites-integrate-with-vnet)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="d1e60-232">To learn more about how App Service integrates with Azure VNets, see [Integrate your app with an Azure Virtual Network](/azure/app-service/web-sites-integrate-with-vnet).</span></span>

### <a name="configure-the-azure-stack-hub-virtual-network"></a><span data-ttu-id="d1e60-233">Azure Stack Hub 가상 네트워크 구성</span><span class="sxs-lookup"><span data-stu-id="d1e60-233">Configure the Azure Stack Hub virtual network</span></span>

<span data-ttu-id="d1e60-234">Azure Stack Hub 가상 네트워크의 로컬 네트워크 게이트웨이는 App Service 지점 및 사이트 간 주소 범위에서 트래픽을 라우팅하도록 구성해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-234">The local network gateway in the Azure Stack Hub virtual network needs to be configured to route traffic from the App Service point-to-site address range.</span></span>

1. <span data-ttu-id="d1e60-235">Azure Stack Hub 포털에서 **로컬 네트워크 게이트웨이** 로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-235">In the Azure Stack Hub portal, go to **Local network gateway**.</span></span> <span data-ttu-id="d1e60-236">**설정** 에서 **구성** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-236">Under **Settings**, select **Configuration**.</span></span>

    ![Azure Stack Hub 로컬 네트워크 게이트웨이의 게이트웨이 구성 옵션](media/solution-deployment-guide-hybrid/image14.png)

2. <span data-ttu-id="d1e60-238">**주소 공간** 에 Azure의 가상 네트워크 게이트웨이의 지점 및 사이트 간 주소 범위를 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-238">In **Address space**, enter the point-to-site address range for the virtual network gateway in Azure.</span></span>

    ![Azure Stack Hub 로컬 네트워크 게이트웨이의 지점 및 사이트 간 주소 공간](media/solution-deployment-guide-hybrid/image15.png)

3. <span data-ttu-id="d1e60-240">**저장** 을 선택하여 구성의 유효성을 검사하고 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-240">Select **Save** to validate and save the configuration.</span></span>

## <a name="configure-dns-for-cross-cloud-scaling"></a><span data-ttu-id="d1e60-241">클라우드 간 크기 조정을 위해 DNS 구성</span><span class="sxs-lookup"><span data-stu-id="d1e60-241">Configure DNS for cross-cloud scaling</span></span>

<span data-ttu-id="d1e60-242">클라우드 간 앱에 대해 DNS를 올바르게 구성하면, 웹앱의 글로벌 Azure 및 Azure Stack Hub 인스턴스에 사용자가 액세스할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-242">By properly configuring DNS for cross-cloud apps, users can access the global Azure and Azure Stack Hub instances of your web app.</span></span> <span data-ttu-id="d1e60-243">이 자습서의 DNS 구성을 사용하면 부하가 늘어나거나 감소하는 경우 Azure Traffic Manager가 트래픽을 라우팅할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-243">The DNS configuration for this tutorial also lets Azure Traffic Manager route traffic when the load increases or decreases.</span></span>

<span data-ttu-id="d1e60-244">이 자습서에서는 App Service 도메인이 작동하지 않기 때문에 Azure DNS를 사용하여 DNS를 관리합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-244">This tutorial uses Azure DNS to manage the DNS because App Service domains won't work.</span></span>

### <a name="create-subdomains"></a><span data-ttu-id="d1e60-245">하위 도메인 만들기</span><span class="sxs-lookup"><span data-stu-id="d1e60-245">Create subdomains</span></span>

<span data-ttu-id="d1e60-246">Traffic Manager는 DNS CNAME을 사용하기 때문에 트래픽을 엔드포인트로 적절하게 라우팅하려면 하위 도메인이 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-246">Because Traffic Manager relies on DNS CNAMEs, a subdomain is needed to properly route traffic to endpoints.</span></span> <span data-ttu-id="d1e60-247">DNS 레코드 및 도메인 매핑에 대한 자세한 내용은 [Traffic Manager를 사용하여 도메인 매핑](/azure/app-service/web-sites-traffic-manager-custom-domain-name)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="d1e60-247">For more information about DNS records and domain mapping, see [map domains with Traffic Manager](/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span></span>

<span data-ttu-id="d1e60-248">Azure 엔드포인트의 경우 사용자가 웹앱에 액세스하는 데 사용할 수 있는 하위 도메인을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-248">For the Azure endpoint, you'll create a subdomain that users can use to access your web app.</span></span> <span data-ttu-id="d1e60-249">이 자습서에서는 **app.northwind.com** 을 사용할 수 있지만, 자체 도메인을 기반으로 이 값을 사용자 지정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-249">For this tutorial, can use **app.northwind.com**, but you should customize this value based on your own domain.</span></span>

<span data-ttu-id="d1e60-250">또한 Azure Stack Hub 엔드포인트에 대한 A 레코드로 하위 도메인을 만들어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-250">You'll also need to create a subdomain with an A record for the Azure Stack Hub endpoint.</span></span> <span data-ttu-id="d1e60-251">**azurestack.northwind.com** 을 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-251">You can use **azurestack.northwind.com**.</span></span>

### <a name="configure-a-custom-domain-in-azure"></a><span data-ttu-id="d1e60-252">Azure에서 사용자 지정 도메인 구성</span><span class="sxs-lookup"><span data-stu-id="d1e60-252">Configure a custom domain in Azure</span></span>

1. <span data-ttu-id="d1e60-253">[CNAME을 Azure App Service에 매핑](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record)하여 **app.northwind.com** 호스트 이름을 Azure 웹앱에 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-253">Add the **app.northwind.com** hostname to the Azure web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span>

### <a name="configure-custom-domains-in-azure-stack-hub"></a><span data-ttu-id="d1e60-254">Azure Stack Hub에서 사용자 지정 도메인 구성</span><span class="sxs-lookup"><span data-stu-id="d1e60-254">Configure custom domains in Azure Stack Hub</span></span>

1. <span data-ttu-id="d1e60-255">[A 레코드를 Azure App Service에 매핑](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record)하여 **azurestack.northwind.com** 호스트 이름을 Azure Stack Hub 웹앱에 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-255">Add the **azurestack.northwind.com** hostname to the Azure Stack Hub web app by [mapping an A record to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span></span> <span data-ttu-id="d1e60-256">App Service 앱에 인터넷 라우팅 가능 IP 주소를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-256">Use the internet-routable IP address for the App Service app.</span></span>

2. <span data-ttu-id="d1e60-257">[CNAME을 Azure App Service에 매핑](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record)하여 **app.northwind.com** 호스트 이름을 Azure Stack Hub 웹앱에 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-257">Add the **app.northwind.com** hostname to the Azure Stack Hub web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span> <span data-ttu-id="d1e60-258">이전 단계(1)에서 구성한 호스트 이름을 CNAME의 대상으로 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-258">Use the hostname you configured in the previous step (1) as the target for the CNAME.</span></span>

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a><span data-ttu-id="d1e60-259">클라우드 간 크기 조정을 위해 SSL 인증서 구성</span><span class="sxs-lookup"><span data-stu-id="d1e60-259">Configure SSL certificates for cross-cloud scaling</span></span>

<span data-ttu-id="d1e60-260">웹앱에서 수집한 중요한 데이터가 SQL 데이터베이스에 저장되거나 전송될 때 보안을 유지하는 것이 중요합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-260">It's important to ensure sensitive data collected by your web app is secure in transit to and when stored on the SQL database.</span></span>

<span data-ttu-id="d1e60-261">들어오는 모든 트래픽에 대해 SSL 인증서를 사용하도록 Azure 및 Azure Stack Hub 웹앱을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-261">You'll configure your Azure and Azure Stack Hub web apps to use SSL certificates for all incoming traffic.</span></span>

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a><span data-ttu-id="d1e60-262">Azure 및 Azure Stack Hub에 SSL 추가</span><span class="sxs-lookup"><span data-stu-id="d1e60-262">Add SSL to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="d1e60-263">Azure에 SSL을 추가하려면 다음을 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-263">To add SSL to Azure:</span></span>

1. <span data-ttu-id="d1e60-264">확보한 SSL 인증서가 생성한 하위 도메인에 유효한지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-264">Make sure that the SSL certificate you get is valid for the subdomain you created.</span></span> <span data-ttu-id="d1e60-265">(와일드카드 인증서를 사용해도 됩니다.)</span><span class="sxs-lookup"><span data-stu-id="d1e60-265">(It's okay to use wildcard certificates.)</span></span>

2. <span data-ttu-id="d1e60-266">Azure Portal의 [기존 사용자 지정 SSL 인증서를 Azure Web Apps에 바인딩](/azure/app-service/app-service-web-tutorial-custom-ssl) 문서에서 **SSL 인증서 바인딩** 및 **웹앱 준비** 섹션의 지침을 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-266">In the Azure portal, follow the instructions in the **Prepare your web app** and **Bind your SSL certificate** sections of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span> <span data-ttu-id="d1e60-267">**SSL 형식** 으로 **SNI 기반 SSL** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-267">Select **SNI-based SSL** as the **SSL Type**.</span></span>

3. <span data-ttu-id="d1e60-268">모든 트래픽을 HTTPS 포트로 리디렉션합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-268">Redirect all traffic to the HTTPS port.</span></span> <span data-ttu-id="d1e60-269">[기존 사용자 지정 SSL 인증서를 Azure Web Apps에 바인딩](/azure/app-service/app-service-web-tutorial-custom-ssl) 문서에서 **HTTPS 적용** 섹션의 지침을 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-269">Follow the instructions in the   **Enforce HTTPS** section of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span>

<span data-ttu-id="d1e60-270">Azure Stack Hub에 SSL을 추가하려면 다음을 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-270">To add SSL to Azure Stack Hub:</span></span>

1. <span data-ttu-id="d1e60-271">Azure Stack Hub 포털에서 Azure에 사용한 1-3단계를 반복합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-271">Repeat steps 1-3 that you used for Azure, using the Azure Stack Hub portal.</span></span>

## <a name="configure-and-deploy-the-web-app"></a><span data-ttu-id="d1e60-272">웹앱 구성 및 배포</span><span class="sxs-lookup"><span data-stu-id="d1e60-272">Configure and deploy the web app</span></span>

<span data-ttu-id="d1e60-273">원격 분석을 올바른 Application Insights 인스턴스에 보고하고 올바른 연결 문자열로 웹앱을 구성하도록 앱 코드를 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-273">You'll configure the app code to report telemetry to the correct Application Insights instance and configure the web apps with the right connection strings.</span></span> <span data-ttu-id="d1e60-274">Application Insights에 대해 자세히 알아보려면 [Application Insights란?](/azure/application-insights/app-insights-overview)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="d1e60-274">To learn more about Application Insights, see [What is Application Insights?](/azure/application-insights/app-insights-overview)</span></span>

### <a name="add-application-insights"></a><span data-ttu-id="d1e60-275">Application Insights 추가</span><span class="sxs-lookup"><span data-stu-id="d1e60-275">Add Application Insights</span></span>

1. <span data-ttu-id="d1e60-276">Microsoft Visual Studio에서 웹앱을 엽니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-276">Open your web app in Microsoft Visual Studio.</span></span>

2. <span data-ttu-id="d1e60-277">웹 트래픽이 늘어나거나 감소하는 경우 Application Insights가 경고를 생성하는 데 사용하는 원격 분석을 전송하도록 [Application Insights를 프로젝트에 추가](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications)합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-277">[Add Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) to your project to transmit the telemetry that Application Insights uses to create alerts when web traffic increases or decreases.</span></span>

### <a name="configure-dynamic-connection-strings"></a><span data-ttu-id="d1e60-278">동적 연결 문자열 구성</span><span class="sxs-lookup"><span data-stu-id="d1e60-278">Configure dynamic connection strings</span></span>

<span data-ttu-id="d1e60-279">웹앱의 각 인스턴스는 다른 방법을 사용하여 SQL 데이터베이스에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-279">Each instance of the web app will use a different method to connect to the SQL database.</span></span> <span data-ttu-id="d1e60-280">Azure의 앱은 SQL Server VM의 개인 IP 주소를 사용하고 Azure Stack Hub의 앱은 SQL Server VM의 공용 IP 주소를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-280">The app in Azure uses the private IP address of the SQL Server VM and the app in Azure Stack Hub uses the public IP address of the SQL Server VM.</span></span>

> [!Note]  
> <span data-ttu-id="d1e60-281">Azure Stack Hub 통합 시스템에서 공용 IP 주소는 인터넷 라우팅이 가능하지 않아야 합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-281">On an Azure Stack Hub integrated system, the public IP address shouldn't be internet-routable.</span></span> <span data-ttu-id="d1e60-282">ASDK에서는 공용 IP 주소는 ASDK 외부로 라우팅할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-282">On an ASDK, the public IP address isn't routable outside the ASDK.</span></span>

<span data-ttu-id="d1e60-283">App Service 환경 변수를 사용하여 앱의 각 인스턴스에 다른 연결 문자열을 전달할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-283">You can use App Service environment variables to pass a different connection string to each instance of the app.</span></span>

1. <span data-ttu-id="d1e60-284">Visual Studio에서 앱을 엽니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-284">Open the app in Visual Studio.</span></span>

2. <span data-ttu-id="d1e60-285">Startup.cs를 열고 다음 코드 블록을 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-285">Open Startup.cs and find the following code block:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. <span data-ttu-id="d1e60-286">이전 코드 블록을 다음 코드로 바꿉니다. 이 코드는 *appsettings* 파일에 정의된 연결 문자열을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-286">Replace the previous code block with the following code, which uses a connection string defined in the *appsettings.json* file:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a><span data-ttu-id="d1e60-287">App Service 앱 설정 구성</span><span class="sxs-lookup"><span data-stu-id="d1e60-287">Configure App Service app settings</span></span>

1. <span data-ttu-id="d1e60-288">Azure 및 Azure Stack Hub에 대한 연결 문자열을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-288">Create connection strings for Azure and Azure Stack Hub.</span></span> <span data-ttu-id="d1e60-289">이 문자열은 사용되는 IP 주소를 제외하고 동일해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-289">The strings should be the same, except for the IP addresses that are used.</span></span>

2. <span data-ttu-id="d1e60-290">Azure 및 Azure Stack Hub에서 웹앱의 [앱 설정으로](/azure/app-service/web-sites-configure) 적절한 연결 문자열을 추가합니다(`SQLCONNSTR\_`를 이름의 접두사로 사용).</span><span class="sxs-lookup"><span data-stu-id="d1e60-290">In Azure and Azure Stack Hub, add the appropriate connection string [as an app setting](/azure/app-service/web-sites-configure) in the web app, using `SQLCONNSTR\_` as a prefix in the name.</span></span>

3. <span data-ttu-id="d1e60-291">웹앱 설정을 **저장** 하고 앱을 다시 시작합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-291">**Save** the web app settings and restart the app.</span></span>

## <a name="enable-automatic-scaling-in-global-azure"></a><span data-ttu-id="d1e60-292">글로벌 Azure에서 자동 크기 조정 사용</span><span class="sxs-lookup"><span data-stu-id="d1e60-292">Enable automatic scaling in global Azure</span></span>

<span data-ttu-id="d1e60-293">App Service Environment에서 웹앱을 만드는 경우 하나의 인스턴스로 시작합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-293">When you create your web app in an App Service environment, it starts with one instance.</span></span> <span data-ttu-id="d1e60-294">자동으로 규모를 확장하여 인스턴스를 추가하고 앱에 컴퓨팅 리소스를 더 많이 제공할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-294">You can automatically scale out to add instances to provide more compute resources for your app.</span></span> <span data-ttu-id="d1e60-295">마찬가지로, 자동으로 규모를 감축하고 앱에 필요한 인스턴스 수를 줄일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-295">Similarly, you can automatically scale in and reduce the number of instances your app needs.</span></span>

> [!Note]  
> <span data-ttu-id="d1e60-296">규모 확장 및 규모 감축을 구성하려면 App Service 계획이 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-296">You need to have an App Service plan to configure scale out and scale in.</span></span> <span data-ttu-id="d1e60-297">계획이 없으면 다음 단계를 시작하기 전에 계획을 만드십시오.</span><span class="sxs-lookup"><span data-stu-id="d1e60-297">If you don't have a plan, create one before starting the next steps.</span></span>

### <a name="enable-automatic-scale-out"></a><span data-ttu-id="d1e60-298">자동 규모 확장 사용</span><span class="sxs-lookup"><span data-stu-id="d1e60-298">Enable automatic scale-out</span></span>

1. <span data-ttu-id="d1e60-299">Azure Portal에서 스케일 아웃하려는 사이트의 App Service 요금제를 찾은 다음, **스케일 아웃(App Service 요금제)** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-299">In the Azure portal, find the App Service plan for the sites you want to scale out, and then select **Scale-out (App Service plan)**.</span></span>

    ![Azure App Service 규모 확장](media/solution-deployment-guide-hybrid/image16.png)

2. <span data-ttu-id="d1e60-301">**자동 크기 조정 사용** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-301">Select **Enable autoscale**.</span></span>

    ![Azure App Service에서 자동 크기 조정 사용](media/solution-deployment-guide-hybrid/image17.png)

3. <span data-ttu-id="d1e60-303">**자동 크기 조정 설정 이름** 에 이름을 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-303">Enter a name for **Autoscale Setting Name**.</span></span> <span data-ttu-id="d1e60-304">**기본** 자동 크기 조정 규칙의 경우 **메트릭 기준 크기 조정** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-304">For the **Default** auto scale rule, select **Scale based on a metric**.</span></span> <span data-ttu-id="d1e60-305">**인스턴스 제한** 을 **최소: 1**, **최대: 10**, **기본값: 1** 로 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-305">Set the **Instance limits** to **Minimum: 1**, **Maximum: 10**, and **Default: 1**.</span></span>

    ![Azure App Service에서 자동 크기 조정 구성](media/solution-deployment-guide-hybrid/image18.png)

4. <span data-ttu-id="d1e60-307">**+ 규칙 추가** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-307">Select **+Add a rule**.</span></span>

5. <span data-ttu-id="d1e60-308">**메트릭 원본** 에서 **현재 리소스** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-308">In **Metric Source**, select **Current Resource**.</span></span> <span data-ttu-id="d1e60-309">규칙에 대해 다음 조건 및 동작을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-309">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="d1e60-310">조건</span><span class="sxs-lookup"><span data-stu-id="d1e60-310">Criteria</span></span>

1. <span data-ttu-id="d1e60-311">**시간 집계** 에서 **평균** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-311">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="d1e60-312">**메트릭 이름** 에서 **CPU 백분율** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-312">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="d1e60-313">**연산자** 에서 **보다 큼** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-313">Under **Operator**, select **Greater than**.</span></span>

   - <span data-ttu-id="d1e60-314">**임계값** 을 **50** 으로 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-314">Set the **Threshold** to **50**.</span></span>
   - <span data-ttu-id="d1e60-315">**기간** 을 **10** 으로 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-315">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="d1e60-316">작업</span><span class="sxs-lookup"><span data-stu-id="d1e60-316">Action</span></span>

1. <span data-ttu-id="d1e60-317">**작업** 에서 **다음을 기준으로 개수 늘이기** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-317">Under **Operation**, select **Increase Count by**.</span></span>

2. <span data-ttu-id="d1e60-318">**인스턴스 수** 를 **2** 로 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-318">Set the **Instance Count** to **2**.</span></span>

3. <span data-ttu-id="d1e60-319">**정지 시간** 을 **5** 로 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-319">Set the **Cool down** to **5**.</span></span>

4. <span data-ttu-id="d1e60-320">**추가** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-320">Select **Add**.</span></span>

5. <span data-ttu-id="d1e60-321">**+ 규칙 추가** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-321">Select the **+ Add a rule**.</span></span>

6. <span data-ttu-id="d1e60-322">**메트릭 원본** 에서 **현재 리소스** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-322">In **Metric Source**, select **Current Resource.**</span></span>

   > [!Note]  
   > <span data-ttu-id="d1e60-323">현재 리소스에 App Service 계획의 이름/GUID가 포함되며 **리소스 유형** 및 **리소스** 드롭다운 목록은 사용할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-323">The current resource will contain your App Service plan's name/GUID and the **Resource Type** and **Resource** drop-down lists will be unavailable.</span></span>

### <a name="enable-automatic-scale-in"></a><span data-ttu-id="d1e60-324">자동 규모 감축 사용</span><span class="sxs-lookup"><span data-stu-id="d1e60-324">Enable automatic scale in</span></span>

<span data-ttu-id="d1e60-325">트래픽이 감소하면 Azure 웹앱이 활성 인스턴스 수를 자동으로 줄여서 비용을 절감할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-325">When traffic decreases, the Azure web app can automatically reduce the number of active instances to reduce costs.</span></span> <span data-ttu-id="d1e60-326">이 작업은 규모 확장보다 덜 적극적이며 앱 사용자에게 미치는 영향을 최소화합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-326">This action is less aggressive than scale-out and minimizes the impact on app users.</span></span>

1. <span data-ttu-id="d1e60-327">**기본** 규모 확장 조건으로 이동한 다음, **+ 규칙 추가** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-327">Go to the **Default** scale out condition, then select **+ Add a rule**.</span></span> <span data-ttu-id="d1e60-328">규칙에 대해 다음 조건 및 동작을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-328">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="d1e60-329">조건</span><span class="sxs-lookup"><span data-stu-id="d1e60-329">Criteria</span></span>

1. <span data-ttu-id="d1e60-330">**시간 집계** 에서 **평균** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-330">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="d1e60-331">**메트릭 이름** 에서 **CPU 백분율** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-331">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="d1e60-332">**연산자** 에서 **보다 작음** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-332">Under **Operator**, select **Less than**.</span></span>

   - <span data-ttu-id="d1e60-333">**임계값** 을 **30** 으로 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-333">Set the **Threshold** to **30**.</span></span>
   - <span data-ttu-id="d1e60-334">**기간** 을 **10** 으로 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-334">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="d1e60-335">작업</span><span class="sxs-lookup"><span data-stu-id="d1e60-335">Action</span></span>

1. <span data-ttu-id="d1e60-336">**작업** 에서 **다음을 기준으로 개수 줄이기** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-336">Under **Operation**, select **Decrease Count by**.</span></span>

   - <span data-ttu-id="d1e60-337">**인스턴스 수** 를 **1** 로 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-337">Set the **Instance Count** to **1**.</span></span>
   - <span data-ttu-id="d1e60-338">**정지 시간** 을 **5** 로 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-338">Set the **Cool down** to **5**.</span></span>

2. <span data-ttu-id="d1e60-339">**추가** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-339">Select **Add**.</span></span>

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a><span data-ttu-id="d1e60-340">Traffic Manager 프로필을 만들고 클라우드 간 크기 조정에 맞게 구성</span><span class="sxs-lookup"><span data-stu-id="d1e60-340">Create a Traffic Manager profile and configure cross-cloud scaling</span></span>

<span data-ttu-id="d1e60-341">Azure Portal에서 Traffic Manager 프로필을 만든 다음, 클라우드 간 스케일링이 가능하도록 엔드포인트를 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-341">Create a Traffic Manager profile using the Azure portal, then configure endpoints to enable cross-cloud scaling.</span></span>

### <a name="create-traffic-manager-profile"></a><span data-ttu-id="d1e60-342">Traffic Manager 프로필 만들기</span><span class="sxs-lookup"><span data-stu-id="d1e60-342">Create Traffic Manager profile</span></span>

1. <span data-ttu-id="d1e60-343">**리소스 만들기** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-343">Select **Create a resource**.</span></span>
2. <span data-ttu-id="d1e60-344">**네트워킹** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-344">Select **Networking**.</span></span>
3. <span data-ttu-id="d1e60-345">**Traffic Manager 프로필** 을 선택하고 다음 설정을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-345">Select **Traffic Manager profile** and configure the following settings:</span></span>

   - <span data-ttu-id="d1e60-346">**이름** 에 사용자의 프로필에 사용할 이름을 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-346">In **Name**, enter a name for your profile.</span></span> <span data-ttu-id="d1e60-347">이 이름은 trafficmanager.net 영역에서 **고유해야 하며**, 새 DNS 이름(예: northwindstore.trafficmanager.net)을 만드는 데 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-347">This name **must** be unique in the trafficmanager.net zone and is used to create a new DNS name (for example, northwindstore.trafficmanager.net).</span></span>
   - <span data-ttu-id="d1e60-348">**라우팅 방법** 으로 **가중치** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-348">For **Routing method**, select the **Weighted**.</span></span>
   - <span data-ttu-id="d1e60-349">**구독** 에는 이 프로필을 만들 구독을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-349">For **Subscription**, select the subscription you want to create  this profile in.</span></span>
   - <span data-ttu-id="d1e60-350">**리소스 그룹** 에서 이 프로필의 새 리소스 그룹을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-350">In **Resource Group**, create a new resource group for this profile.</span></span>
   - <span data-ttu-id="d1e60-351">**리소스 그룹 위치** 에서 리소스 그룹의 위치를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-351">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="d1e60-352">이 설정은 리소스 그룹의 위치를 나타내며 전역적으로 배포되는 Traffic Manager 프로필에는 영향을 미치지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-352">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile that's deployed globally.</span></span>

4. <span data-ttu-id="d1e60-353">**만들기** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-353">Select **Create**.</span></span>

    ![Traffic Manager 프로필 만들기](media/solution-deployment-guide-hybrid/image19.png)

   <span data-ttu-id="d1e60-355">Traffic Manager 프로필의 전역 배포가 완료되면 프로필을 만든 리소스 그룹의 리소스 목록에 해당 프로필이 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-355">When the global deployment of your Traffic Manager profile is complete, it's shown in the list of resources for the resource group you created it under.</span></span>

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="d1e60-356">Traffic Manager 엔드포인트 추가</span><span class="sxs-lookup"><span data-stu-id="d1e60-356">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="d1e60-357">만든 Traffic Manager 프로필을 검색합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-357">Search for the Traffic Manager profile you created.</span></span> <span data-ttu-id="d1e60-358">프로필의 리소스 그룹으로 이동한 경우 프로필을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-358">If you navigated to the resource group for the profile, select the profile.</span></span>

2. <span data-ttu-id="d1e60-359">**Traffic Manager 프로필** 의 **설정** 에서 **엔드포인트** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-359">In **Traffic Manager profile**, under **SETTINGS**, select **Endpoints**.</span></span>

3. <span data-ttu-id="d1e60-360">**추가** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-360">Select **Add**.</span></span>

4. <span data-ttu-id="d1e60-361">**엔드포인트 추가** 에서 Azure Stack Hub에 대해 다음 설정을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-361">In **Add endpoint**, use the following settings for Azure Stack Hub:</span></span>

   - <span data-ttu-id="d1e60-362">**형식** 의 경우 **외부 엔드포인트** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-362">For **Type**, select **External endpoint**.</span></span>
   - <span data-ttu-id="d1e60-363">엔드포인트에 사용할 **이름** 을 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-363">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="d1e60-364">**FQDN(정규화된 도메인 이름) 또는 IP** 에는 Azure Stack Hub 웹앱의 외부 URL을 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-364">For **Fully qualified domain name (FQDN) or IP**, enter the external URL for your Azure Stack Hub web app.</span></span>
   - <span data-ttu-id="d1e60-365">**가중치** 의 경우 기본값 **1** 을 유지합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-365">For **Weight**, keep the default, **1**.</span></span> <span data-ttu-id="d1e60-366">가중치를 이렇게 설정하면 모든 트래픽이 정상일 경우 이 엔드포인트로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-366">This weight results in all traffic going to this endpoint if it's healthy.</span></span>
   - <span data-ttu-id="d1e60-367">**사용 안 함으로 추가** 는 선택하지 않은 상태로 유지합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-367">Leave **Add as disabled** unchecked.</span></span>

5. <span data-ttu-id="d1e60-368">**확인** 을 선택하여 Azure Stack Hub 엔드포인트를 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-368">Select **OK** to save the Azure Stack Hub endpoint.</span></span>

<span data-ttu-id="d1e60-369">다음으로 Azure 엔드포인트를 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-369">You'll configure the Azure endpoint next.</span></span>

1. <span data-ttu-id="d1e60-370">**Traffic Manager 프로필** 에서 **엔드포인트** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-370">On **Traffic Manager profile**, select **Endpoints**.</span></span>
2. <span data-ttu-id="d1e60-371">**+추가** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-371">Select **+Add**.</span></span>
3. <span data-ttu-id="d1e60-372">**엔드포인트 추가** 에서 Azure에 대해 다음 설정을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-372">On **Add endpoint**, use the following settings for Azure:</span></span>

   - <span data-ttu-id="d1e60-373">**형식** 에는 **Azure 엔드포인트** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-373">For **Type**, select **Azure endpoint**.</span></span>
   - <span data-ttu-id="d1e60-374">엔드포인트에 사용할 **이름** 을 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-374">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="d1e60-375">**대상 리소스 형식** 에는 **App Service** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-375">For **Target resource type**, select **App Service**.</span></span>
   - <span data-ttu-id="d1e60-376">**대상 리소스** 의 경우 동일한 구독의 Web Apps 목록을 볼 수 있도록 **앱 서비스 선택** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-376">For **Target resource**, select **Choose an app service** to see a list of Web Apps in the same subscription.</span></span>
   - <span data-ttu-id="d1e60-377">**리소스** 에서 첫 번째 엔드포인트로 추가할 앱 서비스를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-377">In **Resource**, pick the App service that you want to add as the first endpoint.</span></span>
   - <span data-ttu-id="d1e60-378">**가중치** 로 **2** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-378">For **Weight**, select **2**.</span></span> <span data-ttu-id="d1e60-379">이렇게 설정하면 기본 엔드포인트가 비정상이거나 트리거되면 트래픽을 리디렉션하는 규칙/경고가 있는 경우 모든 트래픽이 이 엔드포인트로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-379">This setting results in all traffic going to this endpoint if the primary endpoint is unhealthy, or if you have a rule/alert that redirects traffic when triggered.</span></span>
   - <span data-ttu-id="d1e60-380">**사용 안 함으로 추가** 는 선택하지 않은 상태로 유지합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-380">Leave **Add as disabled** unchecked.</span></span>

4. <span data-ttu-id="d1e60-381">**확인** 을 선택하여 Azure 엔드포인트를 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-381">Select **OK** to save the Azure endpoint.</span></span>

<span data-ttu-id="d1e60-382">두 엔드포인트를 모두 구성한 후 **엔드포인트** 를 선택하면 **Traffic Manager 프로필** 에 나열됩니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-382">After both endpoints are configured, they're listed in **Traffic Manager profile** when you select **Endpoints**.</span></span> <span data-ttu-id="d1e60-383">다음 화면 캡처에 있는 예에는 각각 상태 및 구성 정보가 있는 엔드포인트가 두 개 있습니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-383">The example in the following screen capture shows two endpoints, with status and configuration information for each one.</span></span>

![Traffic Manager 프로필의 엔드포인트](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting-in-azure"></a><span data-ttu-id="d1e60-385">Azure에서 Application Insights 모니터링 및 경고 설정</span><span class="sxs-lookup"><span data-stu-id="d1e60-385">Set up Application Insights monitoring and alerting in Azure</span></span>

<span data-ttu-id="d1e60-386">Azure Application Insights를 사용하면 구성한 조건에 따라 앱을 모니터링하고 경고를 보낼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-386">Azure Application Insights lets you monitor your app and send alerts based on conditions you configure.</span></span> <span data-ttu-id="d1e60-387">앱을 사용할 수 없음, 앱에 오류가 발생함, 앱에 성능 문제가 있음 등이 그 예입니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-387">Some examples are: the app is unavailable, is experiencing failures, or is showing performance issues.</span></span>

<span data-ttu-id="d1e60-388">Azure Application Insights 메트릭을 사용하여 경고를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-388">You'll use Azure Application Insights metrics to create alerts.</span></span> <span data-ttu-id="d1e60-389">이러한 경고가 트리거되면, 웹앱의 인스턴스가 규모 확장을 위해 Azure Stack Hub에서 Azure로 자동 전환된 다음, 규모 감축을 위해 Azure Stack Hub로 다시 전환됩니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-389">When these alerts trigger, your web app's instance will automatically switch from Azure Stack Hub to Azure to scale out, and then back to Azure Stack Hub to scale in.</span></span>

### <a name="create-an-alert-from-metrics"></a><span data-ttu-id="d1e60-390">메트릭에서 경고 만들기</span><span class="sxs-lookup"><span data-stu-id="d1e60-390">Create an alert from metrics</span></span>

<span data-ttu-id="d1e60-391">Azure Portal에서 이 자습서의 리소스 그룹으로 이동한 다음, Application Insights 인스턴스를 선택하여 **Application Insights** 를 엽니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-391">In the Azure portal, go to the resource group for this tutorial, and select the Application Insights instance to open **Application Insights**.</span></span>

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

<span data-ttu-id="d1e60-393">이 보기를 사용하여 규모 확장 경고와 규모 감축 경고를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-393">You'll use this view to create a scale-out alert and a scale-in alert.</span></span>

### <a name="create-the-scale-out-alert"></a><span data-ttu-id="d1e60-394">규모 확장 경고 만들기</span><span class="sxs-lookup"><span data-stu-id="d1e60-394">Create the scale-out alert</span></span>

1. <span data-ttu-id="d1e60-395">**구성** 에서 **경고(클래식)** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-395">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="d1e60-396">**메트릭 경고 추가(클래식)** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-396">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="d1e60-397">**규칙 추가** 에서 다음 설정을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-397">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="d1e60-398">**이름** 에 **Burst into Azure Cloud** 를 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-398">For **Name**, enter **Burst into Azure Cloud**.</span></span>
   - <span data-ttu-id="d1e60-399">**설명** 은 선택 사항입니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-399">A **Description** is optional.</span></span>
   - <span data-ttu-id="d1e60-400">**원본** > **경고 대상** 에서 **메트릭** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-400">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="d1e60-401">**조건** 에서 구독, Traffic Manager 프로필의 리소스 그룹, 리소스의 Traffic Manager 프로필 이름을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-401">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="d1e60-402">**메트릭** 에 대해 **요청 빈도** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-402">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="d1e60-403">**조건** 에는 **보다 큼** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-403">For **Condition**, select **Greater than**.</span></span>
6. <span data-ttu-id="d1e60-404">**임계값** 에 **2** 를 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-404">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="d1e60-405">**기간** 에는 **지난 5분** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-405">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="d1e60-406">**다음을 통해 알림** 에서:</span><span class="sxs-lookup"><span data-stu-id="d1e60-406">Under **Notify via**:</span></span>
   - <span data-ttu-id="d1e60-407">**이메일 소유자, 기여자 및 구독자** 확인란을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-407">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="d1e60-408">**추가 관리자 이메일** 에 사용할 이메일 주소를 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-408">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="d1e60-409">메뉴 모음에서 **저장** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-409">On the menu bar, select **Save**.</span></span>

### <a name="create-the-scale-in-alert"></a><span data-ttu-id="d1e60-410">규모 감축 경고 만들기</span><span class="sxs-lookup"><span data-stu-id="d1e60-410">Create the scale-in alert</span></span>

1. <span data-ttu-id="d1e60-411">**구성** 에서 **경고(클래식)** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-411">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="d1e60-412">**메트릭 경고 추가(클래식)** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-412">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="d1e60-413">**규칙 추가** 에서 다음 설정을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-413">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="d1e60-414">**이름** 에 **Scale back into Azure Stack Hub** 를 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-414">For **Name**, enter **Scale back into Azure Stack Hub**.</span></span>
   - <span data-ttu-id="d1e60-415">**설명** 은 선택 사항입니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-415">A **Description** is optional.</span></span>
   - <span data-ttu-id="d1e60-416">**원본** > **경고 대상** 에서 **메트릭** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-416">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="d1e60-417">**조건** 에서 구독, Traffic Manager 프로필의 리소스 그룹, 리소스의 Traffic Manager 프로필 이름을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-417">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="d1e60-418">**메트릭** 에 대해 **요청 빈도** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-418">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="d1e60-419">**조건** 에는 **보다 작음** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-419">For **Condition**, select **Less than**.</span></span>
6. <span data-ttu-id="d1e60-420">**임계값** 에 **2** 를 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-420">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="d1e60-421">**기간** 에는 **지난 5분** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-421">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="d1e60-422">**다음을 통해 알림** 에서:</span><span class="sxs-lookup"><span data-stu-id="d1e60-422">Under **Notify via**:</span></span>
   - <span data-ttu-id="d1e60-423">**이메일 소유자, 기여자 및 구독자** 확인란을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-423">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="d1e60-424">**추가 관리자 이메일** 에 사용할 이메일 주소를 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-424">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="d1e60-425">메뉴 모음에서 **저장** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-425">On the menu bar, select **Save**.</span></span>

<span data-ttu-id="d1e60-426">다음 스크린샷은 규모 확장 및 규모 감축에 대한 경고를 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-426">The following screenshot shows the alerts for scale-out and scale-in.</span></span>

   ![Application Insights 경고(클래식)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a><span data-ttu-id="d1e60-428">Azure와 Azure Stack Hub 간에 트래픽 리디렉션</span><span class="sxs-lookup"><span data-stu-id="d1e60-428">Redirect traffic between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="d1e60-429">Azure와 Azure Stack Hub 간에 웹앱 트래픽 수동 또는 자동 전환을 구성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-429">You can configure manual or automatic switching of your web app traffic between Azure and Azure Stack Hub.</span></span>

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="d1e60-430">Azure와 Azure Stack Hub 간의 수동 전환 구성</span><span class="sxs-lookup"><span data-stu-id="d1e60-430">Configure manual switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="d1e60-431">웹 사이트가 구성한 임계값에 도달하면 경고를 받게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-431">When your web site reaches the thresholds that you configure, you'll receive an alert.</span></span> <span data-ttu-id="d1e60-432">다음 단계를 사용하여 트래픽을 Azure로 수동 리디렉션합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-432">Use the following steps to manually redirect traffic to Azure.</span></span>

1. <span data-ttu-id="d1e60-433">Azure Portal에서 Traffic Manager 프로필을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-433">In the Azure portal, select your Traffic Manager profile.</span></span>

    ![Azure Portal의 Traffic Manager 엔드포인트](media/solution-deployment-guide-hybrid/image20.png)

2. <span data-ttu-id="d1e60-435">**엔드포인트** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-435">Select **Endpoints**.</span></span>
3. <span data-ttu-id="d1e60-436">**Azure 엔드포인트** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-436">Select the **Azure endpoint**.</span></span>
4. <span data-ttu-id="d1e60-437">**상태** 에서 **사용** 을 선택한 다음, **저장** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-437">Under **Status**, select **Enabled**, and then select **Save**.</span></span>

    ![Azure Portal에서 Azure 엔드포인트 사용](media/solution-deployment-guide-hybrid/image23.png)

5. <span data-ttu-id="d1e60-439">Traffic Manager 프로필의 **엔드포인트** 에서 **외부 엔드포인트** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-439">On **Endpoints** for the Traffic Manager profile, select **External endpoint**.</span></span>
6. <span data-ttu-id="d1e60-440">**상태** 에서 **사용 안 함** 을 선택한 다음, **저장** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-440">Under **Status**, select **Disabled**, and then select **Save**.</span></span>

    ![Azure Portal에서 Azure Stack Hub 엔드포인트 사용 안 함](media/solution-deployment-guide-hybrid/image24.png)

<span data-ttu-id="d1e60-442">엔드포인트가 구성된 후에는 앱 트래픽이 Azure Stack Hub 웹앱 대신 Azure 규모 확장 웹앱으로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-442">After the endpoints are configured, app traffic goes to your Azure scale-out web app instead of the Azure Stack Hub web app.</span></span>

 ![Azure 웹앱 트래픽에서 엔드포인트가 변경됨](media/solution-deployment-guide-hybrid/image25.png)

<span data-ttu-id="d1e60-444">흐름을 Azure Stack Hub로 되돌리려면 이전 단계를 사용하여 다음을 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-444">To reverse the flow back to Azure Stack Hub, use the previous steps to:</span></span>

- <span data-ttu-id="d1e60-445">Azure Stack Hub 엔드포인트를 사용하도록 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-445">Enable the Azure Stack Hub endpoint.</span></span>
- <span data-ttu-id="d1e60-446">Azure 엔드포인트를 사용하지 않도록 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-446">Disable the Azure endpoint.</span></span>

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="d1e60-447">Azure와 Azure Stack Hub 간의 자동 전환 구성</span><span class="sxs-lookup"><span data-stu-id="d1e60-447">Configure automatic switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="d1e60-448">앱이 Azure Functions에서 제공하는 [서버리스](https://azure.microsoft.com/overview/serverless-computing/) 환경에서 실행되는 경우 Application Insights 모니터링을 사용할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-448">You can also use Application Insights monitoring if your app runs in a [serverless](https://azure.microsoft.com/overview/serverless-computing/) environment provided by Azure Functions.</span></span>

<span data-ttu-id="d1e60-449">이 시나리오에서는 함수 앱을 호출하는 웹후크를 사용하도록 Application Insights를 구성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-449">In this scenario, you can configure Application Insights to use a webhook that calls a function app.</span></span> <span data-ttu-id="d1e60-450">이 앱은 경고에 대한 응답으로 엔드포인트를 사용하거나 사용하지 않도록 자동으로 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-450">This app automatically enables or disables an endpoint in response to an alert.</span></span>

<span data-ttu-id="d1e60-451">다음 단계를 지침으로 사용하여 자동 트래픽 전환을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-451">Use the following steps as a guide to configure automatic traffic switching.</span></span>

1. <span data-ttu-id="d1e60-452">Azure 함수 앱을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-452">Create an Azure Function app.</span></span>
2. <span data-ttu-id="d1e60-453">HTTP 트리거 함수를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-453">Create an HTTP-triggered function.</span></span>
3. <span data-ttu-id="d1e60-454">Resource Manager, Web Apps 및 Traffic Manager용 Azure SDK를 가져옵니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-454">Import the Azure SDKs for Resource Manager, Web Apps, and Traffic Manager.</span></span>
4. <span data-ttu-id="d1e60-455">다음을 수행하는 코드를 개발합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-455">Develop code to:</span></span>

   - <span data-ttu-id="d1e60-456">Azure 구독을 인증합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-456">Authenticate to your Azure subscription.</span></span>
   - <span data-ttu-id="d1e60-457">Traffic Manager 엔드포인트를 전환하는 매개 변수를 사용하여 Azure 또는 Azure Stack Hub로 트래픽을 전달합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-457">Use a parameter that toggles the Traffic Manager endpoints to direct traffic to Azure or Azure Stack Hub.</span></span>

5. <span data-ttu-id="d1e60-458">코드를 저장하고 Application Insights 경고 규칙 설정의 **웹후크** 섹션에 적절한 매개 변수가 있는 함수 앱의 URL을 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-458">Save your code and add the function app's URL with the appropriate parameters to the **Webhook** section of the Application Insights alert rule settings.</span></span>
6. <span data-ttu-id="d1e60-459">Application Insights 경고가 발생하면 트래픽이 자동으로 리디렉션됩니다.</span><span class="sxs-lookup"><span data-stu-id="d1e60-459">Traffic is automatically redirected when an Application Insights alert fires.</span></span>

## <a name="next-steps"></a><span data-ttu-id="d1e60-460">다음 단계</span><span class="sxs-lookup"><span data-stu-id="d1e60-460">Next steps</span></span>

- <span data-ttu-id="d1e60-461">Azure Cloud 패턴에 대해 자세히 알아보려면 [클라우드 디자인 패턴](/azure/architecture/patterns)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="d1e60-461">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
