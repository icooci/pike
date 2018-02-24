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

配置apache服务器
---

编辑apache2配置文件
> /etc/apache2/apache2.conf

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
