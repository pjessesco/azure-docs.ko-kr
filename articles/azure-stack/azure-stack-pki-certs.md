---
title: "Azure 스택에 대 한 azure 스택 공개 키 인프라 인증서 요구 사항 통합 시스템 | Microsoft Docs"
description: "Azure 스택 통합 시스템에 대 한 Azure 스택 PKI 인증서 배포 요구 사항에 설명합니다."
services: azure-stack
documentationcenter: 
author: jeffgilb
manager: femila
editor: 
ms.assetid: 
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/31/2018
ms.author: jeffgilb
ms.reviewer: ppacent
ms.openlocfilehash: d96e2e6767ca01c8c16403a8846e3ab9d16796bc
ms.sourcegitcommit: d87b039e13a5f8df1ee9d82a727e6bc04715c341
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/21/2018
---
# <a name="azure-stack-public-key-infrastructure-certificate-requirements"></a>Azure 스택 공개 키 인프라에 대 한 인증서 요구 사항
Azure 스택 소수의 Azure 스택 서비스 및 테 넌 트 Vm에 할당 된 외부에서 액세스할 수 있는 공용 IP 주소를 사용 하는 공용 인프라 네트워크를 있습니다. Azure 스택 배포 하는 동안 이러한 Azure 스택 공개 인프라 끝점에 대 한 적절 한 DNS 이름으로 PKI 인증서가 필요 합니다. 이 문서에 대 한 정보를 제공합니다.

- 어떤 인증서는 배포 Azure 스택
- 이러한 사양 일치 하는 인증서를 가져오는 과정
- 준비 확인 하 고 배포 하는 동안 해당 인증서를 사용 하는 방법
> [!NOTE]
> 배포 하는 동안 인증서 (Azure AD 또는 AD FS)에 대 한 배포 하는 id 공급자와 일치 하는 배포 폴더에 복사 해야 합니다. 모든 끝점에 대 한 단일 인증서를 사용 하는 경우에 아래 표에 설명 된 대로 각 배포 폴더에 해당 인증서 파일을 복사 해야 합니다. 폴더 구조는 미리 배포 가상 컴퓨터에서 만들어지고에서 찾을 수 있습니다: C:\CloudDeployment\Setup\Certificates 합니다. 

## <a name="certificate-requirements"></a>인증서 요구 사항
다음 목록에서는 Azure 스택을 배포 하는 데 필요한 인증서 요구 사항을 설명 합니다. 
- 인증서는 내부 인증 기관 또는 공용 인증 기관에서 발급 되어야 합니다. 공용 인증 기관 사용 되는 경우 Microsoft 신뢰할 수 있는 루트 인증 기관 프로그램의 일부로 기본 운영 체제 이미지에 포함 되어야 합니다. 전체 목록은 여기를 찾을 수 있습니다: https://gallery.technet.microsoft.com/Trusted-Root-Certificate-123665ca 
- 인증서 이름 SAN (주체 대체) 필드에서 모든 네임 스페이스를 포함 하는 단일 와일드 카드 인증서를 수 있습니다. 또는 와일드 카드를 사용 하 여 acs는 필요한 키 자격 증명 모음 등 끝점에 대 한 개별 인증서를 사용할 수 있습니다. 
- 인증서 서명 알고리즘 보다 강력한 이루어야 SHA1, 일 수 없습니다. 
- Azure 스택 설치에 필요한을 공개 및 개인 키 인증서 형식이 PFX를 해야 합니다. 
- 인증서 pfx 파일의 "키 사용" 필드에 값 "디지털 서명을" 및 "KeyEncipherment" 있어야 합니다.
- 모든 인증서 pfx 파일에 암호가 동일 해야 배포 시
- 주체 이름 및 모든 인증서의 주체 대체 이름을 실패 한 배포를 방지 하기 위해이 문서에 설명 된 사양은 일치 하는지 확인 합니다.

> [!NOTE]
> 중간 인증 기관에서 인증서의 신뢰 체인 IS 있으면 지원 됩니다. 

