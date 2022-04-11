```shell
由于springboot兼容性问题，只能选择6.4.2版本
下载
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.4.2.tar.gz
解压缩
tar zxvf elasticsearch-6.4.2.tar.gz
创建数据文件目录
mkdir /data/elasticsearch/data
创建日志文件目录
mkdir /data/elasticsearch/logs
修改配置文件 $elasticsearch_path/config/elasticsearch.yml
cluster.name: portsafe 
node.name: node-192  # 将IP地址的末位
node.attr.rack: r1 # attr不同其余两个节点分别为r2 和r3
network.host: 0.0.0.0 # 绑定IP地址
discovery.zen.ping.unicast.hosts: ["172.17.64.192"]   # es集群的节点 A->B B->C C->A
discovery.zen.minimum_master_nodes: 2 # es集群最小数量，小于它启动不成功
添加用户
adduser elasticsearch
修改 elasticsearch 目录所属用户
chown -R elasticsearch /usr/local/elasticsearch-6.4.2
chown -R elasticsearch /data/elasticsearch/

出现max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]问题
用户修改否则有权限问题
vi /etc/sysctl.conf  最后加入
更改linux禁用swapping，添加或修改
vm.swappiness = 0
更改linux一个进行能拥有的最多的内存区域要求 256M
vm.max_map_count = 262144
使sysctl.conf 生效
/sbin/sysctl -p

出现max file descriptors [65535] for elasticsearch process is too low, increase to at least [65536]问题
vi /etc/security/limits.conf 最后加入
elasticsearch soft nofile 65536
elasticsearch hard nofile 65536

出现max number of threads [1024] for user [elasticsearch] is too low, increase to at least [4096]问题
vi /etc/security/limits.d/90-nproc.conf 
*          soft    nproc     4096 

出现system call filters failed to install; check the logs and fix your configuration or disable system call filters at your own risk问题
原因：
这是在因为Centos6不支持SecComp，而ES5.2.0默认bootstrap.system_call_filter为true进行检测，所以导致检测失败，失败后直接导致ES不能启动。
解决：
在elasticsearch.yml中配置bootstrap.system_call_filter为false，注意要在Memory下面:
bootstrap.memory_lock: false
bootstrap.system_call_filter: false

配置网络
网络端口配置：
iptables -I INPUT -p tcp --dport 9300 -j ACCEPT
/etc/rc.d/init.d/iptables save
/etc/init.d/iptables status
查看网络端口状态

切换到elasticsearch用户
su elasticsearch

./elasticsearch -d # -d后台启动，否则前台启动，至此启动成功

设置自启动
启动脚本
#!/bin/sh
#chkconfig: 2345 80 05
#description:start or stop elasticsearch

export JAVA_HOME=/usr/local/jdk1.8.0_221
export JAVA_BIN=/usr/local/jdk1.8.0_221/bin
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export JAVA_HOME JAVA_BIN PATH CLASSPATH

case "$1" in
start)
    su elasticsearch<<!
    cd /usr/local/elasticsearch-6.4.2
    ./bin/elasticsearch -d
!
    echo "elasticsearch startup"
    ;;
stop)
    es_pid=`ps aux|grep elasticsearch | grep -v 'grep elasticsearch' | awk '{print $2}'`
    kill -9 $es_pid
    echo "elasticsearch stopped"
    ;;
restart)
    ${0} stop
    ${0} start
    ;;
*)
    echo "start|stop|restart"
    ;;
esac

exit $?
增加权限
chmod +x elasticsearch
设置自启动
chkconfig elasticsearch on

下载可视化工具kibana
wget https://artifacts.elastic.co/downloads/kibana/kibana-6.4.2-linux-x86_64.tar.gz

出现Error registering Kibana Privileges with Elasticsearch for kibana-.kibana: [index_not_found_exception] no such index, with问题

只能使用相匹配的版本进行管理
配置
使用外部用户可以进行访问
server.host: "0.0.0.0"
没有设置登录认证

http://172.17.64.193:5601/

在有数据后需要创建下index pattern

配置开机自启
#!/bin/bash
# chkconfig:   2345 98  02
# description: for kibana start auto

KIBANA_HOME=/usr/local/kibana-6.4.2-linux-x86_64
case $1 in
        start)
                nohup $KIBANA_HOME/bin/kibana >/data/elasticsearch/kibana/kibana.log 2>1 &
                echo "kibana started"
                ;;
        stop)
                pid=`ps aux|grep kibana | grep -v 'grep kibana' | awk '{print $2}'`
                kill -9 $pid
                echo "kibana stopped"
                ;;
        *) echo "start|stop";;
esac

查看自启动项
chkconfig --list
应该最少是有redis、elasticsearch,193上还应该有kibana


```

