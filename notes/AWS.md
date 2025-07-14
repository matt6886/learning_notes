# AWS

## 基础知识

* Linux基础
* 一种编程语言（Nojde.js，Python，Go，Java等）
* 网络基础知识（TCP/IP，HTTP/HTTPS，SSH/SCP，Network等）
* 数据库基础知识（RDB，SQL， NoSQL等）
* 容器基础知识（Docker，K8S）



## 课程内容

1. 组件规划网络

* VPC
* SubNet
* ACL
* SecurityGroup
* RouteTable

2. 设计网络应用

* Web
* ELB
* Database
* Auto Scaling
* MultiAZ
* MultiRegion

3. AWS服务

* VPC
* EC2
* S3
* IAM
* RDS
* DynamoDB
* Lambda
* Route 53
* API Gateway
* CloudTrail
* CloudWatch
* Elastic Beanstalk
* SQS
* ECR
* ECS

4. 开发流程

* CodeCommit
* CodeBuild
* CodeDeploy
* CodePipeline

5. 进阶应用

* Config
* System Manager
* CloudFormation
* Opsworks
* IoT
* 机器学习
* 工作流
* 移动开发(Amplify)
* CLI
* SDK(Node.js, Python, Go三种语言)

### 深度学习

deeplearnaws.com

### 课程目标

掌握以下系统架构能力

* 实现弹性架构
* 实现高性能架构
* 实现高可用架构
* 实现安全的架构
* 实现成本优化

### 系统架构师

设计系统结构需要考虑的问题

* 需要几台服务器
* 每个服务器的功能（web，cache，DB...）
* 需要什么硬件配置的服务器（CPU，MEM，HDD/SDD, ...）
* 服务器放在哪里（IDC，Cloud...）
* 服务器需要什么OS（Linuex，Window...）
* 服务器需要什么软件(Apache, Nginx, Tomcat, MySQL, PostgreSQL, ...)
* 网络构成是怎样的（客户端，中间层，服务层，数据层，分析层，AI层....）
* 网络IP如何规划（根据角色分配IP范围）
* 需要几个子网（对外公开，对内公开）
* 如何设计网络安全（服务访问授权，跟踪，数据加密，传输加密，WAF，DDos...）
* 如何与本地连接（公司内网和云端通信机制）
* 如何提高服务可用性（负载均衡，多区域部署）
* 如何节省成本（性价比）

![image-20250512161908088](/Users/matt/Desktop/Learning/learning_notes/notes/images/aws09.png)



## 申请AWS账号

### 准备

* 邮箱
* 手机号
* 信用卡

### 官网

https://aws.amazon.com/cn

### 实战

* 申请账号
* 设置CloudWatch
* 设置IAM管理员（No Root）
* 设置CloudTrail



## 设置CloudWatch-监控成本

监控我的服务费用。

### 实战演习

* 账单和成本管理控制面板
  * 账单首选项：开启CloudWatch账单提醒
* 利用CloudWatch监控费用增长
  * 使用CloudWatch设置账单警报
  
  1. 警报-->账单-->创建警报
  2. 创建警报主题：需要验证接收警报邮箱
  3. 创建警报，设置警报的阈值



## 设置IAM管理员-把权利关进笼子

避免使用超级管理员Root根账号，应当分配适当的有权限的账号进行工作管理。

### 区别用户

* Root根账号
  * 控制其他管理账号时
  * 账号契约解约时
  * 账号付款时
* 普通管理员
  * 管理一般用户(User, Group, Role)
  * 管理AWS资源(EC2, RDB...)
* 普通用户
  * AWS资源使用

### 实战演习

* IAM（用户与访问管理）
  * 简化用户登录URL
  
  设置用户账号ID的别名
  
  * 建立管理员组
  
  IAM -> 用户组 -> 创建组并设置该组的访问权限
  
  * 建立普通管理员
  
  IAM -> 用户 -> 设置用户名和密码   -> 选择用户组 -> 创建用户
  
  目前创建用户有两种方式Identify Center和之前IAM用户两种方式，目前使用的IAM方式创建的。现在AWS推荐使用Identify Center。后面研究下。
  
* 使用创建的IAM普通用户登录

1. 输入账号ID别名
2. 输入用户名和密码
3. 登录访问控制台



## 设置CloudTrail-你干啥我都知道

记录系统中用户或API的活动行为(Who, When, What...)

https://aws.amazon.com/cn/cloudtrail

### 注意点

* CloudTrail本体免费（真实的谎言），如果记录特殊的分析事件需要另行付费
* CloudTrail 默认免费记录90天操作日志
* 开启系统跟踪服务后，日志会写入S3存储，S3存储桶另行收费

### 实战演习

* 选择我们所在区
* 建立我们的系统跟踪

CloudTrail -> 右上角选择区域 -> 选择跟踪 -> 创建跟踪（设置跟踪的名字，存储位置（S3，如果没有创建S3，选择新建，会自动创建一个S3存储桶），是否对数据加密，标签，选择需要记录的事件）



## 理解区域和可用区的概念

> **一个“区域（Region）”是一个地理位置，例如“东京”或“美国东部”，每个区域内部包含多个相互独立的数据中心，这些数据中心称为“可用区（AZ）”。**

**区域（Region）是什么？**

- 区域是 **一个物理地理区域**
- 每个区域都是 AWS 在世界各地部署的独立物理基础设施
- 每个区域都有唯一的代码，比如：

| **区域名称**         | **地理位置**           | **Region Code** |
| -------------------- | ---------------------- | --------------- |
| 美国东部（弗吉尼亚） | USA                    | us-east-1       |
| 亚太地区（东京）     | 日本                   | ap-northeast-1  |
| 亚太地区（香港）     | 香港                   | ap-east-1       |
| 中国（北京）         | 中国（由光环新网运营） | cn-north-1      |

- 你选择区域时是在决定**“我把服务部署在哪个城市”**

**可用区（Availability Zone, AZ）是什么？**

- 每个区域内部划分出 **2~6 个可用区**

- 每个 AZ 是一个 **物理隔离的数据中心群**，通常由一个或多个真正的数据中心组成

- AZ 之间：**独立供电、网络和冷却系统**，但高速网络连接

  示例（东京 Region ap-northeast-1）中可能包含：

```txt
ap-northeast-1a
ap-northeast-1c
ap-northeast-1d
```

