## LevelDB 总体架构
特点
- `LevelDB`是一个持久化存储的K-V系统，和Redis这种内存型系统不同，它是将大部分的数据存储在了磁盘中。
- 其存储Key值是有序存储的，用户可以自定义Key值比较函数。
- 支持数据快照，这样在程序读数据的时候不会受到写操作的影响。

 LevelDB的存储结构可以简单的分为三个部分，在内存中的可变`Memtable`，在内存中的不可变`Immutable Memtable`以及多个级别的`SSTable`。其操作增、删、改、查的大致操作流程如下。

 增（PUT）：
 1. 首先使用顺序写的方式将这条记录加入到`WAL LOG`中，这是磁盘文件，但是输入顺序写操作，所以速度上不会降低太多。
 2. 将数据更新到`Memtable`中。
 3. 当`Memtable size`超过一定的限制之后，将其整理成为一个`Immutable Memtable`,这个表之后不可更改。
 4. 把不可更改的KV表写入磁盘，并清理`WAL LOG`文件。
 5. 当`Level0 SST`文件大小太多的时候，`Level0 SST`文件就会与更高级的`SST`文件进行合并，保证有序性。
 6. `Level1 SST`同理。

 查（GET）：
 1. 先在`Memtable`中查找相应的键值。
 2. 如果第1步中未找到，再到`Immutable Memtable`中查找。
 3. 如果第2步中未找到，就需要在磁盘块中寻找。

为了方便查找，会将所有涉及到SST的文件操作都在`Manifest`文件中做记录。**Manifest就是SSTable的WAL LOG**
- MANIFEST 文件中记录了 LevelDB 中所有层级中的表、每一个 SSTable 的 Key 范围和其他重要的元数据
- 以日志格式进行存储
- 对所有文件的增删操作都会追加到这个日志中

---

## `Write`操作 [参考](https://blog.csdn.net/swartz2015/article/details/66970885)
`Write`操作会向`Memtable`中写入数据，可以是一条键值对，也可以是一个batch。其中具体操作流程是:
1. 先将需要插入的KV封装到一个`batch`中，然后调用`Write(opt, &batch)`
2. 首先会创建一个`Writer w`对象，然后使用互斥量`Mutex`上锁，将`w`传入`writers_`队列中，并且判断当前的`w`是否完成或者是否在队列最前面。
3. 如果完成则返回`Status OK`。
4. 如果是在队列最前面则需要当前进程处理，首先调用`MakeRoomForWrite(updates == nullptr)`让`Memtable`中有足够的空间可以存放KV。
> `MakeRoomForWrite`函数内部逻辑：
> 1. 判断是否可以延迟写且`level0`文件过多。如果是的话则释放锁并`Sleep`（延迟，等待背景进程完成工作）
> 2. 判断是否有数据写入并且`Memtable`空间足够，如果有则返回。
> 3. 执行到这说明`Mem`没有足够空间，此时需要将`Mem`赋值给`Imm`，然后重新分配`Mem`给用户写，但是之前必须检查`Imm`是否为空，不为空的话只能等待背景线程完成。
> 4. 因为第三步的原因，需要再次判断`level0`文件数量。
> 5. 申请一个新的`Mem`，将旧的`Imm`写盘，将旧的`Mem`写入`Imm`。
> 6. 调用`MaybeScheduleCompaction();`开启背景线程进行写盘和合并。

[MakeRoomForWrite函数解析](https://blog.csdn.net/swartz2015/article/details/66972106)

> `MaybeScheduleCompaction()`函数内部逻辑：详情见后
>
> 当确定当前数据库中没有背景线程，也不存在错误，同时确实有工作需要背景线程来完成，就通过env_->Schedule(&DBImpl::BGWork, this)启动背景线程，前面的bg_compaction_scheduled_设置主要是告诉其他线程当前数据库中已经有一个背景线程在运行了。其中调用了`BackgroundCompaction()`函数：
> - 如果当前的imm_非空，则将其写盘生成一个新的sstable
> - 对各个level的文件进行合并，避免level中文件过多，以及删掉被删除的key-value(因为leveldb里面采用的是lazy delete的方法，用户调用delete时没有真正删除元素，只有在背景线程对文件进行合并时才会真的删除元素)。

5. 调用`BuildBatchGroup()`对可以合并的多个batch进行合并同时写入`Memtable`（更新sequence）。
6. 在写入`WAL LOG和Mem`中时可以释放锁，因为其他线程是不满足第四步的条件的。
7. 删除`writers_`中处理好的`Writer`。
8. 唤醒`writers_`中下一个`Writer`线程。
---

### 基本数据结构

#### `Slice`
将数据和数据的长度包装成`Slice`对象使用。

#### `WriteBatch`
`PUT`操作读入数据时首先写入这个结构中，通过一个Batch写入`Memtable`中，其中就是操作一个`std::string`类型。

- 这里面牵扯到大端序转小端序、从Int类型转换成Varint类型。[参考](https://blog.csdn.net/caoshangpa/article/details/78815940)
