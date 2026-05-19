# `itertools` --- 为高效循环创建迭代器的函数

------

本模块实现一系列 [iterator](https://docs.python.org/zh-cn/3.13/glossary.html#term-iterator) ，这些迭代器受到APL，Haskell和SML的启发。为了适用于Python，它们都被重新写过。

本模块标准化了一个快速、高效利用内存的核心工具集，这些工具本身或组合都很有用。它们一起形成了“迭代器代数”，这使得在纯Python中有可能创建简洁又高效的专用工具。

例如，SML有一个制表工具： `tabulate(f)`，它可产生一个序列 `f(0), f(1), ...`。在Python中可以组合 [`map()`](https://docs.python.org/zh-cn/3.13/library/functions.html#map) 和 [`count()`](https://docs.python.org/zh-cn/3.13/library/itertools.html#itertools.count) 实现： `map(f, count())`。

**无穷迭代器：**

| 迭代器                                                       | 实参            | 结果                                  | 示例                                  |
| :----------------------------------------------------------- | :-------------- | :------------------------------------ | :------------------------------------ |
| [`count()`](https://docs.python.org/zh-cn/3.13/library/itertools.html#itertools.count) | [start[, step]] | start, start+step, start+2*step, ...  | `count(10) → 10 11 12 13 14 ...`      |
| [`cycle()`](https://docs.python.org/zh-cn/3.13/library/itertools.html#itertools.cycle) | p               | p0, p1, ... plast, p0, p1, ...        | `cycle('ABCD') → A B C D A B C D ...` |
| [`repeat()`](https://docs.python.org/zh-cn/3.13/library/itertools.html#itertools.repeat) | elem [,n]       | elem, elem, elem, ... 重复无限次或n次 | `repeat(10, 3) → 10 10 10`            |

**根据最短输入序列长度停止的迭代器：**

| 迭代器                                                       | 实参                        | 结果                                          | 示例                                                     |
| :----------------------------------------------------------- | :-------------------------- | :-------------------------------------------- | :------------------------------------------------------- |
| [`accumulate()`](https://docs.python.org/zh-cn/3.13/library/itertools.html#itertools.accumulate) | p [,func]                   | p0, p0+p1, p0+p1+p2, ...                      | `accumulate([1,2,3,4,5]) → 1 3 6 10 15`                  |
| [`batched()`](https://docs.python.org/zh-cn/3.13/library/itertools.html#itertools.batched) | p, n                        | (p0, p1, ..., p_n-1), ...                     | `batched('ABCDEFG', n=2) → AB CD EF G`                   |
| [`chain()`](https://docs.python.org/zh-cn/3.13/library/itertools.html#itertools.chain) | p, q, ...                   | p0, p1, ... plast, q0, q1, ...                | `chain('ABC', 'DEF') → A B C D E F`                      |
| [`chain.from_iterable()`](https://docs.python.org/zh-cn/3.13/library/itertools.html#itertools.chain.from_iterable) | iterable -- 可迭代对象      | p0, p1, ... plast, q0, q1, ...                | `chain.from_iterable(['ABC', 'DEF']) → A B C D E F`      |
| [`compress()`](https://docs.python.org/zh-cn/3.13/library/itertools.html#itertools.compress) | data, selectors             | (d[0] if s[0]), (d[1] if s[1]), ...           | `compress('ABCDEF', [1,0,1,0,1,1]) → A C E F`            |
| [`dropwhile()`](https://docs.python.org/zh-cn/3.13/library/itertools.html#itertools.dropwhile) | predicate, seq              | seq[n], seq[n+1], 从 predicate 未通过时开始   | `dropwhile(lambda x: x<5, [1,4,6,3,8]) → 6 3 8`          |
| [`filterfalse()`](https://docs.python.org/zh-cn/3.13/library/itertools.html#itertools.filterfalse) | predicate, seq              | predicate(elem) 未通过的 seq 元素             | `filterfalse(lambda x: x<5, [1,4,6,3,8]) → 6 8`          |
| [`groupby()`](https://docs.python.org/zh-cn/3.13/library/itertools.html#itertools.groupby) | iterable[, key]             | 根据key(v)值分组的迭代器                      | `groupby(['A','B','DEF'], len) → (1, A B) (3, DEF)`      |
| [`islice()`](https://docs.python.org/zh-cn/3.13/library/itertools.html#itertools.islice) | seq, [start,] stop [, step] | seq[start:stop:step]中的元素                  | `islice('ABCDEFG', 2, None) → C D E F G`                 |
| [`pairwise()`](https://docs.python.org/zh-cn/3.13/library/itertools.html#itertools.pairwise) | iterable -- 可迭代对象      | (p[0], p[1]), (p[1], p[2])                    | `pairwise('ABCDEFG') → AB BC CD DE EF FG`                |
| [`starmap()`](https://docs.python.org/zh-cn/3.13/library/itertools.html#itertools.starmap) | func, seq                   | func(*seq[0]), func(*seq[1]), ...             | `starmap(pow, [(2,5), (3,2), (10,3)]) → 32 9 1000`       |
| [`takewhile()`](https://docs.python.org/zh-cn/3.13/library/itertools.html#itertools.takewhile) | predicate, seq              | seq[0], seq[1], 直到 predicate 未通过         | `takewhile(lambda x: x<5, [1,4,6,3,8]) → 1 4`            |
| [`tee()`](https://docs.python.org/zh-cn/3.13/library/itertools.html#itertools.tee) | it, n                       | it1, it2, ... itn 将一个迭代器拆分为n个迭代器 |                                                          |
| [`zip_longest()`](https://docs.python.org/zh-cn/3.13/library/itertools.html#itertools.zip_longest) | p, q, ...                   | (p[0], q[0]), (p[1], q[1]), ...               | `zip_longest('ABCD', 'xy', fillvalue='-') → Ax By C- D-` |

**排列组合迭代器：**

| 迭代器                                                       | 实参                 | 结果                                  |
| :----------------------------------------------------------- | :------------------- | :------------------------------------ |
| [`product()`](https://docs.python.org/zh-cn/3.13/library/itertools.html#itertools.product) | p, q, ... [repeat=1] | 笛卡尔积，相当于嵌套的for循环         |
| [`permutations()`](https://docs.python.org/zh-cn/3.13/library/itertools.html#itertools.permutations) | p[, r]               | 长度r元组，所有可能的排列，无重复元素 |
| [`combinations()`](https://docs.python.org/zh-cn/3.13/library/itertools.html#itertools.combinations) | p, r                 | 长度r元组，有序，无重复元素           |
| [`combinations_with_replacement()`](https://docs.python.org/zh-cn/3.13/library/itertools.html#itertools.combinations_with_replacement) | p, r                 | 长度r元组，有序，元素可重复           |

| 例子                                       | 结果                                              |
| :----------------------------------------- | :------------------------------------------------ |
| `product('ABCD', repeat=2)`                | `AA AB AC AD BA BB BC BD CA CB CC CD DA DB DC DD` |
| `permutations('ABCD', 2)`                  | `AB AC AD BA BC BD CA CB CD DA DB DC`             |
| `combinations('ABCD', 2)`                  | `AB AC AD BC BD CD`                               |
| `combinations_with_replacement('ABCD', 2)` | `AA AB AC AD BB BC BD CC CD DD`                   |