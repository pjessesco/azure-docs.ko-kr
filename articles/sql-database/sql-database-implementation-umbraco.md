---
title: "Azure SQL Database Azure 사례 연구 - Umbraco | Microsoft Docs"
description: "어떻게 Umbraco가 SQL Database를 사용하여 클라우드의 수천 개의 테넌트를 위해 서비스를 신속하게 프로비전하고 확장하는지 알아봅니다."
services: sql-database
documentationcenter: 
author: CarlRabeler
manager: jhubbard
editor: 
ms.assetid: 5243d31e-3241-4cb0-9470-ad488ff28572
ms.service: sql-database
ms.custom: reference
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: Inactive
ms.date: 01/10/2017
ms.author: carlrab
ms.openlocfilehash: c25a66daa87da96d4e77c9021a1ceb4366d7a224
ms.sourcegitcommit: 059dae3d8a0e716adc95ad2296843a45745a415d
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/09/2018
---
# <a name="umbraco-uses-azure-sql-database-to-quickly-provision-and-scale-services-for-thousands-of-tenants-in-the-cloud"></a>Umbraco는 Azure SQL Database를 사용하여 클라우드의 수천 개의 테넌트를 위해 서비스를 신속하게 프로비전하고 확장합니다.
![Umbraco 로고](./media/sql-database-implementation-umbraco/umbracologo.png)

Umbraco는 소규모 캠페인 또는 브로슈어 사이트에서 포춘지 선정 500대 기업과 글로벌 미디어 웹 사이트를 위한 복잡한 응용 프로그램까지 실행이 가능한 인기 높은 오픈 소스 CMS(콘텐츠 관리 시스템)입니다. 

> "우리는 이 시스템을 사용하는 대규모 개발자 커뮤니티를 보유하고 있습니다. 우리 포럼에는 100,000명이 넘는 개발자가 있으며 350,000개 이상의 사이트에서 Umbraco를 실행하고 있습니다."
> 
> - Umbraco기술 책임자, Morten Christensen
> 
> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Azure-SQL-Database-Case-Study-Umbraco/player]
> 
> 

고객 배포를 단순화하기 위해 Umbraco는 UaaS(Umbraco-as a Service)를 추가했습니다. 이 SaaS(software-as-a-service)는 개발자가 솔루션 관리가 아닌 제품 혁신에 집중하도록 함으로써 온-프레미스 배포에 대한 필요성을 없애고, 기본 제공 규모 조정 기능을 제공하고, 관리 오버헤드를 없애줍니다. Umbraco는 Microsoft Azure에서 제공하는 유연한 PaaS(platform-as-a-service) 모델을 활용하여 이러한 모든 혜택을 제공할 수 있습니다.

UaaS는 SaaS 고객이 이전에는 가능하지 않았던 Umbraco CMS 기능을 사용할 수 있도록 합니다. 이러한 고객은 프로덕션 데이터베이스를 포함하는 작업 CMS 환경으로 프로비전됩니다. 고객은 해당 요구에 따라, 개발 및 스테이징 환경을 위해 최대 두 개의 데이터베이스를 더 추가할 수 있습니다. 새 환경이 요청되면 자동화된 프로세스가 해당 고객에게 데이터베이스를 자동으로 할당합니다. 데이터베이스가 사용 가능한 데이터베이스의 Azure 탄력적 풀에서 Umbraco에 의해 미리 프로비전되었으므로 새 데이터베이스는 몇 오 안에 사용 가능해집니다(그림 1 참조).

![Umbraco 프로비전 수명 주기](./media/sql-database-implementation-umbraco/figure1.png)

그림 1. UaaS(Umbraco as a Service)에 대한 프로비전 수명 주기

## <a name="azure-elastic-pools-and-automation-simplify-deployments"></a>Azure 탄력적 풀 및 자동화로 배포 간소화
Azure SQL Database 및 기타 Azure 서비스를 사용하여 Umbraco 고객은 자신의 환경을 자체 프로비전할 수 있으며, Umbraco는 직관적인 워크플로의 일부로 데이터베이스를 쉽게 모니터링하고 관리할 수 있습니다.

