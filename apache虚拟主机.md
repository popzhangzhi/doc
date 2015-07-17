    1．基于相同IP不同Port的虚拟主机
    
    1）vi/etc/http/conf/httpd.conf
    
    2）将Listen字段改为
    Listen80
    Listen8888
    （以上设置表示使用80以及8888端口）
    
    3）更改虚拟主机部分为：
    <VirtualHost192.168.0.1:80>
    DocumentRoot/var/www/html/website1
    </VirtualHost>
    <VirtualHost192.168.0.1:8888>
    DocumentRoot/var/www/html/website2
    </VirtualHost>
    
    4）保存以上设置
    
    5）创建目录以及页面文件：
    mkdir /var/www/html/website1
    mkdir /var/www/html/website2
    cd /var/www/html/website1
    cat>index.html<<EOF
    >website1
    >EOF
    cd /var/www/html/website2
    cat>index.html<<EOF
    >website2
    >EOF
    
    （注：在/etc/httpd/conf/httpd.conf中有DirectoryIndexindex.htmlindex.html.var，表示只读index.html，而不读index.htm，切记）
    
    6）servicehttpdrestart
    
    完成以上设置后，可以通过以下方式访问：
    
    1）打开浏览器
    
    2）输入http://192.168.0.1:80以及http://192.168.0.1:8888
    
    2．基于相同Port不同IP的虚拟主机
    
    1）不同IP地址的配置：
    cd /etc/sysconfig/network-scripts
    cp ifcfg-eth0 ifcfg-eth0:1
    vi ifcfg-eth0:1
    将eth0:1更改为：
    DEVICE=eth0:1
    ONBOOT=YES
    BOOTPROTO=static
    IPADDR=192.168.0.2
    NETMASK=255.255.255.0
    
    2）servicenetworkrestart
    
    3）vi/etc/httpd/conf/httpd.conf
    
    4）更改虚拟主机部分为：
    <VirtualHost192.168.0.1:80>
    DocumentRoot/var/www/html/website1
    </VirtualHost>
    <VirtualHost192.168.0.2:80>
    DocumentRoot/var/www/html/website2
    </VirtualHost>
    
    5）创建目录以及页面文件：
    #mkdir–p/var/www/html/website1
    #mkdir–p/var/www/html/website2
    #cd/var/www/html/website1
    #cat>index.html<<EOF
    >website1
    >EOF
    #cd/var/www/html/website2
    #cat>index.html<<EOF
    >website2
    >EOF
    
    完成以上设置后，可以通过以下方式访问：
    
    1）打开浏览器
    
    2）输入http://192.168.0.1:80以及http://192.168.0.2:80
    
    3．基于域名的虚拟主机的访问
    
    1）vi/etc/http/conf/httpd.conf
    
    2）更改虚拟主机部分为：
    NameVirtualHost192.168.0.1
    <VirtualHostwww1.example.com>
    DocumentRoot/var/www/html/website1
    ServerNamewww1.example.com
    </VirtualHost>
    <VirtualHostwww2.example.com>
    DocumentRoot/var/www/html/website2
    ServerNamewww2.example.com
    </VirtualHost>
    
    （注：以上设置中NameVirtualHost不可以省略）
    
    3）创建目录以及页面文件：
    #mkdir–p/var/www/html/website1
    #mkdir–p/var/www/html/website2
    #cd/var/www/html/website1
    #cat>index.html<<EOF
    >website1
    >EOF
    #cd/var/www/html/website2
    #cat>index.html<<EOF
    >website2
    >EOF
    
    4）完成以上设置后，可以通过以下方式访问：
    
    1）打开浏览器
    
    2）输入http://www1.example.com以及http://www2.example.com
