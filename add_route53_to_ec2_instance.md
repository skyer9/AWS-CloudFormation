# EC2 인스턴스에 Route53 추가하기

이전 챕터에서 생성한 ```single-instance.yml``` 을 기반으로 DNS 를 추가하도록 합니다.

## 1. AWS CloudFormation 문서 확인하기

우선, AWS CloudFormation 문서중 [AWS::Route53::RecordSet](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-route53-recordset.html#w2ab2c21c10e1037c15) 을 확인합니다.

## 2. single-instance.yml 수정하기

우선 파일을 복사해서 새로운 파일을 만듭니다.

```sh
cp single-instance.yml instance-and-route53.yml
```

```instance-and-route53.yml``` 안에 ```Resources``` 파트에 아래 내용을 추가해줍니다.

```yaml
Resources:
  ......
  DnsRecord:
    Type: AWS::Route53::RecordSet
    Properties:
      HostedZoneName: !Ref 'HostedZoneName'
      Comment: DNS name for my instance.
      Name: !Join ['', [!Ref 'Subdomain', ., !Ref 'HostedZoneName']]
      Type: A
      TTL: '900'
      ResourceRecords:
      - !GetAtt EC2Instance.PublicIp
```

그리고, ```Parameters``` 파트에 다음 내용을 추가해줍니다.

```yaml
Parameters:
  ......
  HostedZoneName:
    Description: The route53 HostedZoneName. For example, "mydomain.com."  Don't forget the period at the end.
    Type: String
  Subdomain:
    Description: The subdomain of the dns entry. For example, hello -> hello.mydomain.com, hello is the subdomain.
    Type: String
```

위 코드가 작동하기 위해서는 Route53 에 host zone 이 등록되어 있어야 합니다. 여기서는 ```test.co.kr``` 이 등록되어 있다고 가정합니다.
최종적으로 우리는 ```testsubdomain.test.co.kr``` 도메인을 생성하고 EC2 인스턴스에 연결하도록 합니다.

## 3. 스택 띄우기

아래 코드로 스택을 생성할 수 있습니다.

```sh
aws cloudformation create-stack \
    --template-body file://instance-and-route53.yml \
    --stack-name route53 \
    --parameters ParameterKey=KeyName,ParameterValue=키이름 ParameterKey=InstanceType,ParameterValue=t2.micro ParameterKey=HostedZoneName,ParameterValue=test.co.kr. ParameterKey=Subdomain,ParameterValue=testsubdomain
```

하지만, 위 방식은 가독성이 매우 안좋으므로 파라미터 파일을 생성하는 방식으로 변경합니다.
```instance-and-route53-parameters.json``` 파일을 생성하고, 아래 내용을 입력해 줍니다.
```test.co.kr.``` 에 마침표가 있는 것은 오타가 아니며 반드시 있어야 합니다.

```json
[
  {
    "ParameterKey": "KeyName",
    "ParameterValue": "키이름"
  },
  {
    "ParameterKey": "InstanceType",
    "ParameterValue": "t2.micro"
  },
  {
    "ParameterKey": "HostedZoneName",
    "ParameterValue": "test.co.kr."
  },
  {
    "ParameterKey": "Subdomain",
    "ParameterValue": "testsubdomain"
  }
]
```

아래 명령으로 스택을 생성합니다.

```sh
aws cloudformation create-stack \
    --template-body file://instance-and-route53.yml \
    --stack-name route53 \
    --parameters file://instance-and-route53-parameters.json
```

아래 명령으로 생성된 EC2 인스턴스에 도메인을 이용해 접속할 수 있습니다.

```sh
ssh -i ~/키이름.pem testsubdomain.test.co.kr
```

## 3. 스택 삭제하기

불필요한 과금방지를 위해 생성한 스택을 삭제합니다.

```sh
aws cloudformation delete-stack --stack-name route53
```