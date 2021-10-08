# 静态ip配置

## Centos

```shell
cd /etc/sysconfig/network-scripts/
#编辑网卡配置
TYPE=Ethernet
ROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPADDR=192.168.1.123
NETMAST=255.255.255.0
GATEWAY=192.168.1.1
DNS1=218.85.152.99
DNS2=114.114.114.114
IPV4_FAILURE_FATAL=no
#IPV6INIT="yes"
#IPV6_AUTOCONF="yes"
#IPV6_DEFROUTE="yes"
#IPV6_FAILURE_FATAL="no"
#IPV6_ADDR_GEN_MODE="stable-privacy"
NAME=ens33
UUID=c835380e-889e-47b2-8d3e-b238151b7478
DEVICE=ens33
#重启
reboot
```

# 换源

## Centos

```shell
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
#CentOS 8
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-8.repo
#或者
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-8.repo
#CentOS 7
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
#或
curl -o /etc/
yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
#生成缓存
yum makecache
```

## ArchLinux

```bash
#编辑配置文件
sudo vim /etc/pacman.conf
#在底部 添加第三方用户源 
[archlinuxcn]
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
#导入 GPG key
sudo pacman -S archlinuxcn-keyring
```

# 图像化界面开启/关闭

```shell
#设置为图像化界面
systemctl set-default graphical.target
#设置为命令行界面
systemctl set-default multi-user.target
```

# 网络代理

```shell
#下载privoxy安装包
wget https://download-ib01.fedoraproject.org/pub/epel/7/x86_64/Packages/p/privoxy-3.0.32-1.el7.x86_64.rpm
#安装
yum install -y privoxy-3.0.32-1.el7.x86_64.rpm
#配置privoxy
vim /etc/privoxy/config
#末尾增加下面内容,/后面是代理服务器的地址:端口,注意最后还有个.
forward-socks5t / x.x.x.x:10808 .
#启动服务
systemctl start privoxy
#设置系统代理变量 注:8118是privoxy默认使用的端口
export all_proxy=http://127.0.0.1:8118
export http_proxy=http://127.0.0.1:8118
export https_proxy=http://127.0.0.1:8118
#使用v2ray代理时要开启允许局域网的连接
```

# 端口命令