| **概念**       | **类比**                         |
| -------------- | -------------------------------- |
| Region         | 城市，例如“东京”                 |
| AZ             | 同一城市内的不同“数据中心园区”   |
| VPC            | 你在这个城市里的“私人网络园区”   |
| 子网（Subnet） | 你园区里划分出来的“街区”或“分区” |



## 设计一个Web应用的网络结构

![image-20250512165853104](/Users/matt/Desktop/Learning/learning_notes/notes/images/aws10.png)



## 网络IP范围设计

建立网络前需要设计IP网段和使用方法

### 实战演习

* 公开网络-主要用于对外公开的网络服务器，如Web，API服务器
  * 网段决定： 
    * 172.16.10.0/24
    * 172.16.11.0/24
* 私有网络-主要用于内部使用的服务器，如DB，Redis等
  * 网段决定：
    * 172.16.20.0/24
    * 172.16.21.0/24

### 扩展性考虑

考虑到系统整体可用性，今后需要将服务器部署到多个AZ区，所以应该提前做好IP地址的规划。





## 建立VPC-建立我们自己的IDC

建立VPC(Virtual Private Cloud)，可以把VPC理解为我们自己私有的IDC（互联网数据中心）

### 实战演习

* CIDR
  * 172.16.0.0/16
* 标签
  * Name: deeplearnaws-vpc

**操作步骤**

1. 搜索VPC打开VPC控制面板， 可以看到有个默认的VPC，这是amazon默认生成的，可以直接用。
2. 选择创建VPC：输入vpc名称，IPV4 CIDR块，无IPV6（收费的），创建vpc

> AWS 中的 **IPv4 CIDR 块** 是网络子网划分的基础，常用于设置 VPC（虚拟私有云）和子网的 IP 范围。
>
> CIDR 块（Classless Inter-Domain Routing）是一种表示 IP 地址范围的方式，在 AWS 中用来定义你 VPC 或子网中可用的 IP 地址范围。
>
> ```tex
> CIDR：10.0.0.0/24
> ```
>
> - 表示的 IP 范围是：10.0.0.0 ~ 10.0.0.255
> - 可分配 IP 数量：**256 个**（实际可用约 251 个）
> - /24：前 24 位是网络地址，后 8 位用于分配主机地址
>
> 在 AWS 中使用 CIDR 的地方：
>
> | **场景**                        | **用法说明**                                                 |
> | ------------------------------- | ------------------------------------------------------------ |
> | **创建 VPC**                    | 你需要提供一个 CIDR 块，比如 10.0.0.0/16，作为整个私有网络的 IP 范围 |
> | **创建子网 Subnet**             | 从 VPC 的 CIDR 中划分出更小的块，比如 10.0.1.0/24，每个子网的设备使用它的 IP 地址 |
> | **安全组 / NACL / 路由表**      | 配置访问规则时，可以使用 CIDR 来指定来源或目标（比如允许 0.0.0.0/0 表示所有 IPv4 地址） |
> | **VPN/Peering/Transit Gateway** | 配置多个 VPC 的互通时，需要确保各个 CIDR 块 **不重叠**       |





## 建立公私网-公私分明才能网络安全

### 知识点

* 建立公有网段
* 建立私有网段

### 实战演习

* 子网所属VPC-deeplearnaws-vpc

* 公开网络

主要用于对外公开的服务器，如Web，API服务器

网段决定：

* 172.16.10.0/24
* ap-northeast-1a

标签：deeplearnaws-web-1a

* 私有网络

主要用于内部使用的服务器，如DB，Redis等

网段决定：

* 172.16.20.0/24
* ap-northeast-1a

标签：deeplearnaws-db-1a

**操作步骤**

1. 搜索vpc，打开vpc控制面板
2. 选择子网，默认会有3个子网。
3. 点击创建子网，选择我们之前创建的vpc，输入子网的名称和子网的CIDR块地址并选择可用区ap-northeast-1a，创建子网



## 建立互联网网关

我们上一步所创建的子网，其实都是私有网络，要使其能够访问互联网，需要设置对应的网关，可以理解为拉一条网线连接到我们的私有网络。这样其才能够访问互联网。

**操作步骤**

1. 搜索vpc，进入vpc控制台
2. 点击互联网网关，有一个默认的互联网网关。
3. 点击创建互联网网关，输入名称deeplearnaws-igw，点击创建
4. 创建的互联网网关默认是detached状态，还没有绑定到哪个vpc上。勾选上，然后点击附加到vpc上，选择我们之前创建的vpc，然后点击连接互联网绑定。



## 建立公网路由表-让我们的子网连接到互联网

### 知识点

* 理解路由表
* 建立路由表

### 建立路由表

+ RTB：deeplearnaws-web-rtb
  + VPC: deeplearnaws-vpc
  + Rule: 加入deeplearn-igw
+ 把新路由绑定到deeplearnaws-web-1a子网

![image-20250512193657189](/Users/matt/Desktop/Learning/learning_notes/notes/images/aws11.png)

**操作步骤**

1. 搜索vpc并打开vpc控制台
2. 点击路由表，默认会有两个路由表
3. 点击创建路由表，输入路由名称，选择vpc，输入标签名称，创建路由
4. 点击刚创建的路由表，切换到路由选项，可以发现只有本地的路由。点击编辑路由，添加路由，输入0.0.0.0/0，并且选择互联网网关中我们之前创建的网关，保存更改。这样除了我们私有网络外，所有外网的访问，都会走互联网网关。

5. 将创建的路由表绑定到我们的子网，选择路由表，切换到子网关联，编辑子网关联，选择我们之前创建的子网deeplearnaws-web-1a，保存关联。接下来我们在该子网创建任何的EC2，这个服务器就可以连接到internet上了。

