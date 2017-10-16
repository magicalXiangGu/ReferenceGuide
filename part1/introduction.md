Introduction
============

Axon is a lightweight framework that helps developers build scalable and extensible applications by addressing these concerns directly in the architecture. This reference guide explains what Axon is, how it can help you and how you can use it.

Axon是一个轻量级框架，可帮助开发人员在架构层直接构建可伸缩和可扩展的应用程序。
本参考指南解释了Axon是什么，它如何够帮助你和你如何使用它。

If you want to know more about Axon and its background, continue reading in [Axon Framework Background](#axon-framework-background). If you're eager to get started building your own application using Axon, go quickly to [Getting Started](#getting-started). If you're interested in helping out building the Axon Framework, [Contributing](#contributing-to-axon-framework) will contain the information you require. All help is welcome. Finally, this chapter covers some legal concerns in [License](#license-information).

如果你想进一步了解Axon和其背景，继续阅读[Axon Framework Background](#axon-framework-background).如果你想尽快使用Axon构建自己的应用， -> [Getting Started](#getting-started).

Axon Framework Background
=========================

A brief history
---------------

The demands on software projects increase rapidly as time progresses. Companies want their (web)applications to evolve together with their business. That means that not only projects and code bases become more complex, it also means that functionality is constantly added, changed and (unfortunately not enough) removed. It can be frustrating to find out that a seemingly easy-to-implement feature can require development teams to take apart an entire application. Furthermore, today's web applications target the audience of potentially billions of people, making scalability an indisputable requirement.

随着时间的推移软件项目的需求迅速增加。公司希望他们的（Web）应用程序与他们的业务一起进化。这意味着不仅项目和基础代码变得更加复杂，它还意味着功能不断地被添加、改变和（不幸的是不够）被移除。发现一个看似容易实现的特性可能需要开发团队拆开整个应用程序，这可能令人沮丧。此外，今天的Web应用程序瞄准了潜在的数十亿人的受众，使可伸缩性成为一个无可争辩的要求。

Although there are many applications and frameworks around that deal with scalability issues, such as GigaSpaces and Terracotta, they share one fundamental flaw. These stacks try to solve the scalability issues while letting developers develop applications using the layered architecture they are used to. In some cases, they even prevent or severely limit the use of a real domain model, forcing all domain logic into services. Although that is faster to start building an application, eventually this approach will cause complexity to increase and development to slow down.

虽然在解决可扩展性问题有许多应用程序和框架，如GigaSpaces和Terracotta，但他们有一个根本的缺陷。这些程序试图解决可伸缩性问题，同时让开发人员使用它们所使用的分层体系结构开发应用程序。在某些情况下，他们甚至阻止或严格限制使用真实域模型，强制将所有域逻辑引入服务。虽然开始构建应用程序的速度更快，但最终这种方法会使复杂性增加和开发速度减慢。

The Command Query Responsibility Segregation (CQRS) pattern addresses these issues by drastically changing the way applications are architected. Instead of separating logic into separate layers, logic is separated based on whether it is changing an application's state or querying it. That means that executing commands (actions that potentially change an application's state) are executed by different components than those that query for the application's state. The most important reason for this separation is the fact that there are different technical and non-technical requirements for each of them. When commands are executed, the query components are (a)synchronously updated using events. This mechanism of updates through events, is what makes this architecture so extensible, scalable and ultimately more maintainable.

命令查询责任分离（CQRS）模式通过大幅改变应用程序架构的方式来解决这些问题。逻辑被分离根据它是改变应用程序的状态还是查询程序状态，而不是将他分离到单独的层中。这意味着执行命令（可能改变应用程序状态的动作）由与应用程序状态查询相对应的不同组件执行。这种分离的最重要的原因是它们各自都有不同的技术和非技术要求。当执行命令时，查询组件（a）使用事件同步更新。这种通过事件更新的机制使得该架构具有可扩展性，可伸缩性和最终更可维护性。

> **Note**
>
> A full explanation of CQRS is not within the scope of this document. If you would like to have more background information about CQRS, visit the Axon Framework website: [www.axonframework.org](http://www.axonframework.org/). It contains links to background information.

CQRS的完整说明不在本文档的范围内。如果您想获得有关CQRS的更多背景信息，请访问Axon Framework网站：[www.axonframework.org](http://www.axonframework.org/)。它包含背景信息的链接。

Since CQRS is fundamentally different than the layered-architecture which dominates today's software landscape, it is not uncommon for developers to walk into a few traps while trying to find their way around this architecture. That's why Axon Framework was conceived: to help developers implement CQRS applications while focusing on the business logic.

由于CQRS与主导当今软件领域的分层架构有着根本的不同，因此开发人员在尝试围绕这个架构找到解决方案的过程中走进几个陷阱并不罕见。这就是为什么Axon框架被提出：帮助开发人员实施CQRS应用程序，同时专注于业务逻辑。

What is Axon?
-------------

Axon Framework helps build scalable, extensible and maintainable applications by supporting developers apply the Command Query Responsibility Segregation (CQRS) architectural pattern. It does so by providing implementations of the most important building blocks, such as aggregates, repositories and event buses (the dispatching mechanism for events). Furthermore, Axon provides annotation support, which allows you to build aggregates and event listeners without tying your code to Axon specific logic. This allows you to focus on your business logic, instead of the plumbing, and helps you to make your code easier to test in isolation.

Axon Framework通过支持开发人员应用命令查询责任分隔（CQRS）架构模式，帮助构建可扩展，可扩展和可维护的应用程序。它通过提供最重要的模块实现，如聚合，存储库和事件总线（事件的调度机制）来实现。此外，Axon提供注释支持，可以让您构建聚合和事件侦听器，而不必将代码与Axon的特定逻辑相联系。这样，您可以专注于业务逻辑，而不是管道，并帮助您使代码更容易被隔离测试。

Axon does not, in any way, try to hide the CQRS architecture or any of its components from developers. Therefore, depending on team size, it is still advisable to have one or more developers with a thorough understanding of CQRS on each team. However, Axon does help when it comes to guaranteeing delivering events to the right event listeners and processing them concurrently and in the correct order. These multi-threading concerns are typically hard to deal with, leading to hard-to-trace bugs and sometimes complete application failure. When you have a tight deadline, you probably don't even want to care about these concerns. Axon's code is thoroughly tested to prevent these types of bugs.

Axon不以任何方式试图将CQRS架构或其任何组件从开发人员中隐藏起来。因此，根据团队规模，建议在每个团队中让一个或多个开发人员对CQRS进行彻底的了解。然而，Axon确实帮助确保将事件传递到正确的事件监听器并且以正确的顺序处理它们。这些多线程问题通常很难处理，导致难以追踪的错误，有时候导致应用程序的故障。当你的截止日期很紧，你可能甚至不想关心这些问题。 Axon的代码经过彻底测试，以防止这些类型的错误。

The Axon Framework consists of a number of modules (jars) that provide the tools and components to build a scalable infrastructure. The Axon Core module provides the basic APIs for the different components, and simple implementations that provide solutions for single-JVM applications. The other modules address scalability or high-performance issues, by providing specialized building blocks.

Axon框架由许多模块（jar）组成，提供用于构建可扩展基础架构的工具和组件。 Axon Core模块提供了不同组件的基本API，以及为single-JVM应用程序提供解决方案的简单实现。其他模块通过提供专门的构建块来解决可扩展性或高性能问题。


When to use Axon?
-----------------

Not every application will benefit from Axon. Simple CRUD (Create, Read, Update, Delete) applications which are not expected to scale will probably not benefit from CQRS or Axon. However, there is a wide variety of applications that do benefit from Axon.
并非每个应用程序都会从Axon受益。简单的CRUD（创建、读取、更新、删除）没有预计规模的程序不会从cqrs或Axon受益。然而，有各种各样的应用程序确实得益于Axon。


Applications that will likely benefit from CQRS and Axon are those that show one or more of the following characteristics:

具有以下一个或多个特征的应用程序可能会从cqrs和Axon效益显示：

-   The application is likely to be extended with new functionality during a long period of time. For example, an online store might start off with a system that tracks progress of Orders. At a later stage, this could be extended with Inventory information, to make sure stocks are updated when items are sold. Even later, accounting can require financial statistics of sales to be recorded, etc. Although it is hard to predict how software projects will evolve in the future, the majority of this type of application is clearly presented as such.

该应用程序可能会在长时间内被扩展新的功能。例如，在线商店可能开始会有跟踪订单进度的系统。在稍后阶段，可能对库存信息进行扩展，以确保在出售物品时更新库存。甚至以后，会计可能要求从销售记录进行财务统计等。虽然很难预测软件项目将来会如何演变，但大多数这种应用程序都是清晰呈现的。

-   The application has a high read-to-write ratio. That means data is only written a few times, and read many times more. Since data sources for queries are different to those that are used for command validation, it is possible to optimize these data sources for fast querying. Duplicate data is no longer an issue, since events are published when data changes.

-   应用程序具有高读写比。这意味着数据只写了几遍，读很多次。由于查询的数据源与用于命令验证的数据源不同，因此可以优化这些数据源以进行快速查询。重复的数据不再是一个问题，因为事件在数据更改时被发布。

-   The application presents data in many different formats. Many applications nowadays don't stop when showing information on a web page. Some applications, for example, send monthly emails to notify users of changes that occurred that might be relevant to them. Search engines are another example. They use the same data your application does, but in a way that is optimized for quick searching. Reporting tools aggregate information into reports that show data evolution over time. This, again, is a different format of the same data. Using Axon, each data source can be updated independently of each other on a real-time or scheduled basis.

-   该应用程序呈现出许多不同格式的数据。当在网页上显示信息时，现在许多应用程序都不会停止。例如，有些应用程序每月发送电子邮件通知用户可能发生的与他们相关的更改。搜索引擎是另一个例子。他们使用您的应用程序所使用的相同数据，但以某种方式进行优化，以便快速搜索。报告工具将信息聚合到随时间推移显示数据演进的报告中。这又是相同数据的不同格式。使用Axon，每个数据源可以以实时或计划的方式彼此独立地更新。

-   When an application has clearly separated components with different audiences, it can benefit from Axon, too. An example of such application is the online store. Employees will update product information and availability on the website, while customers place orders and query for their order status. With Axon, these components can be deployed on separate machines and scaled using different policies. They are kept up-to-date using the events, which Axon will dispatch to all subscribed components, regardless of the machine they are deployed on.

-   当应用程序需从不同的受众进行清楚的组件时，也可以从Axon中受益。

-   Integration with other applications can be cumbersome work. The strict definition of an application's API using commands and events makes it easier to integrate with external applications. Any application can send commands or listen to events generated by the application.

Getting started
===============

This section will explain how you can obtain the binaries for Axon to get started. There are currently two ways: either download the binaries from our website or configure a repository for your build system (Maven, Gradle, etc).

Download Axon
-------------

You can download the Axon Framework from our downloads page: [axonframework.org/download](http://www.axonframework.org/download).

This page offers a number of downloads. Typically, you would want to use the latest stable release. However, if you're eager to get started using the latest and greatest features, you could consider using the snapshot releases instead. The downloads page contains a number of assemblies for you to download. Some of them only provide the Axon library itself, while others also provide the libraries that Axon depends on. There is also a "full" zip file, which contains Axon, its dependencies, the sources and the documentation, all in a single download.

If you really want to stay on the bleeding edge of development, you can clone the Git repository: <git://github.com/AxonFramework/AxonFramework.git>, or visit <https://github.com/AxonFramework/AxonFramework> to browse the sources online.

Configure Maven
---------------

If you use maven as your build tool, you need to configure the correct dependencies for your project. Add the following code in your dependencies section:

``` xml
<dependency>
    <groupId>org.axonframework</groupId>
    <artifactId>axon-core</artifactId>
    <version>${axon.version}</version>
</dependency>
```

Most of the features provided by the Axon Framework are optional and require additional dependencies. We have chosen not to add these dependencies by default, as they would potentially clutter your project with artifacts you don't need.

Infrastructure requirements
---------------------------

Axon Framework doesn't impose many requirements on the infrastructure. It has been built and tested against Java 8, making that more or less the only requirement.

Since Axon doesn't create any connections or threads by itself, it is safe to run on an Application Server. Axon abstracts all asynchronous behavior by using `Executor`s, meaning that you can easily pass a container managed Thread Pool, for example. If you don't use a full blown Application Server (e.g. Tomcat, Jetty or a stand-alone app), you can use the `Executors` class or the Spring Framework to create and configure Thread Pools.

When you're stuck
-----------------

While implementing your application, you might run into problems, wonder about why certain things are the way they are, or have some questions that need an answer. The Axon Users mailing list is there to help. Just send an email to [axonframework@googlegroups.com](mailto:axonframework@googlegroups.com). Other users as well as contributors to the Axon Framework are there to help with your issues.

If you find a bug, you can report them at [issues.axonframework.org](http://issues.axonframework.org). When reporting an issue, please make sure you clearly describe the problem. Explain what you did, what the result was and what you expected to happen instead. If possible, please provide a very simple Unit Test (JUnit) that shows the problem. That makes fixing it a lot simpler.

Contributing to Axon Framework
==============================

Development on the Axon Framework is never finished. There will always be more features that we like to include in our framework to continue making development of scalable and extensible applications easier. This means we are constantly looking for help in developing our framework.

There are a number of ways in which you can contribute to the Axon Framework:

-   You can report any bugs, feature requests or ideas for improvements on our issue page: [issues.axonframework.org](http://issues.axonframework.org). All ideas are welcome. Please be as exact as possible when reporting bugs. This will help us reproduce and thus solve the problem faster.

-   If you have created a component for your own application that you think might be useful to include in the framework, send us a patch or a zip containing the source code. We will evaluate it and try to fit it in the framework. Please make sure code is properly documented using javadoc. This helps us to understand what is going on.

-   If you know of any other way you think you can help us, do not hesitate to send a message to the [Axon Framework mailing list](mailto:axonframework@googlegroups.com).

Commercial Support
==================

Axon Framework is open source and freely available for anyone to use. However, if you have specific requirements, or just want to be assured of someone to be standby to help you in case of trouble, AxonIQ provides several commercial support services for Axon Framework. These services include Training, Consultancy and Operational Support and are provided by the people that know Axon more than anyone else.

For more information about AxonIQ and its services, visit [axoniq.io](http://axoniq.io) or [axoniq.io/services](http://axoniq.io/services).

License information
===================

The Axon Framework and its documentation are licensed under the Apache License, Version 2.0. You may obtain a copy of the License at <http://www.apache.org/licenses/LICENSE-2.0>.

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the [License](http://www.apache.org/licenses/LICENSE-2.0) for the specific language governing permissions and limitations under the License.
