```shell
1. 网络实现 （2020-10-12 ~ 10.18）
    a. 网络命名空间：虚拟的网络环境
    b. Veth设备对：成对出现，一端发送数据，对端能进行接收，用于不同网络命名空间通信
    c. 网桥：能连接多个网络接口，并在接口间进行数据报文转发
    d. iptables: Linux主机上的防火墙机制，俗称软墙，能对主机上进出的数据包按规则进行转发、过滤、丢弃等
    e. 路由： ip地址寻址路径。相关信息存在于路由表中
```

