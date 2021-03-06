---
title: "성능 권장 사항 - Azure SQL Database | Microsoft Docs"
description: "Azure SQL Database는 현재 쿼리 성능을 향상시킬 수 있는 SQL Database에 대한 권장 사항을 제공합니다."
services: sql-database
documentationcenter: 
author: stevestein
manager: jhubbard
editor: monicar
ms.assetid: 1db441ff-58f5-45da-8d38-b54dc2aa6145
ms.service: sql-database
ms.custom: monitor & tune
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: On Demand
ms.date: 09/20/2017
ms.author: sstein
ms.openlocfilehash: ea1069d4ec29ad66562a6798a8b13998d0d2ef89
ms.sourcegitcommit: 9292e15fc80cc9df3e62731bafdcb0bb98c256e1
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/10/2018
---
# <a name="performance-recommendations"></a>성능 권장 사항

Azure SQL Database에서 응용 프로그램을 학습하여 여기에 맞게 변경하고 사용자 지정된 권장 사항을 제공하여 SQL Database의 성능을 최대화할 수 있습니다. SQL Database 사용 기록을 분석하여 지속적으로 성능을 평가합니다. 제공된 권장 사항은 고유한 데이터베이스 워크로드 패턴에 기반하며 성능을 개선하도록 도와줍니다.

> [!TIP]
> [자동 조정](sql-database-automatic-tuning.md)은 권장되는 성능 조정 방법입니다. [인텔리전스 Insights](sql-database-intelligent-insights.md)는 권장되는 성능 모니터링 방법입니다. 
>

## <a name="create-index-recommendations"></a>인덱스 만들기 권장 사항
Azure SQL Database는 지속적으로 실행되는 쿼리를 모니터링하고 성능을 향상시킬 수 있는 인덱스를 식별합니다. 특정 인덱스가 없다는 충분한 신뢰도가 빌드되면 새 **인덱스 만들기** 권장 사항이 생성됩니다. Azure SQL Database는 시간이 지나면 인덱스가 사용할 수 있는 성능 향상을 예상하여 신뢰도를 빌드합니다. 예상된 성능 향상에 따라 권장 사항은 높음, 보통, 낮음으로 분류됩니다. 

권장 사항을 사용하여 만든 인덱스는 항상 auto_created 인덱스로 플래그가 지정됩니다. sys.indexes 보기를 확인하여 어떤 인덱스가 auto_created인지를 확인할 수 있습니다. 자동으로 만든 인덱스는 ALTER/RENAME 명령을 차단하지 않습니다. 자동으로 만든 인덱스가 있는 열을 삭제하려고 하면 명령이 전달되고 자동으로 만든 인덱스가 만든 명령을 사용하여 삭제됩니다. 일반 인덱스는 인덱싱된 열에서 ALTER/RENAME 명령을 차단합니다.

인덱스 만들기 권장 사항이 적용되면 Azure SQL Database는 초기 성능과 쿼리의 성능을 비교합니다. 새 인덱스가 성능을 향상시킨 경우 권장 사항은 성공한 것으로 플래그가 지정되고 영향 보고서를 사용할 수 있습니다. 인덱스로 인해 이점이 발생하지 않은 경우 자동으로 되돌려집니다. 이러한 방식으로 Azure SQL Database를 사용하면 권장 사항을 사용하여 데이터베이스 성능을 향상하도록 할 수 있습니다.

모든 **인덱스 만들기** 권장 사항에는 데이터베이스 또는 풀의 리소스 사용량이 높은 경우 권장 사항을 적용하지 않는 백 오프 정책이 있습니다. 백 오프 정책은 CPU, 데이터 IO, 로그 IO 및 사용 가능한 저장소를 고려합니다. CPU, 데이터 IO 또는 로그 IO가 지난 30분 이내에 80%보다 높은 경우 인덱스 만들기가 연기됩니다. 인덱스를 만든 후에 사용 가능한 저장소가 10% 아래로 떨어지면 권장 사항은 오류 상태로 전환됩니다. 며칠 후에도 자동 조정 기능이 인덱스가 유용할 것이라고 판단할 경우 프로세스가 다시 시작됩니다. 이 프로세스는 인덱스를 만드는 데 사용할 수 있는 저장소가 충분하거나, 인덱스가 더 이상 유용하지 않을 것으로 확인될 때까지 반복됩니다.

