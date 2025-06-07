# Python 中常见设计模式

1. https://refactoring.guru/design-patterns
2. https://jellyfish.tech/blog/implementation-of-common-design-patterns-in-python/
3. https://www.geeksforgeeks.org/python-design-patterns/



直接回答

以下是 Python 中除装饰器模式外的常见设计模式，适合初学者理解：

- **关键点**：
  - 研究表明，Python 中常用的设计模式包括创建型、结构型和行为型模式。
  - 这些模式帮助解决软件设计中的常见问题，提升代码可维护性和可扩展性。
  - 不同模式适用场景不同，需根据具体需求选择。

创建型模式

- **单例模式**：确保一个类只有一个实例，如配置管理或数据库连接。
- **工厂方法模式**：定义创建对象的接口，让子类决定实例化哪个类，适合框架扩展。
- **抽象工厂模式**：提供创建相关对象家族的接口，不指定具体类，适合创建多个相关对象。
- **建造者模式**：分步构建复杂对象，允许同一过程创建不同表示，适合复杂对象构造。

结构型模式

- **适配器模式**：让不兼容的接口协同工作，常用在整合第三方库。
- **桥接模式**：将抽象和实现解耦，独立发展，适合大型类或相关类集。
- **组合模式**：将对象组成树状结构，统一处理个体和组合，适合文件系统或组织结构。
- **外观模式**：简化复杂子系统的接口，方便使用。
- **代理模式**：为对象提供替身，控制访问，适合延迟加载或访问控制。

行为型模式

- **观察者模式**：定义对象间一对多的依赖，当一个对象状态改变时通知其他对象，适合事件处理。
- **策略模式**：定义算法家族，封装每个算法，使它们可互换，适合运行时选择行为。
- **模板方法模式**：在超类中定义算法骨架，子类可重写步骤，保持结构不变。
- **迭代器模式**：提供遍历集合元素的方式，不暴露内部表示，Python 内置支持。

