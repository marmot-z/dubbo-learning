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