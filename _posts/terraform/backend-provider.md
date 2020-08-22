---
title: "backend.tf, provider.tf role_arn configuration"
date: 2020-08-22 14:30:00 -0400
categories: Terraform backend.tf provider.tf role_arn assume_role
---
IAM 계정의 유저를 사용하여 PRD, STG, DEV 계정으로 자원을 생성하는 것을 테스트하고자 시도하였으나 여러가지 문제로 인하여 접근 에러가 발생하는 문제가 발생하였다. 그래서 여러 케이스를 만든 후에 테스트를 하여서 아래와 같이 간단히 타 계정에 자원을 배포하도록 하는 설정 방법을 찾았다. 

정리하면, 

* backend.tf 설정과 provider.tf 설정은 서로 연관이 없음
* backend.tf에서 타 계정의 s3로 tfstate 파일을 생성하고자 한다면 role_arn 설정하여 해결 가능
* provider.tf에서 타 계정에 자원을 배포하고자 한다면, profile 설정하여 해결 가능 (해당 구성 전, ~/.aws/credentials와 config 파일 사전 설정  필요)
* provider.tf에 처음부터 assume_role 설정으로 타 계정에 자원을 배포 가능함

## Lesson Learned

Terraform doesn't use the AWS provider.tf configuration in order to access the terraform.tfstate bucket. It only uses the backend.tf file. If we want to make the backend bucket in other account, we use "role_arn" in the backend.tf file.

```
terraform {
  backend "s3" {
    bucket         = "xxxxx-terraform-tfstates"
    key            = "terraform/xxxxx_state"
    region         = “ap-northeast-2"
    role_arn       = "arn:aws:iam::<MY_AWS_ACCOUNT_ID>:role/role_name"
  }
}
```

and then, we make the provider.tf with assume_role option or profile option, so we can make the resources in the other aws account.

Option1) Use assume_role option

```
provider "aws" {
    region           = var.aws_region
    # profile          = var.profile_name
    
    assume_role {
        session_name = var.aws_session_name
        role_arn     = "arn:aws:iam::${var.aws_account}:role/${var.aws_role}"
    }
}
```

Option2) Use profile (the source_profile value should be the profile which can be changed to switch role of other account)

```
provider "aws" {
    region           = var.aws_region
      profile          = var.profile_name
    }
}
```

## Others

If the cofiguration files or other directories after 'git push', It is able to use "git rm -r cached ." and add the file names and directories' name to .gitignore. after that, run "git push" command. 