> 1.**核心概念快速解释**
>
> | **名词**                                | **通俗解释**       | **功能**                                           |
> | --------------------------------------- | ------------------ | -------------------------------------------------- |
> | **VPC（Virtual Private Cloud）**        | 你的“专属网络”     | 所有网络资源（EC2、RDS 等）运行的环境              |
> | **子网（Subnet）**                      | VPC 中的“分区”     | 可以理解为一个房间/小网段，用来放置 EC2 实例等资源 |
> | **路由表（Route Table）**               | 子网的“导航地图”   | 决定数据如何从子网内流向其他子网或外部网络         |
> | **路由（Route）**                       | 路由表中的一条规则 | 指定某个目标 IP 范围该怎么走                       |
> | **互联网网关（Internet Gateway, IGW）** | 出口路由器         | 让子网中的资源可以访问公网（Internet）             |
> | **NAT 网关（NAT Gateway）**             | 内网访问公网的代理 | 让**私有子网**中的实例能访问外网，但不被外网访问   |
> | **目标（Destination）**                 | 你想去的 IP 段     | 比如 0.0.0.0/0 代表“所有 IP”                       |
> | **下一跳（Target）**                    | 指定如何去         | 比如 igw-xxxx 表示走互联网网关                     |
>
> 2.**简单类比**
>
> 假设：
>
> - VPC 是你在 AWS 上建的“园区”
> - 子网是你园区里的“各个房间”
> - 路由表是你房间里张贴的“导航路线图”
> - 路由是一条“去哪儿走哪条路”的指令
> - IGW 是“你家通向互联网的光猫”
> - NAT 网关是“只出不进的上网代理”
>
> 3.**详细说明各个组件**
>
> * **VPC（虚拟私有云）**
>
>   * 创建 AWS 网络的起点
>   * 你定义它的 IP 范围（CIDR 块，如 10.0.0.0/16）
>
> * **子网（Subnet）**
>
>   * 是 VPC 中的小块 IP 地址范围
>   * 可以是 **公有子网**（可访问公网）或 **私有子网**（不能访问公网）
>   * AWS 要求你将 EC2 实例等资源**部署到某个子网**
>
> * **路由表（Route Table）**
>
>   * 每个子网都关联一个路由表
>   * 决定这个子网内的资源流量该怎么出去
>   * 每张表中有一堆路由规则（比如去哪走 IGW、去哪走 NAT）
>
> * **路由（Route）**
>
>   * 一条路由的结构是：
>
>   ```tex
>   目的地（Destination）   ->   下一跳（Target）
>   0.0.0.0/0              ->   igw-xxxxxxx （走互联网）
>   10.0.1.0/24            ->   local       （走内部网）
>   ```
>
>   * 0.0.0.0/0 表示访问“任意公网 IP”
>
> * **互联网网关（Internet Gateway，IGW）**
>
>   * 是 VPC 与 Internet 之间的通道
>   * 绑定到 VPC
>   * 子网要想连外网，需要：
>     * 有 IGW
>     * 路由表中指向 IGW 的路由
>     * 安全组/NACL 放行 80/443 等端口
>
> * **如何让一个 EC2 可以上网？**
>
> | **步骤**         | **操作**                          |
> | ---------------- | --------------------------------- |
> | 1️⃣ 创建 VPC       | CIDR 如 10.0.0.0/16               |
> | 2️⃣ 创建子网       | 公有子网 10.0.1.0/24              |
> | 3️⃣ 创建 IGW       | 并附加到该 VPC                    |
> | 4️⃣ 配置路由表     | 添加 0.0.0.0/0 -> igw-xxxx 的路由 |
> | 5️⃣ 子网绑定路由表 | 将该路由表绑定到 10.0.1.0/24 子网 |
> | 6️⃣ 部署 EC2       | 放在这个子网中                    |
> | 7️⃣ 安全组设置     | 开放入站/出站 80、443 端口        |



## 两个防火墙-网络ACL和安全组SG

### 知识点

* 网络存取控制列表 — ACL
* 安全组 — Security Group

### 实战演习

1. 确认ACL和SG的基本概念 - deeplearnaws-vpc
2. 建立deeplearnaws的WEB安全组

在建立实际的EC2之前，应该设置一下防火墙，否则这台服务器的所有端口都是对外公开的。

在建立VPC的时候，aws会自动建立一套默认的ACL和SG，默认全部端口都是打开的。

![image-20250512194514495](/Users/matt/Desktop/Learning/learning_notes/notes/images/aws12.png)

![image-20250512194629898](/Users/matt/Desktop/Learning/learning_notes/notes/images/aws13.png)

> - Security Group（SG）是“虚拟防火墙”**，控制**实例级别（EC2）**的访问权限
> - Network ACL（ACL）是“子网层的访问控制”**，控制**整个子网**的入站和出站流量
>
> | **概念**       | **类比**                                             |
> | -------------- | ---------------------------------------------------- |
> | Security Group | EC2 实例自己的门卫，只允许通过身份核验的访客         |
> | Network ACL    | 小区大门的保安，决定哪些 IP 可以进出整个小区（子网） |
>
> **Security Group（SG）详解：**
>
> - 绑定到 EC2、RDS 等具体资源
> - 控制哪些 IP/端口/协议可以进出这个资源
> - 主要用于实例级别的网络安全控制
>
> **Network ACL（ACL）详解：**
>
> - 绑定到 **子网（Subnet）**
> - 控制这个子网中所有资源的网络访问权限
>
> 总结:
>
> - **SG 更常用、更易用**，你大多数时候只用 SG 就够了
> - **ACL 更底层，适合高级网络隔离、黑名单防护**
> - 如果 SG 和 ACL 冲突（一个允许，一个拒绝），**最终是被 ACL 拒绝**



**操作步骤**

1. 搜索vpc，打开vpc控制台
2. 点击左侧的网络ACL，可以看到默认生成了两个ACL，其中一个是我们创建vpc时默认生成的，默认其出站和入站规则是允许所有IP的访问，是不安全的。
3. 点击左侧的安全组，可以看到有两个默认的安全组，其中一个是我们创建vpc对应的安全组，其默认是允许所有的ip访问，所有的端口都是开放的。

4. 点击右上角的创建安全组，输入安全组的名称，描述，选择之前创建的vpc，设置入站规则（例如选择SSH，允许访问的ip地址），添加标签，保存。



## 建立EC2实例-开启我们的云端服务器之旅

### 知识点

建立一个公网可访问的EC2服务器实例。

### 实战演习

1. EC2实例的具体设置

* Amazon Linux 2023 AMI（带有很多自带的软件，不需要手动安装）
* t3.micro(新用户免费，可以使用12个月)
* deepleanaws-vpc(vpc)
* deeplearnaws-web-1a(子网)
* 自动分配公有IP
* 内网IP： 172.16.10.10/32
* Name: deeplearnaws-web1
* 密钥对：deeplearnaws-ssh-key

2. SSH连接

```bash
cp ~/Download/deeplearnaws-ssh-key.pem ./
chmod 400 deeplearnaws-ssh-key.pem
# Amazon Linux 2023
ssh -i ./deeplearnaws-ssh-key ec2-user@ip
```

**操作步骤**

