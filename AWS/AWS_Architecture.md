AWS 기반 아키텍처 설계
====

> "모든 것은 오류를 낸다" - 아마존 CTO 워너보걸스

|              | 설명                                                         | AWS 서비스             |
| ------------ | ------------------------------------------------------------ | ---------------------- |
| 장애허용     | 다운타임 없이 자동으로 오류를 복구할 수 있음                 | S3, DynamoDB, Route 53 |
| 고가용성     | 짧은 다운타임 동안 자동으로 일부 오류를 복구할 수 있음       | RDS, EBS               |
| 수동장애처리 | 오류를 자동으로 복구하지 않지만 고가용성 인프라를 구축할 수 있는 도구를 제공함 | EC2                    |

* 작업부하를 기준으로 AWS상의 가상서버 수를 늘리는 방법
* 신뢰할 수 있는 시스템을 설계하는 방법



# 고가용성 달성하기: 가용영역, 오토스케일링, 클라우드 와치

데이터센터 손상 위험에 대응하기 위해 다른 데이터센터에 있는 단일 EC2 인스턴스를 복구하는 방법을 알아본다.

* EC2는 기본적으로 고가용성이 아님

## 고가용성

다운타임(downtime) 없이 운영되는 시스템.

장애가 발생하더라도 높은 확률로 지속적인 서비스 제공 가능하다. (Harvard Research Group : 99.99% uptime / 1 year)

### vs 장애허용

* 고가용성 : 짧은 다운타임동안 자동으로 오류를 복구한다. (서버가 알아서 복구함)
* 장애허용 : 결함이 발생해도 중단하지 않고 그냥 서비스를 제공한다. (서버가 죽지 않음)



## AWS에서 고가용성 적용하기

EC2인스턴스를 기반으로 고가용성 시스템을 구축할 수 있다.

* 클라우드와치 : 가상서버의 상태 모니터링. 자동 복구 설정하기
* 고가용성 인프라 구축하기 : 1 리전당 여러개의 가용영역 설정하기
* 오토스케일링 : 일정 수 이상의 가상서버가 실행되도록 보장하기. 장애 발생한 인스턴스는 자동으로 교체시키기.



## 서버 장애 복구하기 - [클라우드와치 CloudWatch](https://console.aws.amazon.com/cloudwatch/)

AWS 리소스에 대한 지표, 로그, 알람을 제공하는 서비스.

###클라우드와치 알람 생성해보기

#### 구성요소

* 지표 : 어떤 데이터를 모니터링 할 것인가 (헬스 체크, CPU 사용량 등...)
* 규칙 : 임계값을 정의하는 규칙. 어떤 상황에 어떤 경보를 울릴 것인가? (기간, 통계함수 등..)
  * 경보종류 : OK, INSUFFICIENT_DATA, ALARM
* 액션 : 알람상태 변경 시 트리거되는 액션. 경보가 울리면 어떤행동을 할 것인가? (인스턴스 복구,종료 등...)

#### 헬스 체크하는 알람 생성하기

EC2 인스턴스의 헬스 상태를 체크해서 복구시키는 알람을 생성해보자.

##### 시스템 상태 검사 (System Status Check)

네트워크 연결, 전력 손실, 호스트의 소프트웨어/하드웨어 문제 검출

- EC2 서비스는 모든 가상서버의 상태를 자동으로 매 분마다 검사함. => 클라우드와치에서 지표로 사용
  - 가상서버와 EC2의 차이? : EC2 서비스로 가상서버를 만드는 거임... 하나의 EC2로 여러개의 인스턴스를 관리할수이씀. EC2만들때 인스턴스 개수 정하라는 메뉴가 뜸. 추후 오토스케일링 가능.

- EC2 인스턴스 복구를 위한 요구사항들
  - VPC 네트워크에서 실행해야 함
  - `t1`은 제공하지 않음

##### 알람 생성 방법

