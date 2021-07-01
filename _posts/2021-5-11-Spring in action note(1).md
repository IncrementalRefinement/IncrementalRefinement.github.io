# What We Talk About When We Talk About Spring --- 当我们谈论Spring时我们在谈论什么
最近在看Spring in action, 以及相关的内容, 遂整理一下学习的收获. 本文为这一系列开个头, 标题模仿雷蒙德卡佛的成名作.

## 什么是Spring
首先给出Spring官方文档的介绍, 抛玉引砖 --- [What We Mean By "Spring"](https://docs.spring.io/spring-framework/docs/current/reference/html/overview.html)

> The term "Spring" means different things in different contexts. It can be used to refer to the Spring Framework project itself, which is where it all started. Over time, other Spring projects have been built on top of the Spring Framework. Most often, when people say "Spring", they mean the entire family of projects. This reference documentation focuses on the foundation: the Spring Framework itself.

> The Spring Framework is divided into modules. Applications can choose which modules they need. At the heart are the modules of the core container, including a configuration model and a dependency injection mechanism. Beyond that, the Spring Framework provides foundational support for different application architectures, including messaging, transactional data and persistence, and web. It also includes the Servlet-based Spring MVC web framework and, in parallel, the Spring WebFlux reactive web framework.

承接上面的引用, 在不同的语境下, Spring这个词可以指代不同的内容:
1. **Spring Framework**: 一个开源的框架, 它设计的目的就是解决企业级开发时的复杂度问题, Spring框架并不是只能用于server端的开发, 任何的Java应用程序都可以考虑使用Spring框架来达到优化设计, 方便测试, 松耦合等优点. 简单点说, Spring是一个轻量级IoC和AOP容器框架. 在Spring的基础上开发出了众多Spring modules
2. **Spring modules**: 在Spring的核心容器框架基础上, 许多Spring modules被开发了出来, 这些模块便利了程序的开发, 它们的功能包括消息, web, 数据持久化等等. 

## 为什么要学习Spring   
1. 首先, Spring是一个成功的开源框架, 甚至可以说是当下J2EE的业界标准. Spring将原本耦合度高的各个类拆分开来, 并将类的实例化的责任转给容器, 从而降低了开发的难度和程序员的心智负担, 使得开发大型项目更加轻松以及测试和维护. Spring也通过面向切面编程, 将通用代码和实际代码隔离开.
2. Spring拥有优秀的设计理念和设计模式, 学习Spring可以增强对于软件工程的理解. 深入Spring框架不光能使得构建出的Spring工程更加优美, 还能从提高程序员的自身素养. 就像玩黑魂时, 变强的不光是游戏里的角色, 还有玩家本身. 