## <a name="mandatory-certificates"></a>필수 인증서
이 섹션의 표는 모두 Azure AD에 대 한 필요한 Azure 스택 공용 끝점 PKI 인증서에 설명 하 고 AD FS Azure 스택 배포 합니다. 인증서 요구 사항으로 영역에서 사용 되는 네임 스페이스 별로 그룹화 되어 및 각 네임 스페이스에 대 한 요구 되는 인증서입니다. 또한 테이블 솔루션 공급자는 공용 끝점 마다 다른 인증서를 복사 하는 폴더를 설명 합니다. 

각 Azure 스택 공개 인프라 끝점에 대 한 적절 한 DNS 이름 가진 인증서가 필요 합니다. 각 끝점의 DNS 이름 형식으로 표현 됩니다:  *&lt;접두사 >.&lt; 지역 > 합니다. &lt;fqdn >*합니다. 

배포 [region]에 [externalfqdn] 값 영역과 Azure 스택 시스템에 대해 선택한 외부 도메인 이름을 일치 해야 합니다. 예를 들어 영역 이름이 경우 *Redmond* 외부 도메인 이름 되었으며 *contoso.com*, DNS 이름 형식을 갖기  *&lt;접두사 >. redmond.contoso.com* .  *&lt;접두사 >* 값은 인증서로 보호 하는 끝점을 설명 하기 위해 Microsoft에서 폴더도 있습니다. 또한는  *&lt;접두사 >* 값 외부 인프라 끝점의 특정 끝점을 사용 하는 Azure 스택을 서비스에 따라 다릅니다. 

|배포 폴더|필요한 인증서 주체 및 주체 대체 이름 (SAN)|범위 (지역) 당|하위 도메인 네임 스페이스|
|-----|-----|-----|-----|
|공용 포털|portal.*&lt;region>.&lt;fqdn>*|포털|*&lt;region>.&lt;fqdn>*|
|관리 포털|adminportal.*&lt;region>.&lt;fqdn>*|포털|*&lt;region>.&lt;fqdn>*|
|Azure Resource Manager Public|management.*&lt;region>.&lt;fqdn>*|Azure 리소스 관리자|*&lt;region>.&lt;fqdn>*|
|Azure 리소스 관리자 관리|adminmanagement.*&lt;region>.&lt;fqdn>*|Azure 리소스 관리자|*&lt;region>.&lt;fqdn>*|
|ACS<sup>1</sup>|다중 하위 도메인 와일드 카드 인증서가 두 개에 대 한 주체 대체 이름:<br>&#42;.blob.*&lt;region>.&lt;fqdn>*<br>&#42;.queue.*&lt;region>.&lt;fqdn>*<br>&#42;.table.*&lt;region>.&lt;fqdn>*|Storage|blob.*&lt;region>.&lt;fqdn>*<br>table.*&lt;region>.&lt;fqdn>*<br>queue.*&lt;region>.&lt;fqdn>*|
|KeyVault|&#42;.vault.*&lt;region>.&lt;fqdn>*<br>(와일드 카드 SSL 인증서 포함)|Key Vault|vault.*&lt;region>.&lt;fqdn>*|
|KeyVaultInternal|&#42;.adminvault.*&lt;region>.&lt;fqdn>*<br>(와일드 카드 SSL 인증서 포함)|내부 Keyvault|adminvault.*&lt;region>.&lt;fqdn>*|
|
<sup>1</sup> ACS 인증서에는 단일 인증서에 와일드 카드 San 3 개 필요 합니다. 모든 공용 인증 기관 단일 인증서에 San 여러 와일드 카드를 지원 하지 수도 있습니다. 

Azure 스택 Azure AD 배포 모드를 사용 하 여 배포 하는 경우 앞의 표에 나열 된 인증서를 요청 하기만 하면 됩니다. 그러나 Azure 스택 AD FS 배포 모드를 사용 하 여 배포 하는 경우 다음 표에 설명 된 인증서 요청 해야 합니다.

|배포 폴더|필요한 인증서 주체 및 주체 대체 이름 (SAN)|범위 (지역) 당|하위 도메인 네임 스페이스|
|-----|-----|-----|-----|
|ADFS|adfs.*&lt;region>.&lt;fqdn>*<br>(SSL 인증서 포함)|ADFS|*&lt;region>.&lt;fqdn>*|
|그래프|graph.*&lt;region>.&lt;fqdn>*<br>(SSL 인증서 포함)|그래프|*&lt;region>.&lt;fqdn>*|
|

