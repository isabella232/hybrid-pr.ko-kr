---
title: Azure Stack Hub에서 고가용성 Kubernetes 클러스터 배포
description: Azure 및 Azure Stack Hub를 사용하여 고가용성을 위해 Kubernetes 클러스터 솔루션을 배포하는 방법을 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: 91f5856aa670bf3810baa5e5f07dbb7dafc9e3f3
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96911924"
---
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a>Azure Stack Hub에서 고가용성 Kubernetes 클러스터 배포

이 문서에서는 서로 다른 물리적 위치에서 여러 Azure Stack Hub 인스턴스에 배포된 고가용성 Kubernetes 클러스터 환경을 빌드하는 방법을 보여줍니다.

이 솔루션 배포 가이드에서는 다음 방법에 대해 알아봅니다.

> [!div class="checklist"]
> - AKS 엔진 다운로드 및 준비
> - AKS 엔진 도우미 VM에 연결
> - Kubernetes 클러스터 배포
> - Kubernetes 클러스터에 연결
> - Kubernetes 클러스터에 Azure Pipelines 연결
> - 모니터링 구성
> - 애플리케이션 배포
> - 자동 크기 조정 애플리케이션
> - Traffic Manager 구성
> - Kubernetes 업그레이드
> - Kubernetes 크기 조정

