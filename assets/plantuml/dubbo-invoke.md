```plantuml
@startuml

skinparam backgroundColor White
skinparam sequenceParticipantBackgroundColor LightBlue

participant ConsumeServiceInterface as CSI #lightblue
participant Proxy_Consumer as PC #lightgreen
participant ClusterInvoker as CI #lightyellow
participant Directory as DIR #lightgray
participant FilterChain_Consumer as FC_C #lightsalmon
participant DubboInvoker as DI #lightsteelblue
participant NettyClient as NC #lightgoldenrodyellow
participant DubboCodec_Consumer as DC_C #lightcyan
participant NettyServer as NS #lightpink
participant DubboCodec_Provider as DC_P #lightseagreen
participant ChannelHandler as CH #lightcoral
participant ExecutorService as ES #lightcoral
participant Invoker as INV #lightblue
participant FilterChain_Provider as FC_P #lightgreen
participant Proxy_Provider as PP #lightyellow
participant ProvideServiceImpl as PSI #lightgray

activate CSI
CSI -> PC : invoke

activate PC
PC -> CI : invoke

activate CI
CI -> DIR : 获取 invokers

activate DIR
DIR -> DIR : route
note right: 按照路由规则筛选出\n符合条件的 invokers
DIR --> CI : invokers

deactivate DIR
CI -> CI : select
note right: 初始化对应的 LoadBlanacer，\n进行负载均衡

CI -> FC_C : invoke

activate FC_C
FC_C -> FC_C : filte
FC_C -> DI : invoke

deactivate FC_C

activate DI
DI -> NC : invoke

activate NC
note right: 为请求生成对应的 DefaultFuture，\n放入全局的 Future 池中
NC -> DC_C : encode

activate DC_C
note right: DubboCodec 编码器通过\nNettyClient#doOpen 初始化设置
DC_C --> NC : serialized data

deactivate DC_C
NC -> NS : request

deactivate NC

activate NS
NS -> DC_P : decode

activate DC_P
note right: DubboCodec 编码器通过\nNettyServer#doOpen 初始化设置
DC_P --> NS : deserialized data

deactivate DC_P

NS -> CH : received

activate CH
CH -> ES : submit task

deactivate CH

activate ES
ES -> INV : execute

deactivate ES

activate INV
INV -> FC_P : invoke

activate FC_P
FC_P -> FC_P : filte
FC_P -> PP : invoke

deactivate FC_P

activate PP
PP -> PSI : invoke

deactivate PP

deactivate INV

deactivate NS

deactivate CI

deactivate PC

deactivate CSI

@enduml

```

