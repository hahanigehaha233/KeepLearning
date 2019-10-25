### Acceptor
用于处理新连接，是TcpServer的内部类，数据成员：
- Socket：是Listening socket即server socket
- Channel：用于观察Readable事件，并回调：：handleRead()

Acceptor会进行Socket的创建和bind，这和Tcpserver中的是一样的，并且为Readable设置回调函数readCallback_()这是创建链接时的回调。

Acceptor::listen()会完成listen系统调用并将channel加入到epoll（loop）中。

[Acceptor例子讲解](https://blog.csdn.net/messiran10/article/details/51692078)