> [!Tip]  
> ![하이브리드 핵심 요소](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub는 Azure의 확장입니다. Azure Stack Hub는 온-프레미스 환경에 클라우드 컴퓨팅의 민첩성과 혁신을 제공하여 어디서나 하이브리드 앱을 빌드하고 배포할 수 있는 유일한 하이브리드 클라우드를 사용하도록 설정합니다.  
> 
> [하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 문서는 하이브리드 앱 디자인, 배포 및 운영에 대한 소프트웨어 품질(배치, 확장성, 가용성, 복원력, 관리 효율성 및 보안)의 핵심 요소를 검토합니다. 디자인 고려 사항은 하이브리드 앱 디자인을 최적화하고 프로덕션 환경에서 문제를 최소화하는 데 도움이 됩니다.

## <a name="prerequisites"></a>필수 구성 요소

이 배포 가이드를 시작하기 전에 다음을 확인하십시오.

- [고가용성 Kubernetes 클러스터 패턴](pattern-highly-available-kubernetes.md) 문서를 검토합니다.
- 이 문서에서 참조되는 추가 자산을 포함하는 [부록 GitHub 리포지토리](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub)의 내용을 검토합니다.
- 최소 ["기여자" 권한](/azure-stack/user/azure-stack-manage-permissions)으로 [Azure Stack Hub 사용자 포털](/azure-stack/user/azure-stack-use-portal)에 액세스할 수 있는 계정이 있어야 합니다.

## <a name="download-and-prepare-aks-engine"></a>AKS 엔진 다운로드 및 준비

AKS 엔진은 Azure Stack Hub Azure Resource Manager 엔드포인트에 도달할 수 있는 모든 Windows 또는 Linux 호스트에서 사용할 수 있는 바이너리입니다. 이 가이드에서는 Azure Stack Hub에 새 Linux(또는 Windows) VM을 배포하는 방법을 설명합니다. AKS 엔진이 Kubernetes 클러스터를 배포하는 경우 나중에 사용됩니다.

> [!NOTE]
> 기존 Windows 또는 Linux VM을 사용하여 AKS 엔진을 사용하는 Azure Stack Hub에 Kubernetes 클러스터를 배포할 수도 있습니다.

AKS 엔진에 대한 단계별 프로세스 및 요구 사항은 여기에 설명되어 있습니다.

* [Azure Stack Hub에서 Linux에 AKS 엔진 설치](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux)(또는 [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows) 사용)

AKS 엔진은 Azure 및 Azure Stack Hub에서 관리되지 않는 Kubernetes 클러스터를 배포하고 운영하기 위한 도우미 도구입니다.

Azure Stack Hub에 있는 AKS 엔진의 세부 정보 및 차이점은 여기에 설명되어 있습니다.

* [Azure Stack Hub에서 AKS 엔진이란?](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* [Azure Stack Hub의 AKS 엔진](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md)(GitHub)

샘플 환경은 Terraform을 사용하여 AKS 엔진 VM의 배포를 자동화합니다. [도우미 GitHub 리포지토리에서 세부 정보와 코드](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md)를 찾을 수 있습니다.

이 단계의 결과는 AKS 엔진 도우미 VM 및 관련 리소스가 포함된 Azure Stack Hub의 새 리소스 그룹입니다.

![Azure Stack Hub에서 AKS 엔진 VM 리소스](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> 연결이 끊긴 에어-갭 환경에 AKS 엔진을 배포해야 하는 경우 [연결이 끊긴 Azure Stack Hub 인스턴스](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances)를 검토하여 자세히 알아보세요.

다음 단계에서는 새로 배포된 AKS 엔진 VM을 사용하여 Kubernetes 클러스터를 배포합니다.

## <a name="connect-to-the-aks-engine-helper-vm"></a>AKS 엔진 도우미 VM에 연결

먼저 이전에 만든 AKS 엔진 도우미 VM에 연결해야 합니다.

VM에는 공용 IP 주소가 있어야 하며 SSH(포트 22/TCP)를 통해 액세스할 수 있어야 합니다.

![AKS 엔진 VM 개요 페이지](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> Windows 10에서 MobaXterm, puTTY 또는 PowerShell과 같은 도구를 사용하여 SSH를 사용하여 Linux VM에 연결할 수 있습니다.

```console
ssh <username>@<ipaddress>
```

연결 후 `aks-engine` 명령을 실행합니다. AKS 엔진 및 Kubernetes 버전에 대한 자세한 내용은 [지원되는 AKS 엔진 버전](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions)으로 이동하세요.

![aks-engine 명령줄 예](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a>Kubernetes 클러스터 배포

AKS 엔진 도우미 VM 자체가 Azure Stack Hub에 Kubernetes 클러스터를 아직 만들지 않았습니다. AKS 엔진 도우미 VM에서 수행할 첫 번째 작업은 클러스터를 만드는 것입니다.

단계별 프로세스는 여기에 설명되어 있습니다.

* [Azure Stack Hub에 AKS 엔진을 사용하여 Kubernetes 클러스터 배포](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

`aks-engine deploy` 명령의 최종 결과와 이전 단계의 준비는 첫 번째 Azure Stack Hub 인스턴스의 테넌트 공간에 배포된 모든 기능을 갖춘 Kubernetes 클러스터입니다. 클러스터 자체는 VM, 부하 분산 장치, VNet, 디스크 등과 같은 Azure IaaS 구성 요소로 구성됩니다.

![클러스터 IaaS 구성 요소 Azure Stack Hub 포털](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) Azure Load Balancer(K8s API 엔드포인트)
2) 작업자 노드(에이전트 풀)
3) 마스터 노드

이제 클러스터가 실행되고 있으며 다음 단계에서 클러스터에 연결합니다.

## <a name="connect-to-the-kubernetes-cluster"></a>Kubernetes 클러스터에 연결

이제 SSH(배포의 일부로 지정된 SSH 키 사용) 또는 `kubectl`(권장)을 통해 이전에 만들어진 Kubernetes 클러스터에 연결할 수 있습니다. Windows, Linux 및 macOS용 Kubernetes 명령줄 도구 `kubectl`은 [여기](https://kubernetes.io/docs/tasks/tools/install-kubectl/)에서 확인할 수 있습니다. 이미 클러스터의 마스터 노드에 사전 설치 및 구성되어 있습니다.

```console
ssh azureuser@<k8s-master-lb-ip>
```

![마스터 노드에서 kubectl 실행](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

관리 작업을 위해 마스터 노드를 jumpbox로 사용하지 않는 것이 좋습니다. `kubectl` 구성은 마스터 노드의 `.kube/config`뿐만 아니라 AKS 엔진 VM에도 저장됩니다. Kubernetes 클러스터에 연결되어 있는 관리 머신에 구성을 복사하고 거기에서 `kubectl` 명령을 사용할 수 있습니다. `.kube/config` 파일은 나중에 Azure Pipelines에서 서비스 연결을 구성하는 데도 사용됩니다.

> [!IMPORTANT]
> 이러한 파일에는 Kubernetes 클러스터에 대한 자격 증명이 포함되어 있으므로 안전하게 보관합니다. 파일에 대한 액세스 권한이 있는 공격자는 관리자 액세스 권한을 얻기에 충분한 정보를 가지고 있습니다. 초기 `.kube/config` 파일을 사용하여 수행되는 모든 작업은 cluster-admin 계정을 통해 수행됩니다.

이제 클러스터의 상태를 확인하기 위해 `kubectl`을 사용하여 다양한 명령을 시도해볼 수 있습니다.

```bash
kubectl get nodes
```

```console
NAME                       STATUS   ROLE     VERSION
k8s-linuxpool-35064155-0   Ready    agent    v1.14.8
k8s-linuxpool-35064155-1   Ready    agent    v1.14.8
k8s-linuxpool-35064155-2   Ready    agent    v1.14.8
k8s-master-35064155-0      Ready    master   v1.14.8
```

```bash
kubectl cluster-info
```

```console
Kubernetes master is running at https://aks.***
CoreDNS is running at https://aks.**_/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
kubernetes-dashboard is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy
Metrics-server is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
```

> [!IMPORTANT]
> Kubernetes에는 세분화된 역할 정의 및 역할 바인딩을 만들 수 있는 자체 _ *RBAC(Role-based Access Control)* * 모델이 있습니다. 이렇게 하면 클러스터 관리 권한을 배포하는 대신 클러스터에 대한 액세스를 제어할 수 있습니다.

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a>Kubernetes 클러스터에 Azure Pipelines 연결

Azure Pipeline을 새로 배포된 Kubernetes 클러스터에 연결하려면 이전 단계에서 설명한 대로 해당 kube config(`.kube/config`) 파일이 필요합니다.

* Kubernetes 클러스터의 마스터 노드 중 하나에 연결합니다.
* `.kube/config` 파일 콘텐츠를 복사합니다.
* Azure DevOps > 프로젝트 설정 > 서비스 연결로 이동하여 새 "Kubernetes" 서비스 연결을 만듭니다(KubeConfig를 인증 방법으로 사용).

> [!IMPORTANT]
> Azure Pipelines(또는 해당 빌드 에이전트)에는 Kubernetes API에 대한 액세스 권한이 있어야 합니다. Azure Pipelines에서 Azure Stack Hub Kubernetes 클러스터까지 인터넷으로 연결된 경우 자체 호스트된 Azure Pipelines 빌드 에이전트를 배포해야 합니다.

Azure Pipelines에 대해 자체 호스팅 에이전트를 배포하는 경우에는 Azure Stack Hub 또는 모든 필수 관리 엔드포인트에 네트워크로 연결된 머신에서 배포할 수 있습니다. 세부 정보는 여기를 참조하세요.

* [Windows](/azure/devops/pipelines/agents/v2-windows) 또는 [Linux](/azure/devops/pipelines/agents/v2-linux)의 [Azure Pipelines 에이전트](/azure/devops/pipelines/agents/agents)

패턴 [배포(CI/CD) 고려 사항](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) 섹션에는 Microsoft 호스팅 에이전트를 사용할지 자체 호스팅 에이전트를 사용할지를 이해하는 데 도움이 되는 의사 결정 흐름이 포함되어 있습니다.

[![의사 결정 흐름 자체 호스팅 에이전트](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)

이 샘플 솔루션에서 토폴로지는 각 Azure Stack Hub 인스턴스에 자체 호스트된 빌드 에이전트를 포함합니다. 에이전트는 Azure Stack Hub 관리 엔드포인트와 Kubernetes 클러스터 API 엔드포인트에 액세스할 수 있습니다.

[![아웃바운드 트래픽 전용](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)

이 디자인은 애플리케이션 솔루션에서 아웃바운드 연결만 있는 일반적인 규제 요구 사항을 충족합니다.

## <a name="configure-monitoring"></a>모니터링 구성

컨테이너에 [Azure Monitor](/azure/azure-monitor/)를 사용하여 솔루션의 컨테이너를 모니터링할 수 있습니다. 그러면 Azure Monitor가 Azure Stack Hub의 AKS 엔진 배포 Kubernetes 클러스터를 가리킵니다.

클러스터에서 Azure Monitor를 사용하도록 설정하는 방법에는 두 가지가 있습니다. 두 가지 방법 모두 Azure에서 Log Analytics 작업 영역을 설정해야 합니다.

* [방법 1](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) Helm 차트 사용
* [방법 2](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) AKS 엔진 클러스터 사양의 일부

샘플 토폴로지에서는 프로세스를 자동화하고 업데이트를 보다 쉽게 설치할 수 있는 "방법 1"이 사용됩니다.

다음 단계에서는 머신에 Azure LogAnalytics 작업 영역(ID 및 키), `Helm`(버전 3) 및 `kubectl`이 필요합니다.

Helm은 macOS, Windows 및 Linux에서 실행되는 바이너리로 사용할 수 있는 Kubernetes 패키지 관리자입니다. 여기에서 다운로드할 수 있습니다. [helm.sh](https://helm.sh/docs/intro/quickstart/) Helm은 `kubectl` 명령에 사용되는 Kubernetes 구성 파일에 의존합니다.

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

이 명령은 Kubernetes 클러스터에 Azure Monitor 에이전트를 설치합니다.

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

Kubernetes 클러스터의 OMS(Operations Management Suite) 에이전트는 모니터링 데이터를 Azure Log Analytics 작업 영역으로 보냅니다(아웃바운드 HTTPS 사용). 이제 Azure Monitor를 사용하여 Azure Stack Hub의 Kubernetes 클러스터에 대한 심층적인 인사이트를 얻을 수 있습니다. 이 디자인은 애플리케이션의 클러스터와 함께 자동으로 배포할 수 있는 분석 능력을 입증하는 강력한 방법입니다.

[![Azure Monitor의 Azure Stack Hub 클러스터](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)

[![Azure Monitor 클러스터 세부 정보](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)

> [!IMPORTANT]
> Azure Monitor가 Azure Stack Hub 데이터를 표시하지 않는 경우 [Azure Loganalytics 작업 영역에 AzureMonitor-Containers 솔루션을 추가하는 방법](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md)에 대한 지침을 따랐는지 확인하세요.

## <a name="deploy-the-application"></a>애플리케이션 배포

샘플 애플리케이션을 설치하기 전에 Kubernetes 클러스터에서 nginx 기반 수신 컨트롤러를 구성하는 또 다른 단계가 있습니다. 수신 컨트롤러는 계층 7 부하 분산 장치로 사용되어 호스트, 경로 또는 프로토콜을 기반으로 클러스터에서 트래픽을 라우팅합니다. Nginx 수신은 Helm 차트로 사용할 수 있습니다. 자세한 지침은 [Helm 차트 GitHub 리포지토리](https://github.com/helm/charts/tree/master/stable/nginx-ingress)를 참조하세요.

이 샘플 애플리케이션은 또한 이전 단계의 [Azure Monitoring Agent](#configure-monitoring)처럼 Helm 차트로 패키지됩니다. 따라서 Kubernetes 클러스터에 애플리케이션을 배포하는 것은 간단합니다. [도우미 GitHub 리포지토리에서 Helm 차트 파일](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)을 찾을 수 있습니다.

샘플 애플리케이션은 3계층 애플리케이션으로, 2개의 Azure Stack Hub 인스턴스 각각의 Kubernetes 클러스터에 배포됩니다. 애플리케이션은 MongoDB 데이터베이스를 사용합니다. 패턴 [데이터 및 스토리지 고려 사항](pattern-highly-available-kubernetes.md#data-and-storage-considerations)에서 여러 인스턴스에 걸쳐 데이터를 복제하는 방법에 대해 자세히 알아볼 수 있습니다.

애플리케이션용 Helm 차트를 배포한 후에는 단일 Pod가 있는 배포 및 상태 저장 세트(데이터베이스 용)로 표시된 애플리케이션의 3계층이 모두 표시됩니다.

```kubectl
kubectl get pod,deployment,statefulset
```

```console
NAME                                         READY   STATUS
pod/ratings-api-569d7f7b54-mrv5d             1/1     Running
pod/ratings-mongodb-0                        1/1     Running
pod/ratings-web-85667bfb86-l6vxz             1/1     Running

NAME                                         READY
deployment.extensions/ratings-api            1/1
deployment.extensions/ratings-web            1/1

NAME                                         READY
statefulset.apps/ratings-mongodb             1/1
```

서비스 쪽에서 nginx 기반 수신 컨트롤러와 해당 공용 IP 주소를 찾을 수 있습니다.

```kubectl
kubectl get service
```

```console
NAME                                         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)
kubernetes                                   ClusterIP      10.0.0.1       <none>        443/TCP
nginx-ingress-1588931383-controller          LoadBalancer   10.0.114.180   *public-ip*   443:30667/TCP
nginx-ingress-1588931383-default-backend     ClusterIP      10.0.76.54     <none>        80/TCP
ratings-api                                  ClusterIP      10.0.46.69     <none>        80/TCP
ratings-web                                  ClusterIP      10.0.161.124   <none>        80/TCP
```

"외부 IP" 주소는 "애플리케이션 엔드포인트"입니다. 이는 사용자가 애플리케이션을 열기 위해 연결하는 방법이며 다음 단계 [트래픽 관리자 구성](#configure-traffic-manager)의 엔드포인트로도 사용됩니다.

## <a name="autoscale-the-application"></a>애플리케이션 자동 크기 조정
선택적으로 [수평 Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)를 구성하여 CPU 사용률과 같은 특정 메트릭을 기준으로 확장 또는 축소할 수 있습니다. 다음 명령은 등급 웹 배포에 의해 제어되는 1-10개의 Pod 복제본을 유지하는 수평 Pod Autoscaler를 만듭니다. HPA는 배포를 통해 복제본 수를 늘리거나 줄여 모든 Pod에서 평균 CPU 사용률을 80%로 유지합니다.

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
다음을 실행하여 자동 크기 조정기의 현재 상태를 확인할 수 있습니다.

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a>Traffic Manager 구성

두 개 이상의 애플리케이션 배포 간에 트래픽을 분산하기 위해 [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview)를 사용합니다. Azure Traffic Manager는 Azure의 DNS 기반 트래픽 부하 분산 장치입니다.

> [!NOTE]
> Traffic Manager는 DNS를 사용하여 클라이언트 요청을 트래픽 라우팅 메서드 및 엔드포인트의 상태를 기반으로 가장 적절한 서비스 엔드포인트로 리디렉션합니다.

Azure Traffic Manager를 사용하는 대신 온-프레미스에서 호스팅되는 다른 전역 부하 분산 솔루션을 사용할 수도 있습니다. 샘플 시나리오에서는 Azure Traffic Manager를 사용하여 애플리케이션의 두 인스턴스 간에 트래픽을 분산합니다. 동일하거나 다른 위치에 있는 Azure Stack Hub 인스턴스에서 실행할 수 있습니다.

![온-프레미스 Traffic Manager](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

Azure에서는 애플리케이션의 서로 다른 두 인스턴스를 가리키도록 Traffic Manager를 구성합니다.

[![TM 엔드포인트 프로필](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)

여기에서 볼 수 있듯이 두 엔드포인트는 [이전 섹션](#deploy-the-application)에서 배포된 애플리케이션의 두 인스턴스를 가리킵니다.

여기에서:
- 수신 컨트롤러를 포함하여 Kubernetes 인프라가 만들어졌습니다.
- 클러스터는 두 개의 Azure Stack Hub 인스턴스를 통해 배포됩니다.
- 모니터링이 구성되었습니다.
- Azure Traffic Manager는 두 Azure Stack Hub 인스턴스에서 트래픽 부하를 분산합니다.
- 이 인프라 외에도 샘플 3계층 애플리케이션이 Helm Chart를 사용하여 자동으로 배포되었습니다. 

이제 솔루션이 가동되어 사용자가 액세스할 수 있습니다!

또한 배포 후 운영 고려 사항에 대해 논의할 몇 가지 사항이 있으며, 이 내용은 다음 두 섹션에서 다룹니다.

## <a name="upgrade-kubernetes"></a>Kubernetes 업그레이드

Kubernetes 클러스터를 업그레이드할 때 다음 항목을 고려합니다.

- Kubernetes 클러스터를 업그레이드하는 작업은 AKS 엔진을 사용하여 수행할 수 있는 복잡한 2일차 작업입니다. 자세한 내용은 [Azure Stack Hub에서 Kubernetes 클러스터 업그레이드](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade)를 참조하세요.
- AKS 엔진을 사용하면 클러스터를 최신 Kubernetes 및 기본 OS 이미지 버전으로 업그레이드할 수 있습니다. 자세한 내용은 [최신 Kubernetes 버전으로 업그레이드하는 단계](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version)를 참조하세요. 
- 또한 기저 노드만 최신 기본 OS 이미지 버전으로 업그레이드할 수 있습니다. 자세한 내용은 [OS 이미지만 업그레이드하는 단계](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image)를 참조하세요.

최신 기본 OS 이미지에는 보안 및 커널 업데이트가 포함되어 있습니다. 새로운 Kubernetes 버전 및 OS 이미지의 가용성을 모니터링하는 것은 클러스터 운영자의 책임입니다. 운영자는 AKS 엔진을 사용하여 이러한 업그레이드를 계획하고 실행해야 합니다. 기본 OS 이미지는 Azure Stack Hub 운영자가 Azure Stack Hub Marketplace에서 다운로드해야 합니다.

## <a name="scale-kubernetes"></a>Kubernetes 크기 조정

크기 조정은 AKS 엔진을 사용하여 오케스트레이션할 수 있는 또 다른 2일차 작업입니다.

크기 조정 명령은 새 Azure Resource Manager 배포에 대한 입력으로 출력 디렉터리에서 클러스터 구성 파일(pimodel.json)을 재사용합니다. AKS 엔진은 특정 에이전트 풀에 대해 크기 조정 작업을 실행합니다. 크기 조정 작업이 완료되면 AKS 엔진은 동일한 apimodel.json 파일에서 클러스터 정의를 업데이트합니다. 클러스터 정의는 업데이트된 현재 클러스터 구성을 반영하기 위해 새 노드 수를 반영합니다.

- [Azure Stack Hub에서 Kubernetes 클러스터 크기 조정](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a>다음 단계

- [하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md)에 대해 자세히 알아보기
- [GitHub에서 이 샘플의 코드](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub)를 검토하고 개선을 제안하세요.