---
title: Azure Stack 허브의 지리적으로 분산 되는 앱 패턴
description: Azure 및 Azure Stack Hub를 사용 하는 지능형에 지에 대 한 지리적으로 분산 된 앱 패턴에 대해 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 1f6243927390c7a520c2607c722664b2d31fc07f
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910882"
---
# <a name="geo-distributed-app-pattern"></a><span data-ttu-id="542d1-103">지리적으로 분산 되는 앱 패턴</span><span class="sxs-lookup"><span data-stu-id="542d1-103">Geo-distributed app pattern</span></span>

<span data-ttu-id="542d1-104">여러 지역에 걸쳐 앱 끝점을 제공 하 고 위치 및 규정 준수 요구 사항에 따라 사용자 트래픽을 라우팅하는 방법을 알아봅니다.</span><span class="sxs-lookup"><span data-stu-id="542d1-104">Learn how to provide app endpoints across multiple regions and route user traffic based on location and compliance needs.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="542d1-105">컨텍스트 및 문제점</span><span class="sxs-lookup"><span data-stu-id="542d1-105">Context and problem</span></span>

<span data-ttu-id="542d1-106">광범위 한 지역이 있는 조직에서는 데이터에 대 한 액세스를 안전 하 고 정확 하 게 배포 하 고 사용 하도록 설정 하는 동시에 사용자, 위치 및 장치 마다 사용자, 위치 및 장치에 대해 필요한 수준의 보안, 규정 준수 및 성능을</span><span class="sxs-lookup"><span data-stu-id="542d1-106">Organizations with wide-reaching geographies strive to securely and accurately distribute and enable access to data while ensuring required levels of security, compliance and performance per user, location, and device across borders.</span></span>

## <a name="solution"></a><span data-ttu-id="542d1-107">해결 방법</span><span class="sxs-lookup"><span data-stu-id="542d1-107">Solution</span></span>

<span data-ttu-id="542d1-108">Azure Stack 허브 지리적 트래픽 라우팅 패턴 또는 지리적으로 분산 된 앱을 사용 하면 다양 한 메트릭에 따라 트래픽을 특정 끝점으로 보낼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="542d1-108">The Azure Stack Hub geographic traffic routing pattern, or geo-distributed apps, lets traffic be directed to specific endpoints based on various metrics.</span></span> <span data-ttu-id="542d1-109">지리적 기반 라우팅 및 끝점 구성을 사용 하 여 Traffic Manager를 만들면 지역 요구 사항, 회사 및 국제 규정 및 데이터 요구 사항에 따라 끝점으로 트래픽을 라우팅합니다.</span><span class="sxs-lookup"><span data-stu-id="542d1-109">Creating a Traffic Manager with geographic-based routing and endpoint configuration routes traffic to endpoints based on regional requirements, corporate and international regulation, and data needs.</span></span>

