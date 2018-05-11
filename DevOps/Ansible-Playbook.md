Ansible Playbook
=====
Ansible Playbook에 대해 알아보자.


# Ansible Playbook 이란?
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
