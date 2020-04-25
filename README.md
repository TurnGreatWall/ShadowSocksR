# SSRR后端部署程序

#### 兼容SSRPanel的自改版SSR(R)后端，可兼容原版SS、SSR，本版本是带有IP自动上报功能的


# 更新软件源
#### CentOS 7
- yum -y update
- yum -y groupinstall "Development Tools"
- yum -y install wget


# Centos7 安装pip

1.先安装epel扩展源

EPEL(http://fedoraproject.org/wiki/EPEL) 是由 Fedora 社区打造，为 RHEL 及衍生发行版如 CentOS、Scientific Linux 等提供高质量软件包的项目。

- yum -y install epel-release


2.安装pip

- yum install python-pip


3.测试pip是否安装成功

- pip

 
#
## (注意：但是有时候发现还是安装不成功，可以用下面办法)

    （1）安装setuptools
     yum install -y python-setuptools
     安装完毕后，easy_install命令就可以使用了
    
    （2）安装pip
     easy_install pip
     通过easy_install安装pip是最为简单的方法。pip默认安装到/usr/bin目录下。
    
    （3）测试pip是否安装成功
     pip



# CentOS 7/8 安装 libsodium 最新版
#### 

1、下载并解压
- wget https://download.libsodium.org/libsodium/releases/libsodium-1.0.18-stable.tar.gz
- tar -zxf libsodium-1.0.18-stable.tar.gz
- cd libsodium-stable
- 备用下载地址：https://down.24kplus.com/linux/libsodium-1.0.18-stable.tar.gz


2、编译安装
- ./configure --prefix=/usr
- make && make check
- make install
- echo /usr/local/lib > /etc/ld.so.conf.d/usr_local_lib.conf
- ldconfig


#### 如出现以下错误：

     config.status: error: Something went wrong bootstrapping makefile fragments
     for automatic dependency tracking.  Try re-running configure with the
     '--disable-dependency-tracking' option to at least be able to build
     the package (albeit without support for automatic dependency tracking). 
#### 执行：

      yum install make -y




# 拉取SSR文件

- cd /root
- git clone https://github.com/TurnGreatWall/shadowsocksr.git
- cd shadowsocksr



# 安装依赖

- sh ./setup_cymysql2.sh
- pip install -r requestment.txt



# 编辑数据库连接信息

- vi usermysql.json

#### host 数据库地址，如果是本机就是127.0.0.1
#### port 数据库连接端口
#### user 数据库连接用户，不推荐使用root
#### password 数据库连接密码
#### db 面板所在数据库
#### node_id 节点ID，对应面板里的 节点列表 最左侧的id（请先将面板搭建好，然后创建一个节点，就有节点ID了）
#### transfer_mul 节点流量计算比例，默认1.0，填1也可以，1表示：用了100M算100M，10表示用了100M算1000M，0.1表示用了100M算10M。
    
    
    
#编辑节点配置（混淆、协议、限速、IPV6）

    vi user-mysql.json

    protocol 协议，带 _compatible 结尾兼容 原版，直接用原版可以改为 origin
    protocol_param 协议参数，配置了的话，客户端也要一致
    obfs 混淆 tls1.2_ticket_auth 可以限制客户端数量 tls1.2_ticket_auth_compatible 兼容原版，直接用原版可以改为 plain
    obfs_param 混淆参数，当obfs为 tls1.2_ticket_auth 的时候，这个值为 1 到 256 之间，表示限制客户端数量
    additional_ports 单端口配置，请看wiki
    additional_ports_only 强制单端口，改为true则所有非设置的单端口都无法连接，只能用additional_ports设置的那些端口连接
    dns_ipv6 为true时，会优先走ipv6，需要节点服务器至少有一个2开头的ipv6地址（有时候会导致IPV4失效，不推荐开启，你可以自己试试）
    connect_verbose_info 为1时记录用户访问网址，推荐打开，可以清楚知道连接成功与否
    redirect 请求失败时返回信息伪造成访问配置里网址



# 运行、关闭、看日志

#### 试运行，如果没有错误输出则可以用Ctrl+C关闭，然后后台运行
- python server.py
 
#### 后台运行
- bash run.sh
 
#### 其他命令
#### 运行并记录日志
- sh logrun.sh
#### 停止
- sh stop.sh
#### 查看日志
- sh tail.sh


#### SSR 开机启动
- chmod +x /etc/rc.d/rc.local
- vi /etc/rc.d/rc.local
 
#### 加入下面的命令，保存
- bash /root/shadowsocksr/run.sh


# CentOS 7.0查看和关闭防火墙

#### 查看防火墙状态

- firewall-cmd --state

#### 停止firewall

- systemctl stop firewalld.service

#### 禁止firewall开机启动

- systemctl disable firewalld.service 


# 更换DNS为谷歌DNS
- echo -e "nameserver 8.8.8.8\nnameserver 8.8.4.4" > /etc/resolv.conf

# 校时
#### 如果架构是“面板机-数据库机-多节点机”，请务必保持各个服务器之间的时间一致，否则会产生：节点的在线数不准确、产生最后使用时间异常、单端口多用户功能失效等。
#### 推荐统一使用CST时间并安装校时服务：
#### 设置时区为东八区
- timedatectl set-timezone Asia/Shanghai
#### 自动同步服务器时间
- yum install ntp
- ntpdate cn.pool.ntp.org

# 其他

#### 数据库机的 iptables、firewall 得对本节点IP开放
#### 数据库机的 mysql 的对本节点进行授权（不推荐使用root账号）

