---
title: Azure 및 Azure Stack Hub에서 AI 기반 footfall 검색 솔루션 배포
description: Azure 및 Azure Stack Hub를 사용 하 여 소매 매장에서 방문자 트래픽을 분석 하기 위한 AI 기반 footfall 검색 솔루션을 배포 하는 방법에 대해 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 6913cc522da447092dad0af24e148a3b2576495c
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910896"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a><span data-ttu-id="e467b-103">Azure 및 Azure Stack Hub를 사용 하 여 AI 기반 footfall 검색 솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="e467b-103">Deploy an AI-based footfall detection solution using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="e467b-104">이 문서에서는 Azure, Azure Stack Hub 및 Custom Vision AI Dev Kit를 사용 하 여 실제 작업에서 통찰력을 생성 하는 AI 기반 솔루션을 배포 하는 방법을 설명 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-104">This article describes how to deploy an AI-based solution that generates insights from real world actions by using Azure, Azure Stack Hub, and the Custom Vision AI Dev Kit.</span></span>

<span data-ttu-id="e467b-105">이 솔루션에서는 다음 방법에 대해 알아봅니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-105">In this solution, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="e467b-106">에 지에 클라우드 네이티브 응용 프로그램 번들 (CNAB)을 배포 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-106">Deploy Cloud Native Application Bundles (CNAB) at the edge.</span></span> 
> - <span data-ttu-id="e467b-107">클라우드 경계를 확장 하는 앱을 배포 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-107">Deploy an app that spans cloud boundaries.</span></span>
> - <span data-ttu-id="e467b-108">Edge에서 유추를 위해 Custom Vision AI Dev Kit를 사용 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-108">Use the Custom Vision AI Dev Kit for inference at the edge.</span></span>

