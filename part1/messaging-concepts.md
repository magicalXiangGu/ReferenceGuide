Messaging concepts
消息概念
==================

One of the core concepts in Axon is messaging. All communication between components is done using message objects. This gives these components the location transparency needed to be able to scale and distribute these components when necessary.

xon的核心概念之一就是消息传递。组件之间的所有通信都使用消息对象完成。这使这些组件具有必要的位置透明性，以便在必要时能够扩展和分发这些组件。

Although all these messages implement the `Message` interface, there is a clear distinction between the different types of Messages and how they are treated.

虽然所有这些消息都实现了Message接口，但是不同类型的Message和它们的处理方式之间有明确的区别。

All Messages contain a payload, meta data and unique identifier. The payload of the message is the functional description of what the message means. The combination of the class name of this object and the data it carries, describe the application's meaning of the message. The meta data allows you to describe the context in which a message is being sent. You can, for example, store tracing information, to allow the origin or cause of messages to be tracked. You can also store information to describe the security context under which a command is being executed.

所有消息包含有效载荷，元数据和唯一标识符。消息的有效载荷是消息意义的功能描述。对象的类名与其携带的数据共同描述了应用程序的消息含义。元数据允许描述发送消息的上下文，例如，您可以存储跟踪信息，以便跟踪消息的来源或原因。您还可以存储信息来描述正在执行命令的安全上下文。

> **Note**
>
> Note that all Messages are immutable. Storing data in a Message actually means creating a new Message based on the previous one, with extra information added to it. This guarantees that Messages are safe to use in a multi-threaded and distributed environment.

>
>请注意，所有Message都是不可变的。在消息中存储数据实际上意味着创建一个基于上一个消息的新消息，并添加了额外的信息。这确保了Message可以安全地在多线程和分布式环境中使用。

Commands
--------

Commands describe an intent to change the application's state. They are implemented as (preferably read-only) POJOs that are wrapped using one of the `CommandMessage` implementations.

命令描述了改变应用程序状态的意图。它们被实现为（最好是只读）POJO，它们使用CommandMessage实现之一进行包装。

Commands always have exactly one destination. While the sender doesn't care which component handles the command or where that component resides, it may be interesting in knowing the outcome of it. That's why Command messages sent over the Command Bus allow for a result to be returned.

命令总是有一个目的地。虽然发件人不在乎哪个组件处理命令或该组件在哪里，但知道其结果可能是有趣的。这就是为什么通过命令总线发送的命令消息允许返回结果。

Events
------

Events are objects that describe something that has occurred in the application. A typical source of Events is the Aggregate. When something important has occurred within the Aggregate, it will raise an Event. In Axon Framework, Events can be any object. You are highly encouraged to make sure all Events are serializable.

事件是描述应用程序中发生的事情的对象。事件的典型来源是聚合。当聚合中发生重要事情时，它会引发一个事件。在Axon框架中，事件可以是任何对象。非常鼓励您确保所有时间都是可序列化的。

When Events are dispatched, Axon wraps them in an `EventMessage`. The actual type of Message used depends on the origin of the Event. When an Event is raised by an Aggregate, it is wrapped in a `DomainEventMessage` (which extends `EventMessage`). All other Events are wrapped in an `EventMessage.` Aside from common `Message` attributes like a unique Identifier an `EventMessage` also contains a timestamp. The `DomainEventMessage` additionally contains the type and identifier of the aggregate that raised the Event. It also contains the sequence number of the event in the aggregate's event stream, which allows the order of events to be reproduced.

事件发送后，Axon将它们包装在EventMessage中。使用的消息的实际类型取决于事件的起源。当一个事件由一个Aggregate引发时，它被包含在一个DomainEventMessage（它继承了EventMessage）中。所有其他事件都包含在EventMessage中。除了常见的Message属性，如唯一的标识符，EventMessage还包含时间戳。DomainEventMessage另外包含引发事件的聚合的类型和标识符。它还包含聚合事件流中事件的序列号，这样可以再现事件的顺序。

> **Note**
>
> Even though the `DomainEventMessage` contains a reference to the Aggregate Identifier, you should always include the identifier in the actual Event itself as well. The identifier in the DomainEventMessage is used by the EventStore to store events and may not always provide a reliable value for other purposes.

>
> 即使DomainEventMessage包含对聚合标识符的引用，您应该始终将标识符包括在实际的事件本身中。 DomainEventMessage中的标识符由EventStore用于存储事件，并不总是为其他目的提供可靠的值。

