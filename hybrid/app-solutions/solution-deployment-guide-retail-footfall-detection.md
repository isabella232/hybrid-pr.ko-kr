---
title: Azure 및 Azure Stack Hub에서 AI 기반 발걸음 감지 솔루션 배포
description: Azure 및 Azure Stack Hub를 사용하여 소매점에서 방문자 트래픽을 분석하는 AI 기반 발걸음 감지 솔루션을 배포하는 방법을 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 5f2e18e164e54f60b1bb7a14026a0c75c7d7ce69
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477170"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a>Azure 및 Azure Stack Hub를 사용하여 AI 기반 발걸음 감지 솔루션 배포

이 문서에서는 Azure, Azure Stack Hub 및 Custom Vision AI Dev Kit를 사용하여 실제 작업에서 인사이트를 생성하는 AI 기반 솔루션을 배포하는 방법을 설명합니다.

이 솔루션에서는 다음을 수행하는 방법을 알아봅니다.

> [!div class="checklist"]
> - 에지에 CNAB(Cloud Native Application Bundle)를 배포합니다. 
> - 클라우드 경계를 확장하는 앱을 배포합니다.
> - Edge에서 유추를 위해 Custom Vision AI Dev Kit를 사용합니다.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub는 Azure의 확장입니다. Azure Stack Hub는 온-프레미스 환경에 클라우드 컴퓨팅의 민첩성과 혁신을 제공하여 어디서나 하이브리드 앱을 빌드하고 배포할 수 있는 유일한 하이브리드 클라우드를 사용하도록 설정합니다.  
> 
> [하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 문서는 하이브리드 앱 디자인, 배포 및 운영에 대한 소프트웨어 품질(배치, 확장성, 가용성, 복원력, 관리 효율성 및 보안)의 핵심 요소를 검토합니다. 디자인 고려 사항은 하이브리드 앱 디자인을 최적화하고 프로덕션 환경에서 문제를 최소화하는 데 도움이 됩니다.

## <a name="prerequisites"></a>필수 구성 요소

이 배포 가이드를 시작하기 전에 다음을 확인하십시오.

- [발걸음 감지 패턴](pattern-retail-footfall-detection.md) 항목을 검토합니다.
- 다음을 통해 ASDK(Azure Stack Development Kit) 또는 Azure Stack Hub 통합 시스템 인스턴스에 대한 사용자 액세스 권한을 확보합니다.
  - [Azure Stack Hub 리소스 공급자에 대한 Azure App Service](/azure-stack/operator/azure-stack-app-service-overview.md)를 설치합니다. 설치하려면 Azure Stack Hub 인스턴스에 대한 운영자 액세스 권한이 있거나 관리자에게 문의해야 합니다.
  - App Service 및 스토리지 할당량을 제공하는 제품을 구독합니다. 제품을 만들려면 운영자 액세스 권한이 필요합니다.
- Azure 구독에 대한 액세스 권한을 얻습니다.
  - Azure 구독이 없으면 시작하기 전에 [평가판 계정](https://azure.microsoft.com/free/)에 등록하세요.
- 디렉터리에 두 개의 서비스 주체를 만듭니다.
  - 하나는 Azure 구독 범위에서 액세스할 수 있는 Azure 리소스에 사용하도록 설정합니다.
  - 하나는 Azure Stack Hub 구독 범위에서 액세스할 수 있는 Azure Stack Hub 리소스에 사용하도록 설정합니다.
  - 서비스 주체를 만들고 액세스 권한을 부여하는 방법에 대한 자세한 내용은 [앱 ID를 사용하여 리소스에 액세스](/azure-stack/operator/azure-stack-create-service-principals.md)를 참조하세요. Azure CLI를 사용하는 것을 선호하는 경우 [Azure CLI를 사용하여 Azure 서비스 주체 만들기](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest)를 참조하세요.
- Azure 또는 Azure Stack Hub에 Azure Cognitive Services를 배포합니다.
  - 먼저 [Cognitive Services에 대해 자세히 알아봅니다](https://azure.microsoft.com/services/cognitive-services/).
  - 그런 다음, [Azure Cognitive Services를 Azure Stack Hub에 배포](/azure-stack/user/azure-stack-solution-template-cognitive-services.md)를 방문하여 Azure Stack Hub에 Cognitive Services를 배포합니다. 먼저 미리 보기에 액세스하려면 등록해야 합니다.
- 구성되지 않은 Azure Custom Vision AI Dev Kit를 복제하거나 다운로드합니다. 자세한 내용은 [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/)를 참조하세요.
- Power BI 계정에 등록합니다.
- Azure Cognitive Services Face API 구독 키 및 엔드포인트 URL. [Cognitive Services 체험하기](https://azure.microsoft.com/try/cognitive-services/?api=face-api) 평가판을 통해 두 가지를 모두 얻을 수 있습니다. 또는 [Cognitive Services 계정 만들기](/azure/cognitive-services/cognitive-services-apis-create-account)의 지침을 따르세요.
- 다음 개발 리소스를 설치합니다.
  - [Azure CLI 2.0](/azure-stack/user/azure-stack-version-profiles-azurecli2.md)
  - [Docker CE](https://hub.docker.com/search/?type=edition&offering=community)
  - [Porter](https://porter.sh/). Porter를 사용하면 제공되는 CNAB 번들 매니페스트를 사용하여 클라우드 앱을 배포할 수 있습니다.
  - [Visual Studio Code](https://code.visualstudio.com/)
  - [Visual Studio Code용 Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [Visual Studio Code용 Python 확장](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [Python](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a>하이브리드 클라우드 앱 배포

먼저 Porter CLI를 사용하여 자격 증명 집합을 생성한 다음, 클라우드 앱을 배포합니다.  

1. https://github.com/azure-samples/azure-intelligent-edge-patterns 에서 솔루션 샘플 코드를 복제하거나 다운로드합니다. 

1. Porter는 앱 배포를 자동화하는 자격 증명 세트를 생성합니다. 자격 증명 생성 명령을 실행하기 전에 다음을 사용할 수 있는지 확인합니다.

    - Azure 리소스에 액세스하기 위한 서비스 주체(예: 서비스 주체 ID, 키 및 테넌트 DNS)
    - Azure 구독에 대한 구독 ID
    - Azure Stack Hub 리소스에 액세스하기 위한 서비스 주체(예: 서비스 주체 ID, 키 및 테넌트 DNS)
    - Azure Stack Hub 구독에 대한 구독 ID
    - Azure Cognitive Services Face API 키 및 리소스 엔드포인트 URL

1. Porter 자격 증명 생성 프로세스를 실행하고 프롬프트를 따릅니다.

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. Porter를 실행하려면 매개 변수 세트도 필요합니다. 매개 변수 텍스트 파일을 만들고 다음과 같은 이름/값 쌍을 입력합니다. 필요한 값에 대한 지원이 필요한 경우 Azure Stack Hub 관리자에게 문의하세요.

   > [!NOTE] 
   > `resource suffix` 값은 배포의 리소스가 Azure 전체에서 고유한 이름을 갖도록 하는 데 사용됩니다. 8자 이하의 문자와 숫자로 구성된 고유한 문자열이어야 합니다.

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
   텍스트 파일을 저장하고 경로를 적어둡니다.

1. 이제 Porter를 사용하여 하이브리드 클라우드 앱을 배포할 준비가 되었습니다. 설치 명령을 실행하고 리소스가 Azure 및 Azure Stack Hub에 배포되는 것을 살펴봅니다.

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. 배포가 완료되면 다음 값을 적어둡니다.
    - 카메라의 연결 문자열
    - 이미지 스토리지 계정 연결 문자열
    - 리소스 그룹 이름

## <a name="prepare-the-custom-vision-ai-devkit"></a>Custom Vision AI DevKit 준비

다음으로 [Vision AI DevKit 빠른 시작](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/)을 참조하여 Custom Vision AI Dev Kit를 설정합니다. 또한 이전 단계에서 제공한 연결 문자열을 사용하여 카메라를 설정하고 테스트합니다.

## <a name="deploy-the-camera-app"></a>카메라 앱 배포

Porter CLI를 사용하여 자격 증명 집합을 생성한 다음, 카메라 앱을 배포합니다.

1. Porter는 앱 배포를 자동화하는 자격 증명 세트를 생성합니다. 자격 증명 생성 명령을 실행하기 전에 다음을 사용할 수 있는지 확인합니다.

    - Azure 리소스에 액세스하기 위한 서비스 주체(예: 서비스 주체 ID, 키 및 테넌트 DNS)
    - Azure 구독에 대한 구독 ID
    - 클라우드 앱을 배포할 때 제공한 이미지 스토리지 계정 연결 문자열

1. Porter 자격 증명 생성 프로세스를 실행하고 프롬프트를 따릅니다.

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. Porter를 실행하려면 매개 변수 세트도 필요합니다. 매개 변수 텍스트 파일을 만들고 다음과 같은 텍스트를 입력합니다. 필요한 값을 모르는 경우 Azure Stack Hub 관리자에게 문의하세요.

    > [!NOTE]
    > `deployment suffix` 값은 배포의 리소스가 Azure 전체에서 고유한 이름을 갖도록 하는 데 사용됩니다. 8자 이하의 문자와 숫자로 구성된 고유한 문자열이어야 합니다.

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    텍스트 파일을 저장하고 경로를 적어둡니다.

4. 이제 Porter를 사용하여 카메라 앱을 배포할 준비가 되었습니다. 설치 명령을 실행하고 IoT Edge 배포가 생성되는 것을 살펴봅니다.

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. `https://<camera-ip>:3000/`에서 카메라 피드를 확인하여 카메라 배포가 완료되었는지 확인합니다. `<camara-ip>`는 카메라 IP 주소입니다. 이 단계는 최대 10분 정도 걸릴 수 있습니다.

## <a name="configure-azure-stream-analytics"></a>Azure Stream Analytics 구성

이제 카메라에서 Azure Stream Analytics로 데이터가 이동하기 때문에 Power BI와 통신할 수 있도록 수동으로 권한을 부여해야 합니다.

1. Azure Portal에서 **모든 리소스**를 열고 *process-footfall\[yoursuffix\]* 작업을 엽니다.

2. Stream Analytics 작업 창의 **작업 토폴로지** 섹션에서 **출력** 옵션을 선택합니다.

3. **traffic-output** 출력 싱크를 선택합니다.

4. **권한 갱신**을 선택하고 사용자의 Power BI 계정에 로그인합니다.
  
    ![Power BI에서 권한 부여 프롬프트 갱신](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. 출력 설정을 저장합니다.

6. **개요** 창으로 이동하고 **시작**을 선택하여 Power BI로 데이터를 보내기 시작합니다.

7. 작업 출력 시작 시간으로 **지금**을 선택하고 **시작**을 선택합니다. 알림 표시줄에서 작업 상태를 볼 수 있습니다.

## <a name="create-a-power-bi-dashboard"></a>Power BI 대시보드 만들기

1. 작업이 성공하면 [Power BI](https://powerbi.com/)로 이동하여 회사 또는 학교 계정으로 로그인합니다. Stream Analytics 작업 쿼리가 결과를 출력 중이면 앞에서 만든 *footfall-dataset* 데이터 세트가 **데이터 세트** 탭에 있는 것입니다.

2. Power BI 작업 영역에서 **+ 만들기**를 선택하여 *발걸음 분석*이라는 새 대시보드를 만듭니다.

3. 창 맨 위에서 **타일 추가**를 선택합니다. **사용자 지정 스트리밍 데이터**를 선택하고 **다음**을 선택합니다. **데이터 세트** 아래에서 **footfall-dataset**를 선택합니다. **시각화 유형** 드롭다운에서 **카드**를 선택하고 **필드**에 **age**를 추가합니다. **다음**을 선택하여 타일 이름을 입력하고, **적용**을 선택하여 타일을 만듭니다.

4. 필요에 따라 필드와 카드를 더 추가할 수 있습니다.

## <a name="test-your-solution"></a>솔루션 테스트

카메라 앞에서 사람들이 걷는 동안 Power BI에서 생성한 카드의 데이터가 어떻게 변하는지 관찰합니다. 기록된 후 추론이 나타나는 데 최대 20초가 걸릴 수 있습니다.

## <a name="remove-your-solution"></a>솔루션 제거

솔루션을 제거하려면 배포용으로 만든 것과 동일한 매개 변수 파일을 사용하고 Porter를 사용하여 다음 명령을 실행합니다.

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a>다음 단계

- [하이브리드 앱 디자인 고려 사항]에 대해 자세히 알아보세요(overview-app-design-considerations.md).
- [GitHub에서 이 샘플의 코드](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis)를 검토하고 개선을 제안하세요.