* CloudFomration템플릿 : [recovery.json](https://github.com/AWSinAction/code/blob/master/chapter11/recovery.json)

* AWS Console 페이지 : EC2 인스턴스의 디테일 페이지에서 상태지표 추가하기

##### [실습] AWS Console 페이지 이용하여 알람 생성하기

- EC2 상세페이지 > `상태검사` > `상태 검사 경보 생성`

![](/Users/yenarue/Developer/TIL/AWS/images/ha_create_alarm.png)

- CloudWatch 상세 페이지 : 임계값 확인 -> `StatusCheckFailed_System`로 체크하고있음

![](/Users/yenarue/Developer/TIL/AWS/images/ha_status_check_failed.png)

- 물리서버에서 오류 발생 -> EC2 상태 이상 감지 -> 클라우드와치 지표에 오류 보고 -> 클라우드와치 알람이 가상서버 복구 실행 -> 타 물리호스트에서 가상서버 시작 -> EBS볼륨, IP 유지하며 새로운 가상서버에 연결 
  - 당연히 인스턴스 스토리지는 복구되지 않음.
  - 일래스틱 IP 써야 유지됨.

### [실습] 젠킨스에 알람을 붙여서 고가용성 CI System 만들어보기

#### 순서 - [CloudFormation](https://github.com/AWSinAction/code/blob/master/chapter11/recovery.json)

1. VPC 생성
   * `Resources.VPC` 참고
2. Elastic IP 사용 설정
   * `ElasticIP`참고
3. VPC에서 가상서버 실행 및 부팅 시 자동으로 젠킨스 설치
   * `Server.Properties.UserData` 참고
4. 가상서버의 헬스 상태 모니터링 클라우드와치 알람 생성
   * RecoveryAlarm` 참고

##### CloudFormation 스택 생성

```bash
$ aws cloudformation create-stack --stack-name jenkins-recovery --template-url https://s3.amazon.com/awsinaction/chapter11/recovery.json --parameters ParameterKey=JenkinsAdminPassword,ParameterValue=$Password
```

##### CloudFormation 스택 상태 확인

젠킨스 설치까지 되려면 좀 오래걸림. 상태를 폴링하여 완료되었는지 체크하자.

```bash
$ aws cloudformation describe-stacks --stack-name jenkins-recovery --query Stacks[0].Outputs
```

젠킨스 URL, Admin ID/PW가 출력된다면 완료된 것이다.

##### CloudWatch 알람 설정 확인하기

AWS Console > CloudWatch 에 접속하여 Alarm이 잘 설정되어는지 확인하자

##### 클린업

```bash
$ aws cloudformation delete-stack --stack-name jenkins-recovery
// DELETE_COMPLETE 확인하기
$ aws cloudformation describe-stacks --stack-anme jenkins-recovery
```



## 데이터센터 중단시 복구하기

이전에 실습한 인스턴스 복구는 동일 데이터센터 내에서 새 인스턴스로 복구해주는 것이다. **데이터센터 자체가 죽으면 복구를 못한다.** 이 경우엔 어떻게 할까...

#### AWS Region의 가용영역

AWS Region은 여러 데이터센터로 구성되어 있다. 이를 가용영역이라고 부른다.

* 각 리전별로 가용영역의 개수가 다름
* 동일 가용영역 내에 있는 데이터센터끼리는 요청/응답이 빠름

가용영역으로 인해 아래와 같은 것들이 가능하다.

* 오토스케일링
* 데이터센터 장애 복구

다만, 타 데이터센터로 복구할 경우엔 아래와 같은 **함정**들을 조심하자.

* EBS 복구 불가능 : 데이터센터가 아예 옮겨지는 것 이므로..
* 동일한 사설 IP로 다른 데이터센터에 새로운 가상서버를 시작할 수 없음 : 언제 다른 데이터센터로 이동하게 될지 모르니까
* 동일한 공인 IP를 유지할 수 없음 : 일래스틱IP써도 안되나? 클라우드와치로 인스턴스 복구했을땐 일래스틱IP쓰면 공인 IP 유지 가능했음.

##### AWS 타 서비스와의 연동

* 여러 리전에 걸쳐 운영하는 서비스 : Route 53 (DNS), CloudFront (CDN)
* 하나의 리전에 여러 가용영역을 사용하여 데이터센터 장애 복구 가능한 서비스 : S3, DynamoDB
* RDS : Multi-AZ 배포를 사용하여 다른 가용영역으로 장애조치를 할 수는 있다. (마스터-스탠바이 설치 배포)
* EC2 : 단일 가용영역에서 실행됨. 대신, 다른 가용영역으로 장애조치를 할 수 있는 EC2 인스턴스 기반 아키텍처 구축 도구를 제공함.

##### AWS Region 관련 CLI Commands

```bash
// Region 전체 리스트 출력
$ aws ec2 descfribe-regions
// 특정 Region의 모든 가용영역 리스트 출력
$ aws ec2 describe-availablity-zones --region $Region
```

##### 주의사항

여러 가용영역으로 장애조치를 하는 EC2 인스턴스 기반 고가용성 아키텍처를 생성할 때 주의해야 할 점들

* VPC는 항상 리전에 바인딩된다
* VPC내부의 서브넷은 가용영역에 연결되어있다
* 가상서버는 단일 서브넷으로 시작된다



### 가상서버가 항상 실행되도록 하기 - 오토스케일링

EC2 서비스의 일부. 일정 수의 가상서버가 실행되고 있는지 확인 할 수 있다. 이 기능들을 이용하여 고가용성을 구현할 수 있다.

* 가상서버 실행
* 기존 서버 실패시 새로운 가상서버 구동
* EC2 인스턴스를 여러 서브넷에서 구동
* 시스템 사용량에 따라 가상서버 수 늘리기 (14장에서 한다고 한다)

#### 오토스케일링 구성하기

EC2에 아래와같은 내용을 구성에 포함하면 된다. [참고 - multiaz.json](https://github.com/AWSinAction/code/blob/master/chapter11/multiaz.json)

* 시작 구성 (`LaunchConfiguration`) : 가상 서버 시작에 필요한 모든 정보 (인스턴스 타입, 구동할 AMI 이미지 등..)
* 오토스케일링 그룹 구성 (`AutoScalingGroup`) : 가상 서버의 수, 인스턴스 모니터링 방법, 서브넷 등

#### [실습] 젠킨스 서버에 오토스케일링 적용하기

CloudFormation 템플릿 : [multiaz.json](https://github.com/AWSinAction/code/blob/master/chapter11/multiaz.json)

##### CloudFormation 스택 생성

오토스케일링이 적용된 젠킨스 서버를 생성하는 CloudFormation 을 실행한다

```bash
// 오토스케일링 적용 - 다른 가용영역에서 복구할 수 있는 가상서버 생성
$ aws cloudformation create-stack --stack-name jenkins-multiaz --template-url https://s3.amazonaws.com awsinaction/chapter11/multiaz.json --parameters ParameterKey=JenkinsAdminPassword,ParameterValue=$Password
```

##### CloudFormation 스택 생성 완료 후 젠킨스 정보 확인하기

젠킨스 서버의 공인 IP 주소를 확인하여 정상동작하는지 접속해보자.

```bash
$ aws ec2 describe-instances --filters "Name=tag:Name,Values=jenkins-multiaz" "Name=instance-state-code,Values=16" --query "Reservations[0].Instances[0].[InstanceId, PublicIpAddress, PrivateIpAddress, SubnetId]"
[
    "instanceId",
    "PublicIP",
    "PrivateIP",
    "subnetId"
]
```

##### 오토스케일링 복구프로세스 테스트

가상서버를 종료시켜보자

```bash
$ aws ec2 terminate-instances --instance-ids $InstanceId
```

잠시 뒤 오토스케일링 그룹이 가상서버가 종료되고 새로운 가상 서버가 시작되는 것을 감지할 것이다. 

이를 확인하기 위해 새로 시작된 가상서버가 출력될 때 까지 아래 명령을 계속 실행해보자.

```bash
$ aws ec2 describe-instances --filters "Name=tag:Name,Values=jenkins-multiaz" "Name=instance-state-code,Values=16" --query "Reservations[0].Instances[0].[InstanceId, PublicIpAddress, PrivateIpAddress, SubnetId]"
[
    "new-instanceId",
    "new-PublicIP",
    "new-PrivateIP",
    "new-subnetId"
]
```

새로 실행된 가상서버의 공인 IP로 접속해서 정상 복구되었는지 확인해보자.

[축] 고가용서 아키텍처를 구축했다!

##### 문제점

데이터센터간 이동이기 때문에 AWS Region 에서 언급한 함정들이 존재한다.

* **디스크 데이터(EBS)의 손실** : 젠킨스는 데이터에 값을 저장하고 있다. 오류 복구를 위해 새로운 가성서버가 생성되면 데이터가 손실된다.
* **공용/사설 IP의 변경** : 동일한 엔드포인트를 사용할 수 없다.



### 함정 극복하기

#### 1. EBS 복구

* 여러 가용영역을 사용하는 관리 서비스로 변경하기 : RDS, DynamicDB, S3
* EBS 볼륨의 스냅샷을 생성하고 S3에 저장해둔 후 복구하기
* 서드파티 스토리지 사용 : GlusterFS, DRBD, MongoDB 등...

##### EBS 스냅샷

1. 젠킨스 서버에 대한 작업추가 (https://$JenkinsURL/newjob)

2. 가상서버 현재상태의 스냅샷으로 AMI 생성

   ```bash
   $ aws ec2 create-image --instanceId $InstanceId --name jenkins-multiaz
   {
       "imageId": $imageId
   }
   
   ## 현재 상태 확인 : AMI를 사용할 수 있을 때까지 대기하자
   $ aws ec2 describe-images --image-id $ImageId --query "Images[].State"
   ```

3. 시작 구성 갱신

   2번 과정에서 생성한 AMI 스냅샷으로 시작하도록 CloudFormation의 시작구성을 변경한다

   ```bash
   $ aws cloudformation update-stack --stack-name jenkins-multiaz --template-url https://s3.amazonaws.com/awsinaction/chapter11/multiaz-ebs.json --parameters ParameterKey=JenkinsAdminPassword,UsePreviousValue=true ParameterKey=AMISnapshot,ParameterValue=$ImageId
   
   ## UPDATE_COMPLETE 가 될 때 까지 대기하자
   $ aws cloudformation describe-stacks --stack-name jenkins-multiaz
   ```

4. 복구테스트

   이전에 했던 것 처럼 강제종료 후 상태를 확인해보자!

   ```bash
   $ aws ec2 terminate-instances --instance-ids $InstanceId
   
   $ aws ec2 describe-instances --filters "Name=tag:Name,Values=jenkins-multiaz" "Name=instance-state-code,Values=16" --query "Reservations[0].Instances[0].[InstanceId, PublicIpAddress]"
   ```

   마지막 명령어에서 출력된 PublicIP로 다시 젠킨스에 접속해보면 이전에 추가했던 작업이 그대로 남아있음을 알 수 있다. EBS 복구 완료!

5. 클린업

   여전히 PublicIP는 바뀐다… 이를 복구하기 위해서는 다른 방법을 사용할 것이니 일단 여태까지 한 것들을 클린업하자

   ```bash
   # ImageId, SnapshotId
   $ aws ec2 describe-images --owners self --query Images[0].[ImageId,BlockDeviceMappings[0].Ebs.SnapshotId]
   
   $ aws cloudformation delete-stack --stack-name jenkins-multiaz
   # DELETE_COMPLETE가 뜰때까지 확인
   $ aws cloudformation describe-stacks --stack-name jenkins-multiaz
   $ aws ec2 deregister-image --image-id $ImageId
   $ aws ec2 delete-snapshot --snapshot-id $SnapshotId
   ```


#### 2. Network Interface 복구

일래스틱 IP도 못쓴다... ㅠㅠ?

고가용성 구축을 위해 오토스케일링 사용시, 정적 엔드포인트를 제공할 수 있는 몇가지 방법이 있다.

* **일래스틱IP**를 할당하고 가상 서버 부팅 시 이 공인 IP주소를 연결한다.
* **Route 53** : 가상 서버의 현재 공용/사설 IP 주소로 연결하는 DNS 항목을 생성/갱신한다.
* **일래스틱 로드 밸런서 (ELB)** : 현재 가상 서버로 오는 요청을 포워딩하는 정적 엔드포인트를 사용한다. => 12장 참고

##### 일래스틱 IP 할당하기

1. CloudFormation Stack 생성
   일래스틱 IP를 할당하는 구성이 추가된 [CloudFormation](https://github.com/AWSinAction/code/blob/master/chapter11/multiaz-elasticip.json)으로 스택을 재생성하자.

   ```bash
   $ aws cloudformation create-stack --stack-name jenkins-multiaz --template-url https://s3.amazonaws.com awsinaction/chapter11/multiaz-elasticip.json --parameters ParameterKey=JenkinsAdminPassword,ParameterValue=$Password --capabilities CAPABILITY_IAM
   ```
2. CloudFormation 스택 생성 확인 및 젠킨스 설정 확인

   ```bash
   $ aws cloudformation describe-stacks --stack-name jenkins-elasticip --query Stacks[0].Outputs
   ```

3. 가상 서버의 인스턴스 ID 확인

   ```bash
   $ aws ec2 describe-instances --filters "Name=tag:Name,Values=jenkins-elasticip" "Name=instance-state-code,Values=16" --query "Reservations[0].Instances[0].InstanceId" --output text
   ```

4. 복구테스트

   ```bash
   $ aws ec2 terminate-instances --instance-ids $InstanceId
   
   $ aws ec2 describe-instances --filters "Name=tag:Name,Values=jenkins-multiaz" "Name=instance-state-code,Values=16" --query "Reservations[0].Instances[0].[InstanceId, PublicIpAddress]"
   ```

5. 클린업

   ```bash
   $ aws cloudformation delete-stack --stack-name jenkins-multiaz
   # DELETE_COMPLETE가 뜰때까지 확인
   $ aws cloudformation describe-stacks --stack-name jenkins-multiaz
   ```


## 재해 복구 요구사항 분석하기

* 복구 시간 목표 (RTO)
* 복구 지점 목표 (RPO)

### 복구 시간 목표 (Recovery Time Objective)

시스템이 오류를 복구하는데 걸리는 시간 = 서비스 중단 후 서비스를 다시 제공하기까지의 시간

### 복구 지점 목표 (Recovery Point Objective)

오류 발생시 수용할 수 있는 데이터 손실 시간.

데이터 손실의 양은 시간으로 측정됨. (오전 10시에 정전되어 오전 9시의 스냅샷으로 복구가되면 1시간 범위의 데이터 손실이 일어났다고 표현함)

### 클라우드와치 vs 오토스케일링

* 클라우드와치로 서버 장애 복구
* 오토스케일링으로 데이터센터 장애 복구

|                                       | RTO     | RPO                                   | 가용성                                                   |
| ------------------------------------- | ------- | ------------------------------------- | -------------------------------------------------------- |
| 클라우드와치 알람으로 트리거되어 복구 | 약 10분 | 데이터 손실 없음                      | - 가상 서버 오류 복구 O<br />- 전체 가용영역 장애 복구 X |
| 오토스케일링으로 복구                 | 약 10분 | 마지막 스냅샷 이후의 모든 데이터 손실 | - 가상 서버 오류 복구 O<br />- 전체 가용영역 장애 복구 O |

여기서 더더더 RPO를 줄여야 한다? => 답은 **무상태 서버**다!

최대한 EBS대신 RDS, S3, DynamoDB를 쓰자! 