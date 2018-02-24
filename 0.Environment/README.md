## 部署环境准备

**系统规划**  

| 节点 | 主机名 | IP地址 |
| ---- | ----- | ------- |
| Controller Node | controller | 192.168.1.11 |
| Compute Node | compute | 192.168.1.21 |
| Block Storage Node | block | 192.168.1.31 |
| Object Storage Node 1 | object1 | 192.168.1.41 |
| Object Storage Node 2 | object2 | 192.168.1.42 |

> All Node OS: Ubuntu Server 16.04.3 LTS

**网络配置**  
> vi /etc/network/interfaces

```
auto ens3
iface ens3 inet static
        address 192.168.1.11
        netmask 255.255.255.0
        network 192.168.1.0
        broadcast 192.168.1.255
        gateway 192.168.1.1
        dns-nameservers 192.168.1.1

auto ens4
iface ens4 inet manual
```

**主机名配置**  
> vi /etc/hostname
```
controller / compute / block
```

**网络名称解析**  
> vi /etc/hosts

```bash
# controller
192.168.1.11       controller

# compute
192.168.1.21       compute

# block
192.168.1.31       block

# object1
192.168.1.41       object1

# object2
192.168.1.42       object2
```
**所有节点部署NTP服务**  

安装chrony软件包  
> apt install chrony

修改配置文件  
> vi /etc/chrony/chrony.conf

控制节点配置:  
```diff
+ allow 192.168.1.0/24
```

其他节点配置:  
```diff
- pool 2.debian.pool.ntp.org offline iburst
+ server controller iburst
```

重启服务  
> service chrony restart

验证NTP同步信息  
> chronyc sources

**所有节点安装openstack client**  
> apt install software-properties-common

添加repository  
> add-apt-repository cloud-archive:newton  
> [ENTER]

更新软件包  

> apt-get update && apt-get dist-upgrade

重启以应用新kernel  

> reboot

安装openstackclient  

> apt install python-openstackclient

**控制节点安装SQL**
> apt install mariadb-server python-pymysql

> vi /etc/mysql/mariadb.conf.d/99-openstack.cnf
```
[mysqld]
bind-address = 192.168.1.11

default-storage-engine = innodb
innodb_file_per_table
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
```
> service mysql restart

执行安全设置
> mysql_secure_installation

**控制节点安装Message queue**  

> apt install rabbitmq-server

> rabbitmqctl add_user openstack asd

> rabbitmqctl set_permissions openstack ".*" ".*" ".*"

**控制节点安装Memcached**

> apt install memcached python-memcache

> vi /etc/memcached.conf
```diff
- -l 127.0.0.1
+ -l 192.168.1.11
```
> service memcached restart
