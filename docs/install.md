# simplecfs部署

## 1.文档历史记录
文档版本 | 描述	| 编写人员 | 更新时间
----    | ---   | ---    | --- 
0.1	| 开始创建 | 李剑	| 2015-03-26

## 2.系统介绍
**simplecfs**是一个支持纠删码的分布式文件系统，使用类*hadoop*的三方架构，分为*mds*, *ds*,和*client*。系统使用`python 2.7`开发，使用库方式调用编码库；系统依赖`redis`作为元数据的存储，依赖`eventlet`作为网络库，作为网络监听和网络传输，依赖`nose`作自动化测试；编码库有`gf-complete`和自己开发的`librlc`。理论上系统可以在多种linux平台下运行。

## 3.机房介绍
小机房，使用代理。

代理IP | 代理端口说明 | 用户信息 
--- | ---- | -----
^_^ | ^_^ | 无

ssh配置方法：

	$ cat ~/.ssh/config

	> Host 192.168.3.*
	> ProxyCommand nc -X connect -x 115.156.209.252:9001 %h %p

	$ ssh f309@192.168.3.121

hostname |机器IP | 端口说明 | 用户信息 | 功能
--- | ---- | ----- | ----- | -----
de01 | 192.168.3.121 | 22 | user:f309 | mds/client
de05 | 192.168.3.125 | 22 | user:f309 | ds
de10 | 192.168.3.130 | 22 | user:f309 | ds
de17 | 192.168.3.137 | 22 | user:f309 | ds
de19 | 192.168.3.139 | 22 | user:f309 | ds
de20 | 192.168.3.140 | 22 | user:f309 | ds

## 4.安装步骤

### 4.1.获取压缩包

	# 使用无线，在内网中获取源代码
	git clone --recursive http://192.168.0.19/hustlijian/simplecfs.git
	tar czf simplecfs.tar.gz simplecfs/

### 4.2.安装依赖

	# 解压源代码，安装python依赖	
	tar xzf simplecfs.tar.gz 
	cd simplecfs
	sudo pip install -r requirements.txt

### 4.3.编译

	make
	
### 4.4.测试

	make test

### 4.5.运行

#### 4.5.1 MDS

0.安装依赖

	cd simplecfs
	sudo pip install -r requirements.txt

1.编译

	make
	
2.修改配置文件

	vim conf/mds.cfg
	
	>[mds]
	>mds_ip=192.168.3.121
	>mds_port=8000
	>...
	>log_level=WARNING
	
3.启动redis-server

	cd ext/redis/src/
	./redis-server &

4.启动mds

	cd simplecfs
	./mds.py


#### 4.5.2 DS

1.安装依赖

	cd simplecfs
	sudo pip install -r requirements.txt
	
2.修改配置文件

	vim conf/ds.cfg
	>ds_ip=192.168.3.125  # 不同ds修改成不同的ip
	>ds_port=7000
	>rack_id=1

	>[mds]
	>mds_ip=192.168.3.121
	>mds_port=8000
	>...
	>log_level=WARNING

3.启动ds

	cd simplecfs
	./ds.py
	
#### 4.5.3 client

1.安装依赖

	cd simplecfs
	sudo pip install -r requirements.txt
	
2.修改配置文件

	vim conf/client.cfg

	>[mds]
	>mds_ip=192.168.3.121
	>mds_port=8000
	>...
	>log_level=WARNING
	>[file]
	>packet_size=512     ;multiples of 256(byte)
	>block_size=1024     ;multiples of 256(byte), block_size>=packet_size

	>[thread]
	>thread_num=1000     ;多线程读写时线程(伪线程:协程)的数量


3.使用client测试文件

	cd simplecfs
	./rs_test.py   # 可以是其它的测试功能，client以api方式使用，使用类封装

#### 4.5.4 查看日志

1.mds日志

	cd simplecfs/log
	cat mds.log
	
2.ds日志

	cd simplecfs/log
	cat ds.log

3.client日志

	cd simplecfs/log
	cat client.log
	
#### 4.5.5 清空数据

1.清空mds元数据

	cd ext/redis/src
	rm dump.rdb   # 删除redis元数据，再重启redis-server就好了

2.清空ds数据

	cd simplecfs
	rm storage/*

3.清空日志数据

	cd simplecfs
	rm log/*
	
## 5.其它

1.安装pip

     wget https://bootstrap.pypa.io/get-pip.py
     sudo python get-pip.py
    
2.建立新用户

	useradd f309
	passwd f309
     
3.安装zsh

	sudo apt-get install zsh # yum -y install zsh
	chsh -s /bin/zsh f309  # f309是用户名
	git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
	cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
	git clone git://github.com/joelthelion/autojump.git
	cd autojump
	./install.py