## <a name="drop-index-recommendations"></a>인덱스 삭제 권장 사항
Azure SQL Database는 누락된 인덱스를 검색하는 작업 이외에도 기존 인덱스의 성능을 지속적으로 분석합니다. 인덱스를 사용하지 않으면 Azure SQL Database에서는 삭제하도록 권장합니다. 다음과 같은 두 가지 경우에는 인덱스를 삭제하는 것이 좋습니다.
* 인덱스가 다른 인덱스의 복제본인 경우(동일하게 인덱스되거나 포함된 열, 파티션 스키마 및 필터)
* 연장된 기간(93) 동안 인덱스를 사용하지 않은 경우

인덱스 삭제 권장 사항은 구현 후에 확인도 수행합니다. 성능이 향상된 경우 영향 보고서를 사용할 수 있습니다. 성능 저하가 감지되는 경우 권장 사항이 되돌려집니다.


## <a name="parameterize-queries-recommendations"></a>쿼리 매개 변수화 권장 사항
**쿼리 매개 변수화** 권장 사항은 지속적으로 다시 컴파일되지만 동일한 쿼리 실행 계획으로 종료되는 하나 이상의 쿼리가 있을 때 나타납니다. 이러한 조건에서 강제 매개 변수화를 적용할 기회가 생기며 이를 통해 쿼리 계획을 캐시하고 다시 사용하여 향후 성능을 향상시키고 리소스 사용량을 줄일 수 있습니다. 

SQL Server에 대해 실행되는 모든 쿼리는 실행 계획을 생성하기 위해 초기에 컴파일되어야 합니다. 생성된 각 계획은 계획 캐시에 추가되고, 이후에 동일한 쿼리를 실행할 때 캐시에서 이 계획이 다시 사용될 수 있으므로 추가 컴파일이 필요하지 않게 됩니다. 

매개 변수화되지 않은 값을 포함하는 쿼리를 보내는 응용 프로그램은 성능 오버헤드를 야기할 수 있습니다. 다른 매개 변수 값을 갖는 이러한 모든 쿼리의 경우 실행 계획이 다시 컴파일되기 때문입니다. 많은 경우에 다른 매개 변수 값을 갖는 동일한 쿼리가 동일한 실행 계획을 생성하지만 이러한 계획은 여전히 계획 캐시에 개별적으로 추가됩니다. 실행 계획 재컴파일에는 데이터베이스 리소스가 사용되고, 쿼리 지속 시간이 늘어나고, 계획 캐시가 과도하게 증가하여 캐시에서 계획이 제거될 수 있습니다. 이러한 SQL Server 동작은 데이터베이스에 대해 강제 매개 변수화 옵션을 설정하여 변경할 수 있습니다. 

이러한 권장 사항의 영향을 예측할 수 있도록 하기 위해 실제 CPU 사용량과 예상 CPU 사용량 간의 비교 결과가 제공됩니다(권장 사항을 적용했다고 가정). CPU 절약 외에도, 컴파일에 소요된 시간에 대한 쿼리 기간이 감소합니다. 계획 캐시의 오버헤드도 훨씬 덜 발생하므로 대부분의 계획이 캐시에 유지되고 다시 사용될 수 있게 됩니다. **적용** 명령을 클릭하여 이 권장 사항을 쉽고 빠르게 적용할 수 있습니다. 

