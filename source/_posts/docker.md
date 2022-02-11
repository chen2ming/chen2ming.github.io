---
title: docker
tag: docker
date: 2022-1-23
---
docke常用命令：

| 命令 | 意义 |
| -- | -- |
| docker pull <镜像名称> | 获取镜像 |
| docker run -it <镜像名称> /bin/bash | 启动容器 |
| exit | 退出终端 |
| docker ps -a | 查看所有的容器 |
| docker rm -f <容器id> | 删除容器 |
| docker start | docker start b750bbbcfd88 |
| docker run -itd --name ubuntu-test ubuntu /bin/bash | 后台运行 |
| docker stop <容器 ID> | 停止一个容器 |
| docker restart <容器 ID> | 重启一个容器 |
| docker attach <容器 ID> | 退出容器（会停止）|
| docker exec -it <容器 ID> /bin/bash | 进入出容器（不会停止）|
| exit | 退出容器 |
| docker export <容器 ID> > <容器名称>.tar | 导出容器 |
| cat docker/ubuntu.tar | docker import - test/ubuntu:v1 | 导入容器 |
| docker import http://example.com/exampleimage.tgz example/imagerepo | 通过地址导入 |
| docker port  <容器id或者名称> | 查看容器端口的映射情况 |
| docker logs -f bf08b7f2cd89 | docker logs [ID或者名字] 可以查看容器内部的标准输出 |
| docker top <容器名称> | 查看容器内部运行的进程 |
| docker search httpd | 查找镜像 |
| docker rmi hello-world | 删除镜像 |
| docker images | 查看当前的镜像列表 |
| docker network create <名称> | 创建一个网络|
| docker network ls | 查看网络 |
| docker network rm <名称> | 断开和移除网络|
| docker network inspect <名称> | 查看网络信息 |
| docker cp /www/runoob 96f7f14e99ab:/www/| 将主机/www/runoob目录拷贝到容器96f7f14e99ab的/www目录下 |

# 使用nginx
江橙的笔记：
>首先在你的服务器上面创建一个工作目录 例如以下创建的是目录为 dockerData
systemctl start docker //启动docker
systemctl enable docker //设置为开机启动
docker version 验证安装是否成功(有client和service两部分表示docker安装启动都成功了
docker network create my_net 创建一个网络

>docker工作区和宿主机目录挂载了之后，修改的文件会进行同步，一般来说只需要修改宿主机的文件就行了，不需要修改工作区
修改宿主机的文件需要重启容器才会同步过去，修改工作区(容器)会立即同步到宿主机里面
删除容器不会删除宿主机的挂载目录的数据

>1、拉取镜像 去dockerhub拉取合适的版本即可 docker pull nginx:1.20.1-alpine
2、创建配置目录和项目目录 -v /dockerData/nginx:/etc/nginx \
3、跑不带映射的容器 docker run -itd --name nginx 7f18bdc92ca5(镜像id)
4、docker cp nginx:/etc/nginx/ /dockerData/nginx/ | docker exec -it nginx sh(目录为容器的目录)
5、删除容器 docker rm -f ngxin(指的是容器名) 因为删除容器是不会删除宿主机的文件的，但是更新容器里面的挂载目录文件时，会更新宿主机文件，更新宿主机文件也会更新容器文件(需重启容器)
```
# /dockerData/nginx/www:/usr/share/nginx/html \ 挂载目录地址
# /dockerData/nginx:/etc/nginx \  挂载nginx配置地址
# /dockerData/nginx/log:var/log/nginx \  挂载log日志地址

docker run -itd --name nginx -p 80:80 -p 443:443 \
 -v /dockerData/nginx/www:/usr/share/nginx/html \
 -v /dockerData/nginx:/etc/nginx \
 -v /dockerData/nginx/log:/var/log/nginx \
 --network my_net --network-alias nginx 7f18bdc92ca5
```

>配置nginx以后需要重新启动
docker stop nginx

