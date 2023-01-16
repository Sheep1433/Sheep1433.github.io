---
title: map函数的使用
date: 2022-03-04 20:42:13
tags: python基础
---

**map()** 会根据提供的函数对指定序列做映射。

map() 函数语法：

```python
map(function, iterable, ...)
```

python2.x版本中，返回的是列表

```python
map(square, [1,2,3,4,5])
```

python3.x版本中，返回的是迭代器

```python
map(square, [1,2,3,4,5])
<map object at 0x100d3d550> 
list(map(square, [1,2,3,4,5]))
[1, 4, 9, 16, 25]
```



