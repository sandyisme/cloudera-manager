# cloudera manager 配置外部数据库（MySQL）

## 使用rpm安装MySQL

### 下载MySQL的安装包

- 根据官网文档下载cloudera manager支持的MySQL（MySQL Support across CM/CDH 5 Releases）

> https://www.cloudera.com/documentation/enterprise/release-notes/topics/rn_consolidated_pcm.html

- 单个下载需要安装的rpm安装包

> https://dev.mysql.com/downloads/mysql/5.6.html#downloads

- 下载rpm的tar包

> https://dev.mysql.com/downloads/file/?id=478830

- 下载MySQL 的Jdbc驱动包

> https://dev.mysql.com/downloads/connector/j/5.1.html

### 安装MySQL

- 将下载的MySQL rpm安装包或MySQL rpm的tar包解压后上传到sandy2（这里使用的是rpm的tar包）

```shell
cd /usr/local/
mkdir -p mysql
cp /home/sandy2/MySQL-5.6.41-1.el7.x86_64.rpm-bundle /usr/local/mysql/
cd mysql/MySQL-5.6.41-1.el7.x86_64.rpm-bundle/
```

- 查看是否使用rpm安装MySQL

```shell
rpm -qa | grep -i mysql
```

- 若查询到安装过MySQL相关的rpm包，先卸载在安装(根据实际情况卸载，我这边只是个例子)

```shell
rpm -e MySQL-python-1.2.5-1.el7.x86_64 --nodeps
```

- 创建mysql的用户和用户组

```shell
sudo groupadd mysql
sudo useradd mysql -g mysql -p mysql
```

- rpm安装相关包

```shell
sudo rpm -ivh MySQL-server-5.6.41-1.el7.x86_64.rpm  --force --nodeps
sudo rpm -ivh MySQL-client-5.6.41-1.el7.x86_64.rpm
sudo rpm -ivh MySQL-devel-5.6.41-1.el7.x86_64.rpm
sudo rpm -ivh MySQL-shared-5.6.41-1.el7.x86_64.rpm 
```

- 安装MySQL-server包时若出现/etc/my.cnf存在的错误，先删除/etc/my.cnf，再执行以上rpm安装

```shell
sudo rm -rf /etc/my.cnf
```

- 安装完成后查看mysql进程、端口是否启动

```shell
ps -ef|grep mysqld 
netstat -an |grep :3306
```

- 若mysql没有启动先启动mysql

```shell
sudo service mysql start
```

- 修改mysql登录密码

```shell
sudo cat /root/.mysql_secret
mysqladmin -u root -p password
```

- 登录mysql

```shell
mysql -uroot -psandy
```

- 允许任何IP远程连接mysql

```mysql
use mysql；
select host from user where user = 'root';
+------------+
| host       |
+------------+
| 127.0.0.1  |
| ::1        |
| localhost  |
| sandy2     |
+------------+
update user set host='%' where user = 'root'; 
select host from user where user = 'root';
+------------+
| host       |
+------------+
| %          |
| 127.0.0.1  |
| ::1        |
| sandy2     |
+------------+
flush privileges;
```
### MySQL 的JDBC驱动添加

- 将下载的MySQL 的Jdbc驱动包上传，并解压

```shell
tar -zxvf mysql-connector-java-5.1.46.tar.gz
sudo mkdir /usr/share/java
sudo mv mysql-connector-java-5.1.46-bin.jar /usr/share/java
cd /usr/share/java
sudo mv mysql-connector-java-5.1.46-bin.jar mysql-connector-java.jar
```

## MySQL创建cloudera manager 所需的数据库

- 创建cm cdh数据库环境并授予权限（根据实际需求创建）

```mysql
CREATE DATABASE hive DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL  privileges ON *.* TO 'hive'@'%' IDENTIFIED BY 'sandy' with grant option;
CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL  privileges ON *.* TO 'hue'@'%' IDENTIFIED BY 'sandy' with grant option;
CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL  privileges ON *.* TO 'oozie'@'%' IDENTIFIED BY 'sandy' with grant option;
GRANT ALL  privileges ON *.* TO 'root'@'%' IDENTIFIED BY 'sandy' with grant option;
```

## 修改cloudera manager数据库连接，配置外部数据库

- 将cloudera manager的嵌入式postgreSQL数据库连接模式，修改成支持的外部数据库mysql连接模式（mysql -h 新节点 -u 用户名 -p ’密码’ --scm-host CMS主机 scm_db_name scm_user scm_password）

```shell
sudo /usr/share/cmf/schema/scm_prepare_database.sh mysql -h sandy2 -uroot -psandy --scm-host sandy scm scm sandy
```

- 查看迁移数据连接是否成功，若连接方式修改成mysql，连接信息和上述相同，则表示前已成功

```shell
sudo cat /etc/cloudera-scm-server/db.properties
com.cloudera.cmf.db.type=mysql
com.cloudera.cmf.db.host=sandy2
com.cloudera.cmf.db.name=scm
com.cloudera.cmf.db.user=scm
com.cloudera.cmf.db.setupType=EXTERNAL
com.cloudera.cmf.db.password=sandy
```

- 重启cloudera manager server，使数据迁移生效

```shell
sudo service cloudera-scm-server stop
sudo service cloudera-scm-server-db stop   
sudo service cloudera-scm-server start
```
- 登录页面，可以看到没有再提示使用的是嵌入式postgresql数据库模式，请在移入生成环境前切换为使用支持的外部数据库

  ![](https://github.com/sandyisme/cloudera-manager/raw/master/Install_Image/mysql.png)

  ![](https://github.com/sandyisme/cloudera-manager/raw/master/Install_Image/postgresql.png)
