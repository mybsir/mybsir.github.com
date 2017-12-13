# Docker-CE 安装

## 官方方法
```
#安装依赖包
yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
#添加yum源
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
#官方软件源默认启用了最新的软件，您可以通过编辑软件源的方式获取各个版本的软件包。例如官方并没有将测试版本的软件源置为可用，你可以通过以下方式开启。同理可以开启各种测试版本等。
yum-config-manager --enable docker-ce-edge

yum-config-manager --enable docker-ce-test

# 安装docker
yum install docker-ce

```

## 推荐使用阿里云yum源

```
# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3: 更新并安装 Docker-CE
sudo yum makecache fast
sudo yum -y install docker-ce
# Step 4: 开启Docker服务
sudo service docker start

# 注意：
# 官方软件源默认启用了最新的软件，您可以通过编辑软件源的方式获取各个版本的软件包。例如官方并没有将测试版本的软件源置为可用，你可以通过以下方式开启。同理可以开启各种测试版本等。
# vim /etc/yum.repos.d/docker-ee.repo
#   将 [docker-ce-test] 下方的 enabled=0 修改为 enabled=1
#
# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# yum list docker-ce.x86_64 --showduplicates | sort -r
#   Loading mirror speeds from cached hostfile
#   Loaded plugins: branch, fastestmirror, langpacks
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            @docker-ce-stable
#   docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable
#   Available Packages
# Step2 : 安装指定版本的Docker-CE: (VERSION 例如上面的 17.03.0.ce.1-1.el7.centos)
# sudo yum -y install docker-ce-[VERSION]
```
## 启动docker
```
systemctl enable docker
systemctl start docker
```

## 安装校验

```
# docker version
Client:
 Version:      17.11.0-ce
 API version:  1.34
 Go version:   go1.8.3
 Git commit:   1caf76c
 Built:        Mon Nov 20 18:35:47 2017
 OS/Arch:      linux/amd64

Server:
 Version:      17.11.0-ce
 API version:  1.34 (minimum version 1.12)
 Go version:   go1.8.3
 Git commit:   1caf76c
 Built:        Mon Nov 20 18:45:06 2017
 OS/Arch:      linux/amd64
 Experimental: false
```

## 添加内核配置参数
```
tee -a /etc/sysctl.conf <<-EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

# 重新加载sysctl.conf
sysctl -p

```

## 镜像加速

```
mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://230dbfkl.mirror.aliyuncs.com"]
}
EOF
systemctl daemon-reload
systemctl restart docker
```

## 问题


安装过高版本docker-ce之后，想再次安装低版本docker-ce ，提示以下错误

```
[root@k8s-master-1 yum.repos.d]# yum -y install docker-ce-17.04.0.ce-1.el7.centos
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
正在解决依赖关系
--> 正在检查事务
---> 软件包 docker-ce.x86_64.0.17.04.0.ce-1.el7.centos 将被 安装
--> 正在处理依赖关系 docker-ce-selinux >= 17.04.0.ce-1.el7.centos，它被软件包 docker-ce-17.04.0.ce-1.el7.centos.x86_64 需要
软件包 docker-ce-selinux 已经被 docker-ce 取代，但是取代的软件包并未满足需求
--> 解决依赖关系完成
错误：软件包：docker-ce-17.04.0.ce-1.el7.centos.x86_64 (docker-ce-edge)
          需要：docker-ce-selinux >= 17.04.0.ce-1.el7.centos
          可用: docker-ce-selinux-17.03.0.ce-1.el7.centos.noarch (docker-ce-stable)
              docker-ce-selinux = 17.03.0.ce-1.el7.centos
          可用: docker-ce-selinux-17.03.1.ce-0.1.rc1.el7.centos.noarch (docker-ce-test)
              docker-ce-selinux = 17.03.1.ce-0.1.rc1.el7.centos
          可用: docker-ce-selinux-17.03.1.ce-1.el7.centos.noarch (docker-ce-stable)
              docker-ce-selinux = 17.03.1.ce-1.el7.centos
          可用: docker-ce-selinux-17.03.2.ce-0.1.rc1.el7.centos.noarch (docker-ce-test)
              docker-ce-selinux = 17.03.2.ce-0.1.rc1.el7.centos
          可用: docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch (docker-ce-stable)
              docker-ce-selinux = 17.03.2.ce-1.el7.centos
          可用: docker-ce-selinux-17.04.0.ce-0.1.rc1.el7.centos.noarch (docker-ce-test)
              docker-ce-selinux = 17.04.0.ce-0.1.rc1.el7.centos
          可用: docker-ce-selinux-17.04.0.ce-0.2.rc2.el7.centos.noarch (docker-ce-test)
              docker-ce-selinux = 17.04.0.ce-0.2.rc2.el7.centos
          可用: docker-ce-selinux-17.04.0.ce-1.el7.centos.noarch (docker-ce-edge)
              docker-ce-selinux = 17.04.0.ce-1.el7.centos
          可用: docker-ce-selinux-17.05.0.ce-0.1.rc1.el7.centos.noarch (docker-ce-test)
              docker-ce-selinux = 17.05.0.ce-0.1.rc1.el7.centos
          可用: docker-ce-selinux-17.05.0.ce-0.2.rc2.el7.centos.noarch (docker-ce-test)
              docker-ce-selinux = 17.05.0.ce-0.2.rc2.el7.centos
          可用: docker-ce-selinux-17.05.0.ce-0.3.rc3.el7.centos.noarch (docker-ce-test)
              docker-ce-selinux = 17.05.0.ce-0.3.rc3.el7.centos
          可用: docker-ce-selinux-17.05.0.ce-1.el7.centos.noarch (docker-ce-edge)
              docker-ce-selinux = 17.05.0.ce-1.el7.centos
 您可以尝试添加 --skip-broken 选项来解决该问题
 您可以尝试执行：rpm -Va --nofiles --nodigest
```

解决方法：

```
yum -y erase container-selinux
yum install --setopt=obsoletes=0    docker-ce-17.03.2.ce-1.el7.centos.x86_64    docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch
```