The original Event object is stored as the Payload of an EventMessage. Next to the payload, you can store information in the Meta Data of an Event Message. The intent of the Meta Data is to store additional information about an Event that is not primarily intended as business information. Auditing information is a typical example. It allows you to see under which circumstances an Event was raised, such as the User Account that triggered the processing, or the name of the machine that processed the Event.

原始Event对象存储为EventMessage的有效负载。在有效载荷之外，您可以将信息存储在事件消息的元数据中。元数据的目的是存储关于事件的附加信息并不是业务信息的主要意图。审计信息是一个典型的例子，它允许您查看在何种情况下引发的事件，例如触发处理的用户帐户，或处理事件的机器名称。

> **Note**
>
> In general, you should not base business decisions on information in the meta-data of event messages. If that is the case, you might have information attached that should really be part of the Event itself instead. Meta-data is typically used for reporting, auditing and tracing.

>
> 一般来说，您不应将业务决策作为事件消息的元数据中的信息。如果是这样的话，您可能有附加的信息，实际上应该是事件本身的一部分。元数据通常用于报告、审计和跟踪。

Although not enforced, it is good practice to make domain events immutable, preferably by making all fields final and by initializing the event within the constructor. Consider using a Builder pattern if Event construction is too cumbersome.

虽然没有执行，但优良做法是使领域事件不可变，最好通过使所有字段最终化，并通过在构造函数中初始化事件。如果事件构造太麻烦，请考虑使用Builder模式。

