# SEATA 实战

> 背景：springcloud + nacos + mysql5.7 + seata + openfeign

## 1.概述

### 1.1 四种模式

* ***AT***

* ***TCC***

* ***Saga***

* ***XA***

  > **上次分享会已经讲解了这四种模式**。

### 1.2 三种角色

* ***TC*** (Transaction Coordinator)

  > 事务协调者：维护全局和分支事务的状态，驱动全局事务提交或者回滚。

* ***TM*** (Transaction Manager)

  > 事务管理器：定义全局事务的范围，开始全局事务、提交或回滚全局事务。

* ***RM*** (Resource Manager)

  > 资源管理器：管理分支事务处理器的资源，与 ***TC*** 交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或者回滚。

  ***TC 为单独部署的 Server 服务端，TM 和 RM 为嵌入到应用中的 Client 客户端。***

### 1.3 框架支持情况

​	mybatis、mybatisPlus等等。

## 2. TC Server 部署

### 2.1 下载 Seata 软件包

选择想要的 Seata 版本：[Seata 下载页面](https://github.com/seata/seata/tree/v1.4.1) 。

```shell
# 创建目录
$ mkdir -p cd /Users/apple/Documents/work/junrun-workspace/project/Seata
$ cd /Users/apple/Documents/work/junrun-workspace/project/Seata

# 下载
$ wget https://github.com/seata/seata/releases/download/v1.1.0/seata-server-1.1.0.tar.gz

# 解压
$ tar -zxvf seata-server-1.1.0.tar.gz

# 查看目录
$ cd seata
$ ls -ls
24 -rw-r--r--    1 yunai  staff  11365 May 13  2019 LICENSE
 0 drwxr-xr-x    4 yunai  staff    128 Apr  2 07:46 bin # 执行脚本
 0 drwxr-xr-x    9 yunai  staff    288 Feb 19 23:49 conf # 配置文件
 0 drwxr-xr-x  138 yunai  staff   4416 Apr  2 07:46 lib #  seata-*.jar + 依赖库 
```

### 2.2 存储模式

#### 2.2.1 Seata 数据库

使用 [`seata.sql`](https://github.com/seata/seata/blob/develop/script/server/db/mysql.sql) 脚本，初始化 Seata TC Server 的 db 数据库。

如图所示:

<img src="http://qiniuyun.whitenip.site/image/seata/seataServer%E6%95%B0%E6%8D%AE%E5%BA%93%E8%84%9A%E6%9C%AC.jpg" />

#### 2.2.2 Seata 存储模式

修改 [2.1 下载 Seata 软件包](#2.1 下载 Seata 软件包) 步骤中解压的 seata.tar.gz 文件。在 `conf/file.conf` 配置文件，修改使用 db 数据库，实现 ***Seata TC Server*** 的全局事务会话信息的共享。如下图所示:

<img src="http://qiniuyun.whitenip.site/image/seata/seata%E9%85%8D%E7%BD%AE1.jpg" alt="seata file.conf 配置" />

***非 MySQL8 支持***

### 2.3 注册配置

​		修改 [2.1 下载 Seata 软件包](#2.1 下载 Seata 软件包) 步骤中解压的 seata.tar.gz 文件。在 `conf/registry.conf` 配置文件，设置使用 Nacos 注册中心。

整个 `registry.conf`分为两部分: ***registry***、***config***。

#### 2.3.1  ***registry*** 

如下图所示：

<img src="http://qiniuyun.whitenip.site/image/seata/registry1.jpg" alt="注册registry部分配置" />

**注：serverAddr和namespace与Server端一致**

#### 2.3.2  ***config***

 如下图所示:

<img src="http://qiniuyun.whitenip.site/image/seata/registry2.jpg" alt="注册config部分配置" />

### 2.4 将配置导入到 nacos

**需要了解的内容—— [事务分组](http://seata.io/zh-cn/docs/user/transaction-group.html)。！！！！**

#### 2.4.1 config.txt

***config.txt*** 是参数明细（包含 Server 和 Client）。

高版本的 nacos 没有 config.txt 文件，需要手动下载，[config.txt 下载地址](https://github.com/seata/seata/blob/develop/script/config-center/config.txt)。

接下来我们需要修改 ***config.txt*** 文件，如下所示：

<img src="http://qiniuyun.whitenip.site/image/seata/seata%E9%85%8D%E7%BD%AE2.jpg" alt="seata 在nacos中的配置"  />

**注：① config.txt 和 conf 文件夹 同级目录；② clusterName与Server端cluster一致**。

#### 2.4.2 将 config.txt 导入 nacos

##### 2.4.2.1 下载	

 高版本的 nacos 没有 config.txt 文件，需要手动下载，[nacos-config.sh 下载地址](https://github.com/seata/seata/blob/develop/script/config-center/nacos/nacos-config.sh)。

**注：nacos-config.sh 脚本位于 conf文件夹内**。

##### 脚本目录

执行 nacos-config.sh 即可将这些配置导入到nacos，这样就不需要将file.conf和registry.conf放到我们的项目中了，需要什么配置就直接从nacos中读取。

#####  2.4.2.2 执行脚本

示例：

```shell
sh nacos-config.sh -h 42.193.185.57 -p 8848 -g SEATA_GROUP -t 69b637a2-9cab-4487-b115-b44f43998043 -u nacos -w nacos
```

如下图所示:

<img src="http://qiniuyun.whitenip.site/%E5%AF%BC%E5%85%A5%E9%85%8D%E7%BD%AE%E5%88%B0seata1.jpg" alt="执行命令" />

##### 2.4.2.3 执行结果

<img src="http://qiniuyun.whitenip.site/%E5%AF%BC%E5%85%A5%E9%85%8D%E7%BD%AE%E5%88%B0nacos2.jpg" alt="执行结果" />

显示 `Complete initialization parameters, total-count:82 , failure-count:0 ` 则表示导入成功。

##### 2.4.2.4 nacos 查询结果

如下图所示:

<img src="http://qiniuyun.whitenip.site/seata%E5%9C%A8nacos%E4%B8%AD%E7%9A%84%E9%85%8D%E7%BD%AE.jpg" />

### 2.5 执行业务库 Seata sql 脚本

使用 [`undo.sql`](https://github.com/seata/seata/blob/develop/script/client/at/db/mysql.sql) 脚本，初始化 Seata Client 的 db 数据库。也就是业务库。

如图所示:

<img src="http://qiniuyun.whitenip.site/image/seata/seataClient%E6%95%B0%E6%8D%AE%E5%BA%93%E8%84%9A%E6%9C%AC.jpg" />

### 2.6 添加 logs 文件夹

在 `bin`同级目录下创建 logs

```shell
cd /Users/apple/Documents/work/junrun-workspace/project/Seata/seata/conf

mkdir logs
```

### 2.6 启动 Seata

* 执行 `nohup sh bin/seata-server.sh -p 18091 -n 1 &` 命令，启动**第一个** TC Server 在后台
* `-p`：Seata TC Server 监听的端口。
* `-n`：Server node。在多个 TC Server 时，需区分各自节点，用于生成不同区间的 transactionId 事务编号，以免冲突。

```shell
$ tail -n500 -f nohup.out
```

在 `nohup.out` 文件中，我们看到如下日志，说明启动成功：

<img src="http://qiniuyun.whitenip.site/image/seata/seata%E5%90%AF%E5%8A%A8%E6%97%A5%E5%BF%97.jpg" />

### 2.7 nacos 查看 seata 服务

**服务信息如下所示：**

<img src="http://qiniuyun.whitenip.site/image/seata/seata%E6%9C%8D%E5%8A%A11.jpg" />

**seata 详情信息:**

<img src="http://qiniuyun.whitenip.site/image/seata/seata%E6%9C%8D%E5%8A%A12.jpg" />

### 3. Java 应用

#### 3.1 seata 依赖

```xml
<dependency>
  <groupId>io.seata</groupId>
  <artifactId>seata-spring-boot-starter</artifactId>
  <version>最新版</version>
</dependency>
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
  <version>2.2.1.RELEASE</version>
  <exclusions>
    <exclusion>
      <groupId>io.seata</groupId>
      <artifactId>seata-spring-boot-starter</artifactId>
    </exclusion>
  </exclusions>
</dependency>
```

#### 3.2 seata yml配置

```properties
seata:
  enabled: true
  enable-auto-data-source-proxy: false
  application-id: ${spring.application.name} # Seata 应用编号，默认为 ${spring.application.name}
  tx-service-group: xcy-business-group
  registry:
    type: nacos
    nacos:
      application: seata-server
      server-addr: 42.193.185.57:8848
      namespace: 69b637a2-9cab-4487-b115-b44f43998043
  config:
    type: nacos
    nacos:
      server-addr: 42.193.185.57:8848
      namespace: 69b637a2-9cab-4487-b115-b44f43998043
      group: SEATA_GROUP
  service:
    vgroup-mapping:
      xcy-business-group: default
    disable-global-transaction: false
  client:
    rm:
      async-commit-buffer-limit: 1000
      report-retry-count: 5
      table-meta-check-enable: false
      report-success-enable: false
      lock:
        retry-interval: 10
        retry-times: 2
        retry-policy-branch-rollback-on-conflict: true
    tm:
      commit-retry-count: 5
      rollback-retry-count: 5
    undo:
      data-validation: true
      log-serialization: jackson
      log-table: undo_log
    log:
      exceptionRate: 100
```

#### 3.3 演示视频

#### 3.3.1事务不回滚

<iframe height=498 width=510 src="http://qiniuyun.whitenip.site/%E5%88%86%E5%B8%83%E5%BC%8F%E5%9C%BA%E6%99%AF-%E6%9C%AC%E5%9C%B0%E4%BA%8B%E5%8A%A1.mp4">




#### 3.3.2 引入SeaTac事务回滚

<iframe height=498 width=510 src="http://qiniuyun.whitenip.site/%E5%88%86%E5%B8%83%E5%BC%8F%E5%9C%BA%E6%99%AF-%E6%9C%AC%E5%9C%B0%E4%BA%8B%E5%8A%A1.mp4">

