# Ansible is simple IT Automation
自动化运维
自动化部署
自动化...

## 安装和基本概念
管理节点要求： Python 2.6 / 2.7
托管节点： ssh / sftp Python 2.6 +
```bash
sudo yum install ansible
```

## playbook

### playbook 介绍
eg.
```yaml
---
- hosts: webservers # 主机
  vars: # 变量
    http_port: 80
    max_clients: 200
  remote_user: root
  tasks: # 任务列表
  - name: ensure apache is at the latest version # 任务名
    yum: pkg=httpd state=latest # 模块
  - name: write the apache config file
    template: src=/srv/httpd.j2 dest=/etc/httpd.conf
    notify: # handler：当文件内容改动时，启动服务
    - restart apache
  - name: ensure apache is running
    service: name=httpd state=started
  handlers: # 在发生改变时操作
    - name: restart apache
      service: name=httpd state=restarted
```
一个 playbook 包含以上部分

#### 


### playbook 角色 Roles 和 Include 语句


### 变量

### 条件语句

### 循环



## 配置管理

## 部署和语法编排

## ad-hoc 并行命令

## ansible 核心模块

## 编写自己的模块

系统性入门：
https://www.cnblogs.com/brianzhu/p/10174130.html
