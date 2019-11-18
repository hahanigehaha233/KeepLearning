# STL算法概述
STL中收录了大概70多种算法，包括了排序、查找、排列组合等，它们通过迭代器在
$[first,last)$ 中标记出来，这些算法可以分为质变算法和非质变算法。

#### 质变算法(mutating algorithm)
在操作过程中会改变区间内的元素内容。诸如拷贝(copy)、互换(swap)、替换(replace)、填写(fill)、删除(remove)、排列组合(permutation)、分割(partition)、随机重排(random shuffling)、排序(sort)等。

这些算法无法运用于常数区间，不然有你好果汁吃。

#### 非质变算法(nonmutating algorithm)
相反，是指运算过程中不会改变区间内元素的内容。诸如查找(find)、匹配(search)、计数(count)、巡访(for_each)、比较(equal、mismatch)、极值(max、min)算法。

---

所有的算法前两个参数都是一对迭代器，通常称为first和last，用以标识算法的操作区间。STL习惯采用前闭后开区间表示法[first,last)，表示区间涵盖first至last（不含last）之间的所有元素。当first==last时，上述所表现的便是一个空区间。

在这个区间中，first经过累加运算后一定可以到达last，不然算法执行会产生错误。

质变算法通常提供两个版本，一个是in-place（就地进行）版，就地改变其操作对象，一个是copy版，将操作对象的内容复制一份副本，然后在副本上进行修改并返回该副本。copy版总是以`_copy`作为函数名称尾词，例如`replace()`和`replace_copy()`。`sort()`就没有copy版。

算法的泛华是用迭代器作为桥梁的，算法在操作对象时，并不会直接读写对象地址，而是通过迭代器的接口获得对象数值。
