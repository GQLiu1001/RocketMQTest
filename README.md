# 测试rocketmq
## 1.创建容器共享网络
RocketMQ 中有多个服务，需要创建多个容器，创建 docker 网络便于容器间相互通信。  
```
docker network create rocketmq
```
## 2.启动NameServer
```
# docker run NameServer
docker run -d --name rmqnamesrv -p 9876:9876 --network rocketmq apache/rocketmq:5.3.1 sh mqnamesrv

# 验证 NameServer 是否启动成功
docker logs -f rmqnamesrv
```
## 3.启动 Broker+Proxy+Dashboard
### 3.1 broker.conf
```
# nameServer 地址多个用;隔开 默认值null
# 例：127.0.0.1:6666;127.0.0.1:8888 
namesrvAddr=rmqnamesrv:9876
# 集群名称
# brokerClusterName = DefaultCluster
# 节点名称
# brokerName = broker-a
# broker id节点ID， 0 表示 master, 其他的正整数表示 slave，不能小于0 
# brokerId = 0
# Broker服务地址	String	内部使用填内网ip，如果是需要给外部使用填公网ip
brokerIP1 =192.168.5.5
tlsTestModeEnable = false
```
### 3.2 rmq-proxy.json
```json
{
    "rocketMQClusterName": "DefaultCluster",
    "grpcServerPort": 8091,
    "remotingListenPort": 8090
  }
```
### 3.3 docker-run
#### 3.3.1 rmqbroker
```
docker run -d --restart=always --name rmqbroker --network rocketmq --privileged=true -p 10912:10912 -p 10911:10911 -p 10909:10909 -p 8090:8090 -p 8091:8091 -e "NAMESRV_ADDR=rmqnamesrv:9876" -v D:/RocketMQ/broker.conf:/home/rocketmq/rocketmq-5.3.1/conf/broker.conf -v D:/RocketMQ/rmq-proxy.json:/home/rocketmq/rocketmq-5.3.1/conf/rmq-proxy.json apache/rocketmq:5.3.1 sh mqbroker --enable-proxy -c /home/rocketmq/rocketmq-5.3.1/conf/broker.conf
```
#### 3.3.2 dashboard
```
docker run -d --restart=always --name rmq-dashboard -p 8083:8080 --network rocketmq -e "JAVA_OPTS=-Xmx256M -Xms256M -Xmn128M -Drocketmq.namesrv.addr=rmqnamesrv:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false" apacherocketmq/rocketmq-dashboard
```


