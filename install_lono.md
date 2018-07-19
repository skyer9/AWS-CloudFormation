# lono 설치 및 실행하기

lono 는 CloudFormation 템플릿 생성 및 실행을 편하게 해주는 툴입니다.

## 1. lono 설치하기

아래 명령으로 lono 를 간단히 설치할 수 있습니다.

```sh
$ ruby -v
# ruby 버전이 2.2 미만이면 업그래이드해야 한다.
# sudo yum install -y ruby24
# sudo alternatives --set ruby /usr/bin/ruby2.4
# sudo yum -y install gcc ruby24-devel rubygems compass
# sudo gem install bundler

$ sudo gem install lono
```

## 2. EC2 인스턴스 생성하기

```sh
$ lono new myinfra
$ cd myinfra
$ lono import https://s3-us-west-2.amazonaws.com/cloudformation-templates-us-west-2/EC2InstanceWithSecurityGroupSample.template --name ec2

# 파라미터 파일을 적당히 수정해 준다.
$ lono cfn create ec2
```

## 3. 생성된 스택 삭제하기

```sh
$ lono cfn delete ec2
```

## 4. 추가 명령 확인하기

아래의 명령들 또한 사용할 수 있다.

```sh
$ lono cfn create mystack-$(date +%Y%m%d%H%M%S) --template mystack --params mystack
$ lono cfn create mystack-$(date +%Y%m%d%H%M%S) # shorthand if template and params file matches.
$ lono cfn diff mystack-1493859659
$ lono cfn preview mystack-1493859659
$ lono cfn update mystack-1493859659
$ lono cfn delete mystack-1493859659
$ lono cfn create -h # getting help
```
