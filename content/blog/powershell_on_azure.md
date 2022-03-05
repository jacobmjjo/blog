---
title: "Powershell 로 에저 리소스 관리하는 법"
date: 2022-02-10T19:52:59+09:00
slug: ""
description: ""
keywords: ["Azure"]
draft: true
tags: ["Azure","PowerShell"]
math: false
toc: false
---

에저는 윈도우에 특화 되어 있다보니 자연스레 powershell을 많이 사용하게 된다.

오늘은 네트워크 문제에 있어서 멀티 vpn을 쓰게 될 경우 경로 기반과 정책 기반으로의 차이와 정책 기반일 경우에 Power shell로 설정 하는 방법을 적어보겠다.

아직 확정은 된건 아니고 추후 적용할 경우를 대비하여 예습했다. 그렇기에 딱 Powershell 연습용으로만 적었다.

하단의 링크는 참고한 링크다.

powershell로 vpn 정책기반 설정

[S2S VPN 및 VNet 간 연결을 위한 IPsec/IKE 정책: PowerShell - Azure VPN Gateway](https://docs.microsoft.com/ko-kr/azure/vpn-gateway/vpn-gateway-ipsecikepolicy-rm-powershell)


우선 Powershell에서 유용하게 쓰기 위해서는 powershell을 쓸 수 있는 환경을 가지고 있어야 한다.

필자가 쓰고 있는 컴퓨터는 맥 OS이길래 cloudshell로 powershell을 사용했다. 

아래 링크를 클릭하면 Azure 용 클라우드 쉘 사용이 가능하다.

[Microsoft Azure](https://portal.azure.com/#cloudshell/)

현재 사용하고 있는 VPN의 정책 기반이 뭔지를 알아보기 위해 사용했던 커맨드 문이다.

위의 Azure 공식사이트 자료는 vpn을 만드는 단계도 포함되어 있지만 여기서는 이미 VPN은 만들어졌기에 조회 위주로 한다.

```powershell
#우선 az 아이디에 로그인 해야한다.
Connect-AzAccount
# 만약 디바이스 권한을 없을 경우 디바이스 권한도 같이 요청하는 커맨드를 입력해야 한다.
Connect-AzAccount -UseDeviceAuthentication

#구독이름
$Sub1 = "{구독 이름}"
Select-AzSubscription -SubscriptionName $Sub1
#리소스 그룹 이릅
$RG1 = "{리소스 그룹 이름}"
#지역 이름
$Location1 = "koreacentral"
#Vnet 이름
$VNetName1     = "{Vnet 이름}"
#Vnet 정보 조회
#이미 만들어 놓은게 있어서 조회만 하면 된다.
#여기서 나온 서브넷 이름이라든지 여러 이름을 변수로 만들어서 쓰면 편하다.

$vnet1      = Get-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1
#위에서 만들어진 object는 Powershell에서 쓰는 객체로 PS를 앞에 붙인다. 가상 네트워크 같은 경우에는 PSVirtualNetwork이다.
#PS 객체들은 여러 속성이 json 형태로 나오는데 원하는 키값을 얻고 싶으면 .키를 하면된다. 예를 들면 $vnet1.id 등이다.

#게이트웨이 서브넷 정보 불러오기
#여기서는 bastion으로 사용하고 있는 서브넷을 썼다
$subnet1    = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet1 
$DevHubSubName = "{개발 서브넷 이름}"

$DevPubSub1 = Get-AzVirtualNetworkSubnetConfig -Name $DevHubSubName -VirtualNetwork $vnet1

$PrdHubSubName = "{운영 서브넷 이름}"
$PrdPubSub1 = Get-AzVirtualNetworkSubnetConfig -Name $PrdHubSubName -VirtualNetwork $vnet1

#공용 ip가 이미 만들어져서 조회하고 vpn pip로 추측되는 이름 찾아서 변수 지정
Get-AzPublicIpAddres
$gwlpip1    = Get-AzPublicIpAddress -Name "{공용 IP 이름}"

#만들어 놓은 vpn이 있으므로 
Get-AzVirtualNetworkGateway -ResourceGroupName $RG1
$VpnGw1 = Get-AzVirtualNetworkGateway -Name "{VPN 이름}" -ResourceGroupName $RG1

#local gateway 조회
$lng = Get-AzLocalNetworkGateway  -ResourceGroupName $RG1

$LngOnPrem = Get-AzLocalNetworkGateway -Name "{로컬 게이트 웨이 이름}" -ResourceGroupName $RG1

$connection1  = Get-AzVirtualNetworkGatewayConnection -Name M365-ONPREMISE-CONNECT-1 -ResourceGroupName $RG1
$LngOnPrem.IpsecPolicies
```

이러면 내가 만들어 놓은 VPN의 연결 정책을 확인 할 수 있다.

현재는 정책기반이 아닌 경로 기반이기에 나오는 값은 없다.

다음에는 정책기반과 VIT&경로 기반에 대해 업로드 하겠다.