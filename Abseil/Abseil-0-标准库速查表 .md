# Abseil ↔ C++ 标准库 速查表

## 1. 基础类型与容器

| Abseil                 |      对应 C++ 标准       |           说明 & 优势            |
| ---------------------- | :----------------------: | :------------------------------: |
| `absl::string_view`    | C++17 `std::string_view` |   Abseil 更早实现，兼容 C++11    |
| `absl::optional<T>`    |  C++17 `std::optional`   |    更早、更稳定，错误处理友好    |
| `absl::variant<Ts...>` |   C++17 `std::variant`   |             更早实现             |
| `absl::any`            |     C++17 `std::any`     |               同上               |
| `absl::flat_hash_map`  |   `std::unordered_map`   | **性能碾压**，缓存友好、内存更小 |
| `absl::flat_hash_set`  |   `std::unordered_set`   |               同上               |
| `absl::node_hash_map`  |   `std::unordered_map`   |     稳定迭代器，支持键不可变     |
| `absl::node_hash_set`  |   `std::unordered_set`   |               同上               |
| `absl::btree_set/map`  |      `std::set/map`      | 平衡树，范围查询更快、内存更紧凑 |
| `absl::InlinedVector`  |      `std::vector`       |     小对象栈上存储，减少分配     |
| `absl::Span<T>`        |    C++20 `std::span`     |   安全视图，替代裸指针 + 长度    |

## 2. 字符串与格式化 - ✅ 

| Abseil                        |         对应标准用法         |                优势                |
| :---------------------------- | :--------------------------: | :--------------------------------: |
| `absl::StrCat`                |     `std::ostringstream`     |        一行拼接，极快、简洁        |
| absl::StrReplaceAll           |             ？？             |                ？？                |
| absl::StrContains             |             ？？             |                ？？                |
| `absl::StrAppend`             |        `+=`/`append`         |              高效追加              |
| `absl::StrFormat`             | `printf`/C++20 `std::format` | 类型安全、不崩程序、支持自定义类型 |
| `absl::StrSplit`              |           手写循环           |   一行分割字符串，支持多种分隔符   |
| `absl::StripPrefix/Suffix`    |         手动 substr          |           安全、不抛异常           |
| `absl::AsciiStrToLower/Upper` |         cctype 函数          |       线程安全、只处理 ASCII       |

## 3. 时间与计时

|          Abseil           |                对应标准                 |           特点           |
| :-----------------------: | :-------------------------------------: | :----------------------: |
|       `absl::Time`        | `std::chrono::system_clock::time_point` | 无歧义、纳秒、跨平台一致 |
|     `absl::Duration`      |         `std::chrono::duration`         |    更易用，无单位陷阱    |
|       `absl::Now()`       |   `std::chrono::system_clock::now()`    |           简洁           |
| `absl::Seconds/Minutes/`… |         `std::chrono::seconds`…         |         写法更短         |

## 4. 命令行 & 配置



|           Abseil           |          对应标准 / 常见库          |
| :------------------------: | :---------------------------------: |
| `absl::Flag` / `ABSL_FLAG` | `getopt` / `boost::program_options` |
|  `absl::ParseCommandLine`  |            手动解析 argv            |

## 5. 错误处理（Abseil 特色）



| Abseil                |          标准近似替代品           |            核心价值            |
| :-------------------- | :-------------------------------: | :----------------------------: |
| `absl::Status`        |  `bool` / `error_code` / `errno`  | 统一错误类型，带消息、码、来源 |
| `absl::StatusOr<T>`   | `std::expected`(C++23) / 双返回值 |    安全返回值或错误，无异常    |
| `absl::OkStatus()`    |            `true` / 0             |                                |
| `ABSL_CHECK/CHECK_OK` |             `assert`              |   生产环境也生效，崩溃更安全   |

## 6. 同步与并发



| Abseil               |           标准            |              优势              |
| :------------------- | :-----------------------: | :----------------------------: |
| `absl::Mutex`        |       `std::mutex`        | 更快、支持死锁检测、优先级继承 |
| `absl::CondVar`      | `std::condition_variable` |        配套 Mutex 使用         |
| `absl::Notification` |       自制条件变量        |      一次性唤醒，非常常用      |
| `absl::Barrier`      |           自制            |            线程屏障            |

## 7. 智能指针与内存



| Abseil                |         标准         |
| :-------------------- | :------------------: |
| `absl::WrapUnique`    |  `std::make_unique`  |
| `absl::WrapShared`    |  `std::make_shared`  |
| `absl::make_optional` | `std::make_optional` |

## 8. 工具函数



| Abseil              |              作用               |
| :------------------ | :-----------------------------: |
| `absl::make_unique` |     构造 `std::unique_ptr`      |
| `absl::exchange`    |      C++14 `std::exchange`      |
| `absl::endian`      |           字节序处理            |
| `absl::Cord`        | 大块字符串 / 二进制，零拷贝拼接 |
| `absl::Random`      |          高质量随机数           |

## 9. 日志（非常常用）



| Abseil         |     作用      |
| :------------- | :-----------: |
| `LOG(INFO)`    |   普通日志    |
| `LOG(WARNING)` |     警告      |
| `LOG(ERROR)`   |     错误      |
| `LOG(FATAL)`   |  致命并退出   |
| `VLOG(n)`      | 调试级别日志  |
| `DLOG`         | 仅 Debug 模式 |



代码参考：

```c++
#include <iostream>
#include <string_view>
#include <absl/strings/str_cat.h>
#include <absl/strings/str_split.h>
#include <absl/strings/str_format.h>
#include <absl/strings/str_replace.h>

using namespace std::literals;
class MyClass
{
public:
    std::string foo(std::string_view str) const
    {
		std::string s = absl::StrCat("Hello, ", str);
		absl::StrAppend(&s, " Welcome to C++17!");
		//split the string into two lines
		//for (auto word : absl::StrSplit(s, ' ')) {
		//	std::cout << word << "\n";
		//}
		//std::cout << "\n";

		//auto vec = absl::StrSplit(s, ' ');
		//for (const auto& word : vec) {
		//	std::cout << word << "\n";
		//}

		//std::string fomatter = absl::StrFormat("Formatted string: %s %ld", s.c_str(), 42);
		//std::cout << fomatter << "\n";

		//auto result = absl::StripPrefix(s, "Hello, ");
		//std::cout << "Stripped prefix: " << result << "\n";

		//result = absl::StripSuffix(s, " Welcome to C++17!");
		//std::cout << "Stripped suffix: " << result << "\n";

		absl::StrReplaceAll({ {"Hello", "Hi"}, {"C++17", "C++20"} }, &s);
		absl::AsciiStrToUpper(&s);
		std::cout << s << "\n";

		//absl::AsciiStrToLower(&s);
		//std::cout << s << "\n";

		if (absl::StrContains(s, "WORLD"))
		{
			std::cout << "contain WORLD\n";
		}
		else {
			std::cout << "does not contain WORLD\n";
		}

        return s;
    }
};
```