1. 搜索ec2，然后打开ec2控制台
2. 点击实例，可以看到目前还没有任何ec2实例
3. 点击启动实例，输入名称deeplearnaws-web1，选择操作系统映像Amazon Linux（AMI 是一种模板，其中包含了启动实例所需的软件配置（操作系统、应用程序服务器和应用程序）。），选择实例t3.micro（免费套餐），创建密钥对（输入密钥对名称，选择密钥对类型，秘钥文件格式），之后会下载该秘钥到本地，网络设置，选择我们之前创建的vpc和子网，启动自动分配公有ip，选择现有安全组，给网卡分配一个ip地址，点击高级网络配置，设置主要ip为172.16.10.10，配置存储，8g，通用性SSD（gp3），点击启动实例。
4. 拷贝秘钥到本地并连接到服务器

```shell
cp ~/Download/deeplearnaws-ssh-key.pem ./
chmod 400 deeplearnaws-ssh-key.pem
# Amazon Linux 2
ssh -i ./deeplearnaws-ssh-key ec2-user@ip
```

> 记录问题：
>
> 第一次连接服务器时报错，一直超时，然后重启服务器后，成功了。中间执行了以下操作：
>
> ```bash
> mkdir ~/.ssh
> chmod 700 ~/.ssh
> touch ~/.ssh/known_hosts
> ```

> 使用 chmod 400 修改私钥文件权限，是为了确保 SSH 客户端**允许你安全地使用私钥连接服务器**，否则连接时会报错。
>
> 当你用如下命令连接 EC2 时：
>
> ```bash
> ssh -i your-key.pem ec2-user@<EC2-IP>
> ```
>
> SSH 会检查 your-key.pem 文件的权限。
>
> 如果该文件是：
>
> - **可被其他用户读取**
> - **有写/执行权限**
>
> 它会直接报错：
>
> ```bash
> Permissions 0644 for 'your-key.pem' are too open.
> It is required that your private key files are NOT accessible by others.
> This private key will be ignored.
> ```



## 建立AMI - 把自己的系统克隆

### 在Amazon Linux 2023上安装必要的软件包

+ Docker
+ Node.js
+ Git

```bash
# Amazon Linux 2023
ssh -i deeplearnaws-ssh-key ec2-user@ip
# 安全登录信息
touch .hushlogin
# 系统包升级-更新系统中所有已安装的软件包到最新版本
sudo yum update -y
# 确认Linux系统版本
cat /etc/os-release
# Docker
# 使用amazon-linux 扩展包安装docker
# 在 Amazon Linux 2023 或更高版本中，amazon-linux-extras这个命令被废弃了。
# amazon-linux-extras 是 Amazon Linux 2 中的一个工具，用来：
# sudo amazon-linux-extras list
# sudo amazon-linux-extras install -y docker

# Amazon Linux 2023 使用标准的 dnf（不是 yum） 和模块化仓库，不再依赖 amazon-linux-extras
sudo dnf install docker -y
# 修改 docker 附加给组 ec2-user
# 让 ec2-user 用户可以在不使用 sudo 的情况下运行 Docker 命令。
# 这里需要注意，使用ec2-user登录后，直接使用这个命令，然后exit重新链接ec2实例，即可
sudo usermod -a -G docker ec2-user
docker version
# 启动 docker 服务
sudo systemctrl start docker
# 确认 docker 运行
ps aux | grep docker
# 注册 docker 服务
# systemctrl 是 Linux 中用于管理系统服务的工具， enable 表示将服务注册为开机自动启动
sudo systemctrl enable docker

# 这里好像拷贝错位置了/usr/bin 写成了/usr/local/bin
sudo curl -L "https://github.com/docker/compose/releases/download/v2.27.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

# 安装Node.js
curl -sL https://rpm.nodesource.com/setup_20.x | sudo bash -
sudo yum install -y gcc-c++ make
sudo yum install -y nodejs
node -v

# 安装git
sudo yum install git -y
git version

# 可选安装
sudo yum install nmap -y

# nmap 是干什么用的？
# 查看某个子网内有哪些设备在线
# 扫描服务器开放了哪些端口（如 22, 80, 443）
# 判断目标是否开启防火墙或屏蔽了端口
# 识别某个端口运行的服务，如 Apache、SSH 等
nmap 192.168.1.10 #扫描目标主机的开放端口：
nmap 192.168.1.0/24 #扫描一整段 IP：
nmap -p 80,443 yourdomain.com #检测远程服务器是否开放了 Web 服务：
```