이 권장 사항을 적용하면 몇 분 이내에 데이터베이스에 대해 강제 매개 변수화가 설정되며, 모니터링 프로세스가 시작됩니다. 이 프로세스는 약 24시간 동안 진행됩니다. 이 기간 후에는 권장 사항을 적용하기 24시간 전 및 24시간 후의 데이터베이스 CPU 사용량을 보여 주는 유효성 검사 보고서를 볼 수 있습니다. SQL Database 관리자는 성능 저하가 감지될 경우 적용된 권장 사항을 자동으로 되돌리는 안전 메커니즘을 제공합니다.

## <a name="fix-schema-issues-recommendations-preview"></a>스키마 문제 해결 권장 사항(미리 보기)

> [!IMPORTANT]
> Microsoft는 "스키마 문제 해결" 권장 사항을 사용하지 않도록 지정하고 있습니다. "스키마 문제 해결" 권장 사항에서 다룬 스키마 문제를 비롯한 데이터베이스 성능 문제를 자동 모니터링하기 위해 [인텔리전스 Insights](sql-database-intelligent-insights.md)를 사용하여 시작해야 합니다.
> 

**스키마 문제 해결** 권장 사항은 SQL Database 서비스가 Azure SQL Database에서 발생하는 여러 스키마 관련 SQL 오류에서 이상을 확인하면 나타납니다. 이 권장 사항은 데이터베이스에서 한 시간 내에 여러 스키마 관련 오류(잘못된 열 이름, 잘못된 개체 이름 등...)를 발견한 경우에 일반적으로 나타납니다.

"스키마 문제"는 SQL 쿼리 정의와 데이터베이스 스키마 정의가 잘 맞지 않을 때 발생하는 SQL Server의 구문 오류 클래스입니다. 예를 들어 쿼리에서 예상하는 열 중 하나가 대상 테이블에서 누락될 수 있고 그 반대의 경우도 마찬가지입니다. 

“스키마 문제 해결” 권장 사항은 Azure SQL Database 서비스가 Azure SQL Database에서 발생하는 여러 스키마 관련 SQL 오류에서 이상을 확인하면 나타납니다. 다음 표에는 스키마 문제와 관련된 오류가 나와 있습니다.

| SQL 오류 코드 | Message |
| --- | --- |
| 201 |프로시저 또는 함수 '*'에서 매개 변수 '*'이(가) 필요하지만 제공되지 않았습니다. |
| 207 |잘못된 열 이름: '*'. |
| 208 |잘못된 개체 이름: '*'. |
| 213 |제공된 값의 개수나 열 이름이 테이블 정의와 일치하지 않습니다. |
| 2812 |저장 프로시저를 찾을 수 없습니다. '*'. |
| 8144 |프로시저 또는 함수 *에 너무 많은 인수가 지정되었습니다. |

## <a name="next-steps"></a>다음 단계
권장 사항을 모니터링하고 개선된 성능을 계속 적용합니다. 데이터베이스 워크로드는 동적 이며 지속적으로 변경합니다. SQL Database 관리자는 데이터베이스 성능을 잠재적으로 향상시킬 권장 사항을 계속 제공하고 모니터링할 것입니다. 

* 데이터베이스 인덱스 및 쿼리 실행 계획의 자동 튜닝에 대 한 자세한 내용은 [Azure SQL Database 자동 튜닝](sql-database-automatic-tuning.md)을 참조하세요.
* 자동화된 진단을 사용하여 데이터베이스 성능을 자동으로 모니터링 및 성능 문제의 근본 원인 분석에 대한 자세한 내용은 [Azure SQL Intelligent Insights](sql-database-intelligent-insights.md)를 참조하세요.
* Azure Portal에서 성능 권장 사항을 사용하는 방법에 대한 단계는 [Azure Portal의 성능 권장 사항](sql-database-advisor-portal.md)을 참조하세요.
* 상위 쿼리의 성능에 미치는 영향을 알아보려면 [Query Performance Insight](sql-database-query-performance.md)를 참조하세요.


