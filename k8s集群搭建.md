### k8s集群搭建

#### kube-prox开启ipvs前置条件

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

####安装docker
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
#### 安装 Kubeadm （主从配置）
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

#### 初始化主节点
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

	##### 按照输出log 加入主节点和工作节点

	####部署扁平网络flannel

	wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
	kubectl apply -f kube-flannel.yml