> **Note**
>
> Although Domain Events technically indicate a state change, you should try to capture the intention of the state in the event, too. A good practice is to use an abstract implementation of a domain event to capture the fact that certain state has changed, and use a concrete sub-implementation of that abstract class that indicates the intention of the change. For example, you could have an abstract `AddressChangedEvent`, and two implementations `ContactMovedEvent` and `AddressCorrectedEvent` that capture the intent of the state change. Some listeners don't care about the intent (e.g. database updating event listeners). These will listen to the abstract type. Other listeners do care about the intent and these will listen to the concrete subtypes (e.g. to send an address change confirmation email to the customer).
>
> ![ "Adding intent to events](state-change-intent.png)

> **Note**
>
> 虽然域事件技术上指示状态变化，但您也应该尝试捕获事件中状态的意图。一个好的做法是使用域事件的抽象实现来捕获某些状态已经改变的事实，并使用该抽象类的具体子实现来指示改变的意图。例如，您可以拥有一个抽象的AddressChangedEvent，以及两个实现ContactMovedEvent和AddressCorrectedEvent来捕获状态更改的意图。一些监听不关心意图（例如数据库更新事件监听器）。他们将监听抽象类型。其他监听会关心意图，并且会监听具体的子类型（例如向客户发送地址更改确认电子邮件）。

When dispatching an Event on the Event Bus, you will need to wrap it in an Event Message. The `GenericEventMessage` is an implementation that allows you to wrap your Event in a Message. You can use the constructor, or the static `asEventMessage()` method. The latter checks whether the given parameter doesn't already implement the `Message` interface. If so, it is either returned directly (if it implements `EventMessage`,) or it returns a new `GenericEventMessage` using the given `Message`'s payload and Meta Data. If an Event is applied (published) by an Aggregate Axon will automatically wrap the Event in a `DomainEventMessage` containing the Aggregate's Identifier, Type and Sequence Number. 

Unit of Work
------------

The Unit of Work is an important concept in the Axon Framework, though in most cases you are unlikely to interact with it directly. The processing of a message is seen as a single unit. The purpose of the Unit of Work is to coordinate actions performed during the processing of a message (Command or Event). Components can register actions to be performed during each of the stages of a Unit of Work, such as onPrepareCommit or onCleanup.

You are unlikely to need direct access to the Unit of Work. It is mainly used by the building blocks that Axon provides. If you do need access to it, for whatever reason, there are a few ways to obtain it. The Handler receives the Unit Of Work through a parameter in the handle method. If you use annotation support, you may add a parameter of type `UnitOfWork` to your annotated method. In other locations, you can retrieve the Unit of Work bound to the current thread by calling `CurrentUnitOfWork.get()`. Note that this method will throw an exception if there is no Unit of Work bound to the current thread. Use `CurrentUnitOfWork.isStarted()` to find out if one is available.

One reason to require access to the current Unit of Work is to attach resources that need to be reused several times during the course of message processing, or if created resources need to be cleaned up when the Unit of Work completes. In such case, the `unitOfWork.getOrComputeResource()` and the lifecycle callback methods, such as `onRollback()`, `afterCommit()` and `onCleanup()` allow you to register resources and declare actions to be taken during the processing of this Unit of Work.

> **Note**
>
> Note that the Unit of Work is merely a buffer of changes, not a replacement for Transactions. Although all staged changes are only committed when the Unit of Work is committed, its commit is not atomic. That means that when a commit fails, some changes might have been persisted, while others are not. Best practices dictate that a Command should never contain more than one action. If you stick to that practice, a Unit of Work will contain a single action, making it safe to use as-is. If you have more actions in your Unit of Work, then you could consider attaching a transaction to the Unit of Work's commit. Use `unitOfWork.onCommit(..)` to register actions that need to be taken when the Unit of Work is being committed.

Your handlers may throw an Exception as a result of processing a message. By default, unchecked exceptions will cause the UnitOfWork to roll back all changes. As a result, scheduled side effects are cancelled.

Axon provides a few Rollback strategies out-of-the-box:
 - `RollbackConfigurationType.NEVER`, will always commit the Unit of Work,
 - `RollbackConfigurationType.ANY_THROWABLE`, will always roll back when an exception occurs,
 - `RollbackConfigurationType.UNCHECKED_EXCEPTIONS`, will roll back on Errors and Runtime Exception
 - `RollbackConfigurationType.RUNTIME_EXCEPTION`, will roll back on Runtime Exceptions (but not on Errors)

When using Axon components to process messages, the lifecycle of the Unit of Work will be automatically managed for you. If you choose not to use these components, but implement processing yourself, you will need to programmatically start and commit (or roll back) a Unit of Work instead.

In most cases, the `DefaultUnitOfWork` will provide you with the functionality you need. It expects processing to happen within a single thread. To execute a task in the context of a Unit Of Work, simply call `UnitOfWork.execute(Runnable)` or `UnitOfWork.executeWithResult(Callable)` on a new `DefaultUnitOfWork`. The Unit Of Work will be started and committed when the task completes, or rolled back if the task fails. You can also choose to manually start, commit or rollback the Unit Of Work if you need more control.

Typical usage is as follows:

``` java
UnitOfWork uow = DefaultUnitOfWork.startAndGet(message);
// then, either use the autocommit approach:
uow.executeWithResult(() -> ... logic here);

// or manually commit or rollback:
try {
    // business logic comes here
    uow.commit();
} catch (Exception e) {
    uow.rollback(e);
    // maybe rethrow...
}
```

A Unit of Work knows several phases. Each time it progresses to another phase, the UnitOfWork Listeners are notified.

-   Active phase: this is where the Unit of Work is started. The Unit of Work is generally registered with the current thread in this phase (through `CurrentUnitOfWork.set(UnitOfWork)`). Subsequently the message is typically handled by a message handler in this phase.

-   Commit phase: after processing of the message is done but before the Unit of Work is committed, the `onPrepareCommit` listeners are invoked. If a Unit of Work is bound to a transaction, the `onCommit` listeners are invoked to commit any supporting transactions. When the commit succeeds, the `afterCommit` listeners are invoked. If a commit or any step before fails, the `onRollback` listeners are invoked. The message handler result is contained in the `ExecutionResult` of the Unit Of Work, if available.

-   Cleanup phase: This is the phase where any of the resources held by this Unit of Work (such as locks) are to be released. If multiple Units Of Work are nested, the cleanup phase is postponed until the outer unit of work is ready to clean up.

The message handling process can be considered an atomic procedure; it should either be processed entirely, or not at all. Axon Framework uses the Unit Of Work to track actions performed by the message handlers. After the handler completed, Axon will try to commit the actions registered with the Unit Of Work.

It is possible to bind a transaction to a Unit of Work. Many componens, such as the CommandBus implementations and all asynchronously processing Event Processors, allow you to configure a Transaction Manager. This Transaction Manager will then be used to create the transactions to bind to the Unit of Work that is used to manage the process of a Message.

When application components need resources at different stages of message processing, such as a Database Connection or an EntityManager, these resources can be attached to the Unit of Work. The `unitOfWork.getResources()` method allows you to access the resources attached to the current Unit of Work. Several helper methods are available on the Unit of Work directly, to make working with resources easier.

When nested Units of Work need to be able to access a resource, it is recommended to register it on the root Unit of Work, which can be accessed using `unitOfWork.root()`. If a Unit of Work is the root, it will simply return itself.
