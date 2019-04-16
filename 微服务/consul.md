---
title: consul
date: 2019-04-16 14:29:56
tags: 服务发现注册
---



# 一、安装

```shell
cd /home/download
wget https://releases.hashicorp.com/consul/1.4.3/consul_1.4.3_linux_amd64.zip
unzip consul_1.4.3_linux_amd64.zip
mv consul /usr/local/consul/
vim /etc/profile
	## 添加以下配置
	export CONSUL_HOME=/usr/local/consul
	export PATH=$JAVA_HOME/bin:$MAVEN_HOME/bin:$CONSUL_HOME:$PATH
	:wq!
source /etc/profile
## 查看版本
consul -v
	Consul v1.4.3
  Protocol 2 spoken by default, understands 2 to 3 (agent will automatically use protocol >2 when speaking to compatible agents)

## 以开发模式启动consul
## -dev开发模式启动的consul，该模式启动的consul作为配置中心的时候，配置数据是不能保存的，不能进行持久化，需要进行持久化，则需要以服务模式启动。
consul agent -dev -http-port 8500 -client 0.0.0.0 &
### -client 0.0.0.0：表明不是绑定的不是默认的127.0.0.1地址，可以通过公网进行访问
### -http-port 8500：通过该参数可以修改consul启动的http端口

```

[参考网站](https://blog.csdn.net/j903829182/article/details/81257298)

