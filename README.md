# deploy_mysql_cluster

一键部署mysql PXC 集群

# 脚本使用方式：


cd deploy_mysql_cluster

#编辑bash.config参数

chmod -R 755 .

./deploy_mysql_master.sh

# 如果发现keepalived启动后报127异常退出的错误

那么请升级系统内核至4+版本脚本内容如下：

#!/bin/bash

setupkernel(){

rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org

rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm

yum --enablerepo=elrepo-kernel install -y kernel-lt kernel-lt-devel

grub2-set-default 0

reboot

}

setupkernel

# bash.config参数介绍

#swarm管理节点IP

masterip="192.168.1.107"

#内网网段
ip_segment="192.168.1"

#VIP

keepalived_vip="192.168.1.150"

#keepalived 监听的哪块网卡

interface="enp0s3"

#root用户密码

root_passwd=root123

#主机名称前缀

hostname=mysql

#主机列表

hostip=(
192.168.1.107
192.168.1.108
)

请严格按照如上的方式进行配置！

该脚本为mysql5.7.25 PXC集群，多节点采用swarm集群的方式。通过KEEPALIVED进行服务存活检查及故障切换


# 如有节点故障

 docker run -d -p 3306:3306 -p 4444:4444 -p 4567:4567 -p 4568:4568 -e MYSQL_ROOT_PASSWORD="mysqlroot密码" -e CLUSTER_JOIN=主节点主机名（mysql1) -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD="mysqlroot密码" -v /opt/mysql/data:/var/lib/mysql -v /opt/mysql/backup:/data -v /etc/localtime:/etc/localtime:ro -v /var/run/docker.sock:/var/run/docker.sock --privileged -e character-set-server=utf8mb4 -e collation-server=utf8mb4_unicode_ci --name="故障节点主机名" --net=swarm_mysql docker.io/percona/percona-xtradb-cluster


同步完成后启动再keepalived

systemctl restart keepalived 

