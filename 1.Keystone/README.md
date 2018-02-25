## Keystone部署

创建keystone数据库

> mysql
```
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'asd';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'asd';
EXIT;
```

安装keystone组件
> apt install keystone apache2 libapache2-mod-wsgi


编辑keystone配置文件

> vi /etc/keystone/keystone.conf
```bash
[DEFAULT]
log_dir = /var/log/keystone
[assignment]
[auth]
[cache]
[catalog]
[cors]
[credential]
[database]
# connection = sqlite:////var/lib/keystone/keystone.db
connection = mysql+pymysql://keystone:asd@192.168.1.11/keystone
[domain_config]
[endpoint_filter]
[endpoint_policy]
[eventlet_server]
[extra_headers]
[federation]
[fernet_tokens]
[healthcheck]
[identity]
[identity_mapping]
[ldap]
[matchmaker_redis]
[memcache]
[oauth1]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[paste_deploy]
[policy]
[profiler]
[resource]
[revoke]
[role]
[saml]
[security_compliance]
[shadow_users]
[signing]
[token]
provider = fernet
[tokenless_auth]
[trust]
```

初始化keystone数据库

> su -s /bin/sh -c "keystone-manage db_sync" keystone

初始化Fernet Key库
> keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone  
> keystone-manage credential_setup --keystone-user keystone --keystone-group keystone  

Bootstrap身份认证服务
```
keystone-manage bootstrap --bootstrap-password asd \
  --bootstrap-admin-url http://192.168.1.11:35357/v3/ \
  --bootstrap-internal-url http://192.168.1.11:5000/v3/ \
  --bootstrap-public-url http://192.168.1.11:5000/v3/ \
  --bootstrap-region-id RegionOne
```

<br />

**配置apache服务器**

---

编辑apache2配置文件
> vi /etc/apache2/apache2.conf

```diff
+ ServerName 192.168.1.11
```
重启服务
> service apache2 restart

配置管理员账户
```bash
export OS_USERNAME=admin
export OS_PASSWORD=asd
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://192.168.1.11:35357/v3
export OS_IDENTITY_API_VERSION=3
```

创建service项目，用于各项服务
> openstack project create --domain default --description "Service Project" service

创建demo项目，用于一般权限任务
> openstack project create --domain default --description "Demo Project" demo

创建demo用户
> openstack user create --domain default --password-prompt demo

创建user角色
> openstack role create user

赋予demo用户user角色
> openstack role add --project demo --user demo user

取消临时变量
> unset OS_AUTH_URL OS_PASSWORD

验证操作
---

使用admin用户请求token
```
openstack --os-auth-url http://192.168.1.11:35357/v3 \
  --os-project-domain-name Default --os-user-domain-name Default \
  --os-project-name admin --os-username admin token issue
```

服务器返回结果如下:
```
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                                                                                   |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2018-02-24T13:29:46+0000                                                                                                                                                                |
| id         | gAAAAABakVq6QVSL41hxXIefptBk8UBPxdLwRqoN_5pA8tR2pM4ZJAtl8PtRoY1alCGJGUU68FCAT9OP1Z0PtcJvD2BYjh2xVfbbsMPviQ8sR7cCzdjQ8tAx3cAibHU1nMMCCYYrhft-YEzswPUX9ul98tX8MDTa-Tx6OVZPhCKq8McLJ7xeetI |
| project_id | cf4ba4ae11124667a72c819a0ad99a3d                                                                                                                                                        |
| user_id    | 2afa305829ff4ec58d0edbb7343a6fb4                                                                                                                                                        |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```


使用demo用户请求token  
```
openstack --os-auth-url http://192.168.1.11:5000/v3 \
  --os-project-domain-name Default --os-user-domain-name Default \
  --os-project-name demo --os-username demo token issue
```

服务器返回结果如下:
```
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                                                                                   |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2018-02-24T13:31:05+0000                                                                                                                                                                |
| id         | gAAAAABakVsJTtVEEhH3kriFhZ8zrwYmqaydvey2_lgy0kE7NTkX5yAcXqrcv8B6EnF-odLKTT5aVTyuNZpXFE9DNpTh-v2V-ToWI8s636XtH-owTt22KPgf6QP1Vn0Mo1qQZWuu1SdF6aLiEqKp93HSUESx_kHq8UouASTqYH4bQU2a9MkdyQo |
| project_id | a58b699ddbbf44af897d416855389f7e                                                                                                                                                        |
| user_id    | ee910ae96d5147e89877c1d3455c205d                                                                                                                                                        |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

配置环境变量脚本

> vi admin-openrc

```bash
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=asd
export OS_AUTH_URL=http://192.168.1.11:35357/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```

> vi demo-openrc

```bash
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=asd
export OS_AUTH_URL=http://192.168.1.11:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```

加载admin环境变量

> . admin-openrc

申请token

> openstack token issue

服务器返回结果如下:
```
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                                                                                   |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2018-02-24T13:35:23+0000                                                                                                                                                                |
| id         | gAAAAABakVwL51zeofRTcXKf-ruS13hYvkyL2y2OuHWflihgXla5dHOcVCP8-edk39Adl9MsJkQBgl1yHLXy5qZDvv1XDAWZJjtJmYZrXXYwXt0M6QvZmGHDf84mkuIycfZ4nJMhWerrOpp5PVhkPuYCtMPMf0JsV-RlCgpYGfJ-Ayf4wfQ7hvc |
| project_id | cf4ba4ae11124667a72c819a0ad99a3d                                                                                                                                                        |
| user_id    | 2afa305829ff4ec58d0edbb7343a6fb4                                                                                                                                                        |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```