相关 `prompt`：
```
使用 plantuml 语法为我生成以下顺序的时序图：
- ConsumeServiceInterface 节点开始一个生命线
- ConsumeServiceInterface 节点请求 Proxy 节点，请求内容为：invoke
- Proxy(Consumer) 节点请求 ClusterInvoker 节点，请求内容为：invoke
- ClusterInvoker 节点请求 Directory 节点，请求内容为：获取 invokers
- Directory 节点请求自身，请求内容为：route
- 在右侧生成一个备注：按照路由规则筛选出符合条件的 invokers
- Directory 节点返回给 ClusterInvoker 节点，返回内容为：invokers
- ClusterInvoker 节点请求自身，请求内容为：select
- 在右侧生成一个备注：初始化对应的 LoadBlanacer，进行负载均衡
- ClusterInvoker 节点请求 FilterChain 节点，请求内容为：invoke
- FilerChain(Consumer) 节点请求自身，请求内容为：filte
- FilerChain(Consumer) 节点请求 DubboInvoker 节点，请求内容为：invoke
- DubboInvoker 节点请求 NettyClient 节点，请求内容为：invoke
- 在右侧生成一个备注：为请求生成对应的 DefaultFuture，放入全局的 Future 池中
- NettyClient 节点请求 DubboCodec(Consumer) 节点，请求内容为：encode
- 在右侧生成一个备注：DubboCodec 编码器通过 NettyClient#doOpen 初始化设置
- DubboCodec(Consumer)节点返回到 NettyClient 节点，返回内容为：serialized data
- NettyClient 节点请求 NettyServer 节点，请求内容为：request
- NettyServer 节点请求 DubboCodec(Provider) 节点，请求内容为：decode
- 在右侧生成一个备注：DubboCodec 编码器通过 NettyServer#doOpen 初始化设置
- DubboCodec(Provider) 节点返回到 NettyServer 节点，返回内容为：deserialized data
- NettyServer 节点请求 ChannelHandler 节点，请求内容为：received
- ChannelHandler 节点请求 ExecutorService 节点，请求内容为：submit task
- ExecutorService 节点异步请求 Invoker 节点，请求内容为：execute
- Invoker 节点请求 FilerChain(Provider)节点，请求内容为：invoke
- FilerChain(Provider)节点请求自身，请求内容为：filte
- FilerChain(Provider)节点请求 Proxy(Provider)节点，请求内容为：invoke
- Proxy(Provider)节点请求 ProvideServiceImpl 节点，请求内容为：invoke

其他的提示：
- 使用 activate, deactivate 关键词开启、关闭生命线
- 每个节点使用不同的柔和的颜色进行背景，注意不要使用红色相关的颜色
- 如果备注中内容长于 20 个字，则换行展示
```

```plantuml
@startuml

skinparam backgroundColor White
skinparam sequenceParticipantBackgroundColor LightBlue

participant NettyServer as NS #lightblue
participant DubboCodec_Provider as DC_P #lightgreen
participant NettyClient as NC #lightyellow
participant DubboCodec_Consumer as DC_C #lightgray
participant ChannelHandler as CH #lightsalmon
participant Invoker as INV #lightsteelblue
participant Proxy_Consumer as PC #lightgoldenrodyellow
participant ConsumerServiceInterface as CSI #lightcyan

NS -> DC_P : encode

activate DC_P
DC_P --> NS : serialized data

deactivate DC_P

NS -> NC : response

activate NC
NC -> DC_C : decode

activate DC_C
DC_C --> NC : deserialized data

deactivate DC_C

NC -> CH : received

activate CH
CH -> CH : future completed
note right: 通过 response#id 在全局的 Future 池中找到\n对应的 DefaultFuture，设置响应内容，\n并标记为 completed

CH ---> INV : notify

activate INV
INV -> PC : return

deactivate INV

activate PC
PC -> CSI : return

deactivate PC


deactivate CH

deactivate NC

deactivate NS

@enduml

```

```
使用 plantuml 语法为我生成以下顺序的时序图：
- NettyServer 节点请求 DubboCodec(Provider) 节点，请求内容为：encode
- DubboCodec(Provider) 节点返回到 NettyServer 节点，返回内容为：serialized data
- NettyServer 节点请求 NettyClient 节点，请求内容为：response
- NettyClient 节点请求 DubboCodec(Consumer)节点，请求内容为：decode
- DubboCodec(Consumer)节点返回到 NettyClient 节点，返回内容为：deserialized data
- NettyClient 节点请求 ChannelHandler 节点，请求内容为：received
- ChannelHandler 节点请求自身，请求内容为：future completed
- 在右侧生成一个备注：通过 response#id 在全局的 Future 池中找到对应的 DefaultFuture，设置响应内容，并标记为 completed
- ChannelHandler 节点请求 Invoker 节点，使用虚线，请求内容为：notify
- Invoker 节点请求 Proxy(Consumer) 节点，请求内容为：return
- Proxy(Consumer) 节点请求 ConsumerServiceInterface 节点，请求内容为：return

其他的提示：
- 使用 activate, deactivate 关键词开启、关闭生命线
- 每个节点使用不同的柔和的颜色进行背景，注意不要使用红色相关的颜色
- 如果备注中内容长于 20 个字，则换行展示
```