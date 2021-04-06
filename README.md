# Kubernetes

k8s是谷歌在2014年开源的容器化集群管理系统

+ 自动装箱 水平扩展 自我修复

  基于容器对应用环境进行自动部署

  当容器失败时会对容器进行重启，当所部署的Node节点有问题时会对容器进行重新部署和重新调度

  当容器未通过监控检查时，会关闭此容器直到正常运行时才会对外提供服务。

  通过简单的命令、用户UI界面或基于CPU等资源使用情况，对应用容器进行规模扩大或规模剪裁。

+ 服务发现和负载均衡

  Service

+ 自动发布和回滚

+ 集中化配置管理和密钥管理

+ 存储编排

+ 任务批处理运行

## 入门

### 四组概念

+ controller
  + 确保预期的pod副本数量
  + 无状态应用部署
    + 即插即用
  + 有状态应用部署
    + 依赖于存储或网络ip必须固定
  + 确保所有的node运行同一个pod
  + 一次性任务和定时任务

+ Pod/Pod控制器

  + Pod

    是k8s能够被运行的最小逻辑单元

    一个Pod里面可以运行多个容器 共享UTS NET IPC名称空间

    一个pod里运行多个容器 又叫sidecar

    共享网络：pod中的所有容器共享网络资源，比如容器A监听80端口，同在一个Pod下的容器B也会监听80端口

    生命周期是短暂的

  + Pod控制器

    Pod控制器是Pod启动的一种模板 用来保证在k8s里启动的Pod应始终按照人们的预期运行(副本数、生命周期、健康状态检查)

    Deployment

    DaemonSet （每个节点起一份）

    ReplicaSet   （Deployment管 ReplicaSet，ReplicaSet管pod）

    StatefulSet   （管理有状态应用的）

    Job

    Cronjob

+ Name/Namespace

  + Name

    由于K8S内部,使用“资源”来定义每一种逻辑概念(功能)故每种"资源”, 都应该有自己的"名称”

    "资源”有api版本( apiVersion )类别( kind )、元数据( metadata)、定义清单( spec)、状态( status )等配置信息

    "名称”通常定义在"资源”的"元数据”信息里

  + Namespace

    随着项目增多、人员增加、集群规模的扩大,需要- -种能够隔离K8S内各种"资源”的方法，这就是名称空间

    名称空间可以理解为K8S内部的虚拟集群组

    不同名称空间内的"资源”名称可以相同,相同名称空间内的同种“资源”，”名称” 不能相同

    合理的使用K8S的名称空间,使得集群管理员能够更好的对交付到K8S里的服务进行分类管理和浏览

    K8S里默认存在的名称空间有: default、 kube-system、 kube-public

    查询K8S里特定“资源”要带上相应的名称空间

