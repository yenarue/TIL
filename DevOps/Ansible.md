프로비저닝 도구
=====
인프라 구축 작업을 자동화하는 구성 관리 도구.

# 현황
프로비저닝 도구에는 대표적으로 Ansible, Chef, Puppet 이 존재한다.

[Ansible vs Chef vs Puppet 구글 트렌드를 확인](https://trends.google.co.kr/trends/explore?cat=5&date=today%205-y&q=ansible,chef,puppet)해보니 최근에는 Ansible이 대세인 듯 하다.

![구글 검색어 트랜드 - Ansible vs Chef vs Puppet](./images/googletrend_ansible_vs_chef_vs_puppet.png)

각 도구들의 구체적인 차이점은 다음과 같다.

 도구명 | 개발언어 | 정의 | Agent/SSH | 통신방법 | github와의 연동성 
------ | ------ | -------- | -------- | -------- | --------
 puppet | ruby | DSL | Agent | http ssl | 중 
 chef | ruby | DSL | Agent | Reset / STOMP | 중 
 ansible | python | YAML | SSH | json | 상 



# Ansible이란?
Python 으로 구현된 오픈소스 IT 자동화 도구. 쉽게 말하자면 **서버에 SSH 로그인하여 명령을 실행하는 형태의 작업들을 자동화**할 수 있도록 도와주는 툴이다.
* 대부분이 멱등성을 제공함.
    * 멱등성을 제공하지 않는 모듈들 : shell, command, file module)
* Agent-less
    * 굳이 Agent 도구를 따로 설치할 필요 없이 SSH로 접속하여 실행 가능.
* 이해하기 쉽다 (SSH, YAML)
    * `playbook`(ansible에서 태스크를 기술하는 파일)이 YAML 형식으로 되어있어 읽기 편하다.
* 다양한 모듈
* 애드혹 (Ad-hoc) 명령
    * 일일히 `playbook`을 작성하지 않아도 `ansible`명령으로 단일 모듈만을 실행할 수 있다.

## 이용 환경
![Ansible 구축 모습](./images/ansible-hosts.png)
* 실행 호스트 : Windows 환경 지원 X, Python 2.6 이상 (Python3은 지원 X)
* 대상 호스트 : SSH로 접속가능한 모든 OS 지원, Python 2.4 이상 (Python3은 지원 X)

# Ansible 사용해보기
* https://github.com/shin1x1/gihyo-devops-ansible

