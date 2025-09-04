

## itertools 高级用法

```rust
use itertools::Itertools;

fn main(){
    let data = vec![1, 1, 2, 2, 3, 3, 3,4];
    let groups = data.clone().into_iter()
        .chunk_by(|&x| x)
        .into_iter()
        .map(|(key, group)| (key, group.collect::<Vec<_>>()))
        .collect::<Vec<_>>();

    for (key, group) in groups {
        println!("Key: {}, Group: {:?}", key, group);
    }


    let group = data.iter().into_group_map_by(|&x| x);

    let sorted = group.iter().sorted_by(|a, b| a.0.cmp(&b.0));

    for (key, value) in sorted {
        println!("数字 {} 个数 {}", key, value.len());
    }

}

```

