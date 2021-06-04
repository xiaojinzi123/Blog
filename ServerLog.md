## 前言

**ServerLog** 是一套消息转发的架构.**ServerLog** 分为两部分

- 服务端 **Server**
- 客户端 **Client** 
  - 每一个 Client 可以有不同的角色.

三者通过长连接. **Producer** <==> **Server** <==> **Customer**

 **ServerLog** 提供了一个通用的数据传送的数据结构. **Message** (结构见下文)

## Server

- 维护与  **Producer** 和  **Customer** 的连接状态. 一旦心跳发送失败, 就踢掉了

- 对数据进行转发. 将 Message 发送给需要此数据的 **Customer** 
- 不直接产生 **Message** 数据, 但是可以拥有对**数据拦截、转发、加工**等操作. 同时也可以**发送消息给任何一个 Producer 或者 Customer**
- 心跳功能, 每隔一段时间会发送一个心跳包, 用于确认连接状态
- 为每一个 **Producer** 或者 **Customer** 生成一个唯一的 Tag

## Client

每一个 **Client** 可以有不同的角色. 可以是消息 **Producer**, 也可以是消息 **Customer**

甚至同时可以是消息  **Producer** 和  **Customer**

每一个 **Client** 的主要功能如下：

- 保持与 **Server** 稳定的连接
- 最好能支持发送**心跳包**
- 拥有接受 **text** 类型的信息和发送 **text** 数据的能力

## Producer

**Producer** 是消息生产者, 本身需要保持和 **Server** 的连接稳定. 提供发送 **Message** 的功能

##  Customer

**Customer** 是消息的消费者, **Customer** 在和 **Server** 连接之后, 会订阅自己感兴趣的数据

##  Message 数据结构

- action 标记数据的类型. 可以自定义
- data 具体的数据. 这个数据的格式和 action 息息相关
- selfTag 表示消息的发送者
- targetTag 表示消息的接受者

### 内置的一些消息类型(action的值)

- heartbeat 表示心跳包类型. data 为 null