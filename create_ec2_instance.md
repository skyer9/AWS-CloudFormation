# EC2 인스턴스 생성하기

아래에는 CloudFormation 을 이용해 EC2 인스턴스를 생성하는 방법을 설명합니다.
최종 결과물은 EC2 인스턴스 한개와 보안그룹 한개가 생성됩니다.

## 1. 템플릿 생성하기

빈 파일에 템플릿을 생성하기보다는 AWS 에서 제공하는 [템플릿](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/sample-templates-services-us-west-2.html#w1ab2c21c45c15c15)을 기반으로 하는게 좋습니다.
여러 템플릿중 Amazon EC2 instance in a security group 으로 명명된 템플릿을 베이스 템플릿으로 합니다.
제공되는 템플릿은 json 으로 되어있습니다. 하지만, yaml 이 보다 편하므로 yaml 로 변환합니다.

```sh
mkdir templates
cd templates
curl -o single-instance.json "https://s3-us-west-2.amazonaws.com/cloudformation-templates-us-west-2/EC2InstanceWithSecurityGroupSample.template"
ruby -ryaml -rjson -e 'puts YAML.dump(JSON.load(ARGF))' < single-instance.json > single-instance.yml
```

기본 템플릿을 분석해 보도록 합니다.

우선, ```jq``` 라는 툴이 설치되어 있지 않으면 설치해 줍니다.

```sh
sudo yum install jq
```

아래 명령으로 최상위 레벨의 Key 들을 확인해 봅니다.

```sh
$ cat single-instance.json | jq -r 'keys[]'
AWSTemplateFormatVersion
Description
Mappings
Outputs
Parameters
Resources
```

- AWSTemplateFormatVersion : 템플릿의 버전을 표시합니다.
- Description : 템플릿을 설명하는 코멘트입니다.
- Mappings : 템플릿 생성시 사용가능한 Key 와 선택가능한 Value 의 목록입니다.
- Outputs : 템플릿을 실행하면서 생성되는 결과물들입니다.
- Parameters : 템플릿 실행시 템플릿에 전달가능한 파라미터 목록입니다.
- Resources : 템플릿이 생성하는 리소스와 속성정보입니다. 유일한 필수값입니다.

위 내용중 가장 중요한 Key 는 Parameters 와 Resources 입니다.
우선 ```EC2Instance``` 리소스를 중점적으로 확인해 봅니다.

```yaml
  EC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType:
        Ref: InstanceType
      SecurityGroups:
      - Ref: InstanceSecurityGroup
      KeyName:
        Ref: KeyName
      ImageId:
        Fn::FindInMap:
        - AWSRegionArch2AMI
        - Ref: AWS::Region
        - Fn::FindInMap:
          - AWSInstanceType2Arch
          - Ref: InstanceType
          - Arch
```

[```AWS::EC2::Instance```](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-instance.html) 는 링크되어 있는 문서에 자세히 설명되어 있습니다.

위에 여러개의 [```Ref```](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html) 가 사용되는 것을 볼 수 있습니다.
예를들어 ```InstanceSecurityGroup``` 은 템플릿에 아래와 같이 정의되어 있습니다.

```yaml
  InstanceSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable SSH access via port 22
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: '22'
        ToPort: '22'
        CidrIp:
          Ref: SSHLocation
```

```InstanceType``` 또한 파라미터 Key 하위에 다음과 같이 정의되어 있습니다.

```yaml
Parameters:
  KeyName:
    Description: Name of an existing EC2 KeyPair to enable SSH access to the instance
    Type: AWS::EC2::KeyPair::KeyName
    ConstraintDescription: must be the name of an existing EC2 KeyPair.
  InstanceType:
    Description: WebServer EC2 instance type
    Type: String
    Default: t2.small
    AllowedValues:
    - t1.micro
    - t2.nano
    - t2.micro
    - t2.small
    - t2.medium
```

## 2. 스택 띄우기

```sh
$ aws cloudformation create-stack \
    --template-body file://single-instance.yml \
    --stack-name single-instance \
    --parameters ParameterKey=KeyName,ParameterValue=키이름 ParameterKey=InstanceType,ParameterValue=t2.micro
```

## 3. 스택 삭제하기

불필요한 과금방지를 위해 생성한 스택을 삭제한다.

```sh
$ aws cloudformation delete-stack \
    --stack-name single-instance
```