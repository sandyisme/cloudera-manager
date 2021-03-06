# Centos7 一键安装cloudera manager

## 集群的规划 
|     主机      | hostname | cloudera-manager |
| :---------: | :------: | :--------------: |
| 192.168.x.x |  sandy1  |      server      |
| 192.168.x.x |  sandy2  |      agent       |
| 192.168.x.x |  sandy3  |      agent       |
| 192.168.x.x |  sandy4  |      agent       |



## 安装环境的准备

### 下载安装cloudera manager安装包

- 在server(sandy1)上下载cloudera manager安装包(下载到/home/sandy目录下)

```shell
wget http://archive.cloudera.com/cm5/installer/latest/cloudera-manager-installer.bin
```

### 修改hosts文件

```shell
sudo vi /etc/hosts
192.168.x.x sandy1
192.168.x.x sandy2
192.168.x.x sandy3
192.168.x.x sandy4
```
### 配置ssh免秘钥登录

- 在各个机器上产生各自的公钥

```shell
ssh-keygen -t rsa
```

- 在server（sandy1）节点上复制公钥

```shell
cd .ssh/
cp id_rsa.pub authorized_keys
```

- 将各个机器生成的公钥发送到server节点`authorized_keys`

```shell
cd .ssh/
ssh-copy-id sandy1
```

- 将server节点生成的带所有机器的秘钥传送到其他节点

```shell
scp -r authorized_keys sandy@sandy2:~/.ssh/
scp -r authorized_keys sandy@sandy3:~/.ssh/
scp -r authorized_keys sandy@sandy4:~/.ssh/
```

- 修改.ssh目录及.ssh目录下文件的权限

```shell
cd
chmod 700 .ssh
chmod 600 .ssh/*
```

- 测试连接（推荐每台机器上测试一下，建立连接）
```
ssh sandy2
ssh sandy3
ssh sandy4
```

### java 环境的配置

- 首先查看是否存在自带的java环境

```shell
java -version(若显示bash: java:未找到命令...，则表示未安装，跳过卸载步骤,直接下载jdk安装包)
openjdk version "1.8.0_131" 
OpenJDK Runtime Environment (build 1.8.0_131-b12)
OpenJDK 64-Bit Server VM (build 25.131-b12, mixed mode)
```

- 卸载自带的java环境

```shell
sudo yum remove java*
```

- 下载1.7的jdk安装包(Linux x64 jdk-7u80-linux-x64.tar.gz)

  > http://www.oracle.com/technetwork/java/java-archive-downloads-javase7-521261.html

- 将下载的jdk-7u80-linux-x64.tar.gz上传到/home/sandy路径下，解压tar包

```shell
tar -zxvf jdk-7u80-linux-x64.tar.gz
mv jdk1.7.0_80 jdk
scp -r jdk sandy@sandy2:~
scp -r jdk sandy@sandy3:~
scp -r jdk sandy@sandy4:~
```

- java配置环境变量

```shell
sudo vi /etc/profile
export JAVA_HOME=/home/sandy/jdk
export CLASSPATH=.:$CLASSPTAH:$JAVA_HOME/lib
export PATH=$PATH:$JAVA_HOME/bin
```

- 使配置的java环境变量生效

```shell
source /etc/profile
```

- 检验配置的java环境变量是否生效

```shell
java -version
```

- java环境变量生效显示以下内容

```shell
java version "1.7.0_80"
Java(TM) SE Runtime Environment (build 1.7.0_80-b15)
Java HotSpot(TM) 64-Bit Server VM (build 24.80-b11, mixed mode)
```
- 在各个节点安装ntp时间同步

```shell
sudo yum install -y ntp
systemctl is-enabled ntpd
systemctl enable ntpd
systemctl is-enabled ntpd
systemctl start ntpd
```

### 关闭SElinux

- 通过修改SELinux的配置文件来关闭SElinux（需要重启才会生效）

```shell
sudo vim /etc/selinux/config
SELINUX=disabled  
```

- 临时修改SELinux 

```shell
setenforce 0
```

- 查看selinux状态

```she
getenforce 
```

- 显示状态为`enforcing`表示selinux没有关闭成功,`disabled `表示通过修改SELinux的配置文件来关闭SElinux成功，`Permissive`临时修改SELinux 成功

### 关闭防火墙

- 查看防火墙状态

```shell
systemctl status firewalld
```

- 防火墙未关闭状态(需关闭防火墙)

```shell
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since 二 2018-07-17 21:48:49 CST; 6h left
```

- 防火墙关闭状态(可跳过防火墙的关闭步骤)

```shell
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: inactive (dead) since 二 2018-07-17 15:00:07 CST; 10min ago
```

- 关闭防火墙（重启后防火墙也会启动）

```shell
systemctl stop firewalld
```
- 禁止防火墙开机启动

```shell
systemctl disable firewalld.service
```

- 查看防火墙是否开机启动

```shell
systemctl is-enabled firewalld.service;echo $?
```

## 安装cloudera manager

- 执行安装命令

```
cd
chmod +x cloudera-manager-installer.bin
sudo ./cloudera-manager-installer.bin
```

- 安装步骤

![ ](https://github.com/sandyisme/cloudera-manager/raw/master/Install_Image/install_cloudera_manager_1.png)

![ ](https://github.com/sandyisme/cloudera-manager/raw/master/Install_Image/install_cloudera_manager_2.png)

![ ](https://github.com/sandyisme/cloudera-manager/raw/master/Install_Image/install_cloudera_manager_3.png)

![ ](https://github.com/sandyisme/cloudera-manager/raw/master/Install_Image/install_cloudera_manager_4.png)

![ ](https://github.com/sandyisme/cloudera-manager/raw/master/Install_Image/install_cloudera_manager_5.png)

![ ](https://github.com/sandyisme/cloudera-manager/raw/master/Install_Image/install_cloudera_manager_6.png)
- 注：此步骤可能多次失败，出现如下页面，那是因为网络原因，重复安装步骤，直到完成安装完成(或者可以根据提示的log,cat log 得到下载地址wget下载，然后rpm安装)

![ ](https://github.com/sandyisme/cloudera-manager/raw/master/Install_Image/install_cloudera_manager_7.png)
- 出现如下页面，恭喜你安装成功

![ ](https://github.com/sandyisme/cloudera-manager/raw/master/Install_Image/install_cloudera_manager_8.png)

![ ](https://github.com/sandyisme/cloudera-manager/raw/master/Install_Image/install_cloudera_manager_9.png)
- 然后打开浏览器，输入192.168.x.x:7180(sandy1的ip),登录用户名和密码都是admin

![ ](https://github.com/sandyisme/cloudera-manager/raw/master/Install_Image/install_cloudera_manager_10.png)
