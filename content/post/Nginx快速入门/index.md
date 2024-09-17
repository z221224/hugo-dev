+++
title = 'Nginx快速入门'
date = 2024-08-26T11:49:49+08:00
draft = true
+++



## Nginx快速入门

### Nginx安装

根据前文的部署方案，需要在`server02`部署Nginx。Nginx官网有详细的安装步骤，具体内容可参考[官方文档](https://nginx.org/en/linux_packages.html)。

#### 1. 配置Nginx yum存储库

创建`/etc/yum.repos.d/nginx.repo`文件

vim /etc/yum.repos.d/nginx.repo

增加如下内容

```plain
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```

#### 2. 在线安装Nginx

执行以下命令，安装Nginx

```
yum -y install nginx
```

#### 3. 启动Nginx

执行以下命令启动Nginx

```
systemctl start nginx
```

执行以下命令查看Nginx运行状态

```
systemctl status nginx
```

执行以下命令设置开机自启

```
systemctl enable nginx
```



#### 4. 访问Nginx服务默认首页

访问`http://192.168.10.102`，能访问到如下页面，则证明Nginx运行正常。![img](https://cdn.nlark.com/yuque/0/2024/png/35472382/1724643157938-25736760-9ee2-44a4-941f-7de088f04336.png)

### 重要的目录、文件

Nginx中有很多十分重要的目录或者文件，下面对核心内容进行介绍

#### 1.配置文件相关

- - `/etc/nginx/`：主要的Nginx配置文件目录。
  - `/etc/nginx/nginx.conf`：Nginx的主配置文件，包含全局配置信息。

#### 2.日志相关

- - `/var/log/nginx/`：Nginx的日志文件目录，包括访问日志和错误日志。
  - `/var/log/nginx/access.log`：访问日志，记录所有进入服务器的请求。
  - `/var/log/nginx/error.log`：错误日志，记录服务器处理过程中的错误信息。

### 配置文件概述

#### 1.配置文件结构

`nginx.conf`文件层次分明，整个文件分为多个区块（block），每个区块下可配置各种参数，也可包含其子级区块，具体结构如下图所示

![img](https://cdn.nlark.com/yuque/0/2024/png/35472382/1724643237074-13a025e6-e759-4d9d-869d-b7939cebf1e1.png)

#### 2.重要配置说明

下面分块介绍重要的配置参数

- **main block**

`main block`位于配置文件的最外层，其包含了影响Nginx服务器整体行为的全局参数，例如

- - `user`：定义Nginx工作进程的用户和用户组。
  - `worker_processes`：指定Nginx使用的工作进程数。
  - `error_log`：配置全局错误日志文件路径。

- **events block**

`events block`位于`main block`中，用于配置Nginx服务器的事件处理机制，主要配置Nginx如何处理客户端连接。

- **http block**

`http block`位于`main block`中，用于配置HTTP服务器相关功能。例如

- - `access_log`：指定访问日志的路径
  - `log_format`：指定访问日志的格式

- **server block**

`server block`位于`http block`，用于配置虚拟主机，一个Nginx服务可包含多个虚拟主机，每个虚拟主机都可以独立的提供服务，因此借助Nginx，我们可以在一台服务器部署多个独立的网站，如下图所示

![img](https://cdn.nlark.com/yuque/0/2024/png/35472382/1724643473652-37b90c9a-3555-4ec3-8796-b4bd91f734db.png)

- 每个虚拟主机使用一个`server block`进行配置，配置的内容包括

- - `listen`：虚拟主机监听的端口号。
  - `server_name`：指定虚拟主机的域名或者IP。

- **location block**

`location block`位于`server block`，用于配置请求的处理逻辑，一个`server block`中可以包含多个`location block`，例如：

```java
server {
    listen 80;
    server_name www.atguigu.com;
    location /index {
        root /var/www/html;
    }

    location /api {
        proxy_pass http://backend-api;
    }
}
```

### 静态资源服务器案例

下面完成一个简单案例，使用Nginx作为静态资源服务器。

项目资料中有一个简单的前端项目`hello-nginx`，其中只包含html、css等静态资源，现将其部署在`server02`上。

#### 1. 上传静态资源到服务器

将`hello-nginx.zip`上传到`server02`服务器任意路径。

#### 2. 解压`hello-nginx.zip`到`/usr/share/nginx/html`中

```
unzip hello-nginx.zip -d /usr/share/nginx/html
```

最终的路径结构如下：

```java
/usr
└── share
    └── nginx
        └── html
            └── hello-nginx
                ├── css
                │   └── style.css
                ├── images
                │   └── img.png
                └── index.html
```

#### 3. 配置Nginx虚拟主机

虚拟主机的配置应位于`/etc/nginx/nginx.conf`的**server block**中，由于`/etc/nginx/nginx.conf`的**http bolck**中引入了`/etc/nginx/conf.d/*.conf`，所以虚拟主机在`/etc/nginx/conf.d/`目录下的任意`.conf`文件配置即可。

创建`/etc/nginx/conf.d/hello-nginx.conf`文件

```
vim /etc/nginx/conf.d/hello-nginx.conf
```

添加如下内容

```java
server {
    listen       8080;
    server_name  192.168.10.102;

    location /hello-nginx {
        root   /usr/share/nginx/html;
        index  index.html;
    }
}
```

- 重新加载Nginx的配置文件

```
systemctl reload nginx
```



#### 4. 访问项目

访问路径为http://192.168.10.102:8080/hello-nginx，若部署成功，可见到如下页面：
 ![img](https://cdn.nlark.com/yuque/0/2024/png/35472382/1724643753290-2dd1aa39-d9f2-4714-9d4d-27bdaecb0add.png)

#### 5. 案例剖析

下面通过上述案例来了解Ngxin处理请求的逻辑。

###### 5.1. 匹配server

由于Nginx中可存在多个虚拟主机的配置，故接收到一个请求后，Nginx首先要确定请求交给哪个虚拟主机进行处理。这很显然是根据`server_name`和`listen`进行判断的。

例如上述的请求路径http://192.168.10.102:8080/hello-nginx，就会匹配到以下的虚拟主机

```plain
server {
    listen       8080;
    server_name  192.168.10.102;
    ......
}
```

###### 5.2. 匹配location

- 由于一个**server block**中可能包含多个**location block**，故Nginx在完成**server**匹配后，还要匹配**location**。
- **location**的匹配是根据请求路径进行判断的。

例如以下写法`location`关键字后边的`/hello-nginx`就是匹配规则，

它表达的含义是匹配以`/hello-nginx`为前缀的请求，例如上述的http://192.168.10.102:8080/hello-nginx请求就会匹配到该**location**，而http://192.168.10.102:8080/nginx则不会。

```plain
location /hello-nginx {
    ......
}
```

###### 5.3. 定位文件

完成**location**的匹配后，Nginx会以**location block**中的`root`作为根目录，然后查找请求路径对应的资源，例如以下配置

```plain
location /hello-nginx {
    #路径
    root   /usr/share/nginx/html;
    #路径下的文件
    index  index.html;
}
```

**当请求http://192.168.10.102:8080/hello-nginx 时，Ngxin会在**`**/usr/share/nginx/html/hello-nginx**`**路径中查找资源，由于该路径为目录（而非文件），故Nginx会在该目录下寻找**`**index**`**，也就是上述配置的**`**index.html**`**。然后将**`**index.html**`**响应给客户端。至此，该请求的处理就结束了。**

**注意**：上述提到的**server_name**和**location**均有多种匹配模式，例如精确匹配、前缀匹配、正则匹配，此处不再展开。
