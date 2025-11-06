# io_uring 最简单的演示代码



```C++
#include <stdio.h>
#include <fcntl.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <liburing.h>

#define BUF_SIZE 4096

int main()
{
    struct io_uring ring;
    int fd, ret;
    char *buf;

    // 初始化 io_uring 实例
    ret = io_uring_queue_init(8, &ring, 0);


    // 打开文件
    fd = open("test.txt", O_RDONLY);


    // 分配缓冲区
    buf = malloc(BUF_SIZE);


    // 获取一个提交队列条目（SQE）
    struct io_uring_sqe *sqe = io_uring_get_sqe(&ring);


    // 设置异步读取操作
    io_uring_prep_read(sqe, fd, buf, BUF_SIZE, 0);  // 从偏移量0读取
    sqe->flags |= IOSQE_IO_LINK;  // 可选：链接多个操作

    // 提交请求到内核
    ret = io_uring_submit(&ring);


    // 等待完成事件
    struct io_uring_cqe *cqe;
    ret = io_uring_wait_cqe(&ring, &cqe);


    // 处理完成结果
    if (cqe->res < 0) {
        fprintf(stderr, "Async read failed: %s\n", strerror(-cqe->res));
    } else {
        printf("Read %d bytes:\n", cqe->res);
        printf("%.*s\n", cqe->res, buf);  // 安全打印读取内容
    }

    // 标记完成事件已处理
    io_uring_cqe_seen(&ring, cqe);

    // 清理资源
    free(buf);
    close(fd);
    io_uring_queue_exit(&ring);

    return 0;
}
```

