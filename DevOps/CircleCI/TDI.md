테스트 주도 인프라 (Test Driven Infrastructure)
====

# 여기서의 환경구성
## 로컬
* Ubuntu 16.04
* ruby 2.3.1
* knife-solo
* Serverspec 2.41.3

## 서버
* VirtualBox
* Vagrant
* CentOS 7


# 환경 구성을 해보자
## ServerSpec
```bash
$ sudo gem install serverspec
```

## knife-solo
```bash
$ sudo gem install knife-solo
```

## ChefDK
최신버전에서 `knife cookbook create` 가 삭제되었다. `ChefDK`를 사용하는 것을 권장하고 있다.
```bash
$ wget https://packages.chef.io/files/stable/chefdk/2.4.17/ubuntu/16.04/chefdk_2.4.17-1_amd64.deb
$ sudo dpkg -i chefdk_2.4.17-1_amd64.deb
```

## 작업용 디렉토리 준비
```bash
$ knife solo init infra-ci-cookbooks
$ cd infra-ci-cookbooks
infra-ci-cookbooks$ ls
cookbooks  data_bags  environments  nodes  roles  site-cookbooks
```

## Vagrantfile 준비
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
    config.vm.box = "bento/centos-7.1"

    config.vm.define :webapp do |host|
        host.vm.hostname = "webapp"
    end
end
```

### vagrant VM에 Chef 설치하기
```bash
$ vagrant up
# VM에 SSH로 접속할때 필요한 정보를 작성
$ vagrant ssh-config webapp >> vagrant_ssh_config
# VM에 Chef 설치
$ knife solo prepare webapp -F vagrant_ssh_config
# 스냅샷
$ vagrant snapshot save webapp webapp-plane
$ vagrant snapshot list
webapp-plane
```

## ServerSpec 준비
```bash
infra-ci-cookbooks$ serverspec-init
Select OS type:

  1) UN*X
  2) Windows

Select number: 1

Select a backend type:

  1) SSH
  2) Exec (local)

Select number: 1

Vagrant instance y/n: y
Auto-configure Vagrant from Vagrantfile? y/n: y
 + spec/
 + spec/webapp/
 + spec/webapp/sample_spec.rb
 + spec/spec_helper.rb
 + Rakefile
 + .rspe

# 샘플 테스트코드 삭제
 infra-ci-cookbooks$ rm spec/webapp/sample_spec.rb
```

# 테스트 주도 인프라 실행해보기
## 어떤 순서로 할거냐면
1. Serverspec 으로 최종 상태의 테스트코드 작성 => 테스트 실패
2. Chef 레시피 작성 후 프로비저닝 실행
3. 테스트 성공되는지 확인 (아니면 2 로 돌아간다)

## 테스트 작성 및 실행
`htppd가 잘 설치되어 있는지`/`실행가능한지`/`실행중인지`/`80포트를 듣고있는지`를 확인하는 테스트 코드이다.

```ruby
# http_spec.rb
require 'spec_helper'

describe package('httpd'), :if => os[:family] == 'redhat' do
    it { should be_installed }
end

describe service('httpd'), :if => os[:family] == 'redhat' do
    it { should be_enabled }
    it { should be_running }
end

describe port(80) do
    it { should be_listening }
end
```

```bash
# 테스트 실행
$ rake spec:webapp
### 실패로그가 떠야 정상!
```

## 레시피 작성 및 프로비저닝 실행
### 레시피 생성
```bash
$ cd site-cookbooks
site-cookbooks$ chef generate cookbook httpd
```

### 레시피 작성
```ruby
# site-cookbooks/httpd/recipes/default.rb
package "httpd" do
	action :install
end

service "httpd" do
	supports :status => true, :restart => true, :reload => true
	action [ :enable, :start ]
end
```

```json
# nodes/webapp.json
{
  "run_list": [
	"recipe[httpd]"
  ],
  "automatic": {
    "ipaddress": "webapp"
  }
}
```

### 프로비저닝 실행
위에서 작성한 레시피를 아래와 같이 실행시키면 vm에 httpd가 설치된 후 실행된다.

```bash
$ knife solo cook webapp -F vagrant_ssh_config
```

## 테스트 재실행
```bash
$ rake spec:webapp
# 성공로그가 떠야 정상
```

## E2E테스트
VM을 초기 상태로 되돌리고

```bash
$ vagrant snapshot restore webapp webapp-plane
```

프로비저닝을 실행해본다.

```bash
$ knife solo cook webapp -F vagrant_ssh_config
Running Chef on webapp...
Checking Chef version...
Uploading the kitchen...
Generating solo config...
Running Chef: sudo chef-solo -c ~/chef-solo/solo.rb -j ~/chef-solo/dna.json
Starting Chef Client, version 13.9.1
resolving cookbooks for run list: ["httpd", "base_packages"]
Synchronizing Cookbooks:
  - base_packages (0.1.0)
  - httpd (0.1.0)
Installing Cookbook Gems:
Compiling Cookbooks...
Converging 8 resources
Recipe: httpd::default
  * yum_package[httpd] action install
    - install version 2.4.6-80.el7.centos of package httpd
  * service[httpd] action enable
    - enable service service[httpd]
  * service[httpd] action start
    - start service service[httpd]
Recipe: base_packages::default
  * yum_package[dstat] action install
    - install version 0.7.2-12.el7 of package dstat
  * yum_package[git] action install
    - install version 1.8.3.1-13.el7 of package git
  * yum_package[tmux] action install
    - install version 1.8-4.el7 of package tmux
  * yum_package[vim-enhanced] action install
    - install version 7.4.160-4.el7 of package vim-enhanced
  * yum_package[yum-plugin-versionlock] action install
    - install version 1.1.31-45.el7 of package yum-plugin-versionlock
  * yum_package[zsh] action install
    - install version 5.0.2-28.el7 of package zsh

Running handlers:
Running handlers complete
Chef Client finished, 9/9 resources updated in 18 seconds
```

테스트를 실행해본다.
```bash
$ rake spec:webapp
/usr/bin/ruby2.3 -I/var/lib/gems/2.3.0/gems/rspec-support-3.7.1/lib:/var/lib/gems/2.3.0/gems/rspec-core-3.7.1/lib /var/lib/gems/2.3.0/gems/rspec-core-3.7.1/exe/rspec --pattern spec/webapp/\*_spec.rb

Package "dstat"
  should be installed

Package "git"
  should be installed

Package "tmux"
  should be installed

Package "vim-enhanced"
  should be installed

Package "yum-plugin-versionlock"
  should be installed

Package "zsh"
  should be installed

Package "httpd"
  should be installed

Service "httpd"
  should be enabled
  should be running

Port "80"
  should be listening

Finished in 0.93423 seconds (files took 4.11 seconds to load)
10 examples, 0 failures
```