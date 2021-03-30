---
title: Azure 및 Azure Stack Hub에서 AI 기반 발걸음 감지 솔루션 배포
description: Azure 및 Azure Stack Hub를 사용하여 소매점에서 방문자 트래픽을 분석하는 AI 기반 발걸음 감지 솔루션을 배포하는 방법을 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: caedbd4758b9ae8c93cf9bb625ed9aac68bfa196
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895369"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a><span data-ttu-id="515f0-103">Azure 및 Azure Stack Hub를 사용하여 AI 기반 발걸음 감지 솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="515f0-103">Deploy an AI-based footfall detection solution using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="515f0-104">이 문서에서는 Azure, Azure Stack Hub 및 Custom Vision AI Dev Kit를 사용하여 실제 작업에서 인사이트를 생성하는 AI 기반 솔루션을 배포하는 방법을 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-104">This article describes how to deploy an AI-based solution that generates insights from real world actions by using Azure, Azure Stack Hub, and the Custom Vision AI Dev Kit.</span></span>

<span data-ttu-id="515f0-105">이 솔루션에서는 다음을 수행하는 방법을 알아봅니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-105">In this solution, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="515f0-106">에지에 CNAB(Cloud Native Application Bundle)를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-106">Deploy Cloud Native Application Bundles (CNAB) at the edge.</span></span> 
> - <span data-ttu-id="515f0-107">클라우드 경계를 확장하는 앱을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-107">Deploy an app that spans cloud boundaries.</span></span>
> - <span data-ttu-id="515f0-108">Edge에서 유추를 위해 Custom Vision AI Dev Kit를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-108">Use the Custom Vision AI Dev Kit for inference at the edge.</span></span>

