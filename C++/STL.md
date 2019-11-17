#### STL::addressof
可以在没有重载&运算符的时候获得地址。

## 空间配置器
一般说的就是`std::alloc`，在`<memory>`中实现，包含三个文件
- `<stl_construct.h>`:定义了全局函数`construct()`和`destroy()`，这两个方法会调用对象的构造和析构函数。
- `<stl_alloc.h>`:内存的配置和释放在这里面实现，内部有两级配置器，第一级结构简单，封装了`malloc`和`free`，主要针对的是大内存分配，而第二级实现了自由链表和内存池，用于提升大量小额内存分配时的性能。
- `<stl_uninitialiezed.h>`：大块内存的操作。

其中，一级配置器有函数
- `allocate()`:调用malloc进行分配
- `deallocate()`:调用free进行回收
- `oom_malloc()和oom_realloc()`:在空间不足的时候进行分配

二级配置器中会对小于128bytes的内存分配使用free-list进行管理，这样会造成一定的资源浪费（指针占用），使用Union实现减少浪费。

一般的内存分配顺序是：
1. 调用空间配置函数allocate()，如果申请的区块大于128bytes则调用第一级配置器，如果小于则检查对应的free-list查看是否有可用的区块。
2. 如果没有可用的区块，将申请大小上调到8的倍数，然后调用refill()，该函数会从内存池中获取20个新的节点，并加入到free-list中。从内存池中寻找是chunk_alloc()的工作。
3. chunk_alloc()首先会检测内存剩余空间，如果满足的话直接返回，如果不满足全部但是满足供应一个或以上的区块，也返回；如果连一个区块都满足不了的时候，只有一些零头，会将其加入适当地free-list中，让后在system_heap中进行配置，如果系统都没有的话会发出bad_alloc异常。

**注意**
- free-list是按照区块大小进行划分的。
- 每次chunk_alloc会申请多一点的空间，多余空间会放入free_list中。
- 参考《STL源码剖析》P68

---

## 迭代器
迭代器是一种行为类似指针的对象，而指针的各种行为最常见也最重要的便是内容提领和成员访问，因此迭代器最重要的编程工作就是对`operator*`和`operator->`进行重载。

---

## vector
Vector的数据结构为连续的线性空间。它里面的三个迭代器
```
protected:
  iterator start;//空间的头
  iterator finish;//目前使用空间的尾
  iterator en_of_storage;//目前可用空间的尾

```
就可以提供首位标识、大小、容量、空容器判断，[]算子等多种运算。
```
public:
  iterator begin(){return start;}
  iterator end(){return finish;}
  size_type size const{return size_type(end()-begin());}
等等
```

vector在新增元素的时候，如果现有的空间不够，则会申请两倍于现在的空间，对之前的元素进行移动。

vector在删除元素的时候，会判断该元素是否为最后一个元素，如若是则直接使用destroy()删除，如果不是则使用copy移动后面的所有元素至该元素位置然后删除最后元素。

---

## list
```
template<class T>
struct __list_node {
  typedef void* void_pointer;
  void_pointer prev;
  void_pointer next;
  T data;
}
```
从上面的结构可以看出，list以链表进行连接，这也体现出来它与vector不同的地方，它的内存配置更加精准，不会造成一丝一毫的浪费，而且对于任意位置的元素插入和元素移除，list可以在常数时间内完成，这是vector这种连续空间所不能具备的。

最为重要的一点是，list的插入和结合操作，不会造成任何的迭代器失效，并且删除操作也只会让一个迭代器失效。

list是一个环状双向链表。因此只需要一个__list_node对象便可以表示，如下：
```
template<class T, class Alloc = alloc>
class list{
protected:
  typedef __list_node<T> list_node;
public:
  typedef list_node* link_type;
protected:
  link_type node;
}
```
如果将node节点指向listnode尾端的一个空白节点中，操作函数便可以轻易实现：
```
iterator begin(){return (link_type)((*node).next);}
iterator end(){return node;}
bool empty() const {return node->next == node;}
size_type size() const{
  size_type result = 0;
  distance(begin(),end(),result);
  return result;
}
reference front(){return *begin();}
reference back(){return *(--end());}

```

**注意：STL遵循前闭后开的区间原则，[first,last)表示last不在此范围内**

---

## deque （具体数据结构未了解）
vector是单向开口的连续线性空间，而deque则是一种双向开口的连续线性空间。vector虽然也可以进行头插，但是效率太低无法被接受。

deque是由一段一段的定量连续空间构成。一旦有必要在deque的前端或尾端增加新空间，便配置一段定量连续空间，串接在整个deque的头端或尾端。deque的最大任务，便是在这些分段的定量连续空间上，维护其整体连续的假象，并提供随机存取的接口。

---


## stack
其是具有先进后出概念的一种数据结构，因为其性质不能遍历访问，所以无需迭代器，又因deque和list跟stack完美满足，所以只需稍微改进这两个数据结构便可以定义stack。