这些模式在 Python 中应用广泛，具体使用时需结合实际需求。更多详情可参考 [Refactoring Guru - Design Patterns in Python](https://refactoring.guru/design-patterns/python)、[Jellyfish.tech - Implementation of Common Design Patterns in Python](https://jellyfish.tech/blog/implementation-of-common-design-patterns-in-python/) 和 [GeeksforGeeks - Python Design Patterns](https://www.geeksforgeeks.org/python-design-patterns/)。

------

详细报告

以下是关于 Python 中除装饰器模式外的常见设计模式的全面分析，旨在为开发者提供深入理解和应用的基础。本报告基于近期研究和权威资源，涵盖创建型、结构型和行为型模式，包含具体描述和适用场景。

背景与概述

设计模式是软件开发中解决常见问题的通用可重用解决方案，特别是在对象和类的交互中。Python 作为一种动态、面向对象的语言，支持多种设计模式，这些模式帮助提升代码的可读性、可维护性和可扩展性。根据 [Refactoring Guru - Design Patterns in Python](https://refactoring.guru/design-patterns/python) 和 [GeeksforGeeks - Python Design Patterns](https://www.geeksforgeeks.org/python-design-patterns/)，设计模式可分为三大类：创建型、结构型和行为型。

创建型模式

创建型模式关注对象的创建过程，旨在提供灵活的实例化方式。以下是常见模式：

- **单例模式**：
  - 描述：确保一个类只有一个实例，并提供全局访问点。
  - 适用场景：如配置管理、数据库连接池，确保唯一实例。
  - 示例：日志记录器或全局缓存。
  - 优势：简化全局访问，减少资源浪费。
  - 参考：[Jellyfish.tech - Implementation of Common Design Patterns in Python](https://jellyfish.tech/blog/implementation-of-common-design-patterns-in-python/) 提到单例模式在 Python 中的实现。
- **工厂方法模式**：
  - 描述：在超类中定义创建对象的接口，子类决定实例化哪个类。
  - 适用场景：框架或库扩展，用户可自定义对象类型。
  - 示例：创建不同类型的文档处理器。
  - 优势：符合开放-封闭原则，易于扩展。
- **抽象工厂模式**：
  - 描述：提供创建相关对象家族的接口，不指定具体类。
  - 适用场景：创建多个相关或依赖的对象，如 GUI 组件集。
  - 示例：创建一组匹配的按钮和文本框。
  - 优势：确保对象家族一致性。
- **建造者模式**：
  - 描述：分步构建复杂对象，允许同一过程创建不同表示。
  - 适用场景：构造复杂对象，如 HTML 文档或复杂配置。
  - 示例：逐步构建汽车，允许不同配置。
  - 优势：分离构造和表示，易于修改。

结构型模式

结构型模式关注类和对象的组合，旨在简化系统结构。以下是常见模式：

- **适配器模式**：
  - 描述：让不兼容的接口协同工作。
  - 适用场景：整合第三方库或旧系统接口。
  - 示例：将旧 API 适配为新系统。
  - 优势：提高兼容性，减少修改成本。
- **桥接模式**：
  - 描述：将抽象和实现解耦，独立发展。
  - 适用场景：大型类或相关类集，如图形系统。
  - 示例：分离形状和渲染方式。
  - 优势：提高灵活性，易于扩展。
- **组合模式**：
  - 描述：将对象组成树状结构，统一处理个体和组合。
  - 适用场景：文件系统、组织结构。
  - 示例：文件夹和文件层次结构。
  - 优势：简化客户端代码，统一操作。
- **外观模式**：
  - 描述：提供简化接口，屏蔽复杂子系统。
  - 适用场景：简化 API 使用，如数据库操作。
  - 示例：统一调用多个子系统功能。
  - 优势：降低客户端复杂性。
- **代理模式**：
  - 描述：为对象提供替身，控制访问。
  - 适用场景：延迟加载、访问控制、安全检查。
  - 示例：虚拟代理加载大图片。
  - 优势：控制对象访问，优化性能。

行为型模式

行为型模式关注对象间的通信和职责分配。以下是常见模式：

- **观察者模式**：
  - 描述：定义对象间一对多的依赖，当一个对象状态改变时通知其他对象。
  - 适用场景：事件处理系统，如 GUI 更新。
  - 示例：订阅-发布机制。
  - 优势：松耦合，易于扩展。
- **策略模式**：
  - 描述：定义算法家族，封装每个算法，使它们可互换。
  - 适用场景：运行时选择算法，如排序策略。
  - 示例：不同支付方式选择。
  - 优势：符合开放-封闭原则。
- **模板方法模式**：
  - 描述：在超类中定义算法骨架，子类可重写步骤。
  - 适用场景：算法结构固定，细节可变。
  - 示例：文档生成流程。
  - 优势：复用代码，简化子类实现。
- **迭代器模式**：
  - 描述：提供遍历集合元素的方式，不暴露内部表示。
  - 适用场景：遍历列表、字典等集合。
  - 示例：Python 的 for 循环内置迭代器。
  - 优势：隐藏实现细节，统一接口。

模式分类与对比

以下表格总结各模式类别和特点：

| 类别       | 模式名称     | 主要功能               | 适用场景             |
| ---------- | ------------ | ---------------------- | -------------------- |
| 创建型模式 | 单例模式     | 确保单一实例，全球访问 | 配置管理、数据库连接 |
|            | 工厂方法模式 | 子类决定实例化类       | 框架扩展             |
|            | 抽象工厂模式 | 创建对象家族           | GUI 组件集           |
|            | 建造者模式   | 分步构建复杂对象       | 复杂对象构造         |
| 结构型模式 | 适配器模式   | 兼容不匹配接口         | 整合第三方库         |
|            | 桥接模式     | 抽象与实现解耦         | 大型类系统           |
|            | 组合模式     | 树状结构统一操作       | 文件系统             |
|            | 外观模式     | 简化复杂子系统接口     | API 封装             |
|            | 代理模式     | 控制对象访问           | 延迟加载、安全检查   |
| 行为型模式 | 观察者模式   | 状态改变通知依赖对象   | 事件处理             |
|            | 策略模式     | 算法家族可互换         | 运行时选择算法       |
|            | 模板方法模式 | 算法骨架，子类定制细节 | 流程固定，细节可变   |
|            | 迭代器模式   | 遍历集合，不暴露内部   | 集合遍历             |

适用性与选择

根据 [Jellyfish.tech - Implementation of Common Design Patterns in Python](https://jellyfish.tech/blog/implementation-of-common-design-patterns-in-python/)，选择设计模式需考虑：

- **重复问题**：当遇到常见设计问题时，使用已验证的模式。
- **灵活性与可重用性**：模式促进代码重用和维护。
- **设计原则**：如关注点分离、封装、依赖倒置。
- **团队沟通**：模式提供标准术语，提升协作效率。

结论

以上列出的模式是 Python 中除装饰器模式外的常见设计模式，涵盖创建、结构和行为三大类。每个模式有其独特优势和适用场景，开发者可根据具体需求选择。更多深入学习可参考相关资源，如 [Refactoring Guru - Design Patterns in Python](https://refactoring.guru/design-patterns/python) 和 [GeeksforGeeks - Python Design Patterns](https://www.geeksforgeeks.org/python-design-patterns/)。

------

关键引文

- [Refactoring Guru Design Patterns in Python catalog](https://refactoring.guru/design-patterns/python)
- [Jellyfish tech Implementation of Common Design Patterns in Python](https://jellyfish.tech/blog/implementation-of-common-design-patterns-in-python/)
- [GeeksforGeeks Python Design Patterns tutorial](https://www.geeksforgeeks.org/python-design-patterns/)