---
title: "Party 클러스터에 Azure Service Fabric 응용 프로그램 배포 | Microsoft Docs"
description: "Party 클러스터에 응용 프로그램을 배포하는 방법에 대해 알아봅니다."
services: service-fabric
documentationcenter: .net
author: mikkelhegn
manager: msfussell
editor: 
ms.assetid: 
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: tutorial
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/09/2017
ms.author: mikhegn
ms.custom: mvc
ms.openlocfilehash: 91d4398589707e8007c4b93639ddb568e39f51a7
ms.sourcegitcommit: 059dae3d8a0e716adc95ad2296843a45745a415d
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/09/2018
---
# <a name="deploy-an-application-to-a-party-cluster-in-azure"></a>Azure에서 Party 클러스터에 응용 프로그램 배포
이 자습서는 시리즈의 2부로, Azure에서 Party 클러스터에 Azure Service Fabric 응용 프로그램을 배포하는 방법을 보여 줍니다.

이 자습서 시리즈의 2부에서는 다음 방법에 대해 알아봅니다.
> [!div class="checklist"]
> * [.NET Service Fabric 응용 프로그램 빌드](service-fabric-tutorial-create-dotnet-app.md)
> * 응용 프로그램을 원격 클러스터에 배포
> * [Visual Studio Team Services를 사용하여 CI/CD 구성](service-fabric-tutorial-deploy-app-with-cicd-vsts.md)
> * [응용 프로그램에 대한 모니터링 및 진단 설정](service-fabric-tutorial-monitoring-aspnet.md)

이 자습서 시리즈에서는 다음 방법에 대해 알아봅니다.
> [!div class="checklist"]
> * Visual Studio를 사용하여 원격 클러스터에 응용 프로그램 배포
> * Service Fabric Explorer를 사용하여 클러스터에서 응용 프로그램 제거

