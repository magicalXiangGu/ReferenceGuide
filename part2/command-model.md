Command Model
===============

In a CQRS-based application, a Domain Model (as defined by Eric Evans and Martin Fowler) can be a very powerful mechanism to harness the complexity involved in the validation and execution of state changes. Although a typical Domain Model has a great number of building blocks, one of them plays a dominant role when applied to Command processing in CQRS: the Aggregate.

在基于CQRS的应用程序中，领域模型（由Eric Evans和Martin Fowler定义）可以是一个非常强大的机制，驾驭涉及验证和执行改变状态的复杂性。虽然典型的域模型具有大量的构建块，但是当应用于CQRS中的命令处理时，其中一个占据主导地位：Aggregate。

A state change within an application starts with a Command. A Command is a combination of expressed intent (which describes what you want done) as well as the information required to undertake action based on that intent. The Command Model is used to process the incoming command, to validate it and define the outcome. Within this model, a Command Handler is responsible for handling commands of a certain type and taking action based on the information contained inside it.

应用程序中的状态更改以命令开头。命令是表达意图（描述您想要完成的内容）以及基于该意图进行操作所需的信息的组合。命令模块用于处理传入的命令，以验证它并定义结果。在该模块中，命令处理器负责处理某种类型的命令，并根据其中包含的信息采取行动。

Aggregate
---------
An Aggregate is an entity or group of entities that is always kept in a consistent state. The Aggregate Root is the object on top of the aggregate tree that is responsible for maintaining this consistent state. This makes the Aggregate the prime building block for implementing a Command Model in any CQRS based application.

聚合是一个始终处于一致状态的实体或实体组。聚合根是聚合树之上的对象，负责维护此一致状态。这使得聚合是在任何基于CQRS的应用程序中实现命令模型的主要构建块。

> **Note**
>
> The term "Aggregate" refers to the aggregate as defined by Evans in Domain Driven Design:
>
> “A cluster of associated objects that are treated as a unit for the purpose of data changes. External references are restricted to one member of the Aggregate, designated as the root. A set of consistency rules applies within the Aggregate's boundaries.”

> **Note**
>
> 术语“聚合”是指由领域驱动设计中的Evans定义的集合：
>
>作为以更改数据为目的的单元的关联对象集群。外部引用仅限于聚合的根成员。一组一致性规则适用于Aggregate的边界。

For example, a "Contact" aggregate could contain two entities: Contact and Address. To keep the entire aggregate in a consistent state, adding an address to a contact should be done via the Contact entity. In this case, the Contact entity is the appointed aggregate root.

例如，“联系人”聚合可以包含两个实体：联系人和地址。为了使整个聚合保持一致状态，可以通过联系人实体向联系人添加地址。在这种情况下，“联系人”实体是指定的聚合根。

In Axon, aggregates are identified by an Aggregate Identifier. This may be any object, but there are a few guidelines for good implementations of identifiers. Identifiers must:

在Axon中，聚合由Aggregate Identifier标识。这可能是任何对象，但是有一些准则可以很好地实现标识符。标识符必须：

-   implement `equals` and `hashCode` to ensure good equality comparison with other instances,

-   implement a `toString()` method that provides a consistent result (equal identifiers should provide an equal toString() result), and

-   preferably be `Serializable`.

-   实现equals和hashCode以确保与其他实例的良好的等式比较，
-   实现一个提供一致结果的toString())方法（等同的标识符应该提供一个相等的toString()结果）
-   并且可序列化

The test fixtures (see [Testing](testing.md)) will verify these conditions and fail a test when an Aggregate uses an incompatible identifier. Identifiers of type `String`, `UUID` and the numeric types are always suitable. Do **not** use primitive types as identifiers, as they don't allow for lazy initialization. Axon may, in some circumstances, falsely assume the default value of a primitive to be the value of the identifier.

