##### 简介

```shell
- 基于vrrp协议实现，会将多台设备虚拟成一个组，对外提供统一的vip地址

- 选主  
  1.vip是否绑定在某台主机上  
  2.state（master/backup） 
  3.priority    
  4.ip大小（大的胜出） 

- keepalived正常启动的时候，共启动3个进程：
  1. 父进程，负责监控子进程 VRRP 和 Healthcheck
  2. VRRP子进程：广播vrrp包， 绑定或去除vip
  3. Healthcheck子进程：检查各自服务器的健康状况，如果检查到master不可用，会通知本机VRRP子进程删除通告，并且去掉虚拟IP，转换为BACKUP状态
     
  * Healthcheck子进程负责衍生子进程，执行vrrp_script中的check脚本
          |___check孙子进程1
          |___check孙子进程n
```

##### 配置文件

```shell
global_defs {                                
     router_id keepalived                     
     vrrp_mcast_group4 224.0.0.118       
}

vrrp_script keepalived_check {      
     script "/etc/keepalived/scripts/check.sh"
     interval 10
     timeout 10
}

vrrp_instance lxp_test {                  
     state BACKUP                            
     interface ens192                      
     virtual_router_id 203                 
     priority 120                            
     preempt
     advert_int 1                         
     authentication {                     
        auth_type PASS                   
        auth_pass 1111                   
     } 
     virtual_ipaddress { 
        172.122.1.10
     }                                    
     track_script {                       
        keepalived_check                        
     }                                    
     notify_master "tomaster.sh"
     notify_fault "tofault.sh"
     notify_stop "tostop.sh"
}
```

