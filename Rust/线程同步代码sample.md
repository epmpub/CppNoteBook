```C++
// use std::sync::{Arc, Mutex};
// use std::thread;

// fn main() {
//     let data = Arc::new(Mutex::new(5));
    
//     let data2 = Arc::clone(&data);
//     let handle = thread::spawn(move || {
//         let mut num = data2.lock().unwrap();
//         println!("In thread: {}", *num);
//         println!("Thread count: {}", Arc::strong_count(&data2));
//         //data2 value add one
//         *num += 1;
//         println!("Thread count after increment: {}", Arc::strong_count(&data2));
//     });
    


//     let data2 = Arc::clone(&data);
//     let handle2 = thread::spawn(move || {
//         let mut num = data2.lock().unwrap();
//         println!("In thread: {}", *num);
//         println!("Thread count: {}", Arc::strong_count(&data2));
//         //data2 value add one
//         *num += 1;
//         println!("Thread count after increment: {}", Arc::strong_count(&data2));
//     });

    
//     handle.join().unwrap();
    
//     handle2.join().unwrap();

//     println!("Final count: {}", Arc::strong_count(&data));
//     println!("Final value: {}", *data.lock().unwrap());
// }


// use std::sync::{Arc, RwLock};
// use std::thread;

// fn main() {
//     let data = Arc::new(RwLock::new(5));
//     let data2 = Arc::clone(&data);
//     let handle = thread::spawn(move || {
//         let num = data2.read().unwrap();
//         println!("In thread: {}", *num);
//         println!("Thread count: {}", Arc::strong_count(&data2));
//         drop(num);
//         let mut num = data2.write().unwrap();
//         *num += 1;
//         println!("Thread count after increment: {}", Arc::strong_count(&data2));

//     });

//     let data2 = Arc::clone(&data);
//     let handle2 = thread::spawn(move || {
//         let num = data2.read().unwrap();
//         println!("In thread: {}", *num);
//         println!("Thread count: {}", Arc::strong_count(&data2));
//         drop(num);
//         let mut num = data2.write().unwrap();
//         *num += 1;
//         println!("Thread count after increment: {}", Arc::strong_count(&data2));

//     });

//     handle.join().unwrap();
//     handle2.join().unwrap();
//     println!("Final count: {}", Arc::strong_count(&data));
//     println!("Final value: {}", *data.read().unwrap());

// }


// use std::sync::atomic::{AtomicI32, Ordering};
// use std::sync::Arc;
// use std::thread;

// fn main() {
//     let data = Arc::new(AtomicI32::new(5));
    
//     let data2 = Arc::clone(&data);
//     let handle = thread::spawn(move || {
//         println!("In thread: {}", data2.load(Ordering::SeqCst));
//         println!("Thread count: {}", Arc::strong_count(&data2));
//         data2.fetch_add(1, Ordering::SeqCst);
//         println!("Thread count after increment: {}", Arc::strong_count(&data2));
//     });

//     let data2 = Arc::clone(&data);
//     let handle2 = thread::spawn(move || {
//         println!("In thread: {}", data2.load(Ordering::SeqCst));
//         println!("Thread count: {}", Arc::strong_count(&data2));
//         data2.fetch_add(1, Ordering::SeqCst);
//         println!("Thread count after increment: {}", Arc::strong_count(&data2));
//     });

//     handle.join().unwrap();
//     handle2.join().unwrap();

//     println!("Final count: {}", Arc::strong_count(&data));
//     println!("Final value: {}", data.load(Ordering::SeqCst));
// }


use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel(); // 创建通道
    let tx2 = tx.clone(); // 克隆发送端给第二个线程

    let handle = thread::spawn(move || {
        let value = 5; // 初始值
        println!("In thread: {}", value);
        tx.send(value + 1).unwrap(); // 发送修改后的值
    });

    let handle2 = thread::spawn(move || {
        let value = 5; // 初始值
        println!("In thread: {}", value);
        tx2.send(value + 1).unwrap(); // 发送修改后的值
    });

    handle.join().unwrap();
    handle2.join().unwrap();

    let mut final_value = 5;
    for received in rx {
        final_value = received; // 接收并更新值
        println!("Received: {}", final_value);
    }

    println!("Final value: {}", final_value);
}
```

