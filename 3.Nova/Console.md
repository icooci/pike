# All Console部署

<br />

## 控制节点配置

安装console组件
> apt install nova-novncproxy nova-xvpvncproxy nova-spiceproxy

<br />

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

验证操作
---

加载admin变量
> . admin-openrc 

获取novnc console地址
> nova get-vnc-console c1 novnc
```
+-------+-----------------------------------------------------------------------------------+
| Type  | Url                                                                               |
+-------+-----------------------------------------------------------------------------------+
| novnc | http://192.168.1.11:6080/vnc_auto.html?token=1eb89cf5-228c-4c0c-9009-a07160405b35 |
+-------+-----------------------------------------------------------------------------------+
```

获xvpvnc console地址
> nova get-vnc-console c1 xvpvnc
```
+--------+-----------------------------------------------------------------------------+
| Type   | Url                                                                         |
+--------+-----------------------------------------------------------------------------+
| xvpvnc | http://192.168.1.11:6081/console?token=237c20a6-cee7-4758-9199-db927ddbb6ec |
+--------+-----------------------------------------------------------------------------+
```

获取spice console地址
> nova get-spice-console c1 spice-html5
```
+-------------+-------------------------------------------------------------------------------------+
| Type        | Url                                                                                 |
+-------------+-------------------------------------------------------------------------------------+
| spice-html5 | http://192.168.1.11:6082/spice_auto.html?token=cab96294-b285-4bba-8bdf-332683e2a25a |
+-------------+-------------------------------------------------------------------------------------+
```


![image](https://github.com/icooci/pike/blob/master/3.Nova/Snapshot/novnc.png)

![image](https://github.com/icooci/pike/blob/master/3.Nova/Snapshot/xvpviewer.png)

![image](https://github.com/icooci/pike/blob/master/3.Nova/Snapshot/xvpviewer_win.png)

![image](https://github.com/icooci/pike/blob/master/3.Nova/Snapshot/spice-html5.png)

