```plantuml
@startuml

skinparam backgroundColor White
skinparam sequenceParticipantBackgroundColor LightBlue

participant DefaultModuleDeployer as DMD #lightblue
participant ServiceConfig as SC #lightgreen
participant InjvmProtocol as IP #lightyellow
participant RegistryProtocol as RP #lightgray
participant DubboProtocol as DP #lightcyan
participant AbstractRegistry as AR #lightpink
participant ServiceDiscoveryRegistry as SDR #lightsalmon
participant ServiceNameMapping as SNM #lightsteelblue

activate DMD
DMD -> SC : 导出服务

activate SC
SC -> IP : 本地导出

activate IP
IP -> IP : export
note right: 在本地记录服务名和\ninvoker 的对应关系

deactivate IP
SC <--- IP

SC -> RP : 远程导出

activate RP
RP -> DP : 本地导出

activate DP
DP -> DP : export
note right: 开启对应协议的服务，\n如：启动一个 dubbo 协议的 netty server

deactivate DP

RP -> AR : 接口级别注册

activate AR
AR -> AR : register
note right: 接口级别注册时，在注册中心\n创建对应服务节点，等待消费方订阅

deactivate AR

RP -> SDR : 应用级别注册

activate SDR
SDR -> SDR : register
note right: 应用级别注册是，\n将服务的 meta 信息写入到本地

deactivate SDR

SC <--- RP

SC -> SNM : 上传映射
note right: 将服务与 app 映射关系写入\n注册中心的 /mapping 路径下
SC ---> DMD : 服务导出完毕

DMD -> DMD : dubbo 启动完毕
note right: 1，对外暴露 metadataService，使得消费方\n可以拉取本地服务的 meta 信息；2，拼接本机实例信息，\n注册到注册中心的 /services 节点，使得消费方可以发现该服务提供实例

@enduml
```

相关 `prompt`：
```
使用 plantuml 语法为我生成以下顺序的时序图：
- DefaultModuleDeployer 节点开始一个生命线
- DefaultModuleDeployer 节点请求 ServiceConfig 节点，请求内容为：导出服务
- ServiceConfig 节点开始一个生命线
- ServiceConfig 节点请求 InjvmProtocol 节点，请求内容为：本地导出
- InjvmProtocol 节点请求自身，请求内容为：export 
- 在右侧生成一个备注：在本地记录服务名和 invoker 的对应关系
- InjvmProtocol 节点返回到 ServiceConfig 节点，使用虚线
- ServiceConfig 节点请求 RegistryProtocol 节点，请求内容为：远程导出
- RegistryProtocol 节点调用 DubboProtocol 节点，请求内容为：本地导出
- DubboProtocol 节点调用自身，请求内容为：export
- 在右侧生成一个备注：开启对应协议的服务，如：启动一个 dubbo 协议的 netty server
- RegistryProtocol 节点调用 AbstractRegistry 节点，请求内容为：接口级别注册
- AbstractRegistry 节点调用自身，请求内容为：register
- 在右侧生成一个备注：接口级别注册时，在注册中心创建对应服务节点，等待消费方订阅
- RegistryProtocol 节点调用 ServiceDiscoveryRegistry 节点，请求内容为：应用级别注册
- ServiceDiscoveryRegistry 节点调用自身，请求内容为：register
- 在右侧生成一个备注：应用级别注册是，将服务的 meta 信息写入到本地
- RegistryProtocol 节点返回到 ServiceConfig 节点，使用虚线
- ServiceConfig 节点请求 ServiceNameMapping 节点，请求内容为：上传映射
- 在右侧生成一个备注：将服务与app映射关系写入注册中心的/mapping路径下
- ServiceConfig 节点返回 DefaultModuleDeployer 节点，使用虚线，返回内容为：服务导出完毕
- DefaultModuleDeployer 节点请求自身，请求内容为：dubbo 启动完毕
- 在右侧生成一个备注：1，对外暴露 metadataService，使得消费方可以拉取本地服务的 meta 信息；2，拼接本机实例信息，注册到注册中心的 /services 节点，使得消费方可以发现该服务提供实例


其他的提示：
- 使用 activate, deactivate 关键词开启、关闭生命线
- 每个节点使用不同的柔和的颜色进行背景，注意不要使用红色相关的颜色
- 如果备注中内容长于 20 个字，则换行展示
```