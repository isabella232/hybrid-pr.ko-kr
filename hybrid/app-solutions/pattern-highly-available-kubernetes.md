---
title: Azure 및 Azure Stack Hub를 사용하는 고가용성 Kubernetes 패턴
description: Kubernetes 클러스터 솔루션에서 Azure 및 Azure Stack Hub를 사용하여 고가용성을 제공하는 방법을 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: f8a733bcdab871695e552ec687d42e3ff4230490
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281315"
---
# <a name="high-availability-kubernetes-cluster-pattern"></a>고가용성 Kubernetes 클러스터 패턴

이 문서에서는 Azure Stack Hub에서 AKS(Azure Kubernetes Service) 엔진을 사용하여 고가용성 Kubernetes 기반 인프라를 설계하고 작동하는 방법에 대해 설명합니다. 이 시나리오는 매우 제한적이고 규제가 엄격한 환경에서 중요한 워크로드가 있는 조직에서 일반적으로 사용됩니다. 이러한 조직으로 금융, 국방 및 정부와 같은 도메인에 속한 조직이 있습니다.

## <a name="context-and-problem"></a>컨텍스트 및 문제점

많은 조직에서 Kubernetes와 같은 최신 서비스와 기술을 활용하는 클라우드 네이티브 솔루션을 개발하고 있습니다. Azure는 전 세계 대부분의 지역에서 데이터 센터를 제공하지만, 경우에 따라 특정 위치에서 중요 비즈니스용 애플리케이션을 실행해야 하는 에지 사용 사례 및 시나리오가 있습니다. 고려해야 할 사항은 다음과 같습니다.

- 위치 민감도
- 애플리케이션 및 온-프레미스 시스템 간의 대기 시간
- 대역폭 보존
- 연결
- 규정 또는 법적 요구 사항

Azure는 Azure Stack Hub와 결합하여 대부분의 이러한 문제를 해결합니다. Azure Stack Hub에서 실행되는 Kubernetes의 성공적인 구현을 위한 광범위한 옵션, 결정 및 고려 사항은 아래에서 설명하고 있습니다.

## <a name="solution"></a>해결 방법

이 패턴에서는 엄격한 제약 조건 세트를 처리해야 한다고 가정합니다. 애플리케이션은 온-프레미스에서 실행되어야 하며, 모든 개인 데이터가 퍼블릭 클라우드 서비스에 연결되지 않아야 합니다. 모니터링 및 기타 비 PII 데이터는 Azure에 보내 처리할 수 있습니다. 퍼블릭 Container Registry 등과 같은 외부 서비스에 액세스할 수 있지만 방화벽 또는 프록시 서버를 통해 필터링할 수 있습니다.

여기서 보여 주는 애플리케이션 예제([Azure Kubernetes Service 워크샵](/learn/modules/aks-workshop/) 기반)는 가능한 경우 언제든지 Kubernetes 네이티브 솔루션을 사용하도록 설계되었습니다. 이 디자인은 플랫폼 네이티브 서비스를 사용하는 대신 공급업체에 한정되지 않도록 방지합니다. 예를 들어 애플리케이션에서 PaaS 서비스 또는 외부 데이터베이스 서비스 대신 자체 호스팅 MongoDB 데이터베이스 백 엔드를 사용합니다.

[![애플리케이션 패턴 하이브리드](media/pattern-highly-available-kubernetes/application-architecture.png)](media/pattern-highly-available-kubernetes/application-architecture.png#lightbox)

앞의 다이어그램에서는 Azure Stack Hub의 Kubernetes에서 실행되는 애플리케이션 예제의 애플리케이션 아키텍처를 보여 줍니다. 앱을 구성하는 몇 가지 구성 요소는 다음과 같습니다.

 1) AKS Azure Stack Hub의 AKS 엔진 기반 Kubernetes 클러스터
 2) [cert-manager](https://www.jetstack.io/cert-manager/) - Let's Encrypt에서 인증서를 자동으로 요청하는 데 사용되는 Kubernetes의 인증서 관리 도구 모음을 제공합니다.
 3) Kubernetes 네임스페이스 - 프런트 엔드(ratings-web), API(ratings-api) 및 데이터베이스(ratings-mongodb)용 애플리케이션 구성 요소가 포함되어 있습니다.
 4) 수신 컨트롤러 - HTTP/HTTPS 트래픽을 Kubernetes 클러스터 내의 엔드포인트로 라우팅합니다.

애플리케이션 예제는 애플리케이션 아키텍처를 설명하는 데 사용됩니다. 모든 구성 요소는 예입니다. 아키텍처에는 단일 애플리케이션 배포만 포함됩니다. HA(고가용성)를 달성하기 위해 두 개의 서로 다른 Azure Stack Hub 인스턴스에서 배포를 두 번 이상 실행합니다. 이러한 인스턴스는 동일한 위치 또는 둘 이상의 서로 다른 사이트에서 실행될 수 있습니다.

![인프라 아키텍처](media/pattern-highly-available-kubernetes/aks-azure-architecture.png)

Azure Container Registry, Azure Monitor 등과 같은 서비스는 Azure 또는 온-프레미스의 Azure Stack Hub 외부에서 호스팅됩니다. 이 하이브리드 디자인은 단일 Azure Stack Hub 인스턴스의 중단으로부터 솔루션을 보호합니다.

## <a name="components"></a>구성 요소

전체 아키텍처를 구성하는 구성 요소는 다음과 같습니다.

**Azure Stack Hub** 는 데이터 센터에서 Azure 서비스를 제공하여 온-프레미스 환경에서 워크로드를 실행할 수 있는 Azure의 확장입니다. 자세한 내용은 [Azure Stack Hub 개요](/azure-stack/operator/azure-stack-overview)를 참조하세요.

