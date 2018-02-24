## Nova部署 - 控制节点

创建nova数据库

> mysql

```
CREATE DATABASE nova_api;
CREATE DATABASE nova;
CREATE DATABASE nova_cell0;

GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY 'asd';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'asd';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'asd';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'asd';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' IDENTIFIED BY 'asd';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' IDENTIFIED BY 'asd';

EXIT;
```

加载admin变量

. admin-openrc

创建nova用户

> openstack user create --domain default --password-prompt nova

为nova用户分配admin角色

> openstack role add --project service --user nova admin  
> `本条命令无回显`

创建nova服务实体

> openstack service create --name nova --description "OpenStack Compute" compute

创建nova服务API Endpoint

> openstack endpoint create --region RegionOne compute public http://192.168.1.11:8774/v2.1  
> openstack endpoint create --region RegionOne compute internal http://192.168.1.11:8774/v2.1  
> openstack endpoint create --region RegionOne compute admin http://192.168.1.11:8774/v2.1  


创建placement用户
> openstack user create --domain default --password-prompt placement

为nova用户分配admin角色

> openstack role add --project service --user placement admin
> `本条命令无回显`

创建placement服务实体
> openstack service create --name placement --description "Placement API" placement

创建placement API Endpoint
> openstack endpoint create --region RegionOne placement public http://192.168.1.11:8778  
> openstack endpoint create --region RegionOne placement internal http://192.168.1.11:8778  
> openstack endpoint create --region RegionOne placement admin http://192.168.1.11:8778  

安装nova组件
> apt install nova-api nova-conductor nova-consoleauth nova-novncproxy nova-scheduler nova-placement-api

编辑nova配置文件
> vi /etc/nova/nova.conf

```bash
[DEFAULT]
# log_dir = /var/log/nova
lock_path = /var/lock/nova
state_path = /var/lib/nova
transport_url = rabbit://openstack:asd@192.168.1.11
my_ip = 192.168.1.11
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[api]
auth_strategy = keystone

[api_database]
# connection = sqlite:////var/lib/nova/nova_api.sqlite
connection = mysql+pymysql://nova:asd@192.168.1.11/nova_api

[barbican]
[cache]
[cells]
enable = False
[cinder]
[compute]
[conductor]
[console]
[consoleauth]
[cors]
[crypto]
[database]
# connection = sqlite:////var/lib/nova/nova.sqlite
connection = mysql+pymysql://nova:asd@192.168.1.11/nova

[ephemeral_storage_encryption]
[filter_scheduler]
[glance]
api_servers = http://192.168.1.11:9292

[guestfs]
[healthcheck]
[hyperv]
[ironic]
[key_manager]
[keystone]
[keystone_authtoken]
auth_uri = http://192.168.1.11:5000
auth_url = http://192.168.1.11:35357
memcached_servers = 192.168.1.11:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = asd

[libvirt]
[matchmaker_redis]
[metrics]
[mks]
[neutron]
[notifications]
[osapi_v21]
[oslo_concurrency]
lock_path = /var/lib/nova/tmp

[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[pci]

[placement]
# os_region_name = openstack
os_region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://192.168.1.11:35357/v3
username = placement
password = asd

[quota]
[rdp]
[remote_debug]
[scheduler]
[serial_console]
[service_user]
[spice]
[trusted_computing]
[upgrade_levels]
[vendordata_dynamic_auth]
[vmware]

[vnc]
enabled = true
vncserver_listen = $my_ip
vncserver_proxyclient_address = $my_ip

[workarounds]
[wsgi]
[xenserver]
[xvp]

```
