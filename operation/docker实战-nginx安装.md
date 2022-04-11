# 通过 Docker 安装 Nginx

## 环境

* 阿里云ESC服务器-CentOS 8.0

## 准备

```shell
03.  -d, --detach=false         指定容器运行于前台还是后台，默认为false     
04.  -i, --interactive=false    打开STDIN，用于控制台交互    
05.  -t, --tty=false            分配tty设备，该可以支持终端登录，默认为false    
06.  -u, --user=""              指定容器的用户    
07.  -a, --attach=[]            登录容器（必须是以docker run -d启动的容器）  
08.  -w, --workdir=""           指定容器的工作目录   
09.  -c, --cpu-shares=0         设置容器CPU权重，在CPU共享场景使用    
10.  -e, --env=[]               指定环境变量，容器中可以使用该环境变量    
11.  -m, --memory=""            指定容器的内存上限    
12.  -P, --publish-all=false    端口映射，格式为：主机(宿主)端口:容器端口   
13.  -p, --publish=[]           指定容器暴露的端口   
14.  -h, --hostname=""          指定容器的主机名    
15.  -v, --volume=[]            给容器挂载存储卷，挂载到容器的某个目录    
16.  --volumes-from=[]          给容器挂载其他容器上的卷，挂载到容器的某个目录  
17.  --cap-add=[]               添加权限，权限清单详见：http://linux.die.net/man/7/capabilities    
18.  --cap-drop=[]              删除权限，权限清单详见：http://linux.die.net/man/7/capabilities    
19.  --cidfile=""               运行容器后，在指定文件中写入容器PID值，一种典型的监控系统用法    
20.  --cpuset=""                设置容器可以使用哪些CPU，此参数可以用来容器独占CPU    
21.  --device=[]                添加主机设备给容器，相当于设备直通    
22.  --dns=[]                   指定容器的dns服务器    
23.  --dns-search=[]            指定容器的dns搜索域名，写入到容器的/etc/resolv.conf文件    
24.  --entrypoint=""            覆盖image的入口点    
25.  --env-file=[]              指定环境变量文件，文件格式为每行一个环境变量    
26.  --expose=[]                指定容器暴露的端口，即修改镜像的暴露端口    
27.  --link=[]                  指定容器间的关联，使用其他容器的IP、env等信息    
28.  --lxc-conf=[]              指定容器的配置文件，只有在指定--exec-driver=lxc时使用    
29.  --name=""                  指定容器名字，后续可以通过名字进行容器管理，links特性需要使用名字    
30.  --net="bridge"             容器网络设置:  
31.                                bridge 使用docker daemon指定的网桥       
32.                                host    //容器使用主机的网络    
33.                                container:NAME_or_ID  >//使用其他容器的网路，共享IP和PORT等网络资源    
34.                                none 容器使用自己的网络（类似--net=bridge），但是不进行配置   
35.  --privileged=false         指定容器是否为特权容器，特权容器拥有所有的capabilities    
36.  --restart="no"             指定容器停止后的重启策略:  
37.                                no：容器退出时不重启    
38.                                on-failure：容器故障退出（返回值非零）时重启   
39.                                always：容器退出时总是重启    
40.  --rm=false                 指定容器停止后自动删除容器(不支持以docker run -d启动的容器)    
41.  --sig-proxy=true           设置由代理接受并处理信号，但是SIGCHLD、SIGSTOP和SIGKILL不能被代理  
```

> 记录一下，问了公司的运维大佬，有关于主机和容器之间的映射关系都是 ------->  主机:容器

## 开始

### 查找 docker 镜像

```shell
docker search nginx
```

执行上面的指令，得到如下结果：

<img src="http://qiniuyun.whitenip.site/image/blog/operation/nginx-search.png" alt="avatar" style="zoom:80%;" />

### 拉取 docker 镜像

```shell
docker pull nginx
```

执行上面的指令，得到如下结果：

