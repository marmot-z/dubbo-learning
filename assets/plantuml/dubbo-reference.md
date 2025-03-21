```plantuml
@startuml
skinparam sequenceMessageAlign center
skinparam sequenceArrowColor #000000
skinparam sequenceLifeLineBackgroundColor #F0F0F0
skinparam sequenceParticipantBackgroundColor #E0F7FA, #FFF3E0, #E8F5E9, #FBE9E7, #ECEFF1

participant ReferenceConfig #E0F7FA
participant RegistryProtocol #FFF3E0
participant MigrationRuleHandler #E8F5E9
participant ProxyFactory #FBE9E7

activate ReferenceConfig
ReferenceConfig -> RegistryProtocol : createInvoker
activate RegistryProtocol
RegistryProtocol -> RegistryProtocol : create clusterInvoker
RegistryProtocol -> MigrationRuleHandler : migrate
activate MigrationRuleHandler
MigrationRuleHandler -> MigrationRuleHandler : refreshInvoker
note right: 订阅服务方提供的服务，刷新 invoker。\n有三种情况：\n1，应用级和接口级共存；\n2，应用级；\n3，接口级
deactivate MigrationRuleHandler
RegistryProtocol --> ReferenceConfig : .. 
deactivate RegistryProtocol
ReferenceConfig -> ProxyFactory : getProxy
activate ProxyFactory
note right: 使用 jdk、javassist 技术\n为目标接口类生成对应的代理
ProxyFactory --> ReferenceConfig : ..
deactivate ProxyFactory
deactivate ReferenceConfig
@enduml
```

相关 `prompt`:
```
使用 plantuml 语法为我生成以下顺序的时序图：
- ReferenceConfig 节点开始一个生命线
- ReferenceConfig 节点请求 RegistryProtocol 节点，请求内容为：createInvoker
- RegistryProtocol 节点请求本身，请求内容为：创建 clusterInvoker
- RegistryProtocol 节点请求 MigrationRuleHandler 节点，请求内容为：migrate
- MigrationRuleHandler 节点请求本身，请求内容为：refreshInvoker
- 在右侧生成一个备注：订阅服务方提供的服务，刷新 invoker。有三种情况1，应用级和接口级共存；2，应用级；3，接口级
- RegistryProtocol 节点返回到 ReferenceConfig 节点，使用虚线
- ReferenceConfig 节点请求 ProxyFactory 节点，请求内容为：getProxy
- 在右侧生成一个备注：使用 jdk、javassist 技术为目标接口类生成对应的代理
- ProxyFactory 节点返回到 ReferenceConfig 节点，使用虚线
- 关闭 ReferenceConfig 节点的生命线

其他的提示：
- 使用 activate, deactivate 关键词开启、关闭生命线
- 每个节点使用不同的柔和的颜色进行背景，注意不要使用红色相关的颜色
- 如果备注中内容长于 20 个字，则换行展示
- 不要使用 `actor` 关键字来初始化节点
```

