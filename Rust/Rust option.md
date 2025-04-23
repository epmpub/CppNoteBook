

# Rust常见数据结构

## Option

```rust
pub enum Option<T> {
    None,
    Some(T),
}

```

## Result

```rust
enum Result<T, E> {
   Ok(T),
   Err(E),
}
```

```rust

use std::result;

#[allow(dead_code)]

#[derive(Debug)]
struct Number {
    value: i32,
}


impl From<i32> for Number {
    fn from(item: i32) -> Self {
        Number { value: item }
    }
}

#[derive(Debug,PartialEq)]
struct EvenNumber(i32);


impl TryFrom<i32> for EvenNumber  {
    type Error = ();

    fn try_from(value: i32) -> Result<Self, Self::Error> {
        if value % 2 == 0 {
            Ok(EvenNumber(value))
        }else {
            Err(())
        }
    }
    
}


#[derive(Debug,PartialEq)]
enum MyResult<T,E> {
    Ok(T),
    Err(E)
}

fn divide(numerator: f64, denominator: f64) -> Option<f64> {
    if denominator == 0.0 {
        None
    } else {
        Some(numerator / denominator)
    }
}


fn add_last_numbers(stack:&mut Vec<i32>) -> Option<i32> {
    Some(stack.pop()?+stack.pop()?)
}

fn main() {
    // let mut v = vec![1; 3];
    // v.push(1);
    // v.push(3);
    // v.push(5);
    // for i in v.iter() {
    //     println!("old -> {}", i);
    // }

    // let v1 = v;
    // for i in v1.iter() {
    //     println!("v1 -> {}", i);
    // }

    // let num = Number::from(10);

    // let num2: Number = 5.into();

    // println!("number = {:?}",num);
    // println!("number2 = {:?}", num2);

    
    // assert_eq!(EvenNumber::try_from(8),Ok(EvenNumber(8)));
    // assert_eq!(EvenNumber::try_from(5),Err(()));


    // let result: Result<EvenNumber, ()> = 8.try_into();
    // assert_eq!(result,Ok(EvenNumber(8)));

    // let result: Result<EvenNumber, ()> = 5.try_into();
    // assert_eq!(result, Err(()));

    // let a:MyResult<i32,()> = MyResult::Ok(5);
    // assert_eq!(a,MyResult::Ok(5));

    // let a:MyResult<i32,()> = MyResult::Err(());
    // assert_eq!(a,MyResult::Err(()));

    // let result = divide(2.0, 3.0);

    // Pattern match to retrieve the value
    // match divide(2.0, 3.0) {
    //     // The division was valid
    //     Some(x) => println!("Result: {x}"),
    //     // The division was invalid
    //     None    => println!("Cannot divide by 0"),
    // }

    let mut v = vec![1,2,3];
    assert_eq!(add_last_numbers(&mut v),Some(5));


    let mut v = vec![1;1];
    assert_eq!(add_last_numbers(&mut v),None);

    println!("{}",add_last_numbers(&mut v).unwrap_or_default());

}

```

**The question mark operator, ?**

```rust

When writing code that calls many functions that return the Result type, the error handling can be tedious. The question mark operator, ?, hides some of the boilerplate of propagating errors up the call stack.

It replaces this:

use std::fs::File;
use std::io::prelude::*;
use std::io;

struct Info {
    name: String,
    age: i32,
    rating: i32,
}

fn write_info(info: &Info) -> io::Result<()> {
    // Early return on error
    let mut file = match File::create("my_best_friends.txt") {
           Err(e) => return Err(e),
           Ok(f) => f,
    };
    if let Err(e) = file.write_all(format!("name: {}\n", info.name).as_bytes()) {
        return Err(e)
    }
    if let Err(e) = file.write_all(format!("age: {}\n", info.age).as_bytes()) {
        return Err(e)
    }
    if let Err(e) = file.write_all(format!("rating: {}\n", info.rating).as_bytes()) {
        return Err(e)
    }
    Ok(())
}

With this:

use std::fs::File;
use std::io::prelude::*;
use std::io;

struct Info {
    name: String,
    age: i32,
    rating: i32,
}

fn write_info(info: &Info) -> io::Result<()> {
    let mut file = File::create("my_best_friends.txt")?;
    // Early return on error
    file.write_all(format!("name: {}\n", info.name).as_bytes())?;
    file.write_all(format!("age: {}\n", info.age).as_bytes())?;
    file.write_all(format!("rating: {}\n", info.rating).as_bytes())?;
    Ok(())
}

```