> `sudo yum update -y`
>
> ```bash
> Amazon Linux 2023 Kernel Livepatch repository  176 kB/s | 17 kB   00:00   
> 
> Dependencies resolved.
> 
> Nothing to do.
> 
> Complete!
> ```
>
> `cat /etc/os-release`
>
> ```bash
> NAME="Amazon Linux"
> VERSION="2023"
> ID="amzn"
> ID_LIKE="fedora"
> VERSION_ID="2023"
> PLATFORM_ID="platform:al2023"
> PRETTY_NAME="Amazon Linux 2023.8.20250707"
> ANSI_COLOR="0;33"
> CPE_NAME="cpe:2.3:o:amazon:amazon_linux:2023"
> HOME_URL="https://aws.amazon.com/linux/amazon-linux-2023/"
> DOCUMENTATION_URL="https://docs.aws.amazon.com/linux/"
> SUPPORT_URL="https://aws.amazon.com/premiumsupport/"
> BUG_REPORT_URL="https://github.com/amazonlinux/amazon-linux-2023"
> ```
>
> `sudo dnf install -y docker`
>
> > Last metadata expiration check: 0:08:57 ago on Sun Jul 13 13:19:40 2025.
> >
> > Dependencies resolved.
> >
> > ================================================================================
> >
> >  Package         Arch   Version         Repository   Size
> >
> > ================================================================================
> >
> > Installing:
> >
> >  **docker**          x86_64  25.0.8-1.amzn2023.0.5  amazonlinux  46 M
> >
> > Installing dependencies:
> >
> >  **container-selinux**    noarch  3:2.233.0-1.amzn2023   amazonlinux  55 k
> >
> >  **containerd**        x86_64  2.0.5-1.amzn2023.0.2   amazonlinux  26 M
> >
> >  **iptables-libs**      x86_64  1.8.8-3.amzn2023.0.2   amazonlinux  401 k
> >
> >  **iptables-nft**       x86_64  1.8.8-3.amzn2023.0.2   amazonlinux  183 k
> >
> >  **libcgroup**        x86_64  3.0-1.amzn2023.0.1    amazonlinux  75 k
> >
> >  **libnetfilter_conntrack**  x86_64  1.0.8-2.amzn2023.0.2   amazonlinux  58 k
> >
> >  **libnfnetlink**       x86_64  1.0.1-19.amzn2023.0.2  amazonlinux  30 k
> >
> >  **libnftnl**         x86_64  1.2.2-2.amzn2023.0.2   amazonlinux  84 k
> >
> >  **pigz**           x86_64  2.5-1.amzn2023.0.3    amazonlinux  83 k
> >
> >  **runc**           x86_64  1.2.6-1.amzn2023.0.1   amazonlinux  3.7 M
> >
> > 
> >
> > Transaction Summary
> >
> > ================================================================================
> >
> > Install 11 Packages
> >
> > 
> >
> > Total download size: 77 M
> >
> > Installed size: 292 M
> >
> > Downloading Packages:
> >
> > (1/11): container-selinux-2.233.0-1.amzn2023.no 1.3 MB/s | 55 kB   00:00   
> >
> > (2/11): iptables-libs-1.8.8-3.amzn2023.0.2.x86_ 13 MB/s | 401 kB   00:00   
> >
> > (3/11): iptables-nft-1.8.8-3.amzn2023.0.2.x86_6 5.9 MB/s | 183 kB   00:00   
> >
> > (4/11): libcgroup-3.0-1.amzn2023.0.1.x86_64.rpm 2.7 MB/s | 75 kB   00:00   
> >
> > (5/11): libnetfilter_conntrack-1.0.8-2.amzn2023 3.0 MB/s | 58 kB   00:00   
> >
> > (6/11): libnfnetlink-1.0.1-19.amzn2023.0.2.x86_ 1.6 MB/s | 30 kB   00:00   
> >
> > (7/11): libnftnl-1.2.2-2.amzn2023.0.2.x86_64.rp 2.8 MB/s | 84 kB   00:00   
> >
> > (8/11): pigz-2.5-1.amzn2023.0.3.x86_64.rpm   3.1 MB/s | 83 kB   00:00   
> >
> > (9/11): runc-1.2.6-1.amzn2023.0.1.x86_64.rpm   63 MB/s | 3.7 MB   00:00   
> >
> > (10/11): containerd-2.0.5-1.amzn2023.0.2.x86_64 63 MB/s | 26 MB   00:00   
> >
> > (11/11): docker-25.0.8-1.amzn2023.0.5.x86_64.rp 67 MB/s | 46 MB   00:00   
> >
> > \--------------------------------------------------------------------------------
> >
> > Total                      107 MB/s | 77 MB   00:00   
> >
> > Running transaction check
> >
> > Transaction check succeeded.
> >
> > Running transaction test
> >
> > Transaction test succeeded.
> >
> > Running transaction
> >
> >  Preparing    :                            1/1 
> >
> >  Installing    : runc-1.2.6-1.amzn2023.0.1.x86_64           1/11 
> >
> >  Installing    : containerd-2.0.5-1.amzn2023.0.2.x86_64        2/11 
> >
> >  Running scriptlet: containerd-2.0.5-1.amzn2023.0.2.x86_64        2/11 
> >
> >  Installing    : pigz-2.5-1.amzn2023.0.3.x86_64            3/11 
> >
> >  Installing    : libnftnl-1.2.2-2.amzn2023.0.2.x86_64         4/11 
> >
> >  Installing    : libnfnetlink-1.0.1-19.amzn2023.0.2.x86_64       5/11 
> >
> >  Installing    : libnetfilter_conntrack-1.0.8-2.amzn2023.0.2.x86_64  6/11 
> >
> >  Installing    : iptables-libs-1.8.8-3.amzn2023.0.2.x86_64       7/11 
> >
> >  Installing    : iptables-nft-1.8.8-3.amzn2023.0.2.x86_64       8/11 
> >
> >  Running scriptlet: iptables-nft-1.8.8-3.amzn2023.0.2.x86_64       8/11 
> >
> >  Installing    : libcgroup-3.0-1.amzn2023.0.1.x86_64          9/11 
> >
> >  Running scriptlet: container-selinux-3:2.233.0-1.amzn2023.noarch    10/11 
> >
> >  Installing    : container-selinux-3:2.233.0-1.amzn2023.noarch    10/11 
> >
> >  Running scriptlet: container-selinux-3:2.233.0-1.amzn2023.noarch    10/11 
> >
> >  Running scriptlet: docker-25.0.8-1.amzn2023.0.5.x86_64         11/11 
> >
> >  Installing    : docker-25.0.8-1.amzn2023.0.5.x86_64         11/11 
> >
> >  Running scriptlet: docker-25.0.8-1.amzn2023.0.5.x86_64         11/11 
> >
> > Created symlink /etc/systemd/system/sockets.target.wants/docker.socket → /usr/lib/systemd/system/docker.socket.
> >
> > 
> >
> >  Running scriptlet: container-selinux-3:2.233.0-1.amzn2023.noarch    11/11 
> >
> >  Running scriptlet: docker-25.0.8-1.amzn2023.0.5.x86_64         11/11 
> >
> >  Verifying    : container-selinux-3:2.233.0-1.amzn2023.noarch     1/11 
> >
> >  Verifying    : containerd-2.0.5-1.amzn2023.0.2.x86_64        2/11 
> >
> >  Verifying    : docker-25.0.8-1.amzn2023.0.5.x86_64          3/11 
> >
> >  Verifying    : iptables-libs-1.8.8-3.amzn2023.0.2.x86_64       4/11 
> >
> >  Verifying    : iptables-nft-1.8.8-3.amzn2023.0.2.x86_64       5/11 
> >
> >  Verifying    : libcgroup-3.0-1.amzn2023.0.1.x86_64          6/11 
> >
> >  Verifying    : libnetfilter_conntrack-1.0.8-2.amzn2023.0.2.x86_64  7/11 
> >
> >  Verifying    : libnfnetlink-1.0.1-19.amzn2023.0.2.x86_64       8/11 
> >
> >  Verifying    : libnftnl-1.2.2-2.amzn2023.0.2.x86_64         9/11 
> >
> >  Verifying    : pigz-2.5-1.amzn2023.0.3.x86_64            10/11 
> >
> >  Verifying    : runc-1.2.6-1.amzn2023.0.1.x86_64           11/11 
> >
> > 
> >
> > Installed:
> >
> >  container-selinux-3:2.233.0-1.amzn2023.noarch                 
> >
> >  containerd-2.0.5-1.amzn2023.0.2.x86_64                     
> >
> >  docker-25.0.8-1.amzn2023.0.5.x86_64                      
> >
> >  iptables-libs-1.8.8-3.amzn2023.0.2.x86_64                   
> >
> >  iptables-nft-1.8.8-3.amzn2023.0.2.x86_64                    
> >
> >  libcgroup-3.0-1.amzn2023.0.1.x86_64                      
> >
> >  libnetfilter_conntrack-1.0.8-2.amzn2023.0.2.x86_64               
> >
> >  libnfnetlink-1.0.1-19.amzn2023.0.2.x86_64                   
> >
> >  libnftnl-1.2.2-2.amzn2023.0.2.x86_64                      
> >
> >  pigz-2.5-1.amzn2023.0.3.x86_64                         
> >
> >  runc-1.2.6-1.amzn2023.0.1.x86_64                        
> >
> > 
> >
> > Complete!
>
> `docker version`
>
> ```bash
> Client:
>  Version:           25.0.8
>  API version:       1.44
>  Go version:        go1.24.4
>  Git commit:        0bab007
>  Built:             Wed Jun 18 00:00:00 2025
>  OS/Arch:           linux/amd64
>  Context:           default
> ```
>
> `ps aux | grep docker`
>
> ```bash
> ec2-user   32460  0.0  0.2 222316  2104 pts/1    S+   13:42   0:00 grep --color=auto docker
> ```
>
> `sudo systemctl enable docker`
>
> ```bash
> Created symlink /etc/systemd/system/multi-user.target.wants/docker.service → /usr/lib/systemd/system/docker.service.
> ```
>
> `sudo curl -L "https://github.com/docker/compose/releases/download/v2.27.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`
>
> ```bash
> % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
>                                  Dload  Upload   Total   Spent    Left  Speed
>   0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
> 100 60.1M  100 60.1M    0     0  37.3M      0  0:00:01  0:00:01 --:--:--  125M
> ```
>
> `sudo chmod +x /usr/local/bin/docker-compose`
>
> `docker-compose version`
>
> ```bash
> Docker Compose version v2.27.1
> ```