[参考来源-Centos8开放防火墙端口](https://blog.csdn.net/qq_32656561/article/details/105619911)

```shell
#查看启动的端口
netstat -tunlp
#查看防火墙某个端口是否开放
firewall-cmd --query-port=3306/tcp
#开放防火墙端口3306
firewall-cmd --zone=public --add-port=3306/tcp --permanent
#注意：开放端口后要重启防火墙生效
#重启防火墙
systemctl restart firewalld
#关闭防火墙端口
firewall-cmd --remove-port=3306/tcp --permanent
#查看防火墙状态
systemctl status firewalld
#关闭防火墙
systemctl stop firewalld
#打开防火墙
systemctl start firewalld
#开放一段端口
firewall-cmd --zone=public --add-port=40000-45000/tcp --permanent
#查看开放的端口列表
firewall-cmd --zone=public --list-ports
#查看被监听(Listen)的端口
netstat -lntp
#检查端口被哪个进程占用
netstat -lnp|grep 3306
```

# docker

## 安装

```shell
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
sudo yum install docker-ce docker-ce-cli containerd.io
sudo systemctl start docker
#开机自启
sudo systemctl enable docker
```

## 非root运行

docker守护进程启动的时候，会默认赋予名字为docker的用户组读写Unix socket的权限，因此只要创建docker用户组，并将当前用户加入到docker用户组中，那么当前用户就有权限访问Unix socket了，进而也就可以执行docker相关命令。

```shell
sudo groupadd docker     #添加docker用户组
sudo gpasswd -a $USER docker     #将登陆用户加入到docker用户组中
newgrp docker     #更新用户组
docker ps    #测试docker命令是否可以使用sudo正常使用
```

## 镜像加速

```shell
vi  /etc/docker/daemon.json
#内容
{"registry-mirrors":["https://hub-mirror.c.163.com/","https://docker.mirrors.ustc.edu.cn/","https://reg-mirror.qiniu.com"]}

#修改后重启服务
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## docker API端口

### 直接模式

[Docker服务开启TCP端口](https://blog.csdn.net/wy122222222/article/details/108281357)

```bash
#编辑配置文件
sudo vim /lib/systemd/system/docker.service
#注释原有的：
#ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
#添加新的：
#-H代表指定docker的监听方式，这里是socket文件文件位置，也就是socket方式，2375就是tcp端口
ExecStart=/usr/bin/dockerd -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375
#重新加载系统服务配置文件（包含刚刚修改的文件）
systemctl daemon-reload
#重启docker服务
systemctl restart docker
#以下步骤非必要，用于测试是否工作正确以及外网对docker的访问
#查看端口是否被docker监听
ss -tnl | grep 2375
#查看防火墙是否开放2375端口
firewall-cmd --zone=public --query-port=2375/tcp
#防火墙添加开放2375端口
firewall-cmd --zone=public --add-port=80/tcp --permanent
#重启防火墙
firewall-cmd --reload
#测试端口是否可以使用
#ip为docker所在的服务器的ip
telnet 192.168.123.68 2375
```

### SSL加密

[centos7 docker配置TLS认证的远程端口的证书生成教程（shell脚本一键生成）_新林的博客-CSDN博客](https://blog.csdn.net/qq_21187515/article/details/90268345)

[centos7 docker开启认证的远程端口2376配置教程_新林的博客-CSDN博客](https://blog.csdn.net/qq_21187515/article/details/90262324)

[使用docker-java远程管理docker精进版(二）_yaofeng_hyy的博客-CSDN博客_docker-java](https://blog.csdn.net/yaofeng_hyy/article/details/80923941)

```shell
#生成认证证书、密钥
#一键生成脚本如下
```

```sh
#!/bin/bash
#相关配置信息
SERVER="192.168.33.76"
PASSWORD="pass123456"
COUNTRY="CN"
STATE="广州省"
CITY="广州市"
ORGANIZATION="公司名称"
ORGANIZATIONAL_UNIT="Dev"
EMAIL="188888888@qq.com"

###开始生成文件###
echo "开始生成文件"

#切换到生产密钥的目录
cd /etc/docker   
#生成ca私钥(使用aes256加密)
openssl genrsa -aes256 -passout pass:$PASSWORD  -out ca-key.pem 2048
#生成ca证书，填写配置信息
openssl req -new -x509 -passin "pass:$PASSWORD" -days 3650 -key ca-key.pem -sha256 -out ca.pem -subj "/C=$COUNTRY/ST=$STATE/L=$CITY/O=$ORGANIZATION/OU=$ORGANIZATIONAL_UNIT/CN=$SERVER/emailAddress=$EMAIL"

#生成server证书私钥文件
openssl genrsa -out server-key.pem 2048
#生成server证书请求文件
openssl req -subj "/CN=$SERVER" -new -key server-key.pem -out server.csr
#使用CA证书及CA密钥以及上面的server证书请求文件进行签发，生成server自签证书
openssl x509 -req -days 3650 -in server.csr -CA ca.pem -CAkey ca-key.pem -passin "pass:$PASSWORD" -CAcreateserial  -out server-cert.pem

#生成client证书RSA私钥文件
openssl genrsa -out key.pem 2048
#生成client证书请求文件
openssl req -subj '/CN=client' -new -key key.pem -out client.csr

sh -c 'echo "extendedKeyUsage=clientAuth" > extfile.cnf'
#生成client自签证书（根据上面的client私钥文件、client证书请求文件生成）
openssl x509 -req -days 3650 -in client.csr -CA ca.pem -CAkey ca-key.pem  -passin "pass:$PASSWORD" -CAcreateserial -out cert.pem  -extfile extfile.cnf

#更改密钥权限
chmod 0400 ca-key.pem key.pem server-key.pem
#更改密钥权限
chmod 0444 ca.pem server-cert.pem cert.pem
#删除无用文件
rm client.csr server.csr

echo "生成文件完成"
###生成结束###
```

```shell
#编辑docker文件：/usr/lib/systemd/system/docker.service
vi /usr/lib/systemd/system/docker.service
#在ExecStart属性后面追加（我们刚刚自己生成的证书路径）(配置 docker 使用 TLS 认证并监听 tcp 端口)
--tlsverify \
--tlscacert=/etc/docker/ca.pem \
--tlscert=/etc/docker/server-cert.pem \
--tlskey=/etc/docker/server-key.pem \
-H tcp://0.0.0.0:2376 \
-H unix:///var/run/docker.sock
```

![在这里插入图片描述](https://i.loli.net/2021/10/01/sDEF1RtL9CdV5My.png)

```shell
#重新加载docker配置后重启docker服务
systemctl daemon-reload
systemctl restart docker
#用netstat -tunlp查看是否存在2376端口，可以看到端口已启动
```

![img](https://i.loli.net/2021/10/01/leMw6QqELuO2Htm.png)



## Docker MySQL

```bash
#下载镜像
sudo docker pull mysql:5.7
#安装镜像
docker run -p 3306:3306 --name mysql \
-v /etc/mysql/log:/var/log/mysql \
-v /etc/mysql/data:/var/lib/mysql \
-v /etc/mysql/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root \
-d mysql:5.7
```

参数说明

```
-p 3306:3306：将容器的 3306 端口映射到主机的 3306 端口
-v /etc/mysql/conf:/etc/mysql：将配置文件夹挂载到主机
-v /etc/mysql/log:/var/log/mysql：将日志文件夹挂载到主机
-v /etc/mysql/data:/var/lib/mysql/：将配置文件夹挂载到主机
-e MYSQL_ROOT_PASSWORD=root：初始化 root 用户的密码
```

mysql配置文件

```bash
#创建文件夹
mkdir -p /etc/mysql/conf
#编辑文件
vi /etc/mysql/conf/my.cnf
#配置完成后重启mysql
docker restart mysql
#开机自启
sudo docker update mysql --restart=always
```

配置内容

```
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
[mysqld]
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
skip-name-resolve
```

配置解释

skip-name-resolve：跳过域名解析

**命令**

```bash
#通过容器的 mysql 命令行工具连接 mysql
docker exec -it mysql mysql -uroot -proot
#进入容器文件系统
docker exec -it mysql /bin/bash
```

设置 root 远程访问

```bash
#连接命令行
docker exec -it mysql mysql -uroot -proot
#允许远程连接
grant all privileges on *.* to 'root'@'%' identified by 'root' with grant option;
flush privileges;
```

## Docker PHPMyAdmin

```shell
#文档 https://registry.hub.docker.com/_/phpmyadmin
#拉取镜像
docker pull phpmyadmin
#启动phpmyadmin
#--name mysql-manager 制定容器名称
#-d 后台运行
#--link 链接docker mysql
# mysql:db , mysql为docker mysql 的名称
#-p端口映射 将本机3307端口映射到容器内部80端口
docker run --name mysql-manager -d --link mysql:db -p 3307:80 phpmyadmin
#开机自启
sudo docker update mysql-manager --restart=always
```

## Docker Nginx

```shell
#镜像下载
docker pull nginx
#启动镜像 -rm 和 -d 一起使用会导致容器停止时，需要重新run
docker run --rm -d -p 80:80 --name nginx nginx
#配置复制
docker cp nginx:/etc/nginx/nginx.conf /etc/nginx/conf/nginx.conf
docker cp nginx:/etc/nginx/conf.d /etc/nginx/conf/
#删除原有镜像
docker rm -f nginx
#启动镜像，并进行配置映射
docker run --rm -d -p 80:80 --name nginx \
  -v /etc/nginx/conf/conf.d:/etc/nginx/conf.d \
  -v /etc/nginx/www:/usr/share/nginx/html \
  -v /etc/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
  -v /etc/nginx/logs:/var/log/nginx \
  nginx
#以下命令再容器退出后，使用docker ps -a 可以看到被退出的容器
docker run -d -p 80:80 --name nginx \
  -v /etc/nginx/conf/conf.d:/etc/nginx/conf.d \
  -v /etc/nginx/www:/usr/share/nginx/html \
  -v /etc/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
  -v /etc/nginx/logs:/var/log/nginx \
  nginx
#原文
#To start a container in detached mode, you use -d=true or just -d option. By design, containers started in detached mode exit when the root process used to run the container exits, unless you also specify the --rm option. If you use -d with --rm, the container is removed when it exits or when the daemon exits, whichever happens first.

#开机自启
sudo docker update nginx --restart=always
```



# Java环境安装

```shell
#查询yum源中的jdk版本
yum list | grep jdk
#安装
yum -y install java-1.8.0-openjdk java-1.8.0-openjdk-devel
#获取JAVA_HOME
dirname $(readlink $(readlink $(which java)))
```

# 常见目录

```shell
#开机自启等配置文件
/usr/lib/systemd/system
#全局环境变量 source /etc/profile 更新
/etc/profile
```

