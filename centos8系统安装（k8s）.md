### 配置主机

#### 打开网卡文件设置ip以及网关等
	 vim /etc/sysconfig/network-scripts/ifcfg-ens33

```
	TYPE=Ethernet
	PROXY_METHOD=none
	BROWSER_ONLY=no
	BOOTPROTO=static
	DEFROUTE=yes
	IPV4_FAILURE_FATAL=no
	IPV6INIT=yes
	IPV6_AUTOCONF=yes
	IPV6_DEFROUTE=yes
	IPV6_FAILURE_FATAL=no
	IPV6_ADDR_GEN_MODE=stable-privacy
	NAME=ens33
	UUID=43eda443-d5cc-4803-b462-3fae6fb51d20
	DEVICE=ens33
	ONBOOT=yes
	IPADDR=192.168.137.100
	NETMAKSK=255.255.255.0
	GATEWAY=192.168.137.1
	DNS1=192.168.137.1
```
	设置BOOTPROTO=static  //表示静态ip
	设置ONBOOT=yes //开机自启
	添加 IPADDR NETMASK GATEWAY DNS1 网络相关数据
	
#### 设置系统主机名和host文件相互解析
	hostnamectl set-hostname k8s-master01

##### 添加对应的ip以及hostname
	vim /etc/hosts

#### 安装相关的依赖包

		yum install -y conntrack  ipvsadm ipset jq iptables curl sysstat libseccomp wget vim net-tools git

#### 设置防火墙为 Iptables 并设置空规则

	systemctl stop firewalld && systemctl disable firewalld

	yum -y install iptables-services && systemctl start iptables && systemctl enable iptables && iptables -F && service iptables save

#### 关闭 SELINUX (关闭虚拟内存，k8s强制要求)

	swapoff -a && sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab 

	setenforce 0 && sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config

#### 调整内核参数，对于 K8S

	vim kubernetes.conf 

##### 插入

	net.bridge.bridge-nf-call-iptables=1 
	net.bridge.bridge-nf-call-ip6tables=1 
	net.ipv4.ip_forward=1 
	net.ipv4.tcp_tw_recycle=0 
	vm.swappiness=0 # 禁止使用 swap 空间，只有当系统 OOM 时才允许使用它 
	vm.overcommit_memory=1 # 不检查物理内存是否够用 
	vm.panic_on_oom=0 # 开启 OOM 
	fs.inotify.max_user_instances=8192 
	fs.inotify.max_user_watches=1048576 
	fs.file-max=52706963 
	fs.nr_open=52706963 
	net.ipv6.conf.all.disable_ipv6=1 
	net.netfilter.nf_conntrack_max=2310720

##### 移动配置文件并且内核生效该文件
	cp kubernetes.conf /etc/sysctl.d/kubernetes.conf 
	sysctl -p /etc/sysctl.d/kubernetes.conf

#### 调整系统时区
	# 设置系统时区为 中国/上海 
	timedatectl set-timezone Asia/Shanghai 
	# 将当前的 UTC 时间写入硬件时钟 
	timedatectl set-local-rtc 0 
	
##### 重启依赖于系统时间的服务 

	systemctl restart rsyslog 
	systemctl restart crond

#### 关闭邮件服务（可选）
	systemctl stop postfix && systemctl disable postfix

#### 设置 rsyslogd 和 systemd journald 2种日志文件只开启一个，当前只开启后者

	mkdir /var/log/journal # 持久化保存日志的目录 
	mkdir /etc/systemd/journald.conf.d

	vim /etc/systemd/journald.conf.d/99-prophet.conf 

##### 插入

	[Journal]
	# 持久化保存到磁盘 
	Storage=persistent 
	# 压缩历史日志 
	Compress=yes 
	SyncIntervalSec=5m 
	RateLimitInterval=30s 
	RateLimitBurst=1000 
	# 最大占用空间 10G S
	ystemMaxUse=10G 
	# 单日志文件最大 200M 
	SystemMaxFileSize=200M 
	# 日志保存时间 2 周 
	MaxRetentionSec=2week 
	# 不将日志转发到 
	syslog ForwardToSyslog=no

##### 启用

	systemctl restart systemd-journald

