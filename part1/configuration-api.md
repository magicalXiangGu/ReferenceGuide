Configuration API
配置API
=================

Axon keeps a strict separation when it comes to business logic and infrastructure configuration. In order to do so, Axon will provide a number of building blocks that take care of the infrastructural concerns, such as transaction management around a message handler. The actual payload of the messages and the contents of the handler are implemented in (as much as possible) Axon-independent Java classes.

Axon在业务逻辑和基础设施配置方面保持严格的分离。为了做到这一点，Axon将提供一些处理基础设施问题的构建块，例如围绕消息处理程序的事务管理。消息的实际有效载荷和处理程序的内容在（尽可能多的）Axon-independent Java类中实现。

To make the configuration of these infrastructure components easier and to define their relationship with each of the functional components, Axon provides a Configuration API.

为了使这些基础设施组件的配置更容易，并定义与每个功能组件的关系，Axon提供了一个Configuration API。

Setting up a configuration
设置配置
--------------------------

Getting a default configuration is very easy:

获取默认配置非常简单：

``` java
Configuration config = DefaultConfigurer.defaultConfiguration()
                                        .buildConfiguration();
```

This configuration provides the building blocks for dispatching Messages using implementations that handle messages on the threads that dispatch them.

此配置提供了在分发消息的线程中处理消息的模块。

Obviously, this configuration would not be very useful. You would have to register your Command Model objects and Event Handlers to this configuration to be useful.

显然，这种配置不是很有用。您将必须注册您的命令模型对象和事件处理器到此配置。

To do so, use the `Configurer` instance returned by the `.defaultConfiguration()` method.

为此，请使用.defaultConfiguration（）方法返回的Configurer实例。

``` java
Configurer configurer = DefaultConfigurer.defaultConfiguration();
```

The configurer provides a multitude of methods that allow you to register these components. How to configure those is described in detail in each component's respective chapter.

配置程序提供了许多方法，允许您注册这些组件。每个组件的相应章节将详细描述如何配置它们。

The general form in which components are registered, is the following:

组件注册的一般形式如下：

``` java
Configurer configurer = DefaultConfigurer.defaultConfiguration();
configurer.registerCommandHandler(c -> doCreateComponent());
```

Note the lambda expression in the `registerCommandBus` invocation. The `c` parameter of this expression is the configuration object that described the complete configuration. If your component requires any other components to function properly, it can use this configuration to retrieve them.

注意registerCommandBus调用中的lambda表达式。此表达式的c参数是描述完整配置的配置对象。如果您的组件需要任何其他组件正常运行，则可以使用此配置来检索它们。

For example, to register a Command Handler that requires a serializer:

例如，要注册一个需要序列化的命令处理程序：

``` java
configurer.registerCommandHandler(c -> new MyCommandHandler(c.serializer());
```

Not all components have their explicit accesor method. To retrieve a component from the 
configuration, use:

并非所有组件都具有其明确的访问方法。要从配置中检索组件，请使用

``` java
configurer.registerCommandHandler(c -> new MyCommandHandler(c.getComponent(MyOtherComponent.class));
```

This component must be registered with the Configurer, using `configurer.registerComponent(componentType, builderFunction)`. The builder function will receive the `Configuration` object as input parameter.

必须使用configurer.registerComponent（componentType，builderFunction）向Configurer注册此组件。构建器函数将接收Configuration对象作为输入参数

Setting up a configuration using Spring
使用Spring设置配置
---------------------------------------

When using Spring, there is no need to explicitly use the `Configurer`. Instead, you can simply put the `@EnableAxon` on one of your Spring `@Configuration` classes.

使用Spring时，不需要明确地使用Configurer。相反，您可以简单地将@EnableAxon放在您的一个Spring @Configuration类中。

Axon will use the Spring Application Context to locate specific implementations of building blocks and provide default for those that are not there. So, instead of registering the building blocks with the `Configurer`, in Spring you just have to make them available in the Application Context as `@Bean`s.

Axon将使用Spring应用程序上下文来定位构建块的特定实现，并为那些不在的提供默认值。所以在Spring中，代替使用Configurer注册构建块，你只需要在应用程序上下文中使用@Bean。