![지리적으로 분산 되는 패턴](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a><span data-ttu-id="542d1-111">구성 요소</span><span class="sxs-lookup"><span data-stu-id="542d1-111">Components</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="542d1-112">클라우드 외부</span><span class="sxs-lookup"><span data-stu-id="542d1-112">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="542d1-113">Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="542d1-113">Traffic Manager</span></span>

<span data-ttu-id="542d1-114">다이어그램에서 Traffic Manager 공용 클라우드 외부에 위치 하지만 로컬 데이터 센터와 공용 클라우드 모두에서 트래픽을 조정할 수 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="542d1-114">In the diagram, Traffic Manager is located outside of the public cloud, but it needs to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="542d1-115">부하 분산 장치는 트래픽을 지리적 위치로 라우팅합니다.</span><span class="sxs-lookup"><span data-stu-id="542d1-115">The balancer routes traffic to geographical locations.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="542d1-116">DNS(Domain Name System)</span><span class="sxs-lookup"><span data-stu-id="542d1-116">Domain Name System (DNS)</span></span>

<span data-ttu-id="542d1-117">Domain Name System, 즉 DNS는 웹 사이트 또는 서비스 이름을 해당 IP 주소로 변환(또는 확인)합니다.</span><span class="sxs-lookup"><span data-stu-id="542d1-117">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="public-cloud"></a><span data-ttu-id="542d1-118">퍼블릭 클라우드</span><span class="sxs-lookup"><span data-stu-id="542d1-118">Public cloud</span></span>

#### <a name="cloud-endpoint"></a><span data-ttu-id="542d1-119">클라우드 엔드포인트</span><span class="sxs-lookup"><span data-stu-id="542d1-119">Cloud Endpoint</span></span>

<span data-ttu-id="542d1-120">공용 IP 주소는 traffic manager를 통해 들어오는 트래픽을 공용 클라우드 앱 리소스 끝점으로 라우팅하는 데 사용 됩니다.</span><span class="sxs-lookup"><span data-stu-id="542d1-120">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-clouds"></a><span data-ttu-id="542d1-121">로컬 클라우드</span><span class="sxs-lookup"><span data-stu-id="542d1-121">Local clouds</span></span>

#### <a name="local-endpoint"></a><span data-ttu-id="542d1-122">로컬 끝점</span><span class="sxs-lookup"><span data-stu-id="542d1-122">Local endpoint</span></span>

<span data-ttu-id="542d1-123">공용 IP 주소는 traffic manager를 통해 들어오는 트래픽을 공용 클라우드 앱 리소스 끝점으로 라우팅하는 데 사용 됩니다.</span><span class="sxs-lookup"><span data-stu-id="542d1-123">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="542d1-124">문제 및 고려 사항</span><span class="sxs-lookup"><span data-stu-id="542d1-124">Issues and considerations</span></span>

<span data-ttu-id="542d1-125">이 패턴을 구현할 방법을 결정할 때 다음 사항을 고려하세요.</span><span class="sxs-lookup"><span data-stu-id="542d1-125">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="542d1-126">확장성</span><span class="sxs-lookup"><span data-stu-id="542d1-126">Scalability</span></span>

<span data-ttu-id="542d1-127">패턴은 트래픽 증가량을 충족 하도록 크기를 조정 하는 대신 지리 트래픽 라우팅을 처리 합니다.</span><span class="sxs-lookup"><span data-stu-id="542d1-127">The pattern handles geographical traffic routing rather than scaling to meet increases in traffic.</span></span> <span data-ttu-id="542d1-128">그러나이 패턴을 다른 Azure 및 온-프레미스 솔루션과 결합할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="542d1-128">However, you can combine this pattern with other Azure and on-premises solutions.</span></span> <span data-ttu-id="542d1-129">예를 들어이 패턴은 클라우드 간 배율 패턴과 함께 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="542d1-129">For example, this pattern can be used with the cross-cloud scaling Pattern.</span></span>

### <a name="availability"></a><span data-ttu-id="542d1-130">가용성</span><span class="sxs-lookup"><span data-stu-id="542d1-130">Availability</span></span>

<span data-ttu-id="542d1-131">로컬로 배포 된 앱이 온-프레미스 하드웨어 구성 및 소프트웨어 배포를 통해 고가용성으로 구성 되어 있는지 확인 합니다.</span><span class="sxs-lookup"><span data-stu-id="542d1-131">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="542d1-132">관리 효율</span><span class="sxs-lookup"><span data-stu-id="542d1-132">Manageability</span></span>

<span data-ttu-id="542d1-133">패턴은 환경 간에 원활한 관리 및 친숙 한 인터페이스를 보장 합니다.</span><span class="sxs-lookup"><span data-stu-id="542d1-133">The pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="542d1-134">이 패턴을 사용해야 하는 경우</span><span class="sxs-lookup"><span data-stu-id="542d1-134">When to use this pattern</span></span>

- <span data-ttu-id="542d1-135">조직에 사용자 지정 지역 보안 및 배포 정책을 요구 하는 국제 분기가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="542d1-135">My organization has international branches requiring custom regional security and distribution policies.</span></span>
- <span data-ttu-id="542d1-136">각 조직 사무소는 직원, 비즈니스 및 시설 데이터를 가져와서 로컬 규정과 표준 시간대에 대 한 보고 작업을 요구 합니다.</span><span class="sxs-lookup"><span data-stu-id="542d1-136">Each of my organization's offices pulls employee, business, and facility data, requiring reporting activity per local regulations and time zone.</span></span>
- <span data-ttu-id="542d1-137">단일 지역 내에서 여러 개의 앱 배포를 수행 하 고 여러 지역에서 부하 요구 사항을 처리 하기 위해 여러 앱 배포를 사용 하 여 앱을 수평 확장 하 여 대규모 요구 사항을 충족할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="542d1-137">High-scale requirements can be met by horizontally scaling out apps, with multiple app deployments being made within a single region and across regions to handle extreme load requirements.</span></span>
- <span data-ttu-id="542d1-138">단일 지역 중단이 발생 하더라도 앱은 항상 사용 가능 하 고 클라이언트 요청에 응답 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="542d1-138">The apps must be highly available and responsive to client requests even in single-region outages.</span></span>

## <a name="next-steps"></a><span data-ttu-id="542d1-139">다음 단계</span><span class="sxs-lookup"><span data-stu-id="542d1-139">Next steps</span></span>

<span data-ttu-id="542d1-140">이 문서에서 소개 하는 항목에 대 한 자세한 내용은 다음을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="542d1-140">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="542d1-141">이 DNS 기반 트래픽 부하 분산 장치 작동 방법에 대해 자세히 알아보려면 [Azure Traffic Manager 개요](/azure/traffic-manager/traffic-manager-overview) 를 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="542d1-141">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="542d1-142">모범 사례에 대 한 자세한 내용을 알아보고 추가 질문에 대 한 답변을 얻으려면 [하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="542d1-142">See [Hybrid app design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="542d1-143">제품 및 솔루션의 전체 포트폴리오에 대해 자세히 알아보려면 [Azure Stack 제품군 및 솔루션](/azure-stack) 을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="542d1-143">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="542d1-144">솔루션 예제를 테스트할 준비가 되 면 [지리적으로 분산 된 앱 솔루션 배포 가이드](solution-deployment-guide-geo-distributed.md)를 계속 진행 합니다.</span><span class="sxs-lookup"><span data-stu-id="542d1-144">When you're ready to test the solution example, continue with the [Geo-distributed app solution deployment guide](solution-deployment-guide-geo-distributed.md).</span></span> <span data-ttu-id="542d1-145">배포 가이드에서는 구성 요소를 배포 및 테스트 하는 방법에 대 한 단계별 지침을 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="542d1-145">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="542d1-146">지리적으로 분산 된 응용 프로그램 패턴을 사용 하 여 다양 한 메트릭에 따라 특정 끝점으로 트래픽을 전송 하는 방법을 알아봅니다.</span><span class="sxs-lookup"><span data-stu-id="542d1-146">You learn how to direct traffic to specific endpoints, based on various metrics using the geo-distributed app pattern.</span></span> <span data-ttu-id="542d1-147">지리적 기반 라우팅 및 끝점 구성을 사용 하 여 Traffic Manager 프로필을 만들면 정보가 지역 요구 사항, 회사 및 국제 규정 및 데이터 요구 사항에 따라 끝점으로 라우팅됩니다.</span><span class="sxs-lookup"><span data-stu-id="542d1-147">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>
