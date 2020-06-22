---
title: Azure 및 Azure Stack Hub의 하이브리드 릴레이 패턴
description: Azure의 하이브리드 릴레이 패턴과 Azure Stack 허브를 사용 하 여 방화벽으로 보호 되는에 지 리소스에 연결 합니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 03b20a20a04f620c977fb20e1ea26f5982e42721
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911148"
---
# <a name="hybrid-relay-pattern"></a><span data-ttu-id="ab56b-103">하이브리드 릴레이 패턴</span><span class="sxs-lookup"><span data-stu-id="ab56b-103">Hybrid relay pattern</span></span>

<span data-ttu-id="ab56b-104">하이브리드 릴레이 패턴 및 Azure Relay를 사용 하 여 방화벽으로 보호 되는에 지 리소스 또는 장치에 연결 하는 방법을 알아봅니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-104">Learn how to connect to edge resources or devices protected by firewalls using the hybrid relay pattern and Azure Relay.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="ab56b-105">컨텍스트 및 문제점</span><span class="sxs-lookup"><span data-stu-id="ab56b-105">Context and problem</span></span>

<span data-ttu-id="ab56b-106">Edge 장치는 종종 회사 방화벽이 나 NAT 장치 뒤에 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-106">Edge devices are often behind a corporate firewall or NAT device.</span></span> <span data-ttu-id="ab56b-107">보안은 안전 하지만 다른 회사 네트워크의 공용 클라우드 또는에 지 장치와 통신 하지 못할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-107">Although they're secure, they may be unable to communicate with the public cloud or edge devices on other corporate networks.</span></span> <span data-ttu-id="ab56b-108">공용 클라우드의 사용자에 게 특정 포트 및 기능을 안전 하 게 노출 하는 것이 필요할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-108">It may be necessary to expose certain ports and functionality to users in the public cloud in a secure manner.</span></span>

## <a name="solution"></a><span data-ttu-id="ab56b-109">해결 방법</span><span class="sxs-lookup"><span data-stu-id="ab56b-109">Solution</span></span>

<span data-ttu-id="ab56b-110">하이브리드 릴레이 패턴은 Azure Relay를 사용 하 여 직접 통신할 수 없는 두 끝점 간에 Websocket 터널을 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-110">The hybrid relay pattern uses Azure Relay to establish a WebSockets tunnel between two endpoints that can't directly communicate.</span></span> <span data-ttu-id="ab56b-111">온-프레미스에는 없지만 온-프레미스 끝점에 연결 해야 하는 장치는 공용 클라우드의 끝점에 연결 됩니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-111">Devices that aren't on-premises but need to connect to an on-premises endpoint will connect to an endpoint in the public cloud.</span></span> <span data-ttu-id="ab56b-112">이 끝점은 보안 채널을 통해 미리 정의 된 경로에서 트래픽을 리디렉션합니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-112">This endpoint will redirect the traffic on predefined routes over a secure channel.</span></span> <span data-ttu-id="ab56b-113">온-프레미스 환경 내의 끝점은 트래픽을 수신 하 고 올바른 대상으로 라우팅합니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-113">An endpoint inside the on-premises environment receives the traffic and routes it to the correct destination.</span></span>

![하이브리드 릴레이 패턴 솔루션 아키텍처](media/pattern-hybrid-relay/solution-architecture.png)

<span data-ttu-id="ab56b-115">하이브리드 릴레이 패턴의 작동 원리는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-115">Here's how the hybrid relay pattern works:</span></span>

1. <span data-ttu-id="ab56b-116">장치는 미리 정의 된 포트에서 Azure의 VM (가상 머신)에 연결 됩니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-116">A device connects to the virtual machine (VM) in Azure, on a predefined port.</span></span>
2. <span data-ttu-id="ab56b-117">트래픽이 Azure의 Azure Relay 전달 됩니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-117">Traffic is forwarded to the Azure Relay in Azure.</span></span>
3. <span data-ttu-id="ab56b-118">Azure Relay에 대 한 장기 연결을 이미 설정한 Azure Stack 허브의 VM은 트래픽을 수신 하 고 대상에 전달 합니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-118">The VM on Azure Stack Hub, which has already established a long-lived connection to the Azure Relay, receives the traffic and forwards it on to the destination.</span></span>
4. <span data-ttu-id="ab56b-119">온-프레미스 서비스 또는 끝점에서 요청을 처리 합니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-119">The on-premises service or endpoint processes the request.</span></span>

