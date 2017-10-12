Architectural Overview
======================

CQRS on itself is a very simple pattern. It only prescribes that the component of an application that processes commands should be separated from the component that processes queries. Although this separation is very simple on itself, it provides a number of very powerful features when combined with other patterns. Axon provides the building blocks that make it easier to implement the different patterns that can be used in combination with CQRS.

CQRS 本身是个非常简单的模式。它仅仅描述了一个应用程序的查询端与命令端应该彼此分离。虽然这种分离本身非常简单，但它与其他模式结合时提供了许多非常强大的特性。Axon提供的模块使不同的模式易于与CQRS结合使用。

The diagram below shows an example of an extended layout of a CQRS-based event driven architecture. The UI component, displayed on the left, interacts with the rest of the application in two ways: it sends commands to the application (shown in the top section), and it queries the application for information (shown in the bottom section).

下图是一个CQRS基于事件驱动的架构示意图。左侧显示的UI组件以两种方式与应用程序的其余部分交互：发送命令，查询信息。

![Architecture overview of a CQRS application](detailed-architecture-overview.png)

Commands are typically represented by simple and straightforward objects that contain all data necessary for a command handler to execute it. A command expresses its intent by its name. In Java terms, that means the class name is used to figure out what needs to be done, and the fields of the command provide the information required to do it.

命令通常由简单而直接的对象表示，这些对象包含命令处理器响应该命令所需的所有数据。命令以其名称表示意图。在Java里使用类名称表示需要做什么，命令类字段提供所需要的信息。
![Architecture overview of a CQRS application](detailed-architecture-overview.png)

The Command Bus receives commands and routes them to the Command Handlers. Each command handler responds to a specific type of command and executes logic based on the contents of the command. In some cases, however, you would also want to execute logic regardless of the actual type of command, such as validation, logging or authorization.

命令总线接收命令并路由他们到相应的命令处理器。每个命令处理器响应特定类型的命令，并根据命令的内容执行逻辑。然而，有些情况，你希望执行逻辑，而不考虑实际的命令类型，例如验证、日志记录或授权。

The command handler retrieves domain objects (Aggregates) from a repository and executes methods on them to change their state. These aggregates typically contain the actual business logic and are therefore responsible for guarding their own invariants. The state changes of aggregates result in the generation of Domain Events. Both the Domain Events and the Aggregates form the domain model.

命令处理器从仓库获取领域对象（聚合），并执行聚合内改变它们状态的方法。这些聚合通常包含实际的业务逻辑，因此聚合本身负责维护自己的状态。领域事件产生聚合状态的变化。领域事件与聚合共同组成领域模型。

Repositories are responsible for providing access to aggregates. Typically, these repositories are optimized for lookup of an aggregate by its unique identifier only. Some repositories will store the state of the aggregate itself (using Object Relational Mapping, for example), while others store the state changes that the aggregate has gone through in an Event Store. The repository is also responsible for persisting the changes made to aggregates in its backing storage.

库是负责获取聚合。通常，这些仓库被优化为只能通过唯一的标识查寻聚合。一些仓库负责存储聚合本身的状态（例如：ORM），然而一些其他的仓库（Event Store）存储聚合状态改变的过程。仓库还负责持久化其后备存储中聚合所做的更改。

Axon provides support for both the direct way of persisting aggregates (using object-relational-mapping, for example) and for event sourcing.

Axon提供了直接持久化聚合（例如使用ORM）和事件溯源的支持。

The event bus dispatches events to all interested event listeners. This can either be done synchronously or asynchronously. Asynchronous event dispatching allows the command execution to return and hand over control to the user, while the events are being dispatched and processed in the background. Not having to wait for event processing to complete makes an application more responsive. Synchronous event processing, on the other hand, is simpler and is a sensible default. By default, synchronous processing will process event listeners in the same transaction that also processed the command.

事件总线可以同步或异步的分发事件给所有感兴趣的事件监听器。当事件正在后台分发和处理时，异步事件发布允许命令执行返回并将控制交给用户，不必等待事件处理完成更多的应用响应。同步事件处理更简单，并且是一个很好的默认值。默认情况下，同步处理将在处理该命令的同一事务中处理事件侦听器。

Event listeners receive events and handle them. Some handlers will update data sources used for querying while others send messages to external systems. As you might notice, the command handlers are completely unaware of the components that are interested in the changes they make. This means that it is very non-intrusive to extend the application with new functionality. All you need to do is add another event listener. The events loosely couple all components in your application together.

事件监听器接收并处理事件。一些处理器将更新用于查询数据源而其他则发送消息到外部系统。正如你所注意到的，命令处理器完全不知道对它们所做更改感兴趣的组件。这意味着扩展新功能是非入侵式的。所要做的只是添加一个事件监听器。这些事件将应用程序中的所有组件松散地连接在一起。