> [!IMPORTANT]
> 이 섹션에 나열 된 모든 인증서에 동일한 암호가 있어야 합니다. 

## <a name="optional-paas-certificates"></a>옵션 PaaS 인증서
계획 중인 경우 Azure 스택 후 추가 Azure 스택 PaaS 서비스 (SQL, MySQL 및 앱 서비스) 배포를 배포 및 구성 된, PaaS 서비스의 끝점을 포함 하는 추가 인증서를 요청 해야 합니다. 

> [!IMPORTANT]
> 앱 서비스, SQL 및 MySQL 리소스 공급자에 대 한 사용 하는 인증서는 글로벌 Azure 스택 끝점에 사용 되는 것과 동일한 루트 기관 해야 합니다. 

다음 표에서 필요한 SQL 및 MySQL 어댑터 및 응용 프로그램 서비스에 대 한 인증서 및 끝점을 설명 합니다. Azure 스택 배포 폴더에 이러한 인증서를 복사할 필요가 없습니다. 대신 추가 리소스 공급자를 설치 하는 경우 이러한 인증서 제공 합니다. 

|범위 (지역) 당|인증서|필요한 인증서 주체 및 주체 대체 이름 (San)|하위 도메인 네임 스페이스|
|-----|-----|-----|-----|
|SQL, MySQL|SQL 및 MySQL|&#42;.dbadapter.*&lt;region>.&lt;fqdn>*<br>(와일드 카드 SSL 인증서 포함)|dbadapter.*&lt;region>.&lt;fqdn>*|
|App Service|웹 트래픽 기본 SSL 인증서|&#42;.appservice.*&lt;region>.&lt;fqdn>*<br>&#42;.scm.appservice.*&lt;region>.&lt;fqdn>*<br>(다중 도메인 와일드 카드 SSL 인증서<sup>1</sup>)|appservice.*&lt;region>.&lt;fqdn>*<br>scm.appservice.*&lt;region>.&lt;fqdn>*|
|App Service|API|api.appservice.*&lt;region>.&lt;fqdn>*<br>(SSL 인증서<sup>2</sup>)|appservice.*&lt;region>.&lt;fqdn>*<br>scm.appservice.*&lt;region>.&lt;fqdn>*|
|App Service|FTP|ftp.appservice.*&lt;region>.&lt;fqdn>*<br>(SSL 인증서<sup>2</sup>)|appservice.*&lt;region>.&lt;fqdn>*<br>scm.appservice.*&lt;region>.&lt;fqdn>*|
|App Service|SSO|sso.appservice.*&lt;region>.&lt;fqdn>*<br>(SSL 인증서<sup>2</sup>)|appservice.*&lt;region>.&lt;fqdn>*<br>scm.appservice.*&lt;region>.&lt;fqdn>*|

<sup>1</sup> 여러 와일드 카드 주체 대체 이름으로 하나의 인증서가 필요 합니다. 단일 인증서에 San 여러 와일드 카드 모든 공용 인증 기관에서 지원 되지 않는 경우 

<sup>2</sup> &#42;. 앱 서비스입니다.  *&lt;지역 >.&lt; fqdn >* 와일드 카드 인증서는 이러한 3 개의 인증서를 대신 사용할 수 없습니다 (api.appservice. *&lt;지역 > 합니다. &lt;fqdn >*, ftp.appservice. *&lt;지역 > 합니다. &lt;fqdn >*, 및 sso.appservice. *&lt;지역 > 합니다. &lt;fqdn >*합니다. 앱 서비스는 이러한 끝점에 대 한 별도 인증서를 사용 하도록 명시적으로 필요합니다. 

## <a name="learn-more"></a>자세한 정보
자세한 방법 [Azure 스택 배포를 위한 PKI 인증서를 생성](azure-stack-get-pki-certs.md)합니다. 

## <a name="next-steps"></a>다음 단계
[Id 통합](azure-stack-integrate-identity.md)