测试fixtures(see [Testing](testing.md))将验证这些条件，并且当聚合使用不兼容的标识符时测试失败。类型为String，UUID和数字类型的标识符始终是合适的。不要使用原始类型作为标识符，因为它们不允许进行延迟初始化。在某些情况下，Axon可能会将原语的默认值误认为是标识符的值。

> **Note**
>
> It is considered a good practice to use randomly generated identifiers, as opposed to sequenced ones. Using a sequence drastically reduces scalability of your application, since machines need to keep each other up-to-date of the last used sequence numbers. The chance of collisions with a UUID is very slim (a chance of 10<sup>−15</sup>, if you generate 8.2 × 10 <sup>11</sup> UUIDs).
>
> Furthermore, be careful when using functional identifiers for aggregates. They have a tendency to change, making it very hard to adapt your application accordingly.


> **Note**
>
> 使用随机生成的标识符被认为是一种很好的做法，而不是排序的标识符。使用序列大大降低了应用程序的可扩展性，因为机器需要保持最新的最后使用的序列号。与UUID发生冲突的记录，率很小（如果生成8.2 × 10 <sup>11</sup> 个UUID，则为10<sup>−15</sup>的几率）。此外，在使用聚合的功能标识符时要小心。他们有变化的趋势，很难适应您的应用程序

Aggregate implementations
-------------------------
An Aggregate is always accessed through a single Entity, called the Aggregate Root. Usually, the name of this Entity is the same as that of the Aggregate entirely. For example, an Order Aggregate may consist of an Order entity, which references several OrderLine entities. Order and Orderline together, form the Aggregate.

总是通过单个Entity访问聚合，称为聚合根。通常，该实体的名称与Aggregate的名称完全相同。例如，订单集合可以由一个订单实体组成，其中引用了多个OrderLine实体。Order和Orderline一起，形成总计。

An Aggregate is a regular object, which contains state and methods to alter that state. Although not entirely correct according to CQRS principles, it is also possible to expose the state of the aggregate through accessor methods.

聚合是一个常规对象，它包含状态和方法来更改该状态。虽然根据CQRS原则不是完全正确的，但也可以通过访问器方法暴露聚合体的状态

An Aggregate Root must declare a field that contains the Aggregate identifier. This identifier must be initialized at the latest when the first Event is published. This identifier field must be annotated by the `@AggregateIdentifier` annotation.
If you use JPA and have JPA annotations on the aggregate, Axon can also use the `@Id` annotation provided by JPA.

聚合根必须声明包含聚合标识符的字段。这个标识符必须在第一个事件发布时最早进行初始化。此标识符字段必须由@AggregateIdentifier注释注释。如果您使用JPA并在聚合上使用JPA注释，Axon还可以使用JPA提供的@Id注释。

Aggregates may use the `AggregateLifecycle.apply()` method to register events for publication. Unlike the `EventBus`, where messages need to be wrapped in an EventMessage, `apply()` allows you to pass in the payload object directly.

聚合可以使用AggregateLifecycle.apply()方法来注册用于发布的事件。与EventBus不同的是，消息需要包装在EventMessage中，apply()可以让您直接传入有效负载对象。

``` java
@Entity // Mark this aggregate as a JPA Entity
public class MyAggregate {
    
    @Id // When annotating with JPA @Id, the @AggregateIdentifier annotation is not necessary
    private String id;
    
    // fields containing state...
    
    @CommandHandler
    public MyAggregate(CreateMyAggregateCommand command) {
        // ... update state
        apply(new MyAggregateCreatedEvent(...));
    }
    
    // constructor needed by JPA
    protected MyAggregate() {
    }
}
```

Entities within an Aggregate can listen to the events the Aggregate publishes, by defining an `@EventHandler` annotated method. These methods will be invoked when an EventMessage is published (before any external handlers are published).

Aggregate中的实体可以通过定义一个@EventHandler注释方法来监听Aggregate发布的事件。当发布EventMessage（在发布任何外部处理程序之前）时，将调用这些方法。

