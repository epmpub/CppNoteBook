

# Stereotype（构造型）

UML（统一建模语言）中的 **Stereotype（构造型）** 是一种扩展机制，用于自定义或扩展现有 UML 元素（如类、组件、用例等）的语义和用途。它允许开发者为 UML 元素添加特定领域的含义，从而更精确地描述系统设计。

### 核心概念

1. **扩展 UML 元素**  
   Stereotype 不是创建新的 UML 元素，而是为现有元素（如 `Class`、`Component`）附加额外的语义。例如：
   
   - 用 `<<Controller>>` 表示一个类在 MVC 模式中的角色。
   - 用 `<<Entity>>` 表示持久化存储的领域对象。
   
2. **语法表示**  
   Stereotype 通常以双尖括号包裹的名称表示，放置在 UML 元素名称的上方或旁边。例如：
   
   ```
   <<Service>> 
   OrderManager
   ```
   也可以用特定图标或颜色来可视化 Stereotype。
   
3. **与元模型的关系**  
   Stereotype 属于 UML 的扩展机制（其他机制包括 **Tagged Values** 和 **Constraints**），它通过 UML 的元模型（Meta-Model）定义，允许在特定上下文中重新解释元素的行为。

---

### 主要用途
1. **领域特定建模**  
   在特定领域（如金融、医疗）中，为 UML 元素赋予领域相关的语义。例如：
   - `<<BankAccount>>` 表示一个银行账户类。
   - `<<RESTful>>` 表示一个符合 REST 规范的接口。

2. **架构分层**  
   标识不同架构层的组件，例如：
   - `<<Presentation>>`（表示层）
   - `<<BusinessLogic>>`（业务逻辑层）
   - `<<Persistence>>`（持久层）

3. **设计模式标记**  
   标注设计模式中的角色，例如：
   - `<<Singleton>>`（单例模式）
   - `<<Factory>>`（工厂模式）

4. **代码生成支持**  
   为模型驱动开发（MDD）提供提示，例如：
   - `<<DTO>>`（数据传输对象，提示生成特定代码结构）
   - `<<Repository>>`（数据库访问层模板）。

---

### 示例
```plantuml
@startuml
class User {
  +id: int
  +name: string
}
<<Entity>> User

class UserService {
  +createUser(): void
}
<<Service>> UserService

class UserController {
  +handleRequest(): void
}
<<Controller>> UserController
@enduml
```
- `<<Entity>>` 表示 `User` 类对应数据库中的实体。
- `<<Service>>` 表示 `UserService` 是业务逻辑层的服务类。
- `<<Controller>>` 表示 `UserController` 是处理请求的控制器。

---

### Stereotype vs. 其他扩展机制
1. **Tagged Values**  
   为元素添加键值对形式的附加属性（如 `{version=1.0}`）。
2. **Constraints**  
   定义元素必须满足的规则（如 `{ordered}` 表示集合有序）。
3. **Stereotype**  
   直接扩展元素的语义和用途（如 `<<DAO>>`）。

---

### 总结
Stereotype 是 UML 中灵活且强大的工具，通过为现有元素附加领域或架构相关的语义，使模型更贴近实际需求。它在复杂系统建模、设计模式标注和代码生成中尤其有用，能显著提升模型的可读性和实用性。