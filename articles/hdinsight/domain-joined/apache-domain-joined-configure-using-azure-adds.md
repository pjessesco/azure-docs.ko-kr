---
title: "Azure Active Directory Domain Services를 사용하여 도메인에 가입된 HDInsight 클러스터 구성- Azure | Microsoft Docs"
description: "Azure Active Directory Domain Services를 사용하여 도메인에 가입된 HDInsight 클러스터를 설정 및 구성하는 방법을 알아봅니다."
services: hdinsight
documentationcenter: 
author: bprakash
manager: jhubbard
editor: cgronlun
tags: 
ms.service: hdinsight
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 11/10/2017
ms.author: bhanupr
ms.openlocfilehash: 77478616eae27828a57a36dc0aaf3884e80ce403
ms.sourcegitcommit: 28178ca0364e498318e2630f51ba6158e4a09a89
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/24/2018
---
# <a name="configure-domain-joined-hdinsight-clusters-using-azure-active-directory-domain-services"></a>Azure Active Directory Domain Services를 사용하여 도메인에 가입된 HDInsight 클러스터 구성

도메인에 가입된 클러스터는 HDInsight에서 다중 사용자 엔터프라이즈 보안 기능을 제공합니다. 도메인에 가입된 HDInsight 클러스터는 Active Directory 도메인에 연결되기 때문에 도메인 사용자는 자신의 도메인 자격 증명을 사용하여 클러스터를 인증하고 빅 데이터 작업을 실행할 수 있습니다. 

도메인에 가입된 HDInsight 클러스터에 연결할 수 있도록 도메인 컨트롤러를 설정하는 두 가지 방법이 있습니다.

- Azure AD DS(Azure Active Directory Domain Services)
- Azure IaaS VM의 Active Directory 도메인 컨트롤러

이 문서에서는 Azure Active Directory Domain Services를 사용하여 도메인에 가입된 HDInsight 클러스터를 구성하는 방법을 알아봅니다.

## <a name="create-azure-adds"></a>Azure ADDS 만들기

HDInsight 클러스터를 만들려면 먼저 Azure AD DS를 만들어야 합니다. Azure ADDS를 만들려면 [Azure Portal을 사용하여 Azure Active Directory Domain Services 활성화](../../active-directory-domain-services/active-directory-ds-getting-started.md)를 참조하세요. 

> [!NOTE]
> 도메인 서비스를 만들 권한은 테넌트 관리자에게만 있습니다. HDInsight에 대한 기본 저장소로 ADLS(Azure Data Lake Storage)를 사용하는 경우 ADLS에 대한 기본 Azure AD 테넌트가 HDInsight 클러스터에 대한 도메인과 동일한지 확인합니다. 

도메인 서비스를 프로비전한 후에는 **Azure AD DC Administrators** 그룹에 서비스 계정을 만들어 HDInsight 클러스터를 만들어야 합니다. 서비스 계정은 Azure AD의 전역 관리자여야 합니다.

## <a name="create-a-domain-joined-hdinsight-cluster"></a>도메인에 가입된 HDInsight 클러스터 만들기

다음 단계는 이전 섹션에서 만든 AAD DS와 서비스 계정을 사용하여 HDInsight 클러스터를 만드는 것입니다.

Azure AD 도메인 서비스와 HDInsight 클러스터를 동일한 Azure VNet(가상 네트워크)에 배치하는 것이 더 쉽습니다. 다른 VNet에 있는 경우 두 VNet 모두를 피어링해야 합니다. 자세한 내용은 [가상 네트워크 피어링](../../virtual-network/virtual-network-peering-overview.md)을 참조하세요.

도메인에 가입된 HDInsight 클러스터를 만드는 경우 다음 매개 변수를 제공해야 합니다.

- **도메인 이름**: Azure AD DS와 연결된 도메인 이름입니다. 예: contoso.onmicrosoft.com.
- **도메인 사용자 이름**: 이전 섹션에서 만든 Azure AD DC Administrators 그룹의 서비스 계정입니다. 예: hdiadmin@contoso.onmicrosoft.com 이 도메인 사용자는 이 도메인에 가입된 HDInsight 클러스터의 관리자입니다.
- **도메인 암호**: 서비스 계정의 암호입니다.
- **조직 구성 단위**: HDInsight 클러스터에 사용하려는 OU의 고유한 이름입니다. 예: OU=HDInsightOU,DC=contoso,DC=onmicrosohift,DC=com. 이 OU가 없을 경우 HDInsight 클러스터는 이 OU를 만들려고 시도합니다. 
- **LDAPS URL**: 예, ldaps://contoso.onmicrosoft.com:636
- **액세스 사용자 그룹**: 클러스터에 동기화할 사용자가 속하는 보안 그룹입니다. 예: HiveUsers. 여러 사용자 그룹을 지정하려면 쉼표 ','로 구분하십시오.
 
다음 스크린샷에는 Azure Portal의 구성이 나와 있습니다.

![Azure HDInsight 도메인에 가입된 Active Directory Domain Services 구성](./media/apache-domain-joined-configure-using-azure-adds/hdinsight-domain-joined-configuration-azure-aads-portal.png)에서도 확인할 수 있습니다.


## <a name="next-steps"></a>다음 단계
* Hive 정책 및 Hive 쿼리 실행에 대한 자세한 내용은 [도메인에 가입된 HDInsight 클러스터에 대한 Hive 정책 구성](apache-domain-joined-run-hive.md)을 참조하세요.
* SSH를 사용하여 도메인 가입 HDInsight 클러스터에 연결하려면 [Linux, Unix 또는 OS X의 HDInsight에서 Linux 기반 Hadoop과 SSH 사용](../hdinsight-hadoop-linux-use-ssh-unix.md#domainjoined)을 참조하세요.