## <a name="components"></a><span data-ttu-id="ab56b-120">구성 요소</span><span class="sxs-lookup"><span data-stu-id="ab56b-120">Components</span></span>

<span data-ttu-id="ab56b-121">이 솔루션은 다음 구성 요소를 사용 합니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-121">This solution uses the following components:</span></span>

| <span data-ttu-id="ab56b-122">계층</span><span class="sxs-lookup"><span data-stu-id="ab56b-122">Layer</span></span> | <span data-ttu-id="ab56b-123">구성 요소</span><span class="sxs-lookup"><span data-stu-id="ab56b-123">Component</span></span> | <span data-ttu-id="ab56b-124">Description</span><span class="sxs-lookup"><span data-stu-id="ab56b-124">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="ab56b-125">Azure</span><span class="sxs-lookup"><span data-stu-id="ab56b-125">Azure</span></span> | <span data-ttu-id="ab56b-126">Azure VM</span><span class="sxs-lookup"><span data-stu-id="ab56b-126">Azure VM</span></span> | <span data-ttu-id="ab56b-127">Azure VM은 온-프레미스 리소스에 대해 공개적으로 액세스할 수 있는 끝점을 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-127">An Azure VM provides a publicly accessible endpoint for the on-premises resource.</span></span> |
| | <span data-ttu-id="ab56b-128">Azure Relay</span><span class="sxs-lookup"><span data-stu-id="ab56b-128">Azure Relay</span></span> | <span data-ttu-id="ab56b-129">[Azure Relay](/azure/azure-relay/) 은 Azure vm 및 AZURE STACK 허브 vm 간의 터널 및 연결을 유지 관리 하기 위한 인프라를 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-129">An [Azure Relay](/azure/azure-relay/) provides the infrastructure for maintaining the tunnel and connection between the Azure VM and Azure Stack Hub VM.</span></span>|
| <span data-ttu-id="ab56b-130">Azure Stack 허브</span><span class="sxs-lookup"><span data-stu-id="ab56b-130">Azure Stack Hub</span></span> | <span data-ttu-id="ab56b-131">Compute</span><span class="sxs-lookup"><span data-stu-id="ab56b-131">Compute</span></span> | <span data-ttu-id="ab56b-132">Azure Stack 허브 VM은 하이브리드 릴레이 터널의 서버 쪽을 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-132">An Azure Stack Hub VM provides the server-side of the Hybrid Relay tunnel.</span></span> |
| | <span data-ttu-id="ab56b-133">Storage</span><span class="sxs-lookup"><span data-stu-id="ab56b-133">Storage</span></span> | <span data-ttu-id="ab56b-134">Azure Stack 허브에 배포 된 AKS 엔진 클러스터는 Face API 컨테이너를 실행할 수 있는 확장 가능한 복원 엔진을 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-134">The AKS engine cluster deployed into Azure Stack Hub provides a scalable, resilient engine to run the Face API container.</span></span>|

## <a name="issues-and-considerations"></a><span data-ttu-id="ab56b-135">문제 및 고려 사항</span><span class="sxs-lookup"><span data-stu-id="ab56b-135">Issues and considerations</span></span>

<span data-ttu-id="ab56b-136">이 솔루션을 구현 하는 방법을 결정할 때 다음 사항을 고려 하십시오.</span><span class="sxs-lookup"><span data-stu-id="ab56b-136">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="ab56b-137">확장성</span><span class="sxs-lookup"><span data-stu-id="ab56b-137">Scalability</span></span>

<span data-ttu-id="ab56b-138">이 패턴은 클라이언트와 서버에서 1:1 포트 매핑만 허용 합니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-138">This pattern only allows for 1:1 port mappings on the client and server.</span></span> <span data-ttu-id="ab56b-139">예를 들어, 포트 80가 Azure 끝점의 한 서비스에 대해 터널링 되는 경우 다른 서비스에 사용할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-139">For example, if port 80 is tunneled for one service on the Azure endpoint, it can't be used for another service.</span></span> <span data-ttu-id="ab56b-140">포트 매핑을 적절 하 게 계획 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-140">Port mappings should be planned accordingly.</span></span> <span data-ttu-id="ab56b-141">트래픽을 처리 하려면 Azure Relay 및 Vm의 크기를 적절히 조정 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-141">The Azure Relay and VMs should be appropriately scaled to handle traffic.</span></span>

