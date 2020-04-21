# Effective Modern C++

## 型别推导

### 模板型别推导

- 在模板型别推导过程中，具有引用型别的实参会被当成非引用型别来处理。引用性会被忽略。
- 万能引用形参（T&& param），左值会推导为左值引用，右值会按照常规来。
- 按值传递的形参推导，const不会传递。
- 在值传递形参中，数组和函数型别的实参会退化为指针。但是在引用形参下不会。



### auto

- auto和模板型别推导一般是一模一样的，但是如果用大括号初始化表达式，模板型别是会报错，而auto则是使用了初始化列表模板（std::initializer_list）进行。
- 在函数返回值或者lambda表达式形参中使用auto，意思是使用模板型别推导而不是auto型别推导。



### 理解decltype

- 一般情况下decltype会得出变量或表达式的型别而不做出任何修改。
- 对于型别为T的左值表达式（x）为左值表达式，decltype((x))会得到T&。
- decltype(auto)的意思是auto制定了要推导的型别，decltype指定了使用decltype规则。这个规则只在C++14中有。

C++返回值类型尾序语法：这样的好处是可以制定返回值为形参的组合类别。

```c++
template<typename Container, typename Index> //C++11
auto authAndAccess(Container&& c,Index i)//万能引用是为了适应左右值的情况
-> decltype(std::forward<Container>(c)[i])
{
	authenticateUser();
	return std::forward<Container>(c)[i];//forward<>是为了适应万能引用
}

template<typename Container, typename Index> //C++14
decltype(auto)
authAndAccess(Container&& c,Index i)
{
    authenticateUser();
    return std::forward<Container>(c)[i];
}

```

