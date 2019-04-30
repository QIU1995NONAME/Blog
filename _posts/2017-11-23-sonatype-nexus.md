---
layout: docs
title:  "Linux 借助 Docker 搭建 Sonatype Nexus"
date:   2017-11-23
categories: Linux Docker
enable_mathjax: false
enable_mermaid: false
enable_comments: true
---
## 首先需要安装与配置 Docker (略)
## 获取 Sonatype Nexus 的镜像 (重点仅仅是 Image 的路径)
```bash
$ docker pull docker.io/sonatype/nexus
```
## 运行 Nexus 之前的准备工作
### 调查创建容器所需要的信息
首先获取这个镜像的详细信息。
```bash
$ docker inspect docker.io/sonatype/nexus | less
```
需要关注的点有：运行的用户ID，开放的端口，Volumes。从上述命令返回的信息中提取到：
```
......
            "User": "nexus",
......
            "ExposedPorts": {
                "8081/tcp": {}
            },
......
            "Volumes": {
                "/sonatype-work": {}
            },
......
```
在这里有一个问题，容器中的用户名对应的用户ID和主机中的并不一定相同，所以需要调查一下这个用户名对应的ID到底是什么。获取之后，删除测试用的容器。
```bash
(host)   $ docker run -it docker.io/sonatype/nexus /bin/bash
(docker) $ id
uid=200(nexus) gid=200(nexus) groups=200(nexus)
(docker) $ exit
(host)   $ docker ps --all
(host)   $ docker rm <test container>
```

### 创建容器运行的环境。
到目前位置为止，我们获取到了创建容器所必要的信息：

+ UID : 200
+ Ports : 8081/tcp
+ Volumes : "/sonatype-work"

准备一个目录，用来映射到 "/sonatype-work" 。假定这个目录为${DOCKER_NEXUS_DATADIR}
``` bash
$ # 创建这个目录
$ sudo mkdir -p ${DOCKER_NEXUS_DATADIR}
$ # 调整这个目录的所有者
$ sudo chown 200:200 ${DOCKER_NEXUS_DATADIR} -R
$ # 调整这个目录的 SELinux TYPE
$ sudo chcon -t svirt_sandbox_file_t ${DOCKER_NEXUS_DATADIR} -R
$ # 更改防火墙设置，如果是本机安装，可以在 Nexus 配置结束之后再对防火墙进行设置。
$ sudo firewall-cmd --add-port 8081/tcp
```

## 启动 Nexus 并作出必要的配置
### 配置并启动容器
使用以下命令创建并启动容器。参数可以自定义。
```
$ docker run --detach                                     \
>         --hostname localhost                            \
>         --publish 8081:8081                             \
>         --name sonatype-nexus                           \
>         --volume ${DOCKER_NEXUS_DATADIR}:/sonatype-work \
>         docker.io/sonatype/nexus:latest
```
### 配置管理员用户
运行之后立刻登陆并更改管理员账户 ( 初始账户密码：“admin”/“admin123” )
建议使用 admin 创建新的管理员账户 ( Nexus Administrator Role )，然后用新的管理员用户登陆并删除所有默认的账户。如果有需要的话可以保留 anonymous 用户。

### 配置 Maven Central 代理
这个代理，Nexus默认就有。建议修改其配置:

| Item                   |   Value   |
| ---------------------- |:---------:|
| Download Remote Indexes|   True    |
| Auto Blocking Enabled  |   False   |


### 配置 JCenter 代理
创建一个新的 Proxy 仓库。并修改配置：

| Item                     |   Value                      |
| ------------------------ |:----------------------------:|
| Repository ID            | JCenter                      |
| Repository Name          | JCenter                      |
| Remote Storage Location  | https://jcenter.bintray.com/ |
| Auto Blocking Enabled    | False                        |

配置完成之后，将仓库添加到 Public Repositories 中。
