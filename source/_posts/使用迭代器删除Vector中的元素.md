---
title: 使用迭代器删除vector中的元素
tags: 
- c
- c++
- STL
- 优雅的程序
---
# C++中的迭代器

迭代器是一种检查容器内元素并遍历元素的数据类型。C++更趋向于使用迭代器而不是下标操作，因为标准库为每一种标准容器（如`std::vector`）定义了一种迭代器类型，而只用少数容器（如`std::vector`）支持下标操作访问容器元素。

# 删除`std::vector`中的元素

`std::vector`删除元素一共提供了三个成员函数：`clear()`、`pop_back()`和`erase()`

1. 使用`clear()`时，`vector`被完全清空，不留任何元素
2. 使用`pop_back()`时，`vector`只会从最后面删除一个元素
3. 使用`erase()`时，`vector`按照给定的迭代器删除元素

前两个成员函数的有很强的局限性，可能不能完全符合使用者的意图。但是使用第三个成员函数的时候，一不小心也容易被坑进去。比如比较常用的删除操作：

<!--more-->

```c++
std::vector<int> ev = {1,2,3,4,5};
for (auto iter = ev.begin(); iter != ev.end(); ++iter){
  if (3 == (*iter))
    ev.erase(iter);
}
```

在真机模拟的时候会发现这个程序会崩溃。~~这难道不对吗？~~ 假如说单独使用`erase()`函数这样是没问题的，但是呢，在`for`循环中使用，就是错的。原因就是，当`erase()`之后，迭代器就指向了空，也就是链断了，当再次使用的时候，肯定就会出错。下面这个才是正确的：

```c++
std::vector<int> ev = {1,2,3,4,5};
auto iter = ev.begin();
for (; iter != ev.end();) {
  if (3 == (*iter))
    iter = ev.erase(iter);
  else
    ++iter;
}
```

# 更优雅的删除

先来看一段程序：
```c++
std::vector<int> ev = {1,2,3,4,5};
ev.erase(std::remove(ev.begin(), ev.end(), 3), ev.end());
```

先来看看`std::remove()`方法的声明：
```c++
template<class ForwardIterator, class T>
ForwardIterator remove(ForwardIterator first, ForwardIterator last, const T& value);
```
`std::remove()`接受三个参数：`first`迭代器、`last`迭代器和任意类型的`value`。这个方法将从`first`迭代器到`last`迭代器中的元素的值与`value`比较，假如相等的话，则将该迭代器指向的元素换到最末尾，而其返回值则是迭代器指向未移动的最后一个元素的下一个位置。更详细的解释[在这里](https://www.cnblogs.com/jingyg/p/5613303.html)。

那这样的话，上面的那个程序就比较好理解多啦。**遍历`ev`并且将其元素等于3的值移动到末尾，然后将所有等于3的元素全部删除**。这样的话就不需要考虑由于`for`循环的失误导致程序的崩溃了。

# 优雅的删除的扩展

既然可以通过`std::remove()`实现比较优雅的删除，~~那么有没有更加自由的呢？~~这不是废话嘛。先来一段代码：

```c++
struct ev_type_t {
  unsigned id;
  unsigned age;
};

std::vector<ev_type_t> ev = {{1,1}, {2, 2}, {3, 3}, {4, 2}};
//! 删除年龄是2的
ev.erase(std::remove_if(ev.begin(), ev.end(), [=](const ev_type_t& ev){return (unsigned)3 == ev.age);}), ev.end);
```

使用`std::remove_if()`在配合上lambda表达式，实现自由度更高的优雅删除。
