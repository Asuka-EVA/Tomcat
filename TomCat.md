# 企业TomCat运维

![image-20230413175434264](D:\图片\typora\image-20230413175434264.png)

## 1、系统环境说明

```shell
[root@java-tomcat1 ~]# cat /etc/redhat-release 
##查看系统版本, 红帽发行信息

[root@java-tomcat1 ~]# uname -a 
##显示系统名、节点名称、操作系统的发行版号、内核版本等等

[root@java-tomcat1 ~]# getenforce 
##查看系统当前 SELinux 的工作模式

[root@java-tomcat1 ~]# systemctl status firewalld
##查看防火墙的状态
```



## 2、安装jdk

```shell
[root@java-tomcat1 ~]# tar xzf jdk-8u191-linux-x64.tar.gz -C /usr/local/

[root@java-tomcat1 ~]# cd /usr/local/

[root@java-tomcat1 local]# mv jdk1.8.0_191/ java

[root@java-tomcat1 local]# vim /etc/profile

##设置环境变量

#############################################

JAVA_HOME=/usr/local/java
PATH=$JAVA_HOME/bin:$PATH

#############################################
```



```shell
export JAVA_HOME=/usr/local/java   #指定java安装目录
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH    #用于指定java系统查找命令的路径
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar  #类的路径，在编译运行java程序时，如果有调用到其他类的时候，在classpath中寻找需要的类。


```

```shell
[root@java-tomcat1 local]# source /etc/profile

##应用环境变量

[root@java-tomcat1 local]# java -version

##检测jdk是否安装成功
```



## 3、安装tomcat

```shell
[root@java-tomcat1 ~]# mkdir /data/application -p
[root@java-tomcat1 ~]# cd /usr/src/
[root@java-tomcat1 ~]# yum -y install wget

[root@java-tomcat1 src]# wget http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-8/v8.5.46/bin/apache-tomcat-8.5.46.tar.gz
[root@java-tomcat1 src]# tar xzf /root/apache-tomcat-8.5.46.tar.gz -C /data/application/
[root@java-tomcat1 src]# cd /data/application/
[root@java-tomcat1 application]# mv apache-tomcat-8.5.46/ tomcat

[root@java-tomcat1 application]# vim /etc/profile
```



```shell
export TOMCAT_HOME=/data/application/tomcat   #指定tomcat的安装目录
##设置环境变量
```

```shell
[root@java-tomcat1 application]# source  /etc/profile

##应用环境变量

[root@java-tomcat1 tomcat]# /data/application/tomcat/bin/version.sh

##查看tomcat是否安装成功

/data/application/tomcat/bin/startup.sh

##启动tomcat程序

[root@java-tomcat1 bin]# netstat -lntp  |grep java

##查看端口号

##打开游览器输入ip查看tomcat是否启动

/data/application/tomcat/bin/shutdown.sh

##关闭tomcat程序
```



## 4、web站点部署

```shell
[root@java-tomcat1 ~]# wget http://updates.jenkins-ci.org/download/war/2.129/jenkins.war
[root@java-tomcat1 ~]# ls
jenkins.war

##下载jenkins的war包

[root@java-tomcat1 ~]# cd /data/application/tomcat   

##进入tomcat目录
[root@java-tomcat1 tomcat]# cp -r webapps/ /opt/    

##将原来的发布网站目录备份

[root@java-tomcat1 tomcat]# cd webapps/
[root@java-tomcat1 webapps]# ls
docs  examples  host-manager  manager  ROOT

[root@java-tomcat1 webapps]# rm -rf *    

##清空发布网站里面的内容
[root@java-tomcat1 webapps]# cp /root/jenkins.war .  

##将war包拷贝到当前目录
[root@java-tomcat1 webapps]# ../bin/startup.sh   

##启动

[root@java-tomcat1 ~]# mkdir /data/application/webapp  

##创建发布目录
[root@java-tomcat1 ~]# vim /data/application/tomcat/conf/server.xml

##将原来的webapps改为webapp

[root@java-tomcat1 ~]# cp /root/jenkins.war /data/application/webapp/
[root@java-tomcat1 ~]# /data/application/tomcat/bin/startup.sh


```



## 5、部署开源站点（jspgou商城）

