---
title: "Azure API Management에서 게시된 API 모니터링 | Microsoft Docs"
description: "이 자습서의 단계에 따라 Azure API Management에서 API를 모니터링하는 방법을 알아봅니다."
services: api-management
documentationcenter: 
author: juliako
manager: cfowler
editor: 
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.custom: mvc
ms.topic: tutorial
ms.date: 11/19/2017
ms.author: apimpm
ms.openlocfilehash: 445723242a76dcef4a6b137439728235d5d6e32a
ms.sourcegitcommit: 059dae3d8a0e716adc95ad2296843a45745a415d
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/09/2018
---
# <a name="monitor-published-apis"></a>게시된 API 모니터링

Azure Monitor는 모든 Azure 리소스 모니터링을 위한 단일 소스를 제공하는 Azure 서비스입니다. Azure Monitor를 통해 API Management와 같은 Azure 리소스의 메트릭과 로그에 대해 시각화, 쿼리, 라우팅, 보관 및 조치를 수행할 수 있습니다. 

이 자습서에서는 다음 방법에 대해 알아봅니다.

> [!div class="checklist"]
> * 활동 로그 보기
> * 진단 로그 보기
> * API의 메트릭 보기 
> * API가 무단 호출을 받을 경우의 경고 규칙 설정

다음 비디오는 Azure Monitor를 사용하여 API Management를 모니터링하는 방법을 보여 줍니다. 

> [!VIDEO https://channel9.msdn.com/Blogs/AzureApiMgmt/Monitor-API-Management-with-Azure-Monitor/player]
>
>

## <a name="prerequisites"></a>필수 조건

+ 다음 빠른 시작 [Azure API Management 인스턴스 만들기](get-started-create-service-instance.md)를 완료합니다.
+ 또한, 다음 자습서 [첫 번째 API 가져오기 및 게시](import-and-publish.md)를 완료합니다.

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="diagnostic-logs"></a>활동 로그 보기

활동 로그는 API Management 서비스에서 수행된 작업에 대한 정보를 제공합니다. 활동 로그를 통해 API Management 서비스에 대한 모든 쓰기 작업(PUT, POST, DELETE)에서 "무엇을, 누가, 언제"를 판단할 수 있습니다. 

> [!NOTE]
> 활동 로그에는 읽기(GET) 작업 또는 Azure Portal에서 수행되었거나 원본 Management API를 사용하는 작업이 포함되지 않습니다.

API Management 서비스에서 활동 로그에 액세스하거나 Azure Monitor에서 모든 Azure 리소스의 로그에 액세스할 수 있습니다. 

활동 로그를 보려면

1. APIM 서비스 인스턴스를 선택합니다.
2. **활동 로그**를 클릭합니다.

## <a name="view-diagnostic-logs"></a>진단 로그 보기

진단 로그는 감사 뿐만 아니라 문제 해결에 중요한 작업 및 오류에 대한 풍부한 정보를 제공합니다. 진단 로그는 활동 로그와 다릅니다. 활동 로그는 Azure 리소스에서 수행된 작업에 대한 정보를 제공합니다. 진단 로그는 리소스 자체에서 수행하는 작업에 대한 정보를 제공합니다.

진단 로그에 액세스하려면

1. APIM 서비스 인스턴스를 선택합니다.
2. **진단 로그**를 클릭합니다.

## <a name="view-metrics-of-your-apis"></a>API의 메트릭 보기

API Management는 1분 간격으로 메트릭을 내보내, 거의 실시간으로 API의 상태를 확인할 수 있도록 합니다. 다음은 몇 가지 사용 가능한 메트릭에 대한 요약입니다.

* 용량(미리 보기): APIM 서비스를 업그레이드/다운그레이드할지 결정하는 데 도움이 됩니다. 메트릭은 1분 간격으로 내보내지며 보고 시점의 게이트웨이 용량을 반영합니다. 메트릭의 범위는 0-100이고, CPU 및 메모리 사용률 등의 게이트웨이 리소스에 따라 계산됩니다.
* 총 게이트웨이 요청: 기간 동안의 API 요청 수입니다. 
* 성공적인 게이트웨이 요청: 304, 307 및 301보다 작은(예: 200) 모든 항목을 포함하여 성공적인 HTTP 응답 코드를 수신하는 API 요청의 수입니다. 
* 실패한 게이트웨이 요청: 400 및 500보다 큰 모든 항목을 포함하여 잘못된 HTTP 응답 코드를 수신하는 API 요청의 수입니다.
* 허가되지 않은 게이트웨이 요청: 401, 403 및 429를 포함하는 HTTP 응답 코드를 수신하는 API 요청의 수입니다. 
* 기타 게이트웨이 요청: 앞의 범주 중 하나에 속하지 않는(예: 418) HTTP 응답 코드를 수신하는 API 요청의 수입니다.

메트릭에 액세스하려면

1. 페이지 맨 아래의 메뉴에서 **메트릭**을 선택합니다.
2. 드롭다운 목록에서 관심 있는 메트릭을 선택합니다(여러 메트릭을 추가할 수 있음). 
    
    예를 들어 사용 가능한 메트릭 목록에서 **총 게이트웨이 요청** 및 **실패한 게이트웨이 요청**을 선택합니다.
3. 차트에는 총 API 호출 수가 표시됩니다. 또한 실패한 API 호출 수도 표시됩니다. 

## <a name="set-up-an-alert-rule-for-unauthorized-request"></a>권한 없는 요청에 대한 경고 규칙 설정

메트릭 및 활동 로그를 기반으로 경고를 수신하도록 구성할 수 있습니다. Azure Monitor를 사용하여 트리거되면 다음을 수행하도록 경고를 구성할 수 있습니다.

* 전자 메일 알림 보내기
* 웹후크 호출
* Azure 논리 앱 호출

경고를 구성하려면

1. 페이지 맨 아래의 메뉴 모음에서 **경고 규칙**을 선택합니다.
2. **메트릭 경고 추가**를 선택합니다.
3. 이 경고의 **이름**을 입력합니다.
4. 모니터링할 메트릭으로 **무단 게이트웨이 요청**을 선택합니다.
5. **전자 메일 소유자, 참가자 및 구독자**를 선택합니다.
6. **확인**을 누릅니다.
7. API 키를 사용하지 않고 회의 API를 호출합니다. 이 API Management 서비스의 소유자는 전자 메일 알림을 받습니다. 

    > [!TIP]
    > 경고 규칙은 트리거될 때 웹 후크 또는 Azure 논리 앱도 호출할 수 있습니다.

    ![set-up-alert](./media/api-management-azure-monitor/set-up-alert.png)

## <a name="next-steps"></a>다음 단계

이 자습서에서는 다음 방법에 대해 알아보았습니다.

> [!div class="checklist"]
> * 활동 로그 보기
> * 진단 로그 보기
> * API의 메트릭 보기 
> * API가 무단 호출을 받을 경우의 경고 규칙 설정

다음 자습서를 진행합니다.

> [!div class="nextstepaction"]
> [호출 추적](api-management-howto-api-inspector.md)
