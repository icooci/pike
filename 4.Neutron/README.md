## Neutron部署 - 控制节点

创建neutron数据库
```
CREATE DATABASE neutron;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'asd';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'asd';
EXIT;
```

加载admin变量

> . admin-openrc

创建neutron用户

> openstack user create --domain default --password-prompt neutron

为neutron用户分配admin角色

> openstack role add --project service --user neutron admin  
> `本条命令无回显`

创建neutron服务实体

> openstack service create --name neutron --description "OpenStack Networking" network

创建neutron服务API Endpoint

> openstack endpoint create --region RegionOne network public http://192.168.1.11:9696  
> openstack endpoint create --region RegionOne network internal http://192.168.1.11:9696  
> openstack endpoint create --region RegionOne network admin http://192.168.1.11:9696  

网络类型: Self-service 网络
---

安装neutron组件
> apt install neutron-server neutron-plugin-ml2 neutron-linuxbridge-agent neutron-l3-agent neutron-dhcp-agent neutron-metadata-agent

编辑neutron配置
> /etc/neutron/neutron.conf

```bash
[DEFAULT]
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = true
transport_url = rabbit://openstack:asd@192.168.1.11
auth_strategy = keystone
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true

[agent]
root_helper = sudo /usr/bin/neutron-rootwrap /etc/neutron/rootwrap.conf
[cors]

[database]
# connection = sqlite:////var/lib/neutron/neutron.sqlite
connection = mysql+pymysql://neutron:asd@192.168.1.11/neutron

[keystone_authtoken]
auth_uri = http://192.168.1.11:5000
auth_url = http://192.168.1.11:35357
memcached_servers = 192.168.1.11:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = asd

[matchmaker_redis]

[nova]
auth_url = http://192.168.1.11:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = asd

[oslo_concurrency]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[quotas]
[ssl]

```

