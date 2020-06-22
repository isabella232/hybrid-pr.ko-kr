---
title: Edge 패턴에서 기계 학습 모델 학습
description: Azure 및 Azure Stack Hub를 사용 하 여에 지에서 기계 학습 모델 학습을 수행 하는 방법에 대해 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: da1012cc8847f221de6bb540ba4e191e43920f41
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911099"
---
# <a name="train-machine-learning-model-at-the-edge-pattern"></a><span data-ttu-id="03510-103">Edge 패턴에서 기계 학습 모델 학습</span><span class="sxs-lookup"><span data-stu-id="03510-103">Train machine learning model at the edge pattern</span></span>

<span data-ttu-id="03510-104">온-프레미스에만 존재 하는 데이터에서 ML (이식 가능한 기계 학습) 모델을 생성 합니다.</span><span class="sxs-lookup"><span data-stu-id="03510-104">Generate portable machine learning (ML) models from data that only exists on-premises.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="03510-105">컨텍스트 및 문제점</span><span class="sxs-lookup"><span data-stu-id="03510-105">Context and problem</span></span>

<span data-ttu-id="03510-106">대부분의 조직에서는 데이터 과학자 이해 하는 도구를 사용 하 여 온-프레미스 또는 레거시 데이터에서 통찰력을 잠금 해제 하려고 합니다.</span><span class="sxs-lookup"><span data-stu-id="03510-106">Many organizations would like to unlock insights from their on-premises or legacy data using tools that their data scientists understand.</span></span> <span data-ttu-id="03510-107">[Azure Machine Learning](/azure/machine-learning/) 는 ML 및 심층 학습 모델을 학습, 조정 및 배포 하기 위한 클라우드 기본 도구를 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="03510-107">[Azure Machine Learning](/azure/machine-learning/) provides cloud-native tooling to train, tune, and deploy ML and deep learning models.</span></span>  

<span data-ttu-id="03510-108">그러나 일부 데이터는 클라우드로의 송신이 너무 크거나 규정 상의 이유로 클라우드로 전송할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="03510-108">However, some data is too large send to the cloud or can't be sent to the cloud for regulatory reasons.</span></span> <span data-ttu-id="03510-109">이 패턴을 사용 하 여 데이터 과학자는 온-프레미스 데이터 및 계산을 사용 하 여 모델을 학습 하는 Azure Machine Learning 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="03510-109">Using this pattern, data scientists can use Azure Machine Learning to train models using on-premises data and compute.</span></span>

## <a name="solution"></a><span data-ttu-id="03510-110">해결 방법</span><span class="sxs-lookup"><span data-stu-id="03510-110">Solution</span></span>

<span data-ttu-id="03510-111">Edge 패턴의 교육은 Azure Stack 허브에서 실행 되는 VM (가상 머신)을 사용 합니다.</span><span class="sxs-lookup"><span data-stu-id="03510-111">The training at the edge pattern uses a virtual machine (VM) running on Azure Stack Hub.</span></span> <span data-ttu-id="03510-112">VM은 Azure ML에서 계산 대상으로 등록 되므로 온-프레미스로만 데이터에 액세스할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="03510-112">The VM is registered as a compute target in Azure ML, letting it access data only available on-premises.</span></span> <span data-ttu-id="03510-113">이 경우 데이터는 Azure Stack 허브의 blob 저장소에 저장 됩니다.</span><span class="sxs-lookup"><span data-stu-id="03510-113">In this case, the data is stored in Azure Stack Hub's blob storage.</span></span>

<span data-ttu-id="03510-114">모델을 학습 한 후에는 Azure ML, 컨테이너 화 된에 등록 되 고 배포를 위해 Azure Container Registry에 추가 됩니다.</span><span class="sxs-lookup"><span data-stu-id="03510-114">Once the model is trained, it's registered with Azure ML, containerized, and added to an Azure Container Registry for deployment.</span></span> <span data-ttu-id="03510-115">이 패턴 반복의 경우 공용 인터넷을 통해 Azure Stack 허브 교육 VM에 연결할 수 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="03510-115">For this iteration of the pattern, the Azure Stack Hub training VM must be reachable over the public internet.</span></span>