+ Label/Label选择器

  + label

    标签是k8s特色的管理方式,便于分类管理资源对象。

    一个标签可以对应多个资源，一个资源也可以有多个标签,它们是多对多的关系。

    一个资源拥有多个标签,可以实现不同维度的管理。

    标签的组成: key=value(值不能多余64个字节字母数字开头 中间只能是 - _ .）

    与标签类似的,还有一种“注解” ( annotations )

  + label便签器

    - 给资源打上标签后,可以使用标签选择器过滤指定的标签

    - 标签选择器目前有两个:基于等值关系(等于、不等于)和基于集合关系(属于、不属于、存在)

    - 许多资源支持内嵌标签选择器字段

      matchLabels

      matchExpressions

+ Service/Ingress

  + Service

    在K8S的世界里,虽然每个Pod都会被分配一个单独的IP地址,但这个IP地址会随着Pod的销毁而消失

    Service (服务)就是用来解决这个问题的核心概念

    一个Service可以看作-一组提供相同服务的Pod的对外访问接口

    Service作用于哪些Pod是通过标签选择器来定义的

  + Ingress

    Ingress是K8S集群里工作在OSI网络参考模型下,第7层的应用,对外暴露的接口

    Service只能进行L4流量调度,表现形式是ip+port

    Ingress则可以调度不同业务域、 不同URL访问路径的业务流量

![img](images/1610788668698-88af2887-1f21-4953-8c9a-d7852496cdcb.svg)

## 核心组件

配置存储中心→etcd服务

高可用奇数个

### 主控(master)节点

#### kube-apiserver

集群统一入口

- 提供了集群管理的RESTAPI接口(包括鉴权、数据校验及集群状态变更)交给etcd存储
- 负责其他模块之间的数据交互,承担通信枢纽功能
- 是资源配额控制的入口
- 提供完备的集群安全机制

#### kube-controller-manager

处理集群中常规的一些任务，一个资源对应一个控制器

- 由一系列控制器组成,通过apiserver监控整个集群的状态,并确保集群处于预期的工作状态

  Node Controller

  Deployment Controller

  Service Controller

  Volume Controller

  Endpoint Controller

  Garbage Controller

  Namespace Controller

  Job Controller

  Resource quta Controller

#### kube-scheduler

- 主要功能是接收调度pod到适合的运算节点上
- 预算策略( predict )
- 优选策略( priorities )、、

#### etcd

存储系统 用于保存集群相关的数据

### 运算(Worker)节点

#### kube-kubelet

- 简单地说, kubelet的主要功能就是定时从某个地方获取节点上pod的期望状态(运行什么容器、运行的副本数量、网络或者存储如何配置等等) ,并调用对应的容器平台接口达到这个状态
- 定时汇报当前节点的状态给apiserver,以供调度的时候使用
- 镜像和容器的清理工作，保证节点上镜像不会占满磁盘空间，退出的容器不会占用太多资源

#### Kube-proxy

- 是K8S在每个节点 上运行网络代理, service资源的载体
- **建立了pod网络和集群网络的关系**( clusterip >podip )
- 常用三种流量调度模式

  Userspace (废弃)

  Iptables (濒临废弃)（绝大部分公司在用）

  Ipvs(推荐)

- 负责建立和删除包括更新调度规则、通知apiserver自己的更新,或者从apiserver哪里获取其他kube- proxy的调度规则变化来更新自己的

![image-20210322162545463](images/image-20210322162545463.png)

![image-20210322163508019](images/image-20210322163508019.png)

## 搭建k8s

### kubeadm

|        |               |
| ------ | ------------- |
| master | 192.168.52.26 |
| node1  | 192.168.52.27 |
| node2  | 192.168.52.28 |

kubeadm是官方社区推出的一个用于快速部署kubernetes集群的工具。

这个工具能通过两条指令完成一个kubernetes集群的部署：

```
# 创建一个 Master 节点
$ kubeadm init

# 将一个 Node 节点加入到当前集群中
$ kubeadm join <Master节点的IP和端口 >
```

#### 1. 安装要求

在开始之前，部署Kubernetes集群机器需要满足以下几个条件：

- 一台或多台机器，操作系统 CentOS7.x-86_x64
- 硬件配置：2GB或更多RAM，2个CPU或更多CPU，硬盘30GB或更多
- 可以访问外网，需要拉取镜像，如果服务器不能上网，需要提前下载镜像并导入节点
- 禁止swap分区

#### 2. 准备环境

| 角色   | IP           |
| ------ | ------------ |
| master | 192.168.1.11 |
| node1  | 192.168.1.12 |
| node2  | 192.168.1.13 |

```
# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

# 关闭selinux
sed -i 's/enforcing/disabled/' /etc/selinux/config  # 永久
setenforce 0  # 临时

# 关闭swap
swapoff -a  # 临时
sed -ri 's/.*swap.*/#&/' /etc/fstab    # 永久

# 根据规划设置主机名
hostnamectl set-hostname <hostname>

# 在master添加hosts
cat >> /etc/hosts << EOF
192.168.44.146 k8smaster
192.168.44.145 k8snode1
192.168.44.144 k8snode2
EOF

# 将桥接的IPv4流量传递到iptables的链
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system  # 生效

# 时间同步
yum install ntpdate -y
ntpdate time.windows.com
```

#### 3. 所有节点安装Docker/kubeadm/kubelet

Kubernetes默认CRI（容器运行时）为Docker，因此先安装Docker。

##### 3.1 安装Docker

```
$ wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
$ yum -y install docker-ce-18.06.1.ce-3.el7
$ systemctl enable docker && systemctl start docker
$ docker --version
Docker version 18.06.1-ce, build e68fc7a
```

```
$ cat > /etc/docker/daemon.json << EOF
{
  "registry-mirrors": ["https://b9pmyelo.mirror.aliyuncs.com"]
}
EOF
```

##### 3.2 添加阿里云YUM软件源

```
$ cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

##### 3.3 安装kubeadm，kubelet和kubectl

由于版本更新频繁，这里指定版本号部署：

```
$ yum install -y kubelet-1.18.0 kubeadm-1.18.0 kubectl-1.18.0
$ systemctl enable kubelet
```

#### 4. 部署Kubernetes Master

在192.168.31.61（Master）执行。

```
$ kubeadm init \
  --apiserver-advertise-address=192.168.44.146 \
  --image-repository registry.aliyuncs.com/google_containers \
  --kubernetes-version v1.18.0 \
  --service-cidr=10.96.0.0/12 \
  --pod-network-cidr=10.244.0.0/16
```

由于默认拉取镜像地址k8s.gcr.io国内无法访问，这里指定阿里云镜像仓库地址。

使用kubectl工具：

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
$ kubectl get nodes
```

#### 5. 加入Kubernetes Node

在192.168.1.12/13（Node）执行。

向集群添加新节点，执行在kubeadm init输出的kubeadm join命令：

```
$ kubeadm join 192.168.1.11:6443 --token esce21.q6hetwm8si29qxwn \
    --discovery-token-ca-cert-hash sha256:00603a05805807501d7181c3d60b478788408cfe6cedefedb1f97569708be9c5
```

默认token有效期为24小时，当过期之后，该token就不可用了。这时就需要重新创建token，操作如下：

```
kubeadm token create --print-join-command
```

#### 6. 部署CNI网络插件

```
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

默认镜像地址无法访问，sed命令修改为docker hub镜像仓库。

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

kubectl get pods -n kube-system
NAME                          READY   STATUS    RESTARTS   AGE
kube-flannel-ds-amd64-2pc95   1/1     Running   0          72s
```

#### 7. 测试kubernetes集群

在Kubernetes集群中创建一个pod，验证是否正常运行：

```
$ kubectl create deployment nginx --image=nginx
$ kubectl expose deployment nginx --port=80 --type=NodePort
$ kubectl get pod,svc
```

访问地址：http://NodeIP:Port  

### 二进制包