## 사전 준비
### VirtualBox & Vagrant 설치
1. [Virtual Box](https://www.virtualbox.org/wiki/Linux_Downloads)
1. [Vagrant 설치](https://www.vagrantup.com/downloads.html)

### Ansible 설치
```bash
$ sudo apt-get update
$ sudo apt-get install software-properties-common

$ sudo apt-add-repository ppa:ansible/ansible

$ sudo apt-get update
$ sudo apt-get install ansible
```

### Vagrant 로 가상서버 구축
1. `Vagrantfile` 생성
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
    config.ssh.insert_key = false
    config.vm.box = "bento/centos-7.1"
    config.vm.hostname = "yenarue-devops-ansible.dev"
    config.vm.network "private_network", ip: "192.168.34.21"
end
```
2. vagrant 업로드 후 ssh 접속
```bash
$ vagrant up
$ vagrant ssh
Last login: Sun Apr 15 15:59:45 2018 from 10.0.2.2
[vagrant@yenarue-devops-ansible ~]$
```

## `ansible.cfg`
ansible 실행시 참조되는 컨피그 파일은 다음과 같은 우선순위로 참조된다.
[공식매뉴얼](docs.ansible.com/ansible/intro_configuration.html)

1. 환경변수 `ANSIBLE_CONFIG`에 지정된 경로
2. `.ansible.cfg`
3. `~/.ansible.cfg`
4. `/etc/ansible/ansible.cfg`

## 인벤토리
태스크를 실행할 대상 호스트의 정보를 기술하는 텍스트 파일.

```bash
# inventoryfile
192.168.34.21 ansible_ssh_user=vagrant ansible_ssh_private_key_file=~/.vagrant.d/insecure_private_key
```

변수 | 내용
-----|-----
ansible_ssh_host | 대상 호스트명
ansible_ssh_port | SSH 접속 포트
ansible_ssh_user | SSH 로그인 사용자명
ansible_ssh_private_key_file | 공개키 인증 비밀키 파일
ansible_ssh_pass | 패스워드 인증할 때의 패스워드

### 호스트 별명 지정
FQDN이나 IP대신에 다른 별명을 지정할 수 있다.

```bash
# inventoryfile-alias
vm1 ansible_ssh_host=192.168.34.21 ansible_ssh_user=vagrant ansible_ssh_private_key_file=~/.vagrant.d/insecure_private_key
```

### 그룹 지정
호스트를 그룹핑 할 수 있다. 역할별/종류별 등등...
즉 그룹별로 설정을 다르게 할 수 있다는 뜻이다. development 그룹 / production 그룹으로 나눠 사용하면 개이득.

```bash
# inventoryfile-group
# 아래와 같이 alias 설정을 해줘야 인식을 한다.
vm1 ansible_ssh_host=192.168.34.21 ansible_ssh_user=vagrant ansible_ssh_private_key_file=~/.vagrant.d/insecure_private_key

[web]
vm1
web01.example.com
web02.example.com
web03.example.com

[db]
vm1
db_a.example.com
db_b.example.com

[development]
vm1

[production]
web01.example.com
web02.example.com
web03.example.com
db_a.example.com
db_b.example.com
```

아래는 위와 같은 표현이다
```bash
# inventoryfile-group-short
vm1 ansible_ssh_host=192.168.34.21 ansible_ssh_user=vagrant ansible_ssh_private_key_file=~/.vagrant.d/insecure_private_key

[web]
vm1
web[01:03].example.com

[db]
vm1
db_[a:b].example.com

[development]
vm1

[production]
web[01:03].example.com
db_[a:b].example.com
```

#### 그룹에 변수 지정하기
그룹 내 호스트들에 대해 일괄적으로 변수 지정 가능.

```bash
# inventoryfile-group-variables
[groupA]
host-a.example.com
host-b.example.com

[groupA:vars]
ansible_ssh_user=vagrant
ansible_ssh_private_key_file=~/.vagrant.d/insecure_private_key
```

### 다이나믹 인벤토리
AWS의 EC2나 VPC와 같은 정보를 동적으로 얻는 스크립트 => 이용해서 동적으로 변경되는 인벤토리를 만들면 된다.
* [ec2.py](https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/ec2.py)
* [ex2.ini](https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/ec2.ini)

## Ansible 실행
ansible의 대상 호스트로, 아까 준비했던 vagrant를 올려놓고 (`vagrant up`) ansible을 실행해보자!

`ansible` 명령으로 실행한다.
### `ansible`의 옵션들
자세한 건 man 페이지 참조.
* `-i` : 인벤토리 지정
* `--list-hosts` : 지정된 호스트들의 목록 출력
* `-m` : 모듈 지정
    * 모듈이란? 대상 호스트에서 실행할 처리를 구현한 라이브러리. [Ansible이 지원하는 모듈 리스트](http://docs.ansible.com/ansible/latest/modules/list_of_all_modules.html)
    * `-a` : 모듈의 매개변수
* `-v` : verbose 옵션. 상세 표시.
    * `-vv`, `-vvv` ... : `v`를 중첩시킬수록 메세지가 상세해짐.


```bash
# 모든 호스트
$ ansible all -i inventoryfile-group-short --list-hosts
  hosts (6):
    vm1
    web01.example.com
    web02.example.com
    web03.example.com
    db_a.example.com
    db_b.example.com

# web 호스트 그룹만 지정
$ ansible web -i inventoryfile-group-short --list-hosts
  hosts (4):
    vm1
    web01.example.com
    web02.example.com
    web03.example.com

# web01.example.com 호스트만 지정
$ ansible web01.example.com -i inventoryfile-group-short --list-hosts
  hosts (1):
    web01.example.com

```

### 모듈 적용해보기
```bash
# shell 모듈로 쉘을 띄우고 uname 명령 날림
$ ansible vm1 -i inventoryfile-alias -m shell -a 'uname -a'
vm1 | SUCCESS | rc=0 >>
Linux yenarue-devops-ansible.dev 3.10.0-229.el7.x86_64 #1 SMP Fri Mar 6 11:36:42 UTC 2015 x86_64 x86_64 x86_64 GNU/Linu
```

### Verbose 옵션 적용해보기
```bash
$ ansible vm1 -i inventoryfile-alias -m shell -a 'uname' -v
Using /etc/ansible/ansible.cfg as config file
vm1 | SUCCESS | rc=0 >>
Linux

# 메세지 출력이 보다 상세해짐
$ ansible vm1 -i inventoryfile-alias -m shell -a 'uname' -vv
ansible 2.5.0
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/yenarue/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.12 (default, Nov 20 2017, 18:23:56) [GCC 5.4.0 20160609]
Using /etc/ansible/ansible.cfg as config file
META: ran handlers
vm1 | SUCCESS | rc=0 >>
Linux

META: ran handlers
META: ran handlers
```

### `yum` 모듈을 이용하여 패키지 갱신하기
가장 유용하게 쓰일 것 같음. 여러개의 호스트에 동시에 패키지 업데이트 가능.
```bash
$ ansible all -i hosts -b -m yum -a "name=openssl state=latest"
```

# Ansible Playbook
대상 호스트에서 실행할 태스크를 기술한 YAML 파일.
* 하나의 모듈 실행 : `ansible` 실행
* 복수개의 모듈 실행 : `ansible-playbook` 실행

## Ansible Playbook 일단 한번 사용해보기
### Apache 설치 및 실행해보기
```yaml
# playbook-install-apache.yml
---
- hosts: vm1 # 192.168.34.21 (inventoryfile-alias에서 정의)
  become: true
  tasks:
    - name: Apache 설치
      yum: name=httpd state=latest

    - name: Apache 실행
      service: name=httpd state=started enabled=yes
```

```bash
$ ansible-playbook -i inventoryfile-alias playbook-install-apache.yml 
PLAY [vm1] *****************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************
ok: [vm1]

TASK [Apache 설치] ***********************************************************************************************************************
changed: [vm1]

TASK [Apache 실행] ***********************************************************************************************************************
changed: [vm1]

PLAY RECAP *****************************************************************************************************************************
vm1                        : ok=3    changed=2    unreachable=0    failed=0  
```
### 결과
vm1(192.168.34.21)에 80포트로 접속해보면 Apache가 정상적으로 설치 및 실행되었음을 알 수 있다.
![](./images/ansible-playbook-install-apache-result.png)

## Playbook 작성 규칙 (구문 문법)
```yaml
--- # 헤더
- hosts: vm1 # = hosts: "vm1"

# 리스트
- value1
- value2
[value1, value2] # 한 행에 전부 적을 때

# 딕셔너리
key1: value1
key2: value2
key3: a b c

{ key1: value1, key2: value2, key3: a b c }

# 논리값
## 참 : true, on, yes
## 거짓 : false, off, no
become: true
become: off

# 들여쓰기
## 딕셔너리, 리스트에서 들여쓰기를 하면 중첩됨
### 1) 리스트
- level-1
    - level-1-1
    - level-1-2
        - level-1-2-1
### JSON으로 나타내면....
### ["level-1", ["level-1-1", "level-1-2", ["level-1-2-1"]]]
### 2) 딕셔너리
level1:
    level1-1: value1
    level1-2: value2
### JSON으로 나타내면....
### ["level1", ["level1-1" : "value1", "level1-2" : "value2"]]

```

## Playbook의 구조
### host
inventory에 포함된 호스트를 지정한다.
특정 호스트나 호스트 그룹을 지정할 수 있다.

### tasks
실행할 태스크를 지정한다. 각 태스크는 딕셔너리로 기술된다.

```yaml
tasks:
    - name: Apache 설치
      yum: name="httpd" state="latest"

## 위와 같음
tasks:
    - name: Apache 설치
      yum:
        name: httpd
        state: latest

tasks:
    - name: Apache 설치
      yum:
        name=httpd
        state=latest
```

### handlers
task 실행 후 실행할 핸들러를 지정한다. status가 변경되면 (=처리 결과가 changed면) task의 notify에 지정된 핸들러가 실행된다. 동일한 핸들러가 여러 태스크에서 호출되더라도 단 한번만 실행된다.

```yaml
tasks:
    - name: 설정 파일 전송
      copy: src="files/httpd.conf" dest="/etc/httpd/conf/httpd.conf"
      notify: httpd 재실행
handelrs:
    - name: httpd 재실행
      services: name="httpd" state="restarted"
```

### 변수 선언 및 정의 하기
파라미터, 결과값 등을 변수로 저장할 수 있다.
#### 인벤토리에서 변수 선언
inventoryfile-group-variables 참고

#### Playbook에서 변수 선언
```yaml
---
- hosts: vm1
  vars:
    key1: value1
    key2: value2
- hosts: web
  vars_files:
    - extra_vars.yml
```

```yaml
---
# extra_vars.yml
key1: value10
key2: value20
```
* `ansible-playbook` 커멘드의 `-e`옵션 이용
`-e(--extra-vars)` 옵션으로 변수에 json형식으로 값을 넣는다. 이 방법으로 생성된 변수가 가장 높은 우선순위를 가진다.

```bash
$ ansible-playbook -i inventory playbook.yml -e "key1=value100 key2=value200"
```

변수 파일을 등록하고 싶을때는,`@`를 붙이면 된다. json파일만 가능하다.

```bash
$ ansible-playbook -i inventory playbook.yml -e "@extra_vars.json"
```

#### 태스크의 실행 결과 값을 변수에 설정하기
태스크 정의시 `register` 키워드를 사용하면 실행결과를 변수로 설정할 수 있다.

```yaml
---
- hosts: vm1
  become: true
  tasks:
    - name: 오늘 날짜를 얻어보자
      shell: date +%Y%m%d
      register: date

    - debug: var=date # date변수 출력

    - name: 오늘 날짜 디렉터리를 생성
      file: path=/tmp/{{ date.stdout }} state=directory owner=vagrant group=vagrant mode=0755
```

```bash
$ ansible-playbook -i inventoryfile-alias playbook-task-return-value.yml

PLAY [vm1] *****************************************************************************************

TASK [Gathering Facts] *****************************************************************************
ok: [vm1]

TASK [오늘 날짜를 얻어보자] *********************************************************************************
changed: [vm1]

TASK [debug] ***************************************************************************************
ok: [vm1] => {
    "date": {
        "changed": true, 
        "cmd": "date +%Y%m%d", 
        "delta": "0:00:00.002061", 
        "end": "2018-05-09 13:07:05.466456", 
        "failed": false, 
        "rc": 0, 
        "start": "2018-05-09 13:07:05.464395", 
        "stderr": "", 
        "stderr_lines": [], 
        "stdout": "20180509", 
        "stdout_lines": [
            "20180509"
        ]
    }
}

TASK [오늘 날짜 디렉터리를 생성] ******************************************************************************
changed: [vm1]

PLAY RECAP *****************************************************************************************
vm1                        : ok=4    changed=2    unreachable=0    failed=0
```

### 변수 값 참조하기
[Jinja2](http://jinja.pocoo.org/) 형식으로 작성하여 변수 값을 참조한다.

`{{ 변수명 }}`와 같은 형식...


#### 딕셔너리 변수
도트 구분자나 대괄호로 참조 가능

```yaml
# vars:
#   dictionary:
#       key1:
#           key2: value10

{{ dictionary.key1.key2 }}      # value10
{{ dictionary["key1"}["key2"]}  # value10
```

#### 리스트 변수
인덱스 지정해서 참조 가능. 문자열도 마찬가지 (char의 리스트니까)

```yaml
# vars:
#   list:
#       - head
#       - value1
#       - tail
{{ list[0] }}   # 첫번째
{{ list[-1] }}  # 마지막
{{ list[-2] }}  # 끝에서 두번째

{{ list[0:2] }} # 처음~두번째
{{ list[:2] }}  # ~두번째 (위와 같다)
{{ list[1:] }}  # 두번재~마지막
```

#### 변수 값 참조하는 playbook 사용해보기
```yaml
# playbook-copyfile.yml
---
- hosts: vm1 
  become: true
  tasks:
#  - name: 파일을 만든다
#    file:
#      path: 'testfile'
#      state: touch
#      mode: 0777

  - name: 파일을 복사한다
    copy: 
      src: 'copyTestFile' # 로컬경로여야 함
      dest: '{{ file_destination }}'

  - name: 복사된 파일을 확인한다
    find:
      path: './'
      patterns: '{{ file_destination }}'
    register: find_result

  - debug: var=find_result
```

``` bash
ansible-playbook -i inventoryfile-alias playbook-copyfile.yml -e "file_destination='copyDescFile'"
Using /etc/ansible/ansible.cfg as config file

PLAY [vm1] *****************************************************************************************

TASK [Gathering Facts] *****************************************************************************
ok: [vm1]

TASK [파일을 복사한다] ************************************************************************************
ok: [vm1] => {"changed": false, "checksum": "fcff57d112692c8f1c3d330714358c59f1c268cc", "dest": "copyDescFile", "gid": 0, "group": "root", "mode": "0644", "owner": "root", "path": "copyDescFile", "secontext": "unconfined_u:object_r:user_home_t:s0", "size": 9, "state": "file", "uid": 0}

TASK [복사된 파일을 확인한다] ********************************************************************************
ok: [vm1] => {"changed": false, "examined": 10, "files": [{"atime": 1525883689.7708306, "ctime": 1525883574.9308305, "dev": 64768, "gid": 0, "gr_name": "root", "inode": 67586701, "isblk": false, "ischr": false, "isdir": false, "isfifo": false, "isgid": false, "islnk": false, "isreg": true, "issock": false, "isuid": false, "mode": "0644", "mtime": 1525883574.7018306, "nlink": 1, "path": "copyDescFile", "pw_name": "root", "rgrp": true, "roth": true, "rusr": true, "size": 9, "uid": 0, "wgrp": false, "woth": false, "wusr": true, "xgrp": false, "xoth": false, "xusr": false}], "matched": 1, "msg": ""}

TASK [debug] ***************************************************************************************
ok: [vm1] => {
    "find_result": {
        "changed": false, 
        "examined": 10, 
        "failed": false, 
        "files": [
            {
                "atime": 1525883689.7708306, 
                "ctime": 1525883574.9308305, 
                "dev": 64768, 
                "gid": 0, 
                "gr_name": "root", 
                "inode": 67586701, 
                "isblk": false, 
                "ischr": false, 
                "isdir": false, 
                "isfifo": false, 
                "isgid": false, 
                "islnk": false, 
                "isreg": true, 
                "issock": false, 
                "isuid": false, 
                "mode": "0644", 
                "mtime": 1525883574.7018306, 
                "nlink": 1, 
                "path": "copyDescFile", 
                "pw_name": "root", 
                "rgrp": true, 
                "roth": true, 
                "rusr": true, 
                "size": 9, 
                "uid": 0, 
                "wgrp": false, 
                "woth": false, 
                "wusr": true, 
                "xgrp": false, 
                "xoth": false, 
                "xusr": false
            }
        ], 
        "matched": 1, 
        "msg": ""
    }
}

PLAY RECAP *****************************************************************************************
vm1                        : ok=4    changed=0    unreachable=0    failed=0
```

### Facts 변수
대상 호스트의 정보를 저장하고 있는 변수. Playbook 실행 시 Ansible이 자동으로 호스트 정보를 수집하여 설정한다.

호스트명, IP주소, 환경변수, 디바이스 정보 등이 저장된다.

```bash
$ ansible vm1 -i inventoryfile-alias -m setup
vm1 | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
## ...너무 길고 프라이빗 데이터가 많다... 생략!
```

#### playbook에서 facts 변수에 접근하기
`setup`모듈 사용시 facts 변수의 구조가 보이는데, 내부 키 값에 대하여 playbook에서 변수로서 바로 접근 가능하다.

```yaml
# playbook-facts.yml
---
- hosts: vm1 
  tasks:
    - debug: var=ansible_nodename
```

```bash
$ ansible-playbook -i inventoryfile-alias playbook-facts.yml 

PLAY [vm1] *****************************************************************************************

TASK [Gathering Facts] *****************************************************************************
ok: [vm1]

TASK [debug] ***************************************************************************************
ok: [vm1] => {
    "ansible_nodename": "yenarue-devops-ansible.dev"
}

PLAY RECAP *****************************************************************************************
vm1                        : ok=2    changed=0    unreachable=0    failed=0
```

### 반복하기
* [ansible 공식사이트 loop관련 문서](http://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html)
    * Ansible 2.5 이전까지는 루프 생성시 `with_<lookup>` 키워드를 주로 사용했음.
    * `loop` 키워드 = `with_list`

리스트 지정후 `item`으로 접근 가능

#### 여러개의 패키지들을 설치할 때

```yaml
# playbook-loop-yum
---
- hosts: vm1
  become: true
  tasks:
    - name: 패키지들 설치하기
      yum: name={{ item }} state=present
      with_items:
        - httpd
        - php
        - git
```

```bash
$ ansible-playbook -i inventoryfile-alias playbook-loop-yum.yml 

PLAY [vm1] *****************************************************************************************

TASK [Gathering Facts] *****************************************************************************
ok: [vm1]

TASK [패키지들 설치하기] ***********************************************************************************
changed: [vm1] => (item=[u'httpd', u'php', u'git'])

PLAY RECAP *****************************************************************************************
vm1                        : ok=2    changed=1    unreachable=0    failed=0
```

#### 여러개의 파일을 처리할 때
```yaml
# playbook-loop-copyfile.yml
---
- hosts: vm1
  tasks:
    - name: 로컬 파일들을 전송한다
      copy: src=files/{{ item.src }} dest={{ item.dest }}
      with_items:
        - { src: file1, dest: ./copyfile1 }
        - { src: file2, dest: ./copyfile2 }

    - name: 복사된 파일을 확인한다
      find:
        path: './'
        patterns: "{{ item }}"
      with_items:
        - copyfile1
        - copyfile2
      register: find_result

    - debug: var=find_result

```

```bash
ansible-playbook -i inventoryfile-alias playbook-loop-copyfile.yml 

PLAY [vm1] *****************************************************************************************

TASK [Gathering Facts] *****************************************************************************
ok: [vm1]

TASK [로컬 파일들을 전송한다] ********************************************************************************
ok: [vm1] => (item={u'dest': u'./copyfile1', u'src': u'file1'})
ok: [vm1] => (item={u'dest': u'./copyfile2', u'src': u'file2'})

TASK [복사된 파일을 확인한다] ********************************************************************************
ok: [vm1] => (item=copyfile1)
ok: [vm1] => (item=copyfile2)

TASK [debug] ***************************************************************************************
ok: [vm1] => {
    "find_result": {
        "changed": false, 
        "msg": "All items completed", 
        "results": [
            {
                "_ansible_ignore_errors": null, 
                "_ansible_item_result": true, 
                "_ansible_no_log": false, 
                "_ansible_parsed": true, 
                "changed": false, 
                "examined": 12, 
                "failed": false, 
                "files": [
                    {
                       ...생략
                        "path": "copyfile1", 
                       ...생략
                    }
                ], 
                ...생략
                "item": "copyfile1", 
                "matched": 1, 
                "msg": ""
            }, 
            {
                "_ansible_ignore_errors": null, 
                "_ansible_item_result": true, 
                "_ansible_no_log": false, 
                "_ansible_parsed": true, 
                "changed": false, 
                "examined": 12, 
                "failed": false, 
                "files": [
                    {
                        ...생략
                        "path": "copyfile2", 
                       ...생략
                    }
                ], 
                ...생략
                "item": "copyfile2", 
                "matched": 1, 
                "msg": ""
            }
        ]
    }
}

PLAY RECAP *****************************************************************************************
vm1                        : ok=4    changed=0    unreachable=0    failed=0
```

### 조건부 구문
`when`을 사용하여 특정 조건에 따라 태스크가 실행되도록 설정할 수 있다.

#### OS볋로 다른 패키지 매니저를 사용하도록 설정할 때
```yaml
# playbook-when-os.yml
---
- hosts: vm1 
  tasks:
    - name: httpd 설치 (yum)
      yum: name=httpd state=latest
      when: ansible_os_family == "RedHat"

    - name: Apache2 설치 (apt-get)
      apt: name=apache2-mpm-prefork state=latest
      when: ansible_os_family == "Debian"

    - name: httpd 설치 (CentOS 6 or 7)
      yum: name=httpd state=present
      when:
        ansible_distribution == "CentOS"
        and (ansible_distribution_major_version == "6" 
             or ansible_distribution_major_version == "7")
```

```bash
ansible-playbook -i inventoryfile-alias playbook-when-os.yml

PLAY [vm1] *****************************************************************************************

TASK [Gathering Facts] *****************************************************************************
ok: [vm1]

TASK [httpd 설치 (yum)] ******************************************************************************
ok: [vm1]

TASK [Apache2 설치 (apt-get)] ************************************************************************
skipping: [vm1]

TASK [httpd 설치 (CentOS 6 or 7)] ********************************************************************
ok: [vm1]

PLAY RECAP *****************************************************************************************
vm1                        : ok=3    changed=0    unreachable=0    failed=0
```

### 필터
변수의 값을 변환할 수 있다.

#### 변수 값 변환하는 경우
```yml
# playbook-filter-list.yml
---
- hosts: vm1 
  vars:
    fruits:
      - Apple
      - Orange
      - Maroon
  tasks:
    - debug: msg={{ fruits|first }}
    - debug: msg={{ fruits|first|lower }}
    - debug: msg={{ fruits|join(',') }}
```

```bash
$ ansible-playbook -i inventoryfile-alias playbook-filter-list.yml 

PLAY [vm1] *****************************************************************************************

TASK [Gathering Facts] *****************************************************************************
ok: [vm1]

TASK [debug] ***************************************************************************************
ok: [vm1] => {
    "msg": "Apple"
}

TASK [debug] ***************************************************************************************
ok: [vm1] => {
    "msg": "apple"
}

TASK [debug] ***************************************************************************************
ok: [vm1] => {
    "msg": "Apple,Orange,Maroon"
}

PLAY RECAP *****************************************************************************************
vm1                        : ok=4    changed=0    unreachable=0    failed=0
```

#### 태스크 실행결과를 판별하는 경우
`when`과 함께 `filter`를 사용하면 테스트성 코드를 작성하기 좋다.
다만, `debug`등의 테스트성 코드를 작성할 때는 `|` 대신 `is`를 사용해줘야한다.
이미 deprecated 되어있으며, 테스트성 코드 작성시에 `|`를 2.9에서부터 지원하지 않을 예정이라고 한다. 확실히 가독성이 훨씬 좋아진 듯 하다.

> [DEPRECATION WARNING]: Using tests as filters is deprecated. Instead of using `result|skipped` 
instead use `result is skipped`. This feature will be removed in version 2.9. Deprecation warnings 
can be disabled by setting deprecation_warnings=False in ansible.cfg.

```yml
# playbook-filter-result.yml
---                
- hosts: vm1 
  tasks:
    - shell: /usr/bin/some_command
      register: result
      ignore_errors: True

    - debug: msg="실패" 
      when: result is failed
    
    - debug: msg="상태 변경됨" 
      when: result is changed

    - debug: msg="성공" 
      when: result is success

    - debug: msg="스킵" 
      when: result is skipped

```

```bash
$ ansible-playbook -i inventoryfile-alias playbook-filter-list.yml 

PLAY [vm1] *****************************************************************************************

TASK [Gathering Facts] *****************************************************************************
ok: [vm1]

TASK [debug] ***************************************************************************************
ok: [vm1] => {
    "msg": "Apple"
}

TASK [debug] ***************************************************************************************
ok: [vm1] => {
    "msg": "apple"
}

TASK [debug] ***************************************************************************************
ok: [vm1] => {
    "msg": "Apple,Orange,Maroon"
}

PLAY RECAP *****************************************************************************************
vm1                        : ok=4    changed=0    unreachable=0    failed=0   

yenarue@yenarue-ubuntu:~/yenaHDD/TIL-sample-projects/DevOps/Ansible$ ansible-playbook -i inventoryfile-alias playbook-filter-result.yml 

PLAY [vm1] *****************************************************************************************

TASK [Gathering Facts] *****************************************************************************
ok: [vm1]

TASK [shell] ***************************************************************************************
fatal: [vm1]: FAILED! => {"changed": true, "cmd": "/usr/bin/some_command", "delta": "0:00:00.005037", "end": "2018-05-09 18:36:01.194270", "msg": "non-zero return code", "rc": 127, "start": "2018-05-09 18:36:01.189233", "stderr": "/bin/sh: /usr/bin/some_command: No such file or directory", "stderr_lines": ["/bin/sh: /usr/bin/some_command: No such file or directory"], "stdout": "", "stdout_lines": []}
...ignoring

TASK [debug] ***************************************************************************************
ok: [vm1] => {
    "msg": "실패"
}

TASK [debug] ***************************************************************************************
ok: [vm1] => {
    "msg": "상태 변경됨"
}

TASK [debug] ***************************************************************************************
skipping: [vm1]

TASK [debug] ***************************************************************************************
skipping: [vm1]

PLAY RECAP *****************************************************************************************
vm1                        : ok=4    changed=1    unreachable=0    failed=0
```

## Playbook 디버깅하기
* `--syntax-check` : Playbook 구문 체크. 실제 실행은 되지 않음.
* `--check` : Playbook 실행시 상태가 changed 되는지 체크. 실제 실행은 되지 않기 때문에 타 태스크에 의존성을 가지는 경우 정상적으로 체크되지 않을 수 있다.
* `--diff` : 템플릿 파일의 변경 부분을 unified diff 형식으로 표시. 실제로 실행이 된다. (`--check`와 함께 사용하면 실제 실행은 되지 않음)
* `-vv`, `-vvv` : verbose 옵션..
* `-t` : 태깅된 태스크만 선택적으로 실행할 수 있음.
* `--step` : 태스크를 하나씩 실행하도록 하는 옵션. 각 태스크 실행 후 (y/n)이 뜸.
* `--list-hosts`, `--list-tags`, `--list-tasks` : 처리대상 호스트, 태그, 태스크들을 출력함
* `debug` 모듈 사용 : 위에서 계속 썼던 그 `debug` 맞음ㅇㅇ

# 관련자료
* [Naver D2 김용환 님의 'Ansible의 이해와 활용' 발표자료](https://www.slideshare.net/deview/1a7ansible)