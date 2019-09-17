## 初探微服务架构

![map](../pic/microservice.png)

首先服务提供者（就是提供服务的一方）按照一定格式的服务描述，向注册中心注册服务，声明自己能够提供哪些服务以及服务的地址是什么，完成服务发布。

接下来服务消费者（就是调用服务的一方）请求注册中心，查询所需要调用服务的地址，然后以约定的通信协议向服务提供者发起请求，得到请求结果后再按照约定的协议解析结果。

而且在服务的调用过程中，服务的请求耗时、调用量以及成功率等指标都会被记录下来用作监控，调用经过的链路信息会被记录下来，用于故障定位和问题追踪。在这期间，如果调用失败，可以通过重试等服务治理手段来保证成功率。

总结一下，微服务架构下，服务调用主要依赖下面几个基本组件：

* 服务描述

常用的服务描述方式包括RESTful API、XML配置以及IDL文件三种。

* 注册中心

一般来讲，注册中心的工作流程是：

- 服务提供者在启动时，根据服务发布文件中配置的发布信息向注册中心注册自己的服务。
- 服务消费者在启动时，根据消费者配置文件中配置的服务信息向注册中心订阅自己所需要的服务。
- 注册中心返回服务提供者地址列表给服务消费者。
- 当服务提供者发生变化，比如有节点新增或者销毁，注册中心将变更通知给服务消费者。

* 服务框架

发起调用之前你还需要解决以下几个问题。

- 服务通信采用什么协议? e.g TCP UDP HTTP
- 数据传输采用什么方式? 同步还是异步，是在单连接上传输，还是多路复用。
- 数据压缩采用什么格式?

* 服务监控

一旦服务消费者与服务提供者之间能够正常发起服务调用，你就需要对调用情况进行监控，以了解服务是否正常。通常来讲，服务监控主要包括三个流程。

- 指标收集
- 数据处理
- 数据展示

* 服务追踪

除了需要对服务调用情况进行监控之外，你还需要记录服务调用经过的每一层链路，以便进行问题追踪和故障定位。

- 服务消费者发起调用前，会在本地按照一定的规则生成一个requestid，发起调用时，将requestid当作请求参数的一部分，传递给服务提供者。
- 服务提供者接收到请求后，记录下这次请求的requestid，然后处理请求。如果服务提供者继续请求其他服务，会在本地再生成一个自己的requestid，然后把这两个requestid都当作请求参数继续往下传递。

* 服务治理

服务监控能够发现问题，服务追踪能够定位问题所在，而解决问题就得靠服务治理了。服务治理就是通过一系列的手段来保证在各种意外情况下，服务调用仍然能够正常进行。

三种最常见的需要引入服务治理的场景

- 单机故障

服务治理可以通过一定的策略，自动摘除故障节点，不需要人为干预，就能保证单机故障不会影响业务。

- 单IDC(网络线路)故障

服务治理可以通过自动切换故障IDC的流量到其他正常IDC，可以避免因为单IDC故障引起的大批量业务受影响。

- 依赖服务不可用

服务治理可以通过**熔断**，在依赖服务异常的情况下，一段时期内停止发起调用而直接返回。这样一方面保证了服务消费者能够不被拖垮，另一方面也给服务提供者减少压力，使其能够尽快恢复。


## 发布和引用服务

### RESTful API

首先来说说RESTful API的方式，主要被用作HTTP或者HTTPS协议的接口定义，即使在非微服务架构体系下，也被广泛采用。

### XML配置

这种方式的服务发布和引用主要分三个步骤：

- 服务提供者定义接口，并实现接口。

- 服务提供者进程启动时，通过加载server.xml配置文件将接口暴露出去。

- 服务消费者进程启动时，通过加载client.xml配置文件来引入要调用的接口。

### IDL文件 （interface description language）

通过一种中立的方式来描述接口，使得在不同的平台上运行的对象和不同语言编写的程序可以相互通信交流。比如你用Java语言实现提供的一个服务，也能被PHP语言调用。

也就是说IDL主要是用作***跨语言平台的服务之间的调用***，有两种最常用的IDL：一个是Facebook开源的Thrift协议，另一个是Google开源的gRPC协议。无论是Thrift协议还是gRPC协议，它们的工作原理都是类似的。