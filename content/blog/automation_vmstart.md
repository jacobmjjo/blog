---
title: "Azure automation을 이용하여 자동 실행"
date: 2022-03-06T23:44:21+09:00
slug: ""
description: ""
keywords: []
draft: true
tags: ["Cloud","Azure"]
math: false
toc: false
---

Azure의 VM은 자동 종료 서비스를 지원해준다. 하지만 자동 실행 서비스는 별도로 만들어야 한다.  
azure function으로 자동 시작을 구현하려 했지만 결국 포기했다.  정보도 많이 없고 오히려 automation이 잘 설정되어있는 듯 하다.  
아래의 링크를 참고하여 만들려고 했다.

[Automatically Start/Stop Azure Windows virtual machines (Azure automation)](https://izy.codes/azure-automation-virtual-machine-on-off-en/)

Azure는 runbook gallery가 많이 활성화 되어 있는데 일종의 aws의 람다로 치면 공개된 템플릿 같은거다.  
다행히도 Window microsoft 에서 start runbook을 친히 만들어 주었다.  
해당 runbook을 import를 하여 사용하면 될 것이다 라고 생각했다.  
하지만 문제가 하나 발생했다. 현재 제공하고 있는 갤러리의 PowerShell 모듈이 너무 오래전 것이다보니 에러가 발생하는 것이었다.  
갤러리에서 Azure 리소스를 조절할때 쓰는 모듈은 AzureRM으로 2024년이후로는 종료되는 파워셀 모듈이다. 그래서 Az로 모듈을 변경해야 한다.  
다행히 전에 Automation을 이용하여 스냅샷을 만들려고 했을때와 동일한 문제여서 그때 방법을 도입했다.  

[Azure automation을 이용하여 자동 스냅샷 생성](https://josephmjjo.github.io/blog/automation_snapshot/)

문제는 import 했던 갤러리가 그랙픽 형태라는 것이다. 

![Untitled](/img/automation_vmstart/Untitled.png)

그러다 보니 수정을 해야하는 입장에서는 커맨드 형태보다 더 파악하기 힘들었다.  
그래서 차라리 아예 새로 만드는 것을 어떤가 싶어서 새로 만들어 보았다.  

```powershell
$connectionName = "AzureRunAsConnection"

try{

#Getting the service principal connection "AzureRunAsConnection"

$servicePrincipalConnection = Get-AutomationConnection -name $connectionName

"Logging into Azure..."

Connect-AzAccount -ServicePrincipal -TenantID $servicePrincipalConnection.TenantID -ApplicationID $servicePrincipalConnection.ApplicationID -CertificateThumbprint $servicePrincipalConnection.CertificateThumbprint

}

catch{

if(!$servicePrincipalConnection){

$ErrorMessage = "Connection $connectionName not found."

throw $ErrorMessage

}else {

Write-Error -Message $_.Exception

throw $_.Exception

}

}

if($err) {

throw $err

}

$resourceGroupName = "{리소스 그룹}"

# Get VMs with snapshot tag

$tagResList = Get-AzResource -TagName "AutoStart" -TagValue "True" | foreach {

Get-AzResource -ResourceId $_.resourceid
}

foreach($tagRes in $tagResList) {

if($tagRes.ResourceId -match "Microsoft.Compute"){

$vmInfo = Get-AzVM -ResourceGroupName $tagRes.ResourceId.Split("/")[4] -Name $tagRes.ResourceId.Split("/")[8] -Status
if ($vmInfo.Statuses[1].DisplayStatus -ne "Vm running"){
	Start-AzVM -ResourceGroupName $resourceGroupName -Name $vmInfo.Name 
	$vmInfo.Name + " is starting"
}

else{
 $vmInfo.Name + " is already running.Therefore don't need to take strating"
 }
}
 

else{

$tagRes.ResourceId + " is not a compute instance"

}

}
```

결과는 잘 되는 것 같다

![Untitled](/img/automation_vmstart/Untitled%201.png)