### 打包AMI

一般情况下，选择停掉ec2实例，然后再打包ami，当然如果你的服务器很多用户再用，也可以不停服务直接打包AMI。

**操作步骤**

1. 停止ec2实例
2. 选择ec2实例，点击操作中的映像与模版，创建映像，输入映像名称，实例卷大小，创建映像
3. 确认创建的AMI状态为可用之后，删除之前的EC2实例
4. 使用AMI启动新的实例。

这样，我们就可以使用AMI启动多台服务器，而不需要去重复安装服务器的配置。



> **Docker基础知识**
>
> **Docker 是一个开源的容器化平台**，用于打包、分发和运行应用程序，使它们在任何地方都能以相同方式运行。
>
> 通俗理解：
>
> 你可以把 Docker 想象成：
>
> **“轻量级的虚拟机”**
>
> 但比传统虚拟机更快速、更高效。
>
> 它让你可以：
>
> - 把应用程序和它需要的环境（代码、依赖、系统库）**打包在一起**
> - 打包成一个 **容器（Container）**
> - 然后在任何机器上“跑起来”，而不管这台机器上装没装 Node.js、Python、MySQL……
>
> 举个例子：
>
> 你开发了一个 Node.js 应用，要部署到服务器：
>
> - 传统做法：你需要在服务器上手动装 Node.js、配置环境、复制代码…
> - Docker 做法：你写一个 Dockerfile，用 Docker 把整个应用打包成容器
> - 然后只要一行命令，就能在任何服务器上运行：
>
> ```bash
> docker run your-app
> ```
>
> 核心概念：
>
> | **概念**              | **说明**                                           |
> | --------------------- | -------------------------------------------------- |
> | **镜像（Image）**     | 应用及其环境的“模板”，相当于“蓝图”                 |
> | **容器（Container）** | 用镜像启动的“运行中的实例”，轻量、高效             |
> | **Dockerfile**        | 定义如何构建镜像的脚本文件                         |
> | **Docker Hub**        | 官方的公共镜像仓库，你可以从这里下载别人做好的镜像 |
>
> 总结：
>
> **Docker 就是一个打包应用+运行环境+依赖的“万能盒子”，可以让你的应用在任何地方都跑得一致，快速、高效、干净。**



## 部署一个Web应用-建立Node.js + Docker的应用程序

### 知识点

* 部署一个Node.js的Web应用
* 把应用通过Docker(Hub)方式进行部署
* 修改安全组，追加开发80端口

### 今后预定

部署到 ECR + ECS

![image-20250512201048889](/Users/matt/Desktop/Learning/learning_notes/notes/images/aws14.png)

**操作步骤**

1. 搜素EC2然后打开EC2控制台
2. 点击安全组，修改安全组的入站规则，添加80端口。这样只要隶属于这个安全组的所有ec2实例都可以访问80端口了。
3. 应用通过docker hub方式部署

```bash
# 从DockerHub上获取deeplearnaws-web镜像
docker pull komavideo/deeplearnaws-web:latest

docker run --name deeplearnaws-web -p 80:3000 -d komavideo/deeplearnaws-web:latest

nmap 127.0.0.1
docker container ls -a
docker logs -f deeplearnaws-web
docker container stop deeplearnaws-web
docker container rm deeplearnaws-web
docker ps

# 编辑调整
docker exec -it deeplearnaws-web sh #在名为 deeplearnaws-web 的正在运行的容器中，打开一个交互式的终端（shell）。
vi app.js # 修改app.js代码 保存并退出
exit # 交互式的终端（shell）
docker container restart deeplearnaws-web # 重启docker服务
```





## 部署一个DB应用

### 知识点

* 在公开网络中建立NAT网关
* 建立DB私有网络的路由表，设置NAT网关
* 建立DB私有网络的安全组，开放3306-MySQL端口

![image-20250512201548112](/Users/matt/Desktop/Learning/learning_notes/notes/images/aws15.png)

### 实战演习

* NAT网关
  * Name：deeplearnaws-web-nat
  * SubNet：deeplearnaws-web-1a
  * Elastic IP

* 路由表
  * Name：deeplearnaws-db-rtb
  * Target：deeplearnaws-web-nat -> 0.0.0.0/0
* 安全组
  * Name：deeplearn-db-sg
  * Port：3306



