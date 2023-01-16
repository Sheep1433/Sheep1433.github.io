---
title: MySQL中limit和offset的用法
date: 2022-03-06 19:26:21
tags: MySQL
---

```python
select* from article LIMIT 1,3 等价于 select * from article LIMIT 3 OFFSET 1	取出第2-4条数据

select* from article LIMIT 0,3  等价于 select* from article LIMIT 3  
```

