#### Abseil  核心容器

Abseil 还提供了以下几类核心容器，主要用于优化 **小对象、动态数组、字符串、迭代视图** 等场景，性能和内存效率普遍优于 `std::` 对应物。

------

## 一、序列容器（替代 std::vector /std::deque）

### 1. `absl::InlinedVector`

**核心定位**：**小尺寸优化（SSO）的动态数组** —— 栈上预存 N 个元素，超过才堆分配。

- **对应 std**：`std::vector`

- **核心优势**

  - **0 堆分配**：N 个以内元素完全在栈上，速度极快、无碎片
  - 接口与 `std::vector` **几乎完全兼容**
  - 比 `std::vector` 快 **2–10 倍**（小数据量）

- **典型用法**

  ```
  // 栈上预存 4 个 int，超过才扩容到堆
  absl::InlinedVector<int, 4> vec;
  ```

- **最佳场景**

  - 长度 **经常很短**（0~ 几十个元素）
  - 临时数组、函数返回值、高频创建销毁的小列表

### 2. `absl::FixedArray`

**核心定位**：**编译期 / 运行期定长、连续内存、不可扩容** 的数组Abseil。

- **对应 std**：`std::array`（但支持运行期大小）
- **核心优势**
  - 内存绝对紧凑、无额外开销
  - 支持 **运行期指定大小**（`std::array` 必须编译期）
  - 不支持 `push_back` / `erase`，**纯静态访问**
- **场景**：大小已知、一次性分配、纯遍历 / 随机访问

------

## 二、字符串与字节流容器

### 1. `absl::Cord`

**核心定位**：**大块数据、写时复制（COW）、分段存储的字符串 / 字节流**。

- **对应 std**：`std::string`（超大数据场景）
- **核心优势**
  - 超大字符串（MB/GB 级）**拷贝近乎 O (1)**（只复制元数据）
  - 拼接、子串、裁剪 **极快**（不移动大块内存）
  - 适合日志、网络缓冲区、大文件内容
- **场景**
  - 长文本、二进制数据、IO 缓冲区、频繁拼接 / 切片

### 2. `absl::Span<T>`

**核心定位**：**非拥有式连续内存视图** —— 指向一段 `T*` + 长度，不管理内存。

- **对应 std**：`std::span`（C++20），Abseil 提前提供兼容 C++11/14

- **核心优势**

  - 统一接口：可接收 `vector`/`array`/`InlinedVector`/ 裸指针
  - 函数参数首选：避免拷贝、安全、清晰表达 “一段数据”

- **用法**

  ```
  void Print(absl::Span<const int> s);
  Print({1,2,3});
  Print(vec);
  ```

  

------

## 三、有序 / 集合增强容器

### 1. `absl::btree_multimap` / `absl::btree_multiset`

- 就是 **允许重复 key** 的 B 树版 `map/set`Abseil
- 对应：`std::multimap` / `std::multiset`
- 优势：比 std 红黑树 **更快遍历、更省内存、缓存更友好**

### 2. `absl::flat_hash_map` / `node_hash_map` 变体

- **`absl::flat_hash_map` / `flat_hash_set`**（默认首选）
- **`absl::node_hash_map` / `node_hash_set`**（指针稳定）
- 均为 **无序哈希表**，性能碾压 `std::unordered_map`

------

## 四、工具型 / 视图型容器（不存数据，只提供视图）

### 1. `absl::optional<T>`

- 表示 “**有 T 或没有**” 的可空对象
- 对应：C++17 `std::optional`，Abseil 向下兼容 C++11

### 2. `absl::variant<...>`

- **类型安全的联合体**：保存多种类型中的一种
- 对应：C++17 `std::variant`

### 3. `absl::any`

- **类型擦除容器**：可存任意类型，运行时查询类型
- 对应：`std::any`

------

## 五、完整对比：Abseil 容器 vs std 容器

|              Abseil 容器              |          对应 std           |           核心优势           |      首选场景      |
| :-----------------------------------: | :-------------------------: | :--------------------------: | :----------------: |
|        **`flat_hash_map/set`**        |     `unordered_map/set`     |   最快、连续内存、缓存友好   |    95% 无序键值    |
|        **`node_hash_map/set`**        |     `unordered_map/set`     |       指针 / 引用稳定        | 大对象、需长期引用 |
| **`btree_map/set/multimap/multiset`** | `map/set/multimap/multiset` |  有序、范围查询、快于红黑树  | 有序遍历、区间查找 |
|        **`InlinedVector<N>`**         |          `vector`           |   小数据栈上、零分配、超快   |  短列表、临时变量  |
|           **`FixedArray`**            |           `array`           |  运行期定长、紧凑、不可扩容  |  大小固定、纯访问  |
|              **`Cord`**               |          `string`           | 大块数据、拷贝 O (1)、拼接快 | 长文本、IO 缓冲区  |
|             **`Span<T>`**             |           `span`            |    非拥有、统一视图、安全    | 函数参数、数组视图 |
|        `optional/variant/any`         |         C++17 对应          |        向下兼容 C++11        |  可空值、多态存储  |

------

## 六、一句话选择指南（工程版）

1. **无序键值**

   

   - 普通小对象 → **`flat_hash_map`**
   - 大对象 / 要指针稳定 → **`node_hash_map`**

   

2. **有序 / 范围查询**

   

   - → **`btree_map/set`**（替代 `std::map`）

   

3. **动态数组**

   

   - 经常很短（<32）→ **`InlinedVector`**
   - 一般情况 → 仍可用 `std::vector`（也可换 `InlinedVector`）

   

4. **大块字符串 / 字节**

   

   - → **`Cord`**（替代大 `std::string`）

   

5. **函数参数传数组 / 字符串**

   

   - → **`Span`** / `string_view`