> **什么是 AWS 的 NAT 网关（****NAT Gateway**）？**
>
> **NAT（Network Address Translation）网关** 是 AWS 提供的一个**托管网络服务**，用于让 **私有子网（Private Subnet）中的实例访问公网**，**但不允许公网主动访问它们**。
>
> 你可以把 NAT 网关理解为：
>
> 一个 **只出不进** 的网络代理。
>
> 比如你有一台 EC2 实例放在私有子网里（没有公网 IP），但它还想访问外部网络（比如下载依赖、更新系统），这时候就需要 NAT 网关来帮它“出网”。
>
> **NAT 网关的主要用途**
>
> | **用途**                               | **说明**                                  |
> | -------------------------------------- | ----------------------------------------- |
> | 🔐 私有子网中的实例访问外部（如互联网） | 比如运行 yum update、访问 API、下载依赖等 |
> | 🛡️ 保证外部无法直接访问这些实例         | 增强安全性，公网进不来                    |
> | ☁️ 与 S3、DynamoDB 等 AWS 服务通信      | 支持 VPC Endpoint 替代方案                |
> | ⚙️ 自动扩展、高可用                     | AWS 托管，弹性可用，不需要你自己维护      |
>
> 总结：
>
> **AWS 的 NAT 网关** 是让私有子网内的资源可以安全访问公网的工具，而不会暴露在公网中，非常适合生产环境中保护 EC2 实例或容器



**操作步骤**

1. 创建NAT网关

* 打开vpc控制台，点击NAT网关，点击创建NAT网关
* 输入名称，选择公网子网，分配弹性IP，连接类型选择公有，创建NAT网关

2. 创建私有子网的路由表

* 点击路由表，创建路由表，输入名称，选择vpc，创建路由表
* 点击创建的路由，选择路由选项，点击编辑路由，添加路由，选择我们刚创建的NAT网关，保存更改。
* 点击子网关联，编辑子网关联，选择我们的数据库子网IP，保存关联

3. 创建安全组，让我们的公网IP可以访问私有数据库

* 点击安全组，创建安全组，
* 输入名称，选择vpc，设置入站规则，选择MYSQL/Aurora，输入我们的公网IP 172.16.10.0/24，创建



## 建立私有网段的服务器

### 知识点

* 修改私有安全组，增加port:22支持
* 在私有网段建立EC2实例
* 通过公有网段 web-ec2 实例连接到 db-ec2实例



### 操作步骤

1. 修改私有网段安全组，增加port:22 支持

* 登录vpc控制台，选择安全组，
* 切换到之前创建的私有安全组，选择入站规则，编辑入站规则，选择SSH，输入IP：172.16.10.0/24，保存

2. 建立私有网段ec2实例

* 登录vpc控制台，选择AMI，选择我们之前创建的AMI实例，点击从AMI启动实例
* 输入名称deeplearnaws-db1，选择密钥对，使用之前创建的密钥对，网络设置：选择vpc和我们的数据库私有子网，禁用自动分配IP，选择现有安全组，选择我们之前创建的私有安全组，网络接口：主要ip设置为172.16.20.10，配置存储：选择8g
* 启动实例

3. 通过公有网段 web-ec2 实例连接到 db-ec2 实例

```bash
# 拷贝本地的密钥到公网服务器，其实一般不会这么操作，而是将服务器的 公钥（~/.ssh/id_rsa.pub） 添加到 AWS EC2 的 authorized_keys，而不是把私钥 .pem 拷贝过去。
scp -i deeplearnaws-ssh-key.pem  deeplearnaws-ssh-key.pem ec2-user@ip
chmod 400 ~/deeplearnaws-ssh-key.pem

# 连接公网服务器
ssh -i deeplearnaws-ssh-key.pem ec2-user@ip

# 连接到私有服务器
ssh -i deeplearnaws-ssh-key.pem 172.16.20.10

# 验证私有服务器是否可以访问外网
sudo yum updte -y
curl https://komavideo.com
```



## 部署一个DB应用 - MySQL

### 知识点

* 在私有网段实例中安装 MySQL@Docker
* 在公有网段中去连接 MySQL 数据库实例

### 操作步骤

```bash
# 首先通过公网服务器连接到私有数据库服务器
# 安装MySQL数据库服务，通过docker方式安装
docker pull komavideo/deeplearnaws-mysql:latest

docker run --name deeplearnaws-mysql \
  -e MYSQL_ROOT_PASSWORD=12345678 \
  -p 3306:3306 \
  --restart=always \
  -d komavideo/deeplearnaws-mysql:latest
 
 docker exec -it deeplearnaws-mysql bash -p
 
 mysql -u root -p -h 127.0.0.1 #密码12345678
 show databases;
 use blgodb;
 select * from user;
 
 exit;
 
 # 通过web1服务器连接我们刚开启的服务器
 # 连接数据库
 mkdir myapp
 cd myapp
 npm init -y
 npm install mysql --save
 nano main.js
 node main.js
```



## 通过Web连接到MySQL

### 知识点

* 更新 web 应用 docker 版本
* 确认连接到数据库实例

### 操作步骤

```bash
# 这些之前已经操作过
docker pull komavideo/deeplearnaws-web:latest

docker run --name deeplearnaws-web -p 80:3000 -d komavideo/deeplearnaws-web:latest

nmap 127.0.0.1
```

然后将根据我们当前的web服务器，创建一个新的AMI映像。deeplearnaws-web-ami



## NAT 用完就删

NAT网关收费比较贵，用完就删。删除NAT网关后，其绑定的弹性IP会一起删除。

> .**当你删除 NAT 网关时：**
>
> 有两种情况：
>
> | **情况**                                      | **弹性 IP 的状态**                                        |
> | --------------------------------------------- | --------------------------------------------------------- |
> | 🔁 **EIP 是 AWS 自动分配的（你没有手动创建）** | 删除 NAT 网关时 EIP 会一并释放 ✅（你描述的情况）          |
> | 🛠️ **EIP 是你自己事先分配的并绑定的**          | 这个 EIP **会被解除绑定但不会自动释放**（你需要手动释放） |



## 负载均衡 - 通过ELB设计高可用性服务

### 知识点

* 设计高可用性网络结构

### 实战演习

* 建立一个新的可用区子网 - deeplearnaws-web-1c
* 在新的子网区添加web-ec2实例
* 通过ELB进行多区多实例之间的负载均衡

![image-20250512203532881](/Users/matt/Desktop/Learning/learning_notes/notes/images/aws16.png)

### 操作步骤

1. 建立新的可用区的子网 - deeplearnaws-web-1c

* 打开vpc控制台，点击创建子网
* 输入子网名称，选择vpc，可用区并输入子网的CIDR地址：172.16.11.0/24，创建
* 选择刚创建的子网，选择路由表，编辑路右边，选择我们之前创建的web路由表，这样其就可以访问外网了。