In some cases, event processing requires new commands to be sent to the application. An example of this is when an order is received. This could mean the customer's account should be debited with the amount of the purchase, and shipping must be told to prepare a shipment of the purchased goods. In many applications, logic will become more complicated than this: what if the customer didn't pay in time? Will you send the shipment right away, or await payment first? The saga is the CQRS concept responsible for managing these complex business transactions.

有些情况下，事件处理过程中需要发送新的命令。例如当接收到一个订单，意味着客户的账户应记入商家相应的购买金额，并告知快递应该准备运输这些购买的商品。在许多应用中，逻辑会变得比这更复杂：如果客户没有及时付款怎么办？你会立即发货还是等待付款？cqrs的saga组件负责管理这些复杂的商业事物。

The thin data layer in between the user interface and the data sources provides a clearly defined interface to the actual query implementation used. This data layer typically returns read-only DTO objects containing query results. The contents of these DTOs are typically driven by the needs of the User Interface. In most cases, they map directly to a specific view in the UI (also referred to as table-per-view).

用户界面和数据源之间的薄数据层为实际查询实现提供了一个明确定义的接口。这个数据层返回的都是包含查询结果的只读DTO对象。这些 DTOs的内容通常是由用户的接口需求驱动。大多数情况，它们直接映射到UI中的特定视图（也称为表视图）。

Axon does not provide any building blocks for this part of the application. The main reason is that this is very straightforward and doesn't differ much from the layered architecture.

Axon没有为这一部分应用程序提供任何的模块。主要原因是，这部分非常直接，与分层结构没有太大区别。

Axon Module Structure
=====================

Axon Framework consists of a number of modules that target specific problem areas of CQRS. Depending on the exact needs of your project, you will need to include one or more of these modules.

Axon框架由许多特定的CQRS问题领域模块组成。根据项目的具体需要，可包含一个或多个这些模块。

As of Axon 2.1, all modules are OSGi compatible bundles. This means they contain the required headers in the manifest file and declare the packages they import and export. At the moment, only the Slf4J bundle (1.7.0 &lt;= version &lt; 2.0.0) is required. All other imports are marked as optional, although you're very likely to need others.



Main modules
------------

Axon's main modules are the modules that have been thoroughly tested and are robust enough to use in demanding production environments. The maven groupId of all these modules is`org.axonframework`.

Axon的主要模块是经过彻底测试的模块，其健壮性足以在苛刻的生产环境中使用。所有这些模块的Maven GroupID是'org.axonframework'。

The Core module contains, as the name suggests, the Core components of Axon. If you use a single-node setup, this module is likely to provide all the components you need. All other Axon modules depend on this module, so it must always be available on the classpath.

顾名思义，核心模块包含Axon的核心部件。如果你使用一个单节点的设置，此模块会提供所有你需要的组件。所有其他轴突模块依赖于这个模块，所以它必须在类路径是可用的。

The Test module contains test fixtures that you can use to test Axon based components, such as your Command Handlers, Aggregates and Sagas. You typically do not need this module at runtime and will only need to be added to the classpath during tests.



The Distributed CommandBus modules contain implementations that can be used to distribute commands over multiple nodes. It comes with JGroups and Spring Cloud Connectors that are used to connect these nodes.

The AMQP module provides components that allow you to build up an EventBus using an AMQP-based message broker as distribution mechanism. This allows for guaranteed-delivery, even when the Event Handler node is temporarily unavailable.

The Spring module allows Axon components to be configured in the Spring Application context. It also provides a number of building block implementations specific to Spring Framework, such as an adapter for publishing and retrieving Axon Events on a Spring Messaging Channel.

MongoDB is a document based NoSQL database. The Mongo module provides Event and Saga Store implementations that store event streams and sagas in a MongoDB database.

Several AxonFramework components provide monitoring information. The Metrics module provides basic implementations based on Codehale to collect the monitoring information. 

Working with Axon APIs
======================

CQRS is an architectural pattern, making it impossible to provide a single solution that fits all projects. Axon Framework does not try to provide that one solution, obviously. Instead, Axon provides implementations that follow best practices and the means to tweak each of those implementations to your specific requirements.

Almost all infrastructure building blocks will provide hook points (such as Interceptors, Resolvers, etc.) that allow you to add application-specific behavior to those building blocks. In many cases, Axon will provide implementations for those hook points that fit most use cases. If required, you can simply implement your own.

Non-infrastructural objects, such as Messages, are generally immutable. This ensures that these objects are safe to use in a multi-threaded environment, without side-effects.

To ensure maximum customization, all Axon components are defined using interfaces. Abstract and concrete implementations are provided to help you on your way, but will nowhere be required by the framework. It is always possible to build a completely custom implementation of any building block using that interface.

Spring Support
==============

Axon Framework provides extensive support for Spring, but does not require you to use Spring in order to use Axon. All components can be configured programmatically and do not require Spring on the classpath. However, if you do use Spring, much of the configuration is made easier with the use of Spring's annotation support.
