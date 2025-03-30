C++线程同步


线程同步是指确保多个线程按特定顺序执行或访问共享资源时不会发生冲突的技术。线程同步的主要目的是避免数据竞争和保证操作的原子性，以确保程序的正确性和一致性。

1. 互斥量（Mutex）

互斥量用于保护对共享资源的访问，以确保同一时间只有一个线程可以访问该资源。C++中提供了std::mutex类来实现互斥量。

std::mutex mtx;

void thread_function() {
    std::lock_guard<std::mutex> lock(mtx);
    // 访问和修改共享资源
}
2. 条件变量（Condition Variable）

条件变量用于让线程等待某个条件成立后再继续执行。它通常与互斥量一起使用，线程可以在条件变量上等待，直到另一个线程通知它条件已满足。

std::mutex mtx;
std::condition_variable cv;
bool ready = false;

void waiting_thread() {
    std::unique_lock<std::mutex> lock(mtx);
    cv.wait(lock, []{ return ready; });
    // 条件满足后继续执行
}

void notifying_thread() {
    {
        std::lock_guard<std::mutex> lock(mtx);
        ready = true;
    }
    cv.notify_one(); // 或者 cv.notify_all();
}
3. 读写锁（Read-Write Lock）

读写锁允许多个线程同时读取共享资源，但在写入资源时会阻止其他线程的读取和写入。C++20中引入了std::shared_mutex来实现读写锁。

std::shared_mutex rw_lock;

void read_function() {
    std::shared_lock<std::shared_mutex> lock(rw_lock);
    // 读取共享资源
}

void write_function() {
    std::unique_lock<std::shared_mutex> lock(rw_lock);
 4. 原子操作（Atomic Operations）

原子操作可以保证对共享变量的操作是不可中断的，从而避免数据竞争。C++11引入了std::atomic类来支持原子操作。

std::atomic<int> atomic_var(0);

void increment_function() {
    atomic_var.fetch_add(1);
}
5. 信号量（Semaphore）
信号量控制对资源的访问，以限制同时访问的线程数量。C++20中引入了std::counting_semaphore。

#include <iostream>
#include <thread>
#include <semaphore>
#include <vector>

std::counting_semaphore<3> sem(3); // 创建一个初始计数为3的信号量

void worker(int id) {
    sem.acquire(); // 请求信号量，可能会阻塞
    std::cout << "Thread " << id << " is working\n";
    std::this_thread::sleep_for(std::chrono::seconds(1)); // 模拟工作
    std::cout << "Thread " << id << " is done\n";
    sem.release(); // 释放信号量
}

int main() {
    std::vector<std::thread> threads;
    for (int i = 0; i < 10; ++i) {
        threads.emplace_back(worker, i);
    }

    for (auto& t : threads) {
        t.join();
    }
     
    return 0;

