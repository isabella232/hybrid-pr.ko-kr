---
title: Azure 및 Azure Stack Hub 앱에 대한 하이브리드 클라우드 ID 구성
description: Azure 및 Azure Stack Hub 앱에 대한 하이브리드 클라우드 ID를 구성하는 방법에 대해 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: cfe2001fcbf91f3ec0d94a7ee257b23ba89065ee
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895350"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a><span data-ttu-id="71d2c-103">Azure 및 Azure Stack Hub 앱에 대한 하이브리드 클라우드 ID 구성</span><span class="sxs-lookup"><span data-stu-id="71d2c-103">Configure hybrid cloud identity for Azure and Azure Stack Hub apps</span></span>

<span data-ttu-id="71d2c-104">Azure 및 Azure Stack Hub 앱에 대한 하이브리드 클라우드 ID를 구성하는 방법에 대해 알아봅니다.</span><span class="sxs-lookup"><span data-stu-id="71d2c-104">Learn how to configure a hybrid cloud identity for your Azure and Azure Stack Hub apps.</span></span>

<span data-ttu-id="71d2c-105">글로벌 Azure와 Azure Stack Hub 모두에서 앱에 대한 액세스 권한을 부여하는 두 가지 옵션이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="71d2c-105">You have two options for granting access to your apps in both global Azure and Azure Stack Hub.</span></span>

 * <span data-ttu-id="71d2c-106">Azure Stack Hub가 인터넷에 지속적으로 연결되어 있는 경우 Azure AD(Azure Active Directory)를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="71d2c-106">When Azure Stack Hub has a continuous connection to the internet, you can use Azure Active Directory (Azure AD).</span></span>
 * <span data-ttu-id="71d2c-107">Azure Stack Hub가 인터넷에서 연결이 끊기면 Azure AD FS(디렉터리 페더레이션 서비스)를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="71d2c-107">When Azure Stack Hub is disconnected from the internet, you can use Azure Directory Federated Services (AD FS).</span></span>

<span data-ttu-id="71d2c-108">서비스 주체를 사용하여 Azure Stack Hub의 Azure Resource Manager를 통해 배포 또는 구성을 위한 Azure Stack Hub 앱에 대한 액세스 권한을 부여합니다.</span><span class="sxs-lookup"><span data-stu-id="71d2c-108">You use service principals to grant access to your Azure Stack Hub apps for deployment or configuration using the Azure Resource Manager in Azure Stack Hub.</span></span>

<span data-ttu-id="71d2c-109">이 솔루션에서는 다음을 수행하는 샘플 환경을 빌드합니다.</span><span class="sxs-lookup"><span data-stu-id="71d2c-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="71d2c-110">글로벌 Azure 및 Azure Stack Hub에서 하이브리드 ID 설정</span><span class="sxs-lookup"><span data-stu-id="71d2c-110">Establish a hybrid identity in global Azure and Azure Stack Hub</span></span>
> - <span data-ttu-id="71d2c-111">Azure Stack Hub API에 액세스하는 토큰을 검색합니다.</span><span class="sxs-lookup"><span data-stu-id="71d2c-111">Retrieve a token to access the Azure Stack Hub API.</span></span>

<span data-ttu-id="71d2c-112">이 솔루션의 단계에 대한 Azure Stack Hub 운영자 권한이 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="71d2c-112">You must have Azure Stack Hub operator permissions for the steps in this solution.</span></span>

