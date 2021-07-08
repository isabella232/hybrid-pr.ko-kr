---
title: Azure 및 Azure Stack Edge를 사용하여 품절 검색
description: Azure 및 Azure Stack Edge 서비스를 사용하여 품절 검색을 구현하는 방법을 알아보세요.
author: BryanLa
ms.topic: article
ms.date: 05/24/2021
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 05/24/2021
ms.openlocfilehash: b25a6391c4e64fa7018031bac4fb7d098c56b529
ms.sourcegitcommit: cf2c4033d1b169f5b63980ce1865281366905e2e
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/25/2021
ms.locfileid: "110343878"
---
# <a name="out-of-stock-detection-at-the-edge-pattern"></a><span data-ttu-id="6dfc6-103">에지 패턴에서 품절 검색</span><span class="sxs-lookup"><span data-stu-id="6dfc6-103">Out of stock detection at the edge pattern</span></span>

<span data-ttu-id="6dfc6-104">이 패턴은 Azure Stack Edge 또는 Azure IoT Edge 디바이스 및 네트워크 카메라를 사용하여 진열대에 품절된 품목이 있는지 확인하는 방법을 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-104">This pattern illustrates how to determine if shelves have out of stock items using an Azure Stack Edge or Azure IoT Edge device and network cameras.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="6dfc6-105">컨텍스트 및 문제점</span><span class="sxs-lookup"><span data-stu-id="6dfc6-105">Context and problem</span></span>

<span data-ttu-id="6dfc6-106">실제 소매 매장에서는 고객이 찾는 품목이 진열대에 없기 때문에 매출이 감소합니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-106">Physical retail stores lose sales because when customers look for an item, it's not present on the shelf.</span></span> <span data-ttu-id="6dfc6-107">그러나 해당 품목은 매장 뒤에 있으며 재입고되지 않았을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-107">However, the item could have been in the back of the store and not been restocked.</span></span> <span data-ttu-id="6dfc6-108">매장은 직원을 보다 효율적으로 활용하고 품목을 재입고해야 할 때 자동으로 알림을 받고자 합니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-108">Stores would like to use their staff more efficiently and get automatically notified when items need restocking.</span></span>

## <a name="solution"></a><span data-ttu-id="6dfc6-109">솔루션</span><span class="sxs-lookup"><span data-stu-id="6dfc6-109">Solution</span></span>

<span data-ttu-id="6dfc6-110">솔루션 예제는 각 매장에서 Azure Stack Edge와 같은 에지 디바이스를 사용하여 매장의 카메라의 데이터를 효율적으로 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-110">The solution example uses an edge device, like an Azure Stack Edge in each store, which efficiently processes data from cameras in the store.</span></span> <span data-ttu-id="6dfc6-111">이러한 최적화된 디자인을 통해 관련 이벤트 및 이미지만 클라우드로 보냅니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-111">This optimized design lets stores send only relevant events and images to the cloud.</span></span> <span data-ttu-id="6dfc6-112">이 디자인은 대역폭, 스토리지 공간을 절약하고 고객의 개인 정보 보호를 보장합니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-112">The design saves bandwidth, storage space, and ensures customer privacy.</span></span> <span data-ttu-id="6dfc6-113">각 카메라에서 프레임을 읽을 때 ML 모델은 이미지를 처리하고 품절 영역을 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-113">As frames are read from each camera, an ML model processes the image and returns any out of stock areas.</span></span> <span data-ttu-id="6dfc6-114">이미지 및 품절 영역은 로컬 웹 앱에 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-114">The image and out of stock areas are displayed on a local web app.</span></span> <span data-ttu-id="6dfc6-115">이 데이터는 Power BI에서 인사이트를 표시하기 위해 Time Series Insight 환경으로 보낼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-115">This data can be sent to a Time Series Insight environment to show insights in Power BI.</span></span>

![에지 솔루션 아키텍처의 품절](media/pattern-out-of-stock-at-edge/solution-architecture.png)

<span data-ttu-id="6dfc6-117">솔루션의 작동 방식은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-117">Here's how the solution works:</span></span>

