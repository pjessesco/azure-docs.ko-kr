---
title: "Azure File Sync 에이전트 릴리스 정보 | Microsoft Docs"
description: "Azure File Sync 릴리스 정보"
services: storage
documentationcenter: 
author: wmgries
manager: klaasl
editor: tamram
ms.assetid: 
ms.service: storage
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 10/08/2017
ms.author: wgries
ms.openlocfilehash: 74f926743713bfd71eb524fdc2794cd7e187166e
ms.sourcegitcommit: 4723859f545bccc38a515192cf86dcf7ba0c0a67
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/11/2018
---
# <a name="azure-file-sync-agent-release-notes"></a>Azure File Sync 에이전트 릴리스 정보
Azure File Sync(미리 보기)를 사용하여 온-프레미스 파일 서버의 유연성, 성능 및 호환성을 희생하지 않고 Azure 파일에서 조직의 파일 공유를 중앙 집중화할 수 있습니다. 이 작업은 Windows Server를 Azure 파일 공유의 빠른 캐시로 변환하여 수행합니다. Windows Server에서 사용할 수 있는 아무 프로토콜이나 사용하여 데이터를 로컬로(SMB, NFS 및 FTPS 포함) 액세스할 수 있으며 세계 전역에 걸쳐 필요한 만큼 캐시를 보유할 수 있습니다.

이 문서에서는 Azure File Sync 에이전트의 지원되는 버전에 대한 릴리스 정보에 대해 설명합니다.

## <a name="supported-versions"></a>지원되는 버전
다음 버전은 Azure File Sync에서 지원됩니다.

| 에이전트 버전 번호 | 릴리스 날짜 | 다음 날짜까지 지원 |
|----------------------|--------------|------------------|
| 2.0.11.0 | 2018-02-08 | 현재 버전 |
| 1.1.0.0 | 2017-09-26 | 2018-07-30 |

### <a name="azure-file-sync-agent-update-policy"></a>Azure 파일 동기화 에이전트 업데이트 정책
[!INCLUDE [storage-sync-files-agent-update-policy](../../../includes/storage-sync-files-agent-update-policy.md)]

## <a name="agent-version-1100"></a>에이전트 버전 1.1.0.0
다음 릴리스 정보는 2017년 9월 9일에 출시된 에이전트 버전 1.1.0.0에 대한 것입니다. 이 릴리스는 Azure File Sync의 초기 미리 보기 릴리스를 표시합니다!

### <a name="agent-installation-and-server-configuration"></a>에이전트 설치 및 서버 구성
Windows Server와 함께 Azure File Sync 에이전트를 설치하고 구성하는 방법에 대한 자세한 내용은 [Azure File Sync(미리 보기) 배포에 대한 계획](storage-sync-files-planning.md) 및 [Azure File Sync(미리 보기)를 배포하는 방법](storage-sync-files-deployment-guide.md)을 참조하세요.

- 에이전트 설치 패키지(MSI)는 상승된(관리자) 권한으로 설치되어야 합니다.
- Windows Server Core 또는 Nano Server 배포 옵션에서 지원되지 않습니다.
- Windows Server 2016 및 2012 R2에서만 지원됩니다.
- 에이전트에는 2GB 이상의 실제 메모리가 필요합니다.

### <a name="interoperability"></a>상호 운용성
- 바이러스 백신, 백업, 그리고 계층화된 파일에 액세스하는 기타 응용 프로그램은 오프라인 특성을 존중하여 해당 파일의 내용 읽기를 건너뛰지 않는 경우 원치 않은 회수가 발생할 수 있습니다. 자세한 내용은 [Azure File Sync(미리 보기) 문제 해결](storage-sync-files-troubleshoot.md)을 참조하세요.
- 파일 서버 리소스 관리자(또는 다른) 파일 화면을 사용하지 마십시오. 파일이 파일 화면으로 인해 차단된 경우 파일 화면에 무한 동기화 오류가 발생할 수 있습니다.
- 등록된 서버(VM 복제 포함)의 중복으로 예기치 않은 결과(특히, 동기화가 수렴되지 않을 수 있음)가 발생할 수 있습니다.
- 데이터 중복 제거 및 클라우드 계층화는 동일한 볼륨에서 지원되지 않습니다.
 
### <a name="sync-limitations"></a>동기화 제한 사항
다음 항목은 동기화되지 않지만 시스템의 나머지 부분은 정상적으로 계속해서 작동합니다.
- 경로는 2048자보다 깁니다.
- 2K(단일 항목에 약 40개 이상의 Access Control 항목이 있는 경우에만 문제임)보다 큰 경우의 보안 설명자의 DACL 부분
- 보안 설명자의 SACL 부분(감사에 사용됨)
- 확장된 특성
- 대체 데이터 스트림
- 재분석 지점
- 하드 링크
- 압축(서버 파일에서 설정하는 경우)은 변경 내용이 다른 끝점의 해당 파일에 대해 동기화할 때 유지되지 않습니다.
- 데이터 읽기에서 서비스를 보호하는 EFS(또는 다른 사용자 모드 암호화)로 암호화된 파일 
    
    > [!Note]  
    > Azure File Sync는 전송 중인 데이터를 항상 암호화하고 Azure에 있는 미사용 데이터를 암호화할 수 있습니다.
 
### <a name="server-endpoints"></a>서버 끝점
- 서버 끝점은 NTFS 볼륨에서만 만들어질 수 있습니다. ReFS, FAT, FAT32 및 다른 파일 시스템은 이 때 Azure File Sync에 의해 지원되지 않습니다.
- 서버 끝점은 시스템 볼륨에 없을 수도 있습니다(예: C:\MyFolder가 탑재 지점이 아닌 한 C:\MyFolder는 적합한 경로가 아님).
- 장애 조치(failover) 클러스터링은 CSV(클러스터 공유 볼륨)가 아닌 클러스터된 디스크로만 지원됩니다.
- 서버 끝점은 중첩될 수 없지만 서로 동시에 동일한 볼륨에 함께 있을 수 있습니다.
- 한 번에 많은 수의 디렉터리를 서버에서 삭제하면(10,000개 이상) 동기화 오류가 발생할 수 있습니다. 10,000개보다 작은 일괄 처리로 디렉터리를 삭제하고 다음 일괄 처리를 삭제하기 전에 삭제 작업이 성공적으로 동기화되었는지 확인합니다.
- 볼륨의 루트에서 지원되지 않습니다.
- 서버 끝점 내에 OS 또는 응용 프로그램 페이징 파일을 저장하지 마십시오.
 
### <a name="cloud-tiering"></a>클라우드 계층화
- 해당 파일을 올바르게 회수할 수 있는지 확인하기 위해 시스템은 새 서버 끝점을 구성한 후 최초 계층화를 포함하여 최대 32시간 동안 새로운 또는 변경된 파일을 자동으로 계층화하지 않을 수 있습니다. 백그라운드 프로세스에 대한 대기 없이 계층화를 보다 효율적으로 평가할 수 있도록 요청 시 계층화하는 PowerShell cmdlet을 제공합니다.
- 계층화된 파일이 Robocopy를 사용하여 다른 위치로 복사되는 경우 결과 파일은 계층화되지 않지만 Robocopy가 복사 작업에 해당 특성을 잘못 포함하므로 오프라인 특성을 설정할 수 있습니다.
- SMB 클라이언트에서 파일 속성을 볼 때 오프라인 특성은 파일 메타데이터의 SMB 캐싱으로 인해 잘못 설정되도록 나타날 수 있습니다.