# WebServer
用C++实现的高性能WEB服务器，经过webbenchh压力测试可以实现上万的QPS

## 功能
* 利用IO复用技术Epoll与线程池实现多线程的Reactor高并发模型；
* 利用正则与状态机解析HTTP请求报文，实现处理静态资源的请求；
* 利用标准库容器封装char，实现自动增长的缓冲区；
* 基于小根堆实现的定时器，关闭超时的非活动连接；
* 利用单例模式与阻塞队列实现异步的日志系统，记录服务器运行状态；
* 利用RAII机制实现了数据库连接池，减少数据库连接建立与关闭的开销，同时实现了用户注册登录功能。
* 增加logsys,threadpool测试单元(todo: timer, sqlconnpool, httprequest, httpresponse) 

## 环境要求
* Linux
* C++14
* MySql 5.7
* Kubernetes
* Docker

## 目录树
```
.
├── code           源代码
│   ├── buffer
│   ├── config
│   ├── http
│   ├── log
│   ├── timer
│   ├── pool
│   ├── server
│   └── main.cpp
├── test           单元测试
│   ├── Makefile
│   └── test.cpp
├── resources      静态资源
│   ├── index.html
│   ├── image
│   ├── video
│   ├── js
│   └── css
├── bin            可执行文件
│   └── server
├── log            日志文件
├── webbench-1.5   压力测试
├── build          
│   └── Makefile
├── Makefile
├── LICENSE
└── readme.md
```


## 项目启动
需要先配置好对应的数据库
```bash
// 建立yourdb库
create database yourdb;

// 创建user表
USE yourdb;
CREATE TABLE user(
    username char(50) NULL,
    password char(50) NULL
)ENGINE=InnoDB;

// 添加数据
INSERT INTO user(username, password) VALUES('name', 'password');
```

```bash
make
./bin/server
```

## 单元测试
```bash
cd test
make
./test
```

## 压力测试
```bash
cd webbench-1.5
make
./webbench-1.5/webbench -c 10000 -t 10 http://ip:port/
```

## k8s部署
### 部署mysql
```bash
kubectl apply -f mysql-rc.yaml
kubectl apply -f mysql-service.yaml
```
部署完mysql之后，在集群中查看mysql的ip地址，将本项目mysql的ip改为以上
### 部署webserver镜像
将Dockerfile复制到上级目录，执行
`docker build -t webserver:latest .`

得到镜像后，将镜像部署到私有仓库里，详见https://yeasy.gitbook.io/docker_practice/repository/registry

部署到k8s，执行
```bash
kubectl apply -f webserver-rs.yaml
kubectl apply -f webserver-service.yaml
```

`webserver-service.yaml`里设置了集群外部访问，端口为`32000`，输入`http://集群外部地址:32000`即可访问webserver

## 集群压测
配置: 1台master节点(不参与负载)，两台node节点均为2G 2核心
- 一个node
	- 一个pod为 6743
	- 两个pod为 8069
	- 三个pod为 8786
	- 四个pod为 9830
- 两个node
	- 两个pod为 12396
	- 三个pod为 11676
	- 四个pod为 13065
	- 六个pod为 15973
