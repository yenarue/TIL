배포자동화
======
CloudFormation, Elastic BeanStalk, OpsWorks 으로 앱 배포하기

# CloudFormation
예제 실행 안된다..ㅠㅠ 흑흑 나만 안되나 왜 안되지
* EC2 AMI 보니까 템플릿에 있는 애들이 없던데... 그래서 그런가?


# Elastic BeanStalk
웹 앱 배포를 도와주는 서비스. OS, 가상서버 위에 한단계 더 추상화된 계층을 추가하기 때문에 OS단을 걱정할 필요가 없다.

## EB 앱 구성
EB 앱은 3가지 요소(버전, 환경, 구성)로 구성되어있다.

* 앱 (Application) : 논리적 컨테이너. 버전, 환경, 구성이 포함되어 있음.
* 버전 (Version) : 앱의 특정 버전. 실행파일을 아카이브 형태로 압축하여 업로드. 이 아카이브를 가리키는 포인터 역할을 함.
* 구성 템플릿 (Configuration Template) : 커스텀 템플릿으로 환경 구성 및 앱 구성을 관리할 수 있다.
* 환경 (Environment) : EB가 앱을 실행하는 환경. 버전과 구성으로 이루어짐. 버전과 구성을 조합하여 앱을 여러 환경에서 실행할 수 있다.


## EB 앱 만들어보기
CLI 기반으로 만들어보자.

```bash
# 앱 생성하기
$ aws elasticbeanstalk create-application --application-name etherpad
{
    "Application": {
        "ConfigurationTemplates": [],
        "DateCreated": "2018-07-14T12:19:45.681Z",
        "ApplicationName": "etherpad",
        "DateUpdated": "2018-07-14T12:19:45.681Z"
    }
}

# S3 버킷에 etherpad 앱 아카이브 올려놓고 EB에 업로드하기
$ aws elasticbeanstalk create-application-version --application-name etherpad --version-label 1.6.6 --source-bundle "S3Bucket=yenarue-awsinaction,S3Key=chapter5/etherpad.zip"
{
    "ApplicationVersion": {
        "VersionLabel": "1.6.6",
        "DateCreated": "2018-07-14T12:19:49.902Z",
        "ApplicationName": "etherpad",
        "SourceBundle": {
            "S3Key": "chapter5/etherpad.zip",
            "S3Bucket": "yenarue-awsinaction"
        },
        "DateUpdated": "2018-07-14T12:19:49.902Z",
        "Status": "UNPROCESSED"
    }
}

# Node.js 환경생성
$ aws elasticbeanstalk list-available-solution-stacks --output text --query "SolutionStacks[?contains(@, 'running Node.js')] | [0]"
64bit Amazon Linux 2018.03 v4.5.1 running Node.js
$ aws elasticbeanstalk create-environment --environment-name etherpad --application-name etherpad --option-settings "Namespace=aws:elasticbeanstalk:environment,OptionName=EnvironmentType,Value=SingleInstance" --solution-stack-name "64bit Amazon Linux 2018.03 v4.5.1 running Node.js" --version-label 1.6.6
{
    "DateUpdated": "2018-07-14T12:38:52.334Z",
    "DateCreated": "2018-07-14T12:38:52.334Z",
    "SolutionStackName": "64bit Amazon Linux 2018.03 v4.5.1 running Node.js",
    "VersionLabel": "1.6.6",
    "EnvironmentId": "e-j2edwhudmq",
    "Health": "Grey",
    "Tier": {
        "Name": "WebServer",
        "Version": "1.0",
        "Type": "Standard"
    },
    "EnvironmentName": "etherpad",
    "ApplicationName": "etherpad",
    "Status": "Launching"
}

# EB 앱 상태 확인하기
$ aws elasticbeanstalk describe-environments --environment-names etherpad
{
    "Environments": [
        {
            "EnvironmentName": "etherpad",
            "Status": "Launching",
            "SolutionStackName": "64bit Amazon Linux 2018.03 v4.5.1 running Node.js",
            "EnvironmentLinks": [],
            "DateCreated": "2018-07-14T12:38:52.316Z",
            "ApplicationName": "etherpad",
            "AbortableOperationInProgress": false,
            "EnvironmentId": "e-j2edwhudmq",
            "Tier": {
                "Version": "1.0",
                "Type": "Standard",
                "Name": "WebServer"
            },
            "Health": "Grey",
            "DateUpdated": "2018-07-14T12:39:02.363Z"
        }
    ]
}
```

