全Console部署

## 控制节点配置

安装console组件
> apt install nova-novncproxy nova-xvpvncproxy nova-spiceproxy

## 计算节点配置

修改nova配置
> vi /etc/nova/nova.conf

```
...
[vnc]
enabled = False
vncserver_listen = 0.0.0.0
vncserver_proxyclient_address = $my_ip
novncproxy_base_url = http://192.168.1.11:6080/vnc_auto.html
xvpvncproxy_base_url = http://192.168.1.11:6081/console

[spice]
enabled = True
server_listen = 0.0.0.0
server_proxyclient_address = $my_ip
html5proxy_base_url = http://192.168.1.11:6082/spice_auto.html
...
```
