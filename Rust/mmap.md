```c++
#include <iostream>
#include <pthread.h>
#include <sys/mman.h>
#include <unistd.h>
#include <fcntl.h>
#include <cstring>
#include <thread>

struct SharedData {
    pthread_mutex_t mutex;
    int counter;
};

void* create_shared_memory(size_t size) {
    // 创建匿名共享内存
    void* ptr = mmap(nullptr, size, PROT_READ | PROT_WRITE, MAP_SHARED | MAP_ANONYMOUS, -1, 0);
    if (ptr == MAP_FAILED) {
        std::cerr << "mmap failed: " << strerror(errno) << std::endl;
        return nullptr;
    }
    return ptr;
}

void thread_func(SharedData* data, int thread_id) {
    for (int i = 0; i < 10; ++i) {
        pthread_mutex_lock(&data->mutex);
        data->counter++; // 保护共享计数器
        std::cout << "Thread " << thread_id << " incremented counter to " << data->counter << std::endl;
        pthread_mutex_unlock(&data->mutex);
    }
}

int main() {
    // 创建共享内存
    size_t size = sizeof(SharedData);
    SharedData* shared_data = static_cast<SharedData*>(create_shared_memory(size));
    if (!shared_data) {
        return 1;
    }

    // 初始化互斥锁属性
    pthread_mutexattr_t mutex_attr;
    pthread_mutexattr_init(&mutex_attr);
    pthread_mutexattr_setpshared(&mutex_attr, PTHREAD_PROCESS_SHARED); // 设置为进程共享

    // 初始化互斥锁
    if (pthread_mutex_init(&shared_data->mutex, &mutex_attr) != 0) {
        std::cerr << "pthread_mutex_init failed" << std::endl;
        munmap(shared_data, size);
        return 1;
    }
    pthread_mutexattr_destroy(&mutex_attr);

    // 初始化计数器
    shared_data->counter = 0;

    // 创建多个线程
    const int num_threads = 8;
    std::thread threads[num_threads];
    for (int i = 0; i < num_threads; ++i) {
        threads[i] = std::thread(thread_func, shared_data, i);
    }

    // 等待线程完成
    for (int i = 0; i < num_threads; ++i) {
        threads[i].join();
    }

    // 打印最终计数
    std::cout << "Final counter value: " << shared_data->counter << std::endl;

    // 清理
    pthread_mutex_destroy(&shared_data->mutex);
    munmap(shared_data, size);

    return 0;
}
```

