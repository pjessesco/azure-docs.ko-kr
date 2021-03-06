---
title: "DSC를 사용하여 Azure에 자격 증명 전달 | Microsoft Docs"
description: "PowerShell 필요한 상태 구성을 사용하여 Azure 가상 머신에 안전하게 자격 증명을 전달하는 방법의 개요"
services: virtual-machines-windows
documentationcenter: 
author: zjalexander
manager: timlt
editor: 
tags: azure-resource-manager
keywords: dsc
ms.assetid: ea76b7e8-b576-445a-8107-88ea2f3876b9
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: na
ms.date: 01/17/2018
ms.author: zachal,migreene
ms.openlocfilehash: 140ca3cc9b72afac720e5bcf1d620ac9b1b72132
ms.sourcegitcommit: 059dae3d8a0e716adc95ad2296843a45745a415d
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/09/2018
---
# <a name="passing-credentials-to-the-azure-dsceextension-handler"></a>Azure DSC 확장 처리기에 자격 증명 전달
[!INCLUDE [learn-about-deployment-models](../../../includes/learn-about-deployment-models-both-include.md)]

이 문서에서는 Azure의 필요한 상태 구성 확장을 다룹니다.
DSC 확장 처리기의 개요는 [Azure 필요한 상태 구성 확장 처리기 소개](extensions-dsc-overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)에 나와 있습니다.

## <a name="passing-in-credentials"></a>자격 증명 전달

구성 프로세스의 일부로, 사용자 계정을 설정하거나, 서비스에 액세스하거나, 사용자 컨텍스트에서 프로그램을 설치해야 할 수 있습니다. 이러한 작업을 수행하려면 자격 증명을 제공해야 합니다.

DSC는 자격 증명이 구성에 전달되고 MOF 파일에 안전하게 저장되는 매개 변수가 있는 구성을 허용합니다.
Azure 확장 처리기는 인증서의 자동 관리를 제공하여 자격 증명 관리를 간소화합니다.

지정된 암호로 로컬 사용자 계정을 만드는 다음과 같은 DSC 구성 스크립트를 고려합니다.

```powershell
configuration Main
{
    param(
        [Parameter(Mandatory=$true)]
        [ValidateNotNullorEmpty()]
        [PSCredential]
        $Credential
    )
    Node localhost {
        User LocalUserAccount
        {
            Username = $Credential.UserName
            Password = $Credential
            Disabled = $false
            Ensure = "Present"
            FullName = "Local User Account"
            Description = "Local User Account"
            PasswordNeverExpires = $true
        }
    }
}
```

*node localhost* 를 구성의 일부로 포함해야 합니다.
확장 처리기는 특히 node localhost 문을 찾으므로 이 문이 없으면 다음 단계가 작동하지 않습니다.
형식 캐스팅 *[PsCredential]*은 자격 증명을 암호화하기 위해 이 특정 형식이 확장을 트리거하므로 함께 포함하는 것이 중요합니다.

이 스크립트를 Blob 저장소에 게시합니다.

`Publish-AzureVMDscConfiguration -ConfigurationPath .\user_configuration.ps1`

Azure DSC 확장을 설정하고 자격 증명을 제공합니다.

```powershell
$configurationName = "Main"
$configurationArguments = @{ Credential = Get-Credential }
$configurationArchive = "user_configuration.ps1.zip"
$vm = Get-AzureVM "example-1"

$vm = Set-AzureVMDSCExtension -VM $vm -ConfigurationArchive $configurationArchive 
-ConfigurationName $configurationName -ConfigurationArgument @configurationArguments

$vm | Update-AzureVM
```

## <a name="how-credentials-are-secured"></a>자격 증명의 보안을 유지하는 방법

이 코드를 실행하면 자격 증명을 묻는 메시지가 나타납니다.
자격 증명이 제공되면 메모리에 잠시 저장됩니다.
`Set-AzureVmDscExtension` cmdlet을 사용하여 게시되는 경우 HTTPS를 통해 VM으로 전송되며, 여기서 Azure 저장소는 로컬 VM 인증서를 사용하여 디스크에서 자격 증명을 암호화합니다.
그런 후 메모리에서 즉시 암호 해독되었다가 DSC로 전달될 수 있게 다시 암호화됩니다.

이러한 동작은 [확장 처리기 없이 보안 구성을 사용](https://msdn.microsoft.com/powershell/dsc/securemof)하는 방법과는 다릅니다. Azure 환경에서는 인증서를 통해 구성 데이터를 안전하게 전송하는 방법을 제공합니다. DSC 확장 처리기를 사용할 때는 ConfigurationData에 $CertificatePath 또는 $CertificateID / $Thumbprint를 제공할 필요가 없습니다.

## <a name="next-steps"></a>다음 단계

Azure DSC 확장 처리기에 대한 자세한 내용은 [Azure 필요한 상태 구성 확장 처리기 소개](extensions-dsc-overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)를 참조하세요.

[DSC 확장에 대한 Azure Resource Manager 템플릿](extensions-dsc-template.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)을 검토합니다.

PowerShell DSC에 대한 자세한 내용은 [PowerShell 설명서 센터를 방문하세요](https://msdn.microsoft.com/powershell/dsc/overview).

PowerShell DSC로 관리할 수 있는 추가 기능을 찾으려면 추가 DSC 리소스는 [PowerShell 갤러리에서 찾아보세요](https://www.powershellgallery.com/packages?q=DscResource&x=0&y=0) .
