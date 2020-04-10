「闭包」和「匿名」
___
	「闭包」和「匿名」这两个概念并不等价（虽然 PHP里面对应的都是同一个东西）。
	匿名是指这个函数可以像变量一样操作，例如可以赋值给另一个变量、作为参数传递、作为函数的返回值等。	
	闭包是指这个函数可以从上下文中捕获变量（而不是通过参数来传递），具体到 PHP 上就是 use 这个子句做的事情。
```
	修改PHP上传文件大小限制的方法
	1. 一般的文件上传,除非文件很小.就像一个5M的文件,很可能要超过一分钟才能上传完.
	但在php中,默认的该页最久执行时间为 30 秒.就是说超过30秒,该脚本就停止执行.
	这就导致出现 无法打开网页的情况.这时我们可以修改 max_execution_time
	在php.ini里查找
	max_execution_time
	默认是30秒.改为
	max_execution_time = 0
	0表示没有限制
	2. 修改 post_max_size 设定 POST 数据所允许的最大大小。此设定也影响到文件上传。
	php默认的post_max_size 为2M.如果 POST 数据尺寸大于 post_max_size $_POST 和 $_FILES superglobals 便会为空.
	查找 post_max_size .改为
	post_max_size = 150M
	3. 很多人都会改了第二步.但上传文件时最大仍然为 8M.
	为什么呢.我们还要改一个参数upload_max_filesize 表示所上传的文件的最大大小。
	查找upload_max_filesize,默认为8M改为
	upload_max_filesize = 100M
	另外要说明的是,post_max_size 大于 upload_max_filesize 为佳.
	Linux下默认安装lamp路径
```	
	apache:
	如果采用RPM包安装，安装路径应在 /etc/httpd目录下
	apache配置文件:/etc/httpd/conf/httpd.conf
	Apache模块路径：/usr/sbin/apachectl
	web目录:/var/www/html
	如果采用源代码安装，一般默认安装在/usr/local/apache2目录下
	
	php:
	如果采用RPM包安装，安装路径应在 /etc/目录下
	php的配置文件:/etc/php.ini
	如果采用源代码安装，一般默认安装在/usr/local/lib目录下
	php配置文件: /usr/local/lib/php.ini
	或/usr/local/php/etc/php.ini
	
	mysql:
	如果采用RPM包安装，安装路径应在/usr/share/mysql目录下
	mysqldump文件位置：/usr/bin/mysqldump
	mysqli配置文件:
	/etc/my.cnf或/usr/share/mysql/my.cnf
	mysql数据目录在/var/lib/mysql目录下
	如果采用源代码安装，一般默认安装在/usr/local/mysql目录下
	
	Linux 下 LAMP YUM安装 centOs-5.x
	1、备份CentOS-Base.repo
		cd /etc/yum.repos.d/ 
		cp CentOS-Base.repo CentOS-Base.repo.bak
		2、替换源
		用vim打开CentOS-Base.repo，并将内容清空，然后将下面的内容复制进去，并保存
		[base] 
		name=CentOS-Base 
		baseurl=http://mirrors.sohu.com/centos/$releasever/os/$basearch/ 
		gpgcheck=1 
		gpgkey=http://mirrors.sohu.com/centos/RPM-GPG-KEY-CentOS-5 
		#released updates 
		[updates] 
		name=CentOS-Updates 
		baseurl=http://mirrors.sohu.com/centos/$releasever/updates/$basearch/ 
		gpgcheck=1 
		gpgkey=http://mirrors.sohu.com/centos/RPM-GPG-KEY-CentOS-5 
		#packages used/produced in the build but not released 
		[addons] 
		name=CentOS-Addons 
		baseurl=http://mirrors.sohu.com/centos/$releasever/addons/$basearch/ 
		gpgcheck=1 
		gpgkey=http://mirrors.sohu.com/centos/RPM-GPG-KEY-CentOS-5 
		#additional packages that may be useful 
		[extras] 
		name=CentOS-Extras 
		baseurl=http://mirrors.sohu.com/centos/$releasever/extras/$basearch/ 
		gpgcheck=1 
		gpgkey=http://mirrors.sohu.com/centos/RPM-GPG-KEY-CentOS-5 
		#additional packages that extend functionality of existing packages 
		[centosplus] 
		name=CentOS-Plus 
		baseurl=http://mirrors.sohu.com/centos/$releasever/centosplus/$basearch/ 
		gpgcheck=1 
		enabled=0 
		gpgkey=http://mirrors.sohu.com/centos/RPM-GPG-KEY-CentOS-5
		3、更新一下。
		yum -y update
		4、用yum安装Apache,Mysql,PHP.
		a、安装Apache
		yum install httpd httpd-devel
		安装完成后，用/etc/init.d/httpd start 启动apache
		设为开机启动:chkconfig httpd on
		b、安装mysql
		yum install mysql mysql-server mysql-devel
		完成后，用/etc/init.d/mysqld start 启动mysql
		设置mysql密码
		  <span style="color:#00ffff;"> <span style="color:#0099ff;">第一种方法</span></span>：
		root用户登录系统
		/usr/local/mysql/bin/mysqladmin -u root -p password 新密码
		enter password 旧密码
		   <span style="color:#0099ff;">第二种方法</span>：
		root用户登录mysql数据库
		mysql&gt; update mysql.user set passwordpassword=password（”新密码”）where User=”root”;
		mysql&gt; flush privileges;
		mysql&gt; quit ;
		   mysql忘记root密码如何处理?
		如果 MySQL 正在运行，首先结束mysql进程：
		killall mysqld
		启动 MySQL (非正常方式起动)：/usr/local/mysql/bin/mysqld_safe –skip-grant-tables &amp;
		这样就可以不需要密码进入 MySQL ：
		/usr/local/mysql/bin/mysql -u root -p　（要求输入密码时直接回车即可）
		mysql&gt; update user mysql.set passwordpassword=password(”新密码”) where user=”root”;
		mysql&gt; flush privileges;
		mysql&gt; quit;
		重新结束进程：
		killall mysqld
		用正常方式启动 MySQL ：/usr/local/mysql/bin/mysqld_safe -user=mysql &amp;
		update语句里的password=password(”新密码”)只有新密码三个字在操作时替换成我们要设置的密码，其它原样照写，之前我做失败的原因就在于把括号及前面的password给略掉造成的．它们的作用是使密码以加密的形式存储在数据库里。
		设为开机启动    
		chkconfig mysqld on
		c、安装php
		yum install php php-mysql php-common php-gd php-mbstring php-mcrypt php-devel php-xml
		/etc/init.d/httpd start
		4. 测试一下
		在/var/www/html/新建个test.php文件，将以下内容写入，然后保存。
		<!--?</p-->
		phpinfo();
		?&gt;
		然后在客户端浏览器里打开http://serverip/test.php，若能成功显示，则表示安装成功。
		
		5、yum 安装php遇到php53-common conflicts with php-common 
		yum 安装php的时候，用命令yum -y install php*遇到提示php53-common conflicts with php-common这个错误信息，这时候可以看到
		Error: php53-common conflicts with php-common
		You could try using --skip-broken to work around the problem
		You could try running: package-cleanup --problems
		package-cleanup --dupes
		这样的错误提示。
		这样在重新安装的时候 用命令
		yum -y install php*  --skip-broken 就可以解决问题了
		
		6、mysql遇到：
		Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2)
		a、<span style="line-height:1.5em;">/etc/rc.d/init.d/mysqld status</span>
		查看是否启动 mysql
		b、如果没有启动mysql，先启动mysql
		service mysqld start
	
	
	
	Linux上安装Apache+Php+Mysql的过程（php apache没问题 mysql 启动每秒重启错误）
	
	
	
	
	操作系统:centos 4.3
	软件列表:httpd-2.2.4.tar.gz, mysql-5.0.18.tar.gz, php-5.2.1.tar.gz
	一．linux下安装mysql
	
	安装步骤:
	1.groupadd mysql
	2.useradd -g mysql mysql
	3.tar zxvf myql-5.0.18.tar.gz
	4.cd mysql-5.0.18.tar.gz
	5../configure= prefix=/usr/local/mysql
	6.make
	7.make install
	8.cp support-files/my-medium.cnf /etc/my.cnf
	9.cd /usr/local/mysql
	10.bin/mysql_install_db --user=mysql
	11.chown -R root .
	12.chown -R mysql var
	13.chgrp -R mysql .
	14.bin/mysqld-safe --user=mysql &
	检查日志:more var/localhost.err:
	070129 15:05:58 mysqld started
	070129 15:05:58 InnoDB: Started; log sequence number 0 43655
	070129 15:05:58 [Note] /usr/local/mysql/libexec/mysqld: ready for connections.
	Version: '5.0.18-log' socket: '/tmp/mysql.sock' port: 3306 Source distribution
	查看进程:ps -aux
	root 17286 0.0 0.4 5144 1096 pts/2 S 15:05 0:00 /bin/sh bin/mysqld_safe --user=mysql
	mysql 17310 0.0 6.0 125272 15104 pts/2 Sl 15:05 0:00 /usr/local/myql/libexec/mysqld --basedir=/usr/local/myql --datadir=/u
	root 17546 0.0 0.8 9280 2200 ? SNs 15:20 0:00 cupsd
	root 17612 0.0 0.3 3876 748 pts/2 R+ 15:21 0:00 ps -aux
	关闭mysql:
	bin/mysqladmin shutdown
	STOPPING server from pid file /usr/local/myql/var/localhost.pid
	070129 15:22:28 mysqld ended
	[1]+ Done bin/mysqld_safe --user=mysql
	[root@localhost myql]# mysql
	Welcome to the MySQL monitor. Commands end with ; or \g.
	Your MySQL connection id is 2 to server version: 5.0.18-log
	Type 'help;' or '\h' for help. Type '\c' to clear the buffer.
	如果不行的话killall mysqld,在重新启动就OK了
	二．linux下安装apache
	1.tar -zxvf httpd-2.2.4.tar.gz
	2.cd httpd-2.2.4
	3. ./configure –prefix=/www --enable-so
	4.make clean
	5.make
	6.make install
	三．linux下安装php
	1.tar -zxvf php-5.2.1.tar.gz
	2.cd php-5.2.1
	3. ./configure --prefix=/usr/local/php --with-mysql=/usr/local/mysql --with-apxs2=/www/bin/apxs
	4.make clean
	5.make
	6.make install
	apache中加载php模块
	7 Setup your php.ini
	cp php.ini-dist /usr/local/lib/php.ini
	8.. Edit your httpd.conf to load the PHP module.
	For PHP 4:
	LoadModule php4_module modules/libphp4.so
	For PHP 5:
	LoadModule php5_module modules/libphp5.so
	9. Tell Apache to parse certain extensions as PHP.
	AddType application/x-httpd-php .php .phtml
	AddType application/x-httpd-php-source .phps(T002)
	
	
	21 个非常有用的 .htaccess 提示和技巧
	
	http://www.oschina.net/question/12_58586
	
	
	 
	 
	 
	 
	 
	 
	 
	
	
	2．基于相同Port不同IP的虚拟主机
	
	1）不同IP地址的配置：
	#cd/etc/sysconfig/network-scripts
	#cpifcfg-eth0ifcfg-eth0:1
	#viifcfg-eth0:1
	将eth0:1更改为：
	DEVICE=eth0:1
	ONBOOT=YES
	BOOTPROTO=static
	IPADDR=192.168.0.2
	NETMASK=255.255.255.0
	
	  DEVICE=eth0:0          #此处添加:0，保持和文件名一致，添加多个IP依次递增
	  ONBOOT=yes             #是否开机激活
	  BOOTPROTO=static       #静态IP，如果需要DHCP获取请输入dhcp
	  IPADDR=192.168.1.2     #此处修改为要添加的IP
	  NETMASK=255.255.255.0  #子网掩码根据你的实际情况作修改
	/etc/init.d/network reload
	
	
	
	
	2）service network restart
	
	3）vi /etc/httpd/conf/httpd.conf
	
	4）更改虚拟主机部分为：
	<VirtualHost 192.168.0.1:80>
	DocumentRoot /var/www/html/website1
	</VirtualHost >
	<VirtualHost 192.168.0.2:80>
	DocumentRoot /var/www/html/website2
	</VirtualHost>
	
	5）创建目录以及页面文件：
	#mkdir –p /var/www/html/website1
	#mkdir –p /var/www/html/website2
	#cd /var/www/html/website1
	#cat>index.html<<EOF
	>website1
	>EOF
	#cd/ var/www/html/website2
	#cat>index.html<<EOF
	>website2
	>EOF
	
	完成以上设置后，可以通过以下方式访问：
	
	1）打开浏览器
	
	2）输入http://192.168.0.1:80以及http://192.168.0.2:80
	
	
	让apache只允许域名访问而禁止IP实现方法
	2013-05-16 10:18:28     我来说两句       作者：tillzhang
	收藏     我要投稿
	让apache只允许域名访问而禁止IP实现方法
	 
	用apache搭建的WEB服务器，如何让网友只能通过设定的域名访问，而不能直接通过服务器的IP地址访问呢，通过查找，有两个方法可以实现，都是修改httpd.conf文件来实现的，下面举例说明。 
	方法一：在httpd.conf文件最后面，加入以下代码 
	　　　　　NameVirtualHost 211.*.*.* 
	　　　　　<VirtualHost 211.*.*.*> 
	　　　　　ServerName 211.*.*.* 
	　　　　　<Location /> 
	   　　　　 Order Allow,Deny 
	    　　　　Deny from all 
	　　　　　</Location> 
	　　　　　</VirtualHost>　　　　 
	　　　　　<VirtualHost 211.*.*.*> 
	　　　　　DocumentRoot "c:/web" 
	　　　　　ServerName tuan.coo8.com 
	　　　　　</VirtualHost>　　　 
	　　　说明：蓝色部分是实现拒绝直接通过211.*.*.*这个IP的任何访问请求，这时如果你用211.*.*.*访问，会提示拒绝访问。红色部分就是允许通过http://tuan.coo8.com/这个域名访问，主目录指向c:/web（这里假设你的网站的根目录是c:/web） 
	 
	　　　方法二：在httpd.conf文件最后面，加入以下代码 
	　　　　　NameVirtualHost 211.*.*.* 
	　　　　　<VirtualHost 211.*.*.*> 
	　　　　　DocumentRoot "c:/test" 
	　　　　　ServerName 211.*.*.* 
	　　　　　</VirtualHost>　　　　　 
	　　　　　<VirtualHost 211.*.*.*> 
	　　　　　DocumentRoot "c:/web" 
	　　　　　ServerName http://tuan.coo8.com 
	 
	　　　　　</VirtualHost>　　　　 
	　　　　 
	　　　说明：蓝色部分是把通过211.*.*.*这个IP直接访问的请求指向c:/test目录下，这可以是个空目录，也可以在里面建一个首页文件，如index.hmtl，首面文件内容可以是一个声明，说明不能通过IP直接访问。
	 
	       　注意：１. 直接复制粘贴的话可能会带有中文空格，请把这些多余的空格去掉。 
	 
	　　　　　　 2.  如果使用了负载均衡，限制的IP不要写外网IP,请填写内网IP。
	
	
	
