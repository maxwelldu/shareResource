Redis的主从实验

* 平台: Mac
* 工具: Vagrant
* 系统: Centos6.5
* 实验目的: Redis主从
 
实现思路

制作一个基于sentos6.5, 已经安装好reids的box, 添加一个脚本配置从机, 然后作为后面主机从机的基础box
 
制作sentos6.5+redis的box
 
`mkdir vagrant_redis`
`cd vagrant_redis`
 
1.初始化vagrant

`vagrant init sentos6.5`
 
2.定义两台机器

`vim Vagrantfile`
 
打开公网

`config.vm.network "public_network"`
 
3.启动

`vagrant up`

选择网络的时候选择1, 设置桥接的网卡
 
4.连接到主机

`vagrant ssh`
 
5. 主机安装redis

切换到root用户安装软件

`sudo su -`
 
先安装redis所需要的依赖包

`yum -y install vim gcc-c++ tcl`
 
下载redis

`wget http://download.redis.io/redis-stable.tar.gz`

解压

`tar xvzf redis-stable.tar.gz`

进入目录

`cd redis-stable`

`make`

在make成功以后，会在src目录下多出一些可执行文件：redis-server，redis-cli等等

为了方便使用, 将其复制到usr目录下

`cp src/redis-server /usr/local/bin/`

`cp src/redis-cli /usr/local/bin/`
 
然后新建目录，存放配置文件
 
`mkdir /etc/redis`

`mkdir /var/redis`

`mkdir /var/redis/log`

`mkdir /var/redis/run`

`mkdir /var/redis/6379`
 
在redis解压根目录中找到配置文件模板，复制到如下位置

`cp redis.conf /etc/redis/6379.conf`
 
通过vim命令修改

`vim /etc/redis/6379.conf`

```
daemonize yes
pidfile /var/redis/run/redis_6379.pid
logfile /var/redis/log/redis_6379.log
dir /var/redis/6379
```
 
最后运行redis

`redis-server /etc/redis/6379.conf`

 
写一个从机上面的一键配置脚本, 使用说明: sh config_redis_slave.sh 主机的ip地址 主机的端口
 
`vim config_redis_slave.sh`

```
#!/bin/bash
 
if [ ! $1 ];then
        echo "Usage: sh config_redis_slave ip_addr port"
        exit
fi
 
if [ ! $2 ];then
        echo "Usage: sh config_redis_slave ip_addr port"
        exit
fi
 
echo "slaveof $1 $2" >> /etc/redis/6379.conf
redis-server /etc/redis/6379.conf
```
 
打包

//打包前的准备
  
`sudo rm -rf /etc/udev/rules.d/70-persistent-net.rules`

`exit`

`vagrant package --output sentos6.5redis.box`

`vagrant box add --name sentos6.5redis sentos6.5redis.box`

`vagrant box list`

 
看到sentos6.5redis的box说明已经添加成功了, 基于这个box搭建redis主从集群

`cd ../`

`mkdir redis_master_slave`

`cd redis_master_slave`

 
1.初始化

`vagrant init sentos6.5redis`
 
2.定义一主三从, 从的数量可以根据需求来配置

`vim Vagrantfile`
 
添加一主一从, 配置网络为公网, 让局域网其他人也可以访问
```
 config.vm.define "master" do |master|
      master.vm.network "public_network"
      master.vm.hostname = "master"
  end  
  config.vm.define "slave1" do |slave1|
      slave1.vm.network "public_network"
      slave1.vm.hostname = "slave1"
  end 
  config.vm.define "slave2" do |slave2|
      slave2.vm.network "public_network"
      slave2.vm.hostname = "slave2"
  end 
  config.vm.define "slave3" do |slave3|
      slave3.vm.network "public_network"
      slave3.vm.hostname = "slave3"
  end 
```
 
3.全部启动

`vagrant up`

选择网络的时候选择1, 设置桥接的网卡
 
4.连接到主机

`vagrant ssh master`
 
5.启动redis服务, 并设置值并读取

`sudo su -`

`redis-server /etc/redis/6379.conf`

`redis-cli`

`set name maxwelldu`

`get name`

`exit`

`ifconfig`

192.168.31.128
 
6.添加一个标签, control+t 一个脚本配置从1

`vagrant ssh slave1`

`sudo su -`
 
`redis-server /etc/redis/6379.conf 192.168.31.194 6379`

`redis-cli`

`get name`
 
后面所有的从机都像第6步一样操作即可
 
相关命令和网址:

关闭redis-server

`pkill redis-server`

官网地址

[http://redis.io/](http://redis.io/)

[http://redis.cn/](http://redis.cn/)
