### k8s集群搭建

#### 1.kube-prox开启ipvs前置条件

	modprobe br_netfilter

	vim  /etc/sysconfig/modules/ipvs.modules

##### 插入

	cat > a.txt <<EOF
	#!/bin/bash 
	modprobe -- ip_vs 
	modprobe -- ip_vs_rr 
	modprobe -- ip_vs_wrr 
	modprobe -- ip_vs_sh 
	modprobe -- nf_conntrack_ipv4 
	EOF
##### 增加权限，并且运行。最后查看
	chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules &&
	lsmod | grep -e ip_vs -e nf_conntrack_ipv4

#### 2.安装docker
	yum install -y yum-utils device-mapper-persistent-data lvm2
	yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
	yum update -y && yum install -y docker-ce	
    创建 /etc/docker 目录
	mkdir /etc/docker
	配置 daemon.
	cat > /etc/docker/daemon.json <<EOF 
	{ "exec-opts": ["native.cgroupdriver=systemd"], 
	"log-driver": "json-file", 
	"log-opts": { "max-size": "100m" } 
	}EOF

	mkdir -p /etc/systemd/system/docker.service.d
	重启docker服务
	systemctl daemon-reload && systemctl restart docker && systemctl enable docker
#### 导入依赖的docker相关镜像

	解压离线adm文件，用脚步导入到docker里
	sh脚步导入kubeadm的 images

	vim load-kubeadm-image.sh

	#!/bin/bash
	cd /home/k8s-master01/offlineZip/kubeadm-basic.images
	ls . > /tmp/image-list.txt
	for i in $( cat /tmp/image-list.txt )
	do
		docker load -i $i
	done

	rm -rf /tmp/image-list.txt
	运行sh
	load-kubeadm-image.sh
##### 高可用支持（多个master节点。负载均衡，高可用）
    在docker内，导入 ha-porxy保证负载均衡。拥有8种策略
    导入keepalived支持高可用。以抢占虚拟ip的形式来保证高可用
    docker load -i 离线包

    使用对应的脚步，开启ha-proxy和keepalived。
    注意配置虚拟ip。集群服务器，和端口安全检查，防止启动过程中的报错



#### 3.安装 Kubeadm （主从配置）
	cat <<EOF > /etc/yum.repos.d/kubernetes.repo 
	[kubernetes] 
	name=Kubernetes 
	baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64 
	enabled=1 
	gpgcheck=0 
	repo_gpgcheck=0 
	gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg 
	EOF

	ps: 由于官网未开放同步方式, 可能会有索引gpg检查失败的情况, 这时请用 yum install -y --nogpgcheck kubelet kubeadm kubectl 安装

	yum -y install kubeadm-1.15.1 kubectl-1.15.1 kubelet-1.15.1 
	systemctl enable kubelet.service


#### 4.初始化主节点
    输出默认配置到新建文件
	kubeadm config print init-defaults > kubeadm-config.yaml 

	localAPIEndpoint: 
		advertiseAddress: 192.168.66.10
	 kubernetesVersion: v1.15.1 
	 networking: 
	 	podSubnet: "10.244.0.0/16" 
	 	serviceSubnet: 10.96.0.0/12 
	 --- apiVersion: kubeproxy.config.k8s.io/v1alpha1 
	 kind: KubeProxyConfiguration 
	 featureGates: 
	 	SupportIPVSProxyMode: true 
	 mode: ipvs 

	 如上修改 添加到podSubnet: "10.244.0.0/16",原因是要加入flannel 扁平化网络的默认网段
	 添加ipvs支持

	 初始化并且输出。同时加入自然颁发证书参数
	 kubeadm init --config=kubeadm-config.yaml --upload-certs | tee kubeadm-init.log

##### 5.按照输出log 加入主节点和工作节点

#### 6.部署扁平网络flannel

	wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
	kubectl apply -f kube-flannel.yml



