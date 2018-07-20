# VPC 구성하기

VPC 스택을 구성합니다.

두개의 AZ 에 각각 퍼블릿서브넷과 프라이빗서브넷을 구성합니다.

## 1. VPC 스택 생성하기

```sh
$ lono new myvpc
$ cd myvpc
$ lono import https://s3.amazonaws.com/codebuild-cloudformation-templates-public/vpc_cloudformation_template.yml --name myvpc

# 파라미터 파일을 적당히 수정해 준다.
$ vi config/params/base/myvpc.txt
EnvironmentName=myvpc
VpcCIDR=10.192.0.0/16
PublicSubnet1CIDR=10.192.10.0/21
PublicSubnet2CIDR=10.192.20.0/21
PrivateSubnet1CIDR=10.192.30.0/21
PrivateSubnet2CIDR=10.192.40.0/21

# 스택을 생성한다.
$ lono cfn create myvpc
```

## 2. 생성된 VPC 스택 업데이트하기

```sh
$ vi app/templates/myvpc.yml
$ lono cfn update myvpc
```

## 3. 생성된 스택 삭제하기

```sh
$ lono cfn delete myvpc
```