![avatar](http://qiniuyun.whitenip.site/image/blog/operation/nginx-pull.png)

这里截图稍微快了点，继续等待即可完成 ***nginx*** 最新镜像的拉取。

### 运行 docker 镜像

#### 查看镜像

首先执行 ***docker images*** 命令查询你的 *docker* 下有哪些镜像。

执行结果如下：

![avatar](http://qiniuyun.whitenip.site/image/blog/operation/nginx-images.png)

从上面得知我的 *docker* 中的 *nginx* 镜像名称为 *nginx*。

#### 创建 nginx 容器

首先执行 ***docker ps***  或者 ***docker ps -a*** 查看有哪些容器，前者是正在运行的容器，后者是包括已经停止运行的容器。

```shell
// 停止
docker stop Name或者ID  
// 启动
docker start Name或者ID  
// 杀死
docker kill Name或者ID  
// 重启
docker restart name或者ID
// 移除
docker rm name获取ID
```

查看更多的 *docker* 指令：*docker --help*

#### 启动 nginx 镜像

##### 方式一

```
docker run -d nginx:latest
```

*-d* 为后台运行

<img src="http://qiniuyun.whitenip.site/image/blog/operation/nginx-run1.png" alt="avatar" style="zoom:80%;" />

##### 方式二

```shell
docker run --name=mynginx -d nginx:latest
```

*-name* 的意思为 *nginx* 启动之后的容器取别名

<img src="http://qiniuyun.whitenip.site/image/blog/operation/nginx-run2.png" alt="avatar" style="zoom:80%;" />

##### 方式三

> 将 *nginx* 内部的配置文件挂载到宿主机器

这里就是一些更详细的自定义容器的配置：

* 简单创建 *nginx* 容器

* 拷贝容器的文件到宿主机器

  * 在我们创建 *nginx* 容器的时候，在自动生成相应的文件夹及配置，我们需要将默认配置拷贝到宿主机器上来完成文件夹的映射：

  ```shell
  $ /etc/nginx/nginx.conf //nginx的默认加载配置文件
  $ /var/log/nginx //日志目录
  $ /usr/share/nginx/html //首页
  $ /etc/nginx/conf.d/default.conf //默认配置
  这上面四个文件及文件夹就是 nginx 容器自动生成的。
  ```

  * 拷贝

    根据 `docker ps` 查询容器，再执行下面的命令

    ```shell
    docker cp 5a68b00da060:/etc/nginx/nginx.conf /data/nginx/conf
    docker cp 5a68b00da060:/etc/nginx/conf.d/default.conf  /data/nginx/conf
    docker cp 5a68b00da060:/usr/share/nginx/html/index.html  /data/nginx/html
    docker cp 5a68b00da060:/usr/share/nginx/html/50x.html  /data/nginx/html
    docker cp 5a68b00da060:/var/log/nginx /data/nginx/log
    ```

* 删除上面创建的 *nginx* 容器

  ```shell
  docker stop 5a68b00da060
  docker rm 5a68b00da060
  ```

* 重新创建 *nginx* 容器

```shell
docker run -d --name=mynginx -p 80:80 \
-v /data/nginx/nginx.conf:/etc/nginx/nginx.conf \
-v/data/nginx/log:/var/log/nginx \
-v /data/nginx/html:/usr/share/nginx/html \
-v /data/nginx/conf:/etc/nginx/conf.d \
--restart="always" \
nginx:latest
```

***建议：***以上指令放到编辑器格式化之后再执行，防止出现 [docker : invalid reference format](https://stackoverflow.com/questions/45682010/docker-invalid-reference-format) 这个错误。

#### 成功画面

![avatar](http://qiniuyun.whitenip.site/image/blog/nginx/nginx-sucess.png)

*nginx 安装完结撒花，敬请期待 nginx 实战篇*。

​																								

​																																				—— *世上最重要的事，不在于我们在何处，而在于我们朝着什么方向走。*



