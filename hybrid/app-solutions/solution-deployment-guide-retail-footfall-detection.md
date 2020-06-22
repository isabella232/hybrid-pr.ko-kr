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
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a>Azure 및 Azure Stack Hub를 사용 하 여 AI 기반 footfall 검색 솔루션 배포

이 문서에서는 Azure, Azure Stack Hub 및 Custom Vision AI Dev Kit를 사용 하 여 실제 작업에서 통찰력을 생성 하는 AI 기반 솔루션을 배포 하는 방법을 설명 합니다.

이 솔루션에서는 다음 방법에 대해 알아봅니다.

> [!div class="checklist"]
> - 에 지에 클라우드 네이티브 응용 프로그램 번들 (CNAB)을 배포 합니다. 
> - 클라우드 경계를 확장 하는 앱을 배포 합니다.
> - Edge에서 유추를 위해 Custom Vision AI Dev Kit를 사용 합니다.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack 허브는 Azure의 확장입니다. Azure Stack 허브는 온-프레미스 환경에 클라우드 컴퓨팅의 민첩성과 혁신을 제공 하 여 어디서 나 하이브리드 앱을 빌드하고 배포할 수 있는 유일한 하이브리드 클라우드를 사용 하도록 설정 합니다.  
> 
> [하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 문서에서는 하이브리드 앱을 디자인, 배포 및 운영 하기 위한 소프트웨어 품질 (배치, 확장성, 가용성, 복원 력, 관리 효율성 및 보안)의 핵심 요소을 검토 합니다. 디자인 고려 사항은 하이브리드 앱 디자인을 최적화 하 고 프로덕션 환경에서 문제를 최소화 하는 데 도움이 됩니다.

## <a name="prerequisites"></a>필수 조건

이 배포 가이드를 시작 하기 전에 다음을 확인 하십시오.

- [Footfall 검색 패턴](pattern-retail-footfall-detection.md) 항목을 검토 합니다.
- 다음을 사용 하 여 Azure Stack Development Kit (ASDK) 또는 Azure Stack 허브 통합 시스템 인스턴스에 대 한 사용자 액세스 권한을 얻습니다.
  - [Azure App Service Azure Stack Hub 리소스 공급자가](/azure-stack/operator/azure-stack-app-service-overview.md) 설치 되었습니다. 운영자가 Azure Stack 허브 인스턴스에 액세스 하거나 관리자에 게 문의 하 여를 설치 해야 합니다.
  - App Service 및 저장소 할당량을 제공 하는 제품에 대 한 구독입니다. 제품을 만들려면 운영자 액세스가 필요 합니다.
- Azure 구독에 대 한 액세스 권한을 얻습니다.
  - Azure 구독이 없는 경우 시작 하기 전에 [무료 평가판 계정](https://azure.microsoft.com/free/) 에 등록 합니다.
- 디렉터리에 두 개의 서비스 주체를 만듭니다.
  - Azure 구독 범위에서 액세스와 함께 Azure 리소스에서 사용 하도록 설정 합니다.
  - Azure Stack 허브 구독 범위에서 액세스 권한이 있는 Azure Stack 허브 리소스와 함께 사용 하도록 설정 합니다.
  - 서비스 주체를 만들고 액세스 권한을 부여 하는 방법에 대 한 자세한 내용은 [앱 id를 사용 하 여 리소스에 액세스](/azure-stack/operator/azure-stack-create-service-principals.md)를 참조 하세요. Azure CLI를 사용 하려면 Azure CLI를 사용 하 [여 Azure 서비스 주체 만들기](https://docs.microsoft.com/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest)를 참조 하세요.
- Azure 또는 Azure Stack 허브에 Azure Cognitive Services를 배포 합니다.
  - 먼저 [Cognitive Services에 대해 자세히 알아보세요](https://azure.microsoft.com/services/cognitive-services/).
  - 그런 다음 [Azure Stack hub에 Azure Cognitive Services 배포](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) 를 방문 하 여 Azure Stack 허브에 Cognitive Services을 배포 합니다. 먼저 미리 보기에 대 한 액세스를 등록 해야 합니다.
- 구성 되지 않은 Azure Custom Vision AI Dev Kit를 복제 하거나 다운로드 합니다. 자세한 내용은 [비전 AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/)를 참조 하세요.
- Power BI 계정에 등록 합니다.
- Azure Cognitive Services Face API 구독 키 및 끝점 URL입니다. [Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) 체험 평가판을 사용 하 여 두 가지를 모두 사용할 수 있습니다. 또는 [Cognitive Services 계정 만들기](/azure/cognitive-services/cognitive-services-apis-create-account)의 지침을 따르세요.
- 다음 개발 리소스를 설치 합니다.
  - [Azure CLI 2.0](/azure-stack/user/azure-stack-version-profiles-azurecli2.md)
  - [Docker CE](https://hub.docker.com/search/?type=edition&offering=community)
  - [Porter](https://porter.sh/). Porter를 사용 하 여 제공 된 CNAB 번들 매니페스트를 사용 하 여 클라우드 앱을 배포 합니다.
  - [Visual Studio Code](https://code.visualstudio.com/)
  - [Visual Studio Code 용 Azure IoT 도구](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [Visual Studio Code 용 Python 확장](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [Python](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a>하이브리드 클라우드 앱 배포

먼저 Porter CLI를 사용 하 여 자격 증명 집합을 생성 한 다음 클라우드 앱을 배포 합니다.  

1. 에서 솔루션 샘플 코드를 복제 하거나 다운로드 https://github.com/azure-samples/azure-intelligent-edge-patterns 합니다. 

1. Porter는 앱의 배포를 자동화 하는 자격 증명 집합을 생성 합니다. 자격 증명 생성 명령을 실행 하기 전에 다음을 사용할 수 있어야 합니다.

    - 서비스 주체 ID, 키 및 테 넌 트 DNS를 포함 하 여 Azure 리소스에 액세스 하기 위한 서비스 주체입니다.
    - Azure 구독에 대 한 구독 ID입니다.
    - 서비스 주체 ID, 키 및 테 넌 트 DNS를 포함 하 여 Azure Stack 허브 리소스에 액세스 하기 위한 서비스 주체입니다.
    - Azure Stack Hub 구독의 구독 ID입니다.
    - Azure Cognitive Services Face API 키와 리소스 끝점 URL입니다.

1. Porter 자격 증명 생성 프로세스를 실행 하 고 프롬프트를 따릅니다.

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. 또한 Porter를 실행 하려면 매개 변수 집합이 필요 합니다. 매개 변수 텍스트 파일을 만들고 다음 이름/값 쌍을 입력 합니다. 필요한 값에 대 한 지원이 필요한 경우 Azure Stack 허브 관리자에 게 문의 하세요.

   > [!NOTE] 
   > `resource suffix`값은 배포 리소스의 이름이 Azure 전체에 고유한 지 확인 하는 데 사용 됩니다. 문자 및 숫자의 고유 문자열 이어야 하며 8 자이 하 여야 합니다.

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
   텍스트 파일을 저장 하 고 해당 경로를 적어 둡니다.

1. 이제 Porter을 사용 하 여 하이브리드 클라우드 앱을 배포할 준비가 되었습니다. Azure에 배포 되 고 Azure Stack 허브에 리소스가 배포 되는 경우 설치 명령을 실행 하 고 감시 합니다.

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. 배포가 완료 되 면 다음 값을 기록해 둡니다.
    - 카메라의 연결 문자열입니다.
    - 이미지 저장소 계정 연결 문자열입니다.
    - 리소스 그룹 이름입니다.

## <a name="prepare-the-custom-vision-ai-devkit"></a>Custom Vision AI DevKit 준비

다음으로, [비전 Ai devkit 빠른](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/)시작에 표시 된 대로 Custom Vision Ai Dev kit를 설정 합니다. 또한 이전 단계에서 제공 된 연결 문자열을 사용 하 여 카메라를 설정 하 고 테스트 합니다.

## <a name="deploy-the-camera-app"></a>카메라 앱 배포

Porter CLI를 사용 하 여 자격 증명 집합을 생성 한 다음 카메라 앱을 배포 합니다.

1. Porter는 앱의 배포를 자동화 하는 자격 증명 집합을 생성 합니다. 자격 증명 생성 명령을 실행 하기 전에 다음을 사용할 수 있어야 합니다.

    - 서비스 주체 ID, 키 및 테 넌 트 DNS를 포함 하 여 Azure 리소스에 액세스 하기 위한 서비스 주체입니다.
    - Azure 구독에 대 한 구독 ID입니다.
    - 클라우드 앱을 배포할 때 제공 되는 이미지 저장소 계정 연결 문자열입니다.

1. Porter 자격 증명 생성 프로세스를 실행 하 고 프롬프트를 따릅니다.

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. 또한 Porter를 실행 하려면 매개 변수 집합이 필요 합니다. 매개 변수 텍스트 파일을 만들고 다음 텍스트를 입력 합니다. 필요한 값 중 일부를 모르는 경우 Azure Stack 허브 관리자에 게 문의 하세요.

    > [!NOTE]
    > `deployment suffix`값은 배포 리소스의 이름이 Azure 전체에 고유한 지 확인 하는 데 사용 됩니다. 문자 및 숫자의 고유 문자열 이어야 하며 8 자이 하 여야 합니다.

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    텍스트 파일을 저장 하 고 해당 경로를 적어 둡니다.

4. 이제 Porter을 사용 하 여 카메라 앱을 배포할 준비가 되었습니다. 설치 명령을 실행 하 고 IoT Edge 배포가 생성 되 면 시청 합니다.

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. 에서 카메라 피드를 보면 카메라의 배포가 완료 되었는지 확인 `https://<camera-ip>:3000/` `<camara-ip>` 합니다. 여기서는 카메라 IP 주소입니다. 이 단계는 최대 10 분 정도 걸릴 수 있습니다.

## <a name="configure-azure-stream-analytics"></a>Azure Stream Analytics 구성

이제 데이터를 카메라에서 Azure Stream Analytics로 이동 했으므로 Power BI와 통신 하도록 수동으로 권한을 부여 해야 합니다.

1. Azure Portal에서 **모든 리소스**와 *프로세스-footfall의 \[ 접미사 \] * 작업을 엽니다.

2. Stream Analytics 작업 창의 **작업 토폴로지** 섹션에서 **출력** 옵션을 선택합니다.

3. **트래픽 출력** 출력 싱크를 선택 합니다.

4. **권한 부여 갱신** 을 선택 하 고 Power BI 계정에 로그인 합니다.
  
    ![Power BI에서 권한 부여 프롬프트 갱신](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. 출력 설정을 저장 합니다.

6. **개요** 창으로 이동 하 고 **시작** 을 선택 하 여 Power BI로 데이터 보내기를 시작 합니다.

7. 작업 출력 시작 시간으로 **지금**을 선택하고 **시작**을 선택합니다. 알림 표시줄에서 작업 상태를 볼 수 있습니다.

## <a name="create-a-power-bi-dashboard"></a>Power BI 대시보드 만들기

1. 작업이 성공 하면 [Power BI](https://powerbi.com/) 으로 이동 하 여 회사 또는 학교 계정으로 로그인 합니다. Stream Analytics 작업 쿼리가 결과를 출력 하는 경우 만든 *footfall* 데이터 집합 **은 데이터 집합 탭에** 있습니다.

2. Power BI 작업 영역에서 **+ 만들기** 를 선택 하 여 *Footfall Analysis* 라는 새 대시보드를 만듭니다.

3. 창 맨 위에서 **타일 추가**를 선택합니다. **사용자 지정 스트리밍 데이터**를 선택하고 **다음**을 선택합니다. **데이터**집합에서 **footfall** 을 선택 합니다. **시각화 유형** 드롭다운에서 **카드** 를 선택 하 고 **필드**에 **age** 를 추가 합니다. **다음**을 선택하여 타일 이름을 입력하고, **적용**을 선택하여 타일을 만듭니다.

4. 필요에 따라 추가 필드와 카드를 추가할 수 있습니다.

## <a name="test-your-solution"></a>솔루션 테스트

카메라 앞에서 다른 사람이 탐색 하는 Power BI에서 만든 카드의 데이터가 어떻게 변경 되는지 관찰 합니다. 추론 기록 된 후에 표시 되는 데 최대 20 초가 걸릴 수 있습니다.

## <a name="remove-your-solution"></a>솔루션 제거

솔루션을 제거 하려면 배포를 위해 만든 것과 동일한 매개 변수 파일을 사용 하 여 Porter를 사용 하 여 다음 명령을 실행 합니다.

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a>다음 단계

- [하이브리드 앱 디자인 고려 사항]에 대해 자세히 알아보세요. (overview-app-design-considerations.md)
- [GitHub에서이 샘플의 코드에 대 한 향상 된](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis)기능을 검토 하 고 제안 합니다.