1. <span data-ttu-id="6dfc6-118">HTTP 또는 RTSP를 통해 네트워크 카메라에서 이미지를 캡처합니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-118">Images are captured from a network camera over HTTP or RTSP.</span></span>
2. <span data-ttu-id="6dfc6-119">이미지 크기가 조정되고 추론 드라이버로 보내지며, 추론 드라이버는 ML 모델과 통신하여 품절 이미지가 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-119">The image is resized and sent to the inference driver, which communicates with the ML model to determine if there are any out of stock images.</span></span>
3. <span data-ttu-id="6dfc6-120">ML 모델은 모든 품절 영역을 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-120">The ML model returns any out of stock areas.</span></span>
4. <span data-ttu-id="6dfc6-121">추론 드라이버는 원시 이미지를 Blob(지정된 경우)에 업로드하고 모델의 결과를 디바이스의 Azure IoT Hub 및 경계 상자 프로세서로 보냅니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-121">The inferencing driver uploads the raw image to a blob (if specified), and sends the results from the model to Azure IoT Hub and a bounding box processor on the device.</span></span>
5. <span data-ttu-id="6dfc6-122">경계 상자 프로세서는 이미지에 경계 상자를 추가하고 메모리 내 데이터베이스에서 이미지 경로를 캐시합니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-122">The bounding box processor adds bounding boxes to the image and caches the image path in an in-memory database.</span></span>
6. <span data-ttu-id="6dfc6-123">웹앱은 이미지를 쿼리하고 받은 순서대로 표시합니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-123">The web app queries for images and shows them in the order received.</span></span>
7. <span data-ttu-id="6dfc6-124">IoT Hub의 메시지는 Time Series Insights에서 집계됩니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-124">Messages from IoT Hub are aggregated in Time Series Insights.</span></span>
8. <span data-ttu-id="6dfc6-125">Power BI는 Time Series Insights의 데이터를 사용하여 시간이 지남에 따라 품절된 품목에 대한 대화형 보고서를 표시합니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-125">Power BI displays an interactive report of out of stock items over time with the data from Time Series Insights.</span></span>


## <a name="components"></a><span data-ttu-id="6dfc6-126">구성 요소</span><span class="sxs-lookup"><span data-stu-id="6dfc6-126">Components</span></span>

<span data-ttu-id="6dfc6-127">이 솔루션은 다음과 같은 구성 요소를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-127">This solution uses the following components:</span></span>