## <a name="prerequisites"></a>필수 조건
이 자습서를 시작하기 전에:
- Azure 구독이 없는 경우 [평가판 계정](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)을 만듭니다.
- [Visual Studio 2017을 설치](https://www.visualstudio.com/)하고 **Azure 개발**과 **ASP.NET 및 웹 개발** 워크로드를 설치합니다.
- [Service Fabric SDK를 설치](service-fabric-get-started.md)합니다.

## <a name="download-the-voting-sample-application"></a>투표 응용 프로그램 예제 다운로드
[이 자습서 시리즈의 1부](service-fabric-tutorial-create-dotnet-app.md)에서 투표 예제 응용 프로그램을 빌드하지 않은 경우 다운로드할 수 있습니다. 명령 창에서 다음 명령을 실행하여 로컬 컴퓨터에 샘플 앱 리포지토리를 복제합니다.

```
git clone https://github.com/Azure-Samples/service-fabric-dotnet-quickstart
```

## <a name="set-up-a-party-cluster"></a>Party 클러스터 설정
Party 클러스터는 평가판으로, Azure에서 호스트되고 Service Fabric 팀이 실행하는 제한 시간 Service Fabric 클러스터입니다. 여기서 누구나 응용 프로그램을 배포하고 플랫폼에 대해 알아볼 수 있습니다. 평가판으로 제공됩니다.

Party 클러스터에 대한 액세스 권한을 얻으려면 http://aka.ms/tryservicefabric 사이트로 이동하고 지침에 따라 클러스터에 대한 액세스 권한을 얻을 수 있습니다. Party 클러스터에 대한 액세스 권한을 얻으려면 Facebook 또는 GitHub 계정이 필요합니다.

원하는 경우 Party 클러스터 대신 자체 클러스터를 사용할 수 있습니다.  ASP.NET Core 웹 프런트 엔드는 역방향 프록시를 사용하여 상태 저장 서비스 백 엔드와 통신합니다.  Party 클러스터 및 로컬 개발 클러스터에는 기본적으로 설정된 역방향 프록시가 있습니다.  투표 응용 프로그램 예제를 사용자 고유의 클러스터에 배포하는 경우 [클러스터에서 역방향 프록시를 사용하도록 설정](service-fabric-reverseproxy.md#setup-and-configuration)해야 합니다.

> [!NOTE]
> Party 클러스터는 보호되지 않으므로 응용 프로그램과 그 안에 있는 데이터는 다른 사람에게 표시될 수 있습니다. 다른 사람이 보기를 원하지 않는 항목은 배포하지 마세요. 모든 세부 사항은 사용 약관을 읽어 보아야 합니다.

[Windows 클러스터에 로그인하고 조인](http://aka.ms/tryservicefabric)합니다. **PFX** 링크를 클릭하여 PFX 인증서를 컴퓨터에 다운로드합니다. 인증서 및 **연결 엔드포인트** 값은 다음 단계에서 사용됩니다.

![PFX 및 연결 엔드포인트](./media/service-fabric-quickstart-containers/party-cluster-cert.png)

Windows 컴퓨터에서 *CurrentUser\My* 인증서 저장소에 PFX를 설치합니다.

```powershell
PS C:\mycertificates> Import-PfxCertificate -FilePath .\party-cluster-873689604-client-cert.pfx -CertStoreLocation Cert:
\CurrentUser\My


  PSParentPath: Microsoft.PowerShell.Security\Certificate::CurrentUser\My

Thumbprint                                Subject
----------                                -------
3B138D84C077C292579BA35E4410634E164075CD  CN=zwin7fh14scd.westus.cloudapp.azure.com
```


## <a name="deploy-the-app-to-the-azure"></a>Azure에 앱 배포
이제 응용 프로그램이 준비되면 Visual Studio에서 Party 클러스터에 직접 배포할 수 있습니다.

1. 솔루션 탐색기에서 **Voting**을 마우스 오른쪽 단추로 클릭하고 **게시**를 선택합니다. 

    ![[게시] 대화 상자](./media/service-fabric-quickstart-containers/publish-app.png)

2. 파티 클러스터 페이지의 **연결 끝점**을 **연결 끝점** 필드에 복사합니다. 예: `zwin7fh14scd.westus.cloudapp.azure.com:19000` **고급 연결 매개 변수**를 클릭하고 다음 정보를 채웁니다.  *FindValue* 및 *ServerCertThumbprint* 값은 이전 단계에서 설치한 인증서의 지문과 일치해야 합니다. **게시**를 클릭합니다. 

    게시가 완료되면 브라우저를 통해 요청을 응용 프로그램에 보낼 수 있습니다.

3. 기본 설정 브라우저를 열고, 클러스터 주소를 입력합니다(포트 정보가 없는 연결 끝점. 예: win1kw5649s.westus.cloudapp.azure.com).

    이제 응용 프로그램을 로컬로 실행할 때와 동일한 결과가 표시됩니다.

    ![클러스터에서 API 응답](./media/service-fabric-tutorial-deploy-app-to-party-cluster/response-from-cluster.png)

## <a name="remove-the-application-from-a-cluster-using-service-fabric-explorer"></a>Service Fabric Explorer를 사용하여 클러스터에서 응용 프로그램 제거
Service Fabric Explorer는 Service Fabric 클러스터에서 응용 프로그램을 탐색하고 관리하는 그래픽 사용자 인터페이스입니다.

Party 클러스터에서 응용 프로그램을 제거하려면:

1. Party 클러스터 등록 페이지에 제공된 링크를 사용하여 Service Fabric Explorer로 이동합니다. 예를 들어 https://win1kw5649s.westus.cloudapp.azure.com:19080/Explorer/index.html입니다.

2. Service Fabric Explorer 왼쪽의 트리 뷰에서 **fabric://Voting** 노드로 이동합니다.

3. 오른쪽 **Essentials** 창에서 **작업** 단추를 클릭하고 **응용 프로그램 삭제**를 선택합니다. 클러스터에서 실행 중인 응용 프로그램의 인스턴스를 제거하는 응용 프로그램 인스턴스 삭제를 확인합니다.

![Service Fabric Explorer에서 응용 프로그램 삭제](./media/service-fabric-tutorial-deploy-app-to-party-cluster/delete-application.png)

## <a name="remove-the-application-type-from-a-cluster-using-service-fabric-explorer"></a>Service Fabric Explorer를 사용하여 클러스터에서 응용 프로그램 형식 제거
응용 프로그램은 Service Fabric 클러스터에서 응용 프로그램 형식으로 배포되므로 클러스터 내에서 실행되는 응용 프로그램의 여러 인스턴스 및 버전을 포함할 수 있습니다. 실행 중인 응용 프로그램 인스턴스를 제거한 후에는 형식도 제거하여 배포 정리를 완료할 수 있습니다.

Service Fabric에서 응용 프로그램 모델에 대한 자세한 내용은 [Service Fabric에서 응용 프로그램 모델링](service-fabric-application-model.md)을 참조하세요.

1. 트리 뷰에서 **VotingType** 노드로 이동합니다.

2. 오른쪽 **Essentials** 창에서 **작업** 단추를 클릭하고 **프로비전 해제 형식**을 선택합니다. 응용 프로그램 형식의 프로비전 해제를 확인합니다.

![Service Fabric Explorer에서 응용 프로그램 형식 프로비전 해제](./media/service-fabric-tutorial-deploy-app-to-party-cluster/unprovision-type.png)

이제 자습서가 완료되었습니다.

## <a name="next-steps"></a>다음 단계
이 자습서에서는 다음 방법에 대해 알아보았습니다.

> [!div class="checklist"]
> * Visual Studio를 사용하여 원격 클러스터에 응용 프로그램 배포
> * Service Fabric Explorer를 사용하여 클러스터에서 응용 프로그램 제거

다음 자습서를 진행합니다.
> [!div class="nextstepaction"]
> [Visual Studio Team Services를 사용하여 연속 통합 설정](service-fabric-tutorial-deploy-app-with-cicd-vsts.md)
