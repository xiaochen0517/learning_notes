# Nginx 高可用

## 1、安装工具包

### 1.1、 `GCC` 

```shell
yum -y install gcc
```

### 1.2、 `keepalive` 

```shell
yum -y install keepalive
```

## 2、修改配置

修改 `/etc/keepalive/keepalive.conf` 

```bash
global_defs {
	notification_email {
	  acassen@firewall.loc
	  failover@firewall.loc
	  sysadmin@firewall.loc
	}
	notification_email_from Alexandre.Cassen@firewall.loc
	smtp_ server 192.168.236.128 # 此处为服务器本机ip
	smtp_connect_timeout 30
	router_id LVS_DEVEL	# LVS_DEVEL这字段在/etc/hosts文件中看；通过它访问到主机
}

vrrp_script chk_http_ port {
	script "/usr/local/src/nginx_check.sh"
	interval 2   # (检测脚本执行的间隔)2s
	weight 2  #权重，如果这个脚本检测为真，服务器权重+2
}

vrrp_instance VI_1 {
	state BACKUP   # 备份服务器上将MASTER 改为BACKUP
	interface ens33 # 网卡名称
	virtual_router_id 51 # 主、备机的virtual_router_id必须相同
	priority 100   #主、备机取不同的优先级，主机值较大，备份机值较小
	advert_int 1	#每隔1s发送一次心跳
	authentication {	# 校验方式， 类型是密码，密码1111
        auth_type PASS
        auth_pass 1111
    }
	virtual_ipaddress { # 虛拟ip
		192.168.236.50 # VRRP H虛拟ip地址
	}
}
```

> 注意配置的虚拟ip的网段，必须和服务器主机网段一致。

修改 `/etc/hosts` 文件，添加以下内容：

```bash
127.0.0.1 LVS_DEVEL
```

新增脚本文件 `/usr/local/src/nginx_check.sh` 

```bash
#! /bin/bash
A=`ps -C nginx -no-header | wc - 1`
if [ $A -eq 0];then
	/usr/local/nginx/sbin/nginx
	sleep 2
	if [`ps -C nginx --no-header| wc -1` -eq 0 ];then
		killall keepalived
	fi
fi
```

> 访问虚拟 `ip` 进行测试，然后关闭其中一台服务器的 `nginx` ，即可自动切换。