2. 在新的可用区子网添加 web-ec2 实例 - deeplearnaws-web2

* 打开ec2控制台，选择我们之前创建的AMI来开启新的ec2实例
* 输入服务器名称，deeplearnaws-web2，密钥对选择我们之前创建的，选择vpc和上步骤创建的子网，启用自动分配公网ip，选择之前创建的web安全组，网络接口主要IP：172.16.11.10，配置存储8g，启动实例。









## 高可用性设计 - 建立ALB负载均衡

### 知识点

* 建立目标组（把要负载的对象放到一个组里面，例如web1和web2）
* 建立ALB负载均衡器

### 实战演习

* 启动web服务器
* 建立目标组
* 建立ALB负载均衡器



## Route53 - 指引前进的方法

### 知识点

* 使用Route53的DNS解析服务

### Route53的特点

* 高度可用且可靠
* 灵活
* 转为与其他Amazon Web Service配合使用而设计（A记录-ALIAS）
* 简单
* 速度快
* 经济高效
* 安全
* 可扩展
* 简化混合云

![image-20250512210319012](/Users/matt/Desktop/Learning/learning_notes/notes/images/aws17.png)

### 实战演习

* 如何申请域名
* 设置名字服务器NS
* 使用Route53路由



## Route53 - 解析我的Web服务

### 知识点

* 使用Route53解析我的Web服务器

### 采用策略 - 简单路由

简单路由策略：对于为您的域执行给定功能的单一资源（例如为exaple.com提供内容的web服务器）

### 实战演习

* 设置DNS解析
  * 启动ec2-web1
  * 设置域名IP解析



## Route53 - 分流东京和新加坡区，多值应答路由

### 知识点

* 使用Route53 - 分流东京和新加坡区
* 使用多值应答路由策略
* 为Web服务配置健康检查

### 实战演习

* 启动各区的服务器
* 设置DNS健康检查
  * 东京IP
  * 新加坡IP
* 设置多值应答路由
* 停止东京区服务-确认DNS解析结果



## RDS - 被管理的数据库服务

### 知识点

* RDS相关知识

### 定义

RDS是被管理的数据库服务，让开发人员免去系统（OS，DB）管理的麻烦，重点把经历放在业务逻辑的开发。

### 支持的数据库引擎

* Amazon Aurora(MySQL, PostgreSQL)
* MySQL
* MariaDB
* PostgreSQL
* Orancle
* SQL Server

### 优点

* 轻松管理
* 高度可扩展
* 可用且持久
* 速度快
* 安全
* 经济实惠

![image-20250512213550086](/Users/matt/Desktop/Learning/learning_notes/notes/images/aws18.png)

![image-20250512213622987](/Users/matt/Desktop/Learning/learning_notes/notes/images/aws19.png)



## 连接MySQL - Node.js Web应用程序

### 知识点

使用Node.js Web应用程序连接MySQL数据库实例



## AWS CLI - AWS认证必须会的命令行工具

### AWS CLI是什么

AWS Command Line Interface(AWS CLI)是一种开源工具，让您能够在命令行Shell中使用命令与AWS服务进行交互。仅需最少配置，即可使用AWS CLI开始运行命令，以便从终端程序中命令提示符实现与浏览器的AWS管理控制台所提供的功能等同的功能。

### AWS操作方法

* 管理控制台（Browser）
* AWS CLI
* AWS SDK



## S3 - AWS的存储核心， Simple Storage Service

### 知识点

S3的基础知识

### 基础介绍

Amazon Simple Storage Service(Amazon S3)是一种对象存储服务，提供行业领先的可扩展性，数据可用性，安全性和性能。这意味着各种规模和行业的客户都可以使用S3来存储并保护各种用例（如数据湖，网站，移动应用程序，备份和还原，存档，企业应用程序，IoT设备和大数据分析）的数据，容量不限。Amazon S3提供了易于使用的各种功能，因此您可以组织数据并配置精细调整过的使用权限控制，从而满足特定的业务，组织和合规性要求。

### 用途

* 数据文件保存
* 日志文件保存
* 备份快找保存
* 静态网站主机
* 数据湖（data lake）

### 安全性

* SSE-S3
* SSE-KMS
* SSE-C
* CSE

### 存储分类

* S3标准-适用于频繁访问的数据的通用存储
* S3智能分层-适用于具有未知或变化的访问模式的数据
* S3标准 - IA - 不频繁访问
* S3单区 - IA - 不频繁访问
* S3 Glacier - 长期存储
* S3 Glacier Deep Archive - 长期存档
* S3 Outposts - 本地存储



## S3 - 静态网站之王，快速建立网站之首选

### 知识点

* 使用S3快速搭建静态网页网站
* 使用Route 53 服务解析网站域名

### 实战演习

* 设计域名
* 建立同名的S3存储桶
* 上传网页文件到存储桶中
* 设置存储桶为静态网站公开
* 设置Route 53，关联S3存储桶
* 确认动作



## S3 - 区域间复制

### 知识点

* 存储桶内容在全球区域间进行复制

### 实战演习

* 在东京区建立存储桶
  * Name： woyaofuzhi
  * 启用版本控制
* 在新加坡建立存储桶
  * Name： woyaofuzhi_backup
  * 启用版本控制
* 在东京区建立复制规则
  * 指定全文件
  * 复制到新加坡存储桶
  * 启用复制时间控制（S3 RTC）
* 查看复制结果
  * 一般需要等待15分钟左右
  * 错误排查



## S3- 网关终端结点 - 私有网络访问S3的捷径

### 知识点

通过设置网关终端节点，使私有网段中的EC2也可以访问到S3服务

### 实战步骤

* 建立一个可以访问S3的角色 - KomaRoleS3FullAccess
* 在私有网段启动一个EC2实例，赋予S3访问的角色
* 在私有网段EC2种访问S3
* 在VPC中建立一个网关终端节点，绑定到私有网段的路由表
* 稍等片刻（添加到路由表需要时间）
* 再次从私有网段中的EC2访问S3，确认动作

![image-20250513091411637](/Users/matt/Desktop/Learning/learning_notes/notes/images/aws20.png)



## Budgets - 使用预算跟踪成本

### 知识点

使用预算跟踪成本，超出预算时发送警告邮件。

### 实战演习

* 预算类型：成本预算
* 设置预算金额：$10
* 预算名称：我的预算我做主
* 提醒阀值：50% 实际预算金额
* SNS提醒

































