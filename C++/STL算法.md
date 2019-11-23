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

---

# copy算法详解
copy算法因为常常被调用，其作用是对一块内存上的数据进行复制，所以在效率上自然是越高越好，为此SGI STL的copy算法用尽各种办法，包括函数重载(function overloading)、类型特性(type traits)、偏特化(partial specialization)等编程技巧。下面来讲解这些优化。

copy算法可将输入区间[first,last)内的元素复制到输出区间[result,result+(last - first))内，也就是说，它会执行赋值操作`*result = *first; *(result+1) = *(first); ...`依次类推。使用copy算法，你可以将任何容器的任何一段区间上的内容，复制到任何容器的任何一段区间上。

copy的赋值操作是前向推进，依次复制的，也就是说，当区间是[first,result,last)这种情况时，可能会导致错误结果，如下：
```
//array:

    ↓first ---  --- --- last↓
0   1   2   3   4   5   6   7   8
                ↑result ---  ---- →

// after copy   ↓   ↓   ↓   ↓   ↓    逐个复制导致的结果


0   1   2   3   2   3   2   3   2   
```

当然也可能不会出现这种错误，如果copy函数调用`memmove()`来执行任务的话，会先将整个输入区间的内容复制下来，就不会出现上面所描述的危险。

下面我们从源码分析copy函数，以下是copy的唯一对外接口
```
template<class InputIterator, class OutputIterator>
inline OutputIterator copy(InputIterator first, InputIterator last,
                            OutputIterator result){
    return __copy_dispatch<InputIterator, OutputIterator>()(fist, last, result);
}
```
除了上述的copy函数，还有两个重载版本，针对原生指针`const char*`和`const wchat_t*`，会直接进行内存拷贝操作：
```
inline char* copy(const char* first, const char* last, char* result){
  memmove(result, first, last - first);
  return result + (last - first);
}

inline wchar_t* copy(const wchar_t* first, const wchar_t* last, char* result){
  memmove(result, first, sizeof(wchar_t)* (last - first));
  return result + (last - first);
}

```
上面这两个函数不载讨论，下面主要讨论`__copy_dispatch()`，此函数有一个完全泛华版本和两个偏特化版本：
### 完全泛华版本
```
template <class InputIterator, class OutputIterator>
struct __copy_dispatch{
  OutputIterator operator()(InputIterator first, InputIterator last,
                            OutputIterator result){
      return __copy(first, last, result, iterator_category(first));
  }
}
```

完全泛华版本会根据迭代器的类型，为不同迭代器循环条件不同
```
// InputIterator版本
template<class InputIterator, class OutputIterator>
inline OutputIterator __copy(InputIterator first, InputIterator last,
                             OutputIterator result, input_iterator_tag){
    for(;first != last;++result,++first){
      *result = *first;
    }                               
}

// randomAccessIterator版本
template<class RandomAccessIterator, class OutputIterator>
inline OutputIterator __copy(RandomAccessIterator first, RandomAccessIterator last,
                             OutputIterator result, random_access_iterator_tag){
    // 其他地方可能用到
    return __copy_d(first, last,result,distance_type(first));                               
}

template<class RandomAccessIterator, class OutputIterator， class Distance>
inline OutputIterator __copy_d(RandomAccessIterator first, RandomAccessIterator last,
                               OutputIterator result, Distance*){
    for(Distance n = last - first; n > 0; --n, ++result, ++first){
      *result = *first;
    }                  
    return result;               
}
```
可以看到InputIterator是使用的迭代器判断，而RandomAccessIterator是使用了n判断循环次数，有关迭代器的类型以及特点可以在迭代器那篇文章中查看。

### 偏特化版本
偏特化是指在完全泛化的情况下，针对某一个泛化类型，有更好的实现方式时特化这个类型。
```
//偏特化会在名称之后加入<>进行特化指定
template<class T>
struct __copy_dispatch<T*,T*>{
  T* operator()(T* first, T* last, T* result){
    typedef typename __type_traits<T>::has_trivial_assignment_operator t;
    return __copy_t(first, last, result, t());
  }
}

template<class T>
struct __copy_dispatch<const T*, T*>{
  T* operator()(const T* first, const T* last, T* result){
    typedef typename __type_traits<T>::has_trivial_assignment_operator t;
    return __copy_t(first, last, result, t());
  }
}



//以下版本适用于“指针对象具备trivial assignment operator”
template<class T>
inline T* __copy_t(const T* first, const T* last, T* result,
                   __true_type){
    memmove(result, first, sizeof(T)*(last - first));
    return result + (last - first);
}

//以下版本适用于“指针对象具备non-trivial assignment operator”
template<class T>
inline T* __copy_t(const T* first, const T* last, T* result,
                   __false_type){
    //原生指针可以看做是一种RandomAccessIterator
    return __copy_d(first, last, result, (ptrdiff_t*) 0);
}

```

在偏特化的版本中，会判断指针所指的是否具有trivial assignment operator（无意义复制函数），如果是的话，可以直接使用内存拷贝方法memmove()完成，
如果不是的话则一定得通过non-trivial assignment operator进行复制。C++语言本身无法得知一个类是否有non-trivial assignment operator，STL是使用了__type_traits<>编程技巧来弥补从而得到的。

#### `__type_traits`
`__type_traits`提供了一种机制，允许针对不同的类型属性，在编译时期完成函数分发（function dispatch）
它属于一个结构体，定义如下：
```
template<class T>
struct __type_traits{
  typedef __true_type this_dummy_member_must_be_first;

  typedef __false_type has_rivial_default_constructor;
  typedef __false_type has_trivial_copy_constructor;
  typedef __false_type has_trivial_assignment_operator;
  typedef __false_type has_trivial_destructor;
  typedef __false_type is_POD_type;
}

struct __true_type{};
struct __false_type{};
```

通过`__type_traits<T>::has_trivial_destructor t()`来判断该类型是否具有non-trivial assignment operator，如果没有则可以直接进行内存拷贝，如果有的话则使用`__copy_d()`进行拷贝

---

简单的做个总结，copy函数会根据你的迭代器类型，数据类型等进行优化，选择使用直接复制内存还是逐个进行遍历赋值。
