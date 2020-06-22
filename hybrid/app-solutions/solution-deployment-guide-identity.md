---
title: Azure 및 Azure Stack Hub 앱에 대 한 하이브리드 클라우드 id 구성
description: Azure 및 Azure Stack Hub 앱에 대 한 하이브리드 클라우드 id를 구성 하는 방법에 대해 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 1b3c683dd3e4a68413f83fd3cc129d6e6f594e1b
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911316"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a><span data-ttu-id="357e6-103">Azure 및 Azure Stack Hub 앱에 대 한 하이브리드 클라우드 id 구성</span><span class="sxs-lookup"><span data-stu-id="357e6-103">Configure hybrid cloud identity for Azure and Azure Stack Hub apps</span></span>

<span data-ttu-id="357e6-104">Azure 및 Azure Stack Hub 앱에 대 한 하이브리드 클라우드 id를 구성 하는 방법에 대해 알아봅니다.</span><span class="sxs-lookup"><span data-stu-id="357e6-104">Learn how to configure a hybrid cloud identity for your Azure and Azure Stack Hub apps.</span></span>

<span data-ttu-id="357e6-105">글로벌 Azure와 Azure Stack Hub 모두에서 앱에 대 한 액세스 권한을 부여 하는 두 가지 옵션이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="357e6-105">You have two options for granting access to your apps in both global Azure and Azure Stack Hub.</span></span>

 * <span data-ttu-id="357e6-106">Azure Stack 허브가 인터넷에 지속적으로 연결 되어 있는 경우 Azure Active Directory (Azure AD)를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="357e6-106">When Azure Stack Hub has a continuous connection to the internet, you can use Azure Active Directory (Azure AD).</span></span>
 * <span data-ttu-id="357e6-107">Azure Stack 허브가 인터넷에서 연결이 끊기면 Azure AD FS (디렉터리 페더레이션 서비스)를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="357e6-107">When Azure Stack Hub is disconnected from the internet, you can use Azure Directory Federated Services (AD FS).</span></span>

<span data-ttu-id="357e6-108">서비스 주체를 사용 하 여 Azure Stack 허브의 Azure Resource Manager를 사용 하 여 배포 또는 구성에 대 한 Azure Stack 허브 앱에 대 한 액세스 권한을 부여 합니다.</span><span class="sxs-lookup"><span data-stu-id="357e6-108">You use service principals to grant access to your Azure Stack Hub apps for deployment or configuration using the Azure Resource Manager in Azure Stack Hub.</span></span>

<span data-ttu-id="357e6-109">이 솔루션에서는 다음을 수행 하는 샘플 환경을 빌드합니다.</span><span class="sxs-lookup"><span data-stu-id="357e6-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="357e6-110">글로벌 Azure 및 Azure Stack 허브에서 하이브리드 id 설정</span><span class="sxs-lookup"><span data-stu-id="357e6-110">Establish a hybrid identity in global Azure and Azure Stack Hub</span></span>
> - <span data-ttu-id="357e6-111">Azure Stack 허브 API에 액세스 하는 토큰을 검색 합니다.</span><span class="sxs-lookup"><span data-stu-id="357e6-111">Retrieve a token to access the Azure Stack Hub API.</span></span>

<span data-ttu-id="357e6-112">이 솔루션의 단계에 대 한 Azure Stack 허브 운영자 권한이 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="357e6-112">You must have Azure Stack Hub operator permissions for the steps in this solution.</span></span>