```
stack<int,list<int>> istack;
istack.push(1);
istack.push(3);
istack.push(5);
istack.push(7);
```

---

## queue
queue是一种先进先出的数据结构，queue也不允许遍历，并且最顶端只能取出元素，最低端只能加入元素。queue默认使用deque作为序列容器，但是也可以显示声明使用list。
```
queue<int,list<int>> iqueue;
iqueue.push(1);
iqueue.push(3);
iqueue.push(5);
iqueue.push(7);
```

---

## 最大堆和最小堆
最大堆和最小堆是优先队列的两个具体实现，它们使用了vector来隐式表达完全二叉树，再配合一定的节点调整算法（上溯算法）。最大堆的条件是每个节点上的值都大于或者等于其子节点的值，最小堆反之。

插入时将节点插入到vector的尾端节点再使用上溯算法进行调整。

取出时将根节点取出并将尾端节点放入根处使用下溯算法进行调整。

heap理所当然的没有迭代器。


---

# 关联式容器
关联式容器中的每一笔数据都有一个键值(key)和一个实值(value)。当元素被插入到关联式容器中时，容器内部结构便依照其键值大小，以某种特定规则将这个元素放于适当地位置。STL容器中的关联式容器都以红黑树作为实现，红黑树可以参考Algorithm中的tree.md中的描述。

## set
set中的实值就是其键值，所以不允许有两个相同的键值，我们无法使用set的迭代器去改变set的元素值，因为其关系到set的排列规则，所以set的迭代器被定义为静态的。（红黑树中实现了两种插入，可重复插入和不可重复插入）

几乎所有的对set的操作，最后都是转调用RB-tree的操作。

---

## map
map的特性是，所有元素都会根据元素的键值自动被排序。map的所有元素都是pair，同时拥有实值(value)和键值(key)。pair的第一元素被视为键值，第二元素被视为实值。

键值作为排序规则的载体，不允许被更改，而实值则可以使用迭代器进行更改。

map不允许拥有两个相同的键值。

几乎所有的对map的操作，最后都是转调用RB-tree的操作

---

## multiset
multiset的特性以及用法和set完全相同，唯一的差别在于它允许键值重复，插入操作调用的是底层的`insert_unique()`函数。

---

## multimap
同上

---

## hashtable
二叉搜索树具有对数平均时间的表现，但这样的表现是假设输入数据有足够的随机性。而hash表可以不用这样的随机性假设也可以达到对数平均时间。

由于hash函数是一个多对一的映射关系，所以再好的hash函数也难免会发生hash碰撞，解决碰撞的方法有多种，包括线性探测散列、二次探测、开链等做法，每一种方法的到处效率各不相同--与array的填满成都有很大的关联。

填充因子：负载系数，表格大小和表格中元素个数比，通常取0.5

#### 线性探测(linear probing)
最为传统的方法，将产生冲突的元素放入之后的表格中，平均情况是寻访一半的表格。

#### 二次探测(quadratic probing)
线性探测遇到冲突采取的是H+1,H+2依次对比，二次探测则是$H+1^{2}$,$H+2^{2}$ 进行探测。

#### 开链(separate chaining)
在每一个表格元素中维护一个list，如果冲突了则放入list中。STL就是采用这种做法。

以下是hashtable的部分定义
```

template<class Value>
struct __hashtable_node{
  __hashtable_node* next;
  Value val;
};


template<class Value, class Key, class HashFcn,
        class ExtractKey, class EqualKey,
        class Alloc>
class hashtable{
public:
  typedef HashFcn hasher;
  typedef EqualKey key_equal;
  typedef size_t size_type;

private:
  hasher hash;Hash函数
  key_equal equals;//判断键值是否相同的方法
  ExtractKey get_key;//取出键值的方法

  typedef __hashtable_node<Value> node; //hash元素节点
  typedef simple_alloc<node, Alloc> node_allocator; //节点内存分配器

  vector<node*, Alloc> buckets;//表格由vector完成
  size_type num_elements; //元素数量

public:
  size_type bucket_count() const {return buckets.size();}
};

```
- Value: 节点的实值类型
- Key: 节点的键值类型
- HashFcn: 函数函数
- ExtractKey: 取出键值的方法
- EqualKey: 判断键值相同的方法
- Alloc: 空间配置器

hashtable在构造时会从28个质数中找到第一个比n大的数来进行vector buckets的构造。

表格进行重建时，首先会更新vecotr buckets，之后会计算表格中每一个__hashtable_node在新表格中的位置，然后进行迁移，最后将新旧两个buckets对调在释放内存。

## hashset
hashset底层也使用hashtable为底层机制，几乎所有的hashset操作，都只是转调用hashtable的操作行为而已。 唯一和set的不同就是hashset中没有自动排序。

## hashmap
hashmap的操作和map的几乎完全相同，唯一不同的就是hashmap中没有自动排序功能。

## hashmultimap和hashmultiset
同上