**AKS 엔진(Azure Kubernetes Service 엔진)** 은 현재 Azure에서 사용할 수 있는 관리되는 Kubernetes 서비스 제품인 AKS(Azure Kubernetes Service)를 지원하는 엔진입니다. Azure Stack Hub의 경우 AKS 엔진을 사용하면 Azure Stack Hub의 IaaS 기능을 사용하여 모든 기능을 갖춘 자체적으로 관리되는 Kubernetes 클러스터를 배포, 크기 조정 및 업그레이드할 수 있습니다. 자세한 내용은 [AKS 엔진 개요](https://github.com/Azure/aks-engine)를 참조하세요.

Azure의 AKS 엔진 및 Azure Stack Hub의 AKS 엔진 간의 차이점에 대한 자세한 내용은 [알려진 문제 및 제한 사항](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#known-issues-and-limitations)을 참조하세요.

**Azure VNet(Virtual Network)** 은 Kubernetes 클러스터 인프라를 호스팅하는 VMs(Virtual Machines)의 각 Azure Stack Hub에서 네트워크 인프라를 제공하는 데 사용됩니다.

**Azure Load Balancer** 는 Kubernetes API 엔드포인트 및 Nginx 수신 컨트롤러에 사용됩니다. 부하 분산 장치는 외부(예: 인터넷) 트래픽을 특정 서비스를 제공하는 노드 및 VM으로 라우팅합니다.

**ACR(Azure Container Registry)** 은 클러스터에 배포되는 프라이빗 Docker 이미지 및 Helm 차트를 저장하는 데 사용됩니다. AKS 엔진은 Azure AD ID를 사용하여 Container Registry를 통해 인증할 수 있습니다. Kubernetes에는 ACR이 필요하지 않습니다. Docker 허브 같은 다른 컨테이너 레지스트리를 사용할 수 있습니다.

**Azure Repos** 는 코드를 관리하는 데 사용할 수 있는 버전 제어 도구 세트입니다. GitHub 또는 기타 git 기반 리포지토리를 사용할 수도 있습니다. 자세한 내용은 [Azure Repos 개요](/azure/devops/repos/get-started/what-is-repos)를 참조하세요.

**Azure Pipelines** 는 Azure DevOps Services의 일부이며 자동화된 빌드, 테스트 및 배포를 실행합니다. Jenkins 같은 타사 CI/CD 솔루션도 사용할 수 있습니다. 자세한 내용은 [Azure Pipelines 개요](/azure/devops/pipelines/get-started/what-is-azure-pipelines)를 참조하세요.

**Azure Monitor** 는 솔루션 및 애플리케이션 원격 분석의 Azure 서비스에 대한 플랫폼 메트릭을 포함하여 메트릭 및 로그를 수집 및 저장합니다. 이 데이터를 사용하여 애플리케이션을 모니터링하고, 경고 및 대시보드를 설정하고, 오류의 근본 원인을 분석할 수 있습니다. Azure Monitor는 Kubernetes와 통합되어 컨트롤러, 노드 및 컨테이너뿐만 아니라 컨테이너 로그 및 마스터 노드 로그에서도 메트릭을 수집합니다. 자세한 내용은 [Azure Monitor 개요](/azure/azure-monitor/overview)를 참조하세요.

**Azure Traffic Manager** 는 다양한 Azure 지역 또는 Azure Stack Hub 배포에서 트래픽을 서비스에 최적으로 분산할 수 있도록 하는 DNS 기반 트래픽 부하 분산 장치입니다. 또한 Traffic Manager는 고가용성 및 응답성을 제공합니다. 애플리케이션 엔드포인트는 외부에서 액세스할 수 있어야 합니다. 다른 온-프레미스 솔루션도 사용할 수 있습니다.

**Kubernetes 수신 컨트롤러** 는 HTTP(S) 경로를 Kubernetes 클러스터의 서비스에 공개합니다. 이를 위해 Nginx 또는 적절한 수신 컨트롤러를 사용할 수 있습니다.

**Helm** 은 배포, 서비스, 비밀과 같은 다양한 Kubernetes 개체를 단일 "차트"로 묶을 수 있는 방법을 제공하는 Kubernetes 배포용 패키지 관리자입니다. 버전 관리를 게시, 배포 및 제어하고 차트 개체를 업데이트할 수 있습니다. Azure Container Registry는 패키지된 Helm 차트를 저장하는 리포지토리로 사용할 수 있습니다.

## <a name="design-considerations"></a>디자인 고려 사항

이 패턴에서는 이 문서의 다음 섹션에서 자세히 설명하는 몇 가지 개략적인 고려 사항을 따릅니다.

- 애플리케이션은 공급업체에 한정되지 않도록 방지하기 위해 Kubernetes 네이티브 솔루션을 사용합니다.
- 애플리케이션에서 마이크로서비스 아키텍처를 사용합니다.
- Azure Stack Hub에는 인바운드가 필요하지 않지만 아웃바운드 인터넷 연결이 허용됩니다.

이러한 추천 사례는 실제 워크로드 및 시나리오에도 적용됩니다.

## <a name="scalability-considerations"></a>확장성 고려 사항

확장성은 사용자에게 애플리케이션에 대한 일관되고, 안정적이며, 효율적으로 수행되는 액세스를 제공하는 데 중요합니다.

샘플 시나리오에서는 애플리케이션 스택의 여러 계층에 대한 확장성을 다루고 있습니다. 여러 계층에 대한 개략적인 개요는 다음과 같습니다.

| 아키텍처 수준 | 영향 | 방법은? |
| --- | --- | ---
| 애플리케이션 | 애플리케이션 | Pod/복제본/Container Instances 수에 따른 수평 크기 조정* |
| 클러스터 | Kubernetes 클러스터 | 노드(1~50개), VM-SKU 크기 및 노드 풀(Azure Stack Hub의 AKS 엔진은 현재 단일 노드 풀만 지원함)의 수 - AKS 엔진의 크기 조정 명령 사용(수동) |
| 인프라 | Azure Stack Hub | Azure Stack Hub 배포 내의 노드, 용량 및 배율 단위의 수 |

\* Kubernetes의 HPA(수평 Pod 자동 크기 조정기)를 사용합니다. 이는 컨테이너 인스턴스 크기(CPU/메모리)를 조정하여 자동화된 메트릭 기반 크기 조정 또는 수직 크기 조정입니다.

**Azure Stack Hub(인프라 수준)**

Azure Stack Hub가 데이터 센터의 실제 하드웨어에서 실행되므로 Azure Stack Hub 인프라는 이 구현의 기반입니다. 허브 하드웨어를 선택하는 경우 CPU, 메모리 밀도, 스토리지 구성 및 서버 수를 선택해야 합니다. Azure Stack Hub의 확장성에 대해 자세히 알아보려면 다음 리소스를 확인하세요.

- [Azure Stack Hub에 대한 용량 계획 개요](/azure-stack/operator/azure-stack-capacity-planning-overview)
- [Azure Stack Hub에 추가 배율 단위 노드 추가](/azure-stack/operator/azure-stack-add-scale-node)

**Kubernetes 클러스터(클러스터 수준)**

Kubernetes 클러스터 자체는 컴퓨팅, 스토리지 및 네트워크 리소스를 포함한 Azure(Stack) IaaS 구성 요소로 구성되며, 이를 기반으로 하여 빌드됩니다. Kubernetes 솔루션에는 Azure(및 Azure Stack Hub)에서 VM으로 배포되는 마스터 및 작업자 노드가 포함됩니다.

- [컨트롤 플레인 노드](/azure/aks/concepts-clusters-workloads#control-plane)(마스터)는 핵심 Kubernetes 서비스 및 애플리케이션 워크로드 오케스트레이션을 제공합니다.
- [작업자 노드](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools)(작업자)는 애플리케이션 워크로드를 실행합니다.

초기 배포에 사용할 VM 크기를 선택하는 경우 다음과 같은 몇 가지 고려 사항이 있습니다.  

- **비용** - 작업자 노드를 계획하는 경우 발생하는 VM당 전체 비용을 고려해야 합니다. 예를 들어 애플리케이션 워크로드에 제한된 리소스가 필요한 경우 더 작은 크기의 VM을 배포하도록 계획해야 합니다. Azure와 마찬가지로, Azure Stack Hub는 일반적으로 사용량을 기준으로 청구되므로 Kubernetes 역할에 맞게 VM 크기를 적절하게 조정하는 것은 사용량 비용을 최적화하는 데 매우 중요합니다. 

- **확장성** - 클러스터의 확장성은 마스터 및 작업자 노드의 수를 축소 및 확장하거나 추가 노드 풀을 추가하여 달성됩니다(현재 Azure Stack Hub에서는 사용할 수 없음). 클러스터 크기 조정은 Container Insights(Azure Monitor + Log Analytics)를 사용하여 수집된 성능 데이터를 기반으로 하여 수행할 수 있습니다. 

    애플리케이션에 더 많은(또는 더 적은) 리소스가 필요한 경우 현재 노드를 수평으로 확장(또는 축소)할 수 있습니다(1~50개 노드). 50개보다 많은 노드가 필요한 경우 별도의 구독에서 추가 클러스터를 만들 수 있습니다. 클러스터를 다시 배포해야 실제 VM을 다른 VM 크기로 수직으로 스케일 업할 수 있습니다.

    크기 조정은 처음에 Kubernetes 클러스터를 배포하는 데 사용된 AKS 엔진 도우미 VM을 사용하여 수동으로 수행됩니다. 자세한 내용은 [Kubernetes 클러스터 크기 조정](https://github.com/Azure/aks-engine/blob/master/docs/topics/scale.md)을 참조하세요.

- **할당량** - Azure Stack Hub에서 AKS 배포를 계획하는 경우 구성한 [할당량](/azure-stack/operator/azure-stack-quota-types)을 고려합니다. 각 [구독](/azure-stack/operator/service-plan-offer-subscription-overview)에 적절한 계획과 할당량이 구성되어 있는지 확인합니다. 규모가 확장될 때 구독은 클러스터에 필요한 컴퓨팅, 스토리지 및 기타 서비스의 양을 수용해야 합니다.

- **애플리케이션 워크로드** - Azure Kubernetes Service에 대한 Kubernetes 핵심 개념 문서에서 [클러스터 및 워크로드 개념](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools)을 참조합니다. 이 문서는 애플리케이션의 컴퓨팅 및 메모리 요구 사항에 따라 적절한 VM 크기의 범위를 지정하는 데 도움이 됩니다.  

**애플리케이션(애플리케이션 수준)**

애플리케이션 계층에서는 Kubernetes [HPA(수평 Pod 자동 크기 조정기)](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)를 사용합니다. HPA는 CPU 사용률과 같은 다양한 메트릭에 따라 배포에서 복제본(Pod/Container Instances) 수를 늘리거나 줄일 수 있습니다.

또 다른 옵션은 컨테이너 인스턴스의 크기를 수직으로 조정하는 것입니다. 이는 특정 배포에 대해 요청되고 사용 가능한 CPU 및 메모리의 양을 변경하여 수행할 수 있습니다. 자세한 내용은 kubernetes.io의 [컨테이너용 리소스 관리](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)를 참조하세요.

## <a name="networking-and-connectivity-considerations"></a>네트워킹 및 연결 고려 사항

네트워킹 및 연결은 Azure Stack Hub의 Kubernetes에 대해 앞에서 언급한 세 가지 계층에도 영향을 줍니다. 다음 표에서는 이러한 계층 및 해당 계층에 포함된 서비스를 보여 줍니다.

| 계층 | 영향 | 무엇을? |
| --- | --- | ---
| 애플리케이션 | 애플리케이션 | 애플리케이션에 어떻게 액세스할 수 있나요? 인터넷에 공개되나요? |
| 클러스터 | Kubernetes 클러스터 | Kubernetes API, AKS 엔진 VM, 컨테이너 이미지 끌어오기(송신), 모니터링 데이터 및 원격 분석 보내기(송신) |
| 인프라 | Azure Stack Hub | Azure Stack Hub 관리 엔드포인트(예: 포털 및 Azure Resource Manager 엔드포인트)의 접근성 |

**애플리케이션**

애플리케이션 계층의 경우 가장 중요한 고려 사항은 애플리케이션이 공개되고 인터넷에서 액세스할 수 있는지 여부입니다. Kubernets 관점에서 인터넷 접근성은 Kubernets Service 또는 수신 컨트롤러를 사용하여 배포 또는 Pod를 공개하는 것을 의미합니다.

> [!NOTE]
> Azure Stack Hub의 프런트 엔드 공용 IP 수가 5개로 제한되므로 수신 컨트롤러를 사용하여 Kubernetes Service를 공개하는 것이 좋습니다. 또한 이 디자인은 Kubernetes Service(LoadBalancer 유형)의 수를 5개로 제한하며, 이는 많은 배포에 비해 너무 적습니다. 자세한 내용은 [AKS 엔진 설명서](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#limited-number-of-frontend-public-ips)를 참조하세요.

Load Balancer 또는 수신 컨트롤러를 통해 공용 IP를 사용하여 애플리케이션을 공개한다고 해서 반드시 인터넷을 통해 애플리케이션에 액세스할 수 있는 것은 아닙니다. Azure Stack Hub에는 로컬 인트라넷에만 표시되는 공용 IP 주소가 있을 수 있으며, 모든 공용 IP가 실제로 인터넷에 연결되는 것은 아닙니다.

이전 블록에서는 애플리케이션에 대한 수신 트래픽을 고려했지만, 성공적인 Kubernetes 배포를 위해 고려해야 하는 또 다른 항목으로 아웃바운드/송신 트래픽이 있습니다. 송신 트래픽이 필요한 몇 가지 사용 사례는 다음과 같습니다.

- DockerHub 또는 Azure Container Registry에 저장된 컨테이너 이미지 끌어오기
- Helm 차트 검색
- Application Insights 데이터(또는 기타 모니터링 데이터) 내보내기

일부 엔터프라이즈 환경에서는 _투명_ 또는 _불투명_ 프록시 서버를 사용해야 할 수 있습니다. 이러한 서버에는 클러스터의 다양한 구성 요소에 대한 특정 구성이 필요합니다. AKS 엔진 설명서에는 네트워크 프록시를 수용하는 방법에 대한 다양한 세부 정보가 포함되어 있습니다. 자세한 내용은 [AKS 엔진 및 프록시 서버](https://github.com/Azure/aks-engine/blob/master/docs/topics/proxy-servers.md)를 참조하세요.

마지막으로, 클러스터 간 트래픽이 Azure Stack Hub 인스턴스 간에 이동해야 합니다. 배포 샘플은 개별 Azure Stack Hub 인스턴스에서 실행되는 개별 Kubernetes 클러스터로 구성됩니다. 두 데이터베이스 간의 복제 트래픽과 같은 두 클러스터 간의 트래픽은 "외부 트래픽"입니다. 두 개의 Azure Stack Hub 인스턴스에서 Kubernetes를 연결하려면 외부 트래픽이 사이트 간 VPN 또는 Azure Stack Hub 공용 IP 주소를 통해 라우팅되어야 합니다.

![클러스터 내부 트래픽 및 클러스터 간 트래픽](media/pattern-highly-available-kubernetes/aks-inter-and-intra-cluster-traffic.png)

**Cluster**  

Kubernetes 클러스터는 반드시 인터넷을 통해 액세스할 필요가 없습니다. 관련 부분은 클러스터를 작동하는 데 사용되는 Kubernetes API입니다(예: `kubectl` 사용). Kubernetes API 엔드포인트는 클러스터를 작동하거나 이를 기반으로 하여 애플리케이션 및 서비스를 배포하는 모든 사용자가 액세스할 수 있어야 합니다. 이 항목은 아래의 [배포(CI/CD) 고려 사항](#deployment-cicd-considerations) 섹션의 DevOps 관점에서 자세히 설명합니다.

클러스터 수준에는 송신 트래픽과 관련된 몇 가지 고려 사항도 있습니다.

- 노드 업데이트(Ubuntu용)
- 모니터링 데이터(Azure LogAnalytics로 전송)
- 아웃바운드 트래픽이 필요한 기타 에이전트(각 배포자의 환경에 따라 다름)

AKS 엔진을 사용하여 Kubernetes 클러스터를 배포하기 전에 최종 네트워킹 디자인을 계획합니다. 전용 Virtual Network를 만드는 대신 클러스터를 기존 네트워크에 배포하는 것이 더 효율적일 수 있습니다. 예를 들어 Azure Stack Hub 환경에 이미 구성된 기존 사이트 간 VPN 연결을 활용할 수 있습니다.

**인프라**  

인프라는 Azure Stack Hub 관리 엔드포인트에 액세스하는 것을 말합니다. 엔드포인트에는 테넌트 및 관리 포털과 Azure Resource Manager 관리 및 테넌트 엔드포인트가 포함됩니다. 이러한 엔드포인트는 Azure Stack Hub 및 핵심 서비스를 작동하는 데 필요합니다.

## <a name="data-and-storage-considerations"></a>데이터 및 스토리지 고려 사항

두 개의 Azure Stack Hub 인스턴스에서 두 개의 애플리케이션 인스턴스가 두 개의 개별 Kubernetes 클러스터에 배포됩니다. 이 디자인에서는 데이터를 복제하고 동기화하는 방법을 고려해야 합니다.

Azure를 사용하면 클라우드 내의 여러 지역 및 영역에서 스토리지를 복제하는 기본 제공 기능을 사용할 수 있습니다. 현재 Azure Stack Hub에는 두 개의 서로 다른 Azure Stack Hub 인스턴스에서 스토리지를 복제하는 기본적인 방법이 없습니다. 두 개의 독립된 클라우드를 만들어 하나의 세트로 관리할 수 있는 중요한 방법이 없습니다. Azure Stack Hub에서 실행되는 애플리케이션의 복원력을 계획하는 경우 애플리케이션 디자인 및 배포에서 이러한 독립성을 고려해야 합니다.

대부분의 경우 AKS에 배포된 복원력이 있고 가용성이 높은 애플리케이션에는 스토리지 복제가 필요하지 않습니다. 그러나 애플리케이션 디자인에서는 Azure Stack Hub 인스턴스당 독립적인 스토리지를 고려해야 합니다. 이 디자인이 솔루션을 Azure Stack Hub에 배포하는 데 문제 또는 장애물이 되는 경우 스토리지 첨부를 제공하는 Microsoft 파트너의 솔루션이 있습니다. 스토리지 첨부는 여러 Azure Stack Hub 및 Azure에서 스토리지 복제 솔루션을 제공합니다. 자세한 내용은 [파트너 솔루션](#partner-solutions)을 참조하세요.

아키텍처에서 고려된 계층은 다음과 같습니다.

**Configuration**

구성에는 Azure Stack Hub, AKS 엔진 및 Kubernetes 클러스터 자체의 구성이 포함됩니다. 구성은 최대한 자동화되어야 하며, Git 기반 버전 제어 시스템(예: Azure DevOps 또는 GitHub)에서 Infrastructure-as-Code로 저장되어야 합니다. 이러한 설정은 여러 배포에서 쉽게 동기화할 수 없습니다. 따라서 외부에서 구성을 저장 및 적용하고 DevOps 파이프라인을 사용하는 것이 좋습니다.

**애플리케이션**

애플리케이션은 Git 기반 리포지토리에 저장해야 합니다. 새 배포, 애플리케이션 변경 또는 재해 복구가 있을 때마다 Azure Pipelines를 사용하여 쉽게 배포할 수 있습니다.

**데이터**

데이터는 대부분의 애플리케이션 디자인에서 가장 중요한 고려 사항입니다. 애플리케이션 데이터는 애플리케이션의 서로 다른 인스턴스 간에 동기화 상태를 유지해야 합니다. 또한 중단이 발생하는 경우에 대비하여 데이터에 대한 백업 및 재해 복구 전략이 필요합니다.

이 디자인을 달성하는 것은 기술 선택에 따라 크게 달라집니다. Azure Stack Hub에서 고가용성 방식으로 데이터베이스를 구현하기 위한 몇 가지 솔루션 예는 다음과 같습니다.

- [Azure 및 Azure Stack Hub에 SQL Server 2016 가용성 그룹 배포](/azure-stack/hybrid/solution-deployment-guide-sql-ha)
- [Azure 및 Azure Stack Hub에 고가용성 MongoDB 솔루션 배포](/azure-stack/hybrid/solution-deployment-guide-mongodb-ha)

여러 위치에서 데이터를 사용하는 경우 고려해야 하는 사항은 가용성이 높고 복원력이 있는 솔루션에 대해 훨씬 더 복잡합니다. 고려할 사항은 다음과 같습니다.

- Azure Stack Hub 간의 대기 시간 및 네트워크 연결
- 서비스 및 권한에 대한 ID 가용성. 각 Azure Stack Hub 인스턴스는 외부 디렉터리와 통합됩니다. 배포 중에 Azure AD(Azure Active Directory) 또는 ADFS(Active Directory Federation Services)를 사용하도록 선택합니다. 따라서 독립적인 여러 Azure Stack Hub 인스턴스와 상호 작용할 수 있는 단일 ID를 사용할 수 있습니다.

## <a name="business-continuity-and-disaster-recovery"></a>비즈니스 연속성 및 재해 복구

BCDR(비즈니스 연속성 및 재해 복구)은 Azure Stack Hub와 Azure 모두에서 중요한 항목입니다. 주요 차이점은 Azure Stack Hub에서는 운영자가 전체 BCDR 프로세스를 관리해야 한다는 것입니다. Azure에서 BCDR의 일부는 Microsoft에서 자동으로 관리합니다.

BCDR은 이전의 [데이터 및 스토리지 고려 사항](#data-and-storage-considerations) 섹션에서 언급한 동일한 영역에 영향을 줍니다.

- 인프라/구성
- 애플리케이션 가용성
- 애플리케이션 데이터

이전 섹션에서 언급한 대로 이러한 영역은 Azure Stack Hub 운영자가 담당하며 조직마다 다를 수 있습니다. 사용 가능한 도구 및 프로세스에 따라 BCDR을 계획합니다.

**인프라 및 구성**

이 섹션에서는 물리적 및 논리적 인프라와 Azure Stack Hub의 구성에 대해 설명합니다. 관리자 및 테넌트 공간의 작업에 대해 설명합니다.

Azure Stack Hub 운영자(또는 관리자)는 Azure Stack Hub 인스턴스를 유지 관리해야 합니다. 이 문서의 범위를 벗어나는 네트워크, 스토리지, ID 및 기타 항목과 같은 구성 요소가 포함됩니다. Azure Stack Hub 작업에 대해 자세히 알아보려면 다음 리소스를 참조하세요.

- [Azure Stack Hub에서 Infrastructure Backup Service를 사용하여 데이터 복구](/azure-stack/operator/azure-stack-backup-infrastructure-backup)
- [관리자 포털에서 백업을 Azure Stack Hub에 사용하도록 설정](/azure-stack/operator/azure-stack-backup-enable-backup-console)
- [치명적인 데이터 손실로부터 복구](/azure-stack/operator/azure-stack-backup-recover-data)
- [Infrastructure Backup Service 모범 사례](/azure-stack/operator/azure-stack-backup-best-practices)

Azure Stack Hub는 Kubernetes 애플리케이션을 배포할 플랫폼 및 패브릭입니다. Kubernetes 애플리케이션의 애플리케이션 소유자는 솔루션에 필요한 애플리케이션 인프라를 배포할 수 있는 액세스 권한이 부여된 Azure Stack Hub의 사용자입니다. 이 경우 애플리케이션 인프라는 AKS 엔진을 사용하여 배포된 Kubernetes 클러스터 및 주변 서비스를 의미합니다. 이러한 구성 요소는 Azure Stack Hub 제안으로 제한된 Azure Stack Hub에 배포됩니다. 전체 솔루션을 Kubernetes 애플리케이션 소유자가 수락한 제안에 배포하는 데 충분한 용량(Azure Stack Hub 할당량으로 표시됨)이 있는지 확인합니다. 이전 섹션에서 추천한 대로 Infrastructure-as-Code 및 배포 파이프라인(예: Azure DevOps Azure Pipelines)을 사용하여 애플리케이션 배포를 자동화해야 합니다.

Azure Stack Hub 제안 및 할당량에 대한 자세한 내용은 [Azure Stack Hub 서비스, 계획, 제안 및 구독 개요](/azure-stack/operator/service-plan-offer-subscription-overview)를 참조하세요.

출력을 포함하여 AKS 엔진 구성을 안전하게 저장하고 보관해야 합니다. 이러한 파일에는 Kubernetes 클러스터에 액세스하는 데 사용되는 기밀 정보가 포함되므로 관리자가 아닌 사용자에게 공개되지 않도록 보호해야 합니다.

**애플리케이션 가용성**

애플리케이션은 배포된 인스턴스의 백업을 사용하지 않아야 합니다. 표준 사례로 Infrastructure-as-Code 패턴을 따라 애플리케이션을 완전히 다시 배포합니다. 예를 들어 Azure DevOps Azure Pipelines를 사용하여 다시 배포합니다. BCDR 절차에서는 애플리케이션을 동일하거나 다른 Kubernetes 클러스터에 다시 배포해야 합니다.

**애플리케이션 데이터**

애플리케이션 데이터는 데이터 손실을 최소화하는 데 중요한 부분입니다. 이전 섹션에서는 둘 이상의 애플리케이션 인스턴스 간에 데이터를 복제하고 동기화하는 기술에 대해 설명했습니다. 데이터를 저장하는 데 사용되는 데이터베이스 인프라(MySQL, MongoDB, MSSQL 또는 기타)에 따라 선택할 수 있는 다양한 데이터베이스 가용성 및 백업 기술이 있습니다.

무결성을 달성하는 데 추천되는 방법은 다음 중 하나를 사용하는 것입니다.
- 특정 데이터베이스에 대한 기본 백업 솔루션.
- 애플리케이션에서 사용하는 데이터베이스 유형의 백업 및 복구를 공식적으로 지원하는 백업 솔루션.

> [!IMPORTANT]
> 애플리케이션 데이터가 있는 동일한 Azure Stack Hub 인스턴스에 백업 데이터를 저장하지 마세요. Azure Stack Hub 인스턴스가 완전히 중단되면 백업도 손상됩니다.

## <a name="availability-considerations"></a>가용성 고려 사항

AKS 엔진을 통해 배포된 Azure Stack Hub의 Kubernetes는 관리되는 서비스가 아닙니다. 이는 Azure IaaS(Infrastructure-as-a-Service)를 사용하여 자동화된 Kubernetes 클러스터 배포 및 구성입니다. 따라서 기본 인프라와 동일한 가용성을 제공합니다.

Azure Stack Hub 인프라는 이미 오류에 대한 복원력을 갖추고 있으며, Availability Sets와 같은 기능을 제공하여 구성 요소를 여러 [장애 및 업데이트 도메인](/azure-stack/user/azure-stack-vm-considerations#high-availability)에 배포합니다. 그러나 하드웨어 오류가 있는 경우에도 기본 기술(장애 조치(failover) 클러스터링)로 인해 영향을 받는 물리적 서버의 VM에 대해 여전히 약간의 가동 중지 시간이 발생합니다.

프로덕션 Kubernetes 클러스터와 워크로드를 둘 이상의 클러스터에 배포하는 것이 좋습니다. 이러한 클러스터는 서로 다른 위치 또는 데이터 센터에서 호스팅되어야 하며, Azure Traffic Manager와 같은 기술을 사용하여 클러스터 응답 시간 또는 지리에 따라 사용자를 라우팅해야 합니다.

![Traffic Manager를 사용하여 트래픽 흐름 제어](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager.png)

단일 Kubernetes 클러스터가 있는 고객은 일반적으로 지정된 애플리케이션의 서비스 IP 또는 DNS 이름에 연결합니다. 다중 클러스터 배포에서는 고객이 각 Kubernetes 클러스터의 서비스/수신을 가리키는 Traffic Manager DNS 이름에 연결해야 합니다.

![Traffic Manager를 사용하여 내부 클러스터로 라우팅](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

> [!NOTE]
> 이 패턴은 [Azure에서 (관리되는) AKS 클러스터에 대한 모범 사례](/azure/aks/operator-best-practices-multi-region#plan-for-multiregion-deployment)이기도 합니다.

AKS 엔진을 통해 배포되는 Kubernetes 클러스터 자체는 세 개 이상의 마스터 노드와 두 개의 작업자 노드로 구성되어야 합니다.

## <a name="identity-and-security-considerations"></a>ID 및 보안 고려 사항

ID 및 보안은 중요한 항목입니다. 특히 솔루션이 독립적인 Azure Stack Hub 인스턴스에 걸쳐 있는 경우 이에 해당합니다. Kubernetes 및 Azure(Azure Stack Hub 포함)에는 모두 RBAC(역할 기반 액세스 제어)에 대한 고유한 메커니즘이 있습니다.

- Azure RBAC는 새 Azure 리소스를 만드는 기능을 포함하여 Azure(및 Azure Stack Hub)의 리소스에 대한 액세스를 제어합니다. 사용자, 그룹 또는 서비스 주체에 권한을 할당할 수 있습니다. (서비스 주체는 애플리케이션에서 사용하는 보안 ID입니다.)
- Kubernetes RBAC는 Kubernetes API 권한을 제어합니다. 예를 들어 Pod 만들기 및 Pod 나열은 RBAC를 통해 사용자에게 권한을 부여(또는 거부)할 수 있는 작업입니다. 사용자에게 Kubernetes 권한을 할당하려면 역할 및 역할 바인딩을 만듭니다.

**Azure Stack Hub ID 및 RBAC**

Azure Stack Hub는 ID 공급자에 대해 두 가지 선택 항목을 제공합니다. 사용하는 공급자는 환경 및 연결되거나 연결되지 않은 환경에서 실행되는지 여부에 따라 달라집니다.

- Azure AD - 연결된 환경에서만 사용할 수 있습니다.
- 기존 Active Directory 포리스트에 대한 ADFS - 연결되거나 연결되지 않은 환경 모두에서 사용할 수 있습니다.

ID 공급자는 리소스 액세스를 위한 인증 및 권한 부여를 포함하여 사용자와 그룹을 관리합니다. 구독, 리소스 그룹 및 개별 리소스(예: VM 또는 부하 분산 장치)와 같은 Azure Stack Hub 리소스에 대한 액세스 권한을 부여할 수 있습니다. 일관된 액세스 모델을 사용하려면 모든 Azure Stack Hub에 대해 동일한 그룹(직접 또는 중첩)을 사용하는 것을 고려해야 합니다. 구성 예는 다음과 같습니다.

![Azure Stack Hub를 사용하여 중첩된 AAD 그룹](media/pattern-highly-available-kubernetes/azure-stack-azure-ad-nested-groups.png)

이 예에는 특정 용도를 위한 전용 그룹(AAD 또는 ADFS 사용)이 포함되어 있습니다. 예를 들어 특정 Azure Stack Hub 인스턴스에서 Kubernetes 클러스터 인프라를 포함하는 리소스 그룹에 대한 기여자(여기서는 "Seattle K8s 클러스터 기여자") 권한을 제공합니다. 그런 다음, 이러한 그룹은 각 Azure Stack Hub에 대한 "하위 그룹"을 포함하는 전체 그룹에 중첩됩니다.

이제 샘플 사용자에게는 전체 Kubernetes 인프라 리소스 세트가 포함된 두 리소스 그룹에 대한 "기여자" 권한이 있습니다. 인스턴스에서 동일한 ID 공급자를 공유하므로 사용자는 두 Azure Stack Hub 인스턴스 모두의 리소스에 액세스할 수 있습니다.

> [!IMPORTANT]
> 이러한 권한은 Azure Stack Hub 및 이를 기반으로 하여 배포된 리소스 중 일부에만 영향을 줍니다. 이 액세스 수준에 있는 사용자는 많은 피해를 줄 수 있지만, Kubernetes 배포에 대한 추가 액세스 권한이 있어야 Kubernetes IaaS VM 및 Kubernetes API에 액세스할 수 있습니다.

**Kubernetes ID 및 RBAC**

Kubernetes 클러스터는 기본적으로 기본 Azure Stack Hub와 동일한 ID 공급자를 사용하지 않습니다. Kubernetes 클러스터, 마스터 및 작업자 노드를 호스팅하는 VM은 클러스터 배포 중에 지정된 SSH 키를 사용합니다. 이 SSH 키는 SSH를 사용하여 이러한 노드에 연결하는 데 필요합니다.

또한 Kubernetes API(예: `kubectl`을 사용하여 액세스)는 서비스 계정(기본 "클러스터 관리자" 서비스 계정 포함)으로도 보호됩니다. 이 서비스 계정에 대한 자격 증명은 처음에 Kubernetes 마스터 노드의 `.kube/config` 파일에 저장됩니다.

**비밀 관리 및 애플리케이션 자격 증명**

연결 문자열 또는 데이터베이스 자격 증명과 같은 비밀을 저장하려면 다음과 같은 몇 가지 선택 사항이 있습니다.

- Azure Key Vault
- Kubernetes 비밀
- HashiCorp Vault와 같은 타사 솔루션(Kubernetes에서 실행)

비밀 또는 자격 증명을 일반 텍스트로 구성 파일, 애플리케이션 코드 또는 스크립트 내에 저장하지 마세요. 버전 제어 시스템에도 저장하지 마세요. 대신 배포 자동화는 필요에 따라 비밀을 검색해야 합니다.

## <a name="patch-and-update"></a>패치 및 업데이트

Azure Kubernetes Service의 **PNU(패치 및 업데이트)** 프로세스는 부분적으로 자동화됩니다. Kubernetes 버전 업그레이드는 수동으로 트리거되지만, 보안 업데이트는 자동으로 적용됩니다. 이러한 업데이트에는 OS 보안 수정 또는 커널 업데이트가 포함될 수 있습니다. AKS는 업데이트 프로세스를 완료하기 위해 이러한 Linux 노드를 자동으로 다시 부팅하지 않습니다. 

Azure Stack Hub에서 AKS 엔진을 사용하여 배포된 Kubernetes 클러스터에 대한 PNU 프로세스는 관리되지 않으며 클러스터 운영자가 담당합니다. 

AKS 엔진에서 지원하는 가장 중요한 두 가지 작업은 다음과 같습니다.

- [최신 Kubernetes 및 기본 OS 이미지 버전으로 업그레이드](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version)
- [기본 OS 이미지만 업그레이드](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image)

최신 기본 OS 이미지에는 최신 OS 보안 수정 및 커널 업데이트가 포함되어 있습니다. 

[무인 업그레이드](https://wiki.debian.org/UnattendedUpgrades) 메커니즘은 Azure Stack Hub Marketplace에서 새 기본 OS 이미지 버전을 사용할 수 있기 전에 릴리스된 보안 업데이트를 자동으로 설치합니다. 무인 업그레이드는 기본적으로 사용하도록 설정되며, 보안 업데이트를 자동으로 설치하지만 Kubernetes 클러스터 노드를 다시 부팅하지 않습니다. 노드를 다시 부팅하는 것은 오픈 소스 [kured(**K** Ubernetes **RE** boot **D** aemon)](/azure/aks/node-updates-kured)를 사용하여 자동화할 수 있습니다. kured는 다시 부팅해야 하는 Linux 노드를 감시한 다음, 실행되는 Pod 및 노드 다시 부팅 프로세스의 다시 예약을 자동으로 처리합니다.

## <a name="deployment-cicd-considerations"></a>배포(CI/CD) 고려 사항

Azure 및 Azure Stack Hub는 동일한 Azure Resource Manager REST API를 표시합니다. 이러한 API는 다른 Azure 클라우드(Azure, Azure 중국 21Vianet, Azure Government)와 동일하게 처리됩니다. API 버전은 클라우드 간에 다를 수 있으며, Azure Stack Hub는 서비스의 하위 집합만 제공합니다. 관리 엔드포인트 URI는 클라우드 및 Azure Stack Hub 인스턴스마다 다릅니다.

언급한 미묘한 차이점 외에도 Azure Resource Manager REST API는 Azure 및 Azure Stack Hub 모두와 상호 작용할 수 있는 일관된 방법을 제공합니다. 여기서는 다른 Azure 클라우드에서 사용하는 것과 동일한 도구 세트를 사용할 수 있습니다. Azure DevOps, Jenkins와 같은 도구 또는 PowerShell을 사용하여 서비스를 Azure Stack Hub에 배포하고 오케스트레이션할 수 있습니다.

**고려 사항**

Azure Stack Hub 배포와 관련된 주요 차이점 중 하나는 인터넷 접근성에 대한 질문입니다. 인터넷 접근성은 CI/CD 작업에 대해 Microsoft 호스팅 또는 자체 호스팅 빌드 에이전트를 선택할지 여부를 결정합니다.

자체 호스팅 에이전트는 Azure Stack Hub를 기반으로 하여 실행되거나(IaaS VM으로) Azure Stack Hub에 액세스할 수 있는 네트워크 서브넷에서 실행될 수 있습니다. 차이점에 대한 자세한 내용은 [Azure Pipelines 에이전트](/azure/devops/pipelines/agents/agents)를 참조하세요.

다음 이미지는 자체 호스팅 또는 Microsoft 호스팅 빌드 에이전트가 필요한지 결정하는 데 도움이 됩니다.

![자체 호스팅 빌드 에이전트 예 또는 아니요](media/pattern-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)

- 인터넷을 통해 Azure Stack Hub 관리 엔드포인트에 액세스할 수 있나요?
  - 예: Microsoft 호스팅 에이전트에서 Azure Pipelines를 사용하여 Azure Stack Hub에 연결할 수 있습니다.
  - 아니요: Azure Stack Hub의 관리 엔드포인트에 연결할 수 있는 자체 호스팅 에이전트가 필요합니다.
- 인터넷을 통해 Kubernetes 클러스터에 액세스할 수 있나요?
  - 예: Microsoft 호스팅 에이전트에서 Azure Pipelines를 사용하여 Kubernetes API 엔드포인트와 직접 상호 작용할 수 있습니다.
  - 아니요: Kubernetes 클러스터 API 엔드포인트에 연결할 수 있는 자체 호스팅 에이전트가 필요합니다.

인터넷을 통해 Azure Stack Hub 관리 엔드포인트 및 Kubernetes API에 액세스할 수 있는 시나리오에서 배포는 Microsoft 호스팅 에이전트를 사용할 수 있습니다. 이 배포로 인해 다음과 같은 애플리케이션 아키텍처가 생성됩니다.

[![퍼블릭 아키텍처 개요](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png#lightbox)

Azure Resource Manager 엔드포인트, Kubernetes API 또는 둘 다 인터넷을 통해 직접 액세스할 수 없는 경우 자체 호스팅 빌드 에이전트를 활용하여 파이프라인 단계를 실행할 수 있습니다. 이 디자인에서는 더 적은 연결이 필요하며, Azure Resource Manager 엔드포인트 및 Kubernetes API에 대한 온-프레미스 네트워크 연결만으로 배포할 수 있습니다.

[![온-프레미스 아키텍처 개요](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png#lightbox)

> [!NOTE]
> **연결되지 않은 시나리오는 어떻게 되나요?** Azure Stack Hub 또는 Kubernetes 중 하나 또는 둘 다에 인터넷 연결 관리 엔드포인트가 없는 시나리오에서도 Azure DevOps를 배포에 사용할 수 있습니다. 온-프레미스에서 자체 호스팅 에이전트 풀(온-프레미스 또는 Azure Stack Hub 자체에서 실행되는 DevOps 에이전트) 또는 완전한 자체 호스팅 Azure DevOps 서버를 사용할 수 있습니다. 자체 호스팅 에이전트에는 아웃바운드 HTTPS(TCP/443) 인터넷 연결만 필요합니다.

이 패턴에서는 각 Azure Stack Hub 인스턴스에서 Kubernetes 클러스터(AKS 엔진을 통해 배포 및 오케스트레이션됨)를 사용할 수 있습니다. 여기에는 프런트 엔드, 중간 계층, 백 엔드 서비스(예: MongoDB) 및 nginx 기반 수신 컨트롤러로 구성된 애플리케이션이 포함됩니다. K8s 클러스터에서 호스팅되는 데이터베이스를 사용하는 대신 "외부 데이터 저장소"를 활용할 수 있습니다. 데이터베이스 옵션에는 MySQL, SQL Server 또는 Azure Stack Hub 외부 또는 IaaS에서 호스팅되는 모든 종류의 데이터베이스가 포함됩니다. 이와 같은 구성은 여기서 다루지 않습니다.

## <a name="partner-solutions"></a>파트너 솔루션

Azure Stack Hub의 기능을 확장할 수 있는 Microsoft 파트너 솔루션이 있습니다. 이러한 솔루션은 Kubernetes 클러스터에서 실행되는 애플리케이션을 배포하는 데 유용합니다.  

## <a name="storage-and-data-solutions"></a>스토리지 및 데이터 솔루션

[데이터 및 스토리지 고려 사항](#data-and-storage-considerations)에서 설명한 대로 Azure Stack Hub에는 현재 여러 인스턴스에서 스토리지를 복제하는 기본 솔루션이 없습니다. Azure와 달리 여러 지역에서 스토리지를 복제하는 기능이 없습니다. Azure Stack Hub에서 각 인스턴스는 고유한 클라우드입니다. 그러나 Azure Stack Hub 및 Azure에서 스토리지 복제를 사용하도록 설정하는 솔루션은 Microsoft 파트너에서 사용할 수 있습니다. 

**SCALITY**

[Scality](https://www.scality.com/)는 2009년부터 디지털 비즈니스를 제공한 웹 규모의 스토리지를 제공합니다. 소프트웨어 정의 스토리지인 Scality RING은 상용 x86 서버를 페타바이트 규모의 모든 형식의 데이터(파일 및 개체)에 대한 무제한 스토리지 풀로 전환합니다.

**CLOUDIAN**

[Cloudian](https://www.cloudian.com/)은 엔터프라이즈 스토리지를 확장 가능한 무제한 스토리지로 간소화하여 대규모 데이터 세트를 관리하기 쉬운 단일 환경으로 통합합니다.

## <a name="next-steps"></a>다음 단계

이 문서에서 소개하는 개념에 대해 자세히 알아보려면 다음을 참조하세요.

- Azure Stack Hub의 [클라우드 간 크기 조정](pattern-cross-cloud-scale.md) 및 [지리적으로 분산되는 앱 패턴](pattern-geo-distributed.md)
- [AKS(Azure Kubernetes Service)의 마이크로서비스 아키텍처](/azure/architecture/reference-architectures/microservices/aks)

솔루션 예를 테스트할 준비가 되면 [고가용성 Kubernetes 클러스터 배포 가이드](/azure/architecture/hybrid/deployments/solution-deployment-guide-highly-available-kubernetes)로 계속 진행하세요. 배포 가이드에서는 해당 구성 요소를 배포 및 테스트하는 방법에 대한 단계별 지침을 제공합니다.