<span data-ttu-id="03510-116">[![가장자리 아키텍처에서 ML 모델 학습](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span><span class="sxs-lookup"><span data-stu-id="03510-116">[![Train ML model at the edge architecture](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span></span>

<span data-ttu-id="03510-117">패턴의 작동 방식은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="03510-117">Here's how the pattern works:</span></span>

1. <span data-ttu-id="03510-118">Azure Stack 허브 VM은 Azure ML을 사용 하 여 계산 대상으로 배포 및 등록 됩니다.</span><span class="sxs-lookup"><span data-stu-id="03510-118">The Azure Stack Hub VM is deployed and registered as a compute target with Azure ML.</span></span>
2. <span data-ttu-id="03510-119">Azure ML에서 계산 대상으로 Azure Stack 허브 VM을 사용 하는 실험을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="03510-119">An experiment is created in Azure ML that uses the Azure Stack Hub VM as a compute target.</span></span>
3. <span data-ttu-id="03510-120">모델을 학습 한 후에는 등록 되 고 컨테이너 화 된 됩니다.</span><span class="sxs-lookup"><span data-stu-id="03510-120">Once the model is trained, it's registered and containerized.</span></span>
4. <span data-ttu-id="03510-121">이제 온-프레미스 또는 클라우드의 위치에 모델을 배포할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="03510-121">The model can now be deployed to locations that are either on-premises or in the cloud.</span></span>

## <a name="components"></a><span data-ttu-id="03510-122">구성 요소</span><span class="sxs-lookup"><span data-stu-id="03510-122">Components</span></span>

<span data-ttu-id="03510-123">이 솔루션은 다음 구성 요소를 사용 합니다.</span><span class="sxs-lookup"><span data-stu-id="03510-123">This solution uses the following components:</span></span>

| <span data-ttu-id="03510-124">계층</span><span class="sxs-lookup"><span data-stu-id="03510-124">Layer</span></span> | <span data-ttu-id="03510-125">구성 요소</span><span class="sxs-lookup"><span data-stu-id="03510-125">Component</span></span> | <span data-ttu-id="03510-126">Description</span><span class="sxs-lookup"><span data-stu-id="03510-126">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="03510-127">Azure</span><span class="sxs-lookup"><span data-stu-id="03510-127">Azure</span></span> | <span data-ttu-id="03510-128">Azure Machine Learning</span><span class="sxs-lookup"><span data-stu-id="03510-128">Azure Machine Learning</span></span> | <span data-ttu-id="03510-129">오케스트레이션는 ML 모델의 학습을 합니다. [Azure Machine Learning](/azure/machine-learning/)</span><span class="sxs-lookup"><span data-stu-id="03510-129">[Azure Machine Learning](/azure/machine-learning/) orchestrates the training of the ML model.</span></span> |
| | <span data-ttu-id="03510-130">Azure Container Registry</span><span class="sxs-lookup"><span data-stu-id="03510-130">Azure Container Registry</span></span> | <span data-ttu-id="03510-131">Azure ML은 모델을 컨테이너에 패키지 하 고 배포를 위해 [Azure Container Registry](/azure/container-registry/) 에 저장 합니다.</span><span class="sxs-lookup"><span data-stu-id="03510-131">Azure ML packages the model into a container and stores it in an [Azure Container Registry](/azure/container-registry/) for deployment.</span></span>|
| <span data-ttu-id="03510-132">Azure Stack 허브</span><span class="sxs-lookup"><span data-stu-id="03510-132">Azure Stack Hub</span></span> | <span data-ttu-id="03510-133">App Service</span><span class="sxs-lookup"><span data-stu-id="03510-133">App Service</span></span> | <span data-ttu-id="03510-134">[App Service를 사용 하 Azure Stack 허브](/azure-stack/operator/azure-stack-app-service-overview) 는에 지의 구성 요소에 대 한 기반을 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="03510-134">[Azure Stack Hub with App Service](/azure-stack/operator/azure-stack-app-service-overview) provides the base for the components at the edge.</span></span> |
| | <span data-ttu-id="03510-135">Compute</span><span class="sxs-lookup"><span data-stu-id="03510-135">Compute</span></span> | <span data-ttu-id="03510-136">Docker를 사용 하 여 Ubuntu를 실행 하는 Azure Stack 허브 VM은 ML 모델을 학습 하는 데 사용 됩니다.</span><span class="sxs-lookup"><span data-stu-id="03510-136">An Azure Stack Hub VM running Ubuntu with Docker is used to train the ML model.</span></span> |
| | <span data-ttu-id="03510-137">Storage</span><span class="sxs-lookup"><span data-stu-id="03510-137">Storage</span></span> | <span data-ttu-id="03510-138">개인 데이터는 Azure Stack Hub blob 저장소에서 호스팅될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="03510-138">Private data can be hosted in Azure Stack Hub blob storage.</span></span> |

## <a name="issues-and-considerations"></a><span data-ttu-id="03510-139">문제 및 고려 사항</span><span class="sxs-lookup"><span data-stu-id="03510-139">Issues and considerations</span></span>

<span data-ttu-id="03510-140">이 솔루션을 구현 하는 방법을 결정할 때 다음 사항을 고려 하십시오.</span><span class="sxs-lookup"><span data-stu-id="03510-140">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="03510-141">확장성</span><span class="sxs-lookup"><span data-stu-id="03510-141">Scalability</span></span>

<span data-ttu-id="03510-142">이 솔루션을 확장할 수 있도록 하려면 Azure Stack 허브에서 적절 한 크기의 VM을 만들어 학습 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="03510-142">To enable this solution to scale, you'll need to create an appropriately sized VM on Azure Stack Hub for training.</span></span>

### <a name="availability"></a><span data-ttu-id="03510-143">가용성</span><span class="sxs-lookup"><span data-stu-id="03510-143">Availability</span></span>

<span data-ttu-id="03510-144">학습 스크립트와 Azure Stack 허브 VM이 학습에 사용 되는 온-프레미스 데이터에 액세스할 수 있는지 확인 합니다.</span><span class="sxs-lookup"><span data-stu-id="03510-144">Ensure that the training scripts and Azure Stack Hub VM have access to the on-premises data used for training.</span></span>

### <a name="manageability"></a><span data-ttu-id="03510-145">관리 효율</span><span class="sxs-lookup"><span data-stu-id="03510-145">Manageability</span></span>

<span data-ttu-id="03510-146">모델을 배포 하는 동안 혼동을 피하기 위해 모델 및 실험을 적절 하 게 등록 하 고, 버전을 지정 하 고, 태그를 지정</span><span class="sxs-lookup"><span data-stu-id="03510-146">Ensure that models and experiments are appropriately registered, versioned, and tagged to avoid confusion during model deployment.</span></span>

### <a name="security"></a><span data-ttu-id="03510-147">보안</span><span class="sxs-lookup"><span data-stu-id="03510-147">Security</span></span>

<span data-ttu-id="03510-148">이 패턴을 통해 Azure ML은 가능한 중요 한 데이터를 온-프레미스에 액세스할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="03510-148">This pattern lets Azure ML access possible sensitive data on-premises.</span></span> <span data-ttu-id="03510-149">Azure Stack 허브 VM에 대 한 SSH에 사용 되는 계정에 강력한 암호가 있고 학습 스크립트가 데이터를 클라우드에 유지 하거나 업로드 하지 않도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="03510-149">Ensure the account used to SSH into Azure Stack Hub VM has a strong password and training scripts don't preserve or upload data to the cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="03510-150">다음 단계</span><span class="sxs-lookup"><span data-stu-id="03510-150">Next steps</span></span>

<span data-ttu-id="03510-151">이 문서에서 소개 하는 항목에 대 한 자세한 내용은 다음을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="03510-151">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="03510-152">ML 및 관련 항목의 개요는 [Azure Machine Learning 설명서](/azure/machine-learning) 를 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="03510-152">See the [Azure Machine Learning documentation](/azure/machine-learning) for an overview of ML and related topics.</span></span>
- <span data-ttu-id="03510-153">컨테이너 배포를 위한 이미지를 작성, 저장 및 관리 하는 방법을 알아보려면 [Azure Container Registry](/azure/container-registry/) 를 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="03510-153">See [Azure Container Registry](/azure/container-registry/) to learn how to build, store, and manage images for container deployments.</span></span>
- <span data-ttu-id="03510-154">리소스 공급자 및 배포 방법에 대 한 자세한 내용은 [Azure Stack Hub의 App Service](/azure-stack/operator/azure-stack-app-service-overview) 를 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="03510-154">Refer to [App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) to learn more about the resource provider and how to deploy.</span></span>
- <span data-ttu-id="03510-155">모범 사례에 대해 자세히 알아보고 추가 질문에 대 한 답변을 얻으려면 [하이브리드 응용 프로그램 디자인 고려 사항](overview-app-design-considerations.md) 을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="03510-155">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get any additional questions answered.</span></span>
- <span data-ttu-id="03510-156">제품 및 솔루션의 전체 포트폴리오에 대해 자세히 알아보려면 [Azure Stack 제품군 및 솔루션](/azure-stack) 을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="03510-156">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="03510-157">솔루션 예제를 테스트할 준비가 되 면 [edge 배포 가이드에서 ML 모델 학습](https://aka.ms/edgetrainingdeploy)을 계속 진행 합니다.</span><span class="sxs-lookup"><span data-stu-id="03510-157">When you're ready to test the solution example, continue with the [Train ML model at the edge deployment guide](https://aka.ms/edgetrainingdeploy).</span></span> <span data-ttu-id="03510-158">배포 가이드에서는 구성 요소를 배포 및 테스트 하는 방법에 대 한 단계별 지침을 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="03510-158">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>
