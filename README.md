# Ansible
단순한 언어를 사용하여, 반복되는 작업을 자동화 함

### 사용처
- 여러 노드나 특정 노드 그룹을 대상으로 원하는 서버 환경을 구성할 수 있음
- Artifacts를 특정 노드로 배포 할 수 있음

### 구조
playbooks > roles > tasks and handlers 로 구성 됨
- playbooks: ansible main app
```yml
<playbooks 예시>
---
- name: setup pinpoint-server
  hosts: pinpoint-server
  become: yes
  roles:
    - pinpoint-server
```
- roles: 폴더로 구성 됨. playbooks에 정의된 roles의 이름과 동일한 이름의 폴더 필요
```
<pinpoint에서 사용된 폴더 구조는 아래와 같다.>
- roles
  - pinpoint-server
    - defaults  # config 역할. main.yml 에 정의 함
    - files     # tasks에서 사용될 파일 
    - handlers  # notify로 수행될 sub task
    - tasks     # task yml files
    - templates # 배포대상 config 등의 templates
```
- tasks: 작업이 수행되는 단위.
   - subtask를 추가 할 수 있음
     - import: playbook이 parse되는 단계에서 실행 됨.(초기 설정 작업 대상)
     - include: tasks 실행 흐름에 따라 실행 됨
     - notify: task의 마지막에 작성되며, 동일한 handlers가 여러번 호출되도 최초 한번만 실행 됨(service start에 주로 사용됨)
   - when: 조건에 따른 task 실행
   - register: task 결과 변수로 저장
```yml
<task 예시>
---
- name: Check if Hbase Service Exists
  stat: path=/etc/systemd/system/hbase.service
  register: hbase_service_status

- include_tasks: init_hbase.yml
  when: not hbase_service_status.stat.exists
  tag: initizlize hbase
```
> [Ansible Documentation](https://docs.ansible.com/ansible/latest/index.html)에서 추가설명 확인 가능

## 0. install
#### instll ansible
```
sudo apt-get update
sudo apt-get install software-properties-common

sudo apt-add-repository ppa:ansible/ansible

sudo apt-get update
sudo apt-get install ansible
```
#### add hosts for ansible
/etc/ansible/hosts
```
[pinpoint-server] # name for hosts in main playbook
localhost ansible_connection=local
```

## 1. pinpoint
pinpoint setup guide on localhost  
> 다른 서버에 setup 하기 위해서는 /etc/ansible/hosts에 해당 서버를 정의 후
> setup-pinpoint.yml 파일의 hosts: 내용을 변경해야 함

### prerequisites
- unzip이 설치되어 있어야 함
- pinpoint 가이드에 따라 JDK6 ,JDK7, JDK8 가 반드시 필요

### configurations
- set configs for collector and web in roles/pinpoint-server/default/main.yml
- set configs for agent in roles/pinpoint-agent/default/main.yml


### how to run setup

```sh
ansible-playbook -v setup-pinpoint.yml
```

### start pinpoint services
- sudo systemctl start pinpoint-web
- sudo systemctl start pinpoint-collector

### how to remove
```sh
# stop&disable service
sudo systemctl stop hbase
sudo systemctl stop pinpoint-web
sudo systemctl stop pinpoint-collector
sudo systemctl disable hbase
sudo systemctl disable pinpoint-web
sudo systemctl disable pinpoint-collector

# remove service 
sudo rm /etc/systemd/system/hbase.service
sudo rm /etc/systemd/system/pinpoint-web.service
sudo rm /etc/systemd/system/pinpoint-collector.service

# reload daemon
sudo systemctl daemon-reload
sudo systemctl reset-failed

# remove folders
sudo rum -rf {pinpoint_hbase_install_dir}
sudo rum -rf {pinpoint_data_dir}
sudo rum -rf {pinpoint_hbase_log_dir}

sudo rum -rf {pinpoint_collector_tomcat_home}
sudo rum -rf {pinpoint_collector_tomcat_log_dir}

sudo rum -rf {pinpoint_web_tomcat_home}
sudo rum -rf {pinpoint_web_tomcat_log_dir}

# remove users&groups (optional)
```