| <span data-ttu-id="6dfc6-128">계층</span><span class="sxs-lookup"><span data-stu-id="6dfc6-128">Layer</span></span> | <span data-ttu-id="6dfc6-129">구성 요소</span><span class="sxs-lookup"><span data-stu-id="6dfc6-129">Component</span></span> | <span data-ttu-id="6dfc6-130">설명</span><span class="sxs-lookup"><span data-stu-id="6dfc6-130">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="6dfc6-131">온-프레미스 하드웨어</span><span class="sxs-lookup"><span data-stu-id="6dfc6-131">On-premises hardware</span></span> | <span data-ttu-id="6dfc6-132">네트워크 카메라</span><span class="sxs-lookup"><span data-stu-id="6dfc6-132">Network camera</span></span> | <span data-ttu-id="6dfc6-133">추론을 위한 이미지를 제공하는 HTTP 또는 RTSP 피드가 있는 네트워크 카메라가 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-133">A network camera is required, with either an HTTP or RTSP feed to provide the images for inference.</span></span> |
| <span data-ttu-id="6dfc6-134">Azure</span><span class="sxs-lookup"><span data-stu-id="6dfc6-134">Azure</span></span> | <span data-ttu-id="6dfc6-135">Azure IoT Hub</span><span class="sxs-lookup"><span data-stu-id="6dfc6-135">Azure IoT Hub</span></span> | <span data-ttu-id="6dfc6-136">[Azure IoT Hub](/azure/iot-hub/)는 에지 디바이스에 대한 디바이스 프로비저닝 및 메시지를 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-136">[Azure IoT Hub](/azure/iot-hub/) handles device provisioning and messaging for the edge devices.</span></span> |
|  | <span data-ttu-id="6dfc6-137">Azure Time Series Insights</span><span class="sxs-lookup"><span data-stu-id="6dfc6-137">Azure Time Series Insights</span></span> | <span data-ttu-id="6dfc6-138">[Azure Time Series Insights](/azure/time-series-insights/)는 시각화를 위해 IoT Hub의 메시지를 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-138">[Azure Time Series Insights](/azure/time-series-insights/) stores the messages from IoT Hub for visualization.</span></span> |
|  | <span data-ttu-id="6dfc6-139">Power BI</span><span class="sxs-lookup"><span data-stu-id="6dfc6-139">Power BI</span></span> | <span data-ttu-id="6dfc6-140">[Microsoft Power BI](https://powerbi.microsoft.com/)는 품절 이벤트에 대한 비즈니스 중심 보고서를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-140">[Microsoft Power BI](https://powerbi.microsoft.com/) provides business-focused reports of out of stock events.</span></span> <span data-ttu-id="6dfc6-141">Power BI는 Azure Stream Analytics의 출력을 볼 수 있는 간편한 대시보드 인터페이스를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-141">Power BI provides an easy-to-use dashboard interface for viewing the output from Azure Stream Analytics.</span></span> |
| <span data-ttu-id="6dfc6-142">Azure Stack Edge 또는</span><span class="sxs-lookup"><span data-stu-id="6dfc6-142">Azure Stack Edge or</span></span><br><span data-ttu-id="6dfc6-143">Azure IoT Edge 디바이스</span><span class="sxs-lookup"><span data-stu-id="6dfc6-143">Azure IoT Edge device</span></span> | <span data-ttu-id="6dfc6-144">Azure IoT Edge</span><span class="sxs-lookup"><span data-stu-id="6dfc6-144">Azure IoT Edge</span></span> | <span data-ttu-id="6dfc6-145">[Azure IoT Edge](/azure/iot-edge/)는 온-프레미스 컨테이너의 런타임을 오케스트레이션하고 디바이스 관리 및 업데이트를 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-145">[Azure IoT Edge](/azure/iot-edge/) orchestrates the runtime for the on-premises containers and handles device management and updates.</span></span>|
| | <span data-ttu-id="6dfc6-146">Azure Project Brainwave</span><span class="sxs-lookup"><span data-stu-id="6dfc6-146">Azure project brainwave</span></span> | <span data-ttu-id="6dfc6-147">Azure Stack Edge 디바이스에서 [Project Brainwave](https://blogs.microsoft.com/ai/build-2018-project-brainwave/)는 FPGA(Field-Programmable Gate Arrays)를 사용하여 ML 추론을 가속화합니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-147">On an Azure Stack Edge device, [Project Brainwave](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) uses Field-Programmable Gate Arrays (FPGAs) to accelerate ML inferencing.</span></span>|

## <a name="issues-and-considerations"></a><span data-ttu-id="6dfc6-148">문제 및 고려 사항</span><span class="sxs-lookup"><span data-stu-id="6dfc6-148">Issues and considerations</span></span>

<span data-ttu-id="6dfc6-149">이 솔루션을 구현할 방법을 결정할 때 다음 사항을 고려하세요.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-149">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="6dfc6-150">확장성</span><span class="sxs-lookup"><span data-stu-id="6dfc6-150">Scalability</span></span>

<span data-ttu-id="6dfc6-151">대부분의 기계 학습 모델은 제공된 하드웨어에 따라 초당 특정 프레임 수에서만 실행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-151">Most machine learning models can only run at a certain number of frames per second, depending on the provided hardware.</span></span> <span data-ttu-id="6dfc6-152">ML 파이프라인이 백업되지 않도록 카메라의 최적 샘플링 속도를 결정합니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-152">Determine the optimal sample rate from your camera(s) to ensure that the ML pipeline doesn't back up.</span></span> <span data-ttu-id="6dfc6-153">하드웨어 유형에 따라 카메라 수와 프레임 속도가 달라집니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-153">Different types of hardware will handle different numbers of cameras and frame rates.</span></span>

### <a name="availability"></a><span data-ttu-id="6dfc6-154">가용성</span><span class="sxs-lookup"><span data-stu-id="6dfc6-154">Availability</span></span>

<span data-ttu-id="6dfc6-155">에지 디바이스의 연결이 끊어진 경우 발생할 수 있는 상황을 고려하는 것이 중요합니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-155">It's important to consider what might happen if the edge device loses connectivity.</span></span> <span data-ttu-id="6dfc6-156">Time Series Insights 및 Power BI 대시보드에서 손실될 수 있는 데이터를 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-156">Consider what data might be lost from the Time Series Insights and Power BI dashboard.</span></span> <span data-ttu-id="6dfc6-157">제공된 예제 솔루션은 고가용성으로 디자인되지 않았습니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-157">The example solution as provided isn't designed to be highly available.</span></span>

### <a name="manageability"></a><span data-ttu-id="6dfc6-158">관리 효율</span><span class="sxs-lookup"><span data-stu-id="6dfc6-158">Manageability</span></span>

<span data-ttu-id="6dfc6-159">이 솔루션은 여러 디바이스 및 위치에 걸쳐 있을 수 있으므로 다루기가 어려울 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-159">This solution can span many devices and locations, which could get unwieldy.</span></span> <span data-ttu-id="6dfc6-160">Azure의 IoT 서비스는 자동으로 새 위치 및 디바이스를 온라인 상태로 전환하고 최신 상태로 유지할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-160">Azure's IoT services can automatically bring new locations and devices online and keep them up to date.</span></span> <span data-ttu-id="6dfc6-161">또한 적절한 데이터 거버넌스 절차를 따라야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-161">Proper data governance procedures must be followed as well.</span></span>

### <a name="security"></a><span data-ttu-id="6dfc6-162">보안</span><span class="sxs-lookup"><span data-stu-id="6dfc6-162">Security</span></span>

<span data-ttu-id="6dfc6-163">이 패턴은 잠재적으로 중요한 데이터를 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-163">This pattern handles potentially sensitive data.</span></span> <span data-ttu-id="6dfc6-164">키가 정기적으로 순환되고 Azure Storage 계정 및 로컬 공유에 대한 권한이 올바르게 설정되었는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-164">Make sure keys are regularly rotated and the permissions on the Azure Storage Account and local shares are correctly set.</span></span>

## <a name="next-steps"></a><span data-ttu-id="6dfc6-165">다음 단계</span><span class="sxs-lookup"><span data-stu-id="6dfc6-165">Next steps</span></span>

<span data-ttu-id="6dfc6-166">이 문서에서 소개하는 항목에 대한 자세한 내용은 다음을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-166">To learn more about topics introduced in this article:</span></span>
- <span data-ttu-id="6dfc6-167">이 패턴에는 [Azure IoT Edge](/azure/iot-edge/), [Azure IoT Hub](/azure/iot-hub/) 및 [Azure Time Series Insights](/azure/time-series-insights/)를 비롯한 여러 IoT 관련 서비스가 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-167">Multiple IoT related services are used in this pattern, including [Azure IoT Edge](/azure/iot-edge/), [Azure IoT Hub](/azure/iot-hub/), and [Azure Time Series Insights](/azure/time-series-insights/).</span></span>
- <span data-ttu-id="6dfc6-168">Microsoft Project Brainwave에 대한 자세한 내용은 [블로그 공지](https://blogs.microsoft.com/ai/build-2018-project-brainwave/)를 참조하고 [Azure Accelerated Machine Learning with Project Brainwave 비디오](https://www.youtube.com/watch?v=DJfMobMjCX0)를 확인하세요.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-168">To learn more about Microsoft Project Brainwave, see [the blog announcement](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) and checkout out the [Azure Accelerated Machine Learning with Project Brainwave video](https://www.youtube.com/watch?v=DJfMobMjCX0).</span></span>
- <span data-ttu-id="6dfc6-169">모범 사례에 대해 자세히 알아보고 추가 질문에 대한 답변을 얻으려면 [하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-169">See [Hybrid app design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers to any additional questions.</span></span>
- <span data-ttu-id="6dfc6-170">제품 및 솔루션의 전체 포트폴리오에 대해 자세히 알아보려면 [Azure Stack 제품군 및 솔루션](/azure-stack)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-170">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="6dfc6-171">솔루션 예제를 테스트할 준비가 되면 [에지 ML 추론 솔루션 배포 가이드](https://aka.ms/edgeinferencingdeploy)를 계속 진행하세요.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-171">When you're ready to test the solution example, continue with the [Edge ML inferencing solution deployment guide](https://aka.ms/edgeinferencingdeploy).</span></span> <span data-ttu-id="6dfc6-172">배포 가이드에서는 해당 구성 요소를 배포 및 테스트하는 방법에 대한 단계별 지침을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="6dfc6-172">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>