> [!Tip]  
> <span data-ttu-id="357e6-113">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="357e6-113">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="357e6-114">Microsoft Azure Stack 허브는 Azure의 확장입니다.</span><span class="sxs-lookup"><span data-stu-id="357e6-114">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="357e6-115">Azure Stack 허브는 온-프레미스 환경에 클라우드 컴퓨팅의 민첩성과 혁신을 제공 하 여 어디서 나 하이브리드 앱을 빌드하고 배포할 수 있는 유일한 하이브리드 클라우드를 사용 하도록 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="357e6-115">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="357e6-116">[하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 문서에서는 하이브리드 앱을 디자인, 배포 및 운영 하기 위한 소프트웨어 품질 (배치, 확장성, 가용성, 복원 력, 관리 효율성 및 보안)의 핵심 요소을 검토 합니다.</span><span class="sxs-lookup"><span data-stu-id="357e6-116">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="357e6-117">디자인 고려 사항은 하이브리드 앱 디자인을 최적화 하 고 프로덕션 환경에서 문제를 최소화 하는 데 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="357e6-117">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a><span data-ttu-id="357e6-118">포털에서 Azure AD에 대 한 서비스 주체 만들기</span><span class="sxs-lookup"><span data-stu-id="357e6-118">Create a service principal for Azure AD in the portal</span></span>

<span data-ttu-id="357e6-119">Azure AD를 id 저장소로 사용 하 여 Azure Stack 허브를 배포한 경우 Azure에 대해 수행 하는 것 처럼 서비스 사용자를 만들 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="357e6-119">If you deployed Azure Stack Hub using Azure AD as the identity store, you can create service principals just like you do for Azure.</span></span> <span data-ttu-id="357e6-120">[앱 id를 사용 하 여 리소스에 액세스](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) 는 포털을 통해 단계를 수행 하는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="357e6-120">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) shows you how to perform the steps through the portal.</span></span> <span data-ttu-id="357e6-121">시작 하기 전에 [필요한 AZURE AD 권한이](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="357e6-121">Be sure you have the [required Azure AD permissions](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) before beginning.</span></span>

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a><span data-ttu-id="357e6-122">PowerShell을 사용 하 여 AD FS에 대 한 서비스 주체 만들기</span><span class="sxs-lookup"><span data-stu-id="357e6-122">Create a service principal for AD FS using PowerShell</span></span>

<span data-ttu-id="357e6-123">AD FS를 사용 하 Azure Stack 허브를 배포한 경우 PowerShell을 사용 하 여 서비스 주체를 만들고, 액세스에 역할을 할당 하 고, 해당 id를 사용 하 여 PowerShell에서 로그인 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="357e6-123">If you deployed Azure Stack Hub with AD FS, you can use PowerShell to create a service principal, assign a role for access, and sign in from PowerShell using that identity.</span></span> <span data-ttu-id="357e6-124">[앱 id를 사용 하 여 리소스에 액세스](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) 하는 방법은 PowerShell을 사용 하 여 필요한 단계를 수행 하는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="357e6-124">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) shows you how to perform the required steps using PowerShell.</span></span>

## <a name="using-the-azure-stack-hub-api"></a><span data-ttu-id="357e6-125">Azure Stack 허브 API 사용</span><span class="sxs-lookup"><span data-stu-id="357e6-125">Using the Azure Stack Hub API</span></span>

<span data-ttu-id="357e6-126">[Azure Stack HUB api](/azure-stack/user/azure-stack-rest-api-use.md) 솔루션은 AZURE STACK 허브 api에 액세스 하는 토큰을 검색 하는 과정을 안내 합니다.</span><span class="sxs-lookup"><span data-stu-id="357e6-126">The [Azure Stack Hub API](/azure-stack/user/azure-stack-rest-api-use.md)  solution walks you through the process of retrieving a token to access the Azure Stack Hub API.</span></span>

## <a name="connect-to-azure-stack-hub-using-powershell"></a><span data-ttu-id="357e6-127">PowerShell을 사용 하 여 Azure Stack 허브에 연결</span><span class="sxs-lookup"><span data-stu-id="357e6-127">Connect to Azure Stack Hub using PowerShell</span></span>