> [!Tip]  
> <span data-ttu-id="71d2c-113">![하이브리드 핵심 요소 다이어그램](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="71d2c-113">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="71d2c-114">Microsoft Azure Stack Hub는 Azure의 확장입니다.</span><span class="sxs-lookup"><span data-stu-id="71d2c-114">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="71d2c-115">Azure Stack Hub는 온-프레미스 환경에 클라우드 컴퓨팅의 민첩성과 혁신을 제공하여 어디서나 하이브리드 앱을 빌드하고 배포할 수 있는 유일한 하이브리드 클라우드를 사용하도록 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="71d2c-115">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="71d2c-116">[하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 문서는 하이브리드 앱 디자인, 배포 및 운영에 대한 소프트웨어 품질(배치, 확장성, 가용성, 복원력, 관리 효율성 및 보안)의 핵심 요소를 검토합니다.</span><span class="sxs-lookup"><span data-stu-id="71d2c-116">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="71d2c-117">디자인 고려 사항은 하이브리드 앱 디자인을 최적화하고 프로덕션 환경에서 문제를 최소화하는 데 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="71d2c-117">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a><span data-ttu-id="71d2c-118">포털에서 Azure AD에 대한 서비스 주체 만들기</span><span class="sxs-lookup"><span data-stu-id="71d2c-118">Create a service principal for Azure AD in the portal</span></span>

<span data-ttu-id="71d2c-119">Azure AD를 ID 저장소로 사용하여 Azure Stack Hub를 배포한 경우 Azure에 대해 수행하는 것처럼 서비스 주체를 만들 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="71d2c-119">If you deployed Azure Stack Hub using Azure AD as the identity store, you can create service principals just like you do for Azure.</span></span> <span data-ttu-id="71d2c-120">[앱 ID를 사용하여 리소스에 액세스](/azure-stack/operator/azure-stack-create-service-principals#manage-an-azure-ad-app-identity)는 포털을 통해 단계를 수행하는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="71d2c-120">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals#manage-an-azure-ad-app-identity) shows you how to perform the steps through the portal.</span></span> <span data-ttu-id="71d2c-121">시작하기 전에 [필요한 Azure AD 사용 권한](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions)이 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="71d2c-121">Be sure you have the [required Azure AD permissions](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) before beginning.</span></span>

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a><span data-ttu-id="71d2c-122">PowerShell을 사용하여 AD FS에 대한 서비스 주체 만들기</span><span class="sxs-lookup"><span data-stu-id="71d2c-122">Create a service principal for AD FS using PowerShell</span></span>

<span data-ttu-id="71d2c-123">AD FS를 사용하여 Azure Stack Hub를 배포한 경우 PowerShell을 사용하여 서비스 주체를 만들고, 액세스에 대한 역할을 할당하고, 해당 ID를 사용하여 PowerShell에서 로그인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="71d2c-123">If you deployed Azure Stack Hub with AD FS, you can use PowerShell to create a service principal, assign a role for access, and sign in from PowerShell using that identity.</span></span> <span data-ttu-id="71d2c-124">[앱 ID를 사용하여 리소스에 액세스](/azure-stack/operator/azure-stack-create-service-principals#manage-an-ad-fs-app-identity)는 PowerShell을 사용하여 필요한 단계를 수행하는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="71d2c-124">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals#manage-an-ad-fs-app-identity) shows you how to perform the required steps using PowerShell.</span></span>

## <a name="using-the-azure-stack-hub-api"></a><span data-ttu-id="71d2c-125">Azure Stack Hub API 사용</span><span class="sxs-lookup"><span data-stu-id="71d2c-125">Using the Azure Stack Hub API</span></span>

<span data-ttu-id="71d2c-126">[Azure Stack Hub API](/azure-stack/user/azure-stack-rest-api-use) 솔루션은 Azure Stack Hub API에 액세스하는 토큰을 검색하는 과정을 안내합니다.</span><span class="sxs-lookup"><span data-stu-id="71d2c-126">The [Azure Stack Hub API](/azure-stack/user/azure-stack-rest-api-use)  solution walks you through the process of retrieving a token to access the Azure Stack Hub API.</span></span>

## <a name="connect-to-azure-stack-hub-using-powershell"></a><span data-ttu-id="71d2c-127">PowerShell을 사용하여 Azure Stack Hub에 연결</span><span class="sxs-lookup"><span data-stu-id="71d2c-127">Connect to Azure Stack Hub using PowerShell</span></span>

<span data-ttu-id="71d2c-128">[Azure Stack Hub에서 PowerShell을 시작하고 실행하는](/azure-stack/operator/azure-stack-powershell-install) 빠른 시작에서는 Azure PowerShell을 설치하고 Azure Stack Hub 설치에 연결하는 데 필요한 단계를 안내합니다.</span><span class="sxs-lookup"><span data-stu-id="71d2c-128">The quickstart [to get up and running with PowerShell in Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install) walks you through the steps needed to install Azure PowerShell and connect to your Azure Stack Hub installation.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="71d2c-129">필수 구성 요소</span><span class="sxs-lookup"><span data-stu-id="71d2c-129">Prerequisites</span></span>

<span data-ttu-id="71d2c-130">액세스할 수 있는 구독을 사용하여 Azure AD에 연결된 Azure Stack Hub를 설치해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="71d2c-130">You need an Azure Stack Hub installation connected to Azure AD with a subscription you can access.</span></span> <span data-ttu-id="71d2c-131">Azure Stack Hub를 설치하지 않은 경우 다음 지침을 사용하여 [ASDK(Azure Stack Development Kit)](/azure-stack/asdk/asdk-install)를 설정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="71d2c-131">If you don't have an Azure Stack Hub installation, you can use these instructions to set up an [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install).</span></span>

#### <a name="connect-to-azure-stack-hub-using-code"></a><span data-ttu-id="71d2c-132">코드를 사용하여 Azure Stack Hub에 연결</span><span class="sxs-lookup"><span data-stu-id="71d2c-132">Connect to Azure Stack Hub using code</span></span>

<span data-ttu-id="71d2c-133">코드를 사용하여 Azure Stack Hub에 연결하려면 Azure Resource Manager 엔드포인트 API를 사용하여 Azure Stack Hub 설치를 위한 인증 및 그래프 엔드포인트를 가져옵니다.</span><span class="sxs-lookup"><span data-stu-id="71d2c-133">To connect to Azure Stack Hub using code, use the Azure Resource Manager endpoints API to get the authentication and graph endpoints for your Azure Stack Hub installation.</span></span> <span data-ttu-id="71d2c-134">그런 다음, REST 요청을 사용하여 인증합니다.</span><span class="sxs-lookup"><span data-stu-id="71d2c-134">Then authenticate using REST requests.</span></span> <span data-ttu-id="71d2c-135">[GitHub](https://github.com/shriramnat/HybridARMApplication)에서 샘플 클라이언트 애플리케이션을 찾을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="71d2c-135">You can find a sample client application on [GitHub](https://github.com/shriramnat/HybridARMApplication).</span></span>

>[!Note]
><span data-ttu-id="71d2c-136">선택하는 언어에 대한 Azure SDK가 Azure API 프로필을 지원하지 않는 한 SDK는 Azure Stack Hub에서 작동하지 않을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="71d2c-136">Unless the Azure SDK for your language of choice supports Azure API Profiles, the SDK may not work with Azure Stack Hub.</span></span> <span data-ttu-id="71d2c-137">Azure API 프로필에 대해 자세히 알아보려면 [API 버전 프로필 관리](/azure-stack/user/azure-stack-version-profiles) 문서를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="71d2c-137">To learn more about Azure API Profiles, see the [manage API version profiles](/azure-stack/user/azure-stack-version-profiles) article.</span></span>

## <a name="next-steps"></a><span data-ttu-id="71d2c-138">다음 단계</span><span class="sxs-lookup"><span data-stu-id="71d2c-138">Next steps</span></span>

- <span data-ttu-id="71d2c-139">Azure Stack Hub에서 ID를 처리하는 방법에 대한 자세한 내용은 [Azure Stack Hub에 대한 ID 아키텍처](/azure-stack/operator/azure-stack-identity-architecture)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="71d2c-139">To learn more about how identity is handled in Azure Stack Hub, see [Identity architecture for Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture).</span></span>
- <span data-ttu-id="71d2c-140">Azure Cloud 패턴에 대해 자세히 알아보려면 [클라우드 디자인 패턴](/azure/architecture/patterns)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="71d2c-140">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
