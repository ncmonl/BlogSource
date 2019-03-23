---
title: Ubuntu搭建Nginx图片服务器
date: 2018-01-11 20:40:11
categories: "技术栈" 
tags:
  - Linux
  - Ubuntu  
  - FTP
---
  
也许算作是建站以来的第一篇真正意义上的总结博客。  

首先感谢开源本主题的viosey同学和辛苦维护本主题的neoFelhz同学，对此主题极为喜欢。

新发现了当前的这个Meterial主题后发现居然有一个Gallery模板可以展示照片，平时的博客使用图片需要存放在一个位置上，平时也有一些业余的摄影爱好需要刚好可以存放，当然可以使用如之前使用过的七牛云等云对象存储平台存储使用，我这里也是刚好有个云服务器可以做存储就想着自己搭建一个图片服务器方便管理，也刚好能学习一下服务器方面的一些知识，经多次尝试，故总结一下,话不多说，开始行动.

<!--more-->
# 环境准备 #

Ubuntu版本Ubuntu 16.04.3
# 安装Nginx #
首先是准备编译Nginx的前期准备工作  

## 先更新一下源 ##
    > apt-get update
## 安装gcc g++依赖库 ##
    > apt-get install build-essential
    > apt-get install libtool
## 安装prce依赖库 ##
    > apt-get install libpcre3 libpcre3-dev
## 安装 zlib依赖库 ##
	> apt-get install zlib1g-dev

## 安装 ssl依赖库 ##
    > apt-get install openssl

## 编译Nginx ##
下载[Nginx](http://nginx.org/en/download.html)有对应的版本，我这里下载是当前的最新版本1.13.8
    
    > #解压下载下来的压缩包
    > tar -zxvf nginx-1.13.8.tar.gz
    > #进入解压目录
    > cd nginx-1.13.8
    > #配置并生成makefile
    > ./configure --prefix=/usr/local/nginx 
    > #编译
    > make
    > #安装
    > make install
    > #启动Nginx
    > /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf

此时Nginx就安装完成了，会使用默认的80端口启动，如果有启动apache2服务的这里会用冲突，这里不是重点，可查询解决，启动完成可直接通过服务器ip或者云解析的域名查看默认网页  

默认的网页长这个样子  
![Nginx-default-index](https://ncmon-blog.oss-cn-beijing.aliyuncs.com/image6.png)

# 安装与配置vsftpd #
## 安装vsftpd ##
    > apt-get install vsftpd

## 启动vsftpd服务 ##
	>service vsftpd start

## 新建ftpuser目录作为ftp主目录 （ftpuser为目录名，随个人喜好创建）##
	>mkdir /home/ftpuser
	
## 新建ftpuser用户指定用户主目录和设置用户密码 （ftpuser为用户名，随个人喜好创建）##
	>useradd -d /home/ftpuser -s /bin/bash ftpuser
	>passwd ftpuser
## 制定用户组 ##
	>chown ftpuser:ftpuser /home/ftpuser
## 新建文件/etc/vsftpd.user_list，用于存放允许访问ftp的用户 ##
	>vim /etc/vsftpd.user_list
	
![ftp-user_list](https://ncmon-blog.oss-cn-beijing.aliyuncs.com/image7.png)

## 编辑vsftpd配置文件  ##
	>vim /etc/vsftpd.conf
    做如下修改：   
    　　打开注释 write_enable=YES   
    　　添加信息 userlist_file=/etc/vsftpd.user_list   
    　　添加信息 userlist_enable=YES   
    　　添加信息 userlist_deny=NO 
    　　修改完成后保存退出。
## 重启vsftpd服务 ##
	>service vsftpd restart


这是可以使用filezilla等ftp软件使用刚刚新建的用户名和密码访问测试是否成功


OK，准备工作完成开始上传图片，开始正式图片服务器工作

## 创建存储图片的根目录 （在ftpuser目录下,我这里使用www/images）##
	>cd /home/ftpuser
	>mkdir -p www/images
## 在nginx目录下创建images目录 ##
	>mkdir /usr/local/nginx/html/images
## 修改nginx/conf/nginx.conf在默认的server里再添加一个location并指定实际路径: ##
	location /images/ {
    	root  /home/ftpuser/www/;
    	autoindex on;
	}  

![images_nginx_conf](https://ncmon-blog.oss-cn-beijing.aliyuncs.com/image8.png)

## 修改用户访问权限 ##
	>chmod 755 /home/ftpuser/www/images

使用filezilla 等ftp工具使用ftpuser用户登录即可上传图片


# 测试 #

![ftp_upload_test](https://ncmon-blog.oss-cn-beijing.aliyuncs.com/image9.png)  
送上一张最近的长安街慢门效果照片*★,°*:.☆\(￣▽￣)/$:*.°★*。大功告成！ 撒花！
![image_test_result](https://ncmon-blog.oss-cn-beijing.aliyuncs.com/image10.png)