## keepalived 配置说明

```shell

global_defs {
    router_id LVS_1
}
vrrp_sync_group test {
    group {
        mysql_ha
    }
}

vrrp_instance mysql_ha {
    state BACKUP 
    interface enp0s8 
    virtual_router_id 61
    priority 150 
    nopreempt 
    advert_int 1 
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress { # vip, 虚拟ip, 在keepalived实例所在的机器之间漂移
        192.168.56.99
    }
    
    
    notify_backup "/root/keepalived/sh.sh BACKUP "  # keepalived状态转移的日志
    notify_master "/root/keepalived/sh.sh MASTER "
    notify_fault "/root/keepalived/sh.sh FAULT "
    notify "/root/keepalived/sh.sh  "
}

virtual_server 192.168.56.99 80 {
    delay_loop 2
    lb_algo rr
    lb_kind DR
    persistence_timeout 20
    protocol TCP
    real_server 192.168.56.101 80 { 
        weight 3
        notify_down "/root/keepalived/kill_keepalived.sh  " # 检测到本机的应用服务(nginx)不可用时， kill掉本机的keepalived， 实现vip（虚拟ip）漂移到备用机器!!!
        
        TCP_CHECK {
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
            connect_port 80
        }
    }

    real_server 192.168.56.102 80 { 
        weight 3
        notify_down "/root/keepalived/sh.sh MYSQL_DOWN " 
        
        TCP_CHECK {
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
            connect_port 80
        }
    }

}



```