### docker 和k8s
#### docker 架构
            docker 后台服务 （docker deamon）
            Rest API
            交互式命令行界面（Docker Cli）
            Image：是只读的指令模板，用于创建docker contianre。通常一个镜像会继承领一个镜像，然后扩展自定义的指令，比如，创建一个继承ubuntu的镜像
                在安装一个apache tomcat 的服务和自己的应用程序，同时修改配置使程序可以允许。用dockerfile 文件可以自定义创建自己的镜像。当修改                     dockerfile文件，重新编译时，仅有那些被修改的层（layer）才会被重新编译，这个就是docker镜像轻量级、体积小、速度快的原因。
            container：image运行的一个实例。可用docker api或者cli 来创建运行、停止、移动、删除容器。容器可用绑定一个或者多个网络（network）或者挂                    载一个磁盘卷（volume），可用通过继承contain来创建一个新的镜像。
            service：允许docker后台服务伸缩扩展容器。组成多主多从的集群，通过service来定义集群的参数。
#### docker常用命令
        docker image [options] [repository[:tag]] 查看本地镜像 
        docker image pull [options] NAME[:tag|@digest] 拉取镜像
        docker run [options] IMAGE [command] [arg...] 运行镜像生成容器
        docker commit [options] CONTAINER [repository[:tag]] 用commit创建镜像或者用--change来修改已创建的镜像
##### 通过dockerfile来创建镜像
                docker image build [options] PATH | URL| -
                首先创建文件dockerfile，并写入以前信息：
                    #From 告诉docker 此镜像基于docker/whalesay
                    From docker/whalesay:lastest
                    #添加fortune程序来命令行输出
                    RUN apt-get -y update & apt-get install -y fortunes
                    CMD /usr/game/fortune -a | cowsay
                 编写完dockerfile后使用docker image build 来生成镜像
                 docker build -t docker-whale .
                 -t 来标识添加tag， . 为dockerfile所在的路径
          docker login [options] [servier] 登录docker hub 仓库
          docker push [options] NAME[:tag] 上传镜像到docker hub
          docker start [options] CONTAINER [container...] 启动运行容器
          docker logs [options] CONTAINER 获取容器的日志信息
          docker attach [options] CONTAINER 进入容器
          docker exec [options] CONTAINER COMMAND [arg...] 在运行的容器中执行命令，在暂停的容器中执行会报错
          docker stop [options] CONTAINER [container...] 停止容器
          docker rm [options] CONTAINER [container...] 删除容器
##### 服务的常用命令
            docker service create [options] IMAGE [command] [arg...] 创建新服务
            
          docker swarm init [options] 集群初始化，原生集群工具，使用标准的docker api。意味着容器能使用docker run命令启动。swarm会选择合适的
            主机来运行容器。同时使用compose也能在swarm使用。
          docker-compose 快速构架环境，依赖，构建服务，并且启动服务。用于快速构建节点
### k8s
        1.create deployment :k8s master会将deployment创建好的实例调度到各个集群节点中
        2.创建deploy的时候，k8s会创建一个pod来托管应用。
            pod是抽象概念由多个容器组成在一起的。并且与其他pod共享资源：卷 网络 contianer
            node是k8s的工作节点，可以是虚拟机或者物理机。每个node由master管理。node上可以有多个pod。master自动调度node中的pod。
            每个node上至少运行这kubelet和container runtime（docker）
            kubectl get - 列出资源
            kubectl describe - 显示资源的详细信息
            kubectl logs - 打印pod中的容器日志
            kubectl exec - pod中容器内部执行命令
         3.services：定义了pod的逻辑分组和访问pod的策略。 例如 kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080
            通过type在serviceSpec中指定所需要的类型：
                ClusterIP（默认） 在集群内部ip上暴露服务。此类型的Service只能从集群中访问
                NodePort 通过每个node上的ip和静态端口（NodePort）暴露服务。该服务会路由到对应的ClusterIP服务上。同时ClusterIP会自动创建，通过 NodeIP：NodePort。可以从集群外部直接访问。
                LoadBalancer 使用云提供商的负载均衡器，可以向外暴露服务，外部的负载均衡器可以路由到NodePort服务和ClusterIP服务
                ExternalName - 通过返回 CNAME 和它的值，可以将服务映射到 externalName 字段的内容，没有任何类型代理被创建。这种类型需要v1.7版本或更高版本kube-dnsc才支持。
          4.使用 kubectl scale deployName --replicas=数字 来伸缩应用
          5.使用 kubectl set image deployName NewImageVersion 来升级应用。例如 kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2
                kubectl rollout status deployments/kubernetes-bootcamp 来回滚升级
                 
                
        
