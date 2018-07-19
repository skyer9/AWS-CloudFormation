# Change Sets = Dry Run Mode

이전 챕터에서 스택을 업데이트하는 방법을 보았습니다. 하지만 실무적으로는 테스트 단계를 가져야 합니다.

여기서는 ```Change Sets``` 을 생성하여 프리뷰하고, 정상작동을 확인 후 스택에 반영하는 방법을 설명합니다.

## 1. 프리뷰 확인하기

아래 명령으로 스택을 생성합니다.

```sh
$ aws cloudformation create-stack \
    --template-body file://single-instance.yml \
    --stack-name single-instance \
    --parameters ParameterKey=KeyName,ParameterValue=키이름 ParameterKey=InstanceType,ParameterValue=t2.micro
```

아래 명령으로 Change Sets 을 생성합니다.

```sh
$ aws cloudformation create-change-set \
    --stack-name single-instance \
    --template-body file://instance-and-route53.yml \
    --parameters file://instance-and-route53-parameters.json \
    --change-set-name changeset-1
```

AWS CloudFormation 콘솔에서 변경세트(Change Sets)이 생성된 것을 확인할 수 있습니다.

아래 명령으로 프리뷰를 볼 수 있습니다.

```sh
$ aws cloudformation describe-change-set \
    --stack-name single-instance \
    --change-set-name changeset-1 | jq '.Changes[]'
```

```json
{
  "ResourceChange": {
    "Action": "Add",
    "ResourceType": "AWS::Route53::RecordSet",
    "Scope": [],
    "Details": [],
    "LogicalResourceId": "DnsRecord"
  },
  "Type": "Resource"
}
```

AWS CloudFormation 콘솔에서 변경세트(Change Sets) 탭으로 이동한 후, 변경세트 선택 후 상세정보를 조회해서 확인할 수도 있습니다.

## 2. Change Sets 반영하기

```Change Sets``` 을 반영하는 방법은 세가지가 있습니다.

AWS CloudFormation 콘솔에서 변경세트(Change Sets) 탭으로 이동한 후, 변경세트 선택 후 ```실행``` 버튼을 눌러 반영하는 방법이 하나입니다.

또 다른 방법으로는 CLI 를 이용해 이전 챕터에서 썼던 ```update-stack``` 명령을 사용하는 방법입니다.

마지막으로 아래 명령으로 ```Change Sets``` 을 실행하는 방법입니다.

```sh
$ aws cloudformation execute-change-set \
    --stack-name single-instance \
    --change-set-name changeset-1
```

AWS CloudFormation 콘솔을 통해 생성된 스택이 업데이트되는 것을 확인할 수 있습니다.

## 3. 스택 삭제하기

불필요한 과금방지를 위해 생성한 스택을 삭제합니다.

```sh
$ aws cloudformation delete-stack \
    --stack-name single-instance
```