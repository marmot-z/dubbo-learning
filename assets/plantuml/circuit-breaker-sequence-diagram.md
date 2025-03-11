@startuml
Consumer -> Consumer: SphU.entry
Consumer --> Provider: 调用
activate Provider #FFBBBB
Provider --> Provider: 线程池等待
deactivate Provider
Provider --> Provider: SphU.entry
activate Provider #FFBBBB
Provider --> Provider: 业务阻塞
Provider -[#red]>x Consumer : timeout
Consumer --> Consumer: entry.exit
note left of Consumer: 消费方限流
Provider --> Provider: entry.exit
deactivate Provider
note right of Provider: 提供方限流
@enduml

// -------------------------------------

@startchen
entity Concsumer1 {
  All References
}

entity Concsumer2 {
  FooService
  ...
  BarService
}

entity Concsumer3 {
  FooService {
    sayHello
    greeting
    echo
    ...
  }
  ...
  BarService {
    sayHello
    greeting
    echo
    ...
  }
}
@endchen