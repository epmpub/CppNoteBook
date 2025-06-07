Windows 批处理（BAT）脚本本身没有内置的复杂数据结构（如数组、列表、字典等），像现代编程语言（如 Python 或 PowerShell）那样。批处理的本质是简单的命令行脚本，主要用于自动化任务，依赖环境变量来存储和操作数据。不过，通过一些技巧，可以模拟基本的数据结构，如数组和键值对。以下是详细说明和实现方法：

------

1. 基本数据存储：环境变量

- **描述**:

  - 批处理主要使用环境变量来存储数据。
  - 用 set 命令定义变量，值通常是字符串，但可以用 set /a 处理整数。

- **示例**:

  cmd

  ```text
  @echo off
  set "name=John"
  set "age=25"
  echo Name: %name%
  echo Age: %age%
  pause
  ```

  - **输出**:

    ```text
    Name: John
    Age: 25
    Press any key to continue . . .
    ```

- **局限**:

  - 变量是全局的，除非用 setlocal 和 endlocal 限制作用域。
  - 没有内置的数组或复杂结构，需要手动模拟。

------

2. 模拟数组

- **描述**:

  - 批处理没有真正的数组，但可以通过变量名加索引的方式模拟一维数组。
  - 命名约定：用一个前缀加上数字索引（如 list1, list2, ...）。

- **实现**:

  - 用 set 定义“数组元素”。
  - 用循环（如 for）遍历或操作。

- **示例：定义和访问数组**

  cmd

  ```text
  @echo off
  setlocal EnableDelayedExpansion
  :: 定义数组
  set "fruits0=Apple"
  set "fruits1=Banana"
  set "fruits2=Orange"
  set "count=3"
  
  :: 遍历数组
  for /l %%i in (0,1,%count%-1) do (
    echo Element %%i: !fruits%%i!
  )
  endlocal
  pause
  ```

  - **输出**:

    ```text
    Element 0: Apple
    Element 1: Banana
    Element 2: Orange
    Press any key to continue . . .
    ```

  - **解释**:

    - set "fruits0=Apple": 模拟数组元素，索引从 0 开始。
    - for /l %%i in (0,1,%count%-1): 循环从 0 到 count-1。
    - !fruits%%i!: 动态组合变量名（fruits0, fruits1, ...），用延迟扩展访问值。

- **动态填充数组**:

  cmd

  ```text
  @echo off
  setlocal EnableDelayedExpansion
  set "index=0"
  for %%f in (Apple Banana Orange) do (
    set "fruits!index!=%%f"
    set /a index+=1
  )
  
  :: 遍历数组
  for /l %%i in (0,1,%index%-1) do (
    echo Element %%i: !fruits%%i!
  )
  endlocal
  pause
  ```

  - **输出**: 同上。
  - **解释**:
    - for %%f in (Apple Banana Orange): 遍历一组值。
    - set "fruits!index!=%%f": 动态创建 fruits0, fruits1, 等。
    - set /a index+=1: 递增索引。

- **用途**:

  - 存储文件列表、用户输入、选项等。

- **局限**:

  - 手动管理索引，容易出错。
  - 性能较差，处理大量数据时不高效。

------

3. 模拟键值对（关联数组/字典）

- **描述**:

  - 批处理没有内置的键值对结构，但可以用变量名模拟“键”，变量值作为“值”。
  - 命名约定：用前缀加键名（如 data_key）。

- **实现**:

  - 用 set 定义键值对。
  - 用变量名访问特定键的值。

- **示例**:

  cmd

  ```text
  @echo off
  setlocal EnableDelayedExpansion
  :: 定义键值对
  set "person_name=John"
  set "person_age=25"
  set "person_city=New York"
  
  :: 访问键值
  echo Name: !person_name!
  echo Age: !person_age!
  echo City: !person_city!
  
  :: 动态设置和访问
  set "key=age"
  echo Value of !key!: !person_%key%!
  endlocal
  pause
  ```

  - **输出**:

    ```text
    Name: John
    Age: 25
    City: New York
    Value of age: 25
    Press any key to continue . . .
    ```

  - **解释**:

    - set "person_name=John": 用变量名模拟键（name）和值（John）。
    - !person_%key%!: 动态组合变量名（person_age），访问对应值。

