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
> add-apt-repository cloud-archive:pike  
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
innodb_file_per_table = on
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

**控制节点安装Etcd**

创建etcd用户

> groupadd --system etcd

> useradd --home-dir "/var/lib/etcd" --system --shell /bin/false -g etcd etcd

创建相关目录

> mkdir -p /etc/etcd  
> chown etcd:etcd /etc/etcd  
> mkdir -p /var/lib/etcd  
> chown etcd:etcd /var/lib/etcd  

下载编译源码

```
ETCD_VER=v3.2.7
rm -rf /tmp/etcd && mkdir -p /tmp/etcd
curl -L https://github.com/coreos/etcd/releases/download/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
tar xzvf /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz -C /tmp/etcd --strip-components=1
cp /tmp/etcd/etcd /usr/bin/etcd
cp /tmp/etcd/etcdctl /usr/bin/etcdctl
```

创建并配置yml文件

> vi /etc/etcd/etcd.conf.yml
```
name: controller
data-dir: /var/lib/etcd
initial-cluster-state: 'new'
initial-cluster-token: 'etcd-cluster-01'
initial-cluster: controller=http://192.168.1.11:2380
initial-advertise-peer-urls: http://192.168.1.11:2380
advertise-client-urls: http://192.168.1.11:2379
listen-peer-urls: http://0.0.0.0:2380
listen-client-urls: http://192.168.1.11:2379
```

创建并配置服务文件
> vi /lib/systemd/system/etcd.service
```
[Unit]
After=network.target
Description=etcd - highly-available key value store

[Service]
LimitNOFILE=65536
Restart=on-failure
Type=notify
ExecStart=/usr/bin/etcd --config-file /etc/etcd/etcd.conf.yml
User=etcd

[Install]
WantedBy=multi-user.target
```

启用etcd服务
> systemctl enable etcd  
> systemctl start etcd  
