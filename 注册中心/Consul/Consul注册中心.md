# Consul介绍

**Consul**是HashiCorp公司推出的开源工具，用于实现分布式系统的**服务发现**与**配置**；一种分布式、高可用、支持水平扩展的服务注册与发现的注册中心，[官方文档](https://www.consul.io/intro/index.html)中说明它包括了如下的功能：

- **服务发现、注册**：Consul通过DNS、HTTP接口支持服务注册和服务发现。
- **健康检查**：健康检查使consul可以快速告警在集群的操作。和服务发现集成，可以防止服务调用至发生故障的服务上；
- **KV存储**：一个用来存储动态配置的系统。
- **多数据中心**（Multi Data Center）：支持多数据中心来避免单点故障。zookeeper和etcd均不支持多数据中心功能；
- **一致性算法：**采用Raft一致性协议算法。使用Gossip协议管理成员和广播消息；
- **服务管理Dashboard：**提供一个Web UI的服务注册于健康状态监控的管理页面；

# Consul 架构

下图为Consul[官方文档](https://www.consul.io/docs/internals/architecture.html)提供的架构设计图：

![](https://www.consul.io/assets/images/consul-arch-420ce04a.png)

上图中包含了两个Consul数据中心，每个数据中心都是一个Consul集群，在数据中心1中可以看到Consul的集群是由N个SERVER，加上M个CLIENT组成的。而不管是SERVER还是CLIENT都是集群中的节点。所有的服务都注册在这些节点上，正是通过这些节点实现服务注册信息的共享。

**CLIENT**

CLIENT表示Consul的CLIENT模式，及客户端模式。是Consul节点的一种模式，在这种模式下，所有注册到该节点的服务都会被转发到SERVER节点，其本身**不持久化**这些信息；

**SERVER**

SERVER表示Consul的SERVER模式，表明这个是SERVER节点。这种模式下，功能和CLIENT都一样，唯一不同的是，它会把所有的信息**持久化**的本地。这样遇到故障，信息是可以被保留的。

 **SERVER-LEADER**

中间那个SERVER下面有LEADER的描述，表明这个SERVER节点是它们的老大。和其它SERVER不一样的一点是，它需要负责**同步注册信息**给其它的SERVER，同时也要负责**各个节点**的**健康监测**。

# Consul通讯接口

## RPC

主要用于内部通讯Gossip、日志分发、选主；

## HTTP API

服务发现、健康检查、KV存储等几乎所有功能；

## Consul Command(CLI)

Consul命令行工具可以与Consul agent进行连接。

实际上Consul CLI 默认调用HTTP API与Consul集群通讯，具体请参考：[官网](https://www.consul.io/docs/commands/index.html#consul_http_addr)

## DNS

常用于服务查询；

# Blocking Queries

在Consul中很多endpoint提供了一个特性，称为"Blocking Queries"。Blocking Queries使用long polling（长轮训）方式等待直至服务端发生变化；[官网](https://www.consul.io/api/features/blocking.html)说明如下：

> Many endpoints in Consul support a feature known as "blocking queries". A blocking query is used to wait for a potential change using long polling. 
>
> Endpoints that support blocking queries return an HTTP header named `X-Consul-Index`. This is a unique identifier representing the current state of the requested resource.
>
> On subsequent requests for this resource, the client can set the `index` query string parameter to the value of `X-Consul-Index`, indicating that the client wishes to wait for any changes subsequent to that index.

提供Blocking Queries特性的endpoint会在返回的HTTP header信息中携带X-Consul-Index，其是请求资源的状态唯一标识。举个例子：

**第一次请求**

- 可以看到返回信息中携带的X-Consul-Index为74：

```shell
$ curl -v http://localhost:8500/v1/health/service/hello-service
*   Trying ::1...
* TCP_NODELAY set
* Connection failed
* connect to ::1 port 8500 failed: Connection refused
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 8500 (#0)
> GET /v1/health/service/hello-service HTTP/1.1
> Host: localhost:8500
> User-Agent: curl/7.54.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Content-Type: application/json
< Vary: Accept-Encoding
< X-Consul-Effective-Consistency: leader
< X-Consul-Index: 74
< X-Consul-Knownleader: true
< X-Consul-Lastcontact: 0
< Date: Wed, 06 Nov 2019 13:34:53 GMT
< Content-Length: 3
<
[]
```

**后续请求**

- index 作为Query Parameter其值为上述X-Consul-Index的值；
- wait表示long polling时的超时时间；

```shell
$ curl -v http://localhost:8500/v1/health/service/hello-service?index=74&wait=1m
```

## 数据同步

通常数据不会在不同的DC之间进行复制。当一个请求请求另外一个DC上的数据时,Local Consul Server会把通过RPC请求转发至远端的Consul Server，当远端Consul Server不可用时，这些请求的资源将会不可用，但是并不会影响Local DC。当一个特殊情况时，有限的数据子集可以被复制，比如Consul 内置ACL Replication，或者另外的工具比如[consul-replicate](https://github.com/hashicorp/consul-replicate/)；





# Consul ACL



# Consul单节点安装

安装[下载地址](https://www.consul.io/downloads.html)下载对应操作系统的安装文件，Consul使用Go语言进行开发和实现，因此安装时只需要运行下载包中的二进制文件即可。

```shell
$ consul agent -dev -ui
```

- -dev表示开发服务器模式启动，不会进行数据持久化；
- -ui表示打开ui界面，即通过http://localhost:8500可以查看UI界面（默认能访问）；

启动信息如下：

```shell
$ consul agent -dev -ui

==> Starting Consul agent...
==> Consul agent running!
           Version: 'v1.4.4'
           Node ID: '0922070e-b5d0-42ef-2795-9bb08631ceb1'
         Node name: '000000133287y'
        Datacenter: 'dc1' (Segment: '<all>')
            Server: true (Bootstrap: false)
       Client Addr: [127.0.0.1] (HTTP: 8500, HTTPS: -1, gRPC: 8502, DNS: 8600)
      Cluster Addr: 127.0.0.1 (LAN: 8301, WAN: 8302)
           Encrypt: Gossip: false, TLS-Outgoing: false, TLS-Incoming: false
==> Log data will now stream in as it occurs:
    2019/11/05 10:49:12 [DEBUG] agent: Using random ID "0922070e-b5d0-42ef-2795-9bb08631ceb1" as node ID
    2019/11/05 10:49:12 [DEBUG] tlsutil: Update with version 1
    2019/11/05 10:49:12 [DEBUG] tlsutil: OutgoingRPCWrapper with version 1
    2019/11/05 10:49:12 [DEBUG] tlsutil: IncomingRPCConfig with version 1
    2019/11/05 10:49:12 [DEBUG] tlsutil: OutgoingRPCWrapper with version 1
    2019/11/05 10:49:12 [INFO] raft: Initial configuration (index=1): [{Suffrage:Voter ID:0922070e-b5d0-42ef-2795-9bb08631ceb1 Address:127.0.0.1:8300}]
    2019/11/05 10:49:12 [INFO] raft: Node at 127.0.0.1:8300 [Follower] entering Follower state (Leader: "")
    2019/11/05 10:49:12 [INFO] serf: EventMemberJoin: 000000133287y.dc1 127.0.0.1
    2019/11/05 10:49:12 [INFO] serf: EventMemberJoin: 000000133287y 127.0.0.1
    2019/11/05 10:49:12 [INFO] consul: Adding LAN server 000000133287y (Addr: tcp/127.0.0.1:8300) (DC: dc1)
    2019/11/05 10:49:12 [INFO] consul: Handled member-join event for server "000000133287y.dc1" in area "wan"
```

几点说明：

- 节点名称：这是代理agent的唯一名称。默认情况为主机名，可通过-node指定；
- 数据中心：这是代理agent运行的数据中心；每个节点都必须设置其数据中心，-datacenter标志可设置数据中心，对于单DC配置，默认为"dc1"；
- 服务器：表示其是以CLIENT还是SERVER模式启动；
- 客户端地址：这是用于代理的客户端接口地址，包括HTTP和DNS接口，默认情况下绑定在localhost；
- 集群地址：用于集群Consul agent之间通信的地址和端口集；
- -dev：该模式不能用于生产环境，该模式下不会持久化任何状态，该模式仅仅是为了快速便捷启动单节点Consul;

# Docker集群安装

## 集群节点规划

模拟在mac环境单机或者其他虚拟机中使用docker进行集群安装：

| 容器名称 | 容器IP地址 | 映射端口      | 服务运行模式  |
| -------- | ---------- | ------------- | ------------- |
| node1    | 172.17.0.2 | 8500 -> 8500  | Server Master |
| node2    | 172.17.0.3 | 9500 -> 8500  | Server        |
| node3    | 172.17.0.4 | 10500 -> 8500 | Server        |
| node4    | 172.17.0.5 | 11500 -> 8500 | Client        |

## 配置参数说明

| 参数列表         | 参数的含义和使用场景说明                                     |
| ---------------- | ------------------------------------------------------------ |
| advertise        | 通知展现地址用来改变我们给集群中的其他节点展现的地址，一般情况下-bind地址就是展现地址 |
| bootstrap        | 用来控制一个server是否在bootstrap模式，在一个datacenter中只能有一个server处于bootstrap模式，当一个server处于bootstrap模式时，可以自己选举为raft leader |
| bootstrap-expect | 在一个datacenter中期望提供的server节点数目，当该值提供的时候，consul一直等到达到指定sever数目的时候才会引导整个集群，该标记不能和bootstrap共用 |
| bind             | 该地址用来在集群内部的通讯IP地址，集群内的所有节点到地址都必须是可达的，默认是0.0.0.0 |
| client           | consul绑定在哪个client地址上，这个地址提供HTTP、DNS、RPC等服务，默认是127.0.0.1 |
| config-file      | 明确的指定要加载哪个配置文件                                 |
| config-dir       | 配置文件目录，里面所有以.json结尾的文件都会被加载            |
| data-dir         | 提供一个目录用来存放agent的状态，所有的agent允许都需要该目录，该目录必须是稳定的，系统重启后都继续存在 |
| dc               | 该标记控制agent允许的datacenter的名称，默认是dc1             |
| encrypt          | 指定secret key，使consul在通讯时进行加密，key可以通过consul keygen生成，同一个集群中的节点必须使用相同的key |
| join             | 加入一个已经启动的agent的ip地址，可以多次指定多个agent的地址。如果consul不能加入任何指定的地址中，则agent会启动失败，默认agent启动时不会加入任何节点 |
| retry-interval   | 两次join之间的时间间隔，默认是30s                            |
| retry-max        | 尝试重复join的次数，默认是0，也就是无限次尝试                |
| log-level        | consul agent启动后显示的日志信息级别。默认是info，可选：trace、debug、info、warn、err |
| node             | 节点在集群中的名称，在一个集群中必须是唯一的，默认是该节点的主机名 |
| protocol         | consul使用的协议版本                                         |
| rejoin           | 使consul忽略先前的离开，在再次启动后仍旧尝试加入集群中       |
| server           | 定义agent运行在server模式，每个集群至少有一个server，建议每个集群的server不要超过5个 |
| syslog           | 开启系统日志功能，只在linux/osx上生效                        |
| pid-file         | 提供一个路径来存放pid文件，可以使用该文件进行SIGINT/SIGHUP(关闭/更新)agent |

## Consul安装步骤

### 拉取Consul镜像

```shell
$ docker pull consul:latest
```

### 启动Server节点

**node1**

```shell
$ docker run -d --name=node1 --restart=always \
             -e 'CONSUL_LOCAL_CONFIG={"skip_leave_on_interrupt": true}' \
             -p 8300:8300 \
             -p 8301:8301 \
             -p 8301:8301/udp \
             -p 8302:8302/udp \
             -p 8302:8302 \
             -p 8400:8400 \
             -p 8500:8500 \
             -p 8600:8600 \
             -h node1 \
             consul agent -server -bind=172.17.0.2 -bootstrap-expect=3 -node=node1 \
             -data-dir=/tmp/data-dir -client 0.0.0.0 -ui
```

**node2**

```shell
$ docker run -d --name=node2 --restart=always \
             -e 'CONSUL_LOCAL_CONFIG={"skip_leave_on_interrupt": true}' \
             -p 9300:8300  \
             -p 9301:8301 \
             -p 9301:8301/udp \
             -p 9302:8302/udp \
             -p 9302:8302 \
             -p 9400:8400 \
             -p 9500:8500 \
             -p 9600:8600 \
             -h node2 \
             consul agent -server -bind=172.17.0.3 \
             -join=172.22.204.53 -node-id=$(uuidgen | awk '{print tolower($0)}') \
             -node=node2 \
             -data-dir=/tmp/data-dir -client 0.0.0.0 -ui
```

**node3**

```shell
$ docker run -d --name=node3 --restart=always \
             -e 'CONSUL_LOCAL_CONFIG={"skip_leave_on_interrupt": true}' \
             -p 10300:8300  \
             -p 10301:8301 \
             -p 10301:8301/udp \
             -p 10302:8302/udp \
             -p 10302:8302 \
             -p 10400:8400 \
             -p 10500:8500 \
             -p 10600:8600 \
             -h node2 \
             consul agent -server -bind=172.17.0.4 \
             -join=172.22.204.53 -node-id=$(uuidgen | awk '{print tolower($0)}') \
             -node=node3 \
             -data-dir=/tmp/data-dir -client 0.0.0.0 -ui
```





# Consul运维

## Consul的数据备份
