---
title: "Azure automation을 이용하여 자동 스냅샷 생성"
date: 2022-03-01T21:21:26+09:00
slug: ""
description: ""
keywords: []
draft: true
tags: ["Cloud",""]
math: false
toc: false
---
Azure VM의 모든 데이터를 백업하는 방식은 크게 2가지가 있다. 

1. Azure의 백업 서비스
2. 스냅샷

Azure에서 제공하는 백업 서비스 링크

[https://docs.microsoft.com/ko-kr/azure/backup/quick-backup-vm-portal](https://docs.microsoft.com/ko-kr/azure/backup/quick-backup-vm-portal)

스냅샷 링크 

[가상 하드 디스크의 Azure 스냅샷 만들기 - Azure Virtual Machines](https://docs.microsoft.com/ko-kr/azure/virtual-machines/snapshot-copy-managed-disk?tabs=portal)

둘의 큰 차이점은 백업은 바로 다이렉트로 백업본으로 복구할 수 있지만 스냅샷은 직접 한 번 스냅샷으로 디스크를 만든 다음에 VM 디스크를 교체하거나 새 VM에 디스크를 붙여야 한다.

즉 자동으로 한번에 복구 vs 수동으로 여러 절차를 걸쳐서 복구 이다.

디스크 백업과 스냅샷 비교 링크

[Azure native backup vs Azure disk snapshot - UnixArena](https://www.unixarena.com/2020/05/azure-native-backup-vs-azure-disk-snapshot.html/)

내용만 보면 백업이 더 좋지만 비용이 조금 더 들고 현재 SQL 백업이 알 수 없는 오류로 인해 실행되지 못하고 있는 상황이다.

그래서 먼저 스냅샷을 구현해야 했는데 스냅샷은 수동으로 실행해야 하는 점이 문제 였다.

그래서 자동으로 스냅샷을 만들 수 있는 방법이 있을까 찾아보다가 다행히도 누군가가 Automation을 이용하여 

자동 스냅샷을 구현한 것을 확인했다.

누군가가 공유한 링크다.

[Automate Disk Snapshots using Azure | StarWind Blog](https://www.starwindsoftware.com/blog/automating-disk-snapshots-using-azure-runbook)

그런데 문제가 하나 생겼다. 해당 powerShell의 모듈이 너무 오래전 것이다보니 에러가 발생한 것이다.

블로그에서 사용한 모듈은 AzureRM으로 2024년 이후로는 서비스가 종료된다.

[Overview of the AzureRM PowerShell module](https://docs.microsoft.com/en-us/powershell/azure/azurerm/overview?view=azurermps-6.13.0)

그래서 해당 모듈을 Az로 변경해야 한다. 

가만히 보니 블로그에 공개된 코드의 AzureRM 명령어를 동일한 기능을 하는 Az 명령어로 바꾸면 될 것 같다는 생각이 들었다.

해당 방법을 적용한 코드이다.

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

# Get VMs with snapshot tag

$tagResList = Get-AzResource -TagName "Snapshot" -TagValue "True" | foreach {

Get-AzResource -ResourceId $_.resourceid

}

foreach($tagRes in $tagResList) {

if($tagRes.ResourceId -match "Microsoft.Compute")

{

# "/"가 아닌 "//"인 것은 아직도 의문이다.
$vmInfo = Get-AzVM -ResourceGroupName $tagRes.ResourceId.Split("//")[4] -Name $tagRes.ResourceId.Split("//")[8]

#Set local variables

$location = $vmInfo.Location

$resourceGroupName = $vmInfo.ResourceGroupName

$timestamp = Get-Date -f MM-dd-yyyy_HH_mm_ss

#Snapshot name of OS data disk

$snapshotName = $vmInfo.Name + $timestamp

#Create snapshot configuration

$snapshot = New-AzSnapshotConfig -SourceUri $vmInfo.StorageProfile.OsDisk.ManagedDisk.Id -Location $location -CreateOption copy

#Take snapshot

New-AzSnapshot -Snapshot $snapshot -SnapshotName $snapshotName -ResourceGroupName $resourceGroupName

if($vmInfo.StorageProfile.DataDisks.Count -ge 1){

#Condition with more than one data disks

for($i=0; $i -le $vmInfo.StorageProfile.DataDisks.Count - 1; $i++){

#Snapshot name of OS data disk

$snapshotName = $vmInfo.StorageProfile.DataDisks[$i].Name + $timestamp

#Create snapshot configuration

$snapshot = New-AzSnapshotConfig -SourceUri $vmInfo.StorageProfile.DataDisks[$i].ManagedDisk.Id -Location $location -CreateOption copy

#Take snapshot

New-AzSnapshot -Snapshot $snapshot -SnapshotName $snapshotName -ResourceGroupName $resourceGroupName

}

}

else{

Write-Host $vmInfo.Name + " doesn't have any additional data disk."

}

}

else{

$tagRes.ResourceId + " is not a compute instance"

}

}
```

결과는 잘 되는 것 같다

![Untitled](/img/automation_snapshot/Untitled.png)