### <a name="availability"></a><span data-ttu-id="ab56b-142">가용성</span><span class="sxs-lookup"><span data-stu-id="ab56b-142">Availability</span></span>

<span data-ttu-id="ab56b-143">이러한 터널 및 연결은 중복 되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-143">These tunnels and connections aren't redundant.</span></span> <span data-ttu-id="ab56b-144">고가용성을 보장 하기 위해 오류 검사 코드를 구현할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-144">To ensure high-availability, you may want to implement error checking code.</span></span> <span data-ttu-id="ab56b-145">또 다른 옵션은 부하 분산 장치 뒤에 Azure Relay 연결 된 Vm의 풀을 포함 하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-145">Another option is to have a pool of Azure Relay-connected VMs behind a load balancer.</span></span>

### <a name="manageability"></a><span data-ttu-id="ab56b-146">관리 효율</span><span class="sxs-lookup"><span data-stu-id="ab56b-146">Manageability</span></span>

<span data-ttu-id="ab56b-147">이 솔루션은 많은 장치 및 위치에 걸쳐 있을 수 있으므로 다루기가 어려울 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-147">This solution can span many devices and locations, which could get unwieldy.</span></span> <span data-ttu-id="ab56b-148">Azure의 IoT 서비스는 자동으로 새 위치 및 장치를 온라인 상태로 전환 하 고 최신 상태로 유지할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-148">Azure's IoT services can automatically bring new locations and devices online and keep them up to date.</span></span>

### <a name="security"></a><span data-ttu-id="ab56b-149">보안</span><span class="sxs-lookup"><span data-stu-id="ab56b-149">Security</span></span>

<span data-ttu-id="ab56b-150">표시 된 것 처럼이 패턴을 사용 하면에 지에서 내부 장치의 포트에 무제한적인 액세스할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-150">This pattern as shown allows for unfettered access to a port on an internal device from the edge.</span></span> <span data-ttu-id="ab56b-151">내부 장치 또는 하이브리드 릴레이 끝점 앞에 있는 서비스에 인증 메커니즘을 추가 하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-151">Consider adding an authentication mechanism to the service on the internal device, or in front of the hybrid relay endpoint.</span></span>

## <a name="next-steps"></a><span data-ttu-id="ab56b-152">다음 단계</span><span class="sxs-lookup"><span data-stu-id="ab56b-152">Next steps</span></span>

<span data-ttu-id="ab56b-153">이 문서에서 소개 하는 항목에 대 한 자세한 내용은 다음을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="ab56b-153">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="ab56b-154">이 패턴은 Azure Relay를 사용 합니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-154">This pattern uses Azure Relay.</span></span> <span data-ttu-id="ab56b-155">자세한 내용은 [Azure Relay 설명서](/azure/azure-relay/)를 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="ab56b-155">For more information, see the [Azure Relay documentation](/azure/azure-relay/).</span></span>
- <span data-ttu-id="ab56b-156">모범 사례에 대 한 자세한 내용을 알아보고 추가 질문에 대 한 답변을 얻으려면 [하이브리드 응용 프로그램 디자인 고려 사항](overview-app-design-considerations.md) 을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="ab56b-156">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and get answers to any additional questions.</span></span>
- <span data-ttu-id="ab56b-157">제품 및 솔루션의 전체 포트폴리오에 대해 자세히 알아보려면 [Azure Stack 제품군 및 솔루션](/azure-stack) 을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="ab56b-157">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="ab56b-158">솔루션 예제를 테스트할 준비가 되 면 [하이브리드 릴레이 솔루션 배포 가이드](https://aka.ms/hybridrelaydeployment)를 계속 진행 합니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-158">When you're ready to test the solution example, continue with the [Hybrid relay solution deployment guide](https://aka.ms/hybridrelaydeployment).</span></span> <span data-ttu-id="ab56b-159">배포 가이드에서는 구성 요소를 배포 및 테스트 하는 방법에 대 한 단계별 지침을 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="ab56b-159">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>