<span data-ttu-id="357e6-128">[Azure Stack hub에서 PowerShell을 시작 하 고 실행 하](/azure-stack/operator/azure-stack-powershell-install.md) 는 빠른 시작은 Azure PowerShell를 설치 하 고 Azure Stack 허브 설치에 연결 하는 데 필요한 단계를 안내 합니다.</span><span class="sxs-lookup"><span data-stu-id="357e6-128">The quickstart [to get up and running with PowerShell in Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install.md) walks you through the steps needed to install Azure PowerShell and connect to your Azure Stack Hub installation.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="357e6-129">필수 조건</span><span class="sxs-lookup"><span data-stu-id="357e6-129">Prerequisites</span></span>

<span data-ttu-id="357e6-130">액세스할 수 있는 구독을 사용 하 여 Azure AD에 연결 된 Azure Stack 허브를 설치 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="357e6-130">You need an Azure Stack Hub installation connected to Azure AD with a subscription you can access.</span></span> <span data-ttu-id="357e6-131">Azure Stack 허브를 설치 하지 않은 경우 다음 지침을 사용 하 여 [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install.md)를 설정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="357e6-131">If you don't have an Azure Stack Hub installation, you can use these instructions to set up an [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install.md).</span></span>

#### <a name="connect-to-azure-stack-hub-using-code"></a><span data-ttu-id="357e6-132">코드를 사용 하 여 Azure Stack 허브에 연결</span><span class="sxs-lookup"><span data-stu-id="357e6-132">Connect to Azure Stack Hub using code</span></span>

<span data-ttu-id="357e6-133">코드를 사용 하 여 Azure Stack 허브에 연결 하려면 Azure Resource Manager 끝점 API를 사용 하 여 Azure Stack 허브 설치를 위한 인증 및 그래프 끝점을 가져옵니다.</span><span class="sxs-lookup"><span data-stu-id="357e6-133">To connect to Azure Stack Hub using code, use the Azure Resource Manager endpoints API to get the authentication and graph endpoints for your Azure Stack Hub installation.</span></span> <span data-ttu-id="357e6-134">그런 다음 REST 요청을 사용 하 여 인증 합니다.</span><span class="sxs-lookup"><span data-stu-id="357e6-134">Then authenticate using REST requests.</span></span> <span data-ttu-id="357e6-135">[GitHub](https://github.com/shriramnat/HybridARMApplication)에서 샘플 클라이언트 응용 프로그램을 찾을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="357e6-135">You can find a sample client application on [GitHub](https://github.com/shriramnat/HybridARMApplication).</span></span>

>[!Note]
><span data-ttu-id="357e6-136">선택 하는 언어에 대 한 Azure SDK가 Azure API 프로필을 지원 하지 않는 한 SDK는 Azure Stack 허브에서 작동 하지 않을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="357e6-136">Unless the Azure SDK for your language of choice supports Azure API Profiles, the SDK may not work with Azure Stack Hub.</span></span> <span data-ttu-id="357e6-137">Azure API 프로필에 대해 자세히 알아보려면 [api 버전 프로필 관리](/azure-stack/user/azure-stack-version-profiles.md) 문서를 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="357e6-137">To learn more about Azure API Profiles, see the [manage API version profiles](/azure-stack/user/azure-stack-version-profiles.md) article.</span></span>

## <a name="next-steps"></a><span data-ttu-id="357e6-138">다음 단계</span><span class="sxs-lookup"><span data-stu-id="357e6-138">Next steps</span></span>

- <span data-ttu-id="357e6-139">Azure Stack Hub에서 id를 처리 하는 방법에 대 한 자세한 내용은 [Azure Stack 허브에 대 한 id 아키텍처](/azure-stack/operator/azure-stack-identity-architecture.md)를 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="357e6-139">To learn more about how identity is handled in Azure Stack Hub, see [Identity architecture for Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture.md).</span></span>
- <span data-ttu-id="357e6-140">Azure Cloud 패턴에 대해 자세히 알아보려면 [클라우드 디자인 패턴](https://docs.microsoft.com/azure/architecture/patterns)을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="357e6-140">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](https://docs.microsoft.com/azure/architecture/patterns).</span></span>
