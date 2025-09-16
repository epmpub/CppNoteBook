## str介绍



```rust
use regex::Regex;

fn main() {
    let msg = "中国 Hello, world!";
    let zh = &msg[0..6];
    println!("1.你好 {}",zh);

    let zh2 = msg.chars().collect::<Vec<char>>();
    println!("2.你好 {}{}",zh2[0],zh2[1]);

    let zh3: String = msg.chars().take(2).collect(); // 取前两个字符
    println!("3.你好 {}", zh3); // 打印 "你好 中国"

    let zh4 = format!("{}{}", msg.chars().nth(0).unwrap(), msg.chars().nth(1).unwrap());
    println!("5.你好 {}", zh4); // 打印 "你好 中国"

    let (zh5, _rest) = msg.split_at(6); // 分割到第 6 个字节
    println!("5.你好 {}", zh5); // 打印 "你好 中国"

    let zh6 = msg.chars().collect::<Vec<char>>();
    let zh6 = zh6[0..2].iter().collect::<String>(); // 切片并转为 String
    println!("6.你好 {}", zh6); // 打印 "你好 中国"

    let re = Regex::new(r"^[^\x00-\x7F]{2}").unwrap(); // 匹配前两个非 ASCII 字符
    let zh7 = re.find(msg).unwrap().as_str();
    println!("7.你好 {}", zh7);// 打印 "你好 中国"

    let zh8 = msg.get(0..6).unwrap(); // 安全获取字节切片
    println!("8.你好 {}", zh8); // 打印 "你好 中国"    
}
```