## EB 앱 삭제하기
```bash
# etherpad 환경 종료
$ aws elasticbeanstalk terminate-environment --environment-name etherpad
{
    "Status": "Terminating",
    "Tier": {
        "Version": "1.0",
        "Type": "Standard",
        "Name": "WebServer"
    },
    "DateCreated": "2018-07-14T12:38:52.316Z",
    "CNAME": "etherpad.v3jiu2p8tt.us-east-1.elasticbeanstalk.com",
    "AbortableOperationInProgress": false,
    "EnvironmentName": "etherpad",
    "Health": "Grey",
    "SolutionStackName": "64bit Amazon Linux 2018.03 v4.5.1 running Node.js",
    "EndpointURL": "18.206.26.63",
    "DateUpdated": "2018-07-14T12:52:56.076Z",
    "ApplicationName": "etherpad",
    "EnvironmentId": "e-j2edwhudmq"
}

# 상태가 Terminated 된 이후에 실행할 것
$ aws elasticbeanstalk delete-application --application-name etherpad
```

# [OpsWorks](https://console.aws.amazon.com/opsworks/)
다중 계층 앱 배포하기 (Multilayer Application)
* 가상서버, 로드밸런서, 데이터베이스 등의 AWS 리소스를 제어하고 앱을 배포하는데 활용함.
* 스케일링, 모니터링, 가상서버 업데이트
* Chef, Puppet 을 이용하여 배포 제어함. ~~Ansible은 왜ㅠㅠ~~

## 제공하는 표준 계층들...
* HAProxy (로드 밸런서)
* MySQL (데이터베이스)
* Java App Server (Tomcat Server)
* 정적 웹 서버
* 갱글리아 (모니터링)
* PHP 웹 서버
* Rails 앱 서버 (루비 온 레일스)
* Memcached (인메모리 캐시)
* AWS 플로 (루비)

## 구성요소
스택, 계층, 인스턴스, 앱
* 스택 : 논리적 컨테이너.
* 계층 : 애플리케이션이나 서비스. 스택 내에 속한다.
* 인스턴스 : 가상 서버. 각 계층에 여러 인스턴스를 실행할 수 있다.
* 앱 : 배포할 소프트웨어. 저장소에서 가져올 수 있음. 아카이브 형태로 업로드하는 것도 가능.

## 사용해보기
IRC 웹 클라이언트와 IRC 서버를 올려볼 것이다.

### 스택 만들기
`Go to OpsWorks Stacks` -> `Add your first stack` 로 들어가 스택을 만든다. chef 기반만 가능함.

### 계층 추가하기
#### IRC 서버 계층 추가하기
`Layers` -> `Add layer`로 들어가 커스텀 계층을 하나 추가한다.

##### 방화벽 설정하기 
`irc-create-cloudformation-stack.sh` 을 참고하여 6667포트에 연결하기 위한 방화벽을 생성하자.

`Layers` -> IRC 서버 계층 -> `Security` -> `Security Group` 을 방금 생성한 방화벽으로 설정한다.

##### 레시피 구성하기
`Layers` -> IRC 서버 계층 -> `Recipes` -> `OS packages`에 `ircd-ircu` 패키지 추가

#### IRC 웹 앱 클라이언트 계층 추가하기
`Layers` -> `Add layer` 로 들어가 Node.js 계층을 하나 추가한다.

##### 앱 추가하기
`Apps` -> `Add App`로 들어가 Node.js 타입의 앱을 하나 만든다. 여기서 저장소를 연결할 수 있다.

### 인스턴스 추가하기
IRC 클라이언트와 서버를 실행할 인스턴스를 추가해보자.

1. `Instances` -> `Node.js App Server` 계층에서 `Add an instance`를 클릭 
2. `Instances` -> `irc-server` 계층에서 `Add an instance`를 클릭
3. 각 인스턴스를 `start` 한다. (`Actions`에서 클릭)
4. `online` 상태가 되고나면 public ip를 이용해 접속할 수 있다 :-)

## OpsWorks 삭제하기
콘솔 홈페이지에서 Instance, Apps, Stack 을 모두 삭제한 뒤 다음 명령 실행. ~~확인사살~~

```bash
$ aws cloudformation delete-stack --stack-name irc
```


# 배포 도구 비교 (CloudFormation vs Elastic Beanstalk vs OpsWorks)

|   | CloudFormation  | Elastic Beanstalk  | OpsWorks |
|---|---|---|---|
| 구성관리도구 | 사용 가능한 모든 도구  | 자체 개발  |  Chef |
| 지원되는 플랫폼 | 모두  | PHP, Node.js, IIS, Java(Tomcat), Python, Ruby, Docker  | Ruby on Rails, Node.js, PHP, Java(Tomcat), Custom Platforms  |
| 일반적인 사용 예 | 복잡하고 비표준적인 환경  | 일반적인 웹앱  |  마이크로서비스 환경 |
| 다운탕미 없이 업데이트 | 가능  | 기능 제공  | 기능 제공  |
| 벤더 종속 효과 | 중간 | 높음 | 중간 |