> [!Tip]  
> <span data-ttu-id="e467b-109">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="e467b-109">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="e467b-110">Microsoft Azure Stack 허브는 Azure의 확장입니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-110">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="e467b-111">Azure Stack 허브는 온-프레미스 환경에 클라우드 컴퓨팅의 민첩성과 혁신을 제공 하 여 어디서 나 하이브리드 앱을 빌드하고 배포할 수 있는 유일한 하이브리드 클라우드를 사용 하도록 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-111">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="e467b-112">[하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 문서에서는 하이브리드 앱을 디자인, 배포 및 운영 하기 위한 소프트웨어 품질 (배치, 확장성, 가용성, 복원 력, 관리 효율성 및 보안)의 핵심 요소을 검토 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-112">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="e467b-113">디자인 고려 사항은 하이브리드 앱 디자인을 최적화 하 고 프로덕션 환경에서 문제를 최소화 하는 데 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-113">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="e467b-114">필수 조건</span><span class="sxs-lookup"><span data-stu-id="e467b-114">Prerequisites</span></span>

<span data-ttu-id="e467b-115">이 배포 가이드를 시작 하기 전에 다음을 확인 하십시오.</span><span class="sxs-lookup"><span data-stu-id="e467b-115">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="e467b-116">[Footfall 검색 패턴](pattern-retail-footfall-detection.md) 항목을 검토 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-116">Review the [Footfall detection pattern](pattern-retail-footfall-detection.md) topic.</span></span>
- <span data-ttu-id="e467b-117">다음을 사용 하 여 Azure Stack Development Kit (ASDK) 또는 Azure Stack 허브 통합 시스템 인스턴스에 대 한 사용자 액세스 권한을 얻습니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-117">Obtain user access to an Azure Stack Development Kit (ASDK) or Azure Stack Hub integrated system instance, with:</span></span>
  - <span data-ttu-id="e467b-118">[Azure App Service Azure Stack Hub 리소스 공급자가](/azure-stack/operator/azure-stack-app-service-overview.md) 설치 되었습니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-118">The [Azure App Service on Azure Stack Hub resource provider](/azure-stack/operator/azure-stack-app-service-overview.md) installed.</span></span> <span data-ttu-id="e467b-119">운영자가 Azure Stack 허브 인스턴스에 액세스 하거나 관리자에 게 문의 하 여를 설치 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-119">You need operator access to your Azure Stack Hub instance, or work with your administrator to install.</span></span>
  - <span data-ttu-id="e467b-120">App Service 및 저장소 할당량을 제공 하는 제품에 대 한 구독입니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-120">A subscription to an offer that provides App Service and Storage quota.</span></span> <span data-ttu-id="e467b-121">제품을 만들려면 운영자 액세스가 필요 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-121">You need operator access to create an offer.</span></span>
- <span data-ttu-id="e467b-122">Azure 구독에 대 한 액세스 권한을 얻습니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-122">Obtain access to an Azure subscription.</span></span>
  - <span data-ttu-id="e467b-123">Azure 구독이 없는 경우 시작 하기 전에 [무료 평가판 계정](https://azure.microsoft.com/free/) 에 등록 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-123">If you don't have an Azure subscription, sign up for a [free trial account](https://azure.microsoft.com/free/) before you begin.</span></span>
- <span data-ttu-id="e467b-124">디렉터리에 두 개의 서비스 주체를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-124">Create two service principals in your directory:</span></span>
  - <span data-ttu-id="e467b-125">Azure 구독 범위에서 액세스와 함께 Azure 리소스에서 사용 하도록 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-125">One set up for use with Azure resources, with access at the Azure subscription scope.</span></span>
  - <span data-ttu-id="e467b-126">Azure Stack 허브 구독 범위에서 액세스 권한이 있는 Azure Stack 허브 리소스와 함께 사용 하도록 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-126">One set up for use with Azure Stack Hub resources, with access at the Azure Stack Hub subscription scope.</span></span>
  - <span data-ttu-id="e467b-127">서비스 주체를 만들고 액세스 권한을 부여 하는 방법에 대 한 자세한 내용은 [앱 id를 사용 하 여 리소스에 액세스](/azure-stack/operator/azure-stack-create-service-principals.md)를 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="e467b-127">To learn more about creating service principals and authorizing access, see [Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md).</span></span> <span data-ttu-id="e467b-128">Azure CLI를 사용 하려면 Azure CLI를 사용 하 [여 Azure 서비스 주체 만들기](https://docs.microsoft.com/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest)를 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="e467b-128">If you prefer to use Azure CLI, see [Create an Azure service principal with Azure CLI](https://docs.microsoft.com/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest).</span></span>
- <span data-ttu-id="e467b-129">Azure 또는 Azure Stack 허브에 Azure Cognitive Services를 배포 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-129">Deploy Azure Cognitive Services in Azure or Azure Stack Hub.</span></span>
  - <span data-ttu-id="e467b-130">먼저 [Cognitive Services에 대해 자세히 알아보세요](https://azure.microsoft.com/services/cognitive-services/).</span><span class="sxs-lookup"><span data-stu-id="e467b-130">First, [learn more about Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).</span></span>
  - <span data-ttu-id="e467b-131">그런 다음 [Azure Stack hub에 Azure Cognitive Services 배포](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) 를 방문 하 여 Azure Stack 허브에 Cognitive Services을 배포 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-131">Then visit [Deploy Azure Cognitive Services to Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) to deploy Cognitive Services on Azure Stack Hub.</span></span> <span data-ttu-id="e467b-132">먼저 미리 보기에 대 한 액세스를 등록 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-132">You first need to sign up for access to the preview.</span></span>
- <span data-ttu-id="e467b-133">구성 되지 않은 Azure Custom Vision AI Dev Kit를 복제 하거나 다운로드 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-133">Clone or download an unconfigured Azure Custom Vision AI Dev Kit.</span></span> <span data-ttu-id="e467b-134">자세한 내용은 [비전 AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/)를 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="e467b-134">For details, see the [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span></span>
- <span data-ttu-id="e467b-135">Power BI 계정에 등록 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-135">Sign up for a Power BI account.</span></span>
- <span data-ttu-id="e467b-136">Azure Cognitive Services Face API 구독 키 및 끝점 URL입니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-136">An Azure Cognitive Services Face API subscription key and endpoint URL.</span></span> <span data-ttu-id="e467b-137">[Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) 체험 평가판을 사용 하 여 두 가지를 모두 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-137">You can get both with the [Try Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) free trial.</span></span> <span data-ttu-id="e467b-138">또는 [Cognitive Services 계정 만들기](/azure/cognitive-services/cognitive-services-apis-create-account)의 지침을 따르세요.</span><span class="sxs-lookup"><span data-stu-id="e467b-138">Or, follow the instructions in [Create a Cognitive Services account](/azure/cognitive-services/cognitive-services-apis-create-account).</span></span>
- <span data-ttu-id="e467b-139">다음 개발 리소스를 설치 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-139">Install the following development resources:</span></span>
  - [<span data-ttu-id="e467b-140">Azure CLI 2.0</span><span class="sxs-lookup"><span data-stu-id="e467b-140">Azure CLI 2.0</span></span>](/azure-stack/user/azure-stack-version-profiles-azurecli2.md)
  - [<span data-ttu-id="e467b-141">Docker CE</span><span class="sxs-lookup"><span data-stu-id="e467b-141">Docker CE</span></span>](https://hub.docker.com/search/?type=edition&offering=community)
  - <span data-ttu-id="e467b-142">[Porter](https://porter.sh/).</span><span class="sxs-lookup"><span data-stu-id="e467b-142">[Porter](https://porter.sh/).</span></span> <span data-ttu-id="e467b-143">Porter를 사용 하 여 제공 된 CNAB 번들 매니페스트를 사용 하 여 클라우드 앱을 배포 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-143">You use Porter to deploy cloud apps using CNAB bundle manifests that are provided for you.</span></span>
  - [<span data-ttu-id="e467b-144">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="e467b-144">Visual Studio Code</span></span>](https://code.visualstudio.com/)
  - [<span data-ttu-id="e467b-145">Visual Studio Code 용 Azure IoT 도구</span><span class="sxs-lookup"><span data-stu-id="e467b-145">Azure IoT Tools for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [<span data-ttu-id="e467b-146">Visual Studio Code 용 Python 확장</span><span class="sxs-lookup"><span data-stu-id="e467b-146">Python extension for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [<span data-ttu-id="e467b-147">Python</span><span class="sxs-lookup"><span data-stu-id="e467b-147">Python</span></span>](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a><span data-ttu-id="e467b-148">하이브리드 클라우드 앱 배포</span><span class="sxs-lookup"><span data-stu-id="e467b-148">Deploy the hybrid cloud app</span></span>

<span data-ttu-id="e467b-149">먼저 Porter CLI를 사용 하 여 자격 증명 집합을 생성 한 다음 클라우드 앱을 배포 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-149">First you use the Porter CLI to generate a credential set, then deploy the cloud app.</span></span>  

1. <span data-ttu-id="e467b-150">에서 솔루션 샘플 코드를 복제 하거나 다운로드 https://github.com/azure-samples/azure-intelligent-edge-patterns 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-150">Clone or download the solution sample code from https://github.com/azure-samples/azure-intelligent-edge-patterns.</span></span> 

1. <span data-ttu-id="e467b-151">Porter는 앱의 배포를 자동화 하는 자격 증명 집합을 생성 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-151">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="e467b-152">자격 증명 생성 명령을 실행 하기 전에 다음을 사용할 수 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-152">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="e467b-153">서비스 주체 ID, 키 및 테 넌 트 DNS를 포함 하 여 Azure 리소스에 액세스 하기 위한 서비스 주체입니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-153">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="e467b-154">Azure 구독에 대 한 구독 ID입니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-154">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="e467b-155">서비스 주체 ID, 키 및 테 넌 트 DNS를 포함 하 여 Azure Stack 허브 리소스에 액세스 하기 위한 서비스 주체입니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-155">A service principal for accessing Azure Stack Hub resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="e467b-156">Azure Stack Hub 구독의 구독 ID입니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-156">The subscription ID for your Azure Stack Hub subscription.</span></span>
    - <span data-ttu-id="e467b-157">Azure Cognitive Services Face API 키와 리소스 끝점 URL입니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-157">Your Azure Cognitive Services Face API key and resource endpoint URL.</span></span>

1. <span data-ttu-id="e467b-158">Porter 자격 증명 생성 프로세스를 실행 하 고 프롬프트를 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-158">Run the Porter credential generation process and follow the prompts:</span></span>

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. <span data-ttu-id="e467b-159">또한 Porter를 실행 하려면 매개 변수 집합이 필요 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-159">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="e467b-160">매개 변수 텍스트 파일을 만들고 다음 이름/값 쌍을 입력 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-160">Create a parameter text file and enter the following name/value pairs.</span></span> <span data-ttu-id="e467b-161">필요한 값에 대 한 지원이 필요한 경우 Azure Stack 허브 관리자에 게 문의 하세요.</span><span class="sxs-lookup"><span data-stu-id="e467b-161">Ask your Azure Stack Hub administrator if you need assistance with any of the required values.</span></span>

   > [!NOTE] 
   > <span data-ttu-id="e467b-162">`resource suffix`값은 배포 리소스의 이름이 Azure 전체에 고유한 지 확인 하는 데 사용 됩니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-162">The `resource suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="e467b-163">문자 및 숫자의 고유 문자열 이어야 하며 8 자이 하 여야 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-163">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    azure_stack_tenant_arm="Your Azure Stack Hub tenant endpoint"
    azure_stack_storage_suffix="Your Azure Stack Hub storage suffix"
    azure_stack_keyvault_suffix="Your Azure Stack Hub keyVault suffix"
    resource_suffix="A unique string to identify your deployment"
    azure_location="A valid Azure region"
    azure_stack_location="Your Azure Stack Hub location identifier"
    powerbi_display_name="Your first and last name"
    powerbi_principal_name="Your Power BI account email address"
    ```
   <span data-ttu-id="e467b-164">텍스트 파일을 저장 하 고 해당 경로를 적어 둡니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-164">Save the text file and make a note of its path.</span></span>

1. <span data-ttu-id="e467b-165">이제 Porter을 사용 하 여 하이브리드 클라우드 앱을 배포할 준비가 되었습니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-165">You're now ready to deploy the hybrid cloud app using Porter.</span></span> <span data-ttu-id="e467b-166">Azure에 배포 되 고 Azure Stack 허브에 리소스가 배포 되는 경우 설치 명령을 실행 하 고 감시 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-166">Run the install command and watch as resources are deployed to Azure and Azure Stack Hub:</span></span>

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. <span data-ttu-id="e467b-167">배포가 완료 되 면 다음 값을 기록해 둡니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-167">Once deployment is complete, make note of the following values:</span></span>
    - <span data-ttu-id="e467b-168">카메라의 연결 문자열입니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-168">The camera's connection string.</span></span>
    - <span data-ttu-id="e467b-169">이미지 저장소 계정 연결 문자열입니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-169">The image storage account connection string.</span></span>
    - <span data-ttu-id="e467b-170">리소스 그룹 이름입니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-170">The resource group names.</span></span>

## <a name="prepare-the-custom-vision-ai-devkit"></a><span data-ttu-id="e467b-171">Custom Vision AI DevKit 준비</span><span class="sxs-lookup"><span data-stu-id="e467b-171">Prepare the Custom Vision AI DevKit</span></span>

<span data-ttu-id="e467b-172">다음으로, [비전 Ai devkit 빠른](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/)시작에 표시 된 대로 Custom Vision Ai Dev kit를 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-172">Next, set up the Custom Vision AI Dev Kit as shown in the [Vision AI DevKit quickstart](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span></span> <span data-ttu-id="e467b-173">또한 이전 단계에서 제공 된 연결 문자열을 사용 하 여 카메라를 설정 하 고 테스트 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-173">You also set up and test your camera, using the connection string provided in the previous step.</span></span>

## <a name="deploy-the-camera-app"></a><span data-ttu-id="e467b-174">카메라 앱 배포</span><span class="sxs-lookup"><span data-stu-id="e467b-174">Deploy the camera app</span></span>

<span data-ttu-id="e467b-175">Porter CLI를 사용 하 여 자격 증명 집합을 생성 한 다음 카메라 앱을 배포 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-175">Use the Porter CLI to generate a credential set, then deploy the camera app.</span></span>

1. <span data-ttu-id="e467b-176">Porter는 앱의 배포를 자동화 하는 자격 증명 집합을 생성 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-176">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="e467b-177">자격 증명 생성 명령을 실행 하기 전에 다음을 사용할 수 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-177">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="e467b-178">서비스 주체 ID, 키 및 테 넌 트 DNS를 포함 하 여 Azure 리소스에 액세스 하기 위한 서비스 주체입니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-178">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="e467b-179">Azure 구독에 대 한 구독 ID입니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-179">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="e467b-180">클라우드 앱을 배포할 때 제공 되는 이미지 저장소 계정 연결 문자열입니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-180">The image storage account connection string provided when you deployed the cloud app.</span></span>

1. <span data-ttu-id="e467b-181">Porter 자격 증명 생성 프로세스를 실행 하 고 프롬프트를 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-181">Run the Porter credential generation process and follow the prompts:</span></span>

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. <span data-ttu-id="e467b-182">또한 Porter를 실행 하려면 매개 변수 집합이 필요 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-182">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="e467b-183">매개 변수 텍스트 파일을 만들고 다음 텍스트를 입력 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-183">Create a parameter text file and enter the following text.</span></span> <span data-ttu-id="e467b-184">필요한 값 중 일부를 모르는 경우 Azure Stack 허브 관리자에 게 문의 하세요.</span><span class="sxs-lookup"><span data-stu-id="e467b-184">Ask your Azure Stack Hub administrator if you don't know some of the required values.</span></span>

    > [!NOTE]
    > <span data-ttu-id="e467b-185">`deployment suffix`값은 배포 리소스의 이름이 Azure 전체에 고유한 지 확인 하는 데 사용 됩니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-185">The `deployment suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="e467b-186">문자 및 숫자의 고유 문자열 이어야 하며 8 자이 하 여야 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-186">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    <span data-ttu-id="e467b-187">텍스트 파일을 저장 하 고 해당 경로를 적어 둡니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-187">Save the text file and make a note of its path.</span></span>

4. <span data-ttu-id="e467b-188">이제 Porter을 사용 하 여 카메라 앱을 배포할 준비가 되었습니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-188">You're now ready to deploy the camera app using Porter.</span></span> <span data-ttu-id="e467b-189">설치 명령을 실행 하 고 IoT Edge 배포가 생성 되 면 시청 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-189">Run the install command and watch as the IoT Edge deployment is created.</span></span>

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. <span data-ttu-id="e467b-190">에서 카메라 피드를 보면 카메라의 배포가 완료 되었는지 확인 `https://<camera-ip>:3000/` `<camara-ip>` 합니다. 여기서는 카메라 IP 주소입니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-190">Verify that the camera's deployment is complete by viewing the camera feed at `https://<camera-ip>:3000/`, where `<camara-ip>` is the camera IP address.</span></span> <span data-ttu-id="e467b-191">이 단계는 최대 10 분 정도 걸릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-191">This step may take up to 10 minutes.</span></span>

## <a name="configure-azure-stream-analytics"></a><span data-ttu-id="e467b-192">Azure Stream Analytics 구성</span><span class="sxs-lookup"><span data-stu-id="e467b-192">Configure Azure Stream Analytics</span></span>

<span data-ttu-id="e467b-193">이제 데이터를 카메라에서 Azure Stream Analytics로 이동 했으므로 Power BI와 통신 하도록 수동으로 권한을 부여 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-193">Now that data is flowing to Azure Stream Analytics from the camera, we need to manually authorize it to communicate with Power BI.</span></span>

1. <span data-ttu-id="e467b-194">Azure Portal에서 **모든 리소스**와 \*프로세스-footfall의 \[ 접미사 \] \* 작업을 엽니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-194">From the Azure portal, open **All Resources**, and the *process-footfall\[yoursuffix\]* job.</span></span>

2. <span data-ttu-id="e467b-195">Stream Analytics 작업 창의 **작업 토폴로지** 섹션에서 **출력** 옵션을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-195">In the **Job Topology** section of the Stream Analytics job pane, select the **Outputs** option.</span></span>

3. <span data-ttu-id="e467b-196">**트래픽 출력** 출력 싱크를 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-196">Select the **traffic-output** output sink.</span></span>

4. <span data-ttu-id="e467b-197">**권한 부여 갱신** 을 선택 하 고 Power BI 계정에 로그인 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-197">Select **Renew authorization** and sign in to your Power BI account.</span></span>
  
    ![Power BI에서 권한 부여 프롬프트 갱신](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. <span data-ttu-id="e467b-199">출력 설정을 저장 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-199">Save the output settings.</span></span>

6. <span data-ttu-id="e467b-200">**개요** 창으로 이동 하 고 **시작** 을 선택 하 여 Power BI로 데이터 보내기를 시작 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-200">Go to the **Overview** pane and select **Start** to start sending data to Power BI.</span></span>

7. <span data-ttu-id="e467b-201">작업 출력 시작 시간으로 **지금**을 선택하고 **시작**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-201">Select **Now** for job output start time and select **Start**.</span></span> <span data-ttu-id="e467b-202">알림 표시줄에서 작업 상태를 볼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-202">You can view the job status in the notification bar.</span></span>

## <a name="create-a-power-bi-dashboard"></a><span data-ttu-id="e467b-203">Power BI 대시보드 만들기</span><span class="sxs-lookup"><span data-stu-id="e467b-203">Create a Power BI Dashboard</span></span>

1. <span data-ttu-id="e467b-204">작업이 성공 하면 [Power BI](https://powerbi.com/) 으로 이동 하 여 회사 또는 학교 계정으로 로그인 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-204">Once the job succeeds, go to [Power BI](https://powerbi.com/) and sign in with your work or school account.</span></span> <span data-ttu-id="e467b-205">Stream Analytics 작업 쿼리가 결과를 출력 하는 경우 만든 *footfall* 데이터 집합 **은 데이터 집합 탭에** 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-205">If the Stream Analytics job query is outputting results, the *footfall-dataset* dataset you created exists under the **Datasets** tab.</span></span>

2. <span data-ttu-id="e467b-206">Power BI 작업 영역에서 **+ 만들기** 를 선택 하 여 *Footfall Analysis* 라는 새 대시보드를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-206">From your Power BI workspace, select **+ Create** to create a new dashboard named *Footfall Analysis.*</span></span>

3. <span data-ttu-id="e467b-207">창 맨 위에서 **타일 추가**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-207">At the top of the window, select **Add tile**.</span></span> <span data-ttu-id="e467b-208">**사용자 지정 스트리밍 데이터**를 선택하고 **다음**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-208">Then select **Custom Streaming Data** and **Next**.</span></span> <span data-ttu-id="e467b-209">**데이터**집합에서 **footfall** 을 선택 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-209">Choose the **footfall-dataset** under **Your Datasets**.</span></span> <span data-ttu-id="e467b-210">**시각화 유형** 드롭다운에서 **카드** 를 선택 하 고 **필드**에 **age** 를 추가 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-210">Select **Card** from the **Visualization type** dropdown, and add **age** to **Fields**.</span></span> <span data-ttu-id="e467b-211">**다음**을 선택하여 타일 이름을 입력하고, **적용**을 선택하여 타일을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-211">Select **Next** to enter a name for the tile, and then select **Apply** to create the tile.</span></span>

4. <span data-ttu-id="e467b-212">필요에 따라 추가 필드와 카드를 추가할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-212">You can add additional fields and cards as desired.</span></span>

## <a name="test-your-solution"></a><span data-ttu-id="e467b-213">솔루션 테스트</span><span class="sxs-lookup"><span data-stu-id="e467b-213">Test Your Solution</span></span>

<span data-ttu-id="e467b-214">카메라 앞에서 다른 사람이 탐색 하는 Power BI에서 만든 카드의 데이터가 어떻게 변경 되는지 관찰 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-214">Observe how the data in the cards you created in Power BI changes as different people walk in front of the camera.</span></span> <span data-ttu-id="e467b-215">추론 기록 된 후에 표시 되는 데 최대 20 초가 걸릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-215">Inferences may take up to 20 seconds to appear once recorded.</span></span>

## <a name="remove-your-solution"></a><span data-ttu-id="e467b-216">솔루션 제거</span><span class="sxs-lookup"><span data-stu-id="e467b-216">Remove Your Solution</span></span>

<span data-ttu-id="e467b-217">솔루션을 제거 하려면 배포를 위해 만든 것과 동일한 매개 변수 파일을 사용 하 여 Porter를 사용 하 여 다음 명령을 실행 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-217">If you'd like to remove your solution, run the following commands using Porter, using the same parameter files that you created for deployment:</span></span>

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a><span data-ttu-id="e467b-218">다음 단계</span><span class="sxs-lookup"><span data-stu-id="e467b-218">Next steps</span></span>

- <span data-ttu-id="e467b-219">[하이브리드 앱 디자인 고려 사항]에 대해 자세히 알아보세요. (overview-app-design-considerations.md)</span><span class="sxs-lookup"><span data-stu-id="e467b-219">Learn more about [Hybrid app design considerations].(overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="e467b-220">[GitHub에서이 샘플의 코드에 대 한 향상 된](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis)기능을 검토 하 고 제안 합니다.</span><span class="sxs-lookup"><span data-stu-id="e467b-220">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span></span>
