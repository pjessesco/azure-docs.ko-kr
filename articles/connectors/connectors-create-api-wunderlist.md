---
title: "Azure Logic Apps의 Wunderlist 커넥터 | Microsoft Docs"
description: "Wunderlist에 대한 연결을 만들고 이 연결을 사용하여 Logic Apps에서 워크플로를 작성합니다."
services: logic-apps
documentationcenter: .net,nodejs,java
author: MandiOhlinger
manager: anneta
editor: 
tags: connectors
ms.assetid: e4773ecf-3ad3-44b4-a1b5-ee5f58baeadd
ms.service: logic-apps
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: integration
ms.date: 08/18/2016
ms.author: mandia; ladocs
ms.openlocfilehash: 3657955ca4280fecd3a0fb1ea64b90e0a5c5c765
ms.sourcegitcommit: be9a42d7b321304d9a33786ed8e2b9b972a5977e
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/19/2018
---
# <a name="get-started-with-the-wunderlist-connector"></a>Wunderlist 커넥터 시작
Wunderlist는 사람들이 작업할 수 있도록 할 일 목록 및 작업 관리자를 제공합니다.  친구와 식료품 목록 공유하거나 프로젝트에 대해 작업하고 휴가를 계획할 경우 Wunderlist를 사용하여 할 일을 쉽게 캡처하고 공유하며 완료할 수 있습니다. Wunderlist는 어디에서든 모든 작업에 액세스할 수 있도록 전화, 태블릿 및 컴퓨터 간을 즉시 동기화합니다.

논리 앱을 만들어 시작합니다. [논리 앱 만들기](../logic-apps/quickstart-create-first-logic-app-workflow.md)를 참조하세요.

## <a name="create-a-connection-to-wunderlist"></a>Wunderlist에 대한 연결 만들기
Wunderlist로 논리 앱을 만들려면 먼저 **연결**을 만든 후에 다음 속성에 대한 세부 정보를 제공해야 합니다.

| 자산 | 필수 | 설명 |
| --- | --- | --- |
| 신뢰 |예 |Wunderlist 자격 증명을 제공 합니다. |

연결을 만든 후에 사용하여 작업을 실행하고 트리거에 대한 수신을 대기할 수 있습니다.

> [!INCLUDE [Steps to create a connection to Wunderlist](../../includes/connectors-create-api-wunderlist.md)]
> 

## <a name="connector-specific-details"></a>커넥터 관련 세부 정보

[커넥터 세부 정보](/connectors/wunderlist/)에서 swagger에 정의된 모든 트리거 및 작업과 제한 사항도 확인할 수 있습니다.

## <a name="more-connectors"></a>추가 커넥터
[API 목록](apis-list.md)으로 돌아갑니다.