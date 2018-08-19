관계형 데이터베이스: RDS
====

## AWS에서 관계형 데이터베이스를 사용하는 방법
* AWS가 제공하는 관계형 데이터베이스 서비스 : [Amazon RDS](https://console.aws.amazon.com/rds/home)
* 가상 서버에 사용자가 직접 관리하는 관계형 데이터베이스 서버를 설치/운영

## Amazon RDS (Amazon Relational Database Service)
* 지원하는 엔진 : MySQL, Oracle Database, MS SQL Server, PostgreSQL, Amazon Aurora, MariaDB
* Managed Service : 서비스 제공자가 관리 역할을 수행함. 즉, AWS에서 관리함.

| | Amazon RDS | 자체호스팅|
|---|---|---|
| AWS 서비스비용 | 더 비싸다 | 더 싸다 (EC2가 더 쌈) |
| 총 소유 비용 (TCO) | 더 싸다. 운영 비용이 여러 고객에게 분할되기 때문. | 더 비싸다. 데이터베이스를 관리하는 자체 인력이 필요하기 때문. (인건비) |
| 품질 | AWS 전문가들이 매니지드 서비스를 책임지고 있음. | 자체 전문가 팀을 구성하고 품질관리를 직접 수행해야 함. |
| 유연성 | 높다 | 더 높다 |

전문가 팀이 있다면 자체호스팅이 더 나을 수 있으나 대부분의 경우 Amazon RDS가 더 유용할 것으로 보인다. ~~그래서 이 책에서는 EC2에 올리는걸 설명하지 않음. 걍 서버에 올리는거랑 같긴함~~

## RDS로 MySQL 사용해보기
워드프레스 애플리케이션에서 MySQL을 사용했던 이전 예제를 RDS에서 돌려보자!

아래와 같은 과정을 거치면 데이터베이스를 간단하게 연결할 수 있다.
* 데이터베이스 인스턴스 시작
* 데이터베이스 엔드포인트에 앱 연결

### 워드프레스 스택 생성하기
1. MySQL과 워드프레스 블로그 플랫폼을 연결하기 위한 [CloudFormation 템플릿 다운로드](https://github.com/AWSinAction/code/blob/master/chapter9/template.json)

2. AWS CLI를 이용하여 WordPress CloudFormation Stack 생성하기

```bash
$ aws cloudformation create-stack --stack-name wordpress --template-url https://s3.amazonaws.com/awsinaction/chapter9/template.json --parameters ParameterKey=KeyName,ParameterValue=mykey ParameterKey=AdminPassword,ParameterValue=test1234 ParameterKey=AdminEMail,ParameterValue=yenarue@gmail.com
```

3. AWS CLI나 콘솔 페이지의 [Amazon CloudFormation](https://console.aws.amazon.com/cloudformation) 메뉴에서 `wordpress` 스택의 상태를 확인한다. (`StackStatus` 필드 값 확인)

```bash
$ aws cloudformation describe-stacks --stack-name wordpress
{
    "Stacks": [
        {
            "StackStatus": "CREATE_IN_PROGRESS",
            "StackName": "wordpress",
            "Parameters": [
....중략
```

 4. `wordpress` 스택의 상태가 `CREATE_COMPLETE`가 되면 `Outputs`의 `OutputValue` 값을 확인하자 : 워드프레스 블로그의 URL

 ```bash
 $ aws cloudformation describe-stacks --stack-name wordpress
{
    "Stacks": [
        {
            "StackStatus": "CREATE_COMPLETE",
            "StackName": "wordpress",
            "Parameters": [
....중략
     "Outputs": [
                {
                    "OutputKey": "URL",
                    "OutputValue": "http://awsinaction-elb-1502024821.us-east-1.elb.amazonaws.com/wordpress",
                    "Description": "Wordpress URL"
                }
            ],
....생략
 ```

5. [Amazon RDS](https://console.aws.amazon.com/rds)로 들어가서 RBS 인스턴스가 생성되어있음을 확인하자.

### 그 외 사용자가 할 수 있는 것 들..
아래와 같은 작업들을 [클라우드와치 모니터링](https://console.aws.amazon.com/cloudwatch/home?region=us-east-1)을 활용하여 처리 할 수 있다. 아마 CloudWatch는 제한된 개수의 지표까지는 평생 무료였던 것 같다. 확인필요.

* 데이터베이스의 남은 스토리지 용량 모니터링 -> 스케일업 결정
* 데이터베이스의 성능 모니터링 -> I/O, 컴퓨터 성능 업그레이드 결정
