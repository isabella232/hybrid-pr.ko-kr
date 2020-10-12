---
title: Azure 및 Azure Stack Hub에 고가용성 MongoDB 솔루션 배포
description: Azure 및 Azure Stack Hub에 고가용성 MongoDB 솔루션을 배포하는 방법에 대해 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: def9abaa2a7231648f11453f66119399be015a4d
ms.sourcegitcommit: 485a1f97fa1579364e2be1755cadfc5ea89db50e
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/08/2020
ms.locfileid: "91852510"
---
# <a name="deploy-a-highly-available-mongodb-solution-to-azure-and-azure-stack-hub"></a>Azure 및 Azure Stack Hub에 고가용성 MongoDB 솔루션 배포

이 문서에서는 두 개의 Azure Stack Hub 환경에서 DR(재해 복구) 사이트를 사용하여 기본 HA(고가용성) MongoDB 클러스터를 자동으로 배포하는 과정을 안내합니다. MongoDB 및 고가용성에 대한 자세한 내용은 [복제본 세트 멤버](https://docs.mongodb.com/manual/core/replica-set-members/)를 참조하세요.

이 솔루션에서는 다음을 수행하는 샘플 환경을 만듭니다.

> [!div class="checklist"]
> - 두 Azure Stack Hub에서 배포를 오케스트레이션합니다.
> - Docker를 사용하여 Azure API 프로필의 종속성 문제를 최소화합니다.
> - 재해 복구 사이트를 사용하여 기본 고가용성 MongoDB 클러스터를 배포합니다.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub는 Azure의 확장입니다. Azure Stack Hub는 온-프레미스 환경에 클라우드 컴퓨팅의 민첩성과 혁신을 제공하여 어디서나 하이브리드 앱을 빌드하고 배포할 수 있는 유일한 하이브리드 클라우드를 사용하도록 설정합니다.  
> 
> [하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 문서는 하이브리드 앱 디자인, 배포 및 운영에 대한 소프트웨어 품질(배치, 확장성, 가용성, 복원력, 관리 효율성 및 보안)의 핵심 요소를 검토합니다. 디자인 고려 사항은 하이브리드 앱 디자인을 최적화하고 프로덕션 환경에서 문제를 최소화하는 데 도움이 됩니다.

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a>Azure Stack Hub가 있는 MongoDB 아키텍처

![Azure Stack Hub의 고가용성 MongoDB 아키텍처](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a>Azure Stack Hub가 있는 MongoDB에 대한 필수 구성 요소

- 2개의 연결된 Azure Stack Hub 통합 시스템(Azure Stack Hub) 이 배포는 ASDK(Azure Stack Development Kit)에서 작동하지 않습니다. Azure Stack Hub에 대한 자세한 내용은 [Azure Stack Hub란?](https://azure.microsoft.com/products/azure-stack/hub/)을 참조하세요.
  - 각 Azure Stack Hub에 대한 테넌트 구독 
  - **각 Azure Stack Hub에 대한 각 구독 ID와 Azure Resource Manager 엔드포인트를 기록해 둡니다.**
- 각 Azure Stack Hub에서 테넌트 구독에 대한 권한이 있는 Azure AD(Azure Active Directory) 서비스 사용자입니다. Azure Stack Azure Active Directory가 다른 Azure AD 테넌트에 대해 배포되는 경우 두 개의 서비스 사용자를 만들어야 할 수 있습니다. Azure Stack Hub에 대한 서비스 주체를 만드는 방법에 대한 자세한 내용은 [앱 ID를 사용하여 Azure Stack Hub 리소스에 액세스](/azure-stack/user/azure-stack-create-service-principals)를 참조하세요.
  - **각 서비스 사용자의 애플리케이션 ID, 클라이언트 암호 및 테넌트 이름(xxxxx.onmicrosoft.com)을 기록해 둡니다.**
- 각 Azure Stack Hub의 Marketplace에 배포된 Ubuntu 16.04 마켓플레이스 배포에 대한 자세한 내용은 [Marketplace 항목을 Azure Stack Hub에 다운로드](/azure-stack/operator/azure-stack-download-azure-marketplace-item)를 참조하세요.
- 로컬 머신에 설치된 [Windows용 Docker](https://docs.docker.com/docker-for-windows/)

## <a name="get-the-docker-image"></a>Docker 이미지 가져오기

각 배포에 대한 Docker 이미지는 서로 다른 Azure PowerShell 버전 간의 종속성 문제를 제거합니다.

1. Windows용 Docker에서 Windows 컨테이너를 사용하고 있는지 확인합니다.
2. 관리자 권한 명령 프롬프트에서 다음 명령을 실행하여 배포 스크립트를 통해 Docker 컨테이너를 가져옵니다.

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a>클러스터 배포

1. 컨테이너 이미지를 성공적으로 끌어온 후 이미지를 시작합니다.

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. 컨테이너가 시작되면 컨테이너에 관리자 권한 PowerShell 터미널이 제공됩니다. 배포 스크립트로 가져올 디렉터리를 변경합니다.

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. 배포를 실행합니다. 필요한 경우 자격 증명과 리소스 이름을 제공합니다. HA는 HA 클러스터가 배포될 Azure Stack Hub를 나타냅니다. DR은 DR 클러스터가 배포될 Azure Stack Hub를 나타냅니다.

    ```powershell
    .\Deploy-AzureResourceGroup.ps1 `
    -AzureStackApplicationId_HA "applicationIDforHAServicePrincipal" `
    -AzureStackApplicationSercet_HA "clientSecretforHAServicePrincipal" `
    -AADTenantName_HA "hatenantname.onmicrosoft.com" `
    -AzureStackResourceGroup_HA "haresourcegroupname" `
    -AzureStackArmEndpoint_HA "https://management.haazurestack.com" `
    -AzureStackSubscriptionId_HA "haSubscriptionId" `
    -AzureStackApplicationId_DR "applicationIDforDRServicePrincipal" `
    -AzureStackApplicationSercet_DR "ClientSecretforDRServicePrincipal" `
    -AADTenantName_DR "drtenantname.onmicrosoft.com" `
    -AzureStackResourceGroup_DR "drresourcegroupname" `
    -AzureStackArmEndpoint_DR "https://management.drazurestack.com" `
    -AzureStackSubscriptionId_DR "drSubscriptionId"
    ```

4. NuGet 공급자 설치를 허용하기 위해 `Y`를 입력합니다. 그러면 설치될 API 프로필 "2018-03-01-hybrid" 모듈이 시작됩니다.

5. HA 리소스가 먼저 배포됩니다. 배포를 모니터링하고 배포가 끝나기를 기다립니다. HA 배포가 완료되었다는 메시지가 표시되면 HA Azure Stack Hub의 포털에서 배포된 리소스를 확인할 수 있습니다.

6. DR 리소스의 배포를 계속 진행하고 DR Azure Stack Hub의 점프 상자를 사용하여 클러스터와 상호 작용하도록 할지 결정합니다.

7. DR 리소스 배포가 완료될 때까지 기다립니다.

8. DR 리소스 배포가 완료되면 컨테이너를 종료합니다.

  ```powershell
  exit
  ```

## <a name="next-steps"></a>다음 단계

- DR Azure Stack Hub에서 점프 상자 VM을 사용하도록 설정한 경우 SSH를 통해 연결하고 mongo CLI를 설치하여 MongoDB 클러스터와 상호 작용할 수 있습니다. MongoDB와 상호 작용하는 방법에 대한 자세한 내용은 [mongo Shell](https://docs.mongodb.com/manual/mongo/)을 참조하세요.
- 하이브리드 클라우드 앱에 대해 자세히 알아보려면 [하이브리드 클라우드 솔루션](/azure-stack/user/)을 참조하세요.
- [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)에서 코드를 이 샘플로 수정합니다.