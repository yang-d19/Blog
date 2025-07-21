---
title: 工作 Debug 日记
typora-root-url: ./
categories: [Technology]
tags: [笔记, 工作日志]
date: 2025-07-17 23:56:08
---

记录一些工作上遇到的，花的时间较多的 bug。大部分和 C++ 相关。

<!--more-->

## 不要使用临时变量的迭代器

### 样例

```c++
std::vector<int> array = {1, 2, 3, 4, 5};

std::vector<int> GetArray() {
    return array;
}

int main() {
    auto new_array = std::vector<int>(
        GetArray().begin() + 1,
        GetArray().end()
    );
    // Expect new_arr == {2, 3, 4, 5}
}

```

### 分析

上面代码的预期行为是：new_array 拷贝了 array 除了首元素的其他所有元素。因此 new_array 的大小应该等于  array 的大小减一。

然而在 g++ 编译环境下，这段代码在编译时就会直接报错：

```
terminate called after throwing an instance of 'std::length_error'
  what():  cannot create std::vector larger than max_size()
Aborted (core dumped)
```

在公司服务器的编译环境下（貌似是魔改版的 gcc），编译不会报错，但是运行时会发现，new_array 的大小超过了 array 大小的两倍，也是不对的。

错误原因是 `GetArray()` 函数每次都返回一个临时对象，调用 `GetArray().begin()`  和 `GetArray().end()` 时，得到的是临时对象 1 的 begin() 迭代器和临时对象 2 的 end() 迭代器。传入两个不属于同一对象的迭代器给 std::vector 的构造函数是非法的。

### 修正

```c++
std::vector<int>& GetArray() {
    return array;
}
```

## 在迭代访问容器内元素时，不要更改当前容器

这个 bug 我遇到了两次，两次掉进了同一个坑 :crying_cat_face:

### 样例

```c++
std::unordered_set<int> my_set = {1, 2, 3, 6, 7, 9, 14, 15, 23, 27};

std::vector<int> my_vec = {1, 2, 3, 4, 5};

int main() {
    for (auto elem : my_set) {
        int new_elem = elem + 1;
        my_set.insert(new_elem);
    }

    for (auto& elem : my_vec) {
        my_vec.push_back(1);
	}
        
    return 0;
}
```

### 分析

在迭代遍历 my_set 容器时，如果向容器中添加了新元素，大部分情况下这可能不会带来毁灭性的影响。但是如果某次添加引发了容器的扩容机制，这会导致当前容器中可能所有元素的地址都会发生变化，这时再使用迭代器访问 下一个元素可能会得到一个错误地址，访问内存异常而造成 segment fault。

遍历 my_vec 容器时，如果向容器中添加了新元素，也可能会有类似的影响。vector 的扩容机制可能会导致当前元素的地址失效，也就是 elem 元素的值在 push_back 操作后可能会发生变化（变成容器中其他元素的值甚至变为任意值）。

### 修正

别在迭代容器时，改变容器本身。

