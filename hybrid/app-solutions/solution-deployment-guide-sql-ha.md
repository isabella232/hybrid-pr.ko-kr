---
title: Azure 및 Azure Stack 허브에 SQL Server 2016 가용성 그룹 배포
description: SQL Server 2016 가용성 그룹을 Azure 및 Azure Stack 허브에 배포 하는 방법에 대해 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ff6d5b9667e63a6b8d232b6dd93db2d8b12fd46d
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911043"
---
# <a name="deploy-a-sql-server-2016-availability-group-to-azure-and-azure-stack-hub"></a>Azure 및 Azure Stack 허브에 SQL Server 2016 가용성 그룹 배포

이 문서에서는 두 개의 Azure Stack 허브 환경에서 비동기 재해 복구 (DR) 사이트를 사용 하 여 기본 HA (고가용성) SQL Server 2016 Enterprise 클러스터를 자동으로 배포 하는 과정을 안내 합니다. SQL Server 2016 및 고가용성에 대 한 자세한 내용은 [Always On 가용성 그룹: 고가용성 및 재해 복구 솔루션](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016)(영문)을 참조 하세요.

이 솔루션에서는 다음을 수행 하는 샘플 환경을 빌드합니다.

> [!div class="checklist"]
> - 두 Azure Stack 허브에서 배포를 오케스트레이션 합니다.
> - Docker를 사용 하 여 Azure API 프로필의 종속성 문제를 최소화 합니다.
> - 재해 복구 사이트를 사용 하 여 기본 항상 사용 가능한 SQL Server 2016 Enterprise 클러스터를 배포 합니다.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack 허브는 Azure의 확장입니다. Azure Stack 허브는 온-프레미스 환경에 클라우드 컴퓨팅의 민첩성과 혁신을 제공 하 여 어디서 나 하이브리드 앱을 빌드하고 배포할 수 있는 유일한 하이브리드 클라우드를 사용 하도록 설정 합니다.  
> 
> [하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 문서에서는 하이브리드 앱을 디자인, 배포 및 운영 하기 위한 소프트웨어 품질 (배치, 확장성, 가용성, 복원 력, 관리 효율성 및 보안)의 핵심 요소을 검토 합니다. 디자인 고려 사항은 하이브리드 앱 디자인을 최적화 하 고 프로덕션 환경에서 문제를 최소화 하는 데 도움이 됩니다.

## <a name="architecture-for-sql-server-2016"></a>SQL Server 2016의 아키텍처

![SQL Server 2016 SQL HA Azure Stack 허브](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a>SQL Server 2016에 대 한 필수 구성 요소

- 2 개의 연결 된 Azure Stack 허브 통합 시스템 (Azure Stack 허브). 이 배포는 Azure Stack Development Kit (ASDK)에서 작동 하지 않습니다. Azure Stack Hub에 대해 자세히 알아보려면 [Azure Stack 개요](https://azure.microsoft.com/overview/azure-stack/)를 참조 하세요.
- 각 Azure Stack 허브에 대 한 테 넌 트 구독
  - **각 Azure Stack 허브에 대 한 각 구독 ID와 Azure Resource Manager 끝점을 기록해 둡니다.**
- 각 Azure Stack 허브에서 테 넌 트 구독에 대 한 권한이 있는 Azure Active Directory (Azure AD) 서비스 사용자입니다. Azure Stack 허브가 다른 Azure AD 테 넌 트에 대해 배포 되는 경우 두 개의 서비스 주체를 만들어야 할 수 있습니다. Azure Stack Hub에 대 한 서비스 주체를 만드는 방법을 알아보려면 [서비스 주체 만들기를 참조 하 여 앱에 Azure Stack 허브 리소스에 대 한 액세스 권한을 부여](https://docs.microsoft.com/azure-stack/user/azure-stack-create-service-principals)합니다.
  - **각 서비스 주체의 응용 프로그램 ID, 클라이언트 암호 및 테 넌 트 이름 (xxxxx.onmicrosoft.com)을 기록해 둡니다.**
- Azure Stack 허브의 Marketplace에 SQL Server 2016 엔터프라이즈 게시. Marketplace 배포에 대 한 자세한 내용은 [Azure Stack 허브에 marketplace 항목 다운로드](https://docs.microsoft.com/azure-stack/operator/azure-stack-download-azure-marketplace-item)를 참조 하세요.
    **조직에 적절 한 SQL 라이선스가 있는지 확인 합니다.**
- 로컬 컴퓨터에 설치 된 [Windows용 Docker](https://docs.docker.com/docker-for-windows/)

## <a name="get-the-docker-image"></a>Docker 이미지 가져오기

각 배포에 대 한 Docker 이미지는 서로 다른 Azure PowerShell 버전 간의 종속성 문제를 제거 합니다.

1. Windows용 Docker Windows 컨테이너를 사용 하 고 있는지 확인 합니다.
2. 관리자 권한 명령 프롬프트에서 다음 스크립트를 실행 하 여 배포 스크립트를 사용 하 여 Docker 컨테이너를 가져옵니다.

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a>가용성 그룹 배포

1. 컨테이너 이미지를 성공적으로 끌어온 후 이미지를 시작 합니다.

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. 컨테이너가 시작 되 면 컨테이너에 관리자 권한 PowerShell 터미널이 제공 됩니다. 배포 스크립트로 가져올 디렉터리를 변경 합니다.

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. 배포를 실행 합니다. 필요한 경우 자격 증명과 리소스 이름을 제공 합니다. HA는 HA 클러스터가 배포 될 Azure Stack 허브를 나타냅니다. DR은 DR 클러스터가 배포 될 Azure Stack 허브를 참조 합니다.

      ```powershell
      > .\Deploy-AzureResourceGroup.ps1 `
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

4. `Y`NuGet 공급자 설치를 허용 하려면를 입력 합니다. 그러면 설치 될 API 프로필 "2018-03-01-하이브리드" 모듈이 시작 됩니다.

5. 리소스 배포가 완료 될 때까지 기다립니다.

6. DR 리소스 배포가 완료 되 면 컨테이너를 종료 합니다.

      ```powershell
      exit
      ```

7. 각 Azure Stack 허브 포털의 리소스를 확인 하 여 배포를 검사 합니다. HA 환경에서 SQL 인스턴스 중 하나에 연결 하 고 SSMS (SQL Server Management Studio)를 통해 가용성 그룹을 검사 합니다.

    ![SQL Server 2016 SQL HA](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a>다음 단계

- SQL Server Management Studio를 사용 하 여 클러스터를 수동으로 장애 조치 (failover) 합니다. [Always On 가용성 그룹의 강제 수동 장애 조치 (Failover) 수행 (SQL Server)](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017) 을 참조 하세요.
- 하이브리드 클라우드 앱에 대해 자세히 알아보세요. [하이브리드 클라우드 솔루션을](https://aka.ms/azsdevtutorials) 참조 하세요.
- 사용자 고유의 데이터를 사용 하거나 [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)에서이 샘플에 대 한 코드를 수정 합니다.
