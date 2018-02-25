## Host - 计算节点部署


添加repository

> apt install software-properties-common 

> add-apt-repository cloud-archive:pike  
> `[ENTER]`


nova-compute部署
---

安装nova-compute组件
> apt install nova-compute

编辑nova配置
> vi /etc/nova/nova.conf

```bash
[DEFAULT]
# log_dir = /var/log/nova
lock_path = /var/lock/nova
state_path = /var/lib/nova
transport_url = rabbit://openstack:asd@192.168.1.11
my_ip = 192.168.1.10
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[api]
auth_strategy = keystone

[api_database]
connection = sqlite:////var/lib/nova/nova_api.sqlite
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
connection = sqlite:////var/lib/nova/nova.sqlite
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
enabled = True
vncserver_listen = 0.0.0.0
vncserver_proxyclient_address = $my_ip
novncproxy_base_url = http://192.168.1.11:6080/vnc_auto.html

[workarounds]
[wsgi]
[xenserver]
[xvp]

```

> `$my_ip 为计算节点管理接口IP`

重启nova-compute服务
> service nova-compute restart

<br />

**添加计算节点到cell数据库 (在控制节点上进行)**

加载admin变量
>. admin-openrc

列出计算节点
> openstack compute service list --service nova-compute

```
+----+--------------+---------+------+---------+-------+----------------------------+
| ID | Binary       | Host    | Zone | Status  | State | Updated At                 |
+----+--------------+---------+------+---------+-------+----------------------------+
|  7 | nova-compute | compute | nova | enabled | up    | 2018-02-24T13:37:04.000000 |
+----+--------------+---------+------+---------+-------+----------------------------+
```

发现计算节点
> su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova

```
Found 2 cell mappings.
Skipping cell0 since it does not contain hosts.
Getting compute nodes from cell 'cell1': 50bc21e9-ca2f-45c9-b5d4-d3afa116eea8
Found 1 unmapped computes in cell: 50bc21e9-ca2f-45c9-b5d4-d3afa116eea8
Checking host mapping for compute host 'compute': 7f010bab-58f4-4f8d-ac21-dc9e9e5ccd09
Creating host mapping for compute host 'compute': 7f010bab-58f4-4f8d-ac21-dc9e9e5ccd09
```

<br />

neutron-linuxbridge部署
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
local_ip = 192.168.1.10
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

查询计算组件运行情况

> openstack compute service list

```
+----+------------------+------------+----------+---------+-------+----------------------------+
| ID | Binary           | Host       | Zone     | Status  | State | Updated At                 |
+----+------------------+------------+----------+---------+-------+----------------------------+
|  5 | nova-conductor   | controller | internal | enabled | up    | 2018-02-25T09:45:11.000000 |
|  7 | nova-scheduler   | controller | internal | enabled | up    | 2018-02-25T09:45:09.000000 |
|  8 | nova-consoleauth | controller | internal | enabled | up    | 2018-02-25T09:45:11.000000 |
|  9 | nova-compute     | icooci     | nova     | enabled | up    | 2018-02-25T09:45:11.000000 |
+----+------------------+------------+----------+---------+-------+----------------------------+
```


查看网络组件运行情况

> openstack network agent list

```
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host       | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
| 0a094afe-3db1-430d-83c2-0b45afd92f18 | L3 agent           | controller | nova              | :-)   | UP    | neutron-l3-agent          |
| 0cbf8d07-60f7-49d9-a5a4-f5d72b8f5f9d | Linux bridge agent | icooci     | None              | :-)   | UP    | neutron-linuxbridge-agent |
| 5c12abc5-48af-4714-98e7-60dd052a3aeb | DHCP agent         | controller | nova              | :-)   | UP    | neutron-dhcp-agent        |
| db67083c-ad30-4ce4-bf13-2a7ac32122f9 | Linux bridge agent | controller | None              | :-)   | UP    | neutron-linuxbridge-agent |
| ecb56a51-303e-43fe-b19b-8803a54c0ad5 | Metadata agent     | controller | None              | :-)   | UP    | neutron-metadata-agent    |
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
```
