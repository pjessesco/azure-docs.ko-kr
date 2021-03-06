---
title: "Azure Active Directory Domain Services: 서비스 주체 구성 문제 해결 | Microsoft Docs"
description: "Azure AD Domain Services에 대한 서비스 주체 구성 문제 해결"
services: active-directory-ds
documentationcenter: 
author: eringreenlee
manager: 
editor: 
ms.assetid: f168870c-b43a-4dd6-a13f-5cfadc5edf2c
ms.service: active-directory-ds
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/17/2018
ms.author: ergreenl
ms.openlocfilehash: e6144a4018f7fbe7dbf7b4693e3f41884e4a80a2
ms.sourcegitcommit: f1c1789f2f2502d683afaf5a2f46cc548c0dea50
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/18/2018
---
# <a name="azure-ad-domain-services---troubleshooting-service-principal-configuration"></a>Azure AD Domain Services - 서비스 주체 구성 문제 해결

도메인을 서비스, 관리 및 업데이트하기 위해 Microsoft에서는 다양한 [서비스 주체](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-application-objects)를 사용하여 디렉터리와 통신합니다. 하나라도 잘못 구성되거나 삭제되면 서비스가 중단될 수 있습니다.

## <a name="aadds102-service-principal-not-found"></a>AADDS102: 서비스 주체를 찾을 수 없음

**경고 메시지:**

*Azure AD Domain Services가 제대로 작동하는 데 필요한 서비스 주체가 Azure AD 디렉터리에서 삭제되었습니다. 이 구성은 관리되는 도메인을 모니터링, 관리, 패치, 동기화하는 Microsoft의 기능에 영향을 줍니다.*

서비스 주체는 Microsoft에서 관리되는 도메인을 관리, 업데이트 및 유지 관리하는 데 사용하는 응용 프로그램입니다. 이러한 서비스 주체가 삭제되면 Microsoft에서 도메인을 서비스하지 못하게 됩니다. 다음 단계에 따라 다시 만들어야 하는 서비스 주체를 확인합니다.

1. Azure Portal에서 [엔터프라이즈 응용 프로그램 - 모든 응용 프로그램](https://portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/AllApps) 페이지로 이동합니다.
2. 표시 드롭다운을 사용하여 **모든 응용 프로그램** 및 **적용**을 선택합니다.
3. 다음 표를 사용하여 검색 상자에 ID를 붙여 넣어 Enter 키를 눌러 각 응용 프로그램 ID를 검색합니다. 검색 결과가 비어 있으면 "해결 방법" 열에 나와 있는 단계를 수행하여 서비스 주체를 다시 만들어야 합니다.

| 응용 프로그램 UI | 해결 방법 |
| :--- | :--- | :--- |
| 2565bd9d-da50-47d4-8b85-4c97f669dc36 | [PowerShell에서 누락된 서비스 주체 다시 만들기](#recreate-a-missing-service-principal-with-powershell) |
| 443155a6-77f3-45e3-882b-22b3a8d431fb | [Microsoft.AAD 네임스페이스에 다시 등록](#re-register-to-the-microsoft-aad-namespace-using-the-azure-portal) |
| abba844e-bc0e-44b0-947a-dc74e5d09022  | [Microsoft.AAD 네임스페이스에 다시 등록](#re-register-to-the-microsoft-aad-namespace-using-the-azure-portal) |
| d87dcbc6-a371-462e-88e3-28ad15ec4e64 | [자체 수정하는 서비스 주체](#service-principals-that-self-correct) |

### <a name="recreate-a-missing-service-principal-with-powershell"></a>PowerShell 사용하여 누락된 서비스 주체 다시 만들기

*누락된 ID에 대해 다음 단계를 수행합니다. 2565bd9d-da50-47d4-8b85-4c97f669dc36*

**재구성:**

이 단계를 완료하려면 Azure AD PowerShell이 있어야 합니다. Azure AD PowerShell을 설치하는 방법에 대한 내용은 [이 문서](https://docs.microsoft.com/en-us/powershell/azure/active-directory/install-adv2?view=azureadps-2.0.)를 참조하세요.

이 문제를 해결하려면 PowerShell 창에서 다음 명령을 입력합니다.
1. Install-Module AzureAD
2. Import-Module AzureAD
3. 다음 PowerShell 명령을 실행하여 Azure AD Domain Services에 필요한 서비스 주체가 디렉터리에서 누락되어 있는지 여부를 확인합니다.
      ```PowerShell
      Get-AzureAdServicePrincipal -filter "AppId eq '2565bd9d-da50-47d4-8b85-4c97f669dc36'"
      ```
4. 다음 PowerShell 명령을 입력하여 서비스 주체를 만듭니다.
     ```PowerShell
     New-AzureAdServicePrincipal -AppId "2565bd9d-da50-47d4-8b85-4c97f669dc36"
     ```
5. 누락된 서비스 주체를 만든 후 2시간 동안 기다렸다가 도메인의 상태를 확인합니다.


### <a name="re-register-to-the-microsoft-aad-namespace-using-the-azure-portal"></a>Azure Portal을 사용하여 Microsoft AAD 네임스페이스에 다시 등록

*누락된 ID에 대해 다음 단계를 수행합니다. 443155a6-77f3-45e3-882b-22b3a8d431fb 및 abba844e-bc0e-44b0-947a-dc74e5d09022*

**재구성:**

다음 단계를 사용하여 디렉터리에서 Domain Services를 복원합니다.

1. Azure Portal의 [구독](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade) 페이지로 이동합니다.
2. 관리되는 도메인에 연결된 구독을 테이블에서 선택합니다.
3. 왼쪽 탐색 영역에서 **리소스 공급자**를 선택합니다.
4. 테이블에서 "Microsoft.AAD"를 검색하고 **다시 등록**을 클릭합니다.
5. 경고가 해결되었는지 확인하려면 2시간 후에 상태 경고 페이지를 확인합니다.


### <a name="service-principals-that-self-correct"></a>자체를 수정하는 서비스 주체

*누락된 ID에 대해 다음 단계를 수행합니다. d87dcbc6-a371-462e-88e3-28ad15ec4e64*

**재구성:**

Microsoft에서는 특정 서비스 주체가 누락되었거나, 잘못 구성되었거나, 삭제된 경우를 확인할 수 있습니다. 서비스를 신속하게 수정하기 위해 Microsoft에서는 서비스 주체를 직접 다시 만듭니다. 2시간 후에 도메인 상태를 확인하여 주체가 다시 만들어졌는지 검토합니다.

## <a name="contact-us"></a>문의처
[지원이 필요하거나 피드백을 공유하려면](active-directory-ds-contact-us.md)Azure Active Directory Domain Services 제품 팀에 문의하세요.
