---
title: "Azure Site Recovery에서 장애 조치 및 복구의 복구 계획 만들기 및 사용자 지정 | Microsoft Docs"
description: "Azure Site Recovery에서 복구 계획을 만들고 사용자 지정하여 VM 및 물리적 서버를 장애 조치(failover)하고 복구하는 방법을 설명합니다."
services: site-recovery
documentationcenter: 
author: rayne-wiselman
manager: carmonm
editor: 
ms.assetid: 72408c62-fcb6-4ee2-8ff5-cab1218773f2
ms.service: site-recovery
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: storage-backup-recovery
ms.date: 01/26/2018
ms.author: raynew
ms.openlocfilehash: 9839a989246b28c1a194b8d1f0e99c1bd80ac2e5
ms.sourcegitcommit: ded74961ef7d1df2ef8ffbcd13eeea0f4aaa3219
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/29/2018
---
# <a name="create-recovery-plans"></a>복구 계획 만들기


이 문서는 [Azure Site Recovery](site-recovery-overview.md)에서 복구 계획을 만들고 사용자 지정하는 정보를 제공합니다.

이 문서의 하단 또는 [Azure Recovery Services 포럼](https://social.msdn.microsoft.com/forums/azure/home?forum=hypervrecovmgr)에서 의견이나 질문을 게시합니다.

 다음을 수행하는 복구 계획을 만듭니다.

* 장애 조치(failover)된 다음 함께 시작하는 컴퓨터의 그룹을 정의합니다.
* 복구 계획 그룹에서 함께 그룹화하여 컴퓨터 간의 종속성을 모델링합니다. 예를 들어, 특정 응용프로그램을 장애 조치하고 표시하기 위해 동일한 복구 계획 그룹에서 해당 응용 프로그램에 대한 VM을 그룹화합니다.
* 장애 조치(Failover)를 실행합니다. 복구 계획에서 테스트를 실행하고 계획되거나 계획되지 않은 장애 조치를 실행할 수 있습니다.

## <a name="why-use-recovery-plans"></a>복구 계획을 사용하는 이유

복구 계획은 관리할 수 있는 독립적인 작은 단위를 만들어서 체계적인 복구 프로세스를 계획하는 데 도움이 됩니다. 이러한 단위는 일반적으로 환경의 응용 프로그램을 나타냅니다. 복구 계획을 사용하면 가상 머신이 시작되는 순서를 정의할 수 있을 뿐만 아니라 복구 중 일반적인 작업을 자동화하는 데 도움이 됩니다.


**기본적으로 클라우드 마이그레이션 또는 재해 복구에 대한 준비가 되었는지 확인하는 한 가지 방법은 사용자의 모든 응용 프로그램이 복구 계획에 속하는지 확인하고 각 복구 계획이 Microsoft Azure로 복구되는지 테스트하는 것입니다. 이러한 준비를 통해 전체 데이터 센터를 Microsoft Azure로 확실하게 마이그레이션하거나 장애 조치(failover)할 수 있습니다.**
 
다음은 복구 계획의 세 가지 핵심 가치 제안입니다.

### <a name="model-an-application-to-capture-dependencies"></a>종속성을 캡처하도록 응용 프로그램 모델링

복구 계획은 일반적으로 함께 장애 조치(failover)되는 응용 프로그램으로 구성된 가상 머신 그룹입니다. 복구 계획 구문을 사용하면 응용 프로그램별 속성을 캡처하도록 이러한 그룹을 향상시킬 수 있습니다.
 
다음은 전형적인 3계층 응용 프로그램의 예입니다.

* SQL 백 엔드 하나
* 미들웨어 하나
* 웹 프런트 엔드 하나

장애 조치(failover) 후 가상 머신이 올바른 순서로 작동하도록 복구 계획을 사용자 지정할 수 있습니다. SQL 백엔드가 먼저 작동되고, 다음으로 미들웨어가 작동되어야 하며, 웹 프런트 엔드가 마지막으로 작동되어야 합니다. 이러한 순서를 통해 마지막 가상 머신이 작동하면 응용 프로그램이 작동하도록 합니다. 예를 들어 미들웨어가 시작되면 SQL 계층에 연결을 시도하며, SQL 계층은 복구 계획에 따라 이미 실행되고 있습니다. 프런트 엔드 서버가 마지막으로 작동하기 때문에 모든 구성 요소가 시작되어 실행 중이고 응용 프로그램이 요청을 수락할 준비가 될 때까지 최종 사용자가 실수로 응용 프로그램 URL에 연결하지 않도록 합니다. 이러한 종속성을 생성하기 위해 그룹을 추가하도록 복구 계획을 사용자 지정할 수 있습니다. 그런 다음, 가상 머신을 선택하고 해당 그룹을 변경하여 그룹 간에 이동합니다.

![샘플 복구 계획](./media/site-recovery-create-recovery-plans/rp.png)

사용자 지정을 완료하면 복구의 정확한 단계를 시각화할 수 있습니다. 다음은 복구 계획의 장애 조치 중 실행되는 단계의 순서입니다.

* 먼저 온-프레미스 가상 머신을 끄려고 시도하는 종료 단계가 있습니다(기본 사이트를 계속 실행해야 하는 테스트 장애 조치(failover) 제외).
* 다음으로 복구 계획의 모든 가상 머신에 대한 장애 조치(failover)를 병렬로 트리거합니다. 장애 조치(failover) 단계에서는 복제된 데이터에서 가상 머신의 디스크를 준비합니다.
* 마지막으로 시작 그룹이 순서대로 실행되어 각 그룹(그룹 1, 그룹 2, 그룹 3 순서로)의 가상 머신이 시작됩니다. 그룹에 가상 머신이 둘 이상이면(예: 부하가 분산된 웹 프런트 엔드) 모든 가상 머신이 병렬로 부팅됩니다.

**그룹 전체에서 시퀀싱하면 다양한 응용 프로그램 계층 사이의 종속성이 유지되고 적절한 경우병렬 처리가 보장되기 때문에 응용 프로그램 복구의 RTO가 향상됩니다.**

   > [!NOTE]
   > 단일 그룹에 속하는 시스템은 병렬로 장애 조치(failover)됩니다. 다른 그룹에 속하는 시스템은 그룹 순서대로 장애 조치(failover)됩니다. 그룹 1의 모든 시스템이 장애 조치되고 부팅된 후에야 그룹 2의 시스템이 장애 조치(failover)를 시작합니다.

### <a name="automate-most-recovery-tasks-to-reduce-rto"></a>RTO를 줄이기 위해 대부분의 복구 작업 자동화

대규모 응용 프로그램 복구는 복잡한 작업일 수 있습니다. 장애 조치(failover) 또는 마이그레이션 후 정확한 사용자 지정 단계를 기억하는 것도 어렵습니다. 때로는 응용 프로그램의 복잡한 사항을 인식하지 못하는 다른 사람이 장애 조치를 트리거해야 하는 경우도 있습니다. 혼돈의 시기에 너무 많은 수동 단계를 기억하는 것은 어렵고 오류가 발생하기 쉽습니다. 복구 계획을 사용하면 Microsoft Azure Automation Runbook을 사용하여 모든 단계마다 수행해야 하는 필수 작업을 자동화하는 방법이 있습니다. Runbook을 사용하면 아래의 예제와 같은 일반적인 복구 작업을 자동화할 수 있습니다. 자동화할 수 없는 작업에 대해서는 복구 계획을 통해 수동 작업을 삽입할 수 있는 기능도 있습니다.

* 장애 조치(failover) 후 Azure 가상 머신에 대한 작업 – 가상 머신에 연결할 수 있도록 하기 위해 필요합니다. 예를 들면 다음과 같은 작업이 있습니다.
    * 장애 조치(failover) 후 가상 머신에 공용 IP 만들기
    * 장애 조치된 가상 머신의 NIC에 NSG 할당
    * 가용성 집합에 부하 분산 장치 추가
* 장애 조치 후 가상 머신 내 작업 – 새로운 환경에서 응용 프로그램이 올바르게 작동하도록 응용 프로그램을 재구성합니다. 예를 들면:
    * 가상 머신 내 데이터베이스 연결 문자열 수정
    * 웹 서버 구성/규칙 변경

**자동화 runbook을 사용하여 복구 후 작업을 자동화하는 완벽한 복구 계획을 통해 한 번의 클릭으로 장애 조치를 수행하고 RTO를 최적화할 수 있습니다.**

### <a name="test-failover-to-be-ready-for-a-disaster"></a>재해에 대비하기 위한 테스트 장애 조치(failover)

복구 계획을 사용하여 장애 조치 또는 테스트 장애 조치를 트리거할 수 있습니다. 장애 조치를 수행하기 전에 항상 응용 프로그램에서 테스트 장애 조치를 완료해야 합니다. 테스트 장애 조치는 응용 프로그램이 복구 사이트에서 작동되는지 여부를 확인하는 데 도움이 됩니다.  놓친 부분이 있으면 쉽게 정리를 트리거하고 테스트 장애 조치를 다시 실행할 수 있습니다. 응용 프로그램이 원활하게 복구된다는 확신이 들 때까지 테스트 장애 조치를 여러 번 수행합니다.

![복구 계획 테스트](./media/site-recovery-create-recovery-plans/rptest.png)

**각 응용 프로그램이 다르기 때문에 각각에 대해 사용자 지정된 복구 계획을 구축해야 합니다. 또한 동적인 데이터 센터 환경에서는 응용 프로그램과 종속성이 계속 변경됩니다. 응용 프로그램에 장애 조치 테스트를 분기에 한 번씩 수행하여 복구 계획이 최신인지 확인하십시오.**

## <a name="how-to-create-a-recovery-plan"></a>복구 계획을 만드는 방법

1. **복구 계획** > **복구 계획 만들기**를 클릭합니다.
   복구 계획 이름, 원본 및 대상을 지정합니다. 원본 위치에는 장애 조치(Failover) 및 복구를 사용하도록 설정한 가상 머신이 있어야 합니다. 복구 계획에 포함시킬 가상 머신을 기반으로 원본 및 대상을 선택합니다. 

   |시나리오                   |원본               |대상           |
   |---------------------------|---------------------|-----------------|
   |Azure 간             |Azure 지역         |Azure 지역     |
   |VMware에서 Azure로            |구성 서버 |Azure            |
   |VMM에서 Azure로               |VMM 식별 이름    |Azure            |
   |Hyper-v 사이트에서 Azure로      |Hyper-v 사이트 이름    |Azure            |
   |물리적 컴퓨터에서 Azure로 |구성 서버 |Azure            |
   |VMM에서 VMM으로                 |VMM 식별 이름    |VMM 식별 이름|

   > [!NOTE]
   > 복구 계획에는 원본 및 대상이 동일한 가상 머신이 포함될 수 있습니다. VMM 및 VMware의 가상 머신은 동일한 복구 계획에 속할 수 없습니다. 하지만 VMware 가상 머신과 물리적 컴퓨터는 모두 원본과 동일한 계획에 추가될 수 있습니다. 둘 다 구성 서버이기 때문입니다.

2. **가상 머신 선택**에서 복구 계획 내 기본 그룹(그룹 1)에 추가하고자 하는 가상 머신(또는 복제 그룹)를 선택합니다. 원본에서 보호되고(복구 계획에 선택된 대로) 대상으로 보호된(복구 계획에 선택된 대로) 가상 머신만 선택이 허용됩니다.

## <a name="how-to-customize-and-extend-recovery-plans"></a>복구 계획을 사용자 지정하고 확장하는 방법

Site Recovery 복구 계획 리소스 블레이드로 이동하여 사용자 지정 탭을 클릭하면 복구 계획을 사용자 지정하고 확장할 수 있습니다.

복구 계획을 사용자 지정 및 확장할 수 있습니다.

- **새 그룹 추가**—기본 그룹에 추가 복구 계획 그룹(최대 7개)을 추가한 다음 해당 복구 계획 그룹에 컴퓨터나 복제 그룹을 추가합니다. 그룹은 추가한 순서대로 번호가 지정됩니다. 가상 머신이나 복제 그룹은 1개의 복구 계획 그룹에만 포함할 수 있습니다.
- **수동 작업 추가**—복구 계획 그룹의 앞 또는 뒤에 실행되는 수동 작업을 추가할 수 있습니다. 복구 계획을 실행하는 경우 수동 작업을 삽입하는 지점에서 중지됩니다. 대화 상자는 수동 작업이 완료되도록 지정하라는 메시지를 표시합니다.
- **스크립트 추가**—복구 계획 그룹 앞뒤로 실행되는 스크립트를 추가할 수 있습니다. 스크립트를 추가하면 해당 그룹에 대해 새로운 작업 집합이 추가됩니다. 예를 들어, 그룹 1에 대한 사전 단계 집합이 Group 1: Pre-steps라는 이름으로 생성됩니다. 모든 사전 단계가 이 집합 내에 나열됩니다. 배포된 VMM 서버를 보유한 경우 주 사이트에만 스크립트를 추가할 수 있습니다. [자세히 알아보기](site-recovery-how-to-add-vmmscript.md).
- **Azure runbook 추가**—Azure runbook을 사용하여 복구 계획을 확장할 수 있습니다. 예를 들어, 작업을 자동화하거나 단일 단계 복구를 만듭니다. [자세히 알아보기](site-recovery-runbook-automation.md).


## <a name="how-to-add-a-script-runbook-or-manual-action-to-a-plan"></a>계획에 스크립트, Runbook 또는 수동 작업을 추가하는 방법

기본 복구 계획 그룹에 VM이나 복제 그룹을 추가하고 계획을 만든 후에 스크립트나 수동 작업을 추가할 수 있습니다.

1. 복구 계획을 엽니다.
2. **단계** 목록에서 아무 항목이나 클릭하고 **스크립트** 또는 **수동 작업**을 클릭합니다.
3. 스크립트나 작업을 선택한 항목의 앞 또는 뒤에 추가할 것인지 지정합니다. 스크립트 위치를 위 또는 아래로 이동하려면 **위로 이동** 및 **아래로 이동** 단추를 사용합니다.
4. VMM 스크립트를 추가하면 **VMM 스크립트에 대한 장애 조치**를 선택합니다. **스크립트 경로**에서 공유에 상대 경로를 입력합니다. 아래의 VMM 예제에서 **\RPScripts\RPScript.PS1** 경로를 지정합니다.
5. Azure Automation Runbook을 추가하는 경우 Runbook이 위치한 Azure Automation 계정을 지정하고 적절한 Azure Runbook 스크립트를 선택합니다.
6. 스크립트가 예상대로 작동하는지 확인하려면 복구 계획의 장애 조치를 수행합니다.

스크립트나 Runbook 옵션은 장애 조치(failover) 또는 장애 복구(failback)를 수행할 때 다음 시나리오에서만 사용할 수 있습니다. 장애 조치(failover)와 장애 복구(failback) 모두에서 수동 작업이 가능합니다.


|시나리오               |장애 조치(failover) |장애 복구 |
|-----------------------|---------|---------|
|Azure 간         |Runbook |Runbook  |
|VMware에서 Azure로        |Runbook |해당 없음       | 
|VMM에서 Azure로           |Runbook |스크립트   |
|Hyper-v 사이트에서 Azure로  |Runbook |해당 없음       |
|VMM에서 VMM으로             |스크립트   |스크립트   |


## <a name="next-steps"></a>다음 단계

장애 조치를 실행하는 방법에 대해 [자세히 알아보세요](site-recovery-failover.md).

다음 동영상에서 실제 복구 계획을 확인하세요.

> [!VIDEO https://channel9.msdn.com/Series/Azure-Site-Recovery/One-click-failover-of-a-2-tier-WordPress-application-using-Azure-Site-Recovery/player]
