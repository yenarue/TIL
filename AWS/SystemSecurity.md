시스템 보안
=======
IAM, 보안 그룹, VPC

AWS는 책임 공유 환경임. 사용자와 책임을 공유한다. 인프라에 관한 보안은 AWS가 책임지지만 권한 부여 등은 사용자가 책임져야 하는 구조.

# MFA (Multifactor authentication, 다단계 인증)
* Root 계정에는 MFA를 걸어두자.
* Root 계정에는 액세스 키를 걸지 않는 것이 좋다.
    * 대신 IAM 계정을 추가하여 액세스 키를 걸도록 하자.

# 사용자 (IAM)
Root 계정 대신 IAM 계정을 추가하여 액세스 하는 편이 더 안전하다.

## 계정 ID 확인하기
CLI에 연결된 계정의 ID를 확인할 수 있음.

```bash
$ aws iam get-user --query "User.Arn" --output text
arn:aws:iam::계정ID:user/username
```

## 정책
### ARN (Amazon Resource Name, 아마존 자원 이름)
계정ID로 ARN을 사용하여 서비스의 특정 자원으로의 접근을 허용할 수 있다.

### ARN 구조
`arn:aws:서비스:리전:계정ID:instance/자원`

### 정책 지정해보기
IAM의 `권한` -> `정책` 에서 Json 형태로 아래와 같이 ARN 지정 가능

```json
{
    "Version" : "2012-10-17",
    "Statement" : [{
        "Sid" : "2",
        "Effect" : "Allow",
        "Action" : ["ec2:TerminateInstances"],
        "Resource" : ["arn:aws:ec2:us-east-1:계정ID:instance/i-3dd4f812"] // ARN
    }]
}
```

## 그룹
정책을 그룹단위로 묶어 사용자를 그룹화시켜 관리할 수 있다.

```bash
$ aws iam create-group --group-name "admin"
$ aws iam attach-group-policy --group-name "admin" --policy-arn "arn:aws:iam::aws:policy/AdministratorAccess"
$ aws iam create-user --user-name "myuser"
$ aws iam add-user-to-group --group-name "admin" --user-name "myuser"

# 콘솔 로그인 가능하도록 패스워드 지정
# IAM 계정으로 로그인하려면 https://계정ID.signin.aws.amazon.com/console 로 로그인해야한다.
$ aws iam create-login-profile --user-name "myuser" --password "$Password"
```