1. 프로비전
   
   Umbraco는 탄력적 풀에서 사용할 수 있는 미리 프로비전된 200개의 데이터베이스 용량을 유지 관리합니다. 새 고객이 UaaS에 등록하면 Umbraco는 고객에게 가용성 풀의 데이터베이스를 할당하여 거의 실시간으로 새 CMS 환경을 제공합니다.
   
   가용성 풀이 임계값에 도달하면 새 탄력적 풀이 생성되고 새 데이터베이스가 필요할 때 고객에게 할당될 수 있게 미리 프로비전됩니다.
   
   구현은 C# 관리 라이브러리 및 Azure Service Bus 큐를 사용하여 완전하게 자동화되었습니다.
2. 활용
   
   고객은 각각이 자체 데이터베이스를 포함하는 1~3가지 환경(프로덕션, 스테이징 및/또는 개발용)을 사용합니다. 고객 데이터베이스는 탄력적 풀에 있으므로 Umbraco에서 과도한 프로비전 없이도 효율적으로 규모를 조정할 수 있습니다.
   
   ![Umbraco 프로젝트 개요](./media/sql-database-implementation-umbraco/figure2.png)
   
   ![Umbraco 프로젝트 세부 정보](./media/sql-database-implementation-umbraco/figure3.png)
   
   그림 2. 프로젝트 개요 및 세부 정보를 표시하는 UaaS(Umbraco-as a Service) 고객 웹 사이트
   
   Azure SQL Database는 DTU(데이터베이스 트랜잭션 단위)를 사용하여 실제 데이터베이스 트랜잭션에 필요한 상대적 성능을 나타냅니다. UaaS 고객의 경우 데이터베이스는 일반적으로 약 10개의 DTU에서 작동하지만 각각은 필요에 따라 탄력 있게 확장될 수 있습니다. 즉, UaaS는 사용량이 최대인 시간에도 고객에게 필요한 리소스가 항상 유지되도록 합니다. 예를 들어 최근에 있던 일요일 야간 스포츠 경기 중에, 한 UaaS 고객은 데이터베이스 최대 사용량이 100DTU까지 높아지는 것을 경험했습니다. Azure의 탄력적 풀은 Umbraco가 성능 저하 없이 이러한 높은 수요를 지원하도록 할 수 있습니다.
3. 모니터
   
   Umbraco는 고객의 사용자 지정 전자 메일 경고와 Azure 포털 내의 대시보드를 사용하여 데이터베이스 활동을 모니터링합니다.
4. 재해 복구
   
   Azure는 두 가지 DR(재해 복구) 옵션을 제공합니다. 바로 활성 지역 복제와 지역에서 복원 기능입니다. 회사에서 선택해야 하는 DR 옵션은 해당 [비즈니스 연속성 목표](sql-database-business-continuity.md)에 따라 달라집니다.
   
   활성 지역 복제는 가동 중지 시간 발생 시 가장 빠른 응답 수준을 제공합니다. 활성 지역 복제를 사용하면 서로 다른 지역의 서버에 최대 4개의 읽기 가능 보조 복제본을 만든 후, 장애가 발생했을 때 보조 복제본으로 장애 조치를 시작할 수 있습니다.
   
   Umbraco는 지역에서 복제 기능이 필요 없지만 Azure 지리적 복원을 활용하여 가동 중단 시 최소 가동 중지 시간을 보장하도록 합니다. 지리적 복원은 지역 중복 Azure Storage의 데이터베이스 백업을 사용합니다. 따라서 사용자는 주 지역에서 작동이 중단될 경우 백업 복사본에서 복원할 수 있습니다.
5. 프로비전 해제
   
   프로젝트 환경이 삭제되면 연결된 모든 데이터베이스(개발, 스테이징 또는 라이브)가 Azure Service Bus 큐 정리 동안 제거됩니다. 이 자동화된 프로세스에서는 미사용 데이터베이스를 Umbraco의 탄력적 데이터베이스 가용성 풀로 복원하여, 최대 사용률을 유지하면서 향후 프로비전에 사용할 수 있도록 합니다.

## <a name="elastic-pools-allow-uaas-to-scale-with-ease"></a>탄력적 풀로 손쉽게 UaaS 크기 조정
Azure의 탄력적 풀을 활용하여 Umbraco는 과도하거나 부족한 프로비전 없이 고객을 위해 성능을 최적화할 수 있습니다. Umbraco는 현재 19의 탄력적 풀에 약 3,000개의 데이터베이스를 보유하고 있으며, 기존의 325,000명 고객 또는 클라우드에 CMS를 배포할 준비가 끝난 새로운 고객을 수용하기 위해 쉽게 확장될 수 있습니다.

