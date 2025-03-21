```plantuml
@startuml

' 定义参与者并设置颜色
participant DefaultModuleDeployer  #ADD8E6
participant ServiceConfig  #90EE90
participant InjvmProtocol  #FFFF99
participant RegistryProtocol  #FFCC99
participant DubboProtocol  #DDA0DD
participant AbstractRegistry  #C2B280
participant ServiceDiscoveryRegistry  #87CEEB

' DefaultModuleDeployer 开始生命线
activate DefaultModuleDeployer
DefaultModuleDeployer -> ServiceConfig : 导出服务

' ServiceConfig 开始生命线
activate ServiceConfig
ServiceConfig -> InjvmProtocol : 本地导出

' InjvmProtocol 自调用
activate InjvmProtocol
InjvmProtocol -> InjvmProtocol : export

' 备注
note right
在本地记录服务名和 invoker 的对应关系
end note

InjvmProtocol --> ServiceConfig

deactivate InjvmProtocol

' ServiceConfig 远程导出
ServiceConfig -> RegistryProtocol : 远程导出

' RegistryProtocol 本地导出
activate RegistryProtocol
RegistryProtocol -> DubboProtocol : 本地导出

' DubboProtocol 自调用
activate DubboProtocol
DubboProtocol -> DubboProtocol : export

' 备注
note right
开启对应协议的服务，如：
启动一个 dubbo 协议的 netty server
end note

deactivate DubboProtocol

' RegistryProtocol 进行接口级别注册
RegistryProtocol -> AbstractRegistry : 接口级别注册

' AbstractRegistry 自调用
activate AbstractRegistry
AbstractRegistry -> AbstractRegistry : register

' 备注
note right
接口级别注册时，在注册中心
创建对应服务节点，等待消费方订阅
end note

deactivate AbstractRegistry

' RegistryProtocol 进行应用级别注册
RegistryProtocol -> ServiceDiscoveryRegistry : 应用级别注册

' ServiceDiscoveryRegistry 自调用
activate ServiceDiscoveryRegistry
ServiceDiscoveryRegistry -> ServiceDiscoveryRegistry : register

' 备注
note right
应用级别注册是，将服务的
meta 信息写入到本地
end note

deactivate ServiceDiscoveryRegistry

RegistryProtocol --> ServiceConfig

deactivate RegistryProtocol

ServiceConfig --> DefaultModuleDeployer : 服务导出完毕

deactivate ServiceConfig

' DefaultModuleDeployer 完成启动
DefaultModuleDeployer -> DefaultModuleDeployer : dubbo 启动完毕

' 备注
note right
1. 对外暴露 metadataService，使得消费方
可以拉取本地服务的 meta 信息；
2. 拼接本机实例信息，注册到注册中心的
/services 节点，使得消费方可以发现该服务提供实例
end note

deactivate DefaultModuleDeployer

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
- ServiceConfig 节点返回 DefaultModuleDeployer 节点，返回内容为：服务导出完毕
- DefaultModuleDeployer 节点请求自身，请求内容为：dubbo 启动完毕
- 在右侧生成一个备注：1，对外暴露 metadataService，使得消费方可以拉取本地服务的 meta 信息；2，拼接本机实例信息，注册到注册中心的 /services 节点，使得消费方可以发现该服务提供实例


其他的提示：
- 使用 activate, deactivate 关键词开启、关闭生命线
- 每个节点使用不同的柔和的颜色进行背景，注意不要使用红色相关的颜色
- 如果备注中内容长于 20 个字，则换行展示
```