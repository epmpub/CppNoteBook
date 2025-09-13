## Rayon 

是 Rust 中用于并行计算的强大库，提供了简单的方式来利用多核处理器进行数据并行和任务并行。以下是一些 Rayon 中常用的方法和模式，涵盖其核心功能，重点介绍 rayon::prelude 中常用的 API，简洁且实用。1. 并行迭代器（Parallel Iterator）Rayon 的并行迭代器是其核心功能，允许你将串行迭代器（如 Vec 的 .iter()）转换为并行版本。需要引入 rayon::prelude::*。常用方法：

- par_iter() / par_iter_mut()
  将标准迭代器转换为并行迭代器，适用于只读或可变引用。

  rust

  

  ```rust
  use rayon::prelude::*;
  let numbers = vec![1, 2, 3, 4, 5];
  let sum: i32 = numbers.par_iter().sum(); // 并行求和
  println!("Sum: {}", sum); // 输出：15
  ```

- map
  对每个元素并行应用函数。

  rust

  

  ```rust
  let squares: Vec<i32> = numbers.par_iter().map(|&x| x * x).collect();
  // 输出：[1, 4, 9, 16, 25]
  ```

- filter
  并行过滤元素。

  rust

  

  ```rust
  let evens: Vec<i32> = numbers.par_iter().filter(|&&x| x % 2 == 0).copied().collect();
  // 输出：[2, 4]
  ```

- fold / reduce
  fold 用于并行归约每个线程的子集，reduce 再合并结果。

  rust

  

  ```rust
  let sum: i32 = numbers.par_iter().fold(|| 0, |acc, &x| acc + x).reduce(|| 0, |a, b| a + b);
  ```

- for_each
  对每个元素并行执行操作，无返回值。

  rust

  

  ```rust
  numbers.par_iter().for_each(|&x| println!("Processing: {}", x));
  ```

- find_any / find_first
  并行查找满足条件的元素，find_any 返回任意线程找到的第一个元素，find_first 按顺序查找。

  rust

  

  ```rust
  let found = numbers.par_iter().find_any(|&&x| x > 3);
  // 输出：Some(4) 或 Some(5)
  ```

- collect
  将并行迭代器的结果收集到容器（如 Vec）。

  rust

  

  ```rust
  let doubled: Vec<i32> = numbers.par_iter().map(|&x| x * 2).collect();
  ```

注意事项：

- 并行迭代器适合数据量较大、计算密集的任务，小数据量可能因线程开销得不偿失。
- 确保操作是线程安全的，避免数据竞争。
- 并行排序（Parallel Sort）Rayon 提供了高效的并行排序方法。

- par_sort
  对可变切片进行并行排序。

  rust

  

  ```rust
  let mut numbers = vec![5, 2, 4, 1, 3];
  numbers.par_sort();
  // 输出：[1, 2, 3, 4, 5]
  ```

- par_sort_by
  自定义排序规则。

  rust

  

  ```rust
  numbers.par_sort_by(|a, b| b.cmp(a)); // 降序
  // 输出：[5, 4, 3, 2, 1]
  ```

- 任务并行（Task Parallelism）Rayon 还支持任务并行，通过 join 和 scope 分派独立任务。

- join(a, b)
  并行执行两个闭包，等待两者完成。

  rust

  

  ```rust
  use rayon::join;
  let (result1, result2) = join(
      || (0..1000).sum::<i32>(),
      || (1000..2000).sum::<i32>(),
  );
  ```

- scope
  创建一个作用域，允许动态生成并行任务。

  rust

  

  ```rust
  use rayon::scope;
  let mut results = vec![];
  scope(|s| {
      s.spawn(|_| results.push(1));
      s.spawn(|_| results.push(2));
  });
  ```

- 线程池配置Rayon 默认使用全局线程池，但可以自定义配置。

- ThreadPoolBuilder
  设置线程数或自定义线程池。

  rust

  

  ```rust
  use rayon::ThreadPoolBuilder;
  let pool = ThreadPoolBuilder::new().num_threads(4).build().unwrap();
  pool.install(|| {
      // 在自定义线程池中运行并行任务
      (0..100).into_par_iter().for_each(|i| println!("Task {}", i));
  });
  ```

- 其他实用方法

- par_extend
  并行扩展集合，适合将并行迭代器的结果追加到现有容器。

  rust

  

  ```rust
  let mut vec = vec![1, 2, 3];
  vec.par_extend((4..10).into_par_iter());
  // 输出：[1, 2, 3, 4, 5, 6, 7, 8, 9]
  ```

- any / all
  检查是否存在或所有元素满足条件。

  rust

  

  ```rust
  let has_even = numbers.par_iter().any(|&x| x % 2 == 0); // true
  let all_positive = numbers.par_iter().all(|&x| x > 0); // true
  ```

- 性能优化建议

- 粒度控制：使用 with_min_len 或 with_max_len 控制并行任务的分割粒度，避免过多线程开销。

  rust

  

  ```rust
  numbers.par_iter().with_min_len(100).map(|&x| x * x).collect::<Vec<_>>();
  ```

- 避免锁竞争：并行迭代中避免共享可变状态，若必须使用，考虑 Mutex 或 Atomic 类型。

- 负载均衡：Rayon 自动进行工作窃取（work-stealing），但确保任务分布均匀以最大化性能。

示例：综合应用

rust



```rust
use rayon::prelude::*;
fn main() {
    let data = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    let result: Vec<i32> = data
        .par_iter()
        .filter(|&&x| x % 2 == 0) // 过滤偶数
        .map(|&x| x * x)         // 计算平方
        .collect();              // 收集结果
    println!("{:?}", result); // 输出：[4, 16, 36, 64, 100]
}
```

总结Rayon 的核心在于并行迭代器（par_iter、map、filter 等）和任务并行（join、scope）。通过这些方法，可以轻松将串行代码转换为高效的并行代码。建议根据任务规模选择合适的并行策略，并注意线程安全和性能优化。如果需要更详细的解释或针对特定场景的示例，请告诉我！