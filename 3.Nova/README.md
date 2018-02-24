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

安装nova软件包


编辑nova配置