Event sourced aggregates
------------------------
Besides storing the current state of an Aggregate, it is also possible to rebuild the state of an Aggregate based on the Events that it has published in the past. For this to work, all state changes must be represented by an Event.

除了存储聚合的当前状态之外，还可以基于过去发布的事件来重建聚合的状态。为了使其工作，所有状态更改必须由事件表示。

For the major part, Event Sourced Aggregates are similar to 'regular' aggregates: they must declare an identifier and can use the `apply` method to publish Events. However, state changes in Event Sourced Aggregates (i.e. any change of a Field value) must be *exclusively* performed in an `@EventSourcingHandler` annotated method. This includes setting the Aggregate Identifier.

对于主要的部分，事件溯源聚合与“常规”聚合相似：它们必须声明一个标识符，并且可以使用apply方法来发布事件。但是，必须在@EventSourcingHandler注释方法中专门执行事件溯源聚合中的状态更改（即字段值的任何更改）。这包括设置聚合标识符。

Note that the Aggregate Identifier must be set in the `@EventSourcingHandler` of the very first Event published by the Aggregate. This is usually the creation Event.

请注意，聚合标识符必须在由Aggregate发布的第一个事件的@EventSourcingHandler中设置。这通常是创建事件。

The Aggregate Root of an Event Sourced Aggregate must also contain a no-arg constructor. Axon Framework uses this constructor to create an empty Aggregate instance before initialize it using past Events. Failure to provide this constructor will result in an Exception when loading the Aggregate.

一个事件溯源聚合的聚合根也必须包含一个无参数构造函数。 Axon Framework使用此构造函数在使用过去的事件初始化之前创建一个空的聚合实例。不提供此构造函数将导致加载Aggregate时出现异常。

``` java
public class MyAggregateRoot {

    @AggregateIdentifier
    private String aggregateIdentifier;
    
    // fields containing state...

    @CommandHandler
    public MyAggregateRoot(CreateMyAggregate cmd) {
        apply(new MyAggregateCreatedEvent(cmd.getId()));
    }

    // constructor needed for reconstruction
    protected MyAggregateRoot() {
    }

    @EventSourcingHandler
    private void handleMyAggregateCreatedEvent(MyAggregateCreatedEvent event) {
        // make sure identifier is always initialized properly
        this.aggregateIdentifier = event.getMyAggregateIdentifier();
        
        // ... update state
    }
}                
```

