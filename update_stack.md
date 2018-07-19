# 스택 업데이트하기

이미 실행중인 스택을 업데이트하는 방법을 설명합니다.

이전 두 챕터를 통해 우리는 아래의 두개의 파일을 가지고 있습니다.

```
single-instance.yml
instance-and-route53.yml
```

먼저 EC2 인스턴스만 생성되도록 스택을 생성하고, 생성된 스택에 DNS 를 붙이도록 하겠습니다.

## 1. EC2 인스턴스 생성하기

아래 명령으로 스택을 생성합니다.

```sh
$ aws cloudformation create-stack \
    --template-body file://single-instance.yml \
    --stack-name single-instance \
    --parameters ParameterKey=KeyName,ParameterValue=키이름 ParameterKey=InstanceType,ParameterValue=t2.micro
```

AWS CloudFormation 콘솔에 스택이 생성된 것을 확인할 수 있습니다.

## 2. 스택 업데이트하기

아래 명령으로 ```single-instance``` 스택을 업데이트합니다.

```sh
$ aws cloudformation update-stack \
    --stack-name single-instance \
    --template-body file://instance-and-route53.yml \
    --parameters file://instance-and-route53-parameters.json
```

AWS CloudFormation 콘솔을 통해 생성된 스택이 업데이트되는 것을 확인할 수 있습니다.

## 3. 스택 삭제하기

불필요한 과금방지를 위해 생성한 스택을 삭제합니다.

```sh
$ aws cloudformation delete-stack \
    --stack-name single-instance
```