```shell
[root@yangge ~]# yum -y install mariadb mariadb-server
[root@yangge ~]# systemctl start mariadb
[root@yangge ~]# mysql
create database jspgou default charset=utf8;	

##在数据库中操作，创建数据库并指定字符集
flush privileges;
exit;

##上传jspgou商城的代码
[root@java-tomcat1 ~]# unzip jspgouV6.1-ROOT.zip
[root@java-tomcat1 ~]# cp -r ROOT/ /data/application/tomcat/webapp/
[root@java-tomcat1 ~]# cd /data/application/tomcat/webapp/
[root@java-tomcat1 webapps]# ls
ROOT

##将数据导入数据库:
[root@java-tomcat1 ~]# cd DB/
[root@java-tomcat1 DB]# ls
jspgou.sql
[root@java-tomcat1 DB]# mysql -uroot -p  jspgou < jspgou.sql
source  jspgou.sql

##启动tomcat访问:
[root@java-tomcat1 ~]# /data/application/tomcat/bin/startup.sh
[root@java-tomcat1 ~]# netstat -lntp
```



# TomCat多实例配置

## 1、复制程序文件

```shell
[root@java-tomcat1 ~]# cd /data/application/
[root@java-tomcat1 application]# ls
tomcat
[root@java-tomcat1 application]# cp -r tomcat/ tomcat_2
[root@java-tomcat1 application]# ls
tomcat  tomcat_2
[root@java-tomcat1 application]# sed -i 's@8005@8011@;s/8080/8081/' tomcat/conf/server.xml
[root@java-tomcat1 application]# sed -i 's#8005#8012#;s#8080#8082#' tomcat_2/conf/server.xml
[root@java-tomcat1 application]# sed -i 's#8009#8019#' tomcat/conf/server.xml
[root@java-tomcat1 application]# sed -i 's#8009#8029#' tomcat_2/conf/server.xml

##修改端口，以启动多实例。多实例之间端口不能一致

[root@java-tomcat1 application]# diff tomcat/conf/server.xml tomcat_2/conf/server.xml  #对比文件不同之处
```



## 2、检查端口查看是否启动

```shell
[root@java-tomcat1 application]# netstat -lntp | grep java 

##在浏览器访问并进行测试
```



# TomCat反向代理集群

![image-20230413183742588](D:\图片\typora\image-20230413183742588.png)

## 1、负载均衡器说明

```shell
关闭防火墙和selinux

yum安装nginx
[root@nginx-proxy ~]# cd /etc/yum.repos.d/
[root@nginx-proxy yum.repos.d]# vim nginx.repo
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
[root@nginx-proxy yum.repos.d]# yum install yum-utils -y
[root@nginx-proxy yum.repos.d]# yum install nginx -y
```



## 2、配置负载均衡器

```shell
备份原配置文件并修改

[root@nginx-proxy ~]# cd /etc/nginx/conf.d/
[root@nginx-proxy conf.d]# cp default.conf default.conf.bak
[root@nginx-proxy conf.d]# mv default.conf tomcat.conf
[root@nginx-proxy conf.d]# vim tomcat.conf
upstream testweb {
	server 192.168.50.114:8081 weight=1 max_fails=1 fail_timeout=2s;
	server 192.168.50.114:8082 weight=1 max_fails=1 fail_timeout=2s;
}
server {
    listen       80;
    server_name  localhost;
    access_log  /var/log/nginx/proxy.access.log  main;

location / {
   root /usr/share/nginx/html;
   index index.html index.htm;
   proxy_pass http://testweb;
   proxy_set_header Host $host:$server_port;
   proxy_set_header X-Real-IP $remote_addr;
   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }       
error_page   500 502 503 504  /50x.html;
location = /50x.html {
    root   /usr/share/nginx/html;
} 


}
创建upstream配置文件:
[root@nginx-proxy conf.d]# vim upstream.conf
upstream testweb {
	server 192.168.50.114:8081 weight=1 max_fails=1 fail_timeout=2s;
	server 192.168.50.114:8082 weight=1 max_fails=1 fail_timeout=2s;
}
```

```shell
启动nginx

[root@nginx-proxy ~]# systemctl start nginx

在浏览器上进行访问测试
```