`@EventSourcingHandler` annotated methods are resolved using specific rules. These rules are the same for the `@EventHandler` annotated methods, and are thoroughly explained in [Annotated Event Handler](./event-handling.md#defining-event-handlers).

@EventSourcingHandler注释方法使用特定的规则解决。这些规则与@EventHandler注释方法相同，并进行了详细解释in [Annotated Event Handler](./event-handling.md#defining-event-handlers)。

> **Note**
>
> Event handler methods may be private, as long as the security settings of the JVM allow the Axon Framework to change the accessibility of the method. This allows you to clearly separate the public API of your aggregate, which exposes the methods that generate events, from the internal logic, which processes the events.
>
> Most IDE's have an option to ignore "unused private method" warnings for methods with a specific annotation. Alternatively, you can add an `@SuppressWarnings("UnusedDeclaration")` annotation to the method to make sure you don't accidentally delete an Event handler method.

 **Note**
>
>事件处理程序方法可能是私有的，只要JVM的安全设置允许Axon Framework更改方法的可访问性。这允许您清楚地分离聚合的公共API，它公开了从内部逻辑中生成事件的方法，这些内部逻辑处理事件。
>
> 大多数IDE可以选择忽略具有特定注释的方法的“未使用的私有方法”警告。或者，您可以向该方法添加一个@SuppressWarnings(“UnusedDeclaration”)注释，以确保不会意外删除事件处理程序方法。

In some cases, especially when aggregate structures grow beyond just a couple of entities, it is cleaner to react on events being published in other entities of the same aggregate. However, since Event Handler methods are also invoked when reconstructing aggregate state, special precautions must be taken.

在某些情况下，特别是当聚合结构不仅仅是几个实体增长时，对于在同一聚合体的其他实体中发布的事件做出反应更为清晰。然而，由于事件处理程序方法在重建聚合状态时也被调用，因此必须采取特殊的预防措施。

It is possible to `apply()` new events inside an Event Sourcing Handler method. This makes it possible for an Entity B to apply an event in reaction to Entity A doing something. Axon will ignore the apply() invocation when replaying historic events. Do note that, in this case, the Event of the inner `apply()` invocation is only published to the Entities after all Entities have received the first Event. If more events need to be published, based on the state of an entity after applying an inner event, use `apply(...).andThenApply(...)`

可以在Event Sourcing Handler方法中`apply()`新事件。Entity B可能发送一个事件对应Entity A做某些事情。当重播历史事件时，Axon将忽略apply()调用。请注意，在这种情况下，内部apply()调用的事件仅在所有实体收到第一个事件后才发布到Entities。如果需要发布更多的事件，基于应用内部事件之后的实体的状态，使用`apply(...).andThenApply(...)`

You can also use the static `AggregateLifecycle.isLive()` method to check whether the aggregate is 'live'. Basically, an aggregate is considered live if it has finished replaying historic events. While replaying these events, isLive() will return false. Using this `isLive()` method, you can perform activity that should only be done when handling newly generated events.

您还可以使用静态`AggregateLifecycle.isLive()`方法来检查聚合是否为“live”。基本上，如果已经完成重播历史事件，则聚合被视为实时的。在重播这些事件时，isLive() 将返回false。Using this `isLive()` method,您可以执行只应在处理新生成的事件时执行的活动。

Complex Aggregate structures
----------------------------
Complex business logic often requires more than what an aggregate with only an aggregate root can provide. In that case, it is important that the complexity is spread over a number of entities within the aggregate. When using event sourcing, not only the aggregate root needs to use events to trigger state transitions, but so does each of the entities within that aggregate.

复杂的业务逻辑通常需要的不仅仅是一个只含有聚合根的聚合能提供的。在这种情况下，重要的是复杂性分布在聚合体内的多个实体中。当使用事件源时，不仅聚合根需要使用事件来触发状态转换，而且聚合中的每个实体也是如此。

> ** Note **
> A common misinterpretation of the rule that Aggregates should not expose state is that none of the Entities should contain any property accessor methods. This is not the case. In fact, an Aggregate will probably benefit a lot if the entities *within* the aggregate expose state to the other entities in that same aggregate. However, is is recommended not to expose the state *outside* of the Aggregate.

> ** Note **
> 对规则的常见误解，聚合不应该暴露状态，实体不应包含任何属性访问方法。情况并非如此。事实上，如果聚合中的实体将状态暴露给同一集合中的其他实体，那么聚合可能会受益匪浅。但是，建议不要暴露状态在聚合之外。

Axon provides support for event sourcing in complex aggregate structures. Entities are, just like the Aggregate Root, simple objects. The field that declares the child entity must be annotated with `@AggregateMember`. This annotation tells Axon that the annotated field contains a class that should be inspected for Command and Event Handlers.

Axon为复杂聚合结构提供事件溯源支持。实体就像聚合根一样，是简单的对象。声明子实体的字段必须注明`@aggregatemember`。这个注释告诉Axon，带注释的字段包含一个应该检查命令和事件处理程序的类。

When an Entity (including the Aggregate Root) applies an Event, it is handled by the Aggregate Root first, and then bubbles down through all `@AggregateMember` annotated fields to its child entities.

当一个实体（包括总根）响应一个事件，它是由总根先处理，然后通过`@aggregatemember`注释字段的子实体。

Fields that (may) contain child entities must be annotated with `@AggregateMember`. This annotation may be used on a number of field types:

-   the Entity Type, directly referenced in a field;

-   inside fields containing an `Iterable` (which includes all collections, such as `Set`, `List`, etc);

-   inside the values of fields containing a `java.util.Map`


字段（可能）包含子实体必须注明`@aggregatemember`。此注释可用于许多字段类型：字段到其子实体。

-   实体类型，直接引用在一个字段中;

-   实体在`Iterable`的字段中 (which includes all collections, such as `Set`, `List`, etc);

-   实体字段在 `java.util.Map`中。

### Handling commands in an Aggregate

It is recommended to define the Command Handlers directly in the Aggregate that contains the state to process this command, as it is not unlikely that a command handler needs the state of that Aggregate to do its job. 

To define a Command Handler in an Aggregate, simply annotate the Command Handling method with `@CommandHandler`. The rules for an `@CommandHandler` annotated method are the same as for any handler method. However, Commands are not only routed by their payload. Command Messages carry a name, which defaults to the fully qualified class name of the Command object.

By default, `@CommandHandler` annotated methods allow the following parameter types:

-   The first parameter is the payload of the Command Message. It may also be of type `Message` or `CommandMessage`, if the `@CommandHandler` annotation explicitly defined the name of the Command the handler can process. By default, a Command name is the fully qualified class name of the Command's payload.

-   Parameters annotated with `@MetaDataValue` will resolve to the Meta Data value with the key as indicated on the annotation. If `required` is `false` (default), `null` is passed when the meta data value is not present. If `required` is `true`, the resolver will not match and prevent the method from being invoked when the meta data value is not present.

-   Parameters of type `MetaData` will have the entire `MetaData` of a `CommandMessage` injected.

-   Parameters of type `UnitOfWork` get the current Unit of Work injected. This allows command handlers to register actions to be performed at specific stages of the Unit of Work, or gain access to the resources registered with it.

- Parameters of type `Message`, or `CommandMessage` will get the complete message, with both the payload and the Meta Data. This is useful if a method needs several meta data fields, or other properties of the wrapping Message.

In order for Axon to know which instance of an Aggregate type should handle the Command Message, the property carrying the Aggregate Identifier in the Command object must be annotated with `@TargetAggregateIdentifier`. The annotation may be placed on either the field or an accessor method (e.g. a getter).

Commands that create an Aggregate instance do not need to identify the target aggregate identifier, although it is recommended to annotate the Aggregate identifier on them as well. 

If you prefer to use another mechanism for routing Commands, the behavior can be overridden by supplying a custom `CommandTargetResolver`. This class should return the Aggregate Identifier and expected version (if any) based on a given command.

> **Note**
>
> When the `@CommandHandler` annotation is placed on an Aggregate's constructor, the respective command will create a new instance of that aggregate and add it to the repository. Those commands do not require to target a specific aggregate instance. Therefore, those commands do not require any `@TargetAggregateIdentifier` or `@TargetAggregateVersion` annotations, nor will a custom `CommandTargetResolver` be invoked for these commands.
>
> When a command creates an aggregate instance, the callback for that command will receive the aggregate identifier when the command executed successfully.

``` java
public class MyAggregate {

    @AggregateIdentifier
    private String id;

    @CommandHandler
    public MyAggregate(CreateMyAggregateCommand command) {
        apply(new MyAggregateCreatedEvent(IdentifierFactory.getInstance().generateIdentifier()));
    }

    // no-arg constructor for Axon
    MyAggregate() {
    }

    @CommandHandler
    public void doSomething(DoSomethingCommand command) {
        // do something...
    }

    // code omitted for brevity. The event handler for MyAggregateCreatedEvent must set the id field
}

public class DoSomethingCommand {

    @TargetAggregateIdentifier
    private String aggregateId;

    // code omitted for brevity

}
```

The Axon Configuration API can be used configure the Aggregate. For example:

```
Configurer configurer = ...
// to use defaults:
configurer.configureAggreate(MyAggregate.class);

// allowing customizations:
configurer.configureAggregate(
        AggregateConfigurer.defaultConfiguration(MyAggregate.class)
                           .configureCommandTargetResolver(c -> new CustomCommandTargetResolver())
);
```

`@CommandHandler` annotations are not limited to the aggregate root. Placing all command handlers in the root will sometimes lead to a large number of methods on the aggregate root, while many of them simply forward the invocation to one of the underlying entities. If that is the case, you may place the `@CommandHandler` annotation on one of the underlying entities' methods. For Axon to find these annotated methods, the field declaring the entity in the aggregate root must be marked with `@AggregateMember`. Note that only the declared type of the annotated field is inspected for Command Handlers. If a field value is null at the time an incoming command arrives for that entity, an exception is thrown.

    public class MyAggregate {

        @AggregateIdentifier
        private String id;

        @AggregateMember
        private MyEntity entity;

        @CommandHandler
        public MyAggregate(CreateMyAggregateCommand command) {
            apply(new MyAggregateCreatedEvent(...);
        }

        // no-arg constructor for Axon
        MyAggregate() {
        }

        @CommandHandler
        public void doSomething(DoSomethingCommand command) {
            // do something...
        }

        // code omitted for brevity. The event handler for MyAggregateCreatedEvent must set the id field
        // and somewhere in the lifecycle, a value for "entity" must be assigned to be able to accept
        // DoSomethingInEntityCommand commands.
    }

    public class MyEntity {

        @CommandHandler
        public void handleSomeCommand(DoSomethingInEntityCommand command) {
            // do something
        }
    }

> **Note**
>
> Note that each command must have exactly one handler in the aggregate. This means that you cannot annotate multiple entities (either root nor not) with @CommandHandler, that handle the same command type. In case you need to conditionally route a command to an entity, the parent of these entities should handle the command, and forward it based on the conditions that apply.
>
> The runtime type of the field does not have to be exactly the declared type. However, only the declared type of the `@AggregateMember` annotated field is inspected for `@CommandHandler` methods.

It is also possible to annotate Collections and Maps containing entities with `@AggregateMember`. In the latter case, the values of the map are expected to contain the entities, while the key contains a value that is used as their reference.

As a command needs to be routed to the correct instance, these instances must be properly identified. Their "id" field must be annotated with `@EntityId`. The property on the command that will be used to find the Entity that the message should be routed to, defaults to the name of the field that was annotated. For example, when annotating the field "myEntityId", the command must define a property with that same name. This means either a `getMyEntityId` or a `myEntityId()` method must be present. If the name of the field and the routing property differ, you may provide a value explicitly using `@EntityId(routingKey = "customRoutingProperty")`.

If no Entity can be found in the annotated Collection or Map, Axon throws an IllegalStateException; apparently, the aggregate is not capable of processing that command at that point in time.

> **Note**
>
> The field declaration for both the Collection or Map should contain proper generics to allow Axon to identify the type of Entity contained in the Collection or Map. If it is not possible to add the generics in the declaration (e.g. because you're using a custom implementation which already defines generic types), you must specify the type of entity used in the `entityType` property on the `@AggregateMember` annotation.

### External Command Handlers

In certain cases, it is not possible, or desired to route a command directly to an Aggregate instance. In such case, it is possible to register a Command Handler object.

A Command Handler object is a simple (regular) object, which has `@CommandHandler` annotated methods. Unlike in the case of an Aggregate, there is only a single instance of a Command Handler object, which handles all commands of the types it declares in its methods.

``` java
public class MyAnnotatedHandler {

    @CommandHandler
    public void handleSomeCommand(SomeCommand command, @MetaDataValue("userId") String userId) {
        // whatever logic here
    }

    @CommandHandler(commandName = "myCustomCommand")
    public void handleCustomCommand(SomeCommand command) {
       // handling logic here
    }

}

// To register the annotated handlers to the command bus:
Configurer configurer = ...
configurer.registerCommandHandler(c -> new MyAnnotatedHandler());
```
