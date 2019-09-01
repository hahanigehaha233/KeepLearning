## IPC
IPC是指Inter-Process Communication，进程间通信，POSIX IPC是指由`POSIX.1b`设计来取代与之类似的`System V IPC`机制的三种IPC机制--消息队列、信号量以及共享内存。

### `POSIX`消息队列
- 引用计数，只用所有当前使用队列的进程都关闭了队列之后才会对队列标记删除。
- 消息之间按照严格的优先级进行排列。
- 在一条消息可用时可用异步通知进程。
- POSIX是在内核2.6.6之后才加入的。
