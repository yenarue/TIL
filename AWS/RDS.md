관계형 데이터베이스: RDS
====

## AWS에서 관계형 데이터베이스를 사용하는 방법
* 가상 서버에 사용자가 직접 관리하는 관계형 데이터베이스 서버를 설치/운영
* AWS가 제공하는 관계형 데이터베이스 서비스 : [Amazon RDS](https://console.aws.amazon.com/rds/home)

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

## MySQL RDS 인스턴스 생성 및 앱과 연결해보기
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

### RDS 인스턴스 정보 확인하기

아래의 명령어를 입력하면 위에서 CloudFormation 으로 생성한 RDS 인스턴스의 정보를 확인할 수 있다. 

```bash
$ aws rds describe-db-instances
```

출력되는 정보들 중 몇가지 정보들에 대해 알아보자.

* EndPoint : 외부에서 데이터베이스를 연결하기 위한 엔드포인트 정보들.
```json
"Endpoint": {
                "HostedZoneId": "Z2R2ITUGPM61AM",
                "Port": 3306,
                "Address": "awsinaction-db.ccwadu7jnlsy.us-east-1.rds.amazonaws.com"
            },
```

* DBName : 자동 생성되는 데이터베이스 이름
```json
"DBName": "wordpress",
"DBInstanceClass": "db.t2.micro",
"DBInstanceIdentifier": "awsinaction-db"  // RDS 인스턴스의 식별자
```

* MasterUsername : 데이터베이스 마스터 사용자의 이름. 비번은 표시되지 않음.
```json
"MasterUsername": "wordpress",
```

* Engine : RDS 인스턴스가 제공하는 관계형 데이터 베이스 시스템
```json
"Engine": "mysql",
"EngineVersion": "5.7.22",
```

* MultiAZ : 고가용성 설정

* SecurityGroup : RDS 인스턴스에 연결된 SecurityGroup을 보여준다. 해당 SecurityGroup의 정보는 EC2에서 확인가능함.

* ParameterGroup : RDS 인스턴스에 연결된 ParameterGroup을 보여준다. Parameter들을 모아놓은 그룹으로서, 설정값의 역할과 유사한 듯 하다.

* VpcId : RDS 인스턴스가 실행될 VPC를 정의한다.

* ReadReplicaDBInstanceIdentifiers : AWS RDS는 일부 데이터베이스 엔진의 ReadOnly Replica를 제공한다. MySQL은 아닌건가? **알아보기.**

### RDS 가격
* EC2위에 올린 데이터베이스보다 약 30% 더 비싸다. (!!!??)
    * 대신 해주는 기능들이 그만큼 많기 때문에


## RDS로 데이터 가져오기
### 마이그레이션
기존 온프레미스 데이터베이스를 AWS로 옮길 떄는 기존 데이터를 마이그레이션 해줘야 한다. 기존 데이터 베이스의 덤프를 **S3로 업로드하고 RDS로 해당 덤프를 가져와야** 한다.

#### 기존 데이터 내보내기
```bash
$ mysqldump -u $UserName -p --all-databases > dump.sql
# 일부 데이터베이스만 내보내기
$ mysqldump -u $UserName -p $DatabaseName > dump.sql
# 네트워크로 내보내기
$ mysqldump -u $UserName -p $DatabaseName --host $Host > dump.sql
```
위의 명령어로 mysql 데이터베이스에서 기존 데이터 덤프를 빼올 수 있지만, 일단 예제 데이터로 진행해보도록하자.

```bash
$ ssh -i [pem파일] [ec2주소]
# 예제 데이터 다운로드
$ wget https://s3.amazonaws.com/awsinaction/chapter9/wordpress-import.sql
```

#### 기존 데이터 RDS로 가져오기
일단 로컬에서 DBHostName을 확인한다.

```bash
$ aws rds describe-db-instances --query DBInstance[0].Endpoint
```
또는 aws console RDS의 Instance Detail에서 Endpoint 를 확인한다.

다시 EC2로 돌아가서, 위에서 찾은 DBHostName을 아래 명령어에 입력한다

```bash
$ mysql --host $DBHostName --user wordpress -p < wordpress-import.sql
```

#### 확인
wordpress 앱에 접속해서 마이그레이션된 데이터들을 확인해보자.

wordpress앱의 URL은 아래의 명령어로 확인할 수 있다. 또는 AWS Console의 CloudFormation 페이지에서 확인 가능하다.

```bash
# wordpress 앱 URL 확인
$ aws cloudformation describe-stacks --stack-name wordpress --query Stacks[0].Outputs[0].OutputValue --output test
```

## RDS 데이터베이스의 백업과 복원
RDS는 RDS 인스턴스의 특정 시점으로의 복원을 위한 수동/자동 **스냅샷** 기능을 제공한다.

* 자동화된 스냅샷의 보존 기간, 시간 구성
* 수동으로 스냅샷 생성
* 특정 스냅샷을 기반으로 새로운 데이터베이스의 인스턴스를 시작시키기
* 복구, 재배치를 위해 다른 리전에 스냅샷 복사하기

### 백업하기
#### 자동 스냅샷 구성하기
* 하루에 한번씩 실행. 하루뒤에 삭제. 보존기간 : 1~35일 사이의 값
* 스냅샷 생성중에는 모든 작업이 프리징 됨. 요청 오류가 발생할 수 있으므로 영향을 최소화 할 수 있는 시간대를 선택해야 함.
* CloudFormation에서의 `BackupRetentionPeriod`, `PreferredBackupWindow` 가 관련된 필드임.

```bash
# 자동 백업 시간 UTC 05:00~06:00 으로, 보존 기간을 3일로 변경하는 커멘드
$ aws cloudformation update-stack --stack-name wordpress --template-url https://s3.amazonaws.com/awsinaction/chapter9/template-snapshot.json --parameters ParameterKey=KeyName,UsePreviousValue=true ParameterKey=AdminPassword,UsedPreviousValue=true ParameterKey=AdminEMail,UsePreviousValue=true
```

#### 수동 스냅샷 생성하기

```bash
$ aws rds create-db-snapshot --db-snapshot-identifier wordpress-manual-snapshot --db-instance-indentifier awsinaction-db
```

```bash
$ aws rds describe-db-snapshots --db-snapshot-identifier wordpress-manaul-snapshot
```

#### 자동 스냅샷을 수동 스냅샷으로 복사하기
```bash
# 자동 스냅샷의 식별자 확인하기
$ aws rds describe-db-snapshots --snapshot-type automated --db-instace-identifier awsinaction-db --query DBSnapshots[0].DBSnapshotIdentifier --output test

# 교체하기
$ aws rds copy-db-snapshot --source-db-snapshot-identifier $SnapshotId --target-db-snapshot-identifier wordpress-copy-snapshot
```

### 복원하기
* 특정 스냅샷을 기준으로 복원하기

```bash
# 기존 데이터베이스의 서브넷 그룹 확인하기
$ aws cloudformation describe-stack-resource --stack-name wordpress --logical-resource-id DBSubnetGroup --query StackResourceDetail.PhysicalResourceId --output text
```

```bash
# 수동으로 생성했던 스냅샷으로 복원을 시켜보자.
$ aws rds restore-db-instance-form-db-snapshot --db-instance-identifier awsinaction-db-resotre --db-snapshot-identifier wordpress-manual-snapshot --db-subnet-group-name $SubnetGroup
```

자동 스냅샷을 사용한 경우, 특정 시점의 데이터베이스로 복원할 수 있다. RDS는 데이터베이스의 변경 로그를 유지하기 떄문.

```bash
# 특정 시간대의 스냅샷을 기준으로 변경 
$ aws rds restore-db-instance-to-point-in-time --target-db-instance-identifier awsinaction-db-restore-time --source-db-instance-identifier awsinaction-db --restore-time $Time --db-subnet-group-name $SubnetGroup
```

### 데이터베이스를 다른 리전으로 복사하기
스냅샷을 이용하면 다른 리전으로의 복사도 쉽게 할 수 있다.

* 재해 복구
* 리전 재배치

```bash
# us-east-1 에서 eu-west-1 으로 복사하는 커멘드
$ aws rds copy-db-snapshot --source-db-snapshot-identifier arn:aws:rds:us-east-1:$AccountId:snapshot:wordpress-manual-snapshot --target-db-snapshot-identifier wordpress-manual-snapshot --region eu-west-1
```

옮기는 리전에서도 옮길 데이터에 법적 문제가 없는지도 검토해야함ㅎㅎ

```bash
# AccountID가 생각나지 않을 경우
$ aws iam get-user --query "User.Arn" --output text
arn:aws:iam:123456789012:user/username   # 12자리 숫자가 AccountID 임
```

### 스냅샷 삭제하기
```bash
$ aws rds delete-db-instance --db-instance-identifier awsinaction-db-restore --skip-final-snapshot
$ aws rds delete-db-instance --db-instance-identifier awsinaction-db-restore-time --skip-final-snapshot
$ aws rds delete-db-instance --db-instance-identifier wordpress-manual-snapshot
$ aws rds delete-db-instance --db-instance-identifier wordpress-copy-snapshot
$ aws --region eu-west-1 rds delete-db-snapshot --db-snapshot-identifier wordpress-manual-snapshot
```

## RDS 데이터베이스 접근제어하기
AWS는 [책임 공유 모델(공동책임모델)](https://aws.amazon.com/ko/compliance/shared-responsibility-model/) 이므로 접근 제어에 대한 책임은 고객도 가져가게 된다. 접근 제어에 대한 규칙을 명확하게 지정해야 한다.

### RDS 데이터베이스 `구성`에 접근제어하기
IAM 서비스를 활용하여 제어.

### RDS 데이터베이스에 `네트워크` 접근제어하기
보안그룹으로 제어.

### 데이터베이스 자체의 `사용자 및 접근 관리`를 활용한 `데이터` 접근 제어하기
데이터베이스 엔진의 자체 접근제어 활용. 그렇기 떄문에 데이터베이스 시스템에 따라 다르다.


## 고가용성 데이터베이스 활용하기

* 2개의 인스턴스로 구성 : `마스터`와 `스탠바이`
    * 클라이언트는 마스터 데이터베이스에 리퀘스트 날림.
    * 데이터는 마스터와 스탠바이 데이터베이스 사이에서 동기적으로 복제됨
    * 마스터 데이터베이스에 장애가 생기면 장애조치 프로세스 실행 => 스탠바이 데이터베이스가 마스터 데이터베이스가 됨.
    * 마스터와 스탠바이는 서로 다른 가용영역(Availability Zone, AZ)에 배정된다. => 그래서 AWS에서는 고가용성 처리를 Multi Availability Zone 처리라고 부른다.
* RDS에서는 자동으로 장애조치를 처리함.

### 고가용성 배포 가능하도록 설정하기

CloudFormation에서 `MultiAZ` 필드를 `true`로 변경하면 된다.
아래의 커멘드를 실행하여 고가용성이 가능하게 수정하도록 하자.

```bash
# 기존 CloudFormation 템플릿을 고가용성 배포 가능하도록 수정하기
$ aws cloudformation update-stack --stack-name wordpress --template-url https://s3.amazonaws.com/awsinaction/chapter9/template-multiaz.json --parameters ParameterKey=KeyName,UsePreviousValue=true ParameterKey=AdminPassword,UsePreviousValue=true ParameterKey=AdminEmail,UsePreviousValue=true
```

## RDS 데이터베이스 성능 튜닝하기
* RDB는 수직적인 확장 : 하드웨어적인 성능 향상이 곧 확장
* NoSQL은 수평적인 확장 : 클러스터에 노드를 추가해서 성능을 높힐 수 있음.

### RDS 자원 늘리기
하드웨어 적인 성능향상이 곧 확장이기 때문에, 인스턴스 유형을 상위 유형으로 변경하면 확장임.

`DBInstanceClass`를 변경하면 됨

### 캐싱으로 Read 성능 개선하기
* RDB도 특정 상황에서는 수평적인 확장이 가능함
* 쓰기요청은 드물지만 읽기요청은 겁나게 많은 경우엔 읽기 전용으로 데이터베이스 인스턴스를 하나 복제하여 사용하는 것도 좋은 생각임.

```bash
# wordpress 블로그의 데이터베이스에 대한 읽기전용 복제 데이터베이스 생성하기
$ aws rds create-db-instance-read-replica --db-instance-identifier awsinaction-db-read --source-db-instance-identifier awsinaction-db
```

위의 동작은 내부적으로 스냅샷을 활용하여 복제를 한다.

읽기 전용으로 복제한 데이터베이스를 독립적인 데이터베이스로 승격시킬수도 있음

```bash
$ aws rds promote-read-replica --db-instance-identifier awsinaction-db-read
```

### ReadOnly Replica 클린업
```bash
$ aws rds delete-db-instance --db-instance-identifier awsinaction-db-read --skip-final-snapshot
```

## 전체 클린업
```bash
$ aws cloudformation delete-stack --stack-name wordpress
```
