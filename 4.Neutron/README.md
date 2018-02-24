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

<br />

网络类型: Self-service 网络
---

安装neutron组件
> apt install neutron-server neutron-plugin-ml2 neutron-linuxbridge-agent neutron-l3-agent neutron-dhcp-agent neutron-metadata-agent

编辑neutron配置
> vi /etc/neutron/neutron.conf

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

编辑ml2配置
> vi /etc/neutron/plugins/ml2/ml2_conf.ini
```bash
[DEFAULT]
[l2pop]
[ml2]
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan
mechanism_drivers = linuxbridge,l2population
extension_drivers = port_security

[ml2_type_flat]
flat_networks = provider

[ml2_type_geneve]
[ml2_type_gre]
[ml2_type_vlan]

[ml2_type_vxlan]
vni_ranges = 1:1000

[securitygroup]
enable_ipset = true
```

编辑linuxbridge_agent配置
> vi /etc/neutron/plugins/ml2/linuxbridge_agent.ini
```
[DEFAULT]
[agent]
[linux_bridge]
physical_interface_mappings = provider:ens3

[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver

[vxlan]
enable_vxlan = true
local_ip = 192.168.1.11
l2_population = true
```
> `local_ip为用于overlay的控制节点接口IP`

编辑l3_agent配置
> vi /etc/neutron/l3_agent.ini
```
[DEFAULT]
interface_driver = linuxbridge
[agent]
[ovs]
```
编辑dhcp_agent配置
> vi /etc/neutron/dhcp_agent.ini
```
[DEFAULT]
interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = true
[agent]
[ovs]
```

---

<br />

编辑metadata配置
> vi /etc/neutron/metadata_agent.ini
```
[DEFAULT]
nova_metadata_host = 192.168.1.11
metadata_proxy_shared_secret = asd
[agent]
[cache]
```

修改nova配置
> vi /etc/nova/nova.conf

```bash
[neutron]
# ...
url = http://192.168.1.11:9696
auth_url = http://192.168.1.11:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = asd
service_metadata_proxy = true
metadata_proxy_shared_secret = asd
```

初始化neutron数据库

> su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf  --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron


重启nova-api服务
> service nova-api restart

重启neutron服务
> service neutron-server restart  
> service neutron-linuxbridge-agent restart  
> service neutron-dhcp-agent restart  
> service neutron-metadata-agent restart  

重启l3_agent服务
> service neutron-l3-agent restart

<br />

验证操作
---

加载admin变量

> . admin-openrc

查看网络组件运行情况

> openstack network agent list
```
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host       | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
| 30a1008b-2469-4456-bff9-41ace775d892 | L3 agent           | controller | nova              | :-)   | UP    | neutron-l3-agent          |
| 4e4254f3-3263-463c-91b3-b00bb47b9d07 | Linux bridge agent | controller | None              | :-)   | UP    | neutron-linuxbridge-agent |
| 8cbe64ab-a0a1-43c6-86cd-9b99aeab8a17 | DHCP agent         | controller | nova              | :-)   | UP    | neutron-dhcp-agent        |
| 962ebf06-160f-4a01-bf9d-4e1ec7628ea1 | Metadata agent     | controller | None              | :-)   | UP    | neutron-metadata-agent    |
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
```

<br />

---

<br />

Neutron部署 - 计算节点
---

安装neutron linuxbridge组件

> apt install neutron-linuxbridge-agent

编辑neutron配置

> vi /etc/neutron/neutron.conf

```bash
[DEFAULT]
core_plugin = ml2
transport_url = rabbit://openstack:asd@192.168.1.11
auth_strategy = keystone

[agent]
root_helper = sudo /usr/bin/neutron-rootwrap /etc/neutron/rootwrap.conf
[cors]
[database]
# connection = sqlite:////var/lib/neutron/neutron.sqlite
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

编辑linuxbridge_agent配置
> vi /etc/neutron/plugins/ml2/linuxbridge_agent.ini

```bash
[DEFAULT]
[agent]
[linux_bridge]
physical_interface_mappings = provider:ens3

[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver

[vxlan]
enable_vxlan = true
local_ip = 192.168.1.21
l2_population = true

```
> `local_ip为用于overlay的计算节点接口IP`

修改nova配置

> vi /etc/nova/nova.conf
```bash
[neutron]
# ...
url = http://192.168.1.11:9696
auth_url = http://192.168.1.11:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = asd
```

重启nova-compute服务
> service nova-compute restart

重启neutron服务
> service neutron-linuxbridge-agent restart

<br />

验证操作
---

**在控制节点上进行验证操作**

加载admin变量

> . admin-openrc

查看网络组件运行情况

> openstack network agent list

```
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host       | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
| 30a1008b-2469-4456-bff9-41ace775d892 | L3 agent           | controller | nova              | :-)   | UP    | neutron-l3-agent          |
| 4e4254f3-3263-463c-91b3-b00bb47b9d07 | Linux bridge agent | controller | None              | :-)   | UP    | neutron-linuxbridge-agent |
| 8cbe64ab-a0a1-43c6-86cd-9b99aeab8a17 | DHCP agent         | controller | nova              | :-)   | UP    | neutron-dhcp-agent        |
| 962ebf06-160f-4a01-bf9d-4e1ec7628ea1 | Metadata agent     | controller | None              | :-)   | UP    | neutron-metadata-agent    |
| f7533fbf-03b4-4230-85ef-a7ba5601f326 | Linux bridge agent | compute    | None              | :-)   | UP    | neutron-linuxbridge-agent |
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+

```