실제로 Umbraco의 기술 책임자인 Morten Christensen에 따르면 “UaaS는 현재 매일 약 30명의 새 고객으로 성장을 거듭하고 있습니다. 우리 고객들은 몇 초 안에 새 프로젝트를 편리하게 프로비전하고, ‘원클릭 배포'를 사용하여 개발 환경에서 라이브 사이트로 즉시 업데이트를 게시하고, 오류를 찾는 즉시 변경될 수 있다는 점에 매우 만족하고 있습니다.”

고객에게 두 번째 및/또는 세 번째 환경이 더 이상 필요하지 않은 경우 해당 환경을 간단히 제거할 수 있습니다. 이렇게 하면 Umbraco 탄력적 데이터베이스 가용성 풀의 일부로, 다른 고객이 사용할 수 있는 리소스가 확보됩니다.

![Umbraco 배포 아키텍처](./media/sql-database-implementation-umbraco/figure4.png)

그림 3. Microsoft Azure의 UaaS 배포 아키텍처

## <a name="the-path-from-datacenter-to-cloud"></a>데이터 센터에서 클라우드로의 경로
Umbraco 개발자들은 초기에 SaaS 모델로 전환하기로 결정했을 때 이 서비스를 구축하기 위한 비용 효과적이고 확장 가능한 방안이 필요하다는 것을 알았습니다.

> "탄력적 풀은 필요에 따라 용량을 늘리고 줄일 수 있으므로 당사의 SaaS 제품에 완벽하게 딱 들어맞습니다. 프로비전도 쉬우며, 당사 설정을 통해 최대 사용률을 유지할 수 있습니다."
> 
> - Umbraco기술 책임자, Morten Christensen
> 
> 

"우리는 인프라 관리가 아니라 고객 문제 해결에 더 많은 시간을 투자하기 원했습니다. 우리는 고객이 쉽게 최고의 가치를 얻을 수 있기 원했습니다.”라고 Umbraco의 설립자인 Niels Hartvig는 말했습니다. "우리는 처음에 서버를 직접 호스트하는 것을 고려했으나 용량 계획은 악몽과도 같은 문제가 되었습니다.” 우연히도 Umbraco는 데이터베이스 관리자를 고용하지 않았으며 이러한 방식은 UaaS 사용을 위한 핵심 가치 제안을 부각시켰습니다.

Umbraco 개발자들의 중요한 목표 한 가지는 UaaS 고객에게 용량 제한 없이 환경을 신속하게 프로비전하는 방법을 제공하는 것이었습니다. 하지만 Umbraco 데이터 센터에 전용 호스티드 서비스를 제공하면서 처리 폭주를 처리하기 위해 과도한 용량이 필요하게 되었습니다. 이것은 정기적으로는 활용도가 낮은 상당량의 계산 인프라를 추가하는 것을 의미하는 것이었습니다.

또한 Umbraco 개발 팀은 기존 코드를 가능한 많이 재사용할 수 있는 솔루션을 원했습니다. Umbraco의 개발자인 Mikkel Madsen은 다음과 같이 말했습니다. "우리는 이미 친숙한 Microsoft SQL Server, Microsoft Azure SQL Database, ASP.net 및 IIS(인터넷 정보 서비스)와 같은 Microsoft 개발 도구에 만족하고 있었습니다. IaaS 또는 PaaS 클라우드 솔루션에 투자하기 전에 Microsoft 도구와 플랫폼을 지원하는지 확인하기 원했으므로 코드베이스를 대폭 변경할 필요가 없었습니다."

모든 조건을 충족하기 위해 Umbraco는 다음과 같은 자격이 있는 클라우드 파트너를 찾았습니다.

* 충분한 용량 및 안정성
* Microsoft 개발 도구에 대한 지원(따라서 Umbraco 엔지니어들은 개발 환경을 완전히 재구축할 필요가 없음)
* UaaS가 경쟁하는 모든 지역 시장에 서비스 제공(기업에서는 해당 데이터에 빠르게 액세스할 수 있는지와 데이터가 해당 지역 규정 요구 사항을 충족하는 위치에 저장되어 있는지를 확인해야 함)