> [!Tip]  
> <span data-ttu-id="515f0-109">![하이브리드 핵심 요소 다이어그램](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="515f0-109">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="515f0-110">Microsoft Azure Stack Hub는 Azure의 확장입니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-110">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="515f0-111">Azure Stack Hub는 온-프레미스 환경에 클라우드 컴퓨팅의 민첩성과 혁신을 제공하여 어디서나 하이브리드 앱을 빌드하고 배포할 수 있는 유일한 하이브리드 클라우드를 사용하도록 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-111">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="515f0-112">[하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 문서는 하이브리드 앱 디자인, 배포 및 운영에 대한 소프트웨어 품질(배치, 확장성, 가용성, 복원력, 관리 효율성 및 보안)의 핵심 요소를 검토합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-112">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="515f0-113">디자인 고려 사항은 하이브리드 앱 디자인을 최적화하고 프로덕션 환경에서 문제를 최소화하는 데 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-113">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="515f0-114">필수 구성 요소</span><span class="sxs-lookup"><span data-stu-id="515f0-114">Prerequisites</span></span>

<span data-ttu-id="515f0-115">이 배포 가이드를 시작하기 전에 다음을 확인하십시오.</span><span class="sxs-lookup"><span data-stu-id="515f0-115">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="515f0-116">[발걸음 감지 패턴](pattern-retail-footfall-detection.md) 항목을 검토합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-116">Review the [Footfall detection pattern](pattern-retail-footfall-detection.md) topic.</span></span>
- <span data-ttu-id="515f0-117">다음을 통해 ASDK(Azure Stack Development Kit) 또는 Azure Stack Hub 통합 시스템 인스턴스에 대한 사용자 액세스 권한을 확보합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-117">Obtain user access to an Azure Stack Development Kit (ASDK) or Azure Stack Hub integrated system instance, with:</span></span>
  - <span data-ttu-id="515f0-118">[Azure Stack Hub 리소스 공급자에 대한 Azure App Service](/azure-stack/operator/azure-stack-app-service-overview)를 설치합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-118">The [Azure App Service on Azure Stack Hub resource provider](/azure-stack/operator/azure-stack-app-service-overview) installed.</span></span> <span data-ttu-id="515f0-119">설치하려면 Azure Stack Hub 인스턴스에 대한 운영자 액세스 권한이 있거나 관리자에게 문의해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-119">You need operator access to your Azure Stack Hub instance, or work with your administrator to install.</span></span>
  - <span data-ttu-id="515f0-120">App Service 및 스토리지 할당량을 제공하는 제품을 구독합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-120">A subscription to an offer that provides App Service and Storage quota.</span></span> <span data-ttu-id="515f0-121">제품을 만들려면 운영자 액세스 권한이 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-121">You need operator access to create an offer.</span></span>
- <span data-ttu-id="515f0-122">Azure 구독에 대한 액세스 권한을 얻습니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-122">Obtain access to an Azure subscription.</span></span>
  - <span data-ttu-id="515f0-123">Azure 구독이 없으면 시작하기 전에 [평가판 계정](https://azure.microsoft.com/free/)에 등록하세요.</span><span class="sxs-lookup"><span data-stu-id="515f0-123">If you don't have an Azure subscription, sign up for a [free trial account](https://azure.microsoft.com/free/) before you begin.</span></span>
- <span data-ttu-id="515f0-124">디렉터리에 두 개의 서비스 주체를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-124">Create two service principals in your directory:</span></span>
  - <span data-ttu-id="515f0-125">하나는 Azure 구독 범위에서 액세스할 수 있는 Azure 리소스에 사용하도록 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-125">One set up for use with Azure resources, with access at the Azure subscription scope.</span></span>
  - <span data-ttu-id="515f0-126">하나는 Azure Stack Hub 구독 범위에서 액세스할 수 있는 Azure Stack Hub 리소스에 사용하도록 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-126">One set up for use with Azure Stack Hub resources, with access at the Azure Stack Hub subscription scope.</span></span>
  - <span data-ttu-id="515f0-127">서비스 주체를 만들고 액세스 권한을 부여하는 방법에 대한 자세한 내용은 [앱 ID를 사용하여 리소스에 액세스](/azure-stack/operator/azure-stack-create-service-principals)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="515f0-127">To learn more about creating service principals and authorizing access, see [Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals).</span></span> <span data-ttu-id="515f0-128">Azure CLI를 사용하는 것을 선호하는 경우 [Azure CLI를 사용하여 Azure 서비스 주체 만들기](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="515f0-128">If you prefer to use Azure CLI, see [Create an Azure service principal with Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true).</span></span>
- <span data-ttu-id="515f0-129">Azure 또는 Azure Stack Hub에 Azure Cognitive Services를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-129">Deploy Azure Cognitive Services in Azure or Azure Stack Hub.</span></span>
  - <span data-ttu-id="515f0-130">먼저 [Cognitive Services에 대해 자세히 알아봅니다](https://azure.microsoft.com/services/cognitive-services/).</span><span class="sxs-lookup"><span data-stu-id="515f0-130">First, [learn more about Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).</span></span>
  - <span data-ttu-id="515f0-131">그런 다음, [Azure Cognitive Services를 Azure Stack Hub에 배포](/azure-stack/user/azure-stack-solution-template-cognitive-services)를 방문하여 Azure Stack Hub에 Cognitive Services를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-131">Then visit [Deploy Azure Cognitive Services to Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services) to deploy Cognitive Services on Azure Stack Hub.</span></span> <span data-ttu-id="515f0-132">먼저 미리 보기에 액세스하려면 등록해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-132">You first need to sign up for access to the preview.</span></span>
- <span data-ttu-id="515f0-133">구성되지 않은 Azure Custom Vision AI Dev Kit를 복제하거나 다운로드합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-133">Clone or download an unconfigured Azure Custom Vision AI Dev Kit.</span></span> <span data-ttu-id="515f0-134">자세한 내용은 [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="515f0-134">For details, see the [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span></span>
- <span data-ttu-id="515f0-135">Power BI 계정에 등록합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-135">Sign up for a Power BI account.</span></span>
- <span data-ttu-id="515f0-136">Azure Cognitive Services Face API 구독 키 및 엔드포인트 URL.</span><span class="sxs-lookup"><span data-stu-id="515f0-136">An Azure Cognitive Services Face API subscription key and endpoint URL.</span></span> <span data-ttu-id="515f0-137">[Cognitive Services 체험하기](https://azure.microsoft.com/try/cognitive-services/?api=face-api) 평가판을 통해 두 가지를 모두 얻을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-137">You can get both with the [Try Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) free trial.</span></span> <span data-ttu-id="515f0-138">또는 [Cognitive Services 계정 만들기](/azure/cognitive-services/cognitive-services-apis-create-account)의 지침을 따르세요.</span><span class="sxs-lookup"><span data-stu-id="515f0-138">Or, follow the instructions in [Create a Cognitive Services account](/azure/cognitive-services/cognitive-services-apis-create-account).</span></span>
- <span data-ttu-id="515f0-139">다음 개발 리소스를 설치합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-139">Install the following development resources:</span></span>
  - [<span data-ttu-id="515f0-140">Azure CLI 2.0</span><span class="sxs-lookup"><span data-stu-id="515f0-140">Azure CLI 2.0</span></span>](/azure-stack/user/azure-stack-version-profiles-azurecli2)
  - [<span data-ttu-id="515f0-141">Docker CE</span><span class="sxs-lookup"><span data-stu-id="515f0-141">Docker CE</span></span>](https://hub.docker.com/search/?type=edition&offering=community)
  - <span data-ttu-id="515f0-142">[Porter](https://porter.sh/).</span><span class="sxs-lookup"><span data-stu-id="515f0-142">[Porter](https://porter.sh/).</span></span> <span data-ttu-id="515f0-143">Porter를 사용하면 제공되는 CNAB 번들 매니페스트를 사용하여 클라우드 앱을 배포할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-143">You use Porter to deploy cloud apps using CNAB bundle manifests that are provided for you.</span></span>
  - [<span data-ttu-id="515f0-144">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="515f0-144">Visual Studio Code</span></span>](https://code.visualstudio.com/)
  - [<span data-ttu-id="515f0-145">Visual Studio Code용 Azure IoT Tools</span><span class="sxs-lookup"><span data-stu-id="515f0-145">Azure IoT Tools for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [<span data-ttu-id="515f0-146">Visual Studio Code용 Python 확장</span><span class="sxs-lookup"><span data-stu-id="515f0-146">Python extension for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [<span data-ttu-id="515f0-147">Python</span><span class="sxs-lookup"><span data-stu-id="515f0-147">Python</span></span>](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a><span data-ttu-id="515f0-148">하이브리드 클라우드 앱 배포</span><span class="sxs-lookup"><span data-stu-id="515f0-148">Deploy the hybrid cloud app</span></span>

<span data-ttu-id="515f0-149">먼저 Porter CLI를 사용하여 자격 증명 집합을 생성한 다음, 클라우드 앱을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-149">First you use the Porter CLI to generate a credential set, then deploy the cloud app.</span></span>  

1. <span data-ttu-id="515f0-150"> https://github.com/azure-samples/azure-intelligent-edge-patterns 에서 솔루션 샘플 코드를 복제하거나 다운로드합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-150">Clone or download the solution sample code from https://github.com/azure-samples/azure-intelligent-edge-patterns.</span></span> 

1. <span data-ttu-id="515f0-151">Porter는 앱 배포를 자동화하는 자격 증명 세트를 생성합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-151">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="515f0-152">자격 증명 생성 명령을 실행하기 전에 다음을 사용할 수 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-152">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="515f0-153">Azure 리소스에 액세스하기 위한 서비스 주체(예: 서비스 주체 ID, 키 및 테넌트 DNS)</span><span class="sxs-lookup"><span data-stu-id="515f0-153">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="515f0-154">Azure 구독에 대한 구독 ID</span><span class="sxs-lookup"><span data-stu-id="515f0-154">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="515f0-155">Azure Stack Hub 리소스에 액세스하기 위한 서비스 주체(예: 서비스 주체 ID, 키 및 테넌트 DNS)</span><span class="sxs-lookup"><span data-stu-id="515f0-155">A service principal for accessing Azure Stack Hub resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="515f0-156">Azure Stack Hub 구독에 대한 구독 ID</span><span class="sxs-lookup"><span data-stu-id="515f0-156">The subscription ID for your Azure Stack Hub subscription.</span></span>
    - <span data-ttu-id="515f0-157">Azure Cognitive Services Face API 키 및 리소스 엔드포인트 URL</span><span class="sxs-lookup"><span data-stu-id="515f0-157">Your Azure Cognitive Services Face API key and resource endpoint URL.</span></span>

1. <span data-ttu-id="515f0-158">Porter 자격 증명 생성 프로세스를 실행하고 프롬프트를 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-158">Run the Porter credential generation process and follow the prompts:</span></span>

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. <span data-ttu-id="515f0-159">Porter를 실행하려면 매개 변수 세트도 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-159">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="515f0-160">매개 변수 텍스트 파일을 만들고 다음과 같은 이름/값 쌍을 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-160">Create a parameter text file and enter the following name/value pairs.</span></span> <span data-ttu-id="515f0-161">필요한 값에 대한 지원이 필요한 경우 Azure Stack Hub 관리자에게 문의하세요.</span><span class="sxs-lookup"><span data-stu-id="515f0-161">Ask your Azure Stack Hub administrator if you need assistance with any of the required values.</span></span>

   > [!NOTE] 
   > <span data-ttu-id="515f0-162">`resource suffix` 값은 배포의 리소스가 Azure 전체에서 고유한 이름을 갖도록 하는 데 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-162">The `resource suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="515f0-163">8자 이하의 문자와 숫자로 구성된 고유한 문자열이어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-163">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

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
   <span data-ttu-id="515f0-164">텍스트 파일을 저장하고 경로를 적어둡니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-164">Save the text file and make a note of its path.</span></span>

1. <span data-ttu-id="515f0-165">이제 Porter를 사용하여 하이브리드 클라우드 앱을 배포할 준비가 되었습니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-165">You're now ready to deploy the hybrid cloud app using Porter.</span></span> <span data-ttu-id="515f0-166">설치 명령을 실행하고 리소스가 Azure 및 Azure Stack Hub에 배포되는 것을 살펴봅니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-166">Run the install command and watch as resources are deployed to Azure and Azure Stack Hub:</span></span>

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. <span data-ttu-id="515f0-167">배포가 완료되면 다음 값을 적어둡니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-167">Once deployment is complete, make note of the following values:</span></span>
    - <span data-ttu-id="515f0-168">카메라의 연결 문자열</span><span class="sxs-lookup"><span data-stu-id="515f0-168">The camera's connection string.</span></span>
    - <span data-ttu-id="515f0-169">이미지 스토리지 계정 연결 문자열</span><span class="sxs-lookup"><span data-stu-id="515f0-169">The image storage account connection string.</span></span>
    - <span data-ttu-id="515f0-170">리소스 그룹 이름</span><span class="sxs-lookup"><span data-stu-id="515f0-170">The resource group names.</span></span>

## <a name="prepare-the-custom-vision-ai-devkit"></a><span data-ttu-id="515f0-171">Custom Vision AI DevKit 준비</span><span class="sxs-lookup"><span data-stu-id="515f0-171">Prepare the Custom Vision AI DevKit</span></span>

<span data-ttu-id="515f0-172">다음으로 [Vision AI DevKit 빠른 시작](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/)을 참조하여 Custom Vision AI Dev Kit를 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-172">Next, set up the Custom Vision AI Dev Kit as shown in the [Vision AI DevKit quickstart](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span></span> <span data-ttu-id="515f0-173">또한 이전 단계에서 제공한 연결 문자열을 사용하여 카메라를 설정하고 테스트합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-173">You also set up and test your camera, using the connection string provided in the previous step.</span></span>

## <a name="deploy-the-camera-app"></a><span data-ttu-id="515f0-174">카메라 앱 배포</span><span class="sxs-lookup"><span data-stu-id="515f0-174">Deploy the camera app</span></span>

<span data-ttu-id="515f0-175">Porter CLI를 사용하여 자격 증명 집합을 생성한 다음, 카메라 앱을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-175">Use the Porter CLI to generate a credential set, then deploy the camera app.</span></span>

1. <span data-ttu-id="515f0-176">Porter는 앱 배포를 자동화하는 자격 증명 세트를 생성합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-176">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="515f0-177">자격 증명 생성 명령을 실행하기 전에 다음을 사용할 수 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-177">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="515f0-178">Azure 리소스에 액세스하기 위한 서비스 주체(예: 서비스 주체 ID, 키 및 테넌트 DNS)</span><span class="sxs-lookup"><span data-stu-id="515f0-178">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="515f0-179">Azure 구독에 대한 구독 ID</span><span class="sxs-lookup"><span data-stu-id="515f0-179">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="515f0-180">클라우드 앱을 배포할 때 제공한 이미지 스토리지 계정 연결 문자열</span><span class="sxs-lookup"><span data-stu-id="515f0-180">The image storage account connection string provided when you deployed the cloud app.</span></span>

1. <span data-ttu-id="515f0-181">Porter 자격 증명 생성 프로세스를 실행하고 프롬프트를 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-181">Run the Porter credential generation process and follow the prompts:</span></span>

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. <span data-ttu-id="515f0-182">Porter를 실행하려면 매개 변수 세트도 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-182">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="515f0-183">매개 변수 텍스트 파일을 만들고 다음과 같은 텍스트를 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-183">Create a parameter text file and enter the following text.</span></span> <span data-ttu-id="515f0-184">필요한 값을 모르는 경우 Azure Stack Hub 관리자에게 문의하세요.</span><span class="sxs-lookup"><span data-stu-id="515f0-184">Ask your Azure Stack Hub administrator if you don't know some of the required values.</span></span>

    > [!NOTE]
    > <span data-ttu-id="515f0-185">`deployment suffix` 값은 배포의 리소스가 Azure 전체에서 고유한 이름을 갖도록 하는 데 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-185">The `deployment suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="515f0-186">8자 이하의 문자와 숫자로 구성된 고유한 문자열이어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-186">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    <span data-ttu-id="515f0-187">텍스트 파일을 저장하고 경로를 적어둡니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-187">Save the text file and make a note of its path.</span></span>

4. <span data-ttu-id="515f0-188">이제 Porter를 사용하여 카메라 앱을 배포할 준비가 되었습니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-188">You're now ready to deploy the camera app using Porter.</span></span> <span data-ttu-id="515f0-189">설치 명령을 실행하고 IoT Edge 배포가 생성되는 것을 살펴봅니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-189">Run the install command and watch as the IoT Edge deployment is created.</span></span>

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. <span data-ttu-id="515f0-190">`https://<camera-ip>:3000/`에서 카메라 피드를 확인하여 카메라 배포가 완료되었는지 확인합니다. `<camara-ip>`는 카메라 IP 주소입니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-190">Verify that the camera's deployment is complete by viewing the camera feed at `https://<camera-ip>:3000/`, where `<camara-ip>` is the camera IP address.</span></span> <span data-ttu-id="515f0-191">이 단계는 최대 10분 정도 걸릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-191">This step may take up to 10 minutes.</span></span>

## <a name="configure-azure-stream-analytics"></a><span data-ttu-id="515f0-192">Azure Stream Analytics 구성</span><span class="sxs-lookup"><span data-stu-id="515f0-192">Configure Azure Stream Analytics</span></span>

<span data-ttu-id="515f0-193">이제 카메라에서 Azure Stream Analytics로 데이터가 이동하기 때문에 Power BI와 통신할 수 있도록 수동으로 권한을 부여해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-193">Now that data is flowing to Azure Stream Analytics from the camera, we need to manually authorize it to communicate with Power BI.</span></span>

1. <span data-ttu-id="515f0-194">Azure Portal에서 **모든 리소스** 를 열고 *process-footfall\[yoursuffix\]* 작업을 엽니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-194">From the Azure portal, open **All Resources**, and the *process-footfall\[yoursuffix\]* job.</span></span>

2. <span data-ttu-id="515f0-195">Stream Analytics 작업 창의 **작업 토폴로지** 섹션에서 **출력** 옵션을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-195">In the **Job Topology** section of the Stream Analytics job pane, select the **Outputs** option.</span></span>

3. <span data-ttu-id="515f0-196">**traffic-output** 출력 싱크를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-196">Select the **traffic-output** output sink.</span></span>

4. <span data-ttu-id="515f0-197">**권한 갱신** 을 선택하고 사용자의 Power BI 계정에 로그인합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-197">Select **Renew authorization** and sign in to your Power BI account.</span></span>
  
    ![Power BI에서 권한 부여 프롬프트 갱신](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. <span data-ttu-id="515f0-199">출력 설정을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-199">Save the output settings.</span></span>

6. <span data-ttu-id="515f0-200">**개요** 창으로 이동하고 **시작** 을 선택하여 Power BI로 데이터를 보내기 시작합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-200">Go to the **Overview** pane and select **Start** to start sending data to Power BI.</span></span>

7. <span data-ttu-id="515f0-201">작업 출력 시작 시간으로 **지금** 을 선택하고 **시작** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-201">Select **Now** for job output start time and select **Start**.</span></span> <span data-ttu-id="515f0-202">알림 표시줄에서 작업 상태를 볼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-202">You can view the job status in the notification bar.</span></span>

## <a name="create-a-power-bi-dashboard"></a><span data-ttu-id="515f0-203">Power BI 대시보드 만들기</span><span class="sxs-lookup"><span data-stu-id="515f0-203">Create a Power BI Dashboard</span></span>

1. <span data-ttu-id="515f0-204">작업이 성공하면 [Power BI](https://powerbi.com/)로 이동하여 회사 또는 학교 계정으로 로그인합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-204">Once the job succeeds, go to [Power BI](https://powerbi.com/) and sign in with your work or school account.</span></span> <span data-ttu-id="515f0-205">Stream Analytics 작업 쿼리가 결과를 출력 중이면 앞에서 만든 *footfall-dataset* 데이터 세트가 **데이터 세트** 탭에 있는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-205">If the Stream Analytics job query is outputting results, the *footfall-dataset* dataset you created exists under the **Datasets** tab.</span></span>

2. <span data-ttu-id="515f0-206">Power BI 작업 영역에서 **+ 만들기** 를 선택하여 *발걸음 분석* 이라는 새 대시보드를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-206">From your Power BI workspace, select **+ Create** to create a new dashboard named *Footfall Analysis.*</span></span>

3. <span data-ttu-id="515f0-207">창 맨 위에서 **타일 추가** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-207">At the top of the window, select **Add tile**.</span></span> <span data-ttu-id="515f0-208">**사용자 지정 스트리밍 데이터** 를 선택하고 **다음** 을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-208">Then select **Custom Streaming Data** and **Next**.</span></span> <span data-ttu-id="515f0-209">**데이터 세트** 아래에서 **footfall-dataset** 를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-209">Choose the **footfall-dataset** under **Your Datasets**.</span></span> <span data-ttu-id="515f0-210">**시각화 유형** 드롭다운에서 **카드** 를 선택하고 **필드** 에 **age** 를 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-210">Select **Card** from the **Visualization type** dropdown, and add **age** to **Fields**.</span></span> <span data-ttu-id="515f0-211">**다음** 을 선택하여 타일 이름을 입력하고, **적용** 을 선택하여 타일을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-211">Select **Next** to enter a name for the tile, and then select **Apply** to create the tile.</span></span>

4. <span data-ttu-id="515f0-212">필요에 따라 필드와 카드를 더 추가할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-212">You can add additional fields and cards as desired.</span></span>

## <a name="test-your-solution"></a><span data-ttu-id="515f0-213">솔루션 테스트</span><span class="sxs-lookup"><span data-stu-id="515f0-213">Test Your Solution</span></span>

<span data-ttu-id="515f0-214">카메라 앞에서 사람들이 걷는 동안 Power BI에서 생성한 카드의 데이터가 어떻게 변하는지 관찰합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-214">Observe how the data in the cards you created in Power BI changes as different people walk in front of the camera.</span></span> <span data-ttu-id="515f0-215">기록된 후 추론이 나타나는 데 최대 20초가 걸릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-215">Inferences may take up to 20 seconds to appear once recorded.</span></span>

## <a name="remove-your-solution"></a><span data-ttu-id="515f0-216">솔루션 제거</span><span class="sxs-lookup"><span data-stu-id="515f0-216">Remove Your Solution</span></span>

<span data-ttu-id="515f0-217">솔루션을 제거하려면 배포용으로 만든 것과 동일한 매개 변수 파일을 사용하고 Porter를 사용하여 다음 명령을 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="515f0-217">If you'd like to remove your solution, run the following commands using Porter, using the same parameter files that you created for deployment:</span></span>

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a><span data-ttu-id="515f0-218">다음 단계</span><span class="sxs-lookup"><span data-stu-id="515f0-218">Next steps</span></span>

- <span data-ttu-id="515f0-219">[하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md)에 대해 자세히 알아보기</span><span class="sxs-lookup"><span data-stu-id="515f0-219">Learn more about [Hybrid app design considerations](overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="515f0-220">[GitHub에서 이 샘플의 코드](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis)를 검토하고 개선을 제안하세요.</span><span class="sxs-lookup"><span data-stu-id="515f0-220">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span></span>
