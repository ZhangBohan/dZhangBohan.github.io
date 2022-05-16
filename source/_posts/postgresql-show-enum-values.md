title: PostgreSQL显示枚举类型内容列表
date: 2016-05-03 11:15:05
tags:
- PostgreSQL
---

PostgreSQL中如何查看枚举类型的值列表呢？

举个例子，执行如下SQL：
```sql
CREATE TYPE rainbow AS ENUM ('red', 'orange', 'yellow', 'green', 'blue', 'purple');
```

## 顺序返回全部枚举值
```sql
SELECT enum_range(NULL::rainbow);
    enum_range
------------------
{red,orange,yellow,green,blue,purple}
(1 row)
```

参考：[Enum Support Functions](http://www.postgresql.org/docs/9.5/interactive/functions-enum.html)
