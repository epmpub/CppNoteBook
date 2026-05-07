ByteRange和IOBuf 转换

- **folly::ByteRange = folly::Range<const uint8_t>**：**只读、轻量、不拥有内存、不管理生命周期**，只是「指针 + 长度」的视图。
- **folly::IOBuf = 可拥有 / 可共享内存、带引用计数、链式、可读写的缓冲区**，**负责生命周期管理**。
- **关联：IOBuf 是数据的 “持有者”，ByteRange 是它的 “只读视图”**，IOBuf 可以零拷贝生成 ByteRange，ByteRange 绝不拥有内存。

------

## 一、类型定义（最简）

```
// ByteRange（本质就是这个）
namespace folly {
  using ByteRange = Range<const uint8_t>;
}

// Range 内部只有两个成员：
template <typename T>
struct Range {
  T* data_;
  size_t size_;
};
```

- **ByteRange**：`(const uint8_t*, size_t)`，**只读、无所有权、无引用计数、不析构内存**。

- **IOBuf**：

  - 有 `data()/length()` → 可读区
  - 有 `writableData()/tailroom()` → 可写区
  - **引用计数 + 自定义 deleter** → 管生命周期
  - **链式（next/prev）** → 多段零拷贝。

  

------

## 二、核心关联：IOBuf → ByteRange（零拷贝视图）

### 1. 从单个 IOBuf 取 ByteRange

```
std::unique_ptr<IOBuf> buf = IOBuf::create(1024);
// ... 写入数据 ...

// 零拷贝生成 ByteRange（只读视图）
folly::ByteRange br = buf->coalesceAsByteRange();
// 或
folly::ByteRange br2(buf->data(), buf->length());
```

- **不拷贝数据**，只拿指针和长度。
- **br 生命周期不能超过 buf**，否则野指针。

### 2. 从 IOBuf 链式取多个 ByteRange

```
IOBuf* chain = ...; // 多段链
std::vector<ByteRange> ranges;
for (IOBuf* p = chain; p; p = p->next()) {
  ranges.emplace_back(p->data(), p->length());
}
```

- 每一段 IOBuf 对应一个 ByteRange。
- 网络发送时可直接把 `ranges` 转成 `iovec` 给 `writev`。

------

## 三、反向：ByteRange → IOBuf（两种方式）

### 1. 零拷贝包装（不拥有内存）

```
folly::ByteRange br = ...;
// 包装外部内存，IOBuf 不负责释放
auto buf = IOBuf::wrapBuffer(br.data(), br.size());
```

- **危险**：`buf` 析构时**不释放内存**，你要保证 `br` 生命周期更长。

### 2. 拷贝生成（安全，有所有权）

```
// 拷贝 ByteRange 到新 IOBuf（有引用计数，安全）
auto buf = IOBuf::copyBuffer(br.data(), br.size());
```

------

## 四、对比表（一眼分清）

|     特性     |     folly::ByteRange     |        folly::IOBuf         |
| :----------: | :----------------------: | :-------------------------: |
|  内存所有权  |       ❌ 无（视图）       |      ✅ 有（引用计数）       |
|     读写     |           只读           |          可读可写           |
| 生命周期管理 |         ❌ 不管理         |        ✅ 析构 / 共享        |
|     链式     |          ❌ 单段          |         ✅ 多段链表          |
|    零拷贝    |       ✅ 天生零拷贝       |     ✅ 可生成零拷贝视图      |
|   典型用途   | 参数传递、解析、只读访问 | 网络 buffer、数据持有、转发 |

------

## 五、典型使用场景（关联最常见）

1. **网络收包 → IOBuf → 解析时用 ByteRange**

   

   - IOBuf 管理内核 / 网卡数据生命周期。
   - 解析协议时，切出多个 ByteRange 做字段解析，**全程零拷贝**。

   

2. **函数参数用 ByteRange，内部用 IOBuf 持有**

   ```
   void process(folly::ByteRange br) {
     auto buf = IOBuf::copyBuffer(br.data(), br.size());
     // ... 异步处理，buf 自己管生命周期 ...
   }
   ```

   

3. **IOBufQueue 批量转 ByteRange 数组**

   

   - 发给 `writev`/`sendmsg`，**零拷贝发送整条链**。

   

------

## 六、一句话总结

- **IOBuf 是带生命周期、链式、可读写的内存管家**。
- **ByteRange 是它的只读、无所有权、轻量指针视图**。
- **IOBuf 可以零拷贝生成 ByteRange；ByteRange 可以包装或拷贝成 IOBuf**。

需要我给你一段可直接编译的示例代码，演示 IOBuf 与 ByteRange 的互相转换和零拷贝用法吗？