---
title: "테라폼 vpc 및 서브넷 모듈화 for each 사용하기"
date: 2022-02-06T18:32:03+09:00
slug: ""
description: ""
keywords: []
draft: true
tags: ["Cloud","Terraform"]
math: false
toc: false
---

현재 하는 프로젝트의 인프라가 간단하여 연습 겸 테라폼으로 인프라를 만들어 보았다.

테라폼을 사용하는 가장 큰 이유인 모듈화와 여러개의 리소스를 한꺼번에 만들 수 있는 for each 사용을 목표로 작성해 보았다.

사용하는 환경은 운영환경과 개발 환경이다. 문제는 두 환경의 인프라가 약간 차이가 있다. 

개발 환경은 비용 절감을 위해 bastion 서버 1대 와 WAS와 DB(MSSQL)를 하나로 합친 서버 1대, 총 2대다.

반면에 운영환경은 WAS와 DB서버를 각각 1대씩 두어서 총 3대를 운영한다. 

서브넷도 마찬가지로 차이가 있다. 인프라를 구축할때 보통 티어라는 개념으로 서브넷을 만드는데 현 프로젝트 운영 환경기준으로는 bastion용 서브넷 1대, WAS용 서브넷 1대 그리고  DB 서브넷 1대 이렇게 총 3대의 서브넷을 만들어야 한다. 

서브넷 갯수와 서버 갯수 차이를 고려하여 모듈화를 해야 한다.

최상위 계층의 폴더는 크게 4개로 만들었다.

1. Module(모듈)
2. global(공통)
3. dev(개발)
4. prd(운영

여기서 공통이란 환경 여부 상관없이 공통적으로 사용하는 리소스를 위한 것으로 대표적으로 스토리지가 이에 해당하는데 AWS에서는 S3, Azure에서는 Blob storage이다.  

테라폼을 이용하게 되면 기본적으로 스토리지를 사용하게 되는데 이유는 테라폼 백업 파일을 저장 및 lock을 걸기 위한 용도이다.

여기서 lock이란 만에 하나 여러명을 테라폼을 수정하게 된다면 우발적인 테라폼 코드 수정으로 인한 인프라 변경을 맞기 위한 용도이다.

 

근데 현 프로젝트는 운영환경과 개발환경이 하나의 VPC를 쓰기로 결정되어서 VPC도 global에 추가했다.

그러면 폴더 구성도는 아래와 같게 된다.

![Untitled](/img/terraform_foreach/Untitled.png)

사실 모듈에는 VPC를 안 만들어도 무방하지만 연습 겸 하나 만들어 봤다. 

또한 모듈에 service라는 폴더를 만든 이유는 IAM이나 KMS 같이 서비스 외의 

프로젝트용 테라폼이기에 폴더안에 있는 모든 테라폼 파일을 모두 보여 줄 수는 없지 이해하는데에 핵심 파일인 모듈과 실제 모듈을 사용한 파일은 수정하여 공유 

Module 폴더의 subnet main.tf는 아래처럼 작성했다.

```
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "=2.46.0"
    }
  }
}

resource "azurerm_subnet" "subnet" {
  for_each = var.azurerm_subnet

  name                 = each.value["name"]
  resource_group_name  = each.value["resource_group_name"]
  virtual_network_name = each.value["virtual_network_name"]
  address_prefixes     = each.value["address_prefixes"]
```

여기서 for_each를 사용한 이유는 아까 썼던대로 서브넷이 환경 마다 다르기에 유연하게 만들기 위해서였다.

그럼 저 main.tf를 적용하여 원하는대로 여러개 서브넷 만들 수 있게끔 하려면 어떻게 해야할 까?

바로 동일한 폴더의 variable.tf에서 type을 map으로 하면된다.

아래의 코드가 module 폴더의 variable.tf이다.

맨 하단에 variable "azurerm_subnet"가 subnet을 만들 때 사용할 설정값의 타입을 map으로 하겠다는 뜻이다.

```
variable "resource_group_name" {
  description = "The name of resource group"
  type = string
}

variable "resource_group_location" {
  description = "The location name for Azure infrastructure"
  type = string
  
}

variable "azurerm_virtual_network_name" {
  description = "The name of azurerm_virtual_network"
  type = string
}

variable "azurerm_subnet" {
  description = "The subnet list of subnets"
  **type = map(any)**
}
```

그럼 이거를 실제로 운영환경에서 사용하려면 어떻게 해야할까?

```
provider "azurerm" {
  features {}
  subscription_id = "{구독 아이디}"
}

terraform {
  backend "azurerm" {
      resource_group_name = "{리소스 그룹 이름}"
      storage_account_name = "{스토리지 이름}"
      container_name = "tfstate"
      key = "prd.blob.terraform.tfstate"
      subscription_id = "{구독 아이디}"
  }
}

module "m365" {
  source = "../../modules/service/subnet"

  resource_group_name = data.{리소스그룹이름}.name
  resource_group_location = data.리소스그룹이름}.location
  azurerm_virtual_network_name = data.azurerm_virtual_network.{Vnet 이름}.name
  **azurerm_subnet = var.azurerm_subnet**
  
}
```

먼저 운영 환경의 main.tf의 코드를 보면 운영환경의 variable.tf 값을  이용해서 쓰고있다.

그렇다면 variable.tf에서 사용하고 있다는 것인데 variable.tf로 가보면

```
variable "resource_group_name" {
  description = "The name of resource group"
  type = string
  default = "{리소스 그룹}"
}

variable "resource_group_location" {
  description = "The name of resource location"
  type = string
  default = "Korea Central"
}

variable "virtual_network_name" {
  description = "The name of virtual network"
  type = string
  default = "{리소스 그룹}"
}

variable "virtual_network_address" {
  description = "The name of virtual network address"
  type = list(string)
  default = ["10.0.0.0/24"]
}

variable "azurerm_subnet" {
  description = "Map of Azure VNET subnet configuration"
  type        = map(any)
  default = {
    pub_subnet = {
      name                 = "{퍼블릭 서브넷 이름}"
      resource_group_name  = "{리소스 그룹 이름}"
      virtual_network_name = "{Vnet 이름}"
      address_prefixes     = ["10.0.0.64/27"]
    },
    pri_subnet = {
      name                 = "{프라이빗 서브넷 이름}"
      resource_group_name  = "{리소스 그룹 이름}"
      virtual_network_name = "{Vnet 이름}"
      address_prefixes     = ["10.0.0.96/27"]
    },
    db_subnet = {
      name                 = "{프라이빗 서브넷 이름}"
      resource_group_name  = "{리소스 그룹 이름}"
      virtual_network_name = "{Vnet 이름}"
      address_prefixes     = ["10.0.0.128/27"]
    }
  }
}
```

맨 하단의 azurerm_subnet처럼 자기가 만들고 싶은 subnet 갯수만큼 맵 데이터를 만들면 된다.

운영환경에서는 3대의 서브넷을 만들고 싶어서 3개의 서브넷 정보를 입력했으며 2대를 만들고 싶으면 이와 맞게 만들면 된다.