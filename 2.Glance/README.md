## Glance部署

创建glance数据库

> mysql

```
CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'asd';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'asd';
EXIT;
```

加载admin变量

> . admin-openrc

创建glance用户

> openstack user create --domain default --password-prompt glance  

为glance用户分配admin角色

> openstack role add --project service --user glance admin  
> `本条命令无回显`

创建服务实体

> openstack service create --name glance --description "OpenStack Image" image

创建glance服务API Endpoint

> openstack endpoint create --region RegionOne image public http://192.168.1.11:9292  
> openstack endpoint create --region RegionOne image internal http://192.168.1.11:9292  
> openstack endpoint create --region RegionOne image admin http://192.168.1.11:9292  
  
安装glance软件包

> apt install glance

编辑glance-api配置文件

> vi /etc/glance/glance-api.conf

```bash
[DEFAULT]
[cors]
[database]
connection = mysql+pymysql://glance:asd@192.168.1.11/glance

[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/

[image_format]
disk_formats = ami,ari,aki,vhd,vhdx,vmdk,raw,qcow2,vdi,iso,ploop.root-tar

[keystone_authtoken]
auth_uri = http://192.168.1.11:5000
auth_url = http://192.168.1.11:35357
memcached_servers = 192.168.1.11:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = glance
password = asd

[matchmaker_redis]
[oslo_concurrency]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]

[paste_deploy]
flavor = keystone

[profiler]
[store_type_location_strategy]
[task]
[taskflow_executor]

```

编辑glance-registry配置文件

> vi /etc/glance/glance-registry.conf
```
[DEFAULT]

[database]
connection = mysql+pymysql://glance:asd@192.168.1.11/glance

[keystone_authtoken]
auth_uri = http://192.168.1.11:5000
auth_url = http://192.168.1.11:35357
memcached_servers = 192.168.1.11:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = glance
password = asd

[matchmaker_redis]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_policy]

[paste_deploy]
flavor = keystone

[profiler]
```

初始化glance数据库
> su -s /bin/sh -c "glance-manage db_sync" glance

重启glance服务

> service glance-registry restart  
> service glance-api restart  

验证操作
---

加载admin变量

> . admin-openrc

获取cirros镜像

> wget http://download.cirros-cloud.net/0.3.5/cirros-0.3.5-x86_64-disk.img

上传cirros镜像至glance

```
openstack image create "cirros" \
  --file cirros-0.3.5-x86_64-disk.img \
  --disk-format qcow2 --container-format bare \
  --public
```

查询镜像列表

> openstack image list

服务器返回结果如下:
```
+--------------------------------------+--------+--------+
| ID                                   | Name   | Status |
+--------------------------------------+--------+--------+
| 1697f53d-bdfd-4220-95af-4a64ccf721ce | cirros | active |
+--------------------------------------+--------+--------+
```

Additional Operation
---

导入metadata
> glance-manage db_load_metadefs
