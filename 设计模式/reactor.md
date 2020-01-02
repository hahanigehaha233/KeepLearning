# Reactor和Procator的区别
最大的区别在于Procator用于异步IO，对于读取事件来说，它往往是在读取完成之后才进行通知，而reactor则是在缓冲区读就绪时候就返回。
[区别](https://www.zhihu.com/question/26943938/answer/656823089)