```plantuml
@startuml

' 定义参与者并设置颜色
participant MigrationInvoker  #ADD8E6
participant InterfaceCompatibleRegistryProtocol  #90EE90
participant Directory  #FFFF99
participant ServiceDiscoveryRegistry  #FFCC99
participant AbstractServiceNameMapping  #C2B280
participant ServiceDiscovery  #DDA0DD
participant ServiceInstancesChangedListener  #87CEEB
participant DubboProtocol  #98FB98

' MigrationInvoker 开始生命线
activate MigrationInvoker
MigrationInvoker -> InterfaceCompatibleRegistryProtocol : getServiceDiscoveryInvoker

' InterfaceCompatibleRegistryProtocol 自调用
activate InterfaceCompatibleRegistryProtocol
InterfaceCompatibleRegistryProtocol -> InterfaceCompatibleRegistryProtocol : 创建 Directory

' InterfaceCompatibleRegistryProtocol 请求 Directory
InterfaceCompatibleRegistryProtocol -> Directory : subscribe

' Directory 请求 ServiceDiscoveryRegistry
activate Directory
Directory -> ServiceDiscoveryRegistry : subscribe

' ServiceDiscoveryRegistry 请求 AbstractServiceNameMapping
activate ServiceDiscoveryRegistry
ServiceDiscoveryRegistry -> AbstractServiceNameMapping : 通过服务名称获取应用名
AbstractServiceNameMapping --> ServiceDiscoveryRegistry : 应用名称

' ServiceDiscoveryRegistry 请求 ServiceDiscovery
ServiceDiscoveryRegistry -> ServiceDiscovery : 通过应用名获取应用实例信息
ServiceDiscovery --> ServiceDiscoveryRegistry : 应用实例信息

' ServiceDiscoveryRegistry 请求 ServiceInstancesChangedListener
ServiceDiscoveryRegistry -> ServiceInstancesChangedListener : 应用实例变化事件

' ServiceInstancesChangedListener 自调用
activate ServiceInstancesChangedListener
ServiceInstancesChangedListener -> ServiceInstancesChangedListener : 响应事件

' 备注
note right
1. 拉取对应实例的 meta 信息；
2. 通知其他监听器实例变化。
end note

' ServiceInstancesChangedListener 返回到 Directory
ServiceInstancesChangedListener --> Directory : 实例变化信息

' Directory 请求 DubboProtocol
Directory -> DubboProtocol : refer

' DubboProtocol 自调用
activate DubboProtocol
DubboProtocol -> DubboProtocol : 创建 netty 连接

deactivate DubboProtocol

' Directory 自调用
Directory -> Directory : 重建本地路由

deactivate ServiceInstancesChangedListener
deactivate ServiceDiscoveryRegistry
deactivate Directory
deactivate InterfaceCompatibleRegistryProtocol
deactivate MigrationInvoker

@enduml
```

相关 `prompt`：
```
使用 plantuml 语法为我生成以下顺序的时序图：
- MigrationInvoker 节点开始一个生命线
- MigrationInvoker 节点请求 InterfaceCompatibleRegistryProtocol 节点，请求内容为：getServiceDiscoveryInvoker
- InterfaceCompatibleRegistryProtocol 节点请求自身，请求内容为：创建 Directory
- InterfaceCompatibleRegistryProtocol 节点请求 Directory 节点，请求内容为：subscribe
- Directory 节点请求 ServiceDiscoveryRegistry 节点，请求内容为：subscribe
- ServiceDiscoveryRegistry 节点请求 AbstractServiceNameMapping 节点，请求内容为：通过服务名称获取应用名
- AbstractServiceNameMapping 节点返回到 ServiceDiscoveryRegistry 节点，返回内容为：应用名称
- ServiceDiscoveryRegistry 节点请求 ServiceDiscovery 节点，请求内容为：通过应用名获取应用实例信息
- ServiceDiscovery 节点返回到 ServiceDiscoveryRegistry 节点，返回内容为：应用实例信息
- ServiceDiscoveryRegistry 节点请求 ServiceInstancesChangedListener 节点，请求内容为：应用实例变化事件
- ServiceInstancesChangedListener 节点请求自身，请求内容为：响应事件
- 在右侧生成一个备注：1，拉取对应实例的 meta 信息；2，通知其他监听器实例变化
- ServiceInstancesChangedListener 节点返回到 Directory 节点，请求内容为：实例变化信息
- Directory 节点请求 DubboProtocol 节点，请求内容为：refer
- DubboProtocol 节点请求自身，请求内容为：创建 netty 连接
- Directory 节点请求自身，请求内容为：重建本地路由

其他的提示：
- 使用 activate, deactivate 关键词开启、关闭生命线
- 每个节点使用不同的柔和的颜色进行背景，注意不要使用红色相关的颜色
- 如果备注中内容长于 20 个字，则换行展示
```