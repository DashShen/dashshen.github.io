---
title: 搭建私有hdp全家桶yum源及ambari安装部署
date: 2017-04-19 20:35:29
tags:
---
# 下载ambari,hdp,hdp-utils的rpm tarball
* 下载hdp

```shell
wget http://public-repo-1.hortonworks.com/HDP/centos7/2.x/updates/2.6.0.3/HDP-2.6.0.3-centos7-rpm.tar.gz
```
* 下载hdp-utils

```shell
wget http://public-repo-1.hortonworks.com/HDP-UTILS-1.1.0.21/repos/centos7/HDP-UTILS-1.1.0.21-centos7.tar.gz
```
* 下载ambari

```shell
wget http://public-repo-1.hortonworks.com/ambari/centos7/2.x/updates/2.5.0.3/ambari-2.5.0.3-centos7.tar.gz
```

# 在 /var/www/html 目录，解压tar包

```shell
cd /var/www/html

tar -zxvf ambari-2.5.0.3-centos7.tar.gz

mkdir hdp

cd hdp

tar -zxvf  HDP-2.6.0.3-centos7-rpm.tar.gz

tar -zxvf  HDP-UTILS-1.1.0.21-centos7.tar.gz
``` 

# 确认新建的仓库

* **Ambari Base URL**
    * http://<web.server>/Ambari-2.5.0.3/<OS>

* **HDP Base URL**
    * http://<web.server>/hdp/HDP/<OS>/2.x/updates/<latest.version>

* **HDP-UTILS Base URL**
    * http://<web.server>/hdp/HDP-UTILS-<version>/repos/<OS>

# 准备ambari的repo文件

* 下载ambari.repo模板

```shell
wget http://public-repo-1.hortonworks.com/ambari/<OS>/2.x/updates/2.5.0.3/ambari.repo
```
* 修改ambari.repo文件，替换`baseurl`为自建仓库，gpgkey可以指向自建仓库GPG-KEY

```
[Updates-Ambari-2.5.0.3]

name=Ambari-2.5.0.3-Updates

baseurl=INSERT-BASE-URL

gpgcheck=1

gpgkey=http://public-repo-1.hortonworks.com/ambari/centos6/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins

enabled=1


priority=1
```

* 把ambari.repo文件放在每台节点上

```shell
cp ambari.repo /etc/yum.repos.d/ambari.repo
```

# 下载repo

```shell
wget http://<web.server>/ambari.repo

cp ambari.repo /etc/yum.repos.d

sudo yum update
```

# 安装ambari-server

```shell
sudo yum install ambari-server
```

## 配置使用ambari-server使用mysql
* 创建数据库

```shell
CREATE DATABASE ambari;
```
* 在mysql中创建用户

```shell
CREATE USER 'ambari'@'%' IDENTIFIED BY 'ambari';

GRANT ALL ON ambari.* TO 'ambari'@'%';
```
* 执行初始化sql

```shell
cd /var/lib/ambari-server/resources/

# 在myusql中执行
source Ambari-DDL-MySQL-CREATE.sql;
```

# 安装ambari-agent
* 可以在server上,让ambari自动安装（推荐）
* 手动安装的方式
下载安装ambari-agent,同时修改agent的配置文件，将hostname指向ambari-server所在节点

```shell
sudo yum install ambari-agent
```

# 修改agent配置的hostname为server的地址

```shell
/etc/ambari-agent/conf
...
[server]
hostname=<SERVER_ADDRESS>
...
```