## <a name="why-umbraco-chose-azure-for-uaas"></a>Umbraco가 UaaS를 위해 Azure를 선택한 이유
Morten Christensen에 따르면 "우리는 모든 옵션을 고려한 후에 Azure가 관리 효율성 및 확장성부터 친숙성 및 비용 효율성 측면에서 우리의 모든 조건에 부합되기 때문에 Azure를 선택했습니다. Azure VM에서 환경을 설정했으며 각 환경은 자체 Azure SQL Database 인스턴스를 갖습니다. 또한 이러한 인스턴스는 모두 탄력적 풀에 포함되어 있습니다. 개발, 스테이징 및 라이브 환경 간에 데이터베이스를 분리하여 고객에 게 규모 조정에 적합한 성능 격리를 제공할 수 있습니다. 이것은 뛰어난 장점입니다.”

"이전에는 웹 데이터베이스에 대한 서버를 수동으로 프로비전해야 했습니다. 이제는 이러한 문제에 대해 생각할 필요가 없습니다. 프로 비전부터 정리에 이르는 모든 것이 자동화됩니다.”라고 Morten은 덧붙였습니다.

Morten은 Azure에서 제공하는 크기 조정 기능에도 매우 만족했습니다. "탄력적 풀은 필요에 따라 용량을 늘리고 줄일 수 있으므로 당사의 SaaS 제품에 완벽하게 딱 들어맞습니다. 프로비전도 쉬우며, 당사 설정을 통해 최대 사용률을 유지할 수 있습니다." "서비스 계층 기반 DTU에 대한 보장과 탄력적 풀의 간편성은 필요할 때 새 리소스 풀을 프로비전할 수 있도록 지원합니다. 최근 큰 고객 중 하나가 라이브 환경에서 100DTU까지 사용한 적이 있었습니다. Azure를 사용하면 DTU 요구 수준을 예측할 필요 없이 탄력적 풀이 실시간으로 필요한 리소스를 고객 데이터베이스에 제공했습니다. 간단히 말해 고객에게는 예상하는 소요 시간이 확보되고, 우리는 성능 서비스 수준 계약을 충족할 수 있습니다.

"우리는 일반적인 SaaS 시나리오(새 고객을 실시간으로 대규모로 온보딩)를 기본 기술(Azure SQL Database와 함께 Azure Service Bus 큐 사용) 위의 응용 프로그램 패턴(개발 및 라이브에서 데이터베이스 사전 프로비전)에 연결하는 강력한 Azure 알고리즘을 수용했습니다.”라고 Mikkel Madsen은 결론지었습니다.

## <a name="with-azure-uaas-is-exceeding-customer-expectations"></a>Azure를 사용하여 UaaS에서 고객의 기대치 부응
Azure를 클라우드 파트너로 선택한 이후부터 Umbraco는 셀프 호스트드 솔루션에 필요한 IT 리소스 투자 없이도, 최적화된 콘텐츠 관리 성능을 UaaS 고객에게 제공할 수 있게 되었습니다. "우리는 Azure가 우리에게 주는 개발자 편리함 및 확장성에 크게 만족하며, 우리 고객들은 이러한 기능과 안정성에 감탄했습니다. 전반적으로는 우리는 큰 혜택을 얻게 되었습니다.”라고 Norten은 말했습니다.

## <a name="more-information"></a>자세한 정보
* Azure의 탄력적 풀에 대한 자세한 내용은 [탄력적 풀](sql-database-elastic-pool.md)을 참조하세요.
* Azure Service Bus에 대한 자세한 내용은 [Azure Service Bus](https://azure.microsoft.com/services/service-bus/)를 참조하세요.
* 가상 네트워킹에 대해 자세히 알아보려면 [가상 네트워킹](https://azure.microsoft.com/documentation/services/virtual-network/)을 참조하세요.    
* 백업 및 복구에 대한 자세한 내용은 [비즈니스 연속성](sql-database-business-continuity.md)을 참조하세요.    
* 모니터링 풀에 대한 자세한 내용은 [모니터링 풀](sql-database-elastic-pool-manage-portal.md)을 참조하세요.    
* 서비스로서의 Umbraco에 대한 자세한 내용은 [Umbraco](https://umbraco.com/cloud)를 참조하세요.