- **用途**:

  - 存储配置数据（如用户名和属性）。
  - 映射关系（如 ID 到名称）。

- **局限**:

  - 键名必须是合法的变量名部分（避免空格、特殊字符）。
  - 无法直接遍历所有键，需预先知道键名或用技巧。

- **遍历技巧**:

  - 用 set 命令列出以某前缀开头的变量：

  cmd

  ```text
  @echo off
  setlocal EnableDelayedExpansion
  set "person_name=John"
  set "person_age=25"
  set "person_city=New York"
  
  :: 列出所有以 "person_" 开头的变量
  for /f "tokens=1,* delims==" %%a in ('set person_') do (
    echo Key: %%a, Value: %%b
  )
  endlocal
  pause
  ```

  - **输出**:

    ```text
    Key: person_age, Value: 25
    Key: person_city, Value: New York
    Key: person_name, Value: John
    Press any key to continue . . .
    ```

  - **解释**:

    - set person_: 列出以 person_ 开头的所有变量。
    - for /f "tokens=1,* delims==": 解析 set 输出，%%a 是变量名，%%b 是值。

------

4. 其他数据结构模拟

- **列表（变体）**:

  - 可以用单个变量存储以分隔符（例如空格、逗号）分隔的值。

  - 示例:

    cmd

    ```text
    @echo off
    set "list=Apple,Banana,Orange"
    for %%i in (%list%) do (
      echo Item: %%i
    )
    pause
    ```

    - **输出**:

      ```text
      Item: Apple
      Item: Banana
      Item: Orange
      Press any key to continue . . .
      ```

    - **局限**: 分隔符（如逗号）不能出现在值中，否则解析错误。

- **栈/队列**:

  - 用数组模拟，结合索引操作。

  - 栈：后进先出（LIFO），用索引添加/移除末尾元素。

  - 队列：先进先出（FIFO），用索引移除头部，添加末尾。

  - 示例（简单栈）:

    cmd

    ```text
    @echo off
    setlocal EnableDelayedExpansion
    set "index=0"
    :: 压入栈
    set "stack!index!=Apple"
    set /a index+=1
    set "stack!index!=Banana"
    set /a index+=1
    :: 弹出栈
    set /a index-=1
    echo Popped: !stack%index%!
    endlocal
    pause
    ```

    - **输出**:

      ```text
      Popped: Banana
      Press any key to continue . . .
      ```

------

5. 局限与注意事项

- **局限**:
  - **性能**: 批处理处理大量数据时效率低，适合小型任务。
  - **类型**: 变量值本质是字符串，set /a 可处理整数，但无浮点数或复杂类型。
  - **复杂性**: 没有内置多维数组、链表、树等，模拟复杂且易出错。
  - **分隔符**: 列表或值中若含空格、逗号、特殊字符（&, |, >），需转义（用 ^）或用引号。
- **建议**:
  - 用 setlocal EnableDelayedExpansion 动态访问变量。
  - 始终用引号（set "var=value"）避免空格问题。
  - 对于复杂数据结构，考虑 PowerShell 或其他语言。

------

6. 总结

- **基本存储**: 环境变量（set "var=value"）。
- **数组**: 模拟为 var0, var1, ...，用循环和索引访问。
- **键值对**: 模拟为 prefix_key=value，用变量名动态访问。
- **其他**: 列表（分隔值）、栈/队列（用索引操作）。
- **用途**: 小型数据管理，如文件列表、配置项、简单计数。

如果你需要具体实现（例如存储文件列表、映射用户数据），告诉我你的需求，我可以帮你写脚本！